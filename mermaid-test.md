---
layout: page
title: Mermaid Test
---

# Mermaid Diagram Test

This is a test page to verify Mermaid.js integration is working properly.

## Simple Flow Chart

```mermaid
graph TD
    A[Start] --> B{Is it working?}
    B -->|Yes| C[Great!]
    B -->|No| D[Debug it]
    D --> B
    C --> E[End]
```

## Architecture Diagram

```mermaid
graph LR
    A[Blazor WASM<br/>Client] -->|HMAC Auth| B[Cloudflare<br/>Worker]
    B -->|API Key| C[AbuseIPDB<br/>API]
    C -->|JSON Data| B
    B -->|CORS + JSON| A
```

If you can see rendered diagrams above instead of code blocks, Mermaid.js is working correctly!
