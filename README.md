# FILE 2K REPORT

A community reporting and evidence directory for NBA 2K, built to match the approved black, crimson, and white designs. The project includes a working public reporting flow, protected moderation desk, and staged source-list importer.

**Repository:** [Terryjones1995/2kReporting](https://github.com/Terryjones1995/2kReporting)

## What is implemented

- Responsive landing page with the approved basketball/file artwork, custom badge, local Barlow fonts, player search, directory preview, and reporting process.
- Searchable directory with platform, source, and review-state filters; sorting; URL-preserved filter state; pagination; responsive player cards.
- Case pages with evidence previews, source provenance, progress, separate 2K submission/outcome fields, evidence additions, and correction requests.
- A validated report form with HTTPS links or file uploads, drag-and-drop, progress, private follow-up email, and a durable submission receipt.
- Protected `/admin` workspace for reviewing, editing, publishing, and withdrawing records, managing evidence/corrections, and recording manually filed 2K tickets.
- CSV/JSON source imports with required provenance, preview, duplicate detection, private defaults, and resumable processing.
- D1 persistence and R2 evidence storage. Submissions are not stored in browser localStorage.

The approved UI references and asset provenance are in [`docs/design`](docs/design). The sample court illustration is fictional and is labelled accordingly. No sample record is inserted into the database.

## Current data state

The expected approximately 37,000-record export was **not present in the checked 2KLabs repository branches on 3 November 2026**. No real list has been imported and no real player count is claimed.

Until there are published records, the public UI displays clearly labelled `DEMO_PLAYER_01`–`DEMO_PLAYER_08` examples. This happens only after a successful API response confirms that the unfiltered published-record count is zero. An API failure displays an error. Once a real record is published, search results come exclusively from the database.

Imported rows enter private review as **Not reviewed**. Source, review status, publication, submission to 2K, and 2K outcome are independent fields. The website records reports and external ticket activity; it does not automatically send tickets to 2K.

## Technology

| Layer | Implementation |
| --- | --- |
| UI | React 19, TypeScript, App Router components, custom responsive CSS |
| Runtime | Vinext / Vite on a Cloudflare Worker |
| Database | Cloudflare D1 with Drizzle schema and tracked migrations |
| Evidence and import staging | Cloudflare R2, served through authorization checks |
| Moderator identity | Sites ChatGPT authentication plus a server-side allowlist |
| Hosting | Sites, with owner-private access by default |

## Local development

Requires Node.js **22.13 or newer** and pnpm **11.25.0**. Use the committed lockfile. In a standard development checkout:

```bash
pnpm install --frozen-lockfile
pnpm build
```

Before the first local database use, apply the initial migration once:

```bash
node --import ./scripts/sites-env.mjs ./node_modules/wrangler/bin/wrangler.js d1 execute DB --local --config dist/server/wrangler.json --persist-to .wrangler/state --file drizzle/0000_clean_sleeper.sql
```

Then start local development:

```bash
pnpm dev
```

Use the URL printed by the server. Local development and hosted data are separate. Do not replay a migration against a database where it has already been applied. The managed Sites environment uses its `sites-preview` command for browser previews instead of starting another server.

The framework's portable environment offers loopback-only development sign-in; managed previews and production use the real Sites identity integration. Moderation remains denied until an authenticated identity matches `MODERATOR_EMAILS` or `MODERATOR_USER_IDS`. Set these through runtime secrets, never in committed source. See [`docs/OPERATIONS.md`](docs/OPERATIONS.md).

## Quality checks

```bash
pnpm test
pnpm typecheck
pnpm build
```

The 17-test domain and API suite exercises actual handlers against isolated in-memory D1/R2 adapters, including private publication defaults, file authorization, contact-data privacy, Origin and honeypot checks, request limits, external ticket validation, concurrent evidence decisions, duplicate handling, and resumable imports. A synthetic 37,000-row fixture validates normalization; it is not a real source list.

Browser verification includes desktop and narrow-screen layouts, mobile menu focus/scroll behavior, search and combined filters, zero-result states, case dialogs, required evidence validation, and a fictional local report submission with a private receipt.

## Source-list preparation

Validate an export without uploading it:

```bash
pnpm import:preview source.csv --manifest source-manifest.json
```

The manifest must contain a source label and public HTTPS source URL; an export version or source commit is recommended. Use the authenticated admin import preview for existing-record checks, confirmation, and actual import. Full input formats, limits, and review procedures are documented in [`docs/OPERATIONS.md`](docs/OPERATIONS.md).

## Publishing

The `.openai/hosting.json` file binds this checkout to its existing Site and declares the `DB` and `BUCKET` bindings. Keep the project ID intact when updating this Site. The Sites workflow pushes the exact source commit, packages its Worker build and assets, applies pending hosted migrations, and deploys the saved version.

The Site is initially owner-private. Audience changes are separate from code updates. Keep runtime secrets, `.wrangler/`, `.sites-runtime/`, dependencies, and generated build output out of Git. GitHub stores the complete source; publishing also saves that exact commit to the Site's source repository.

Additional framework/runtime guidance is retained in [`docs/FRAMEWORK.md`](docs/FRAMEWORK.md).

## Main files

| Area | Location |
| --- | --- |
| Shared brand, navigation, footer, X profile URL | `components/site-chrome.tsx` |
| Home and directory | `components/home-page.tsx`, `components/directory-page.tsx` |
| Reporting and case pages | `components/report-form.tsx`, `components/case-detail.tsx` |
| Moderation workspace | `components/admin-client.tsx` |
| Shared design and page-specific styles | `app/globals.css`, `components/report-pages.module.css` |
| API routes | `app/api/` |
| Validation, privacy, import and storage logic | `lib/report-*.ts` |
| Database schema and migrations | `db/schema.ts`, `drizzle/` |
| Approved design references and asset licenses | `docs/design/`, `public/fonts/` |

The X link currently uses the proposed `@File2KReport` handle. Update the two profile links in `components/site-chrome.tsx` if the final account handle differs.
