---
aliases:
  - Sentry Source Maps
  - Sentry Setup Reference
tags:
  - RPRT
  - software-development
  - sentry
  - observability
  - source-maps
  - reference
type: integration-reference
status: conceptual
updated: 2026-09-01
---
# Sentry and Source Maps

> [!abstract] Observability Foundation
> This note explains the basic Sentry exception flow, the difference between a DSN and a source-map upload token, the conceptual Next.js integration shape, and the selected staging-only rollout for [[Studio Repertório]].

**Related notes:** [[Studio Repertório]] · [[Software Delevopment|Software Development]]

> [!example] Indexed Visual Explainer
> [Open the canonical Sentry and source maps visual explainer](https://uploads.linear.app/445023d5-264c-47cb-a70c-e4a15a58da5c/125bcf06-049a-498b-a259-10b1203dc4f4/de1c9633-b27f-4cab-be8e-d92bca992216). The HTML is attached to the Linear delivery record and has a synchronized physical copy in the vault.

---

## <span style="color:#80352F">Purpose</span>

Sentry receives unexpected application exceptions, groups related events into issues, connects each event to an environment and release, restores readable source locations, and sends operator alerts. It does not replace normal operational logs or durable business evidence.

## <span style="color:rgb(112, 48, 160)">Credential Roles</span>

| Credential | Purpose | Runtime |
|---|---|---|
| DSN | Routes exception events to the correct Sentry project | Application runtime |
| Source-map organization token | Uploads source maps and release artifacts | Trusted build or release process only |

The source-map token is a server-only secret. It must not enter browser code, Git, Linear, logs, screenshots, or test fixtures. Create upload tokens only when the implementation owns a verified upload path.

## <span style="color:rgb(0, 112, 192)">Source-Map Flow</span>

1. Next.js builds original source into minified JavaScript and source maps.
2. A trusted release step uses an organization token to upload source maps with an immutable release identity.
3. The deployed application sends an exception stack and the same release identity through the DSN.
4. Sentry joins the runtime event to the matching source map.
5. Operators see the original file, function, and line instead of only a minified bundle position.

## <span style="color:rgb(0, 176, 80)">Initial Environment Policy</span>

| Environment | Initial Sentry Reporting |
|---|:---:|
| Local | Off |
| Feature Preview | Off |
| Staging (`dev`) | On |
| Production | Off until production approval |

## <span style="color:rgb(255, 192, 0)">Scope Boundary</span>

The initial provider setup creates the Sentry organization and Next.js project, enables MFA, records operators, defines the alert inbox, selects the US data region, and records secret names and approved storage locations without values.

The later application implementation installs the SDK, configures browser, server, and edge exception capture, adds redaction and sampling, creates only the upload tokens required by the selected paths, and proves one mapped synthetic staging exception.

> [!warning] Conceptual State
> This reference does not state that the Sentry SDK, DSN, source-map upload, or alerts are implemented. Repository code and reviewed project documentation become authoritative after implementation.
