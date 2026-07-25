# Field Mapping — Glowe Onboarding Form

Every field the form collects, where it goes in the JSON payload, and its
intended GoHighLevel destination. "Both" in the Mode column means the field
is used identically in initial and update submissions.

Legend for GoHighLevel destination: a slug in code means it matches a slug
Ezekiel confirmed. "Zapier/internal only" means the field is sent to Zapier
for workflow logic but has no confirmed GoHighLevel custom field.

## Section 1 - Your Business

| Label | Field id | JSON path | GHL destination | Required | Type | Mode |
|---|---|---|---|---|---|---|
| Business name | biz_name | business.name | glowe_biz_name | Yes | string | Both |
| Your first name | owner_name | business.owner_first_name | Zapier/internal only | Yes | string | Both |
| Business phone | biz_phone | business.phone | glowe_biz_phone | Yes | string, normalized +1XXXXXXXXXX | Both |
| Business email | biz_email | business.email | glowe_biz_email | Yes | string, lowercased | Both |
| Business website URL | biz_website | business.website_url | glowe_website_url | No | URL | Both |
| Business address | biz_address | business.address.line1 | glowe_business_address | Yes | string | Both |
| City | biz_city | business.address.city | glowe_city | Yes | string | Both |
| State | biz_state | business.address.state | glowe_state | Yes | string | Both |
| ZIP Code | biz_postcode | business.address.postal_code | glowe_postcode | Yes | string | Both |
| Country (not a form field) | - | business.address.country | glowe_country | - | constant United States | Both |
| Timezone | biz_timezone | business.timezone | glowe_timezone | Yes | string enum | Both |
| Google Business Profile URL | gbp_url | business.google_business_profile_url | glowe_google_business_profile_url | No | URL | Both |
| Google review link | review_link | business.google_review_url | glowe_google_review_url | Yes | URL | Both |
| Primary service category | service_category | business.primary_service_category | business_primary_service | Yes | string enum | Both |
| Booking platform | booking_platform | business.booking_platform | Zapier/internal only | Yes | string enum | Both |
| Booking link URL | booking_url | business.booking_url | glowe_booking_url | No | URL | Both |
| Instagram URL | instagram_url | business.instagram_url | glowe_instagram_url | No | URL | Both |
| Facebook URL | facebook_url | business.facebook_url | glowe_facebook_url | No | URL | Both |

## Section 2 - Service Menu (0-10 entries, array)

Each entry in services[]:

| Label | Field id pattern | JSON path | GHL destination (n = 1-10 by position) | Required | Type | Mode |
|---|---|---|---|---|---|---|
| Service name | svc_name_i | services[n].name | service{n}_name | Yes | string | Both |
| Category | svc_cat_i | services[n].category | service{n}_category | No | string enum | Both |
| Duration | svc_duration_i / svc_duration_custom_i | services[n].duration_minutes | service{n}_duration | No | integer minutes or null | Both |
| Price (approx.) | svc_price_i | services[n].price | Not confirmed - Zapier/internal only | No | number or null | Both |
| Return interval | svc_interval_num_i | services[n].return_interval | service{n}_return_interval | Yes per service | integer | Both |
| Return unit | svc_interval_unit_i | services[n].return_unit | service{n}_return_unit | Yes per service | enum days/weeks/months/years | Both |
| Notes for messaging | svc_notes_i | services[n].notes | service{n}_notes | No | string | Both |
| Include in reactivation | svc_reactivation_i | services[n].reactivation_enabled | Not confirmed - Zapier/internal only | No | boolean | Both |
| system generated | - | services[n].service_number | Zapier loop index, 1-based array position | - | integer | Both |
| system generated | - | services[n].service_id | Not sent to GHL, internal tracking id only | - | string | Both |

services is always a JSON array ([] when empty), 0-10 items.

## Section 3 - Your Customers

| Label | Field id | JSON path | GHL destination | Required | Type | Mode |
|---|---|---|---|---|---|---|
| Active client count | contact_count | onboarding.customers.active_client_range | glowe_monthly_customers (unconfirmed option values, see README) | Yes | string enum | Both |
| Average visit value | avg_visit_value | onboarding.customers.avg_visit_value | business_visit_value | Yes | number | Both |
| Google review count | review_count | onboarding.customers.review_count | glowe_review_count | No | integer or null | Both |
| Google star rating | star_rating | onboarding.customers.star_rating | glowe_rating | No | string enum | Both |

## Section 4 - Voice and Tone

| Label | Field id | JSON path | GHL destination | Required | Type | Mode |
|---|---|---|---|---|---|---|
| Tone preference | selectedTone (radio cards) | onboarding.branding.tone | business_tone | Yes | string enum | Both |
| Words/phrases to never use | never_use | onboarding.branding.restricted_words | restricted_words | No | string | Both |
| Active promotions | promotions | onboarding.branding.current_promotion | business_current_promo | No | string | Both |
| Tone preview approved | tone_preview_approved | onboarding.branding.tone_preview_approved | tone_preview_approved | No | boolean | Both |

## Section 5 - Missed Call Preferences

| Label | Field id | JSON path | GHL destination | Required | Type | Mode |
|---|---|---|---|---|---|---|
| During-hours response | during_hours_response | onboarding.missed_call.during_hours_response | Zapier/internal only | Yes | string enum | Both |
| Custom during-hours message | during_hours_custom | onboarding.missed_call.during_hours_custom_message | Zapier/internal only | No | string | Both |
| After-hours response | after_hours_response | onboarding.missed_call.after_hours_response | Zapier/internal only | Yes | string enum | Both |
| Custom after-hours message | after_hours_custom | onboarding.missed_call.after_hours_custom_message | Zapier/internal only | No | string | Both |
| Business hours Mon-Sun | day_open / day_close | onboarding.missed_call.business_hours.day.open/close | Zapier/internal only | No | HH:MM string or null | Both |
| Internal missed call alert phone | alert_phone | onboarding.missed_call.internal_alert_phone | Zapier/internal only | No | string, normalized phone | Both |

## Section 6 - Review Setup

| Label | Field id | JSON path | GHL destination | Required | Type | Mode |
|---|---|---|---|---|---|---|
| Google review link (display only, value from Section 1) | review_link_display (read-only) | uses business.google_review_url | glowe_google_review_url (same field as Section 1) | Yes (enforced in Section 1) | URL | Both |
| Review timing preference | review_timing | onboarding.reviews.timing_preference | Zapier/internal only | No | string enum | Both |
| Personal note on review requests | review_personal_note | onboarding.reviews.personal_note | Zapier/internal only | No | string | Both |

## Section 7 - Notifications and Reporting

| Label | Field id | JSON path | GHL destination | Required | Type | Mode |
|---|---|---|---|---|---|---|
| Missed call alert phone (SMS) | notif_missed_phone | onboarding.notifications.missed_call_alert_phone | Zapier/internal only | Yes | string, normalized phone | Both |
| New lead notification email | notif_lead_email | onboarding.notifications.new_lead_email | Not confirmed - Zapier/internal only | Yes | string, lowercased | Both |
| Monthly report email | notif_report_email | onboarding.notifications.monthly_report_email | owner_report_email | Yes | string, lowercased | Both |
| Anything else we should know | additional_notes | onboarding.additional_notes | Zapier/internal only, consider a GHL note not a custom field | No | string | Both |

## Meta / system fields

| Field | JSON path | GHL destination | Notes |
|---|---|---|---|
| Schema version | schema_version | n/a | Always "1.0" for this build |
| Submission mode | submission_mode | n/a | "initial" or "update", read from ?mode= URL param |
| Submitted timestamp | submitted_at | n/a | ISO-8601, new Date().toISOString() |
| Source | source | n/a | Always "glowe_onboarding_form" |
| GoHighLevel contact id (update mode) | contact_ref.ghl_contact_id | Used by Zapier/GHL to find the existing contact | From ?cid= URL param, null on initial |
| Update token (update mode) | contact_ref.update_token | Used only for secure server-side lookup validation | From ?token= URL param, null on initial, never trust client-side alone |
| Onboarding status | not sent by form | onboarding_status | Recommend Zapier sets this internally rather than trusting client input |

## Fields intentionally NOT collected by this form

These GoHighLevel slugs you shared have no corresponding field in the 7
onboarding sections supplied, and are presumed to be populated by a separate
process (sales/discovery call, or internal Glowe reporting automation):
glowe_gbp, glowe_business_stage, glowe_monthly_customers (note: contact_count
above is tentatively mapped here, please confirm), glowe_missed_followup,
glowe_crm, glowe_primary_goal, glowe_decision_maker, qualification_score,
activation_date, qa_status, account_status, last_monthly_report,
cancellation_reason, exit_feedback, google_review_count, google_star_rating.
