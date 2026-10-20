# Finish your Base44 migration

## What moved

The exporter copied `Flashcard`, `Note`, `Review`, `User` (1 documents) from Base44 app `6acf5c779ecb5c10a9ca0115` into MongoDB. Base44 IDs and document field values were retained, so relationships between documents stay intact. Each Base44 id is stored as the document's string `_id` (not an ObjectId), and built-in fields like `created_date`, `updated_date`, and `created_by_id` are kept as-is.

## What did not move automatically

- **Base44 Auth:** the `User` documents carry profile data, but passwords, active sessions, and OAuth links did not move — they are not sign-in identities in a new provider. Set up Supabase Auth (or another provider), create identities through its supported flow, map each new auth user to its Base44 user id, and have users reauthenticate.
- **Backend functions:** port each function to a Deno-compatible runtime (e.g. Supabase Edge Functions) — a rewrite, not a copy: replace Base44 SDK calls with a server-side MongoDB repository or API, and re-check limits, timeouts, and networking. Never connect to MongoDB from the browser.
- **Automations and schedules:** recreate triggers, schedules, retry policies, and monitoring in your new stack.
- **Integrations, connectors, and secrets:** reconnect every external service and create fresh secrets. Connector credentials were not exported.
- **RLS and security rules:** Base44 row- and field-level access rules did not move. Reimplement authorization in your API and database layer, default to deny, filter protected fields from responses, and test every user role and ownership boundary.
- **Uploaded file bytes:** file URI and URL fields remain in documents, but the files themselves did not move. Copy them to Supabase Storage or another object store and update references after validating the copy.
- **Realtime behavior:** Base44 subscriptions and events do not transfer automatically. Rethink which features need realtime updates and implement them against the new API.

## Cutover checklist

- [ ] Add MongoDB indexes based on entity schemas and real query patterns.
- [ ] Replace Base44 SDK entity calls with a server-side MongoDB repository or API. Never connect to MongoDB or expose the connection string from browser code.
- [ ] Choose and configure Supabase Auth or another new auth provider.
- [ ] Reconnect integrations and recreate secrets without copying old connector tokens.
- [ ] Port backend functions to a Deno-compatible runtime (e.g. Supabase Edge Functions) — rewriting Base44 SDK calls, not copying — and port automations or schedules.
- [ ] Reimplement Base44 RLS and security rules in the new API.
- [ ] Migrate uploaded file bytes to Supabase Storage and verify every rewritten reference.
- [ ] Update application environment variables and deployment configuration.
- [ ] Run smoke tests for sign-in, permissions, writes, integrations, files, and critical user journeys.
- [ ] Compare source and MongoDB documents — per-collection counts, plus spot-checks of ids, fields, types, and cross-document references — and investigate every mismatch.
- [ ] Switch production traffic only after backups and rollback steps are ready.
- [ ] Remove `dreamlit-export-helper` and rotate or delete `EXPORT_SECRET` after verification.
