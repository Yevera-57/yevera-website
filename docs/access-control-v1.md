# Yevera Access Control — implementation specification v1.0

Status: DESIGN ONLY. Authentication and authorization are NOT deployed. Never publish private reports until server-side checks pass.

## Authorization matrix
- star: administrator; read all profiles; only administrator can approve publication or change grants.
- bryan: read bryan and star profiles; no editing or publishing rights.
- future users: read only own profile, unless explicit administrator grant.
- anonymous: only items marked public and reviewed for public release.
- Default deny for unknown users, missing grants, missing classification, and errors.

## Authentication
Support both verified email one-time codes and Google OpenID Connect via an identity provider. Match verified email addresses to an explicit server-side account registry; a Google login alone must never imply access to a profile. Require strong session cookie security (HttpOnly, Secure, SameSite), short-lived sessions, CSRF protection, rate limits, and audit logs. Do not store OTPs or secrets in GitHub.

## Storage and enforcement
- Existing GitHub repository and static Cloudflare Worker assets are PUBLIC at the deployed website, even if GitHub repository is private.
- Do not put private report HTML, natal charts, names, email addresses, authorization lists, or private report URLs into public assets or public manifest.
- Store private reports outside the public asset directory in protected storage (e.g. private R2 bucket). Serve via Worker route only after server-side identity and authorization checks.
- Public manifest contains only approved public content. Private manifest is generated per authorized user server-side.
- Do not use UI hiding or client-side filtering as an access control.
- Deny direct static access to private paths and protect APIs. Verify unauthenticated requests and cross-profile access return 401/403; never return report content.
- Prevent caching of private responses (Cache-Control: private, no-store) and avoid leaking sensitive data through logs.
- Existing static star preview has been replaced by a public access notice, but old deployments, caches, git history, and copies may retain prior content. Audit these separately.

## Publication workflow
draft -> source verified -> owner/admin approved -> private or public classification -> server-side publication -> deployment -> authenticated read-back -> audit entry.
Approval must not automatically imply public visibility. Preserve versioned historical reports; latest pointer updates only after successful publication and read-back.
No automatic public publication until tests pass.

## Tests before enabling
1. Anonymous: cannot list or fetch star, bryan, or other private reports.
2. star: can fetch all authorized profiles.
3. bryan: can fetch bryan and star, not other users.
4. New user: can fetch own only, not star or bryan.
5. Direct URL, manifest, API, cached URL, and stale sessions all enforce authorization.
6. Google and email-code login map to the same verified account where appropriate.
7. Revocation takes effect; audit log records access decisions without sensitive content.
8. GitHub Actions and Cloudflare deployment cannot accidentally publish private content.
