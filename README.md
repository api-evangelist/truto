# Truto

Truto is a unified API and embedded integration platform that enables B2B SaaS companies to ship native integrations without writing integration-specific code. Founded in 2023, Truto uses a declarative, config-driven architecture where every connector is data, not code. The platform provides Unified APIs for HRIS (41 providers), ATS (27 providers), and CRM (27 providers), plus an Admin API for managing integrated accounts and provisioning MCP servers for AI agent access.

Truto supports real-time pass-through (no data stored between calls), full schema customization via JSONata, and one-API-call MCP server generation. Available as Truto Cloud or on-premise.

**URL:** [https://raw.githubusercontent.com/api-evangelist/truto/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/truto/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Position:** Consuming
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

## APIs

| API | Description | Resources | Auth | Docs |
|-----|-------------|-----------|------|------|
| Truto Admin API | Manage integrated accounts, generate link tokens, provision MCP servers | Integrated Accounts, Link Tokens, MCP Servers | Bearer | [Docs](https://truto.one/docs/api-reference/admin) |
| Truto Unified HRIS API | Normalized HR data across 41 HRIS providers (BambooHR, Workday, Rippling, Gusto, HiBob, Personio) | 20 resources: Employees, Employments, Companies, Groups, Timeoff | Bearer | [Docs](https://truto.one/docs/api-reference/unified-hris-api) |
| Truto Unified ATS API | Normalized recruiting data across 27 ATS providers (Greenhouse, Lever, Workable, SmartRecruiters, Ashby) | 17 resources: Jobs, Candidates, Applications, Offers, Interviews | Bearer | [Docs](https://truto.one/docs/api-reference/unified-ats-api) |
| Truto Unified CRM API | Normalized CRM data across 27 CRM providers (Salesforce, HubSpot, Pipedrive, Attio, Outreach) | 17 resources: Contacts, Accounts, Opportunities, Tasks, Stages | Bearer | [Docs](https://truto.one/docs/api-reference/unified-crm-api) |

## Artifacts

| Artifact | Location | Description |
|----------|----------|-------------|
| OpenAPI: Admin | [openapi/truto-admin-openapi.yml](openapi/truto-admin-openapi.yml) | OpenAPI 3.0.3 spec for the Truto Admin API |
| OpenAPI: Unified HRIS | [openapi/truto-unified-hris-openapi.yml](openapi/truto-unified-hris-openapi.yml) | OpenAPI 3.0.3 spec for the Unified HRIS API |
| OpenAPI: Unified ATS | [openapi/truto-unified-ats-openapi.yml](openapi/truto-unified-ats-openapi.yml) | OpenAPI 3.0.3 spec for the Unified ATS API |
| OpenAPI: Unified CRM | [openapi/truto-unified-crm-openapi.yml](openapi/truto-unified-crm-openapi.yml) | OpenAPI 3.0.3 spec for the Unified CRM API |
| Spectral Rules | [rules/truto-rules.yml](rules/truto-rules.yml) | API linting rules for Truto API standards |
| Naftiko: Admin | [capabilities/shared/admin.yaml](capabilities/shared/admin.yaml) | Shared capability definition for Admin API |
| Naftiko: Unified HRIS | [capabilities/shared/unified-hris.yaml](capabilities/shared/unified-hris.yaml) | Shared capability definition for Unified HRIS API |
| Naftiko: Unified ATS | [capabilities/shared/unified-ats.yaml](capabilities/shared/unified-ats.yaml) | Shared capability definition for Unified ATS API |
| Naftiko: Unified CRM | [capabilities/shared/unified-crm.yaml](capabilities/shared/unified-crm.yaml) | Shared capability definition for Unified CRM API |
| Naftiko: Unified Integrations | [capabilities/unified-integrations.yaml](capabilities/unified-integrations.yaml) | Workflow capability combining all 4 APIs |
| JSON Schema: Integrated Account | [json-schema/truto-integrated-account-schema.json](json-schema/truto-integrated-account-schema.json) | JSON Schema for integrated account records |
| JSON Schema: Employee | [json-schema/truto-employee-schema.json](json-schema/truto-employee-schema.json) | JSON Schema for normalized HRIS employee records |
| JSON Schema: Candidate | [json-schema/truto-candidate-schema.json](json-schema/truto-candidate-schema.json) | JSON Schema for normalized ATS candidate records |
| JSON Structure: Integrated Account | [json-structure/truto-integrated-account-structure.json](json-structure/truto-integrated-account-structure.json) | Structure reference for integrated account fields |
| JSON Structure: Employee | [json-structure/truto-employee-structure.json](json-structure/truto-employee-structure.json) | Structure reference for normalized employee fields |
| JSON-LD Context | [json-ld/truto-context.jsonld](json-ld/truto-context.jsonld) | Linked data context mapping to schema.org and HR ontologies |
| Examples | [examples/](examples/) | Request/response examples for Admin, HRIS, ATS, and CRM APIs |
| Vocabulary | [vocabulary/truto-vocabulary.yml](vocabulary/truto-vocabulary.yml) | Domain vocabulary for integration platform concepts |

## GitHub Repositories

- [trutohq/truto-ts-sdk](https://github.com/trutohq/truto-ts-sdk) - TypeScript/JavaScript SDK
- [trutohq/truto-python-sdk](https://github.com/trutohq/truto-python-sdk) - Python SDK
- [trutohq/truto-mcp-stdio](https://github.com/trutohq/truto-mcp-stdio) - MCP stdio proxy for HTTP Streamable MCP servers
- [trutohq/truto-jsonata](https://github.com/trutohq/truto-jsonata) - Truto JSONata implementation

## Authentication

All Truto APIs use Bearer token authentication. Obtain your token from the Truto dashboard:

```
Authorization: Bearer YOUR_TRUTO_API_TOKEN
```

For Unified API requests, include the `integrated_account_id` query parameter to specify which connected provider instance to query:

```
GET /unified/hris/employees?integrated_account_id=ia_abc123
Authorization: Bearer YOUR_TRUTO_API_TOKEN
```

## Key Concepts

- **Integrated Account**: A connection record linking your Truto tenant to a customer's connected app. Created via link token flow or Admin API.
- **Link Token**: Short-lived credential to initiate a customer-facing OAuth/API key connection flow.
- **Pass-Through**: Real-time data access with no caching — customer data goes directly from source to your application.
- **JSONata Mappings**: Customize any unified model field mapping without code deploys.
- **MCP Server**: Provision an AI-accessible MCP server for any integrated account with a single API call.

## Resources

- [Truto Website](https://truto.one/)
- [Developer Documentation](https://truto.one/docs/)
- [Getting Started](https://truto.one/docs/getting-started)
- [API Reference](https://truto.one/docs/api-reference/overview/introduction)
- [Integration Directory](https://truto.one/integrations/)
- [Truto Blog](https://truto.one/blog/)
- [GitHub: trutohq](https://github.com/trutohq)

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-05-03

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
