---
layout: page
title: Mermaid Test
---

# Mermaid Diagram Test

This page tests Mermaid.js integration.

<div class="mermaid">
graph TD
    A[Start] --> B{Is it working?}
    B -->|Yes| C[Great!]
    B -->|No| D[Debug it]
    D --> B
    C --> E[End]
</div>

<div class="mermaid">
graph LR
    A[Blazor WASM<br/>Client] -->|HMAC Auth| B[Cloudflare<br/>Worker]
    B -->|API Key| C[AbuseIPDB<br/>API]
    C -->|JSON Data| B
    B -->|CORS + JSON| A
</div>

If you see rendered diagrams above, Mermaid is working!

<script src="https://unpkg.com/mermaid@10.6.1/dist/mermaid.min.js"></script>
<script>
mermaid.initialize({ startOnLoad: true, theme: 'default' });
</script>
