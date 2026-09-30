# Browserbase

> Managed headless browser platform for agents that browse and interact with the web

| Field | Value |
|-------|-------|
| Group | Browser Automation |
| Type | API/SDK |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://docs.browserbase.com/) |

## Overview

Browserbase is a hosted platform for running headless browsers at scale, targeted at AI agents that need to browse and interact with the web like humans. One API key gives an agent access to cloud-hosted Chromium browsers, integrated web search, page-fetch utilities, a sandbox runtime, and a multi-model gateway covering every major LLM provider. The product handles the operational overhead — proxy rotation, captcha solving, geographic distribution, session recording, fingerprinting — that a self-managed Playwright deployment would otherwise carry. [1]

Browserbase is closely associated with **Stagehand**, an open-source agent framework purpose-built for browser control on top of Browserbase, that lets agents describe what to do in natural language (`page.act("click the login button")`) rather than writing CSS selectors. Stagehand can also run against any Chromium-compatible browser but is most commonly paired with Browserbase. [2]

## Core Concepts

### Session

A Session is a single browser instance — Chromium running in Browserbase's cloud. Sessions have a lifecycle: created via the API, connected to via CDP (Chrome DevTools Protocol) for Playwright/Puppeteer control, and torn down explicitly or by inactivity timeout. Each session has its own cookies, local storage, and viewport. [3]

### Connect URL

When a session is created, the API returns a `connectUrl` — a WebSocket URL that Playwright, Puppeteer, or the Chrome DevTools Protocol can connect to. From there, the SDK is the standard browser-automation SDK; Browserbase replaces the local Chromium process with a remote one. [3]

### Stagehand (companion framework)

Stagehand is the open-source layer on top of Playwright that adds AI-driven page interaction: `page.act("fill in the form")`, `page.extract({ schema })`, `page.observe()`. Stagehand uses an LLM to translate intent into Playwright actions, with retries and self-correction baked in. [2]

### Browser API and Runtime

The Browserbase Platform splits into several products:

- **Browser API**: Create and control browser sessions programmatically.
- **Runtime**: Deploy and run browser agents on Browserbase infrastructure, on schedule or on demand.
- **Search**: Hosted web search optimized for agent consumption.
- **Fetch**: Single-URL fetch with a managed browser (similar to Jina Reader).
- **Model Gateway**: Unified access to frontier LLMs (Claude, OpenAI, Gemini, others) behind the same API key. [1]

## Architecture

A typical Browserbase exchange:

1. Agent calls `bb.sessions.create()` via the Browserbase SDK.
2. Browserbase provisions a Chromium browser in a managed VM; returns a session ID and a `connectUrl`.
3. Agent's Playwright/Puppeteer client connects to `connectUrl` over CDP.
4. Agent drives the browser using standard Playwright APIs (`page.goto`, `page.click`, `page.screenshot`).
5. Optional features kick in transparently: proxies for IP rotation, captcha solving, session recording.
6. Agent closes the session or it auto-times out after the configured period. [3]

The agent code itself can run anywhere — local dev machine, serverless function, agent framework like LangGraph — because the heavy lifting (the browser process) lives in Browserbase's cloud.

## Key Features and Functionality

### Cloud Browsers

Browserbase runs Chromium browsers in its cloud with managed scaling, persistent storage, and snapshotting. Concurrent sessions scale to many hundreds depending on plan tier. [1]

### Proxies and Geographic Distribution

Built-in proxy support with country/region targeting lets agents appear to originate from a specific geo. Residential proxies are available for sites that block datacenter IPs. [4]

### Captcha Solving

Integrated captcha-solving for common challenges (reCAPTCHA, hCaptcha) reduces hand-off requirements when an agent encounters a gate.

### Session Recording

Each session can be recorded as video plus a step-level event log, viewable in the dashboard. Critical for debugging long-running agent failures.

### Live Debugger

`bb.sessions.debug(sessionId)` returns a debugger URL where a developer can watch the live browser session in real time — useful during agent development.

### Fingerprinting

Browserbase manages browser fingerprints (user agent, screen resolution, timezone, fonts) to reduce bot-detection signals.

### Storage and Persistence

Cookies and local storage can be persisted across sessions using browser contexts — so an agent can sign in once and reuse the auth on the next run.

### Search API

A dedicated `/v1/search` endpoint runs a web search and returns LLM-ready results, similar in shape to Jina Search. Useful when an agent needs to discover URLs to browse. [5]

### Model Gateway

Unified access to frontier LLMs (Claude, OpenAI, Gemini, Mistral, others) under the same API key, with shared billing and observability. Removes the need for separate per-provider API keys when building multi-model agents. [6]

## Use Cases

### Browser Agents

Pair an LLM (Claude, GPT-5) with Stagehand running on Browserbase. The LLM emits high-level intents (`click the cookie banner`, `extract product prices`), Stagehand translates each into Playwright calls, and Browserbase executes them in the cloud browser. [2]

### Web Automation Workflows

Automate logins, registrations, multi-step form submissions, and recurring web actions on schedule. Browserbase Runtime can host the agent code itself so the whole loop runs in the cloud. [4]

### Web Data Retrieval

Extract structured data from pages that require JavaScript rendering or interaction (infinite scroll, click-to-reveal). Combines well with Stagehand's `page.extract({ schema })` for typed output.

### End-to-End Testing

Run cross-browser tests at scale without managing browser infrastructure. The Playwright/Puppeteer compatibility means existing test suites port directly.

### Customer-Facing Browser Agents

Embed a browser agent in a product that needs to act on the user's behalf — e.g., a travel-booking assistant that fills out airline forms.

## API Reference Summary

### Browserbase SDK (Sessions)

- `bb.sessions.create(options?)` — Create a new browser session, returns `{ id, connectUrl, ... }`
- `bb.sessions.retrieve(id)` — Get session metadata
- `bb.sessions.list(filters?)` — List sessions
- `bb.sessions.update(id, { status: "REQUEST_RELEASE" })` — Release a session
- `bb.sessions.debug(id)` — Get live debugger URLs

### Session Options

- `projectId` — Required project association
- `proxies` — Proxy configuration
- `browserSettings` — Viewport, user agent, fingerprint overrides
- `extensionId` — Inject a Chrome extension
- `keepAlive` — Don't auto-close on disconnect

### Search API

- `POST /v1/search` — Web search returning LLM-ready results [5]

### Fetch API

- `POST /v1/fetch` — Single-URL fetch via managed browser

## Configuration

### Authentication

Set `BROWSERBASE_API_KEY` and `BROWSERBASE_PROJECT_ID` environment variables.

### Proxies

Proxies are enabled per-session via the `proxies` option. Built-in proxies offer per-country targeting; you can also bring your own. [4]

### Browser Settings

Per-session overrides for viewport, user agent, locale, and fingerprint reduce bot-detection signals on sites that check these.

### Concurrency Limits

Plans cap concurrent sessions; production deployments often pool sessions or use the Runtime product for managed concurrency.

## Integration Patterns

### With Playwright

Drop-in replacement for local Chromium: `chromium.connectOverCDP(session.connectUrl)` instead of `chromium.launch()`. Existing Playwright code runs unchanged.

### With Puppeteer

Same pattern: `puppeteer.connect({ browserWSEndpoint: session.connectUrl })`.

### With Stagehand

Stagehand wraps Playwright with AI-driven action helpers. The combo (Browserbase + Stagehand + Claude/GPT) is the canonical "browser agent" stack.

### With Agent Frameworks (LangGraph, CrewAI, OpenAI Agents SDK)

Expose Browserbase-driven Stagehand actions as agent tools; the agent reasons about the page and emits actions via the framework's tool-call interface.

### With LLM Providers (via Model Gateway)

Use Browserbase's Model Gateway to bill all LLM usage (Claude, OpenAI, Gemini) under one account, useful for projects that switch models frequently. [6]

## Examples

### Playwright Quickstart (Node.js)

```js
import Browserbase from "@browserbasehq/sdk";
import { chromium } from "playwright-core";

const bb = new Browserbase({ apiKey: process.env.BROWSERBASE_API_KEY });

(async () => {
  const session = await bb.sessions.create();
  console.log(`Session created, id: ${session.id}`);

  const browser = await chromium.connectOverCDP(session.connectUrl);
  const defaultContext = browser.contexts()[0];
  const page = defaultContext.pages()[0];

  await page.goto("https://www.browserbase.com/", { waitUntil: "domcontentloaded" });

  const debugUrls = await bb.sessions.debug(session.id);
  console.log(`Live debug at: ${debugUrls.debuggerUrl}`);

  await page.screenshot({ fullPage: true });

  await page.close();
  await browser.close();
})();
```

### Stagehand with AI Actions

```js
import { Stagehand } from "@browserbasehq/stagehand";

const stagehand = new Stagehand({ env: "BROWSERBASE" });
await stagehand.init();
await stagehand.page.goto("https://news.ycombinator.com");
await stagehand.page.act("click the first story link");
const data = await stagehand.page.extract({
  instruction: "extract the article title and author",
  schema: { title: "string", author: "string" },
});
console.log(data);
await stagehand.close();
```

## Limitations and Considerations

- **Per-session cost**: Sessions are billed by duration and concurrency tier. Bots that idle in a session burn budget.
- **Captcha solving is not 100%**: Hard captchas (image grids, behavioral checks) still fail sometimes; build a human-handoff fallback.
- **Sites detecting cloud IPs**: Some destinations block datacenter IPs; residential proxies help but add cost.
- **Stagehand is LLM-dependent**: AI actions cost LLM tokens per step; latency = LLM round-trip + browser action. Batching actions matters.
- **Chrome-only**: Browserbase runs Chromium; Firefox/WebKit are not first-class targets.
- **Closed-source service**: For air-gapped or compliance-restricted environments, the open-source Playwright running on self-managed infra is the alternative.

## Changelog Highlights

- **Model Gateway**: Unified LLM access (Claude, OpenAI, Gemini, etc.) under one API key
- **Runtime**: Hosted agent execution on Browserbase infrastructure
- **Search and Fetch APIs**: First-party LLM-ready web search and URL fetch
- **Stagehand integration**: First-class support for AI-driven page actions
- **Session recording and live debugger**: Built-in observability for browser agents [1]

## Citations

- [1] Introducing Browserbase - <https://docs.browserbase.com/welcome/introduction>
- [2] Stagehand - <https://docs.stagehand.dev/>
- [3] Playwright Quickstart - <https://docs.browserbase.com/welcome/quickstarts/playwright>
- [4] Browser Automation Use Case - <https://docs.browserbase.com/use-cases/automating-form-submissions>
- [5] Search Platform - <https://docs.browserbase.com/platform/search/overview>
- [6] Model Gateway Platform - <https://docs.browserbase.com/platform/model-gateway/overview>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- Browserbase
- managed headless browser
- cloud Chromium
- Playwright remote
- Puppeteer remote
- CDP (Chrome DevTools Protocol)
- connectOverCDP
- Stagehand
- AI page actions
- page.act
- page.extract
- proxy rotation
- residential proxies
- captcha solving
- session recording
- live debugger
- fingerprint management
- session persistence
- Browserbase Runtime
- Search API
- Fetch API
- Model Gateway
- browser agent
- web automation at scale

### Verb-Noun Tasks

- Spin up a cloud Chromium browser via API
- Connect Playwright or Puppeteer to a remote browser over CDP
- Drive a web page with natural-language intents via Stagehand
- Extract typed data from a page with `page.extract({ schema })`
- Route browser traffic through country-targeted residential proxies
- Solve reCAPTCHA and hCaptcha challenges during automation
- Record a browser session as video plus a step-level event log
- Watch a live browser session via the Browserbase debugger URL
- Run web search via the hosted `/v1/search` endpoint
- Fetch and render a single URL with a managed browser
- Embed a browser agent inside a customer-facing product
- Pool concurrent sessions for high-throughput automation

### User Intent Phrases

- How do I run Playwright at scale without managing Chromium infrastructure?
- I need a headless browser my AI agent can drive in the cloud
- How can I bypass bot detection and captchas in web automation?
- How do I make an LLM click buttons and fill forms on a JavaScript-heavy site?
- I want the agent to describe actions in natural language, not CSS selectors
- How do I record and replay a long-running browser agent session?
- How do I rotate residential proxies per country for a scraping agent?
- How do I extract structured data from a page that requires interaction?
- What is the canonical "browser agent" stack?
- How do I let a travel-booking assistant fill out airline forms?

### Problem Statements

- Self-managed Playwright deployments inherit proxy, captcha, and fingerprinting headaches
- Datacenter IPs get blocked; residential proxies are expensive and operationally complex
- Long browser sessions burn budget if they idle in a session
- CSS-selector automation breaks every time the site changes
- LLM-driven actions add token cost and round-trip latency per step
- Hard captchas still require human handoff
- Firefox/WebKit are not first-class targets; some sites require non-Chromium

### When to Pick This

- Pick Browserbase when your agent's action surface is the web — pages, forms, clicks, scrapes — rather than arbitrary code
- Pick Browserbase + Stagehand when you want AI-driven page intent (`page.act`, `page.extract`) instead of brittle CSS selectors
- Pick Browserbase over self-hosted Playwright when you need managed proxies, captcha solving, fingerprinting, and session recording out of the box
- Pick E2B instead when the agent needs to write and execute Python/code, not drive a browser
- Pair with MCP by wrapping `page.act` or `page.extract` as MCP tools usable by any MCP host
- Pick self-hosted Playwright when compliance forbids closed-source managed services

### Related Terms and Aliases

- cloud browser platform
- browser-as-a-service
- agentic web automation
- LLM-driven browser
- Playwright cloud
- Puppeteer cloud
- Chromium-as-a-service
- Stagehand framework
- web acting agent
- AI scraping browser
