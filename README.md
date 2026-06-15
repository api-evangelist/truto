# Truto (truto)

Truto is a unified API and embedded integration platform that enables B2B SaaS companies to ship native integrations without writing integration-specific code. Founded in 2023, Truto uses a declarative, config-driven architecture where every connector is data, not code. The platform provides Unified APIs across four major categories — HRIS (41 providers, 20 resources), ATS (27 providers, 17 resources), CRM (27 providers, 17 resources), and an expanding set of additional categories — plus an Admin API for managing integrated accounts, generating link tokens, and programmatic MCP server provisioning. Truto supports real-time pass-through (no data stored in between), full schema customization via JSONata, and one-API-call MCP server generation for AI agent access. Authentication uses Bearer tokens. Truto supports over 250 integrations and is available as Truto Cloud or on-premise.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/truto/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/truto/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Access:** 3rd-Party

## Tags

- Unified API
- Integration Platform
- HRIS
- ATS
- CRM
- Embedded Integrations
- MCP
- AI Agents
- SaaS

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-05-19

## APIs

### Truto Admin API

The Truto Admin API enables programmatic management of the Truto platform, including creating and managing integrated accounts, generating link tokens for customer-initiated OAuth flows, running post-install actions, and provisioning MCP servers. Integrated accounts represent a connection between your Truto account and a customer's connected app. Link tokens initiate the Truto linking process from within your application. The Admin API uses Bearer token authentication with the tenant API token.

- **Human URL:** [https://truto.one/docs/api-reference/admin](https://truto.one/docs/api-reference/admin)
- **Base URL:** `https://api.truto.one`

#### Tags

- Administration
- Integrated Accounts
- Link Tokens
- MCP

#### Properties

- [Documentation](https://truto.one/docs/api-reference/admin)
- [OpenAPI](https://raw.githubusercontent.com/api-evangelist/truto/refs/heads/main/openapi/truto-admin-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/truto-admin.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-admin.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-ats.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-ats.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-crm.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-crm.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-hris.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-hris.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Truto Unified HRIS API

The Truto Unified HRIS API provides a single normalized interface for accessing HR data across 41 HRIS providers including BambooHR, Workday, Rippling, Gusto, HiBob, Personio, and more. Offers 20 unified resources including employees, employments, companies, groups (departments and roles), timeoff requests, and more. Data is accessed in real-time with no caching. Schema can be extended using JSONata mappings. Authentication uses Bearer token with integrated_account_id query parameter to specify the target connected account.

- **Human URL:** [https://truto.one/docs/api-reference/unified-hris-api](https://truto.one/docs/api-reference/unified-hris-api)
- **Base URL:** `https://api.truto.one/unified/hris`

#### Tags

- HRIS
- Human Resources
- Employees
- Payroll
- Unified API

#### Properties

- [Documentation](https://truto.one/docs/api-reference/unified-hris-api)
- [Documentation](https://truto.one/unified-apis/hris/)
- [OpenAPI](https://raw.githubusercontent.com/api-evangelist/truto/refs/heads/main/openapi/truto-unified-hris-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/truto-admin.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-admin.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-ats.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-ats.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-crm.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-crm.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-hris.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-hris.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Truto Unified ATS API

The Truto Unified ATS API provides a single normalized interface for accessing applicant tracking system data across 27 ATS providers including Greenhouse, Lever, Workable, SmartRecruiters, Ashby, Teamtailor, and more. Offers 17 unified resources covering the full recruiting lifecycle: jobs, candidates, applications, interview stages, interviews, scorecards, offers, EEOC data, departments, offices, reject reasons, activities, attachments, tags, and users. Authentication uses Bearer token with integrated_account_id query parameter.

- **Human URL:** [https://truto.one/docs/api-reference/unified-ats-api](https://truto.one/docs/api-reference/unified-ats-api)
- **Base URL:** `https://api.truto.one/unified/ats`

#### Tags

- ATS
- Applicant Tracking
- Recruiting
- Jobs
- Candidates
- Unified API

#### Properties

- [Documentation](https://truto.one/docs/api-reference/unified-ats-api)
- [Documentation](https://truto.one/unified-apis/ats/)
- [OpenAPI](https://raw.githubusercontent.com/api-evangelist/truto/refs/heads/main/openapi/truto-unified-ats-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/truto-admin.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-admin.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-ats.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-ats.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-crm.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-crm.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-hris.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-hris.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Truto Unified CRM API

The Truto Unified CRM API provides a single normalized interface for accessing CRM data across 27 CRM providers including Salesforce, HubSpot, Pipedrive, Attio, Outreach, and more. Offers 17 unified resources including accounts, contacts, opportunities, tasks, users, stages, engagements, notes, fields, associations, and more. Data is accessed in real-time via pass-through to underlying CRM APIs. Authentication uses Bearer token with integrated_account_id query parameter.

- **Human URL:** [https://truto.one/docs/api-reference/unified-crm-api](https://truto.one/docs/api-reference/unified-crm-api)
- **Base URL:** `https://api.truto.one/unified/crm`

#### Tags

- CRM
- Customer Relationship Management
- Contacts
- Opportunities
- Unified API

#### Properties

- [Documentation](https://truto.one/docs/api-reference/unified-crm-api)
- [Documentation](https://truto.one/unified-apis/crm/)
- [OpenAPI](https://raw.githubusercontent.com/api-evangelist/truto/refs/heads/main/openapi/truto-unified-crm-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/truto-admin.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-admin.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-ats.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-ats.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-crm.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-crm.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [Postman Collection](collections/truto-unified-hris.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/truto-unified-hris.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [LinkedIn](https://www.linkedin.com/company/gettruto)
- [Website](https://truto.one/)
- [Documentation](https://truto.one/docs/)
- [Documentation](https://truto.one/docs/api-reference/overview/introduction)
- [Getting Started](https://truto.one/docs/getting-started)
- [Integrations](https://truto.one/integrations/)
- [Documentation](https://truto.one/unified-apis/)
- [Blog](https://truto.one/blog/)
- [Git Hub](https://github.com/trutohq)
- [Privacy Policy](https://truto.one/docs/api-reference/overview/introduction)
- [Spectral Rules](https://raw.githubusercontent.com/api-evangelist/truto/refs/heads/main/rules/truto-rules.yml)
- [M C P Server](https://truto.one/blog/announcing-truto-docs-mcp-stop-ai-hallucinations-in-api-integrations/)
- [Agent Skill](https://truto.one/blog/truto-agent-skills-stop-ai-hallucinations-when-building-integrations/)
- [L L Ms Txt](https://truto.one/llms.txt)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
