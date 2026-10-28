# Relay — WhatsApp commerce prototype

React owner-facing PWA for catalogue-based sellers. Includes overview, inbox, human takeover, editable catalogue, order fulfilment updates, reports, assistant preferences, CSV export, responsive layout, and an install manifest.

## Run

Node.js 22+ recommended. The current environment uses Node 24.

```sh
cd /workspace/autoresponder
npm ci --cache /workspace/.npm-cache
npm run dev
```

Production validation: `npm run build`, then `npm run preview -- --port 4173`. HTTPS is required outside localhost for service workers. Offline mode supports previously fetched static resources; it does not queue messages or order mutations for server delivery. Full install criteria depend on browser and platform; add raster app icons before production release.

## Prototype boundaries

All displayed customers, messages, payments, metrics and events are samples. No WhatsApp messages, payment requests, authentication, billing, live analytics or AI calls occur. Products and assistant preferences persist in browser localStorage; conversation replies and order state reset on reload. This is not a multi-user database. Changing reporting period demonstrates chart layout only; summary cards remain fixtures. Export produces the sample order CSV, not a verified financial report. Avoid entering real customer data.

## Planned backend: Django + Supabase

Use Django REST Framework as the authenticated API and the only trusted authority for business actions. Supabase supplies managed PostgreSQL and optionally Auth and Storage. Keep all Twilio, AI, payment and Supabase privileged credentials on the server. Use migrations owned by Django rather than conflicting schema managers.

Suggested endpoints: business profile; products/variants; stock adjustments; conversations/messages; takeover/resume; orders; payment callbacks; report aggregates; Twilio inbound/status webhooks. Every business record needs tenant ownership, authorization and indexed tenant-scoped queries. Test cross-tenant access explicitly. If Supabase Auth is selected, Django must verify JWT signature, issuer, audience and expiry and derive tenant membership server-side. If browsers directly access Supabase later, add and test RLS; never expose the service-role key.

## What supports many customers

1. Stateless Django API instances behind a managed load balancer, with HTTPS, rate limits, health checks and horizontal scaling.
2. Celery workers plus managed Redis or RabbitMQ. Validate Twilio webhook signatures, durably persist/deduplicate inbound IDs, acknowledge promptly, and process AI replies asynchronously. Use per-conversation ordering and idempotent order/payment actions. Respect Twilio throughput limits and WhatsApp policy windows.
3. Supabase PostgreSQL connection pooling sized for the number of web and worker processes. Use the appropriate pooler mode/driver configuration, indexes on tenant/time/status, pagination, and atomic stock reservations to prevent overselling. Add backup/restore drills and migrations before expanding capacity.
4. Object storage for product photos with signed uploads, content/size checks and a CDN; do not put uploaded media on individual app servers.
5. Provider timeouts, bounded retries, backoff, dead-letter handling, per-business AI budgets and quotas. Pause automation reliably during human takeover, including replies already queued.
6. Centralized structured logs with customer data redacted, error reporting, queue-depth and latency alerts, provider delivery metrics, and per-tenant usage metering.
7. Separate staging and production, CI checks, secret management, verified payment webhooks, audit events, access controls and retention/deletion policies.

Start with a managed Django service, one worker, managed queue and Supabase. Kubernetes and microservices are unnecessary for the pilot. Capacity depends on peak messages per second, concurrent conversations, AI response time, database size and provider quotas—not simply account count. Set targets and load-test realistic traffic before making a capacity promise.

## Next integration milestone

Build authenticated tenant-scoped Django APIs and migrations, replace frontend fixtures, then connect one Twilio Sandbox sender. Verify real inbound/outbound messages, signature rejection, duplicate webhook handling, handoff race conditions and stock correctness. No external accounts have been provisioned by this prototype.
