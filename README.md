# Production Readiness Checklist for AI-Built Shopify Apps

Building an app with AI is fast. Running it in production requires evidence that it protects merchant data, handles failures, and keeps costs under control.

This repository provides a practical pre-launch checklist for Shopify apps built with Cursor, coding agents, and other AI-assisted development tools.

**[Open the full checklist](CHECKLIST.md)**

## What it covers

- Shopify authentication, authorization, and tenant isolation.
- Webhook verification, duplicate events, and idempotency.
- Payments, subscriptions, credits, refunds, and business logic.
- Database safety, queues, retries, and recovery.
- Input validation, secrets, dependencies, and infrastructure.
- Shopify API limits, infrastructure spending, and LLM cost controls.
- Monitoring, privacy, adversarial testing, and the final launch decision.

## How to use it

1. Copy [CHECKLIST.md](CHECKLIST.md) into your project or a tracking issue.
2. Review the code and relevant production settings.
3. Record **PASS**, **FAIL**, **NOT VERIFIED**, or **N/A** for each item.
4. Attach evidence: a file/function, configuration, test result, log, query, or provider setting.
5. Fix failures, rerun the relevant checks, and complete the production launch gate.

Use checked boxes for verified passes. Record other statuses beside the item. Explain why an item is not applicable. A checked box alone is not evidence.

For example, a tenant ownership check can be marked PASS with a reference to the tenant-scoped query and a passing Shop A / Shop B isolation test. Production backup settings remain NOT VERIFIED until the provider configuration has been inspected.

## Launch criteria

Do not launch with unresolved applicable CRITICAL items marked FAIL or NOT VERIFIED. Document any N/A decision and its justification. Investigate every unverified item that could cause cross-tenant access, unauthorized payments or refunds, secret exposure, data loss, or unbounded spending.

The checklist includes a final launch gate. Monitoring, spending alerts, and a rollback or kill-switch path should be ready before real merchants use the app.

## Shopify authentication note

Shopify's official libraries and framework integrations can handle authentication and webhook verification. You do not need to write custom HMAC code when the supported integration already performs the check. Verify that it actually runs before business logic and cannot be bypassed.

A valid Shopify session or webhook signature does not establish ownership of every requested database record. Your application must enforce tenant isolation and authorization separately.

## AI-assisted review

AI can help enumerate endpoints, inspect queries, and identify missing checks. Treat its conclusions as findings to verify. Require evidence, and manually review high-impact business rules and production settings that cannot be confirmed from the repository.

## Scope and limitations

This is a production readiness aid, not a penetration test, security certification, or guarantee that an application is vulnerability-free. Adapt it to your app's architecture and enabled features. It does not replace checking the current Shopify requirements that apply to your app.

## Contributions

Suggestions and corrections are welcome. Describe the failure a proposed check catches, when it applies, and how a developer can verify it.
