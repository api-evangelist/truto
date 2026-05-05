---
title: "How to Automate User Access Reviews Across Unmanaged SaaS Using Unified APIs"
url: "https://truto.one/blog/how-to-automate-user-access-reviews-across-unmanaged-saas-using-unified-apis/"
date: "Wed, 29 Apr 2026 00:00:00 GMT"
author: "uday@truto.one (Uday Gajavalli)"
feed_url: "https://truto.one/blog/feed.xml"
---
<p>If your engineering roadmap includes the requirement to <em>"automate quarterly user access reviews across every SaaS app our customers use"</em>, you are staring down a massive architectural challenge. The implicit search query your enterprise customers are asking is how to automatically pull a list of users, roles, and access levels from every single application their employees use, without relying on manual spreadsheets.</p>
<p>Your product manager drops this requirement in a sprint planning meeting. The goal is to automate SOC 2, SOX, and ISO 27001 user access reviews. Your customers want to stop exporting CSVs and chasing department heads via Slack to confirm who still needs access to Jira, HubSpot, or the 100+ other tools their teams use.</p>
<p>As we covered in our <a href="/developer-tutorial-pulling-user-lists-end-to-end-from-any-saas-api/">developer tutorial on pulling user lists</a>, automating this workflow requires moving beyond one-off API scripts. You need a highly scalable, unified architecture capable of normalizing identity data across hundreds of disparate systems. Building this yourself, one integration at a time, is an engineering trap. You will spend the next three years maintaining point-to-point connectors while fighting terrible vendor API documentation, aggressive rate limits, and undocumented edge cases.</p>
<p>This guide breaks down the architectural reality of identity sprawl, why point-to-point connectors fail at scale, and exactly how to automate quarterly user access reviews across unmanaged SaaS applications using a declarative Unified User Directory API.</p>
<h2 id="the-compliance-nightmare-why-manual-user-access-reviews-break-at-scale"><a class="anchor-link" href="#the-compliance-nightmare-why-manual-user-access-reviews-break-at-scale">The Compliance Nightmare: Why Manual User Access Reviews Break at Scale</a></h2>
<p>Security frameworks like SOC 2 and SOX mandate periodic user access reviews (UARs). Organizations must prove that access to their systems is restricted to authorized personnel, based on the principle of least privilege, and that dormant accounts are promptly deprovisioned.</p>
<p>UARs are a hard compliance control. If a service organization's policies say they conduct quarterly logical access reviews, that organization will need to provide quarterly evidence from the preceding year confirming those reviews were conducted. Auditors want evidence the control operated throughout the observation period. The most common failure is having a policy that says user access is reviewed quarterly, but only being able to produce one access review from six months ago. The policy exists, but the evidence doesn't support that it operates as described.</p>
<p>Historically, IT and security teams handled this manually. The traditional workflow looks like this:</p>
<ol>
<li>IT exports a CSV from every SaaS admin panel they have credentials to.</li>
<li>The list is split by application owner and emailed to managers.</li>
<li>Managers eyeball spreadsheets, cross-reference them against the HRIS, and rubber-stamp approvals.</li>
<li>Someone consolidates the responses into a master audit folder.</li>
<li>Three months later, repeat.</li>
</ol>
<p>This manual process is a massive drain on resources and breaks immediately upon contact with reality. CSVs are inherently stale the moment they are exported. Apps without admin API access force IT to scrape UIs. Half the time, the right "manager" no longer works there.</p>
<p>If you are a GRC, SSPM, or identity governance product trying to automate this for your customers, you face an even harder problem: you don't have admin credentials at all. You have an OAuth connection from your end user, which means you need to programmatically pull the user list, role assignments, and last login data through whatever API the vendor offers. Multiply that by 100 SaaS apps per customer, and you have a severe engineering problem.</p>
<h2 id="the-blind-spot-unmanaged-saas-and-identity-sprawl"><a class="anchor-link" href="#the-blind-spot-unmanaged-saas-and-identity-sprawl">The Blind Spot: Unmanaged SaaS and Identity Sprawl</a></h2>
<p>The standard engineering reflex when tasked with pulling user lists is to integrate with Okta, Microsoft Entra ID, or Google Workspace. If a customer wants user data, they should just pull it from the Identity Provider (IdP) via SCIM (System for Cross-domain Identity Management), a standard we detail in our <a href="/what-are-directory-integrations-2026-saas-architecture-guide/">guide to directory integrations</a>.</p>
<p>This approach fails because SCIM and IdPs only see a visible slice of the SaaS estate. Connecting to major IdPs is table stakes, but it leaves a massive blind spot where the real risk lives.</p>
<p>Product-led growth has decentralized software purchasing. Marketing teams buy their own SEO tools. Engineering teams spin up new monitoring dashboards. Sales teams expense new prospecting software. Gartner projects that by 2027, 75% of employees will use technology outside of IT's purview. Organizations officially recognize only 10% of Shadow IT cloud services, which actually operate at ten times that amount. A typical business operates 108 identified cloud services, yet it secretly uses 975 additional cloud services that exist without detection.</p>
<p>Identity sprawl directly translates to breaches. Excessive permissions remain a leading cause of SaaS security incidents. Studies show that 85% of SaaS users have more privileges than their roles require, creating unnecessary attack surfaces. AppOmni's 2025 data reveals that 75% of organizations experienced a SaaS security incident in the past 12 months, with a significant number of these incidents tied to unauthorized applications.</p>
<p>The attack surface is growing exponentially. A 2026 Grip Security report found a year-over-year 490% spike in public SaaS attacks, with 80% of documented incidents involving PII and/or customer data. The poster boy example is the Salesloft Drift incident, where attackers stole active OAuth tokens used by customers to connect the Drift Chatbot to local Salesforce installations. Armed with legitimate tokens, attackers impersonated Drift and logged directly into Salesforce. One breach of a SaaS app cascaded into hundreds of compromises.</p>
<p>The takeaway: an access review program that only covers centralized IdPs misses the long tail of unmanaged SaaS. To provide genuine security value, your platform must connect directly to the underlying SaaS applications to audit local accounts, bypass shadow IT blind spots, and read the actual permissions granted within the app. For a deeper dive into this architectural gap, see our guide on <a href="/the-long-tail-of-identity-why-your-grc-platform-needs-coverage-beyond-the-top-5-idps/">the long tail of identity</a>.</p>
<h2 id="why-building-point-to-point-integrations-for-user-lists-fails"><a class="anchor-link" href="#why-building-point-to-point-integrations-for-user-lists-fails">Why Building Point-to-Point Integrations for User Lists Fails</a></h2>
<p>To audit the long tail of SaaS, you must integrate directly with the APIs of the applications your customers use. The instinct of every senior engineer staring at this requirement is to write a quick script. "It's just <code>GET /users</code> from each app, right?"</p>
<p>If you decide to build these <a href="/scaling-grc-integrations-why-compliance-platforms-are-abandoning-point-to-point-connectors/">point-to-point connectors</a> in-house, the math quickly becomes terrifying. The average enterprise uses over 130 different SaaS applications. A realistic engineering team can ship two or three high-quality, production-grade integrations per quarter. Building 100+ this way means your access review feature ships in 2030.</p>
<p>The difficulty is not just writing the HTTP requests. Three weeks in, you will find your codebase infected with integration-specific logic trying to handle the following:</p>
<ul>
<li><strong>Authentication Chaos:</strong> Application A uses standard OAuth 2.0 authorization code. Application B requires a static API key passed in a custom header. Application C requires you to exchange a signed JWT for a short-lived session token every 15 minutes. Others use Basic Auth or session cookies refreshed via post-install hooks. Each one is a separate code path with its own failure modes.</li>
<li><strong>Pagination Hell:</strong> Pagination is a tax on every endpoint. Application A uses cursor-based pagination (<code>?after=</code>). Application B uses offset and limit parameters (<code>?page=</code>). Application C uses HTTP Link headers (RFC 5988). Your sync job must handle all strategies flawlessly to ensure no users are skipped during an audit.</li>
<li><strong>Data Model Fragmentation:</strong> Field shapes rarely match a unified concept of a "user." HubSpot exposes contacts inside <code>properties.firstname</code>. Salesforce uses flat PascalCase. Workday calls them workers. Each provider has its own enums for status (e.g., active, suspended, deleted, archived) and license type.</li>
<li><strong>Rate Limits:</strong> Aggressive and undocumented. A single tenant scan of 5,000 users can burn the entire daily API quota for some applications.</li>
<li><strong>Missing Webhooks:</strong> Half of these applications do not emit user lifecycle events, forcing you to poll on a schedule.</li>
<li><strong>Silent Token Expiration:</strong> A nightly sync that worked for six months will quietly start failing when refresh tokens rotate or scopes change.</li>
</ul>
<p>If you hardcode these differences, you end up with a massive, fragile codebase filled with <code>if (provider === 'hubspot')</code> statements. Every time a vendor deprecates an endpoint or changes a field name, your sync jobs break, your customers fail their compliance audits, and your engineering team drops feature work to fix technical debt.</p>
<div class="callout callout-warning"><span class="callout-title">
<svg class="lucide lucide-triangle-alert" fill="none" height="18" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" width="18" xmlns="http://www.w3.org/2000/svg">
  <path d="m21.73 18-8-14a2 2 0 0 0-3.48 0l-8 14A2 2 0 0 0 4 21h16a2 2 0 0 0 1.73-3">
  <path d="M12 9v4">
  <path d="M12 17h.01">
</svg>
Warning</span><p><strong>The Maintenance Trap:</strong> The hidden cost is not the initial build - it is the ongoing maintenance. APIs deprecate endpoints, change pagination, rotate auth flows, and introduce breaking changes. Every change requires a code deploy, regression testing, and a release. Do that 100 times in parallel and your product engineering team will become an integration maintenance team.</p></div>
<h2 id="how-to-automate-quarterly-user-access-reviews-using-unified-directory-apis"><a class="anchor-link" href="#how-to-automate-quarterly-user-access-reviews-using-unified-directory-apis">How to Automate Quarterly User Access Reviews Using Unified Directory APIs</a></h2>
<p>To solve this problem at scale, you must abandon integration-specific code. As explained in our guide on <a href="/how-to-pull-user-lists-from-any-saas-app-with-a-unified-directory-api/">pulling user lists with a Unified Directory API</a>, a <strong>Unified User Directory API</strong> abstracts the per-vendor mess into a single, normalized schema. Instead of learning the idiosyncrasies of 100 individual platforms, you call one endpoint, and the unified platform handles the translation, authentication, and pagination behind the scenes.</p>
<h3 id="the-unified-relational-schema"><a class="anchor-link" href="#the-unified-relational-schema">The Unified Relational Schema</a></h3>
<p>To automate access reviews, a Unified API normalizes data into an identity-centric relational model. The resources you need for an audit-ready program look like this:</p>
<table>
<thead>
<tr>
<th align="left">Unified Resource</th>
<th align="left">What It Answers</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left"><code>Users</code></td>
<td align="left">Who exists in the app and what is their status (active / suspended / deactivated)?</td>
</tr>
<tr>
<td align="left"><code>Groups</code></td>
<td align="left">Which departments, teams, or distribution lists exist?</td>
</tr>
<tr>
<td align="left"><code>Roles</code></td>
<td align="left">What permission tiers does the app define?</td>
</tr>
<tr>
<td align="left"><code>RoleAssignments</code></td>
<td align="left">Which user has which role? (The heart of access reviews)</td>
</tr>
<tr>
<td align="left"><code>Licenses</code></td>
<td align="left">What paid seats are allocated and to whom?</td>
</tr>
<tr>
<td align="left"><code>Activities</code></td>
<td align="left">Login events and admin actions for SIEM ingestion</td>
</tr>
</tbody>
</table>
<figure class="mermaid-container"><pre class="mermaid">graph TD
    Org[Organization / Workspace] --&gt;|Contains| User[User]
    Org --&gt;|Defines| Role[Role]
    Org --&gt;|Defines| Group[Group]
    User --&gt;|Belongs to| Group
    User --&gt;|Granted access via| RoleAssignment[RoleAssignment]
    RoleAssignment --&gt;|Links to| Role</pre></figure>
<p>The call to fetch users from any connected app collapses to a single standardized request:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #61AFEF;">curl</span><span style="color: #D19A66;"> -X</span><span style="color: #98C379;"> GET</span><span style="color: #98C379;"> "https://api.unified-platform.com/unified/user-directory/users?integrated_account_id=acc_123"</span><span style="color: #56B6C2;"> \</span></span>
<span><span style="color: #D19A66;">  -H</span><span style="color: #98C379;"> "x-api-key: </span><span style="color: #E06C75;">$YOUR_API_KEY</span><span style="color: #98C379;">"</span></span></code></pre></figure>
<p>You receive a normalized response, regardless of whether it came from Okta, BambooHR, GitHub, Salesforce, or a niche industry app:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #ABB2BF;">{</span></span>
<span><span style="color: #E06C75;">  "result"</span><span style="color: #ABB2BF;">: [</span></span>
<span><span style="color: #ABB2BF;">    {</span></span>
<span><span style="color: #E06C75;">      "id"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"u_a91"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">      "email"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"jane@acme.com"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">      "first_name"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"Jane"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">      "last_name"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"Doe"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">      "status"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"active"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">      "groups"</span><span style="color: #ABB2BF;">: [</span><span style="color: #98C379;">"engineering"</span><span style="color: #ABB2BF;">, </span><span style="color: #98C379;">"on-call"</span><span style="color: #ABB2BF;">],</span></span>
<span><span style="color: #E06C75;">      "last_login_at"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"2026-04-12T08:14:11Z"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">      "remote_data"</span><span style="color: #ABB2BF;">: { </span><span style="color: #E06C75;">"...the raw provider payload..."</span><span style="color: #ABB2BF;">: </span><span style="color: #D19A66;">true</span><span style="color: #ABB2BF;"> }</span></span>
<span><span style="color: #ABB2BF;">    }</span></span>
<span><span style="color: #ABB2BF;">  ],</span></span>
<span><span style="color: #E06C75;">  "next_cursor"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"eyJvZmZzZXQiOjEwMH0="</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">  "result_count"</span><span style="color: #ABB2BF;">: </span><span style="color: #D19A66;">100</span></span>
<span><span style="color: #ABB2BF;">}</span></span></code></pre></figure>
<p>The <code>remote_data</code> field is critical for compliance - the platform never throws away the raw response, so when an auditor asks for the original provider record, you have the exact evidence.</p>
<h3 id="zero-integration-specific-code-via-jsonata"><a class="anchor-link" href="#zero-integration-specific-code-via-jsonata">Zero Integration-Specific Code via JSONata</a></h3>
<p>The architectural secret to a scalable unified API is moving integration logic out of the codebase and into configuration data.</p>
<p>Advanced platforms achieve this using JSONata - a functional query and transformation language for JSON. Every field mapping, query translation, and conditional logic rule is stored as a JSONata expression. When a request is made, a generic execution engine reads the configuration, fetches the data, and evaluates the expression.</p>
<p>For example, transforming a complex response from HubSpot into a clean Unified User object requires a single expression:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span>response.{</span></span>
<span><span>  "id": $string(id),</span></span>
<span><span>  "email": properties.email,</span></span>
<span><span>  "first_name": properties.firstname,</span></span>
<span><span>  "last_name": properties.lastname,</span></span>
<span><span>  "status": properties.archived = "true" ? "deactivated" : "active"</span></span>
<span><span>}</span></span></code></pre></figure>
<p>The Salesforce equivalent is a different expression against <code>Id</code>, <code>FirstName</code>, <code>Email</code>, and <code>IsActive</code>. Both produce the exact same unified output. This means adding support for the 101st SaaS application is a data operation, not a code deployment. Your code never branches on the provider name. For a deeper read on this pattern, see our <a href="/zero-integration-specific-code-how-to-ship-new-api-connectors-as-data-only-operations/">zero integration-specific code writeup</a>.</p>
<h2 id="architecting-the-sync-best-practices-for-continuous-access-discovery"><a class="anchor-link" href="#architecting-the-sync-best-practices-for-continuous-access-discovery">Architecting the Sync: Best Practices for Continuous Access Discovery</a></h2>
<p>Wiring up a Unified API is only the first step. Running <code>GET /users</code> reliably across thousands of customer-connected accounts, every night, in a way auditors will accept, is the actual engineering challenge. Your backend architecture must handle the harsh realities of background synchronization at scale.</p>
<h3 id="1-proactive-oauth-token-management-and-distributed-locks"><a class="anchor-link" href="#1-proactive-oauth-token-management-and-distributed-locks">1. Proactive OAuth Token Management and Distributed Locks</a></h3>
<p>Most background sync failures are not API failures - they are token failures. Access tokens expire frequently (often every 60 minutes). If a quarterly access review sync job running at 2 AM attempts to use an expired token, the request fails.</p>
<p>The naive approach is to catch the 401 Unauthorized error, refresh the token, and retry. In a high-concurrency environment, this causes severe race conditions. If five sync jobs wake up simultaneously and attempt to refresh the same token, the identity provider flags it as a replay attack, issues an <code>invalid_grant</code> error, and permanently revokes the connection.</p>
<p>The reliable pattern utilizes proactive refreshes and distributed mutex locks. The platform schedules background work to refresh the token 60 to 180 seconds before expiration. It acquires a durable, per-account mutex lock so only one refresh runs at a time. Concurrent requests await the lock resolution rather than firing duplicate requests. On unrecoverable failures (like a user revoking access), the account is marked as <code>needs_reauth</code> and a webhook fires so your application can prompt the customer to reconnect.</p>
<h3 id="2-standardized-rate-limit-handling-at-the-edge"><a class="anchor-link" href="#2-standardized-rate-limit-handling-at-the-edge">2. Standardized Rate Limit Handling at the Edge</a></h3>
<p>Pulling thousands of users and their role assignments inevitably triggers upstream API rate limits.</p>
<div class="callout callout-warning"><span class="callout-title">
<svg class="lucide lucide-triangle-alert" fill="none" height="18" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" width="18" xmlns="http://www.w3.org/2000/svg">
  <path d="m21.73 18-8-14a2 2 0 0 0-3.48 0l-8 14A2 2 0 0 0 4 21h16a2 2 0 0 0 1.73-3">
  <path d="M12 9v4">
  <path d="M12 17h.01">
</svg>
Warning</span><p><strong>Architectural Reality Check:</strong> No unified API platform can magically absorb rate limits for you. If the upstream SaaS application returns an HTTP 429 Too Many Requests error, that error must be passed back to your application. Your application owns the retry policy, because background syncs and interactive requests require different backoff behaviors.</p></div>
<p>The problem is that every vendor communicates rate limit resets differently (<code>X-RateLimit-Reset</code>, <code>Retry-After</code>, custom headers). To make retry logic manageable, a robust unified API intercepts varied upstream responses and normalizes them into standard IETF draft headers:</p>
<ul>
<li><code>ratelimit-limit</code>: The maximum number of requests permitted.</li>
<li><code>ratelimit-remaining</code>: The number of requests remaining in the current window.</li>
<li><code>ratelimit-reset</code>: The time at which the rate limit window resets (in UTC epoch seconds).</li>
</ul>
<p>Your engineering team can write a single, standardized exponential backoff wrapper that reads these headers, completely decoupled from upstream quirks:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #7F848E; font-style: italic;">// Example: exponential backoff using normalized headers</span></span>
<span><span style="color: #C678DD;">async</span><span style="color: #C678DD;"> function</span><span style="color: #61AFEF;"> fetchWithBackoff</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75; font-style: italic;">url</span><span style="color: #ABB2BF;">: </span><span style="color: #E5C07B;">string</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75; font-style: italic;">attempt</span><span style="color: #56B6C2;"> =</span><span style="color: #D19A66;"> 0</span><span style="color: #ABB2BF;">): </span><span style="color: #E5C07B;">Promise</span><span style="color: #ABB2BF;">&#x3c;</span><span style="color: #E5C07B;">Response</span><span style="color: #ABB2BF;">> {</span></span>
<span><span style="color: #C678DD;">  const</span><span style="color: #E5C07B;"> res</span><span style="color: #56B6C2;"> =</span><span style="color: #C678DD;"> await</span><span style="color: #61AFEF;"> fetch</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75;">url</span><span style="color: #ABB2BF;">, { </span><span style="color: #E06C75;">headers</span><span style="color: #ABB2BF;">: { </span><span style="color: #98C379;">'x-api-key'</span><span style="color: #ABB2BF;">: </span><span style="color: #E5C07B;">process</span><span style="color: #ABB2BF;">.</span><span style="color: #E5C07B;">env</span><span style="color: #ABB2BF;">.</span><span style="color: #E06C75;">API_KEY</span><span style="color: #56B6C2;">!</span><span style="color: #ABB2BF;"> } })</span></span>
<span><span style="color: #C678DD;">  if</span><span style="color: #ABB2BF;"> (</span><span style="color: #E5C07B;">res</span><span style="color: #ABB2BF;">.</span><span style="color: #E06C75;">status</span><span style="color: #56B6C2;"> !==</span><span style="color: #D19A66;"> 429</span><span style="color: #ABB2BF;">) </span><span style="color: #C678DD;">return</span><span style="color: #E06C75;"> res</span></span>
<span> </span>
<span><span style="color: #C678DD;">  const</span><span style="color: #E5C07B;"> reset</span><span style="color: #56B6C2;"> =</span><span style="color: #61AFEF;"> Number</span><span style="color: #ABB2BF;">(</span><span style="color: #E5C07B;">res</span><span style="color: #ABB2BF;">.</span><span style="color: #E5C07B;">headers</span><span style="color: #ABB2BF;">.</span><span style="color: #61AFEF;">get</span><span style="color: #ABB2BF;">(</span><span style="color: #98C379;">'ratelimit-reset'</span><span style="color: #ABB2BF;">) </span><span style="color: #56B6C2;">??</span><span style="color: #D19A66;"> 30</span><span style="color: #ABB2BF;">)</span></span>
<span><span style="color: #C678DD;">  const</span><span style="color: #E5C07B;"> jitter</span><span style="color: #56B6C2;"> =</span><span style="color: #E5C07B;"> Math</span><span style="color: #ABB2BF;">.</span><span style="color: #61AFEF;">random</span><span style="color: #ABB2BF;">() </span><span style="color: #56B6C2;">*</span><span style="color: #D19A66;"> 1000</span></span>
<span><span style="color: #C678DD;">  const</span><span style="color: #E5C07B;"> delay</span><span style="color: #56B6C2;"> =</span><span style="color: #E5C07B;"> Math</span><span style="color: #ABB2BF;">.</span><span style="color: #61AFEF;">min</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75;">reset</span><span style="color: #56B6C2;"> *</span><span style="color: #D19A66;"> 1000</span><span style="color: #ABB2BF;">, </span><span style="color: #D19A66;">2</span><span style="color: #56B6C2;"> **</span><span style="color: #E06C75;"> attempt</span><span style="color: #56B6C2;"> *</span><span style="color: #D19A66;"> 1000</span><span style="color: #ABB2BF;">) </span><span style="color: #56B6C2;">+</span><span style="color: #E06C75;"> jitter</span></span>
<span> </span>
<span><span style="color: #E5C07B;">  console</span><span style="color: #ABB2BF;">.</span><span style="color: #61AFEF;">log</span><span style="color: #ABB2BF;">(</span><span style="color: #98C379;">`Rate limited. Sleeping for </span><span style="color: #C678DD;">${</span><span style="color: #E06C75;">delay</span><span style="color: #C678DD;">}</span><span style="color: #98C379;">ms...`</span><span style="color: #ABB2BF;">);</span></span>
<span><span style="color: #C678DD;">  if</span><span style="color: #ABB2BF;"> (</span><span style="color: #E06C75;">attempt</span><span style="color: #56B6C2;"> >=</span><span style="color: #D19A66;"> 5</span><span style="color: #ABB2BF;">) </span><span style="color: #C678DD;">throw</span><span style="color: #C678DD;"> new</span><span style="color: #61AFEF;"> Error</span><span style="color: #ABB2BF;">(</span><span style="color: #98C379;">'Rate limit retries exhausted'</span><span style="color: #ABB2BF;">)</span></span>
<span><span style="color: #ABB2BF;">  </span></span>
<span><span style="color: #C678DD;">  await</span><span style="color: #C678DD;"> new</span><span style="color: #E5C07B;"> Promise</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75; font-style: italic;">r</span><span style="color: #C678DD;"> =></span><span style="color: #61AFEF;"> setTimeout</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75;">r</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">delay</span><span style="color: #ABB2BF;">))</span></span>
<span><span style="color: #C678DD;">  return</span><span style="color: #61AFEF;"> fetchWithBackoff</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75;">url</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">attempt</span><span style="color: #56B6C2;"> +</span><span style="color: #D19A66;"> 1</span><span style="color: #ABB2BF;">)</span></span>
<span><span style="color: #ABB2BF;">}</span></span></code></pre></figure>
<h3 id="3-build-an-audit-ready-it-graph-not-snapshots"><a class="anchor-link" href="#3-build-an-audit-ready-it-graph-not-snapshots">3. Build an Audit-Ready IT Graph, Not Snapshots</a></h3>
<p>A quarterly CSV export is a snapshot. An audit-ready system is a graph: every user, role assignment, group membership, and license seat must be timestamped with a history of changes.</p>
<p>For each connected account, schedule a recurring sync of <code>Users</code>, <code>Groups</code>, <code>Roles</code>, and <code>RoleAssignments</code>. Diff each run against the previous to capture additions, removals, and role changes. That diff is the immutable evidence package your auditor wants. Pair this with the unified <code>Activities</code> endpoint for login events - if a user has not logged in for 90 days, that is an access reviewer's first flag.</p>
<h3 id="4-handling-custom-permission-models-via-overrides"><a class="anchor-link" href="#4-handling-custom-permission-models-via-overrides">4. Handling Custom Permission Models via Overrides</a></h3>
<p>Enterprise customers rarely use default permission models. Salesforce orgs have custom profiles. GitHub Enterprise has custom org-level roles. NetSuite has subsidiary-scoped permissions. If your unified API relies on rigid, hardcoded data models, it will drop this custom data, lying to your customers during an audit.</p>
<p>The architectural fix is a multi-level override hierarchy. If a specific customer uses a bespoke security field to determine administrative access, you can apply an account-level JSONata override to map that custom field directly into the unified <code>RoleAssignments</code> array. This customization happens entirely in configuration, meaning you can support bespoke enterprise permission models without touching your core platform code.</p>
<h3 id="5-be-honest-about-the-trade-offs"><a class="anchor-link" href="#5-be-honest-about-the-trade-offs">5. Be Honest About the Trade-Offs</a></h3>
<p>A unified API is not a magic bullet. Be aware of the trade-offs:</p>
<ul>
<li><strong>Coverage of esoteric features:</strong> If your access review needs a highly vendor-specific attribute, you will need to lean on a proxy/passthrough API or per-account overrides.</li>
<li><strong>Webhook fidelity varies:</strong> Some vendors emit excellent lifecycle webhooks; others emit none. For non-webhook providers, you are polling - plan your sync cadence accordingly.</li>
<li><strong>Roadmap coupling:</strong> Ensure your unified API provider supports data-only addition of new connectors so you are not blocked behind their sprint queue when a customer demands a niche integration.</li>
</ul>
<h2 id="stop-chasing-csvs-and-start-shipping"><a class="anchor-link" href="#stop-chasing-csvs-and-start-shipping">Stop Chasing CSVs and Start Shipping</a></h2>
<p>Automating user access reviews across unmanaged SaaS applications is a massive technical challenge, but it is also a massive competitive advantage. The explosive growth of SaaS, the surge in Shadow IT, and the rapid adoption of AI have created a tsunami of risks. Customers are desperate to replace manual CSV exports with continuous, automated compliance monitoring.</p>
<p>Attempting to build and maintain the necessary API integrations in-house will consume your engineering roadmap. By leveraging a declarative Unified Directory API, you abstract away the authentication chaos, pagination differences, and schema fragmentation of the SaaS ecosystem.</p>
<p>You stop maintaining auth flows and start shipping the actual product: review campaigns, automated reviewer assignment, evidence collection, and anomaly detection.</p>
<aside class="cta cta-row"><p>Building a GRC, SSPM, or identity product and need to ship user access review integrations across 100+ SaaS apps? Talk to our team about how a Unified User Directory API handles custom role models and shadow IT without code.</p><a class="cta-button" href="https://cal.com/truto/partner-with-truto">Talk to us</a></aside>
