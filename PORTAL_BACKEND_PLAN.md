# UBF participant portal — implementation notes

## Current status
`portal.html` is the mobile-first portal interface and API client prototype. It is **not an active authentication service yet**. It deliberately does not save passwords locally or simulate successful sign-in. Account creation, sign-in, dashboard data and booking require the Cloudflare API to be deployed.

## Recommended Cloudflare setup
- Cloudflare Workers for same-origin API routes.
- Cloudflare D1 for participants, programme access codes, sessions, milestones and audit events.
- Cloudflare Turnstile and rate limiting on public authentication endpoints.
- Use Workers' Web Crypto / a vetted password hashing strategy; never store plaintext passwords.
- Store only a hash of programme access codes, set expiry/active status, and limit one-time or cohort-based redemption.
- Keep admin/coordinator permissions separate from participant permissions. Participants must only see their own profile, bookings and progress.
- Do not store session tokens in localStorage. This prototype uses sessionStorage only to keep the demo simple; production should prefer secure, HttpOnly, SameSite cookies set by the Worker.
- Add email verification and password reset before onboarding real participants.

## Suggested D1 tables
- `participants`: id, name, email (unique), password_hash, status, created_at, verified_at.
- `programme_access_codes`: id, code_hash, programme_key, expires_at, max_uses, uses, active.
- `enrolments`: id, participant_id, programme_key, status, joined_at.
- `sessions`: id, participant_id, preferred_date, preferred_time, note, status, confirmed_at, coordinator_note, created_at.
- `milestones`: id, programme_key, title, sort_order.
- `participant_milestones`: id, participant_id, milestone_id, status, updated_by, updated_at, note.
- `documents`: id, participant_id or programme_key, title, object_key, visibility, uploaded_by, created_at.
- `audit_events`: id, actor_id, action, target_type, target_id, created_at.

## API contract expected by portal.html
- `POST /api/auth/register`: `{ accessCode, email, password }`; verify access code, create participant and enrolment; return session information.
- `POST /api/auth/login`: `{ accessCode, email, password }`; authenticate and return session information.
- `GET /api/me`: authenticated participant profile, progress, next session and tasks.
- `POST /api/sessions`: authenticated participant submits a preferred date/time; creates a *pending* request, not an automatic confirmed booking.
- Add coordinator-only routes for confirming/rescheduling sessions, updating milestones and managing access codes.

## Suggested simple workflow
1. UBF creates a cohort access code and shares it privately with selected participants.
2. Participant enters code, email and password and verifies their email.
3. Participant books an intake interview by requesting a date/time.
4. Coordinator confirms the session and records intake notes.
5. Participant sees agreed milestones and their statuses on the dashboard.
6. Coordinator updates progress; participant sees only their own information.

## Production readiness
Before collecting real participant data, connect the Worker, D1 database, email delivery, abuse protection, access controls, backup/retention policy and POPIA notices. Review the privacy notice to identify the actual hosting and service providers. The portal must not be treated as live until the backend endpoints and security checks are deployed and tested.
