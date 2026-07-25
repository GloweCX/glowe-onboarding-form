# Glowe Onboarding Form

Production client onboarding form for Glowe, deployed via GitHub Pages at
onboarding.helloglowe.com. This repository is independent from the Revenue
Assessment repository (glowe-retention-assessment / retention.helloglowe.com);
nothing here modifies that deployment.

## What this is

A single static page (index.html) that walks a client through 7 onboarding
sections, collects up to 10 services, builds a single canonical JSON payload,
and POSTs it directly to a Zapier Catch Hook webhook. Zapier then loops the
services array and updates GoHighLevel custom fields and triggers workflows.

No backend, build step, or server-side code is required; GitHub Pages serves
the file as-is.

## Repository structure

index.html: production onboarding form (HTML, CSS, and vanilla JS).
CNAME: custom domain file, must contain exactly onboarding.helloglowe.com.
sample-payload.json: example payload with fake data, safe to share, no secrets.
FIELD_MAPPING.md: full field to JSON path to GoHighLevel slug table.
README.md: this file.
.gitignore: local, OS, and editor files that should never be committed.

There is no docs folder and no GitHub Actions build workflow. GitHub Pages
is configured to deploy from the main branch, root directory. Every push to
main republishes the live site automatically; there is no manual rebuild step.


## Deployment (already done)

Repository created as GloweCX/glowe-onboarding-form, public, matching the
visibility of the Revenue Assessment repo. GitHub Pages enabled under
Settings, Pages, Source set to Deploy from a branch, Branch set to main and
root. Custom domain field set to onboarding.helloglowe.com (GitHub
auto-detected this from the committed CNAME file). Build confirmed
successful via the pages build and deployment Action.

## DNS (you still need to do this)

onboarding.helloglowe.com does not exist in DNS yet. Add exactly one record
in GoDaddy, mirroring the same pattern already used for
retention.helloglowe.com.

Record type: CNAME.
Host or name: onboarding.
Value or target: glowecx.github.io.
TTL: 1 hour, or your provider default.
Conflicting records to remove: none expected; only remove or edit this if an
old onboarding record already exists.

Do not touch any record for helloglowe.com, www, retention, or any mail
records; this new record is additive only.

After adding the record, DNS propagation is typically fast, within minutes,
but can take up to 24 to 48 hours depending on registrar and TTL caching.
Confirm propagation by checking GitHub Settings, Pages, where "DNS check
unsuccessful" should change to a green "DNS check successful," or by using
any public DNS lookup tool for onboarding.helloglowe.com (CNAME) from a
computer other than this one. Once the DNS check succeeds, return to
Settings, Pages, and enable Enforce HTTPS; it is greyed out until DNS
validates and cannot be turned on earlier.

## Configuration

All environment-specific values live in one place: the CONFIG object at the
top of the script block in index.html. It defines SCHEMA_VERSION, SOURCE,
ZAPIER_WEBHOOK_URL (the Zapier Catch Hook URL), UPDATE_LOOKUP_ENDPOINT
(currently null, see Update mode below), and SUBMIT_TIMEOUT_MS.

Important security note about the webhook URL: because this is a fully
static site with no backend, the browser must call the Zapier webhook
directly, which means the URL is visible to anyone who views the page source
or the Network tab. This is unavoidable with a pure static-hosting
architecture and is the same trade-off any static form-to-Zapier integration
has. It is a Catch Hook endpoint (accepts POSTs, cannot be used to read
existing data), so the exposure risk is limited to unwanted or spam
submissions, not data disclosure. If that risk becomes unacceptable, the
smallest fix is a lightweight serverless proxy (for example a Cloudflare
Worker) placed in front of the Zapier webhook; this is out of scope for
plain GitHub Pages and was not built here since it was not required to reach
the stated goal.

## Initial mode vs update mode

The form reads mode, cid, and token from the URL query string, for example
onboarding.helloglowe.com/?mode=update&cid=CONTACT_ID&token=TOKEN.

If mode is absent or not equal to update, the form runs in initial mode:
it starts blank, and submission_mode in the payload is "initial".

If mode equals update, submission_mode is "update", and the form attempts
to pre-populate from CONFIG.UPDATE_LOOKUP_ENDPOINT.

Status: update mode pre-population is not yet connected.
CONFIG.UPDATE_LOOKUP_ENDPOINT is null on purpose. Ezekiel has not yet
confirmed a GoHighLevel read method (API endpoint, required identifier, and
authentication approach) that is safe to call. Until that exists, a client
opening an update link sees a banner explaining that pre-population is not
connected yet, and can still fill out and submit the form manually; their
submission still flows through the same Zapier webhook with submission_mode
set to "update". The code path that will consume the read endpoint
(initUpdateMode and applyExistingData in index.html) is already written and
was tested against a hypothetical response shape; it is ready to wire up as
soon as the endpoint is confirmed. Do not point it at a guessed endpoint.

Do not implement this by calling the GoHighLevel API directly from the
browser. A private GoHighLevel API token must never live in client-side
JavaScript. Once Ezekiel confirms the read method, the safest options are,
in order of preference: first, a Zapier-side lookup exposed through its own
separate Catch Hook or Zapier Interfaces endpoint that only returns data for
a validated contact_id plus one-time token pair; second, a minimal
serverless function such as a Cloudflare Worker holding the private
GoHighLevel token, which the form calls instead of GoHighLevel directly.
Both keep the private token off the client.

Preventing one client from seeing another client's data: the update link
must include an unguessable token, not just a contact ID or email, that the
lookup endpoint validates server-side before returning anything. Do not ship
an update-mode implementation that trusts cid alone.

## Field mapping and JSON schema

See FIELD_MAPPING.md for the full table: form label, internal field name,
JSON property path, GoHighLevel custom-field slug or destination, required
or optional status, data type, validation rule, and whether it applies to
initial mode, update mode, or both.

See sample-payload.json for a complete example payload using fake data.

Top-level payload shape: an object with schema_version, submission_mode,
submitted_at, source, contact_ref, business, onboarding, and services.
onboarding itself contains customers, branding, missed_call, reviews,
notifications, and additional_notes. services is always a JSON array.

## Known reconciliation notes for Ezekiel and Zapier

The original draft form flattened services into service1_name,
service2_name, and so on at the top level. This build instead sends a
proper services array, since the stated goal is that Zapier processes the
services array. Your Zapier service loop step's field mapping will need to
point at the new JSON paths in FIELD_MAPPING.md rather than the old flat
keys, since the shape changed.

Two review-link fields in the original draft (review_link in Section 1 and
review_link_confirm in Section 6) have been consolidated into a single
business.google_review_url, entered once in Section 1 and only displayed
read-only in Section 6, to avoid creating competing names for the same
field.

Several GoHighLevel slugs you shared (glowe_gbp, glowe_business_stage,
glowe_monthly_customers, glowe_missed_followup, glowe_crm,
glowe_primary_goal, glowe_decision_maker, qualification_score) belong to a
Qualification field group with no corresponding field in the 7 onboarding
sections you supplied. These are presumed to be populated by a separate
sales or discovery step, not this form. Please confirm if that is correct.

glowe_review_count and glowe_rating (Qualification group) and
google_review_count and google_star_rating (Reporting and Internal group)
look like two overlapping pairs of fields. This form writes the client's
onboarding-time baseline to glowe_review_count and glowe_rating only. Please
confirm whether the Reporting and Internal pair should be left for your
monthly-report automation to populate instead.

No confirmed GoHighLevel slug was supplied for a per-service price or a
per-service reactivation on/off flag. Both are still collected (useful data,
and the toggle lets an owner exclude one service from reactivation
messaging) and included in the payload as services[].price and
services[].reactivation_enabled, but are not currently mapped to any
GoHighLevel field. Flag to Ezekiel if a slug should be added.

Duration and category dropdown option values use this form's existing
values (for example 15/30/45/60/90/120/custom minutes; lash/brow/hair/skin/
waxing/nails/massage/wellness/fitness/other categories). Please confirm
these exactly match your GoHighLevel dropdown option values; if GoHighLevel
expects different option text, the Zap will need a lookup or formatter
step, or this form's option values should be updated to match exactly.

## Testing performed

HTML/CSS structure integrity was verified (balanced tags, valid nesting).
The JS was verified to parse and run with no syntax errors, with every
function executing correctly. Step navigation and per-step validation
(required fields, email, phone, and URL formats) were confirmed to block
progression and show inline errors correctly. Services were tested by
adding up to 10, confirming an 11th is blocked, removing a middle item
without corrupting order or IDs, duplicate names, special characters,
decimal prices, a custom duration value, and disabling reactivation on a
single service; all produced a correct services array. Payload construction
was checked for correct schema_version, submission_mode, ISO-8601
submitted_at, normalized phone/email/URL values, null for empty optional
numeric fields, and valid round-trip JSON. Submission handling was tested
with a simulated network failure, confirming the error banner appears, the
submit button re-enables, and all entered data is preserved. A simulated
slow successful submission confirmed a second concurrent click is blocked,
and a repeat call after success is also blocked. A localStorage save and
restore round trip was confirmed for resuming an in-progress form. The
update-mode scaffolding (applyExistingData) was confirmed to correctly map
a hypothetical GoHighLevel-shaped response back into the form and services
array. The GitHub Pages build completed successfully per the repository
Actions tab.

Not yet tested, and requiring your explicit go-ahead plus real credentials:
an actual end-to-end POST to the live Zapier webhook, confirming Zapier
receives and loops the array and updates the correct GoHighLevel contact
and fields. This was intentionally not run automatically, to avoid creating
a real (fake-data) contact or update in your live GoHighLevel account
without your confirmation.

## Manual testing (no code knowledge required)

See the VA QA checklist provided separately in the project report.

## Rollback

The entire onboarding form lives in one repository, independent of the
Revenue Assessment repo; deleting, unpublishing, or reverting this repo has
no effect on retention.helloglowe.com. To roll back a bad change, use
GitHub's commit history on index.html (Code, index.html, History) and
revert to the previous commit, or use git revert from a local clone. To
take the onboarding site down entirely without deleting anything, use
Settings, Pages, Unpublish site. The custom domain is controlled entirely
by the CNAME file plus the DNS record; removing the DNS record stops
traffic from reaching this site without touching GitHub at all.

## Maintenance

Any future change to form copy, fields, or logic only requires editing
index.html and committing to main. GitHub Pages republishes automatically
within about a minute. No build step, no manual rebuild.
