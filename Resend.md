---
tags:
  - software-development
  - email
  - resend
  - reference
updated: 2026-08-19
status: active
documentation-index: https://resend.com/docs/llms.txt
---
# Resend

> [!abstract] Resend Operations Reference
> Secure setup, sandbox limits, delivery testing, Supabase SMTP, application transport, production-domain readiness, and webhook safety.
>
> **Documentation index:** [resend.com/docs/llms.txt](https://resend.com/docs/llms.txt)  
> **Sandbox sender:** `onboarding@resend.dev` for controlled tests only

Related note: [[Software Delevopment|Software Development]]

## <span style="color:rgb(0, 112, 192)">Documentation Strategy</span>

1. Start with the [Resend documentation index](https://resend.com/docs/llms.txt).
2. Open only the relevant targeted Markdown (`.md`) pages.
3. Check the current page before a provider or security decision.
4. Do not load `llms-full.txt` unless a complete offline snapshot is necessary.

## <span style="color:rgb(0, 176, 80)">Secure Setup</span>

1. Create a Resend account and enable native MFA from [Profile](https://resend.com/profile).
2. Create one key for one environment and purpose. Prefer **Sending access** and a domain restriction when available.
3. Store application access in the server-only `RESEND_API_KEY` variable.
4. Store a Supabase Auth SMTP credential only in hosted Supabase Auth SMTP settings.
5. Keep local environment files out of Git. Never place a key in browser code, source code, logs, tickets, or notes.
6. Use sandbox checks before a verified domain exists.
7. Add a signature-verified webhook before production callback processing.

> [!danger] Secret Rule
> Do not record API keys, SMTP values, webhook secrets, MFA seeds, recovery data, Auth tokens, full Auth links, recipient lists, or message content in project evidence.

### Official SDKs

<table style="width:100%; border-collapse:collapse; margin:8px 0 16px;">
<thead>
<tr>
<th style="padding:8px; border-bottom:1px solid #d0d7de; text-align:left;">Stack</th>
<th style="padding:8px; border-bottom:1px solid #d0d7de; text-align:left;">Package or module</th>
</tr>
</thead>
<tbody>
<tr><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;">Node.js / TypeScript</td><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;"><code>resend</code></td></tr>
<tr><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;">Python</td><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;"><code>resend</code></td></tr>
<tr><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;">PHP</td><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;"><code>resend/resend-php</code></td></tr>
<tr><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;">Ruby</td><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;"><code>resend</code></td></tr>
<tr><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;">Go</td><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;"><code>github.com/resend/resend-go/v3</code></td></tr>
<tr><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;">Rust</td><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;"><code>resend-rs</code></td></tr>
<tr><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;">Java</td><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;"><code>resend-java</code></td></tr>
<tr><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;">.NET</td><td style="padding:7px 8px; border-bottom:1px solid #e6e8eb;"><code>Resend</code></td></tr>
</tbody>
</table>

See the [official SDK list](https://resend.com/docs/sdks.md) for current versions and repositories.

### Server Environment

```env
RESEND_API_KEY=re_xxxxxxxxx
```

Keep email sending on the server. Do not expose this key through `NEXT_PUBLIC_*` or another client variable.

## <span style="color:rgb(255, 140, 0)">Sandbox Boundary</span>

> [!warning] Recipient Restriction
> `onboarding@resend.dev` can send a real message only to the email address attached to the Resend account. Another real recipient returns HTTP `403`.

- Treat HTTP `403` from this sandbox restriction as a permanent configuration failure.
- Do not retry the request.
- Test messages count against the account quota.
- Keep normal unit tests network-free.
- Run provider checks only through an explicit integration or staging command.

### Sandbox Test Matrix

<table style="width:100%; border-collapse:separate; border-spacing:0 8px;">
<colgroup>
<col style="width:27%;">
<col style="width:28%;">
<col style="width:45%;">
</colgroup>
<thead>
<tr>
<th style="padding:8px; border-bottom:1px solid #d0d7de; text-align:left;">Recipient</th>
<th style="padding:8px; border-bottom:1px solid #d0d7de; text-align:left;">Expected event</th>
<th style="padding:8px; border-bottom:1px solid #d0d7de; text-align:left;">Required handling</th>
</tr>
</thead>
<tbody>
<tr><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;"><code>delivered@resend.dev</code></td><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;"><code>email.delivered</code></td><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;">Record provider acceptance and delivery. Do not treat this as proof of inbox placement or an open.</td></tr>
<tr><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;"><code>bounced@resend.dev</code></td><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;"><code>email.bounced</code></td><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;">Classify SMTP <code>550 5.1.1</code> as permanent and stop future sends.</td></tr>
<tr><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;"><code>complained@resend.dev</code></td><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;"><code>email.complained</code></td><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;">Record the complaint and stop future sends.</td></tr>
<tr><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;"><code>suppressed@resend.dev</code></td><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;"><code>email.suppressed</code></td><td style="padding:9px 8px; border-bottom:1px solid #e6e8eb;">Do not retry. Inspect the suppression reason. This address does not support labels.</td></tr>
</tbody>
</table>

Use labels for distinct runs:

```text
delivered+<flow>-<run-id>@resend.dev
bounced+<flow>-<run-id>@resend.dev
complained+<flow>-<run-id>@resend.dev
```

## <span style="color:rgb(0, 112, 192)">Supabase Auth SMTP</span>

| Setting | Value |
|---|---|
| Host | `smtp.resend.com` |
| Port | `465` |
| Username | `resend` |
| Password | Resend API key stored only in Supabase Auth settings |
| Sandbox From | `onboarding@resend.dev` |
| Sandbox real recipient | Resend account email only |

### Direct Auth-Link Proof

Use one controlled password-recovery message to the Resend account email.

**Accepted link shape:**

```text
https://<supabase-project-ref>.supabase.co/auth/v1/verify
  ?type=recovery
  &redirect_to=https://<staging-host>/auth/callback
```

**Rejected link shape:**

```text
https://links.<domain>/...
https://resend.<domain>/...
https://<other-tracking-host>/...
```

Evidence must record only the host and path. Redact the token and full query string.

## <span style="color:rgb(0, 176, 80)">Portable API Test</span>

Use the official SDK in application code. Use this raw request only for a controlled one-off transport check:

```bash
curl --request POST "https://api.resend.com/emails" \
  --header "Authorization: Bearer $RESEND_API_KEY" \
  --header "Content-Type: application/json" \
  --header "User-Agent: resend-test/1.0" \
  --data '{
    "from": "Test <onboarding@resend.dev>",
    "to": ["delivered@resend.dev"],
    "subject": "Resend transport test",
    "text": "The Resend request works."
  }'
```

A successful API response proves only that Resend accepted the request. Use provider events for the final transport outcome.

## <span style="color:rgb(112, 48, 160)">Sending Contract</span>

A single email needs `from`, `to`, `subject`, and one content source: `html`, `text`, `react`, or a published `template`.

- Do not combine a hosted `template` with `html`, `text`, or `react`.
- Use one stable idempotency key for one logical email, such as `welcome-email/user-123`.
- Idempotency keys expire after 24 hours and have a maximum length of 256 characters.
- Reusing a key with a different payload causes a conflict.
- For the Node.js SDK, check the returned `{ data, error }` result.
- Retry only temporary failures, such as rate limits and server errors.
- Use the same idempotency key for each safe retry.

## <span style="color:rgb(255, 140, 0)">Production Domain</span>

1. Approve the root domain, DNS provider, DNS operator, and transactional sender subdomain.
2. Select the sending region nearest to the expected recipients.
3. Add the exact DKIM, SPF, and return-path records supplied by Resend.
4. Review DMARC without weakening a valid existing policy.
5. Keep open and click tracking disabled for Auth and sensitive transactional email.
6. Approve the final Auth From, application From, monitored Reply-To, and DMARC report addresses.
7. Use separate production SMTP and application API credentials.
8. Verify Resend status, public DNS, direct Auth links, and SPF/DKIM/DMARC headers.

Generic sender examples:

| Purpose | Address |
|---|---|
| Auth | `Product Name <auth@mail.example.com>` |
| Application | `Product Name <orders@mail.example.com>` |
| Replies | `support@example.com` |

## <span style="color:rgb(192, 0, 0)">Webhook Safety</span>

A production webhook handler must:

- Read the raw request body before JSON parsing.
- Verify `svix-id`, `svix-timestamp`, and `svix-signature` with `RESEND_WEBHOOK_SECRET`.
- Reject an invalid signature before business work.
- Store processed event IDs because delivery is at least once.
- Use the provider event time when order matters because delivery order is not guaranteed.
- Keep all state changes idempotent.
- Return HTTP `200` only after successful processing.

Useful events include `email.sent`, `email.delivered`, `email.bounced`, `email.complained`, `email.suppressed`, `email.failed`, and `email.delivery_delayed`.

## <span style="color:rgb(192, 0, 0)">Security Checklist</span>

- [ ] Use separate credentials for development, staging, and production.
- [ ] Grant only the required permission.
- [ ] Keep application keys out of client bundles.
- [ ] Keep SMTP credentials only in the approved secret store.
- [ ] Keep local secret files out of Git.
- [ ] Verify every webhook signature before processing.
- [ ] Replace a key before deleting the old key during rotation.
- [ ] Rotate immediately after suspected exposure.
- [ ] Keep recipients and message content out of operational logs.

## <span style="color:rgb(0, 112, 192)">Targeted Sources</span>

- [Documentation index](https://resend.com/docs/llms.txt)
- [Multi-factor authentication](https://resend.com/changelog/multi-factor-authentication.md)
- [Account quotas and limits](https://resend.com/docs/knowledge-base/account-quotas-and-limits.md)
- [Sandbox recipient restriction](https://resend.com/docs/knowledge-base/403-error-resend-dev-domain.md)
- [Send test emails](https://resend.com/docs/dashboard/emails/send-test-emails.md)
- [Send through Supabase SMTP](https://resend.com/docs/send-with-supabase-smtp.md)
- [Supabase custom SMTP](https://supabase.com/docs/guides/auth/auth-smtp)
- [Add and verify a domain](https://resend.com/docs/add-a-domain.md)
- [Open and click tracking](https://resend.com/docs/dashboard/domains/tracking.md)
- [Manage webhooks](https://resend.com/docs/webhooks/introduction.md)
- [Verify webhook requests](https://resend.com/docs/webhooks/verify-webhooks-requests.md)
- [Handle API keys](https://resend.com/docs/knowledge-base/how-to-handle-api-keys.md)
