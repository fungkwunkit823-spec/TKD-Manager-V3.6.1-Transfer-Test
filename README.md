# TKD Manager V3.6.1 — Full-schema integration candidate

**Status: code assembled and JS syntax-checked, NOT verified in a live full-schema database. NOT ready for production deployment.**

## Isolation

- Production V3.6 and existing 30 real students / 6 classes **must remain untouched**.
- Existing V3.6.1 **8-table** mini-test project also remains unchanged; **do not run the 17-table SQL there**.
- Create a *new* independent Supabase project with a **separate** Auth test admin. Admin UID in these files is `2dabf304-fb41-469f-9599-a6c965192728`: if that UID belongs only to the previous 8-table project, it WILL NOT exist in a fresh project. Supabase Auth UIDs are project-specific: **you must replace every occurrence with the new full-test admin UID, including in the schema, guard, audit and config, before use**. Otherwise preflight stops. This is deliberately fail-closed.
- `00_FRESH_17_TABLE_TEST_SCHEMA.sql` comes from a November 2026 saved schema snapshot, not the *live* current structure. Functions and grants must be reviewed against any subsequent V3.6 changes, including the later refund fix. It contains no student data.

## Setup (iPad Safari)

1. Create a new full-schema Supabase test project, not either existing project. Create its new Auth test admin.
2. Replace the known test UID in `00`, `01`, `03` and `config.js` with the **new** Auth UID, then validate the 17-table schema has not drifted. A clean project is required.
3. Run `00_FRESH_17_TABLE_TEST_SCHEMA.sql` in the new project's SQL Editor. Stop on any error.
4. Run `01_FULL_TRANSFER_GUARD.sql` in same test project. Stop on any error.
5. Run `02_FULL_SYNTHETIC_DATA.sql` there. Only fake students; do not restore `data.sql` from production backup.
6. Run `03_FULL_READONLY_AUDIT.sql` and inspect checks.
7. Create new separate GitHub Pages repo to host `index.html` and `config.js` (create config from example). The `index.html` has a guard that restricts the admin UID to the earlier known test UID, so if using a newly created Auth user you must also update the exact UID in `index.html` before serving. It explicitly refuses the old 8-table URL and known production URLs; don't remove these protections.
8. Test student A: safe preview + transfer; B: invoice -> denied; C: target enrollment -> denied; D: attendance -> denied. Inspect audit trail, ensure only A moved. Confirm no real student data in the new project.

## Safety caveats

- This version is a **conservative error-correction** workflow, not a formal student transfer with historical records. It blocks student-wide tuition payments, invoices, discounts, attendance, credits, makeup and multi-class scenarios.
- Preview and commit are separate transactions. Commit re-runs all checks, but external insert paths require an additional concurrency/security review before use on production.
- `students.class_name` is a legacy display field; the implementation rejects multi-enrollment students instead of silently overwriting that field.
- Financial calculations and other application modules have not been end-to-end revalidated. Do not deploy this code or SQL to production yet.
- The SQL `01` creates an **additional** audit table alongside the 17 V3.6 tables. The installation result is 18 total public tables, not 17.
- Original schema sample may include default privileges, functions or old test/production references. Review carefully before applying; don't assume copying a backup's schema safely reproduces a modern live instance.
