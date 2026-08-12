# LiveIntent

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

LiveIntent is a people-based email marketing, advertising and identity platform. It publishes four
partner APIs, two of which carry machine-readable OpenAPI 3.0.0 definitions:

| API | Contract | Base URL |
|---|---|---|
| [Audiences API](https://audiences.liveintent.com/api-guide) | OpenAPI 3.0.0 — 12 operations, 48 schemas | `https://audiences.liveintent.com` |
| [Privacy Management API](https://privacy.liadm.com/api-guide) | OpenAPI 3.0.0 — 8 operations, 17 schemas | `https://privacy.liadm.com` |
| [Reporting API](https://support.liveintent.com/connecting-to-liveintents-reporting-api/) | Documented only — no spec published | `https://connect.liveintent.com` |
| [Programmatic Bidding API](https://support.liveintent.com/programmatic-bidding-api/) | IAB OpenRTB 2.5 + Native 1.1/1.2 | DSP-supplied bidding URL |

Neither OpenAPI document was published at a standalone URL — both were harvested from the Redocly
state embedded in LiveIntent's own rendered API guides. All API access is bearer-token based and
provisioned by a LiveIntent account team; there is no self-service signup, no published pricing, and
no OAuth authorization server. LiveIntent has been acquired by Zeta Global.

Backed by: battery-ventures, bullpen-capital — https://www.liveintent.com
