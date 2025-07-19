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
- �️ **Real-time IP reputation checking** via AbuseIPDB
- 🏗️ **Modern architecture** combining the best of client and edge computing

This algorithm efficiently compares IP addresses against CIDR ranges by:
1. **Splitting the CIDR** notation (e.g., "10.0.0.0/8")
2. **Converting to bytes** for efficient comparison
3. **Comparing full bytes** first (faster than bit-by-bit)
4. **Handling remainder bits** with bitwise masking
5. **Early termination** on mismatch for performance

## Security Implementationion (no exposed API keys)
- ⚡ **Fast client-side app** with server-side API protection
- 🛡️ **Real-time IP reputation checking** via AbuseIPDB
- 🏗️ **Modern architecture** combining the best of client and edge computing

**Tech Stack:** Blazor WASM, Cloudflare Workers, HMAC-SHA256, AbuseIPDB API, Cloudflare Radar API

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
# Generate-AuthKey.ps1 - Updated implementation
function Generate-SecureKey {
    # Generate a cryptographically secure random key
    $keyBytes = New-Object byte[] 32  # 256 bits
    $rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
    $rng.GetBytes($keyBytes)
    
    # Convert to Base64 and make UTF-8 compatible
    $randomBase64 = [System.Convert]::ToBase64String($keyBytes)
    $authKeyString = $randomBase64.Substring(0, [Math]::Min(32, $randomBase64.Length)).Replace("/", "_").Replace("+", "-")
    
    # Convert UTF-8 string to base64 for storage
    $authKeyBytes = [System.Text.Encoding]::UTF8.GetBytes($authKeyString)
    return [System.Convert]::ToBase64String($authKeyBytes)
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
<Target Name="GenerateAuthKey" BeforeTargets="BeforeCompile" Condition="'$(SkipCodeGeneration)' != 'true' AND '$(DesignTimeBuild)' != 'true'">
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
    if (string.IsNullOrEmpty(_authKey))
    {
        Console.WriteLine("Debug: Auth key is null or empty");
        return string.Empty;
    }

    if (string.IsNullOrEmpty(requestUrl))
    {
        Console.WriteLine("Debug: Request URL is null or empty");
        return string.Empty;
    }

    try
    {
        Console.WriteLine($"Debug: Generating HMAC for URL: {requestUrl}");
        Console.WriteLine($"Debug: Using auth key: {_authKey}");

        using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(_authKey));
        var hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(requestUrl));
        var token = Convert.ToBase64String(hash);
        
        Console.WriteLine($"Debug: Generated HMAC hash: {token}");
        return token;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Debug: Exception in GenerateHmacToken: {ex.Message}");
        return string.Empty;
    }
}
```

### Worker-Side Validation (JavaScript)

```javascript
async function validateHmacToken(receivedToken, requestUrl) {
    const keyBytes = new TextEncoder().encode(AUTH_KEY);
    const messageBytes = new TextEncoder().encode(message);
    
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
5. **Gen-AI Search**: AI-powered search for IP reputation and abuse history
6. **Real-Time Notifications**: WebSocket-based alerts for high-risk IPs

## Cloud IP Manifest Search: Identifying Cloud Infrastructure

One of Albatross's most valuable features is its **Cloud IP Manifest Search** functionality, which allows users to identify whether any IP address belongs to major cloud service providers. This feature leverages official IP range manifests from AWS, Microsoft Azure, and Google Cloud Platform to provide accurate cloud infrastructure attribution.

### The Challenge of Cloud IP Attribution

In today's cloud-first world, understanding which cloud provider owns a specific IP address is crucial for:

- **Network Security**: Identifying legitimate cloud traffic vs. potential threats
- **Compliance**: Understanding data flow through different cloud jurisdictions  
- **Infrastructure Analysis**: Mapping application dependencies and service architectures
- **Incident Response**: Quickly identifying the source cloud provider during security investigations

### Architecture and Data Sources

The system maintains up-to-date IP range manifests from four major cloud providers:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   AWS IP Ranges │    │ Azure IP Ranges │    │  GCP IP Ranges  │    │  OCI IP Ranges  │
│                 │    │                 │    │                 │    │                 │
│ • 76,000+ IPs   │    │ • 135,000+ IPs  │    │ • 3,100+ IPs    │    │ • 2,800+ IPs    │
│ • Regional Data │    │ • Service Tags  │    │ • Scope Data    │    │ • Regional Tags │
│ • Service Info  │    │ • Platform Info │    │ • Global Ranges │    │ • Service Data  │
└─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘
          │                       │                       │                       │
          └───────────────────────┼───────────────────────┼───────────────────────┘
                                  │                       │
                          ┌─────────────────┐             │
                          │  Albatross      │             │
                          │  Search Engine  │             │
                          │                 │             │
                          │ • CIDR Matching │             │
                          │ • IPv4 & IPv6   │             │
                          │ • Real-time     │             │
                          │ • Parallel Exec │←────────────┘
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

#### Oracle Cloud Infrastructure IP Ranges
- **Source**: `https://docs.oracle.com/en-us/iaas/tools/public_ip_ranges.json`
- **Updates**: Regularly updated by Oracle
- **Contains**: Regional data, service tags, CIDR blocks
- **Format**: Regional structure with comprehensive metadata

```json
{
  "region": "us-ashburn-1",
  "cidr": "129.213.0.0/16",
  "tags": ["OCI", "OracleServices", "us-ashburn-1"]
}
```

### Enhanced Cloud Provider Coverage

With the recent addition of **Oracle Cloud Infrastructure (OCI)** support, Albatross now provides comprehensive coverage of the four major cloud providers:

- **Amazon Web Services (AWS)**: 76,000+ IP ranges
- **Microsoft Azure**: 135,000+ IP ranges  
- **Google Cloud Platform (GCP)**: 3,100+ IP ranges
- **Oracle Cloud Infrastructure (OCI)**: 2,800+ IP ranges

This expansion significantly improves the tool's ability to identify cloud-hosted infrastructure across the enterprise landscape.

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
        
        var ipBytes = ipAddress.GetAddressBytes();
        var networkBytes = networkAddress.GetAddressBytes();
        
        if (ipBytes.Length != networkBytes.Length)
            return false;
        
        var bytesToCheck = prefixLength / 8;
        var remainderBits = prefixLength % 8;
        
        // Compare full bytes
        for (int i = 0; i < bytesToCheck; i++)
        {
            if (ipBytes[i] != networkBytes[i])
                return false;
        }
        
        // Check remainder bits if any
        if (remainderBits > 0 && bytesToCheck < ipBytes.Length)
        {
            var mask = (byte)(0xFF << (8 - remainderBits));
            if ((ipBytes[bytesToCheck] & mask) != (networkBytes[bytesToCheck] & mask))
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

# Fetch Oracle IP ranges
curl -sSL "https://docs.oracle.com/en-us/iaas/tools/public_ip_ranges.json" \
     -o ./wwwroot/ip-manifests/Oracle.json

# Fetch Azure IP ranges (dynamic URL discovery)
curl -sSL "https://www.microsoft.com/en-us/download/confirmation.aspx?id=56519" \
     > /tmp/azure_page.html
AZURE_URL=$(grep -o 'https://download.microsoft.com/download/[^"]*ServiceTags[^"]*\.json' \
           /tmp/azure_page.html | head -1)
curl -sSL "$AZURE_URL" -o ./wwwroot/ip-manifests/Azure.json
```

### Performance Optimizations

Despite searching through 217,000+ IP ranges across four cloud providers, the system maintains excellent performance:

1. **Client-Side Processing**: All searches run in the browser using WebAssembly
2. **Efficient Algorithms**: Optimized CIDR matching with early termination
3. **Cached Manifests**: Static JSON files served via CDN
4. **Parallel Processing**: Simultaneous searches across all four providers
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

## Recent Enhancements (July 2025)

Since the June updates, Albatross has received several critical improvements focused on deployment reliability, security hardening, and cross-browser compatibility:

### Static Sitemap Implementation

A significant architectural change was made to improve SEO and deployment reliability:

- **Removed Dynamic Generation**: Eliminated all dynamic sitemap.xml generation from MSBuild targets and PowerShell scripts
- **Static SEO Optimization**: Implemented a manually-maintained sitemap.xml with fixed URLs and appropriate metadata
- **Improved Indexing**: Better search engine discoverability with consistent sitemap structure
- **Simplified Deployment**: Reduced build complexity by removing dynamic XML generation

The static sitemap approach provides more reliable SEO benefits while simplifying the build process:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://albatross.devnomadic.com/</loc>
    <lastmod>2025-07-12</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
  <!-- IP Manifest Files -->
  <url>
    <loc>https://albatross.devnomadic.com/ip-manifests/AWS.json</loc>
    <lastmod>2025-07-12</lastmod>
    <changefreq>daily</changefreq>
    <priority>0.8</priority>
  </url>
</urlset>
```

### Cross-Browser ASCII Art Rendering

Addressed significant rendering differences between Chrome and Firefox for ASCII art images:

- **Pixel-Perfect Rendering**: Added comprehensive `image-rendering` CSS properties for consistent display across browsers
- **Cross-Browser Compatibility**: Implemented browser-specific rendering hints for Chrome, Firefox, Safari, and IE
- **Responsive Scaling**: Enhanced mobile responsiveness with proper pixel scaling for ASCII art
- **Performance Optimization**: Eliminated image interpolation artifacts that caused blurry ASCII art

```css
img[alt="ascii-text-art-albatross"] {
    width: 25% !important;
    max-width: 40% !important;
    /* Pixel-perfect rendering for ASCII art */
    image-rendering: pixelated !important;
    image-rendering: -moz-crisp-edges !important;
    image-rendering: crisp-edges !important;
    image-rendering: -webkit-optimize-contrast !important;
    -ms-interpolation-mode: nearest-neighbor !important;
}
```

### Production Deployment Security Hardening

Major security improvements were implemented in the GitHub Actions deployment pipeline:

#### **Fixed Script Injection Vulnerability**
The deployment pipeline was vulnerable to script injection through commit messages. This was resolved with comprehensive input sanitization:

```bash
# Safely sanitize commit message to prevent injection
SAFE_COMMIT_MESSAGE="$( printf '%s' '${{ github.event.head_commit.message }}' | head -c 500 | tr -d '\0\r' | sed 's/[`$"\\]/\\&/g' )"
```

**Security Controls:**
- **Character Limiting**: Truncated to 500 characters to prevent excessive payloads
- **Dangerous Character Escaping**: Escaped backticks, dollar signs, quotes, and backslashes
- **Null Byte Removal**: Eliminated null characters and carriage returns
- **Command Injection Prevention**: Blocked all forms of command substitution and variable expansion

#### **Enhanced Release Naming**
Improved GitHub release naming to match application build identifiers:

- **Build Timestamp Format**: Uses the same `yyyyMMdd-HHmm` format displayed in the application
- **Short SHA Integration**: Includes 8-character commit SHA for unique identification
- **Consistent Versioning**: Release names now match internal build identifiers

```yaml
RELEASE_NAME="${{ needs.build-and-deploy.outputs.build-timestamp }} (${{ needs.build-and-deploy.outputs.build-id }})"
# Example: "Albatross Build 20250719-1430 (a1b2c3d4)"
```

#### **Cloudflare Pages Production Deployment Fix**
Resolved deployment issues where builds were incorrectly deploying to preview environments:

- **Removed Branch Parameters**: Eliminated `--branch` and `--production` flags that were causing deployment confusion  
- **Default Production Behavior**: Leveraged Cloudflare Pages' default production deployment when no branch is specified
- **Simplified Command**: Streamlined deployment command for reliability

```bash
# Simplified production deployment (deploys to production by default)
npx wrangler pages deploy ./dist/wwwroot --project-name ${{ secrets.CLOUDFLARE_PAGES_PROJECT }}
```

#### **Artifact Security Enhancement**
Improved release artifact handling to exclude sensitive files:

- **Excluded Worker Files**: Prevented `cloudflare-worker.js` from being included in public releases
- **Security Logging**: Added clear audit trail showing which files are excluded and why
- **Compressed Archives**: Enhanced artifact upload with compressed directory archives
- **Selective Distribution**: Only includes safe artifacts (SPA files, build constants, version info)

```bash
# Skip sensitive files during artifact upload
if [ -f "$file" ] && [ "$file" != "cloudflare-worker.js" ]; then
    echo "Uploading file: $file"
    gh release upload "${TAG_NAME}" "$file"
elif [ -f "$file" ] && [ "$file" = "cloudflare-worker.js" ]; then
    echo "Skipping cloudflare-worker.js (contains sensitive data)"
fi
```

### Mobile Loading Screen Fix

Addressed visual issues with the loading screen on mobile devices:

- **Fixed Icon Stretching**: Resolved aspect ratio issues on mobile screens
- **Enhanced CSS Specificity**: Added `!important` declarations to prevent style conflicts
- **Responsive Sizing**: Proper scaling for different device sizes (80px desktop, 64px tablet, 48px mobile)
- **Conflict Resolution**: Changed alt text to prevent generic image style conflicts

### Build System Reliability

Multiple improvements to the build and deployment pipeline:

- **Dependency Management**: Enhanced npm package handling in CI/CD
- **Error Handling**: Improved build script error detection and reporting  
- **Version Consistency**: Aligned version numbering between application and releases
- **Deployment Validation**: Added verification steps for successful deployments

## Recent Enhancements (June 2025)

Since the initial release, Albatross has received several significant improvements that enhance both functionality and user experience:

### Oracle Cloud Infrastructure (OCI) Support

The most substantial addition is comprehensive **Oracle Cloud Infrastructure** support, making Albatross the first tool to provide unified IP range detection across all four major cloud providers:

- **Complete Coverage**: Official OCI IP ranges from Oracle's public API
- **Regional Identification**: Detailed region and service tag information
- **Automatic Updates**: Integrated into the automated manifest update system
- **Enhanced Search**: Parallel searching across AWS, Azure, GCP, and OCI

### Enhanced Data Structures

The search functionality has been significantly improved with new **CloudMatch** objects:

```csharp
public class CloudMatch 
{
    public string ServiceName { get; set; }
    public string Region { get; set; }
    public string Provider { get; set; }
    public string CidrRange { get; set; }
    public List<string> Tags { get; set; }
}
```

This structured approach provides:
- **Richer Metadata**: More detailed information about matched services
- **Consistent Interface**: Uniform data structure across all cloud providers
- **Better Performance**: Optimized search and display logic
- **Enhanced UX**: Improved visual presentation of results

### ASN Integration with Cloudflare Radar

New integration with **Cloudflare Radar API** provides Autonomous System Number (ASN) information:

- **ASN Lookup**: Automatic ASN resolution for searched IP addresses
- **Organization Details**: Company and ISP information
- **Enhanced Context**: Additional intelligence for security analysis
- **Real-time Data**: Live ASN data from Cloudflare's global network

### Improved User Interface

The search results display has been enhanced with:
- **Visual Service Cards**: Better organized and styled result presentation
- **Provider Icons**: Clear visual identification of cloud providers
- **Enhanced Metadata**: More detailed service and regional information
- **Responsive Design**: Improved mobile and desktop experience

### Build System Improvements

Recent build system enhancements include:
- **Fixed Solution Build Issues**: Resolved `--output` parameter conflicts in CI/CD
- **Enhanced Logging**: Better debugging and deployment visibility
- **Improved Error Handling**: More robust build script error management
- **Streamlined Build Process**: Improved consistency across development and CI/CD environments

### Enhanced Security and Input Validation (June 2025)

Recent security and user experience improvements include:

- **Public IP Validation**: Comprehensive validation that only accepts public routable IP addresses, blocking all private networks (10.x.x.x, 192.168.x.x, 172.16-31.x.x), loopback addresses (127.x.x.x), link-local, multicast, and other reserved ranges according to RFC standards
- **Improved Error Messaging**: Clear, user-friendly error messages explaining why certain IP addresses aren't accepted
- **Better UI Layout**: Fixed layout shifting issues during search operations with dedicated result containers
- **Enhanced User Guidance**: Updated placeholder text and help documentation to clarify IP address requirements
- **RFC Compliance**: Full compliance with RFC 1918 (Private Address Space), RFC 3927 (Link-Local), RFC 6598 (Carrier-Grade NAT), and other relevant networking standards

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

## 📋 Changelog

- **2025-07-19:** Major deployment and security update with static sitemap implementation, cross-browser ASCII art rendering fixes, script injection vulnerability patching, Cloudflare Pages production deployment fixes, mobile loading screen improvements, and enhanced release artifact security
- **2025-06-22:** Enhanced security with comprehensive public IP validation (RFC compliant), improved UI layout stability, better error messaging, and enhanced user guidance
- **2025-06-16:** Major update with Oracle Cloud Infrastructure support, enhanced CloudMatch data structures, ASN integration with Cloudflare Radar API, improved UI/UX, and build system fixes
- **2025-06-15:** Updated tag format and added changelog  
- **2025-06-10:** Initial publication

---

*Albatross is live at [https://albatross.devnomadic.com](https://albatross.devnomadic.com) and the source code is available on [GitHub](https://github.com/devnomadic/albatross).*