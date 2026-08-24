---
tags:
  - RPRT
  - database
  - identity
  - auth
  - profiles
  - addresses
  - RLS
  - schema
  - reference
type: schema-reference
---
[[Studio Repertório]]
[[Database Schemas]]
[[Administrator Membership Schema]]

# Auth Users, Profiles, and Addresses

> [!abstract] Identity Schema
> Supabase Auth owns login identity. Repertório stores customer profile data and saved Brazilian delivery addresses in application tables with owner-only access.

> [!warning] Source Of Truth
> The repository migrations, pgTAP tests, and `docs/database.md` are authoritative.

> [!example] Visual Explainer
> [[Visual Explainers/RPRT-47 Identity Schema Visual Explainer.html|Open the local identity schema visual explainer]]

## <span style="color:rgb(112, 48, 160)">Schema Boundary</span>

| Object | Owner | Purpose |
|---|---|---|
| `auth.users` | Supabase Auth | Login identity, email, providers, and authentication state |
| `public.profiles` | Repertório | Optional customer display name and canonical mobile phone |
| `public.addresses` | Repertório | Mutable saved Brazilian delivery addresses |
| `public.admin_users` | Repertório | Administrator membership; see [[Administrator Membership Schema]] |

Login email stays in `auth.users`. Profiles do not store email, CPF, passwords, provider identity, or authorization roles.

## <span style="color:rgb(0, 176, 240)">Relationship Model</span>

```mermaid
erDiagram
    AUTH_USERS ||--o| PROFILES : "owns"
    AUTH_USERS ||--o{ ADDRESSES : "saves"
    AUTH_USERS ||--o| ADMIN_USERS : "can hold membership"

    AUTH_USERS {
        uuid id PK
        text email
        timestamptz created_at
    }

    PROFILES {
        uuid user_id PK, FK
        text full_name "nullable"
        text phone "nullable"
        timestamptz created_at
        timestamptz updated_at
    }

    ADDRESSES {
        uuid id PK
        uuid user_id FK
        text recipient_name
        text phone
        text postal_code
        text street
        text number
        text complement "nullable"
        text district
        text city
        text state_code
        text country_code
        timestamptz created_at
        timestamptz updated_at
    }

    ADMIN_USERS {
        uuid user_id PK, FK
        timestamptz created_at
    }
```

## <span style="color:rgb(0, 112, 192)">Profiles</span>

`public.profiles.user_id` is both the primary key and a foreign key to `auth.users.id`.

| Column | Type | Rule |
|---|---|---|
| `user_id` | `uuid` | Required; one profile per Auth user; client cannot change it |
| `full_name` | `text` | Optional; trimmed; 1 to 200 characters when present |
| `phone` | `text` | Optional; canonical Brazilian mobile number in `+55...` format |
| `created_at` | `timestamptz` | Required; database-authored; client cannot change it |
| `updated_at` | `timestamptz` | Required; database-authored and advanced by trigger |

Authenticated owners can read their profile and update only `full_name` and `phone`. They cannot insert or delete a profile directly.

## <span style="color:rgb(0, 176, 80)">Addresses</span>

`public.addresses` supports many saved addresses for one Auth user.

| Field group | Columns | Rule |
|---|---|---|
| Identity | `id`, `user_id` | Database UUID and immutable Auth owner |
| Recipient | `recipient_name`, `phone` | Trimmed recipient name and canonical Brazilian mobile phone |
| Location | `postal_code`, `street`, `number`, `complement`, `district`, `city` | Brazilian delivery address with length and format checks |
| Region | `state_code`, `country_code` | Two uppercase state letters; country is fixed to `BR` |
| Time | `created_at`, `updated_at` | Database-authored timestamps |

Important constraints:

- `postal_code` contains exactly eight digits.
- `state_code` contains exactly two uppercase letters.
- `country_code` is always `BR`.
- Required text is trimmed and non-empty.
- `complement` is null or a trimmed value of at most 100 characters.
- `addresses_user_id_idx` supports owner lookups.

Authenticated owners can create, read, update, and delete only their own addresses. Column grants prevent changes to address identity, ownership, country, and timestamps.

## <span style="color:rgb(255, 192, 0)">Auth Initialization</span>

```mermaid
flowchart LR
    A[Insert auth.users row]
    T[on_auth_user_created trigger]
    P[Create minimal profile]
    O[Enqueue welcome-customer-email]
    C[Commit one transaction]

    A --> T
    T --> P
    P --> O
    O --> C
```

`private.handle_new_auth_user()` runs after an Auth user is inserted. In the same transaction, it:

1. Creates one minimal profile with database-authored time.
2. Enqueues one idempotent `welcome-customer-email` outbox job.
3. Uses the profile identity and normalized creation time as durable evidence.

The function ignores user metadata, uses `security definer` with an empty fixed `search_path`, and has no application-role execute grant. A profile or outbox failure rolls back Auth-user creation.

`private.set_updated_at()` supplies `updated_at` for profile and address changes. Application roles cannot execute it directly.

## <span style="color:rgb(192, 0, 0)">Access Control</span>

| Actor | Profiles | Addresses |
|---|---|---|
| Anonymous | No direct access | No direct access |
| Authenticated owner | Read own row; update `full_name` and `phone` | Create, read, update, and delete own rows |
| Authenticated other user | No access | No access |
| Administrator | No wider access than row ownership | No wider access than row ownership |
| `service_role` | No direct table grant | No direct table grant |

Both application tables have RLS enabled. Ownership uses `(select auth.uid()) = user_id`. Update policies use both `using` and `with check`.

## <span style="color:rgb(112, 48, 160)">Deletion And Retention</span>

- Auth user ID updates are restricted.
- Deleting an Auth user cascades to its profile and saved addresses.
- Saved addresses are mutable account data, not order history.
- Orders keep separate immutable contact and shipping snapshots.
- Guest order access belongs to the order schema, not the identity schema.

## <span style="color:rgb(0, 112, 192)">Repository Sources</span>

- `supabase/migrations/20260819205344_add_identity_access_schema.sql`
- `supabase/tests/identity_access.test.sql`
- `docs/database.md`

> [!summary] Core Rule
> Supabase Auth owns login identity. Repertório stores only application profile and address data, and RLS limits each customer to their own rows.
