# EduAxis Tutor Centre — Independent Website & Parent Portal

**Version:** 1.0.0  
**Prepared for:** EduAxis Tutor Centre, Villieria, Pretoria  
**Business contact:** Vernique Santos · 060 521 3315 · eduaxis.tutor@gmail.com

This is an independent, self-hostable website/PWA and Node.js application. It does not require Base44 at runtime and contains no Base44 SDK, API calls, project identifiers or hosted Base44 dependencies.

## 1. What is included

- Responsive marketing website with Home, About, Learning Approach, Grades & Curriculum, Services, Meet Tutor, Parent Information, Enrolment, Consultation Booking, Resources, News, Gallery, Testimonials, Parent Portal and Contact sections.
- Local search metadata, `robots.txt`, sitemap starter and mobile-friendly layout.
- WhatsApp, telephone, email and Google Maps links using the contact details supplied in the project brief.
- Working public enquiry, initial application and consultation-request forms.
- SQLite-backed records for enquiries, applications and consultation requests.
- Parent account registration and password login using Node.js `scrypt` password hashing.
- Server-side session records, random session tokens, HttpOnly/SameSite cookies, CSRF checks on authenticated writes, basic per-IP rate limits and security response headers.
- Parent portal with family-scoped learner records. A parent can only retrieve learners whose `parent_user_id` is their own account ID.
- Admin functions to review submitted forms, link learners to parent accounts, add homework/progress/timetable records, and publish centre notices/news.
- PWA manifest and service worker for caching the public shell.
- Docker deployment files and a repeatable integration test.

## 2. Requirements

- Node.js **22.13 or later** (Node 22 is required for the built-in `node:sqlite` module used by this package).
- Windows, Linux or macOS.
- No npm packages are required. The application uses Node.js built-in modules only.

> Node's built-in SQLite module is marked experimental in Node 22. For a high-volume production deployment, plan a migration to a managed PostgreSQL service and have the migration reviewed before go-live.

## 3. Run locally on Windows

1. Extract the ZIP to a normal folder, for example `C:\EduAxis`.
2. Install Node.js 22 LTS or later from the official Node.js website.
3. Open the extracted folder in File Explorer.
4. Copy `.env.example` to `.env`.
5. Edit `.env` in Notepad and set:
   - `SESSION_SECRET` to a unique random secret of at least 40 characters.
   - `ADMIN_EMAIL` to the centre's chosen administrator email.
   - `ADMIN_PASSWORD` to a unique password of at least 14 characters.
   - `SITE_URL` to the final HTTPS domain when known.
6. Open Command Prompt in the project folder and run:

   ```bat
   npm run create-admin
   npm start
   ```

7. Browse to `http://localhost:8080`.
8. Sign into the Parent Portal using the administrator email/password from `.env`.

The first run creates `data\eduaxis.sqlite`. Keep the `data` folder backed up. Do not delete it when updating the website.

## 4. Create a parent account and connect a learner

1. A parent uses **Create account** in the Parent Portal.
2. The parent account is created, but no learner information is exposed yet.
3. The administrator signs into the portal.
4. Under **Link a learner to a parent account**, enter the parent's exact account email, learner name and grade.
5. The administrator can then add homework, progress updates and timetable entries for that learner.
6. The parent signs in and sees only their own linked learner(s).

If the parent has not created an account yet, the learner-link operation will reject the request. This is intentional.

## 5. Production deployment

### Option A — Docker

1. Install Docker Engine / Docker Desktop on the server.
2. Copy `.env.example` to `.env` and set a long random `SESSION_SECRET`.
3. Build and start:

   ```bash
   docker compose up -d --build
   ```

4. Put a trusted HTTPS reverse proxy (for example, Caddy or Nginx) in front of port 8080.
5. Ensure the `data` volume is persistent and backed up. Keep port 8080 private behind the reverse proxy where possible.
6. Run the administrator bootstrap inside the container after the first deployment:

   ```bash
   docker compose exec eduaxis node scripts/create-admin.js
   ```

   Set `ADMIN_EMAIL` and `ADMIN_PASSWORD` in the container environment before running this command, or run the script on the host with the same `DATABASE_PATH` pointing to the persistent database.

### Option B — Node hosting

Upload the project folder to a Node.js 22+ host, set the environment variables in the hosting control panel, set the persistent database path, and use `node server.js` as the start command. Configure HTTPS at the host or reverse proxy. Do not expose the SQLite database or `.env` file publicly.

### Before opening the site to the public

- Replace the placeholder brand mark and visual illustrations with the client's original logo and approved centre photographs.
- Replace `REPLACE-WITH-YOUR-DOMAIN` in `public/robots.txt` and `public/sitemap.xml`.
- Set `SITE_URL` and connect the chosen domain with HTTPS.
- Change the bootstrap administrator password after initial setup if it was temporarily shared.
- Configure off-server encrypted backups for the SQLite database.
- Review the privacy notice, terms, retention schedule and consent wording with a South African privacy/legal professional against the centre's actual practices.
- Test the forms and parent portal using separate parent accounts.

## 6. Data storage

| Data | Location |
|---|---|
| Website files | `public/` |
| SQLite database | `data/eduaxis.sqlite` (created at runtime) |
| Future private uploads | `uploads/` |
| Environment secrets | `.env` (never publish or commit) |

The database includes records for parent users, sessions, learners, homework, progress, timetables, enquiries, initial applications, consultation requests, notices, resources, testimonials, gallery entries and news.

## 7. Security and privacy notes

- Passwords are stored as salted `scrypt` hashes; plaintext passwords are not stored.
- Session tokens are random and only their SHA-256 hashes are stored in the database.
- Session cookies are HttpOnly and SameSite=Lax; Secure is enabled in production mode.
- Authenticated writes require a per-session CSRF token.
- Parent portal queries scope learner records by the authenticated parent user ID.
- Public forms intentionally do not accept ID documents, medical records or file uploads.
- This starter does not yet implement email notifications, multi-factor authentication, a full audit log, file malware scanning, formal POPIA operator agreements, or a tested disaster-recovery process. Those are required work items before sensitive/high-volume production use.
- The basic rate limiter is in-memory and per application process. Use a shared rate-limiting service if you scale to multiple instances. If deployed behind a trusted reverse proxy that overwrites `X-Forwarded-For`, set `TRUST_PROXY=1`; otherwise leave it disabled to prevent spoofed client IP headers.

## 8. Payment integration

No payment gateway is activated in this package. South African online payments require the centre's merchant account, verified business details, credentials, webhook configuration, refund handling and reconciliation rules. Do not accept or store card details in this application. A PayFast (or other selected provider) integration should be a separately tested phase once merchant credentials and payment rules are supplied.

## 9. Tests

Run:

```bash
npm test
```

The integration test starts a temporary server and database and checks public submissions, parent registration/login, administrator access, learner linking, homework and timetable writes, family-level isolation, and rejection of parent access to admin endpoints and invalid CSRF logout attempts.

## 10. Important scope statement

This is a clean independent implementation based on the client brief and information available from the published EduAxis website. It is **not a verified pixel-for-pixel export of the Base44 application**: the Base44 source files, original logo and centre photo assets were not available for extraction in this session. The included brand mark and hero artwork are temporary design placeholders, not the client's original identity assets. The package deliberately does not invent tutor qualifications, fees, testimonials, social accounts, learner images or school-registration claims.
