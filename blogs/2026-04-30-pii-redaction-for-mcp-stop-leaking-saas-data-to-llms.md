---
title: "PII Redaction for MCP: Stop Leaking SaaS Data to LLMs"
url: "https://truto.one/blog/how-to-implement-pii-redaction-when-passing-saas-data-to-llms-via-mcp/"
date: "Thu, 30 Apr 2026 00:00:00 GMT"
author: "yuvraj@truto.one (Yuvraj Muley)"
feed_url: "https://truto.one/blog/feed.xml"
---
<p>If you are wiring AI agents into Salesforce, Workday, or Jira through the <a href="/what-is-an-mcp-server-the-2026-architecture-guide-for-saas-pms/">Model Context Protocol</a>, every tool call is a potential data breach.</p>
<p>Exposing your B2B SaaS application to AI models used to require building custom, point-to-point API connectors for every single LLM provider. The Model Context Protocol (MCP) changed that architecture entirely. By acting as a universal standard for tool calling, MCP collapses the N x M integration problem into a simple N + M hub-and-spoke model. You build one MCP server, and your product instantly works with Claude, ChatGPT, Cursor, and custom LangChain agents.</p>
<p>But this architectural shift introduces a massive security vulnerability. When an AI agent invokes an MCP tool like <code>list_all_workday_employees</code>, the default behavior of most integrations is to return the entire JSON payload from the upstream API. The MCP server pulls a record from the upstream SaaS API, hands the raw JSON to the LLM, and that payload now lives inside a third-party model provider's context window. Social Security numbers, unhashed passwords, salary data, customer emails, support ticket bodies with PHI—all of it.</p>
<p>PII redaction at the MCP boundary is the only architectural pattern that lets you ship AI agents without expanding your SOC 2 scope to include every model provider your customers prefer. This guide breaks down the architectural patterns for masking sensitive data before it reaches the LLM, the trade-offs between static redaction and context-aware tokenization, and why a zero data retention proxy layer is the only defensible way to handle third-party SaaS data.</p>
<h2 id="the-ai-agent-security-gap-why-pii-redaction-is-mandatory-for-mcp"><a class="anchor-link" href="#the-ai-agent-security-gap-why-pii-redaction-is-mandatory-for-mcp">The AI Agent Security Gap: Why PII Redaction is Mandatory for MCP</a></h2>
<p>When AI agents use MCP to query SaaS APIs, they inadvertently pull raw Personally Identifiable Information (PII) into LLM context windows. Engineering teams often underestimate the blast radius of this data exposure until they fail an InfoSec audit.</p>
<p>The regulatory and security ground has shifted hard in the last twelve months. <cite>Fifty percent of all enterprise cybersecurity incident response efforts will focus on incidents involving custom-built AI-driven applications by 2028, according to Gartner.</cite> <cite>As Christopher Mixter, VP Analyst at Gartner, put it: "AI is evolving quickly, yet many tools - especially custom-built AI applications - are being deployed before they're fully tested. These systems are complex, dynamic and difficult to secure over time. Most security teams still lack clear processes for handling AI-related incidents."</cite></p>
<p>The compliance side is just as unforgiving. <cite>Through 2027, manual AI compliance processes will expose 75% of regulated organizations to fines exceeding 5% of their global revenue.</cite> For a $200M ARR SaaS, that is a $10M ceiling on a single bad audit. The core issue is that these systems are highly dynamic; an agent might decide to pull an invoice from QuickBooks today and an employee record from BambooHR tomorrow.</p>
<p>The risk of LLM data ingestion is not theoretical. <cite>Researchers at Truffle Security trawled through the December 2024 Common Crawl archive, consisting of 400TB of web data gathered from 2.67 billion web pages, and found 11,908 live secrets using their open source secret scanner, TruffleHog.</cite> Similarly, security researchers at Lasso Security recently analyzed an open-source training dataset used for LLM development on Hugging Face and found nearly 12,000 live API keys, passwords, and credentials exposed in the clear.</p>
<p>If credentials leak that easily into LLM training data, expecting your customers' PII to stay inside a model's context window is wishful thinking. When you send customer data to an LLM provider, you lose control over where that data is cached, how it is logged, and whether it will be used to train future foundation models.</p>
<div class="callout callout-warning"><span class="callout-title">
<svg class="lucide lucide-triangle-alert" fill="none" height="18" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" width="18" xmlns="http://www.w3.org/2000/svg">
  <path d="m21.73 18-8-14a2 2 0 0 0-3.48 0l-8 14A2 2 0 0 0 4 21h16a2 2 0 0 0 1.73-3">
  <path d="M12 9v4">
  <path d="M12 17h.01">
</svg>
Warning</span><p>Once data hits a third-party LLM, you cannot un-send it. Provider data retention policies, log analysis pipelines, and abuse-monitoring queues all become sub-processors of your customers' regulated data. Redaction has to happen <em>before</em> the tool call returns to the model.</p></div>
<h2 id="what-counts-as-sensitive-data-in-a-saas-api-response"><a class="anchor-link" href="#what-counts-as-sensitive-data-in-a-saas-api-response">What Counts as Sensitive Data in a SaaS API Response</a></h2>
<p>Most engineering teams underestimate the surface area of sensitive data. A typical <code>GET /contacts</code> response from HubSpot or Salesforce isn't just a name and email. It often contains a sprawling JSON tree of liabilities:</p>
<ul>
<li><strong>Direct identifiers</strong>: email, phone, full name, SSN, tax ID, employee ID.</li>
<li><strong>Quasi-identifiers</strong>: date of birth, zip code, IP address, device ID (re-identifiable when combined).</li>
<li><strong>Financial</strong>: bank routing, card last-four, salary, compensation history (Workday, BambooHR).</li>
<li><strong>Health</strong>: anything in a Zendesk or Intercom ticket from a healthcare customer (instant PHI).</li>
<li><strong>Secrets</strong>: webhook URLs, API tokens, OAuth refresh tokens stored in custom fields.</li>
<li><strong>Free-text fields</strong>: notes, descriptions, comments—the worst offender, because regex misses unstructured PII.</li>
</ul>
<p>A helpdesk integration is the canonical worst case. A user types their credit card into a support ticket. Your AI agent calls <code>list_zendesk_tickets</code>, the LLM ingests the body, and now Anthropic or OpenAI has logged a PCI violation in their abuse pipeline. If your architecture allows an LLM to request a resource and directly receive the raw SaaS API response, you are flying blind. You need an interception layer.</p>
<h2 id="how-pii-redaction-works-in-an-mcp-architecture"><a class="anchor-link" href="#how-pii-redaction-works-in-an-mcp-architecture">How PII Redaction Works in an MCP Architecture</a></h2>
<p>To understand where to place your redaction logic, you need to understand the MCP JSON-RPC lifecycle. When an MCP client (like Claude Desktop) connects to an MCP server, all communication happens over HTTP POST using JSON-RPC 2.0 messages.</p>
<p>The pattern is simple in concept: insert a redaction step between the upstream SaaS response and the JSON-RPC <code>tools/call</code> reply that the MCP client receives. When the LLM decides to use a tool, it sends a <code>tools/call</code> request. The arguments arrive as a single flat object. The MCP server executes the API request against the third-party SaaS platform, receives the response, and wraps it in an MCP-compliant result object.</p>
<p>PII redaction must happen exactly between the moment the SaaS API returns the data and the moment the MCP server constructs the JSON-RPC result.</p>
<p>Here is the architectural flow for a secure MCP proxy layer:</p>
<figure class="mermaid-container"><pre class="mermaid">sequenceDiagram
    participant LLM as AI Agent (MCP Client)
    participant Gateway as MCP Proxy Gateway
    participant Redact as Redaction Layer
    participant SaaS as Upstream SaaS API

    LLM-&gt;&gt;Gateway: JSON-RPC tools/call&lt;br&gt;{"name": "get_employee", "arguments": {"id": "123"}}
    Gateway-&gt;&gt;SaaS: GET /api/v1/employees/123&lt;br&gt;Authorization: Bearer token
    SaaS--&gt;&gt;Gateway: HTTP 200 OK (Raw JSON with PII)
    Gateway-&gt;&gt;Redact: Scan + transform payload
    Redact--&gt;&gt;Gateway: Sanitized payload
    Gateway--&gt;&gt;LLM: JSON-RPC Response (PII-free)</pre></figure>
<p>If you inspect the raw payload from the upstream SaaS API, it might look like this:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #ABB2BF;">{</span></span>
<span><span style="color: #E06C75;">  "id"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"emp_89324"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">  "first_name"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"Jane"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">  "last_name"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"Doe"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">  "email"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"jane.doe@enterprise.com"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">  "social_security_number"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"999-99-9999"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">  "base_salary"</span><span style="color: #ABB2BF;">: </span><span style="color: #D19A66;">125000</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">  "department"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"Engineering"</span></span>
<span><span style="color: #ABB2BF;">}</span></span></code></pre></figure>
<p>Your proxy layer must intercept this payload, apply a masking policy, and return the sanitized version to the LLM inside the standard MCP content array:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #ABB2BF;">{</span></span>
<span><span style="color: #E06C75;">  "jsonrpc"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"2.0"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">  "id"</span><span style="color: #ABB2BF;">: </span><span style="color: #D19A66;">42</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">  "result"</span><span style="color: #ABB2BF;">: {</span></span>
<span><span style="color: #E06C75;">    "content"</span><span style="color: #ABB2BF;">: [{</span></span>
<span><span style="color: #E06C75;">      "type"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"text"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">      "text"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"{</span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">id</span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">: </span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">emp_89324</span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">, </span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">first_name</span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">: </span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">Jane</span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">, </span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">department</span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">: </span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">Engineering</span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">, </span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">email</span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">: </span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">&#x3c;EMAIL_1></span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">, </span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">social_security_number</span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">: </span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">[REDACTED]</span><span style="color: #56B6C2;">\"</span><span style="color: #98C379;">}"</span></span>
<span><span style="color: #ABB2BF;">    }]</span></span>
<span><span style="color: #ABB2BF;">  }</span></span>
<span><span style="color: #ABB2BF;">}</span></span></code></pre></figure>
<p>This interception guarantees that the LLM only ever sees the data it strictly needs to perform its reasoning. A robust redaction layer must maintain three non-negotiable properties:</p>
<ol>
<li><strong>Pre-model inspection.</strong> <cite>Gateways analyze content before it reaches the model, not after. Once data hits an LLM, it's already exposed.</cite></li>
<li><strong>Deterministic transformation.</strong> Redaction must produce the same output for the same input across requests, otherwise pagination cursors and follow-up tool calls get confused.</li>
<li><strong>Reversibility for write-back.</strong> If the agent later calls <code>update_contact</code>, your gateway needs a way to swap masked tokens back to real values, or the agent will overwrite a real email with <code>&#x3c;EMAIL_1></code>.</li>
</ol>
<p>The redaction layer typically combines three detection strategies:</p>
<table>
<thead>
<tr>
<th>Strategy</th>
<th>Catches</th>
<th>Misses</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Regex / Luhn / IBAN validators</strong></td>
<td>SSNs, credit cards, IBANs, structured IDs</td>
<td>Names, addresses, contextual PII</td>
</tr>
<tr>
<td><strong>Named-entity recognition (NER)</strong></td>
<td>Person names, locations, organizations in free text</td>
<td>Domain-specific identifiers (employee IDs, custom field PHI)</td>
</tr>
<tr>
<td><strong>Schema-driven field rules</strong></td>
<td>Known sensitive fields by JSON path</td>
<td>New fields added by the SaaS vendor</td>
</tr>
</tbody>
</table>
<p>Production systems run all three. Microsoft Presidio combined with spaCy and a YAML rule file is the open-source baseline. <cite>OpenAI's Privacy Filter is a small model with frontier personal data detection capability, designed for high-throughput privacy workflows, able to perform context-aware detection of PII in unstructured text</cite> and is a viable drop-in for the NER step.</p>
<h2 id="techniques-for-masking-saas-data-static-redaction-vs-context-aware-tokenization"><a class="anchor-link" href="#techniques-for-masking-saas-data-static-redaction-vs-context-aware-tokenization">Techniques for Masking SaaS Data: Static Redaction vs. Context-Aware Tokenization</a></h2>
<p>Once you have the interception layer in place, you must choose how to alter the data. This is where most implementations fail. Replacing every sensitive field with <code>[REDACTED]</code> is the lazy answer, and it actively kills agent reasoning.</p>
<h3 id="static-redaction-regex-and-declarative-mapping"><a class="anchor-link" href="#static-redaction-regex-and-declarative-mapping">Static Redaction (Regex and Declarative Mapping)</a></h3>
<p>Static redaction involves identifying sensitive fields by key name or regex pattern and replacing their values with a static string like <code>[REDACTED]</code>, <code>null</code>, or <code>***</code>. This approach is fast, deterministic, and easy to audit.</p>
<p>However, naive masking breaks LLM context. As noted by AI security firm Protecto AI, removing data entirely degrades AI accuracy. Consider this Salesforce contact list:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #ABB2BF;">[</span></span>
<span><span style="color: #ABB2BF;">  { </span><span style="color: #E06C75;">"id"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"003"</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">"email"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"jane@acme.com"</span><span style="color: #ABB2BF;">,  </span><span style="color: #E06C75;">"company"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"Acme"</span><span style="color: #ABB2BF;"> },</span></span>
<span><span style="color: #ABB2BF;">  { </span><span style="color: #E06C75;">"id"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"004"</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">"email"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"jane@acme.com"</span><span style="color: #ABB2BF;">,  </span><span style="color: #E06C75;">"company"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"Acme"</span><span style="color: #ABB2BF;"> },</span></span>
<span><span style="color: #ABB2BF;">  { </span><span style="color: #E06C75;">"id"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"005"</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">"email"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"bob@globex.com"</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">"company"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"Globex"</span><span style="color: #ABB2BF;"> }</span></span>
<span><span style="color: #ABB2BF;">]</span></span></code></pre></figure>
<p>If you use static redaction, every email turns into <code>[REDACTED]</code>. The agent can no longer answer "which contacts are duplicates?" because the values are identical strings. Worse, when the user says "send a follow-up to Jane," the agent has nothing to reference. If an LLM is trying to correlate support tickets submitted by the same user, but every email address has been replaced with the exact same static string, the model loses the ability to group those tickets.</p>
<h3 id="context-aware-tokenization"><a class="anchor-link" href="#context-aware-tokenization">Context-Aware Tokenization</a></h3>
<p>To solve the context degradation problem, enterprise architectures use context-aware tokenization or synthetic data replacement. Enterprise gateways replace actual values with semantically meaningful, format-preserving tokens.</p>
<p>Instead of wiping out the data, the tokenization engine replaces it with a synthetic but consistent value:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #ABB2BF;">[</span></span>
<span><span style="color: #ABB2BF;">  { </span><span style="color: #E06C75;">"id"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"003"</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">"email"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"&#x3c;EMAIL_1>"</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">"company"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"&#x3c;COMPANY_1>"</span><span style="color: #ABB2BF;"> },</span></span>
<span><span style="color: #ABB2BF;">  { </span><span style="color: #E06C75;">"id"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"004"</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">"email"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"&#x3c;EMAIL_1>"</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">"company"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"&#x3c;COMPANY_1>"</span><span style="color: #ABB2BF;"> },</span></span>
<span><span style="color: #ABB2BF;">  { </span><span style="color: #E06C75;">"id"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"005"</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">"email"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"&#x3c;EMAIL_2>"</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">"company"</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"&#x3c;COMPANY_2>"</span><span style="color: #ABB2BF;"> }</span></span>
<span><span style="color: #ABB2BF;">]</span></span></code></pre></figure>
<p>If the LLM sees <code>&#x3c;EMAIL_1></code> across five different Jira tickets, it can correctly reason that the same user submitted all five tickets. It can generate a summary report based on that correlation.</p>
<p>When the LLM outputs its final response or attempts to execute a write operation (like <code>update_jira_ticket</code>), the proxy layer intercepts the outbound request, detokenizes <code>&#x3c;EMAIL_1></code> back to <code>jane@acme.com</code>, and forwards the request to the SaaS API. This is the same pattern <cite>LiteLLM uses with Presidio: for 'replace' operations, the gateway can check the LLM response and replace the masked token with the user-submitted values</cite>.</p>
<p>To implement this safely, you must keep a short-lived, in-memory token vault keyed to the conversation or session. Never persist these mappings to disk. If you write the token map to a database, you have just rebuilt the exact data retention problem you were trying to solve.</p>
<h2 id="implementing-a-centralized-pii-gateway-for-ai-agents"><a class="anchor-link" href="#implementing-a-centralized-pii-gateway-for-ai-agents">Implementing a Centralized PII Gateway for AI Agents</a></h2>
<p>Security policies fail when they are decentralized. The biggest mistake teams make is putting redaction logic inside the AI agent's internal prompt logic or a client-side library. Every new agent re-implements the rules, drifts, and eventually leaks, a problem that compounds quickly in <a href="/handling-auth-tool-sharing-in-multi-agent-frameworks-via-mcp/">multi-agent systems</a>. If an engineer updates the agent's system prompt and accidentally removes the instruction to ignore SSNs, the data leaks immediately. <cite>Without a centralized control layer, every application must implement its own filtering logic for data privacy. That approach leads to inconsistent security and compliance gaps.</cite></p>
<p>PII redaction must happen at a centralized gateway or proxy layer. The MCP server already holds the privileged credentials, possesses the schema knowledge, and sits at the perfect network position. It is the natural enforcement point. A centralized gateway provides several architectural advantages:</p>
<h3 id="1-schema-driven-field-masking-with-jsonata"><a class="anchor-link" href="#1-schema-driven-field-masking-with-jsonata">1. Schema-Driven Field Masking with JSONata</a></h3>
<p>For structured fields, declarative transforms beat imperative code. A response transformation expressed as JSONata is auditable, reviewable in a pull request, and trivially diffable for compliance teams. Instead of writing custom Python or Node.js code for every single integration, you define a JSONata expression that maps the raw payload to a sanitized schema.</p>
<p>Here is an example JSONata expression that strips and tokenizes sensitive HR data from a raw Workday API response:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span>(</span></span>
<span><span>  $maskLastName := function($str) { $substring($str, 0, 1) &#x26; "***" };</span></span>
<span><span>  </span></span>
<span><span>  $map($$ , function($v) {</span></span>
<span><span>    {</span></span>
<span><span>      "id": $v.id,</span></span>
<span><span>      "first_name": $v.first_name,</span></span>
<span><span>      "last_name": $maskLastName($v.last_name),</span></span>
<span><span>      "email": "&#x3c;EMAIL_" &#x26; $hash($v.email) &#x26; ">",</span></span>
<span><span>      "social_security_number": null,</span></span>
<span><span>      "compensation": $exists($v.base_salary) ? "&#x3c;REDACTED>" : null,</span></span>
<span><span>      "manager_id": $v.manager_id,</span></span>
<span><span>      "department": $v.department</span></span>
<span><span>    }</span></span>
<span><span>  })</span></span>
<span><span>)</span></span></code></pre></figure>
<p>This pattern—declarative response mapping with a transformation language—is exactly how Truto already handles unified API normalization. The same hooks that map <code>first_name</code> to a unified <code>firstName</code> field can drop, hash, or tokenize the value. See our <a href="/developer-guide-mapping-api-data-with-jsonata-code-samples/">JSONata mapping guide</a> for the broader pattern.</p>
<h3 id="2-free-text-scanning-for-tickets-and-notes"><a class="anchor-link" href="#2-free-text-scanning-for-tickets-and-notes">2. Free-Text Scanning for Tickets and Notes</a></h3>
<p>Structured field rules don't help when the SaaS payload contains a raw Zendesk ticket body or a Salesforce note. Run free-text fields through a detector before serializing:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #C678DD;">import</span><span style="color: #ABB2BF;"> { </span><span style="color: #E06C75;">AnalyzerEngine</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">AnonymizerEngine</span><span style="color: #ABB2BF;"> } </span><span style="color: #C678DD;">from</span><span style="color: #98C379;"> "presidio-client"</span><span style="color: #ABB2BF;">;</span></span>
<span> </span>
<span><span style="color: #C678DD;">async</span><span style="color: #C678DD;"> function</span><span style="color: #61AFEF;"> sanitizeFreeText</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75; font-style: italic;">payload</span><span style="color: #ABB2BF;">: </span><span style="color: #E5C07B;">any</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75; font-style: italic;">fields</span><span style="color: #ABB2BF;">: </span><span style="color: #E5C07B;">string</span><span style="color: #ABB2BF;">[]) {</span></span>
<span><span style="color: #C678DD;">  for</span><span style="color: #ABB2BF;"> (</span><span style="color: #C678DD;">const</span><span style="color: #E5C07B;"> path</span><span style="color: #C678DD;"> of</span><span style="color: #E06C75;"> fields</span><span style="color: #ABB2BF;">) {</span></span>
<span><span style="color: #C678DD;">    const</span><span style="color: #E5C07B;"> text</span><span style="color: #56B6C2;"> =</span><span style="color: #61AFEF;"> get</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75;">payload</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">path</span><span style="color: #ABB2BF;">);</span></span>
<span><span style="color: #C678DD;">    if</span><span style="color: #ABB2BF;"> (</span><span style="color: #56B6C2;">!</span><span style="color: #E06C75;">text</span><span style="color: #ABB2BF;">) </span><span style="color: #C678DD;">continue</span><span style="color: #ABB2BF;">;</span></span>
<span><span style="color: #C678DD;">    const</span><span style="color: #E5C07B;"> findings</span><span style="color: #56B6C2;"> =</span><span style="color: #C678DD;"> await</span><span style="color: #E5C07B;"> analyzer</span><span style="color: #ABB2BF;">.</span><span style="color: #61AFEF;">analyze</span><span style="color: #ABB2BF;">({</span></span>
<span><span style="color: #E06C75;">      text</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">      language</span><span style="color: #ABB2BF;">: </span><span style="color: #98C379;">"en"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #E06C75;">      entities</span><span style="color: #ABB2BF;">: [</span><span style="color: #98C379;">"PERSON"</span><span style="color: #ABB2BF;">, </span><span style="color: #98C379;">"EMAIL_ADDRESS"</span><span style="color: #ABB2BF;">, </span><span style="color: #98C379;">"PHONE_NUMBER"</span><span style="color: #ABB2BF;">,</span></span>
<span><span style="color: #98C379;">                 "US_SSN"</span><span style="color: #ABB2BF;">, </span><span style="color: #98C379;">"CREDIT_CARD"</span><span style="color: #ABB2BF;">, </span><span style="color: #98C379;">"IBAN_CODE"</span><span style="color: #ABB2BF;">]</span></span>
<span><span style="color: #ABB2BF;">    });</span></span>
<span><span style="color: #C678DD;">    const</span><span style="color: #E5C07B;"> anonymized</span><span style="color: #56B6C2;"> =</span><span style="color: #C678DD;"> await</span><span style="color: #E5C07B;"> anonymizer</span><span style="color: #ABB2BF;">.</span><span style="color: #61AFEF;">anonymize</span><span style="color: #ABB2BF;">({ </span><span style="color: #E06C75;">text</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">analyzer_results</span><span style="color: #ABB2BF;">: </span><span style="color: #E06C75;">findings</span><span style="color: #ABB2BF;"> });</span></span>
<span><span style="color: #61AFEF;">    set</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75;">payload</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">path</span><span style="color: #ABB2BF;">, </span><span style="color: #E5C07B;">anonymized</span><span style="color: #ABB2BF;">.</span><span style="color: #E06C75;">text</span><span style="color: #ABB2BF;">);</span></span>
<span><span style="color: #ABB2BF;">  }</span></span>
<span><span style="color: #C678DD;">  return</span><span style="color: #E06C75;"> payload</span><span style="color: #ABB2BF;">;</span></span>
<span><span style="color: #ABB2BF;">}</span></span></code></pre></figure>
<p>Run the detector with a confidence threshold appropriate to the field. Ticket bodies in healthcare-adjacent products should err toward over-redaction; product feedback fields can be more permissive.</p>
<h3 id="3-tool-surface-minimization-and-namespace-resolution"><a class="anchor-link" href="#3-tool-surface-minimization-and-namespace-resolution">3. Tool-Surface Minimization and Namespace Resolution</a></h3>
<p>The other half of the gateway story is <em>which tools you expose at all</em>. Instead of exposing every endpoint a SaaS API offers, a gateway can dynamically generate MCP tools based strictly on approved documentation records. If your agent only needs ticket subjects and statuses, do not ship an <code>attachments.download</code> tool. Documentation-driven tool generation forces a deliberate review for every endpoint that touches an LLM. We covered this pattern deeply in <a href="/auto-generated-mcp-tools-for-ai-agents-a-2026-architecture-guide/">Auto-Generated MCP Tools</a>.</p>
<p>Furthermore, when an MCP client calls a tool, all arguments arrive in a single flat JSON object. A centralized gateway uses predefined JSON Schemas to split these arguments into query parameters and body parameters before forwarding them to the proxy API handlers. This prevents prompt injection attacks from smuggling malicious payloads into unexpected HTTP headers or query strings.</p>
<h3 id="4-strict-rate-limit-handling"><a class="anchor-link" href="#4-strict-rate-limit-handling">4. Strict Rate Limit Handling</a></h3>
<p>AI agents are notoriously aggressive when scraping data. A centralized gateway protects your infrastructure by enforcing rate limits. It is a critical architectural requirement that your gateway does not absorb or silently retry upstream rate limit errors. Truto, for example, explicitly passes upstream HTTP 429 errors directly to the caller. Truto normalizes the upstream rate limit information into standardized headers (<code>ratelimit-limit</code>, <code>ratelimit-remaining</code>, <code>ratelimit-reset</code>) per the IETF specification. The AI agent's orchestration layer remains in full control of retry logic and exponential backoff, ensuring predictable system behavior.</p>
<h2 id="zero-data-retention-the-ultimate-defense-for-third-party-saas-data"><a class="anchor-link" href="#zero-data-retention-the-ultimate-defense-for-third-party-saas-data">Zero Data Retention: The Ultimate Defense for Third-Party SaaS Data</a></h2>
<p>Redacting PII in transit is only half the battle. Redaction prevents PII from reaching the <em>LLM</em>. It does not prevent PII from being stored by the <em>MCP infrastructure itself</em>.</p>
<p>If the infrastructure routing your MCP calls stores a copy of the unredacted third-party data on its own disks, you have created a massive compliance liability. If your AI agent connects to Salesforce or BambooHR through a managed MCP server platform, the data flowing through that server is governed by that platform's data retention policy. If the platform caches API responses in a database to speed up subsequent queries, they become a sub-processor of your customers' highly regulated enterprise data.</p>
<p>Your SOC 2 scope immediately expands. Your GDPR obligations multiply. Enterprise InfoSec teams will flag your application during procurement when they see a third-party caching their HR records.</p>
<p>As detailed in our breakdown of <a href="/mcp-server-data-retention-policies-compared-which-platforms-keep-your-data-2026/">MCP Server Data Retention Policies</a>, the major LLM providers explicitly wash their hands of what happens during a tool call. OpenAI's documentation states that data sent to remote MCP servers is subject entirely to the third-party's retention policies.</p>
<p>The only defensible architecture for handling sensitive SaaS data is a pass-through proxy with strictly zero data retention. In a zero data retention architecture, the platform acts entirely as a conduit. It receives the request from the LLM, resolves the OAuth tokens from a secure key-value store, proxies the request to the upstream SaaS API, applies the JSONata redaction transformations in memory, and returns the response to the LLM. The underlying SaaS data is never written to a database table, never stored in a durable state mechanism, and never logged in plain text.</p>
<div class="callout callout-tip"><span class="callout-title">
<svg class="lucide lucide-lightbulb" fill="none" height="18" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" width="18" xmlns="http://www.w3.org/2000/svg">
  <path d="M15 14c.2-1 .7-1.7 1.5-2.5 1-.9 1.5-2.2 1.5-3.5A6 6 0 0 0 6 8c0 1 .2 2.2 1.5 3.5.7.7 1.3 1.5 1.5 2.5">
  <path d="M9 18h6">
  <path d="M10 22h4">
</svg>
Tip</span><p>Ask any MCP platform vendor exactly three questions: (1) Do you store the response body of tool calls? (2) For how long, and where? (3) Is that storage in scope of your SOC 2 report? If they hesitate, walk away.</p></div>
<p>For a deeper dive into this compliance posture, review our guide on <a href="/zero-data-retention-mcp-servers-building-soc-2-gdpr-compliant-ai-agents/">Building SOC 2 &#x26; GDPR Compliant AI Agents</a>.</p>
<h2 id="the-honest-trade-offs-of-gateway-side-redaction"><a class="anchor-link" href="#the-honest-trade-offs-of-gateway-side-redaction">The Honest Trade-offs of Gateway-Side Redaction</a></h2>
<p>This architectural rigor is not free. Senior engineers should plan for these realities when implementing gateway-side redaction:</p>
<ul>
<li><strong>Latency.</strong> NER on a 50KB ticket body adds 50-200ms per call. Caching detectors in-process and running regex first helps, but agent loops will feel slower.</li>
<li><strong>False positives.</strong> A product name that looks like a person's name will get tokenized. Build an allowlist for known-safe terms per integration.</li>
<li><strong>False negatives.</strong> No detector catches everything. Layer schema rules, regex, and NER, and accept that adversarial inputs (creative formatting of SSNs) will sometimes slip through. Assume a 5-10% false-negative rate on free text and design downstream controls to compensate.</li>
<li><strong>Reversibility complexity.</strong> Token-to-value swap on write paths is the single most common source of bugs. Test it explicitly with multi-turn agent traces.</li>
<li><strong>Schema drift.</strong> When Salesforce adds a new custom field your customer uses for SSNs, your schema-rule list will not know about it. Pair redaction with <a href="/how-to-build-per-account-api-mappings-field-discovery-caching-monitoring/">field-discovery and schema-drift monitoring</a>.</li>
</ul>
<h2 id="why-trutos-pass-through-architecture-solves-the-mcp-security-problem"><a class="anchor-link" href="#why-trutos-pass-through-architecture-solves-the-mcp-security-problem">Why Truto's Pass-Through Architecture Solves the MCP Security Problem</a></h2>
<p>Building a centralized, zero-data-retention MCP gateway that handles OAuth token lifecycles, JSONata transformations, NER scanning pipelines, and dynamic tool generation is a massive engineering undertaking. This is exactly the infrastructure Truto provides out of the box.</p>
<p>Truto is a unified API and MCP server platform built around design choices that line up directly with strict InfoSec compliance requirements:</p>
<ul>
<li><strong>Pass-Through by Default (Zero Data Retention):</strong> Truto's proxy API never stores the third-party SaaS data flowing through its MCP servers. The payload is fetched, processed in memory, returned to the caller, and discarded. Your customers' data is not warehoused on our side, eliminating sub-processor compliance risks entirely.</li>
<li><strong>Declarative Transforms via JSONata:</strong> Truto's built-in JSONata transformation layer allows your engineering team to declaratively strip or tokenize sensitive fields from API responses before they ever reach the LLM. Redaction is configuration, not a fork of our code.</li>
<li><strong>Documentation-Driven Tool Exposure:</strong> Truto derives MCP tool definitions dynamically from integration resources and documentation records. An integration's resource only becomes an MCP tool if you explicitly document it, ensuring AI agents cannot access unauthorized endpoints.</li>
<li><strong>Transparent Rate Limiting:</strong> Truto passes upstream HTTP 429 rate limit errors directly to your agent with standardized IETF headers (<code>ratelimit-limit</code>, <code>ratelimit-remaining</code>, <code>ratelimit-reset</code>), ensuring your orchestration layer maintains precise control over execution flow and retry backoff.</li>
</ul>
<p>What Truto does not do: it is not a full DLP product. If you need adaptive NER on free text with custom entity training, pair Truto's transform layer with Presidio, OpenAI's Privacy Filter, or a commercial DLP gateway in front of the MCP endpoint. The pass-through architecture supports that composition cleanly.</p>
<h2 id="where-to-start-this-quarter"><a class="anchor-link" href="#where-to-start-this-quarter">Where to Start This Quarter</a></h2>
<p>Exposing enterprise SaaS data to LLMs requires extreme architectural discipline. You cannot rely on prompt engineering to protect PII, and you cannot route sensitive payloads through platforms that cache your customers' data. If you are bringing AI agents anywhere near customer SaaS data, run this sequence:</p>
<ol>
<li><strong>Inventory the tool surface.</strong> List every MCP tool your agent can call and every field each one returns. This alone catches half the leaks.</li>
<li><strong>Classify fields by sensitivity.</strong> Tag fields as PII, PHI, PCI, quasi-identifier, or safe. Get legal and security to sign off on the matrix.</li>
<li><strong>Pick a redaction strategy per class.</strong> Decide whether to drop, hash, tokenize, or pass through. Document the rule for each field.</li>
<li><strong>Centralize the enforcement at the MCP layer.</strong> Not in the agent. Not in the prompt. At the gateway.</li>
<li><strong>Verify zero data retention upstream.</strong> Read the SOC 2 report of your MCP infrastructure. If raw payloads land on disk anywhere, your redaction is theater.</li>
<li><strong>Test reversibility on write paths.</strong> Send a multi-turn agent trace that reads, then writes, then reads again. Ensure tokens round-trip correctly.</li>
</ol>
<p>PII redaction is not a feature you bolt on the week before an InfoSec review. It is an architectural choice that determines whether your agent product can be sold to a regulated customer at all.</p>
<aside class="cta cta-row"><p>Evaluating MCP server platforms for PII-sensitive workloads? Stop failing InfoSec reviews over third-party data access. Talk to our engineering team about deploying zero-data-retention MCP servers with declarative JSONata transformations for your AI agents today.</p><a class="cta-button" href="https://cal.com/truto/partner-with-truto">Talk to us</a></aside>
