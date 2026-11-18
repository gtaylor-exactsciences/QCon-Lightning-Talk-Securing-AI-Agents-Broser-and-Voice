<script setup>
defineProps({ stage: { type: Number, default: 1 } })
</script>

<div class="flex h-96 items-center justify-center" :style="{ '--stage2': stage >= 2 ? 1 : 0, '--stage3': stage >= 3 ? 1 : 0 }">

```mermaid {scale: 1.25, flowchart: {wrappingWidth: 120, rankSpacing: 20, nodeSpacing: 20}, themeVariables: {edgeLabelBackground: '#ffffff'}}
---
config:
  themeCSS: |
    [id*=-flowchart-agent-], [data-id=handoff], [data-id=exchange] { opacity: var(--stage2, 1); }
    [id*=-flowchart-api-], [data-id=access] { opacity: var(--stage3, 1); }
---
flowchart LR
  browser@{ img: "/icons/chat.svg", label: "Browser chat", pos: "b", h: 64, constraint: "on" }
  agent@{ img: "/icons/bot.svg", label: "AI agent", pos: "b", h: 64, constraint: "on" }
  auth@{ img: "/icons/security.svg", label: "Authorization server", pos: "b", h: 64, constraint: "on" }
  api@{ img: "/icons/bare-metal-server.svg", label: "Patient APIs", pos: "b", h: 64, constraint: "on" }

  browser -->|Authorization code<br/>flow · RFC 6749| auth
  browser handoff@-->|Patient token| agent
  agent exchange@-->|Token exchange<br/>RFC 8693| auth
  agent access@-->|Delegated token| api

  classDef default fill:none,stroke:none;
  linkStyle default stroke:#333,stroke-width:3px;
```

</div>
