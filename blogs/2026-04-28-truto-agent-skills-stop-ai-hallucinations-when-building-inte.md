---
title: "Truto Agent Skills: Stop AI Hallucinations When Building Integrations"
url: "https://truto.one/blog/truto-agent-skills-stop-ai-hallucinations-when-building-integrations/"
date: "Tue, 28 Apr 2026 00:00:00 GMT"
author: "uday@truto.one (Uday Gajavalli)"
feed_url: "https://truto.one/blog/feed.xml"
---
<p>If you are looking for AI agent skills for integrations, stop trying to out-prompt a generic model. The reliable fix is to give the assistant vendor-specific context before it writes code. Your AI coding assistant has no idea how Truto actually works under the hood. It will confidently generate API calls to endpoints that don't exist, fabricate authentication flows, and invent pagination parameters.</p>
<p>This isn't a bug in the AI - it's a context problem. The model's training data is inherently stale, generic, or both when it comes to specific platforms. <strong>Agent Skills solve this by injecting accurate, structured knowledge directly into your AI's context</strong> so it writes correct integration code on the first try.</p>
<p>Truto Skills packages Truto's unified API, proxy API, CLI, Link SDK, JSONata mapping patterns, and API conventions into a public <code>SKILL.md</code>-compatible repository for Cursor, Claude Code, and other compatible agents. This post covers what Agent Skills are, how to install the official Truto skills, and why a declarative integration platform is the perfect match for AI-assisted development.</p>
<h2 id="the-hallucination-problem-in-api-integrations"><a class="anchor-link" href="#the-hallucination-problem-in-api-integrations">The Hallucination Problem in API Integrations</a></h2>
<p>AI coding tools are now standard kit, not a toy. The 2025 Stack Overflow Developer Survey (n = 49,000+) revealed that AI tool usage in developer workflows has climbed to 80%. However, that same survey found that 66% of developers say their biggest frustration with AI tools is solutions that are "almost right, but not quite," and only 29% of developers trust AI outputs to be completely accurate.</p>
<p>Faster code is easy. Correct integration code is the part that still hurts. A CodeRabbit analysis of 470 open-source pull requests from December 2025 found roughly <strong>1.7x more issues in AI-coauthored pull requests</strong> compared to human-only PRs when the AI lacks proper domain context.</p>
<p><strong>Integration hallucination</strong> is code that looks plausible but is wrong for the actual API contract you are calling. Ask Claude Code or Cursor to "build a Truto integration that syncs HubSpot contacts" without any additional context, and you'll get something that looks right but is wrong in subtle, time-wasting ways.</p>
<p>What this looks like in practice:</p>
<ul>
<li>It guesses that the base URL is <code>https://api.truto.io</code> (it's actually <code>https://api.truto.one</code>).</li>
<li>It invents a <code>/v2/contacts</code> endpoint when the actual path is <code>/unified/crm/contacts</code>.</li>
<li>It writes <code>page</code> or <code>offset</code> loops where the caller should use <code>next_cursor</code>.</li>
<li>It assumes the platform will magically absorb HTTP 429 rate limit errors.</li>
<li>It drops provider-specific write fields that should be passed through the <code>remote_data</code> object.</li>
<li>It mixes Truto Unified API assumptions with raw provider fields.</li>
</ul>
<p>These are not abstract problems. Every SaaS platform has its own quirks: different auth flows, non-standard error codes, unique pagination schemes, and undocumented rate limit behaviors. An AI model trained on generic web data doesn't know that <a href="/blog/architecting-ai-agents-langgraph-langchain-and-the-saas-integration-bottleneck/">Salesforce returns <code>PascalCase</code> field names while HubSpot nests everything under a <code>properties</code> object</a>.</p>
<p>When building integrations manually, engineers spend hours reading vendor documentation to figure out these edge cases. When an AI agent writes the code, it skips the reading phase and goes straight to hallucinating. Prompting harder is not a strategy. When the API rules are specific and changing, long chats without structured context usually just produce more confident nonsense.</p>
<h2 id="what-are-agent-skills-skillmd"><a class="anchor-link" href="#what-are-agent-skills-skillmd">What Are Agent Skills (SKILL.md)?</a></h2>
<p><strong>Agent Skills are portable, version-controlled packages of instructions that teach AI coding assistants how to perform domain-specific tasks correctly.</strong></p>
<p>The format was created by Anthropic in October 2025 and released as an open standard on December 18, 2025. It has since been adopted by Claude Code, Cursor, GitHub Copilot, OpenAI Codex CLI, Databricks, and others. Rather than dumping a 50-page API specification into your prompt window - which burns through your context window and degrades the model's instruction-following capabilities - Agent Skills use <strong>progressive disclosure</strong>.</p>
<p>At its core, a skill is a folder containing a <code>SKILL.md</code> file with YAML frontmatter (name and description) and Markdown instructions:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span>my-skill/</span></span>
<span><span>├── SKILL.md          # Required: metadata + instructions</span></span>
<span><span>├── scripts/          # Optional: executable code</span></span>
<span><span>├── references/       # Optional: documentation</span></span>
<span><span>└── assets/           # Optional: templates, resources</span></span></code></pre></figure>
<p>The three-level loading system keeps token usage incredibly lean:</p>
<ol>
<li><strong>Metadata only</strong> - At startup, the agent reads just the name and description of every installed skill (~50-100 tokens each). This is enough for the AI to know <em>when</em> to activate each skill.</li>
<li><strong>Full instructions</strong> - When the AI decides a skill is relevant to the current task, it loads the complete <code>SKILL.md</code> into context.</li>
<li><strong>Reference files</strong> - Scripts, docs, and templates load only if the instructions explicitly reference them.</li>
</ol>
<p>This means you can install dozens of skills without bloating every prompt. The pattern is catching on fast. Pulumi released Agent Skills to teach AI assistants infrastructure-as-code best practices. Auth0 published skills covering 20+ framework-specific implementations. Databricks provides skills for their apps, notebooks, and ML workflows. The market has already voted: if your product has tricky conventions, a generic coding assistant is not enough.</p>
<h2 id="introducing-truto-skills"><a class="anchor-link" href="#introducing-truto-skills">Introducing Truto Skills</a></h2>
<p><strong>Truto Skills</strong> is Truto's official skills repository for teaching coding assistants how to build against Truto without guessing. The official repository packages everything an AI coding assistant needs to build integrations on Truto correctly.</p>
<p>The repository includes five main capabilities:</p>
<table>
<thead>
<tr>
<th>Skill</th>
<th>What it teaches the model</th>
<th>Common failure it prevents</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong><code>truto</code></strong></td>
<td>Core Truto integration building workflow, making unified and proxy API calls, and setting up sync jobs.</td>
<td>Made-up API surface and wrong product abstractions.</td>
</tr>
<tr>
<td><strong><code>truto-api-conventions</code></strong></td>
<td>Base URL, auth header patterns, URL structure, pagination, and idempotency rules.</td>
<td>Subtle platform misuse that is annoying to debug.</td>
</tr>
<tr>
<td><strong><code>truto-cli</code></strong></td>
<td>Installing, authenticating, and using the Truto CLI for local debugging, resource management, and bulk exports.</td>
<td>Broken shell commands and bad bulk export choices.</td>
</tr>
<tr>
<td><strong><code>truto-jsonata</code></strong></td>
<td>Writing JSONata expressions for Truto config using custom <code>$functions</code> from <code>@truto/truto-jsonata</code>.</td>
<td>Invalid transforms and low-signal mapping code.</td>
</tr>
<tr>
<td><strong><code>truto-link-sdk</code></strong></td>
<td>Embedding the Truto connection flow in your frontend using <code>@truto/truto-link-sdk</code>.</td>
<td>Broken connect flows and UI guesswork.</td>
</tr>
</tbody>
</table>
<p>That coverage matters because Truto is not just one API. It is a unified API platform with 200+ third-party tools and multiple surfaces: Unified APIs, Proxy APIs, Sync Jobs, Webhooks, Custom APIs, MCP servers, CLI workflows, and frontend connection flows. If the assistant only knows one slice, it will keep reaching for the wrong tool.</p>
<div class="callout callout-info"><span class="callout-title">
<svg class="lucide lucide-info" fill="none" height="18" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" width="18" xmlns="http://www.w3.org/2000/svg">
  <circle cx="12" cy="12" r="10">
  <path d="M12 16v-4">
  <path d="M12 8h.01">
</svg>
Info</span><p><strong>Agent Skills vs. Agent Toolsets (MCP Servers)</strong>
Do not confuse Agent Skills with Agent Toolsets. <strong>Agent Skills</strong> teach your AI coding assistant (like Cursor) how to <em>write code</em> for your application. <strong>Agent Toolsets</strong> (using the Model Context Protocol) give your <em>application's AI agents</em> runtime access to execute actions against third-party APIs. If you want your application's chatbot to read a Zendesk ticket, <a href="/blog/connect-affinity-to-ai-agents-sync-contacts-enrich-profiles/">sync Affinity contacts</a>, or <a href="/blog/connect-pylon-to-ai-agents-streamline-helpdesk-ops-data-sync/">automate Pylon support workflows</a>, you want <a href="/blog/introducing-truto-agent-toolsets/">Truto Agent Toolsets</a>. Skills and toolsets solve different problems, and using both together is usually the right move.</p></div>
<h2 id="how-to-install-truto-skills-in-cursor-and-claude-code"><a class="anchor-link" href="#how-to-install-truto-skills-in-cursor-and-claude-code">How to Install Truto Skills in Cursor and Claude Code</a></h2>
<p>Getting your AI assistant equipped with Truto skills takes less than a minute. The fastest path depends on your environment.</p>
<h3 id="claude-code"><a class="anchor-link" href="#claude-code">Claude Code</a></h3>
<p>If you are using Anthropic's Claude Code CLI, add the Truto skills repository as a plugin marketplace and install it directly. Anthropic's plugin docs support adding marketplaces from GitHub repos:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #61AFEF;">/plugin</span><span style="color: #98C379;"> marketplace</span><span style="color: #98C379;"> add</span><span style="color: #98C379;"> trutohq/truto-skills</span></span>
<span><span style="color: #61AFEF;">/plugin</span><span style="color: #98C379;"> install</span><span style="color: #98C379;"> truto@truto-skills</span></span></code></pre></figure>
<p>The skills will be automatically namespaced in your environment (e.g., <code>truto:truto-api-conventions</code>, <code>truto:truto-jsonata</code>) so they do not collide with local project skills.</p>
<h3 id="cursor"><a class="anchor-link" href="#cursor">Cursor</a></h3>
<p>Cursor supports fetching remote rules directly from GitHub repositories. This ensures your AI always has the latest API conventions without you needing to manually update local markdown files.</p>
<ol>
<li>Open <strong>Cursor Settings</strong>.</li>
<li>Navigate to <strong>Rules</strong>.</li>
<li>Click <strong>Add Rule</strong> under the <strong>Project Rules</strong> section.</li>
<li>Select <strong>Remote Rule (GitHub)</strong>.</li>
<li>Enter the repository URL: <code>https://github.com/trutohq/truto-skills</code></li>
</ol>
<p>This automatically pulls in the skills and applies the <code>truto-api</code> rule to your project. Cursor keeps the rules synced with the source repository, so updates are reflected automatically.</p>
<h3 id="any-agent-via-npx-skills"><a class="anchor-link" href="#any-agent-via-npx-skills">Any Agent (via <code>npx skills</code>)</a></h3>
<p>For other compatible AI agents, or if you want a tool-agnostic path, you can use the open-source Skills CLI to install the skills locally into your project directory:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #61AFEF;">npx</span><span style="color: #98C379;"> skills</span><span style="color: #98C379;"> add</span><span style="color: #98C379;"> trutohq/truto-skills</span></span></code></pre></figure>
<div class="callout callout-tip"><span class="callout-title">
<svg class="lucide lucide-lightbulb" fill="none" height="18" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" width="18" xmlns="http://www.w3.org/2000/svg">
  <path d="M15 14c.2-1 .7-1.7 1.5-2.5 1-.9 1.5-2.2 1.5-3.5A6 6 0 0 0 6 8c0 1 .2 2.2 1.5 3.5.7.7 1.3 1.5 1.5 2.5">
  <path d="M9 18h6">
  <path d="M10 22h4">
</svg>
Tip</span><p><strong>Prompting Tip:</strong> A good first prompt after install is not 'build me a Salesforce integration'. Be specific. Tell the model which Truto skill to use, which API surface you want, and which behaviors it must respect. Example: <em>"Use <code>truto:truto-api-conventions</code> and <code>truto:truto-cli</code> to create a Node.js worker that syncs <code>crm/contacts</code>, paginates with <code>next_cursor</code>, and handles upstream rate limits in caller code."</em></p></div>
<h2 id="teaching-the-ai-truto-api-conventions-that-actually-matter"><a class="anchor-link" href="#teaching-the-ai-truto-api-conventions-that-actually-matter">Teaching the AI Truto API Conventions That Actually Matter</a></h2>
<p>Providing an AI agent with API documentation is not just about showing it the endpoints. It is about teaching the AI the operational realities of the platform. The point of the <code>truto-api-conventions</code> skill is simple: kill the expensive, boring mistakes before they show up in review. Here is exactly what the AI learns.</p>
<h3 id="unified-api-vs-proxy-api"><a class="anchor-link" href="#unified-api-vs-proxy-api">Unified API vs Proxy API</a></h3>
<p>Truto gives you two data-plane surfaces. The Unified API transforms provider data into a common schema. The Proxy API returns data in the provider's native format. The skill teaches the AI that if you are building cross-provider business logic, it should default to Unified. If you need provider-only fields or endpoints the common model does not cover, the AI should drop to Proxy APIs (<code>/proxy/{resource}</code>) purposefully, not by accident.</p>
<h3 id="handling-rate-limits-and-retries-caller-managed"><a class="anchor-link" href="#handling-rate-limits-and-retries-caller-managed">Handling Rate Limits and Retries (Caller-Managed)</a></h3>
<p>This is the single most common mistake AI assistants make with Truto. They assume the platform absorbs rate limits and retries automatically behind the scenes. It doesn't.</p>
<p>When an upstream API returns an HTTP 429, Truto passes that error directly to the caller. What Truto <em>does</em> provide is normalized rate limit metadata in standardized headers per the IETF specification (<code>ratelimit-limit</code>, <code>ratelimit-remaining</code>, <code>ratelimit-reset</code>, and <code>Retry-After</code>). Your code is responsible for reading these headers and implementing retry and exponential backoff logic.</p>
<p>With the Truto skill loaded, the AI generates the retry logic correctly, rather than writing a naive <code>while</code> loop that crashes your application:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #C678DD;">async</span><span style="color: #C678DD;"> function</span><span style="color: #61AFEF;"> fetchWithBackoff</span><span style="color: #ABB2BF;">(</span></span>
<span><span style="color: #61AFEF;">  run</span><span style="color: #ABB2BF;">: () </span><span style="color: #C678DD;">=></span><span style="color: #E5C07B;"> Promise</span><span style="color: #ABB2BF;">&#x3c;</span><span style="color: #E5C07B;">Response</span><span style="color: #ABB2BF;">>,</span></span>
<span><span style="color: #E06C75; font-style: italic;">  attempt</span><span style="color: #56B6C2;"> =</span><span style="color: #D19A66;"> 0</span></span>
<span><span style="color: #ABB2BF;">): </span><span style="color: #E5C07B;">Promise</span><span style="color: #ABB2BF;">&#x3c;</span><span style="color: #E5C07B;">Response</span><span style="color: #ABB2BF;">> {</span></span>
<span><span style="color: #C678DD;">  const</span><span style="color: #E5C07B;"> res</span><span style="color: #56B6C2;"> =</span><span style="color: #C678DD;"> await</span><span style="color: #61AFEF;"> run</span><span style="color: #ABB2BF;">();</span></span>
<span> </span>
<span><span style="color: #C678DD;">  if</span><span style="color: #ABB2BF;"> (</span><span style="color: #E5C07B;">res</span><span style="color: #ABB2BF;">.</span><span style="color: #E06C75;">status</span><span style="color: #56B6C2;"> !==</span><span style="color: #D19A66;"> 429</span><span style="color: #ABB2BF;">) {</span></span>
<span><span style="color: #C678DD;">    return</span><span style="color: #E06C75;"> res</span><span style="color: #ABB2BF;">;</span></span>
<span><span style="color: #ABB2BF;">  }</span></span>
<span> </span>
<span><span style="color: #C678DD;">  const</span><span style="color: #E5C07B;"> retryAfter</span><span style="color: #56B6C2;"> =</span><span style="color: #61AFEF;"> Number</span><span style="color: #ABB2BF;">(</span><span style="color: #E5C07B;">res</span><span style="color: #ABB2BF;">.</span><span style="color: #E5C07B;">headers</span><span style="color: #ABB2BF;">.</span><span style="color: #61AFEF;">get</span><span style="color: #ABB2BF;">(</span><span style="color: #98C379;">'Retry-After'</span><span style="color: #ABB2BF;">) </span><span style="color: #56B6C2;">||</span><span style="color: #D19A66;"> 0</span><span style="color: #ABB2BF;">);</span></span>
<span><span style="color: #C678DD;">  const</span><span style="color: #E5C07B;"> reset</span><span style="color: #56B6C2;"> =</span><span style="color: #61AFEF;"> Number</span><span style="color: #ABB2BF;">(</span><span style="color: #E5C07B;">res</span><span style="color: #ABB2BF;">.</span><span style="color: #E5C07B;">headers</span><span style="color: #ABB2BF;">.</span><span style="color: #61AFEF;">get</span><span style="color: #ABB2BF;">(</span><span style="color: #98C379;">'ratelimit-reset'</span><span style="color: #ABB2BF;">) </span><span style="color: #56B6C2;">||</span><span style="color: #D19A66;"> 0</span><span style="color: #ABB2BF;">);</span></span>
<span><span style="color: #C678DD;">  const</span><span style="color: #E5C07B;"> waitSeconds</span><span style="color: #56B6C2;"> =</span><span style="color: #E5C07B;"> Math</span><span style="color: #ABB2BF;">.</span><span style="color: #61AFEF;">max</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75;">retryAfter</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">reset</span><span style="color: #ABB2BF;">, </span><span style="color: #E5C07B;">Math</span><span style="color: #ABB2BF;">.</span><span style="color: #61AFEF;">min</span><span style="color: #ABB2BF;">(</span><span style="color: #D19A66;">2</span><span style="color: #56B6C2;"> **</span><span style="color: #E06C75;"> attempt</span><span style="color: #ABB2BF;">, </span><span style="color: #D19A66;">60</span><span style="color: #ABB2BF;">));</span></span>
<span> </span>
<span><span style="color: #C678DD;">  await</span><span style="color: #C678DD;"> new</span><span style="color: #E5C07B;"> Promise</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75; font-style: italic;">resolve</span><span style="color: #C678DD;"> =></span><span style="color: #61AFEF;"> setTimeout</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75;">resolve</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">waitSeconds</span><span style="color: #56B6C2;"> *</span><span style="color: #D19A66;"> 1000</span><span style="color: #ABB2BF;">));</span></span>
<span><span style="color: #C678DD;">  return</span><span style="color: #61AFEF;"> fetchWithBackoff</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75;">run</span><span style="color: #ABB2BF;">, </span><span style="color: #E06C75;">attempt</span><span style="color: #56B6C2;"> +</span><span style="color: #D19A66;"> 1</span><span style="color: #ABB2BF;">);</span></span>
<span><span style="color: #ABB2BF;">}</span></span></code></pre></figure>
<h3 id="normalizing-cursor-based-pagination"><a class="anchor-link" href="#normalizing-cursor-based-pagination">Normalizing Cursor-Based Pagination</a></h3>
<p>Third-party APIs use wildly different pagination strategies - offset, page numbers, cursor-based, or link headers. Truto normalizes all of these into a standard cursor-based format.</p>
<p>The AI learns that every list response from Truto contains a <code>next_cursor</code> field. It knows never to append <code>?page=2</code> to a Truto unified API call. It will write clean, predictable pagination loops that work identically whether you are pulling data from Jira, Greenhouse, or QuickBooks:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #C678DD;">type</span><span style="color: #E5C07B;"> Page</span><span style="color: #ABB2BF;">&#x3c;</span><span style="color: #E5C07B;">T</span><span style="color: #ABB2BF;">> </span><span style="color: #56B6C2;">=</span><span style="color: #ABB2BF;"> { </span><span style="color: #E06C75;">result</span><span style="color: #ABB2BF;">: </span><span style="color: #E5C07B;">T</span><span style="color: #ABB2BF;">[]; </span><span style="color: #E06C75;">next_cursor</span><span style="color: #C678DD;">?</span><span style="color: #ABB2BF;">: </span><span style="color: #E5C07B;">string</span><span style="color: #ABB2BF;"> | </span><span style="color: #E5C07B;">null</span><span style="color: #ABB2BF;"> };</span></span>
<span> </span>
<span><span style="color: #C678DD;">async</span><span style="color: #C678DD;"> function</span><span style="color: #61AFEF;"> listAllContacts</span><span style="color: #ABB2BF;">(</span><span style="color: #61AFEF;">fetchPage</span><span style="color: #ABB2BF;">: (</span><span style="color: #E06C75; font-style: italic;">nextCursor</span><span style="color: #C678DD;">?</span><span style="color: #ABB2BF;">: </span><span style="color: #E5C07B;">string</span><span style="color: #ABB2BF;">) </span><span style="color: #C678DD;">=></span><span style="color: #E5C07B;"> Promise</span><span style="color: #ABB2BF;">&#x3c;</span><span style="color: #E5C07B;">Page</span><span style="color: #ABB2BF;">&#x3c;</span><span style="color: #E5C07B;">any</span><span style="color: #ABB2BF;">>>) {</span></span>
<span><span style="color: #C678DD;">  const</span><span style="color: #E5C07B;"> out</span><span style="color: #ABB2BF;">: </span><span style="color: #E5C07B;">any</span><span style="color: #ABB2BF;">[] </span><span style="color: #56B6C2;">=</span><span style="color: #ABB2BF;">[];</span></span>
<span><span style="color: #C678DD;">  let</span><span style="color: #E06C75;"> nextCursor</span><span style="color: #ABB2BF;">: </span><span style="color: #E5C07B;">string</span><span style="color: #ABB2BF;"> | </span><span style="color: #E5C07B;">undefined</span><span style="color: #ABB2BF;">;</span></span>
<span> </span>
<span><span style="color: #C678DD;">  do</span><span style="color: #ABB2BF;"> {</span></span>
<span><span style="color: #C678DD;">    const</span><span style="color: #E5C07B;"> page</span><span style="color: #56B6C2;"> =</span><span style="color: #C678DD;"> await</span><span style="color: #61AFEF;"> fetchPage</span><span style="color: #ABB2BF;">(</span><span style="color: #E06C75;">nextCursor</span><span style="color: #ABB2BF;">);</span></span>
<span><span style="color: #E5C07B;">    out</span><span style="color: #ABB2BF;">.</span><span style="color: #61AFEF;">push</span><span style="color: #ABB2BF;">(...</span><span style="color: #E5C07B;">page</span><span style="color: #ABB2BF;">.</span><span style="color: #E06C75;">result</span><span style="color: #ABB2BF;">);</span></span>
<span><span style="color: #E06C75;">    nextCursor</span><span style="color: #56B6C2;"> =</span><span style="color: #E5C07B;"> page</span><span style="color: #ABB2BF;">.</span><span style="color: #E06C75;">next_cursor</span><span style="color: #56B6C2;"> ??</span><span style="color: #D19A66;"> undefined</span><span style="color: #ABB2BF;">;</span></span>
<span><span style="color: #ABB2BF;">  } </span><span style="color: #C678DD;">while</span><span style="color: #ABB2BF;"> (</span><span style="color: #E06C75;">nextCursor</span><span style="color: #ABB2BF;">);</span></span>
<span> </span>
<span><span style="color: #C678DD;">  return</span><span style="color: #E06C75;"> out</span><span style="color: #ABB2BF;">;</span></span>
<span><span style="color: #ABB2BF;">}</span></span></code></pre></figure>
<h3 id="unified-writes-still-need-an-escape-hatch"><a class="anchor-link" href="#unified-writes-still-need-an-escape-hatch">Unified Writes Still Need an Escape Hatch</a></h3>
<p>Another easy miss is write behavior. Truto's Unified API lets you send common fields and merge provider-specific fields through the <code>remote_data</code> object. That is a clean compromise: keep the shared model for most of the request, but leave room for the vendor oddities every real integration eventually needs. The skill explicitly teaches the AI to use <code>remote_data</code> when custom provider fields are requested, preventing the model from giving up on the Unified API too early.</p>
<h3 id="writing-complex-jsonata-transformations"><a class="anchor-link" href="#writing-complex-jsonata-transformations">Writing Complex JSONata Transformations</a></h3>
<p>One of Truto's most powerful features is the ability to map custom API data with JSONata. Writing complex JSONata expressions from scratch can be tedious. The <code>truto-jsonata</code> skill provides the AI with detailed examples of Truto's extended JSONata functions, such as <code>$mapValues</code>, <code>$firstNonEmpty</code>, and specific date formatters from the <code>@truto/truto-jsonata</code> package.</p>
<p>A response mapping that normalizes HubSpot contacts into Truto's unified schema becomes a single expression string generated effortlessly by the AI:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span>response.{</span></span>
<span><span>  "id": $string(id),</span></span>
<span><span>  "first_name": properties.firstname,</span></span>
<span><span>  "last_name": properties.lastname,</span></span>
<span><span>  "email": properties.email,</span></span>
<span><span>  "created_at": properties.createdate</span></span>
<span><span>}</span></span></code></pre></figure>
<p>If you prompt Cursor with: <em>"Write a Truto mapping that takes a Salesforce response, extracts the custom <code>Industry__c</code> field, and maps it to <code>company_sector</code>, defaulting to 'Unknown' if missing,"</em> the AI will instantly generate the correct expression using the <code>$firstNonEmpty</code> function.</p>
<h2 id="building-with-the-truto-cli-and-truto-skills"><a class="anchor-link" href="#building-with-the-truto-cli-and-truto-skills">Building with the Truto CLI and Truto Skills</a></h2>
<p>Writing code is only half the battle. You still need to test it, inspect the data, and manage platform resources. The <code>truto-cli</code> skill bridges the gap between writing integration logic and actually running it.</p>
<p>The <a href="https://truto.one/cli">Truto CLI</a> covers the full admin API, data-plane APIs, and power-user features like bulk export and diffing (read our <a href="/blog/truto-cli/">CLI blog post</a> for a deep dive). Without context, an AI will guess the CLI syntax. It will invent flags like <code>--account-id</code> instead of the required <code>-a</code>, or it will try to write a custom Python script to paginate through records when a single CLI command would do the job.</p>
<p>With the skill loaded, the AI knows the exact command structures. You can prompt your assistant to scaffold resources directly from your terminal.</p>
<p><strong>Managing Admin Resources</strong>
Instead of clicking through a dashboard, tell the AI to create an integration. It knows the required fields and optimistic locking rules:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #61AFEF;">truto</span><span style="color: #98C379;"> integrations</span><span style="color: #98C379;"> create</span><span style="color: #D19A66;"> -b</span><span style="color: #98C379;"> '{"name":"slack","config":{"label":"Slack","auth_type":"oauth2"}}'</span></span></code></pre></figure>
<p><strong>Testing Data-Plane APIs</strong>
When you need to verify a unified model or proxy endpoint, the AI knows how to construct the right call. It understands that unified paths contain a slash (<code>crm/contacts</code>) while proxy paths do not (<code>tickets</code>), and that the <code>-a</code> flag is mandatory:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #61AFEF;">truto</span><span style="color: #98C379;"> unified</span><span style="color: #98C379;"> crm</span><span style="color: #98C379;"> contacts</span><span style="color: #D19A66;"> -m</span><span style="color: #98C379;"> search</span><span style="color: #D19A66;"> -a</span><span style="color: #ABB2BF;"> &#x3c;</span><span style="color: #98C379;">account-i</span><span style="color: #ABB2BF;">d> </span><span style="color: #D19A66;">-b</span><span style="color: #98C379;"> '{"query":"Jane"}'</span></span></code></pre></figure>
<p><strong>Bulk Exports and Data Piping</strong>
If you need to extract data for analysis, the AI won't write a custom pagination script. It knows the CLI has an <code>export</code> command that handles auto-pagination natively. It also knows to use <code>ndjson</code> for streaming large datasets instead of buffering everything in memory:</p>
<figure><pre style="background-color: #282c34; color: #abb2bf;" tabindex="0"><code style="display: grid;"><span><span style="color: #61AFEF;">truto</span><span style="color: #98C379;"> export</span><span style="color: #98C379;"> crm/contacts</span><span style="color: #D19A66;"> -a</span><span style="color: #ABB2BF;"> &#x3c;</span><span style="color: #98C379;">account-i</span><span style="color: #ABB2BF;">d> </span><span style="color: #D19A66;">-o</span><span style="color: #98C379;"> ndjson</span><span style="color: #ABB2BF;"> | </span><span style="color: #61AFEF;">jq</span><span style="color: #98C379;"> '.email'</span></span></code></pre></figure>
<p>By combining the CLI with Truto Skills, your AI assistant becomes an operator, not just a code generator. It can scaffold the configuration, test the endpoints, and export the results without leaving the editor.</p>
<h2 id="why-declarative-platforms-are-the-best-match-for-ai-agents"><a class="anchor-link" href="#why-declarative-platforms-are-the-best-match-for-ai-agents">Why Declarative Platforms Are the Best Match for AI Agents</a></h2>
<p>There's a broader point here that goes beyond Truto specifically. The platforms that benefit most from AI coding assistants are the ones where "writing code" means authoring configuration and expressions rather than imperative logic.</p>
<p>Consider the contrast. On a code-heavy integration platform, asking the AI to add a new CRM connector means generating hundreds of lines across multiple files - an HTTP client class, authentication handler, pagination logic, response serializers, error mappers, and tests for all of it. The surface area for hallucination is enormous.</p>
<p>As detailed in our guide to <a href="/blog/zero-integration-specific-code-how-to-ship-new-api-connectors-as-data-only-operations/">shipping API connectors as data operations</a>, Truto uses zero integration-specific code. Everything is handled via declarative JSON configurations and JSONata mapping expressions. The runtime engine - which handles auth, pagination, error normalization, and everything else - is the same generic pipeline for every integration. No per-integration code to hallucinate.</p>
<p>This architectural choice wasn't made with AI in mind; it was made for extensibility and reliability. But it turns out that a platform designed around declarative data rather than imperative code is exactly the kind of system where AI assistants excel. The skill just needs to teach the AI the shape of the config and the JSONata expression syntax. The platform handles the rest.</p>
<h2 id="stop-fighting-the-ai-start-shipping-integrations"><a class="anchor-link" href="#stop-fighting-the-ai-start-shipping-integrations">Stop Fighting the AI, Start Shipping Integrations</a></h2>
<p>The gap between AI coding assistant adoption and trust in their output tells a clear story: developers use these tools because the speed gains are real, but they spend too much time fixing the output. A generic coding assistant is fast but generic. A skill-equipped assistant is still not infallible, but now it knows your platform rules, your escape hatches, and the difference between a real abstraction and an invented one.</p>
<p>If your team is already using Cursor or Claude Code to build B2B SaaS integrations, the next step is obvious: stop pasting the same Truto docs into chat threads and install the context once. You stop fighting the AI over outdated documentation and start treating it like a senior integration engineer who has memorized the entire platform specification.</p>
<aside class="cta cta-row"><p>Want to stop maintaining brittle API integrations? Partner with Truto to normalize your third-party data, handle OAuth lifecycle management, and ship customer-facing integrations in days, not months.</p><a class="cta-button" href="https://cal.com/truto/partner-with-truto">Talk to us</a></aside>
