# Production Readiness Checklist for AI-Built Shopify Apps

> A practical pre-launch checklist for Shopify apps built substantially with Cursor, coding agents, or other AI-assisted development tools.

This checklist is not a penetration test and does not prove an application is vulnerability-free. Its purpose is to catch common security, reliability, multi-tenant, cost, and business-logic failures before production.

## How to use it

For every item, record one of: **PASS**, **FAIL**, **NOT VERIFIED**, or **N/A**.

A `PASS` should have evidence: a file/function, configuration, automated test, manual test, log, query, or provider setting. Do not accept “looks correct” as evidence.

**Launch rule:** no unresolved **CRITICAL** item should remain `FAIL` or `NOT VERIFIED` before production.

### Important notes

- Authentication and authorization are separate. Authentication answers *who is making the request?* Authorization answers *may this identity perform this operation on this resource?*
- Shopify HMAC verification is **request authenticity**, not resource authorization. For standard Shopify OAuth/webhook flows, prefer Shopify's official libraries rather than implementing cryptography yourself. The check is that verification actually occurs and is not bypassed.
- A valid Shopify session does not prove that a requested database object belongs to that shop. Tenant ownership must still be enforced by your application.
- Browser input is untrusted, including hidden fields and values produced by your own frontend.
- AI output is untrusted input too.
- Provider/infrastructure settings that cannot be verified from the repository should be marked `NOT VERIFIED`, not guessed as `PASS`.

---

## 1. Authentication & Shopify Identity - CRITICAL
- [ ] Authentication is performed server-side using Shopify-supported mechanisms.
- [ ] Official Shopify libraries are used for standard authentication/signature verification where practical; custom cryptographic implementations have a documented reason and tests.
- [ ] The application never trusts `shop`, `userId`, `customerId`, or similar identity values solely because they came from query parameters, form data, headers, or JSON input.
- [ ] The authenticated Shopify identity is the source of truth for the current shop.
- [ ] Session/access tokens are validated correctly before protected operations.
- [ ] OAuth state/nonce validation is implemented where applicable.
- [ ] OAuth callback parameters are validated.
- [ ] Redirect URLs cannot be manipulated into arbitrary redirects.
- [ ] Offline and online access tokens are stored only server-side.
- [ ] Tokens are never exposed in browser JavaScript, HTML, URLs, analytics, or logs.
- [ ] App uninstall/reinstall behavior has been tested.
- [ ] Expired, revoked, malformed, and missing tokens fail closed.
- [ ] Development authentication shortcuts cannot run in production.

## 2. Multi-Tenant Isolation / BOLA / IDOR - CRITICAL
- [ ] Every tenant-owned database record is associated with a shop/tenant identifier where appropriate.
- [ ] Every read operation verifies that the requested resource belongs to the authenticated shop.
- [ ] Every update operation verifies ownership.
- [ ] Every delete operation verifies ownership.
- [ ] Resource IDs from the client are never sufficient authorization by themselves.
- [ ] Database queries are scoped by tenant wherever practical, e.g. `WHERE id = ? AND shopId = ?`.
- [ ] Exports, downloads, files, reports, and generated URLs enforce tenant ownership.
- [ ] Background jobs carry a trusted tenant identity and cannot cross tenant boundaries.
- [ ] Admin/debug endpoints follow the same tenant isolation rules.
- [ ] Tests exist where Shop A creates a resource and Shop B attempts GET, UPDATE, and DELETE against it.
- [ ] Sequential, guessed, leaked, or copied resource IDs do not expose another shop's data.

## 3. Authorization & Privileged Operations - CRITICAL
- [ ] Authentication and authorization are treated as separate checks.
- [ ] Every privileged endpoint explicitly checks permission/role/scope where required.
- [ ] Admin-only actions cannot be triggered by ordinary users.
- [ ] Required Shopify scopes are minimal and documented.
- [ ] The app does not request unnecessary Shopify scopes.
- [ ] Server-side authorization cannot be bypassed by hiding UI controls only.
- [ ] Mass assignment cannot set protected fields such as `role`, `shopId`, `status`, `price`, `credit`, or permissions.
- [ ] Internal endpoints are not accidentally publicly reachable.

## 4. Shopify Webhooks - CRITICAL
- [ ] Shopify webhook authenticity is verified using Shopify's required HMAC/signature mechanism, preferably through the official Shopify library/framework integration.
- [ ] No webhook route bypasses signature verification before business logic executes.
- [ ] Verification uses the exact request representation required by the Shopify library/protocol; body parsing does not invalidate or bypass verification.
- [ ] Invalid signatures are rejected before business logic runs.
- [ ] Webhook handlers are idempotent.
- [ ] Duplicate delivery of the same event cannot duplicate orders, credits, refunds, jobs, emails, or state changes.
- [ ] Out-of-order webhook delivery is considered.
- [ ] Slow work is moved to a queue/background worker where appropriate.
- [ ] Webhook endpoints acknowledge valid requests quickly.
- [ ] Retry behavior is understood and tested.
- [ ] App uninstall handling removes/revokes data and credentials according to the application's policy.
- [ ] Required Shopify privacy/compliance webhooks are configured where applicable.

## 5. Payments, Billing, Credits & Refunds - CRITICAL
- [ ] Prices and payable amounts are calculated or verified server-side.
- [ ] The backend never trusts an amount supplied by the browser.
- [ ] Plan/subscription identity is verified server-side.
- [ ] A redirect or client flag such as `success=true` is never treated as proof of payment.
- [ ] Payment status is verified with the authoritative payment system.
- [ ] Payment webhook signatures are verified using the provider's supported mechanism/library.
- [ ] Payment and refund handlers are idempotent.
- [ ] Duplicate payment callbacks cannot create duplicate value.
- [ ] Duplicate refund requests cannot issue multiple refunds.
- [ ] Refund ownership and transaction ownership are checked.
- [ ] Users cannot modify product, price, quantity, discount, currency, credit, or refund amount to obtain unintended value.
- [ ] Subscription upgrade/downgrade/cancellation edge cases are tested.
- [ ] Concurrent requests cannot produce double credit, double refund, or inconsistent balances.
- [ ] Monetary state changes use database transactions/locking where necessary.
- [ ] Trial/reinstall/cancel flows cannot be abused for unlimited free access.

## 6. Business Logic Abuse - CRITICAL

For every valuable workflow ask: **How can a user make money, receive value, avoid payment, exceed a limit, or affect another tenant by using valid operations in an unexpected order?**

- [ ] Critical workflows have explicit invariants documented.
- [ ] Operations have been tested in an unexpected order.
- [ ] Repeated operations have been tested.
- [ ] Concurrent operations have been tested.
- [ ] Zero, negative, extremely large, missing, duplicated, and stale values have been tested where relevant.
- [ ] Coupon/discount/credit usage cannot exceed intended limits.
- [ ] Cancellation after receiving value is handled correctly.
- [ ] Retry/replay cannot generate additional value.
- [ ] Race conditions around counters, capacity, inventory, bookings, group size, credits, and payments are considered.
- [ ] Account deletion/reinstall does not reset restrictions that should persist.

## 7. Database Safety & Cost
- [ ] All tenant-owned queries are scoped correctly.
- [ ] Required foreign keys and unique constraints exist.
- [ ] Idempotency is reinforced by database constraints where practical.
- [ ] Transactions protect related state changes.
- [ ] Main query paths have appropriate indexes.
- [ ] N+1 query patterns have been reviewed.
- [ ] Every potentially large collection is paginated or bounded.
- [ ] Client-controlled `limit`, offset, search range, or date range has a server-side maximum.
- [ ] Connection pool size is explicitly configured for the deployment environment.
- [ ] Expensive queries have been identified and measured.
- [ ] Query/database timeouts are appropriate.
- [ ] Unbounded table/storage growth has been considered.
- [ ] Cleanup/retention exists for logs, jobs, temporary data, sessions, and other high-growth tables where needed.
- [ ] Production backup is enabled.
- [ ] Restore from backup has been tested or there is a documented restore procedure.

## 8. Resource & Cost Abuse - CRITICAL

For every public or merchant-triggered endpoint ask: **What happens if it is called 10, 1,000, or 1,000,000 times?**

- [ ] Rate limiting exists for abuse-sensitive endpoints.
- [ ] Limits are considered per IP, user, shop/tenant, and global application capacity as appropriate.
- [ ] Request body size has a maximum.
- [ ] Upload size has a maximum.
- [ ] Pagination size has a maximum.
- [ ] Search/date-range complexity is bounded.
- [ ] Concurrency is bounded for expensive operations.
- [ ] Background queue growth is bounded/monitored.
- [ ] Retry counts are bounded.
- [ ] Retries use exponential backoff and jitter where appropriate.
- [ ] External-service failures cannot create infinite retry loops.
- [ ] Timeouts exist for external calls.
- [ ] Circuit-breaking/degradation behavior is considered for critical external dependencies.
- [ ] Spending/usage alerts exist for paid infrastructure and third-party APIs.
- [ ] One malicious or buggy tenant cannot consume unlimited shared resources.

## 9. LLM / AI Cost & Security - CRITICAL when AI is used
- [ ] LLM API keys exist only server-side.
- [ ] The browser cannot call the provider using the application's secret key.
- [ ] Input size is bounded before an LLM request is made.
- [ ] Maximum output tokens are configured.
- [ ] Requests per user/shop/time period are bounded where appropriate.
- [ ] LLM usage and estimated/actual cost can be attributed per tenant or feature.
- [ ] Expensive models are used only where their capability is required.
- [ ] Large context objects are reduced to necessary fields.
- [ ] Reusable/static context is cached where appropriate.
- [ ] Failed LLM calls cannot create uncontrolled retry spending.
- [ ] Agent/tool loops have maximum iterations.
- [ ] Model output is treated as untrusted input.
- [ ] Model output is validated against an explicit schema when structured output is expected.
- [ ] LLM output cannot directly become SQL, shell commands, `eval`, privileged API calls, or unsanitized HTML.
- [ ] The LLM is not the authority for authentication or authorization decisions.
- [ ] Prompt injection cannot reveal application secrets or tenant data.
- [ ] Tool-enabled AI receives least privilege and explicit allowed operations.
- [ ] AI-triggered expensive or destructive actions require deterministic server-side checks.
- [ ] Per-feature cost kill switches/feature flags exist for unusually expensive AI functionality.

## 10. Shopify Admin API & GraphQL Cost
- [ ] GraphQL requests ask only for fields actually required.
- [ ] Pagination is implemented correctly.
- [ ] Large product/order/catalog operations use appropriate bulk/background approaches.
- [ ] Shopify throttling is handled explicitly.
- [ ] Retry behavior respects rate limits and does not amplify load.
- [ ] API retries are bounded.
- [ ] API responses are cached when safe and useful.
- [ ] Reconciliation exists for important synchronization flows.
- [ ] A failed partial synchronization can resume safely.
- [ ] API calls are idempotent where the business operation requires it.
- [ ] Large synchronization jobs have observable progress/state.

## 11. Input Validation & Injection
- [ ] Every externally supplied value is validated server-side.
- [ ] Validation defines type, allowed range, length, format, and enum where appropriate.
- [ ] SQL injection paths have been reviewed even when using an ORM.
- [ ] XSS paths have been reviewed.
- [ ] Command injection paths have been reviewed.
- [ ] Path traversal has been reviewed.
- [ ] SSRF has been reviewed for any feature accepting URLs or making server-side HTTP requests.
- [ ] Open redirects have been reviewed.
- [ ] Uploaded files are validated by type, size, and intended use.
- [ ] User-controlled filenames and paths are not trusted.
- [ ] Output is escaped for its destination context.
- [ ] GraphQL/REST inputs are validated before business logic.
- [ ] Prototype pollution / unsafe object merging is considered in JavaScript code handling arbitrary objects.

## 12. Secrets, Dependencies & Supply Chain - CRITICAL
- [ ] `.env` and production secrets are excluded from Git.
- [ ] Repository history has been checked for accidentally committed secrets.
- [ ] Shopify client secret is never exposed client-side.
- [ ] Database credentials are never exposed client-side.
- [ ] Payment provider secrets are never exposed client-side.
- [ ] LLM/API secrets are never exposed client-side.
- [ ] Development and production credentials are separated.
- [ ] Secrets can be rotated without code changes.
- [ ] Logs redact tokens, passwords, cookies, authorization headers, payment secrets, and sensitive personal data.
- [ ] Dependencies are scanned for known vulnerabilities.
- [ ] AI-generated package names have been manually verified as legitimate packages.
- [ ] Newly added dependencies have been checked for suspicious ownership/name/installation scripts.
- [ ] Lockfiles are committed and deterministic builds are used where practical.
- [ ] Unused dependencies are removed.

## 13. Browser & Web Security
- [ ] Production uses HTTPS only.
- [ ] Cookies use appropriate `Secure`, `HttpOnly`, and `SameSite` settings.
- [ ] CORS is restrictive and intentional.
- [ ] CSRF protection exists where cookie-authenticated state-changing requests require it.
- [ ] Security headers/CSP have been reviewed.
- [ ] Sensitive data does not appear in URLs.
- [ ] Sensitive responses are not incorrectly cached by shared/public caches.
- [ ] Frontend source maps/debug information do not expose sensitive implementation details unnecessarily.

## 14. Infrastructure & Production Configuration
- [ ] Production configuration is separate from local/development configuration.
- [ ] Debug mode is disabled in production.
- [ ] Stack traces and internal error details are not returned to users.
- [ ] Database/network access is restricted to what is required.
- [ ] Administrative dashboards are protected.
- [ ] Development/test endpoints are removed or protected.
- [ ] Cron/scheduled-job endpoints cannot be triggered freely by the public unless explicitly designed and protected.
- [ ] Production environment variables have been reviewed individually.
- [ ] CPU/memory/storage limits and scaling behavior are understood.
- [ ] Deployment rollback procedure exists.
- [ ] Database migration rollback/recovery strategy exists for risky migrations.

## 15. Queues, Workers & Scheduled Jobs
- [ ] Jobs are idempotent.
- [ ] Duplicate jobs are safe.
- [ ] Jobs have bounded retries.
- [ ] Permanent and transient failures are distinguished.
- [ ] Dead-letter/failed-job handling exists where appropriate.
- [ ] Job payloads contain trusted tenant identity.
- [ ] A job cannot operate on another tenant by changing an untrusted ID.
- [ ] Worker concurrency is explicitly controlled.
- [ ] Queue backlog is observable.
- [ ] Scheduled jobs cannot overlap dangerously.
- [ ] Reconciliation can recover missed/failed events for critical synchronization.

## 16. Logging, Monitoring & Financial Kill Switches
- [ ] Centralized production error monitoring exists.
- [ ] Authentication failures are observable.
- [ ] Authorization failures are observable.
- [ ] Payment/refund events have an audit trail.
- [ ] Critical business-state changes have sufficient audit data.
- [ ] LLM usage/cost is observable.
- [ ] Shopify API failures/throttling are observable.
- [ ] Database/storage growth is observable.
- [ ] Queue backlog/failure rate is observable.
- [ ] Alerts exist for abnormal request volume.
- [ ] Alerts exist for abnormal infrastructure/API spending.
- [ ] Expensive features can be disabled quickly using configuration/feature flags.
- [ ] A single abusive tenant can be disabled/throttled without shutting down the application.
- [ ] Credentials can be rotated quickly during an incident.

## 17. Privacy & Data Lifecycle
- [ ] The app stores only data it actually needs.
- [ ] Sensitive/customer data fields are inventoried.
- [ ] Data retention periods are documented where relevant.
- [ ] Uninstall/deletion workflows are implemented according to Shopify and legal requirements applicable to the app.
- [ ] Logs do not retain unnecessary customer data.
- [ ] Backups and exports are included in the data lifecycle plan.
- [ ] Access to production customer data is limited.

## 18. Failure & Recovery Testing
- [ ] Shopify API unavailable for 20 minutes has been simulated or reasoned through and recovery behavior is documented.
- [ ] Database temporarily unavailable has been considered.
- [ ] Payment provider timeout has been considered.
- [ ] LLM provider timeout/rate limit has been considered where applicable.
- [ ] Worker crash halfway through a job is safe.
- [ ] Server restart during a critical operation is safe.
- [ ] Duplicate webhook after restart is safe.
- [ ] Partial synchronization can be reconciled.
- [ ] External API returning malformed/unexpected data does not corrupt state.
- [ ] Production can degrade without silently losing critical operations.

## 19. AI-Assisted Adversarial Review

Run a separate review after implementation. The reviewer should not trust previous AI-generated conclusions.

- [ ] Enumerate all public endpoints and classify authentication requirements.
- [ ] Enumerate all state-changing endpoints.
- [ ] Enumerate all places where tenant-owned objects are loaded by ID.
- [ ] Enumerate all payment/credit/refund paths.
- [ ] Enumerate all externally callable operations that create financial cost.
- [ ] Enumerate all LLM calls and estimate their maximum cost per request/workflow.
- [ ] Enumerate all loops, queues, retries, scheduled jobs, and recursion that can amplify cost.
- [ ] Search for hard-coded secrets and accidental credentials.
- [ ] Search for authorization based solely on client-provided IDs.
- [ ] Search for missing limits on pagination, uploads, text input, arrays, and date ranges.
- [ ] Search for `eval`, shell execution, dynamic SQL, raw queries, unsafe HTML, arbitrary URL fetching, and dangerous deserialization.
- [ ] Review every `TODO`, temporary bypass, debug flag, mock, and development shortcut.

## 20. Mandatory Adversarial Tests Before Launch
- [ ] Shop B attempts to read Shop A's resource.
- [ ] Shop B attempts to modify Shop A's resource.
- [ ] Shop B attempts to delete Shop A's resource.
- [ ] Invalid/expired authentication is used against every critical endpoint.
- [ ] A Shopify webhook is sent without a valid signature and is rejected.
- [ ] The same valid webhook is delivered multiple times.
- [ ] The same state-changing request is sent concurrently.
- [ ] Payment success is forged from the browser/client.
- [ ] Price/amount/discount fields are modified client-side.
- [ ] Refund/credit action is repeated concurrently.
- [ ] Maximum input sizes are exercised.
- [ ] Rate limits are exercised.
- [ ] External API failures and timeouts are simulated.
- [ ] Background worker is interrupted mid-operation.
- [ ] High-volume endpoints are load-tested at a safe scale.
- [ ] LLM prompt injection is tested if AI can access data/tools.
- [ ] An intentionally malicious tenant attempts to maximize infrastructure/API cost.

---

# Production Launch Gate

## CRITICAL: must be PASS
- [ ] Authentication
- [ ] Tenant isolation / BOLA / IDOR
- [ ] Authorization
- [ ] Webhook authenticity + idempotency
- [ ] Payment/billing/refund integrity
- [ ] Business-logic invariants
- [ ] Secret management
- [ ] Resource/cost limits
- [ ] LLM safety/cost controls, if applicable
- [ ] Backup/recovery for critical data
- [ ] Mandatory adversarial tests

## Launch decision
- [ ] All CRITICAL checks are `PASS`.
- [ ] All HIGH failures have been fixed or explicitly accepted with documented reasoning.
- [ ] No `NOT VERIFIED` item can plausibly cause cross-tenant access, unauthorized payment/refund, secret exposure, data loss, or unbounded financial cost.
- [ ] Monitoring and spending alerts are active before opening the app to real merchants.
- [ ] A rollback/kill-switch path is known.

---

## Scope note

This checklist is intentionally broader than a security checklist. A production failure can come from authorization bugs, duplicate events, race conditions, unbounded database/API/LLM usage, bad retry behavior, or business-logic abuse without looking like a classic vulnerability.

Use automated tooling and AI to accelerate the review, but require evidence and manually review high-impact business rules and infrastructure settings.
