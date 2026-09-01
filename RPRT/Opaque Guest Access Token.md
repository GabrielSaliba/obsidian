
[[Studio Repertório]]
[[Auth Users and Profiles]]

> [!example] Visual Explainer
> [Open the account and guest order journey visual explainer in Linear](https://uploads.linear.app/445023d5-264c-47cb-a70c-e4a15a58da5c/5caadee1-c691-4e53-ae4c-623fa2e1e62a/195367a3-e015-4fed-acfa-5cfd92ff2d7b)

> [!abstract] Definition

> An opaque token is a random secret with *no business information inside it*. The application cannot obtain meaning by decoding it. It is only a secret lookup credential.


## <span style="color:rgb(112, 48, 160)">Token Requirements</span>

Example:

```text

n4w8Vv7rQm2...random-256-bit-value

```

The token must not be:

- The order ID
- The order number
- The customer email
- A readable JSON object
- A predictable value

The token must contain at least **256 bits of cryptographic randomness**.

## <span style="color:rgb(0, 176, 80)">Creation Flow</span>

When the atomic `create_order` command creates a guest order:

1. The server generates a cryptographically secure random token.
2. The server calculates a one-way hash of the token.
3. PostgreSQL stores the hash, not the raw token.
4. The raw token is returned once to the server.
5. The server gives the browser immediate access.
6. The server includes an access link in the order email.

  

### Token Storage

  

```text

Raw token:
n4w8Vv7rQm2...secret

Stored value:
SHA-256(n4w8Vv7rQm2...secret)

```

  
Example table:

```sql

create table public.guest_order_access (

  id uuid primary key default gen_random_uuid(),

  order_id uuid not null references public.orders(id) on delete cascade,

  token_hash text not null unique,

  expires_at timestamptz not null,

  revoked_at timestamptz,

  created_at timestamptz not null default now()

);

```

  
## <span style="color:rgb(0, 176, 240)">Email Access Flow</span>

Example link:
```text

https://shop.example.com/order-access?token=<raw-token>

```

  
When the link opens:
1. A Route Handler reads the token.
2. It hashes the supplied token.
3. It finds the matching `guest_order_access` row.
4. It checks expiration and revocation.
5. It creates a secure, short-lived guest order session.
6. It stores the session in an `HttpOnly`, `Secure`, `SameSite=Lax` cookie.
7. It redirects to a clean URL.

  
```text

https://shop.example.com/pedidos/REP-12345

```

<mark style="background: #ADCCFFA6;">The clean URL must not contain the secret token.</mark>

> [!important] Token Exposure

> The access-token route must set a strict referrer policy and must never log the query string.


This exchange reduces token exposure through:
- Browser history
- Screenshots
- Copied URLs
- Referrer headers
- Analytics tools
- Application logs

## <span style="color:rgb(255, 192, 0)">Immediate Checkout Access</span>

  
The guest should not need to open the email after creating an order.
After successful order creation:

1. The Server Action creates the guest order session cookie immediately.
2. The browser opens the clean order status page.
3. The email token remains available as a recovery mechanism for another browser or device.
## <span style="color:rgb(0, 176, 240)">Capability Scope</span>

<mark style="background: #BBFABBA6;">The guest capability must be narrow, temporary, read-only access</mark>

### Permitted Actions

- Read one order
- Read its payment status
- Read its shipment status
- Request another transactional order email, with rate limits

### Forbidden Actions

- Read another order
- Change the shipping address after payment
- Change prices or items
- Mark payment as approved
- Change shipment status
- Read customer profiles
- Read payment-provider payloads

## <span style="color:rgb(255, 0, 0)">Why Store Only the Hash?</span>

If the database is exposed, an attacker who obtains the stored hash cannot directly use it as the guest credential.


This follows the same principle as password storage:

- Store only the hash.
- Hash the supplied token.
- Compare the hashes.
- Never store the usable secret.

A fast SHA-256 hash is acceptable because the token has high random entropy. It is not a human-selected password and is not vulnerable to dictionary attacks.

## <span style="color:rgb(112, 48, 160)">Relationship With RLS</span>

Guest users do not have a Supabase `auth.uid()`. Ordinary ownership policies cannot identify them.


> [!warning] RLS Boundary

> Do not create broad RLS rules for anonymous order access.

### Guest Access

```text

Guest browser

  -> Secure guest session cookie

  -> Server Action or Route Handler

  -> Validate capability for one order

  -> Perform a narrowly scoped server-side read

```

### Authenticated Access

Authenticated customers continue to use normal RLS:

```sql

auth.uid() = orders.user_id

```

The guest and authenticated access models must remain separate.

## <span style="color:rgb(0, 176, 80)">Token Lifecycle</span>

Recommended lifecycle:

- Tokens expire after a defined period, such as 30 or 90 days.
- Access is revoked when the order is linked to a registered account.
- A replacement token invalidates the previous token.
- Support staff can revoke access.
- The raw token is never stored in logs.
- The raw token is never sent to Sentry.
- The raw token is never sent to analytics.
- The raw token is never included in webhook payloads.

The guest can still receive order updates by email after browser access expires.

## <span style="color:rgb(0, 176, 240)">Observability</span>

Wide events may include:

```json

{
  "actor_type": "guest",
  "order_id": "internal-order-id",
  "guest_session_id": "internal-session-id",
  "access_outcome": "granted"
}

```


Wide events must not include:

```json

{
  "token": "...",
  "token_hash": "...",
  "email": "...",
  "url_query": "..."
}

```


> [!danger] Logging Rule

> Never log the raw token, token hash, customer email, or URL query string.


## <span style="color:rgb(255, 192, 0)">Security Behavior</span>

Anyone who has the complete secret URL can access the permitted order details. The raw token is a bearer credential: possession proves access.

This is normal behavior for:
- Password-reset links.
- Email verification links.
- Magic login links.
- Guest order-status links.
- Private document-sharing links.
- It is secure only when the link is treated as a secret.

What the Database Hash Does
The stored hash protects against database exposure. It does not protect against theft of the raw URL.

Database leak
→ attacker gets only token hash
→ cannot use it as the access token

Email or URL leak
→ attacker gets raw token
→ can use it while valid


<mark style="background: #FF5582A6;">The key rule is:</mark>
**Anyone with the secret link can act as that guest, so the link must grant only narrow, temporary, read-only access.**
