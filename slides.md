---
theme: default
title: Securing AI Agents Across Voice and Web
info: A five-minute talk about OAuth building blocks and interoperable delegation
layout: cover
colorSchema: light
defaults:
  class: '[&_a]:border-0 [&_a]:text-blue-600 [&_a]:underline [&_h3]:text-3xl
    [--slidev-code-font-size:16px] [--slidev-code-line-height:24px]'
fonts:
  provider: none
---

# Securing AI Agents Across Voice and Web

[Greg Taylor](https://www.linkedin.com/in/gregorysdtaylor) @ Abbott

<!--
Target 00:00–00:49 · 49 seconds

Hi, I’m Greg Taylor, a staff software engineer at Abbott. I’d like to share how we secured AI agents across voice and web, using web standards as a security foundation and a way to interoperate.

There’s a link to the slides at the end.

I'm focusing on the API integration security layer, but would recommend checking out the recording / notes for

Weiping Peng's talk on How Airbnb Guardrailed Its AI Customer Support Agent for internal controls for the AI agent itself, and 

Amanda Walker's talk on Agentic Trust: Securing AI Agents in Production for a more complete and holistic view of the security system.
-->

---
title: Patient support — browser and voice
---

# Patient support: browser and voice

<div class="flex h-96 items-center justify-center">

```mermaid {scale: 1.25, flowchart: {wrappingWidth: 120}, themeVariables: {edgeLabelBackground: '#ffffff'}}
flowchart LR
  chat@{ img: "/icons/chat.svg", label: "Browser chat", pos: "b", h: 64, constraint: "on" }
  phone@{ img: "/icons/phone-voice.svg", label: "Phone call", pos: "b", h: 64, constraint: "on" }
  agent@{ img: "/icons/bot.svg", label: "Support agent", pos: "b", h: 64, constraint: "on" }
  api@{ img: "/icons/bare-metal-server.svg", label: "Patient APIs", pos: "b", h: 64, constraint: "on" }

  chat --> agent
  phone --> agent
  agent --> api

  classDef default fill:none,stroke:none;
  linkStyle default stroke:#333,stroke-width:3px;
```

</div>

<!--
Target 00:49–01:13 · 24 seconds

We needed patient support over browser chat and phone, with read and write access to sensitive data. Some patients didn’t have an account in our enterprise identity provider, so we needed custom authentication. We also needed the authentication to carry explicit identity and context for authorization and audit.
-->

---
title: MCP authorization
layout: default
class: flex flex-col justify-center
---

# <span class="text-red-600">✕</span> MCP authorization

We didn’t need or want dynamic client registration or tool discovery.

<!--
Target 01:13–01:31 · 18 seconds

Our original brief was an MCP server with authorization. But our clients and tools were known in advance. The dynamic registration and discovery that MCP would provide turned out to not helpful for our case.
-->

---
title: Service-account-only access
layout: default
class: flex flex-col justify-center
---

# <span class="text-red-600">✕</span> Service-account-only access

Quick integration, but no patient identity for audit or authorization.

<!--
Target 01:31–01:48 · 17 seconds

Service-account credentials for API calls would allow for a quick integration. But that credential didn’t explicitly carry both the patient and AI agent identity. That wasn’t enough for our audit and authorization requirements.
-->

---
title: Browser authentication
---

<script setup>
import BrowserDiagram from './slides/browser-diagram.md'
</script>

# Browser authentication

<BrowserDiagram :stage="1" />

<!--
Target 01:48–02:12 · 24 seconds

For browser chat, we used the standard authorization code flow. Our own authorization server let us plug in knowledge-based authentication for patients without an existing login. Because this is a standard flow, we can use other identity providers for this step when a patient already has an account.
-->

---
title: Scopes
---

# [RFC 6749](https://www.rfc-editor.org/rfc/rfc6749.html#section-3.3) · Access Token Scope

<div class="flex h-96 flex-col justify-center">

<div class="grid grid-cols-[1fr_auto_1fr] items-start gap-6">
<div>

### Token request

```text
scope=account.read account.write
```

</div>
<div class="self-center text-3xl">→</div>
<div>

### Access token

```json
{
  "scope": "account.read account.write"
}
```

</div>
</div>

</div>

<!--
Target 02:12–02:28 · 16 seconds

Scopes gave us coarse permissions, like account read or write. I think there’s room for the standards to go further on fine-grained permissions, but these aren't needed for our use case.
-->

---
title: Browser authentication
---

<script setup>
import BrowserDiagram from './slides/browser-diagram.md'
</script>

# Browser authentication

<BrowserDiagram :stage="2" />

<!--
Target 02:28–02:37 · 9 seconds

We can represent explicit identities for both the patient and the AI agent delegate using token exchange.
-->

---
title: Token exchange
---

# [RFC 8693](https://www.rfc-editor.org/rfc/rfc8693.html) · OAuth 2.0 Token Exchange

<div class="flex h-96 flex-col justify-center">

<div class="grid grid-cols-[1fr_auto_1fr_auto_1fr] gap-3 items-start">
<div>

### Patient token

```json
{
  "sub": "patient-123",
  "client_category":
    "human_operated_app",
  "may_act": {
    "sub": "agent-7",
    "client_category":
      "ai_agent"
  }
}
```

</div>
<div class="text-3xl">+</div>
<div>

### Agent token

```json
{
  "sub": "agent-7",
  "client_category":
    "ai_agent"
}
```

</div>
<div class="text-3xl">→</div>
<div>

### Delegated token

```json
{
  "sub": "patient-123",
  "client_category":
    "ai_agent",
  "act": {
    "sub": "agent-7",
    "client_category":
      "ai_agent"
  }
}
```

</div>
</div>

```http
POST /oauth2/token

grant_type=urn:ietf:params:oauth:grant-type:token-exchange
```

</div>

<!--
Target 02:37–03:08 · 31 seconds

The patient token's may_act claim names the eligible delegate. The standards didn't specify how to set this, so we extended them with a custom parameter.

The patient identy ends up in the subject, agent identity as the actor.

We added a custom client category to distinguish between AI agents and human-operated apps.

We also restrict initial patient tokens to exchange-only.
-->

---
title: Browser authentication
---

<script setup>
import BrowserDiagram from './slides/browser-diagram.md'
</script>

# Browser authentication

<BrowserDiagram :stage="3" />

<!--
Target 03:08–03:23 · 15 seconds

The agent now calls the patient API with its delegated token. The API enforces the permissions for those identities; without relying on a non-deterministic agent to following its prompt perfectly.
-->

---
title: Resource indicators
---

# [RFC 8707](https://www.rfc-editor.org/rfc/rfc8707.html) · Resource Indicators for OAuth 2.0

<div class="flex h-96 flex-col justify-center">

<div class="grid grid-cols-[1fr_auto_1fr] items-start gap-6">
<div>

### Token request

```text
resource=https://api.example/patients
```

</div>
<div class="self-center text-3xl">→</div>
<div>

### Access token

```json
{
  "aud": ["https://api.example/patients"]
}
```

</div>
</div>

</div>

<!--
Target 03:23–03:50 · 27 seconds

The resource parameter locks the token to the target API using its audience claim. At this point, we've given the API enough information to make a strong security decision:
- Who is the end patient
- Which agent acting on their behalf
- What coarse permissions have been granted
- The token is intended for this API
-->

---
title: Voice authentication
---

# Voice authentication


<div class="flex h-96 items-center justify-center">

```mermaid {scale: 1.25, flowchart: {wrappingWidth: 120, rankSpacing: 20, nodeSpacing: 20}, themeVariables: {edgeLabelBackground: '#ffffff'}}
flowchart LR
  phone@{ img: "/icons/phone-voice.svg", label: "Phone call", pos: "b", h: 64, constraint: "on" }
  agent@{ img: "/icons/bot.svg", label: "AI agent", pos: "b", h: 64, constraint: "on" }
  auth@{ img: "/icons/security.svg", label: "Authorization server", pos: "b", h: 64, constraint: "on" }
  api@{ img: "/icons/bare-metal-server.svg", label: "Patient APIs", pos: "b", h: 64, constraint: "on" }

  phone -->|Voice interaction| agent
  agent -->|First-party<br/>applications flow<br/>IETF draft 04| auth
  agent -->|Token exchange<br/>RFC 8693| auth
  agent -->|Delegated token| api

  classDef default fill:none,stroke:none;
  linkStyle default stroke:#333,stroke-width:3px;
```

</div>

<!--
Target 03:50–04:14 · 24 seconds

The previous system doesn't work outside of a browser, but supporting voice  just requires a change to the front door. The First-Party Applications draft lets our trusted client collect verification inputs without a browser redirect. After authentication, we require the same token exchange and delegated API access.
-->

---
title: OAuth 2.0 for First-Party Applications
---

# [IETF Draft 04](https://www.ietf.org/archive/id/draft-ietf-oauth-first-party-apps-04.html) · OAuth 2.0 for First-Party Applications

<div class="flex h-96 flex-col justify-center">

<div class="grid grid-cols-[1fr_auto_1fr] items-start gap-6">
<div>

### Challenge request

```http
POST /authorize-challenge

response_type=code
phone=…
date_of_birth=…
postal_code=…
scope=account.read
```

</div>
<div class="self-center text-3xl">→</div>
<div>

### Code response

```json
{
  "authorization_code": "code-123"
}
```

</div>
</div>

</div>

<!--
Target 04:14–04:34 · 20 seconds

Send a challenge request and get an authorization code. Our patient-verification fields are custom. The code is still redeemed at the token endpoint, before token exchange. The draft is a work in progress but I see real value here.
-->

---
title: Conclusion
layout: default
class: flex flex-col justify-center
---

# Existing web standards are a strong foundation for agent security in curated experiences.

<!--
Target 04:34–04:50 · 16 seconds

Existing web standards are a great fit for a curated experience with known clients and tools. They gave us a security foundation we could build on and a way to interoperate.
-->

---
title: Thank you
---

# Thank you

For writing drafts, explaining the patterns, and helping work through real use cases in public.

<div class="grid h-80 grid-cols-[1fr_300px] items-center gap-8">
<div class="flex flex-col gap-6 [&_p]:m-0">

[OAuth 2.0 for First-Party Applications](https://www.ietf.org/archive/id/draft-ietf-oauth-first-party-apps-04.html) — Aaron Parecki, George Fletcher, Pieter Kasselman

[On-Behalf-Of Is a Pattern. Token Exchange Is the Standard.](https://auth0.com/blog/on-behalf-of-is-a-pattern-token-exchange-is-the-standard/) — Andrea Chiarelli

[Browserless authentication and related proposals](https://github.com/modelcontextprotocol/ext-auth/issues/19) — MCP authorization discussion contributors

</div>

<a href="https://github.com/gtaylor-exactsciences/QCon-Lightning-Talk-Securing-AI-Agents-Broser-and-Voice">
<img src="/repository-qr.png" alt="QR code linking to the presentation GitHub repository" class="w-75 h-75" />
<div class="mt-2 text-center text-lg">Slides & source</div>
</a>
</div>

<!--
Target 04:50–05:00 · 10 seconds

And thank you to the people writing drafts, explaining the patterns, and helping work through real use cases in public.

Reference credits:
- MCP discussion: calvertj, travishaagen, aaronpk, localden, WhammyLeaf.
- Related proposal authors: Atul Tulshibagwale and Erik Gustavson; Thilina Shashimal Senarath and Ayesha Dissanayaka; Karl McGuinness; Niki Aimable Niyikiza; Joseph Salowey, Yaroslav Rosomakho, and Hannes Tschofenig. Full attribution and links are in the README acknowledgments.
-->
