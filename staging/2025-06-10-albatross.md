---
layout: post
title: Albatross
description: "Albatross Cloud IP maifest & reputation search"
date: 2025-06-10
tags: [blazorWebAssembly, cloudflareWorkers, devSecOps, webDevelopment, cyberSecurity]
---

# Building Albatross: A Secure IP Abuse Checker with Blazor WebAssembly and Cloudflare Workers

*Published: June 9, 2025*

## 📝 TL;DR

Built **Albatross**, a secure IP abuse checker using Blazor WebAssembly + Cloudflare Workers. Key features:
- 🔐 **Secure API proxy** with HMAC authentication (no exposed API keys)
- ⚡ **Fast client-side app** with server-side API protection
- 🛡️ **Real-time IP reputation checking** via AbuseIPDB
- 🏗️ **Modern architecture** combining the best of client and edge computing

**Tech Stack:** Blazor WASM, Cloudflare Workers, HMAC-SHA256, AbuseIPDB API

---

In an era where cybersecurity threats are constantly evolving, having reliable tools to check IP addresses for malicious activity has become essential. Today, I'm excited to share the journey of building **Albatross** - a secure, modern web application that leverages the AbuseIPDB API to provide real-time IP abuse checking through a sophisticated architecture combining Blazor WebAssembly and Cloudflare Workers.

## The Challenge: Secure API Proxy Architecture

When building client-side applications that need to interact with third-party APIs, developers face a common dilemma: how do you protect sensitive API keys while maintaining a seamless user experience? Traditional approaches often involve:

1. **Exposing API keys in client code** (security nightmare)
2. **Building a full backend service** (overhead and complexity)
3. **Using serverless functions** (cold starts and vendor lock-in)

For Albatross, I chose a different path: **Cloudflare Workers as a secure API proxy** with a sophisticated authentication system that generates unique keys at build time.

## Architecture Overview

```
┌─────────────────┐    HMAC Auth     ┌─────────────────┐    API Key    ┌─────────────────┐
│                 │   ──────────→    │                 │  ──────────→  │                 │
│  Blazor WASM    │                  │ Cloudflare      │               │   AbuseIPDB     │
│     Client      │   ←──────────    │    Worker       │  ←──────────  │      API        │
│                 │    CORS + JSON   │                 │   JSON Data   │                 │
└─────────────────┘                  └─────────────────┘               └─────────────────┘
```

### Key Components:

1. **Blazor WebAssembly Frontend**: Modern, responsive UI built with .NET 8
2. **Cloudflare Worker Proxy**: Secure middleware handling authentication and API calls
3. **Build-Time Key Generation**: Cryptographically secure authentication keys generated per build
4. **HMAC Authentication**: Request signing using SHA-256 for tamper-proof communication

## The Security Foundation: Build-Time Key Generation

One of the most innovative aspects of Albatross is its build-time authentication system. Instead of using static API keys or tokens, the system generates a unique 256-bit cryptographic key for every build.

### PowerShell Key Generation Script

```powershell
# Generate-AuthKey.ps1
function Generate-SecureKey {
    $rng = [System.Security.Cryptography.RNGCryptoServiceProvider]::Create()
    $keyBytes = New-Object byte[] 32  # 256 bits
    $rng.GetBytes($keyBytes)
    return [Convert]::ToBase64String($keyBytes)
}

$authKey = Generate-SecureKey
$buildId = [System.Guid]::NewGuid().ToString("N").Substring(0, 8)
$timestamp = Get-Date -Format "yyyyMMdd-HHmm"
```

This approach provides several security benefits:

- **Automatic Key Rotation**: Every build gets a fresh authentication key
- **No Hardcoded Secrets**: Keys are generated dynamically and never stored in source code
- **Build Integrity**: Each deployment has a unique authentication signature

### MSBuild Integration

The key generation is seamlessly integrated into the .NET build process using custom MSBuild targets:

```xml
<Target Name="GenerateAuthKey" BeforeTargets="CoreCompile">
    <Exec Command="pwsh -ExecutionPolicy Bypass -File &quot;$(ProjectDir)Generate-AuthKey.ps1&quot;" 
          ContinueOnError="false" />
</Target>

<Target Name="ProcessWorker" AfterTargets="GenerateAuthKey">
    <Exec Command="pwsh -ExecutionPolicy Bypass -File &quot;$(ProjectDir)Process-Worker.ps1&quot;" 
          ContinueOnError="false" />
</Target>
```

## HMAC Authentication Implementation

The authentication system uses HMAC-SHA256 to sign requests, ensuring both authenticity and integrity.

### Client-Side Signing (C#)

```csharp
private string GenerateHmacToken(string requestUrl)
{
    using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(BuildConstants.AuthKey));
    var hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(requestUrl.ToLower()));
    return Convert.ToBase64String(hash);
}
```

### Worker-Side Validation (JavaScript)

```javascript
async function validateHmacToken(receivedToken, requestUrl) {
    const keyBytes = new TextEncoder().encode(AUTH_KEY);
    const messageBytes = new TextEncoder().encode(requestUrl);
    
    const cryptoKey = await crypto.subtle.importKey(
        'raw', keyBytes, 
        { name: 'HMAC', hash: 'SHA-256' }, 
        false, ['sign']
    );
    
    const signature = await crypto.subtle.sign('HMAC', cryptoKey, messageBytes);
    const expectedToken = arrayBufferToBase64(signature);
    
    return expectedToken === receivedToken;
}
```

## Advanced CORS and Security Features

Beyond basic authentication, Albatross implements sophisticated browser-only validation to prevent abuse from non-browser clients like Postman or curl.

### Browser-Only Validation

```javascript
// Enforce browser-only access
const userAgent = request.headers.get('User-Agent') || '';
const isBrowserRequest = origin && (
    userAgent.includes('Mozilla') || 
    userAgent.includes('Chrome') || 
    userAgent.includes('Safari') || 
    userAgent.includes('Firefox') || 
    userAgent.includes('Edge')
);

const hasValidOrigin = origin && isAllowedOrigin(origin);

if (!isBrowserRequest || !hasValidOrigin) {
    return new Response(JSON.stringify({ 
        error: "Access denied: Browser requests from allowed origins only"
    }), { status: 403 });
}
```

### Restrictive CORS Policy

```javascript
function corsHeaders(origin) {
    const headers = {
        'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
        'Access-Control-Allow-Headers': 'Content-Type, Worker-Token, Authorization',
        'Access-Control-Max-Age': '86400',
    };
    
    if (origin && isAllowedOrigin(origin)) {
        headers['Access-Control-Allow-Origin'] = origin;
        headers['Access-Control-Allow-Credentials'] = 'true';
    } else {
        headers['Access-Control-Allow-Origin'] = 'null';
    }
    
    return headers;
}
```

## Automated Deployment Pipeline

The project uses GitHub Actions for comprehensive CI/CD that handles both the SPA and Worker deployments:

### Production Workflow

```yaml
name: Deploy Production
on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Generate Authentication Keys
        run: pwsh Generate-AuthKey.ps1
        
      - name: Build Blazor App
        run: dotnet publish -c Release
        
      - name: Deploy to Cloudflare Pages
        uses: cloudflare/pages-action@v1
        
      - name: Deploy Worker
        run: wrangler publish cloudflare-worker.js
```

## Lessons Learned and Best Practices

### 1. Security Through Obscurity Isn't Enough

While the build-time key generation adds a layer of security, the real protection comes from:
- Proper CORS implementation
- Origin validation
- Browser-only access controls
- Request signing with HMAC

### 2. PowerShell for Cross-Platform Build Scripts

Using PowerShell Core for build scripts provides:
- Cross-platform compatibility (Windows, macOS, Linux)
- Rich cryptographic APIs
- Easy integration with .NET projects
- Consistent behavior across environments

### 3. Cloudflare Workers Are Perfect for API Proxies

The combination of:
- Global edge deployment
- Zero cold starts
- Built-in security features
- Excellent developer experience

Makes Cloudflare Workers ideal for API proxy scenarios.

### 4. MSBuild Custom Targets for Complex Builds

Custom MSBuild targets allow for:
- Seamless integration with the .NET build process
- Automatic execution during development and CI/CD
- Consistent behavior across different environments
- Easy maintenance and updates

### 5. AI-Powered Development with GitHub Copilot

Building Albatross was significantly accelerated by leveraging **GitHub Copilot powered by Claude Sonnet**. The AI assistant proved invaluable for:

- **Complex Algorithm Implementation**: Generated efficient CIDR matching logic and HMAC authentication code
- **Security Pattern Recognition**: Suggested best practices for CORS implementation and origin validation
- **PowerShell Automation**: Helped create robust build scripts with proper error handling
- **Architecture Decisions**: Provided insights on Cloudflare Worker optimization and WebAssembly performance
- **Documentation**: Assisted in writing comprehensive code comments and technical documentation

The combination of human creativity and AI-powered code generation allowed for rapid prototyping and implementation of sophisticated security features that might have taken significantly longer to develop manually. Claude Sonnet's deep understanding of security patterns and modern web architecture made it an ideal pair programming partner for this project.

## Performance and Monitoring

The application includes comprehensive logging and monitoring:

```javascript
console.log('HMAC validation:', {
    message: message.substring(0, 100) + '...',
    expectedToken: expectedToken.substring(0, 20) + '...',
    receivedToken: receivedToken.substring(0, 20) + '...',
    authKeySource: BUILD_INFO.keySource
});
```

Response times are consistently under 100ms thanks to:
- Cloudflare's global edge network
- Efficient HMAC validation
- Minimal payload sizes
- Strategic caching headers

## Future Enhancements

Several exciting features are planned for future releases:

1. **Rate Limiting**: Implement per-IP rate limiting in the Worker
2. **Analytics Dashboard**: Track usage patterns and abuse attempts
3. **API Key Rotation**: Automated rotation of AbuseIPDB API keys
4. **Multi-Provider Support**: Integration with additional threat intelligence APIs
5. **Real-Time Notifications**: WebSocket-based alerts for high-risk IPs

## Cloud IP Manifest Search: Identifying Cloud Infrastructure

One of Albatross's most valuable features is its **Cloud IP Manifest Search** functionality, which allows users to identify whether any IP address belongs to major cloud service providers. This feature leverages official IP range manifests from AWS, Microsoft Azure, and Google Cloud Platform to provide accurate cloud infrastructure attribution.

### The Challenge of Cloud IP Attribution

In today's cloud-first world, understanding which cloud provider owns a specific IP address is crucial for:

- **Network Security**: Identifying legitimate cloud traffic vs. potential threats
- **Compliance**: Understanding data flow through different cloud jurisdictions  
- **Infrastructure Analysis**: Mapping application dependencies and service architectures
- **Incident Response**: Quickly identifying the source cloud provider during security investigations

### Architecture and Data Sources

The system maintains up-to-date IP range manifests from three major cloud providers:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   AWS IP Ranges │    │ Azure IP Ranges │    │  GCP IP Ranges  │
│                 │    │                 │    │                 │
│ • 76,000+ IPs   │    │ • 135,000+ IPs  │    │ • 3,100+ IPs    │
│ • Regional Data │    │ • Service Tags  │    │ • Scope Data    │
│ • Service Info  │    │ • Platform Info │    │ • Global Ranges │
└─────────────────┘    └─────────────────┘    └─────────────────┘
          │                       │                       │
          └───────────────────────┼───────────────────────┘
                                  │
                          ┌─────────────────┐
                          │  Albatross      │
                          │  Search Engine  │
                          │                 │
                          │ • CIDR Matching │
                          │ • IPv4 & IPv6   │
                          │ • Real-time     │
                          └─────────────────┘
```

### Official Data Sources

#### AWS IP Ranges
- **Source**: `https://ip-ranges.amazonaws.com/ip-ranges.json`
- **Updates**: Multiple times daily
- **Contains**: Service names, regions, network border groups
- **Format**: CIDR blocks with metadata

```json
{
  "ip_prefix": "3.5.140.0/22",
  "region": "ap-northeast-2", 
  "service": "AMAZON",
  "network_border_group": "ap-northeast-2"
}
```

#### Microsoft Azure Service Tags
- **Source**: Dynamic URL from Microsoft Download Center
- **Updates**: Weekly (typically Mondays)
- **Contains**: Service names, regions, platform information
- **Format**: Hierarchical service groupings

```json
{
  "name": "AzureCognitiveSearch.EastUS",
  "properties": {
    "region": "eastus",
    "platform": "Azure",
    "systemService": "AzureCognitiveSearch",
    "addressPrefixes": ["20.42.4.128/26"]
  }
}
```

#### Google Cloud Platform IP Ranges  
- **Source**: `https://www.gstatic.com/ipranges/cloud.json`
- **Updates**: As needed (typically weekly)
- **Contains**: Service types, regional scopes
- **Format**: Simplified CIDR list with scope information

```json
{
  "ipv4Prefix": "34.1.208.0/20",
  "service": "Google Cloud", 
  "scope": "africa-south1"
}
```

### Smart Search Algorithm

The cloud IP detection uses a sophisticated CIDR (Classless Inter-Domain Routing) matching algorithm:

```csharp
private bool IsIpInRange(IPAddress ipAddress, string cidrRange)
{
    if (string.IsNullOrEmpty(cidrRange) || !cidrRange.Contains('/'))
        return false;
        
    try
    {
        var parts = cidrRange.Split('/');
        var networkAddress = IPAddress.Parse(parts[0]);
        var prefixLength = int.Parse(parts[1]);
        
        // Convert to bytes for bitwise operations
        var ipBytes = ipAddress.GetAddressBytes();
        var networkBytes = networkAddress.GetAddressBytes();
        
        // Handle IPv4 vs IPv6 compatibility
        if (ipBytes.Length != networkBytes.Length)
            return false;
            
        // Calculate subnet mask
        var totalBits = ipBytes.Length * 8;
        var maskBits = prefixLength;
        
        // Perform bitwise comparison
        for (int byteIndex = 0; byteIndex < ipBytes.Length; byteIndex++)
        {
            var bitsInThisByte = Math.Min(8, Math.Max(0, maskBits - (byteIndex * 8)));
            if (bitsInThisByte == 0) break;
            
            var mask = (byte)(0xFF << (8 - bitsInThisByte));
            if ((ipBytes[byteIndex] & mask) != (networkBytes[byteIndex] & mask))
                return false;
        }
        
        return true;
    }
    catch
    {
        return false;
    }
}
```

### Real-Time Search Implementation

The search process efficiently scans through hundreds of thousands of IP ranges:

```csharp
private async Task ValidateAndSearch()
{
    // Parse and validate input IP
    IPAddress enteredIp = IPAddress.Parse(ipAddress);
    bool foundMatch = false;
    
    // Search Azure IP ranges (135,000+ entries)
    var azureData = await Http.GetStringAsync("ip-manifests/Azure.json");
    var azureRanges = JsonSerializer.Deserialize<AzureIpRanges>(azureData);
    
    foreach (var value in azureRanges.values)
    {
        foreach (var prefix in value.properties.addressPrefixes)
        {
            if (IsIpInRange(enteredIp, prefix))
            {
                azureMatchedServices.Add(value.name);
                foundMatch = true;
            }
        }
    }
    
    // Search AWS IP ranges (76,000+ entries)  
    var awsData = await Http.GetStringAsync("ip-manifests/AWS.json");
    var awsRanges = JsonSerializer.Deserialize<AwsIpRanges>(awsData);
    
    foreach (var prefix in awsRanges.prefixes)
    {
        if (IsIpInRange(enteredIp, prefix.ip_prefix))
        {
            string serviceInfo = $"{prefix.service} ({prefix.region})";
            awsMatchedServices.Add(serviceInfo);
            foundMatch = true;
        }
    }
    
    // Search GCP IP ranges (3,100+ entries)
    var gcpData = await Http.GetStringAsync("ip-manifests/GCP.json");
    var gcpRanges = JsonSerializer.Deserialize<GcpIpRanges>(gcpData);
    
    foreach (var prefix in gcpRanges.prefixes) 
    {
        string ipRange = prefix.ipv4Prefix ?? prefix.ipv6Prefix;
        if (IsIpInRange(enteredIp, ipRange))
        {
            string serviceInfo = $"{prefix.service} ({prefix.scope ?? "Global"})";
            gcpMatchedServices.Add(serviceInfo);
            foundMatch = true;
        }
    }
}
```

### Automated Manifest Updates

To ensure data freshness, Albatross includes an automated update system:

```bash
#!/bin/sh
echo "🌐 Updating cloud IP manifests..."

# Fetch AWS IP ranges
curl -sSL "https://ip-ranges.amazonaws.com/ip-ranges.json" \
     -o ./wwwroot/ip-manifests/AWS.json

# Fetch GCP IP ranges  
curl -sSL "https://www.gstatic.com/ipranges/cloud.json" \
     -o ./wwwroot/ip-manifests/GCP.json

# Fetch Azure IP ranges (dynamic URL discovery)
curl -sSL "https://www.microsoft.com/en-us/download/confirmation.aspx?id=56519" \
     > /tmp/azure_page.html
AZURE_URL=$(grep -o 'https://download.microsoft.com/download/[^"]*ServiceTags[^"]*\.json' \
           /tmp/azure_page.html | head -1)
curl -sSL "$AZURE_URL" -o ./wwwroot/ip-manifests/Azure.json
```

### Performance Optimizations

Despite searching through 200,000+ IP ranges, the system maintains excellent performance:

1. **Client-Side Processing**: All searches run in the browser using WebAssembly
2. **Efficient Algorithms**: Optimized CIDR matching with early termination
3. **Cached Manifests**: Static JSON files served via CDN
4. **Parallel Processing**: Simultaneous searches across all three providers
5. **Smart Indexing**: Future enhancement for sub-second responses

### Example Search Results

When searching for an IP like `20.42.4.150`, the system returns:

**Azure Results:**
- `AzureCognitiveSearch.EastUS`
- `Storage.EastUS` 
- `AzureCloud.EastUS`

**AWS Results:**
- No matches found

**GCP Results:**  
- No matches found

### Security and Privacy Benefits

The cloud IP search functionality provides significant security value:

1. **Threat Attribution**: Quickly identify if suspicious traffic originates from legitimate cloud infrastructure
2. **Network Forensics**: Understand traffic patterns and service dependencies  
3. **Compliance Validation**: Verify that data flows through expected cloud regions
4. **Incident Response**: Rapidly classify IP addresses during security investigations

### Future Enhancements

Several exciting improvements are planned:

1. **Additional Providers**: Oracle Cloud, IBM Cloud, Alibaba Cloud support
2. **Historical Data**: Track IP range changes over time
3. **Geolocation Integration**: Combine cloud attribution with geographic data
4. **API Integration**: Programmatic access to cloud IP attribution
5. **Bulk Processing**: Upload and process IP lists

The Cloud IP Manifest Search feature demonstrates how modern web applications can provide enterprise-grade functionality while maintaining simplicity and performance. By leveraging official cloud provider data and efficient client-side processing, Albatross delivers accurate, real-time cloud infrastructure attribution that's invaluable for security professionals and network administrators.

## Conclusion

Building Albatross has been an incredible journey into modern web security architecture. The combination of Blazor WebAssembly's rich client-side capabilities with Cloudflare Workers' edge computing power creates a robust, secure, and performant solution for IP abuse checking.

The key innovations - build-time key generation, HMAC authentication, and browser-only validation - demonstrate that security doesn't have to come at the expense of user experience or developer productivity.

### Key Takeaways:

- **Security by Design**: Build security considerations into the architecture from day one
- **Automation is Key**: Automated key generation and deployment reduces human error
- **Edge Computing**: Leverage edge networks for better performance and security
- **Modern Tooling**: Use the best tools for each layer of the stack

## Try It Yourself

The complete source code for Albatross is available on GitHub, including:
- All PowerShell build scripts
- MSBuild integration examples
- Cloudflare Worker implementation
- GitHub Actions workflows

Whether you're building your own API proxy or exploring modern web security patterns, I hope Albatross serves as both inspiration and a practical reference for your next project.

---

*Albatross is live at [https://albatross.devnomadic.com](https://albatross.devnomadic.com) and the source code is available on [GitHub](https://github.com/your-username/albatross).*