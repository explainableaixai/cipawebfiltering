# CIPA Web Filtering Database, Node.js Client

A Node.js module for integrating the [cipa web filtering](https://www.cipawebfiltering.com) domain classification database into school, library, and district web filtering infrastructure. The database contains over 120 million domains classified across 57+ content categories and 165+ subcategories specifically designed to meet the requirements of the Children's Internet Protection Act (CIPA).

CIPA is a United States federal law enacted in 2000 that requires schools and libraries receiving E-Rate funding or LSTA grants to implement internet safety policies and technology protection measures. The [FCC's CIPA compliance guide](https://www.fcc.gov/consumers/guides/childrens-internet-protection-act) outlines these requirements in detail, and the [American Library Association's CIPA resource page](https://www.ala.org/advocacy/advleg/federallegislation/cipa) provides practical implementation guidance for institutions. This package provides programmatic access to a domain classification database built specifically to satisfy those filtering mandates.

## Why CIPA-Specific Filtering Categories Matter

General-purpose domain taxonomies like IAB Content Taxonomy are designed for advertising and brand safety, they categorize content into verticals like "Automotive" and "Personal Finance" that have no relevance to student safety. CIPA compliance requires filtering three specific categories of content: material that is obscene, child sexual abuse material (CSAM), and content harmful to minors.

Our database maps every domain against these CIPA-mandated categories while also providing 57+ granular content labels that give school IT administrators fine-grained policy control. A single domain can carry multiple labels, for example, a social media platform might be tagged as "Social Media", "User-Generated Content", and "Messaging" simultaneously, allowing policy engines to make nuanced decisions by grade band rather than applying binary allow/block verdicts across the entire student body.

The database also includes 18 dedicated AI-tool categories with 165+ subcategories that track essay writers, homework solvers, code generators, image generators, deepfake and face-swap tools (200+ domains), voice cloning services (250+ domains), and AI companion chat applications (470+ domains). This granularity is critical for schools implementing acceptable-use policies around AI, a policy might allow Khan Academy's AI tutor while blocking consumer chatbots and deepfake generators.

## Installation

```
npm i cipawebfiltering
```

## Quick Start

```javascript
const CIPAWebFilter = require('cipawebfiltering');

const filter = new CIPAWebFilter('your_api_key');

// Check API key status and subscription details
filter.status().then(result => {
  console.log(result);
});
```

## API Endpoints

The database is accessible through a REST API that supports the following operations:

| Action | Description |
|--------|-------------|
| `status` | Check API key validity and subscription details |
| `database_info` | Get database metadata, last updated date, domain count, file size |
| `download_database` | Download the full categorized domain database as CSV |
| `download_categories` | Download the current category taxonomy as CSV |
| `download_changelog` | Download the category changelog showing recent updates |

All endpoints authenticate via API key passed as a query parameter or `X-API-Key` header.

## Usage Examples

### Check subscription status (curl)

```
curl -X GET "https://www.cipawebfiltering.com/api/?action=status&api_key=your_api_key"
```

Example response:

```json
{
  "status": "active",
  "plan": "district",
  "domains_covered": 120482391,
  "categories": 57,
  "subcategories": 165,
  "last_updated": "2026-07-31",
  "daily_updates": true,
  "api_calls_remaining": "unlimited"
}
```

### Get database metadata (curl)

```
curl -X GET "https://www.cipawebfiltering.com/api/?action=database_info&api_key=your_api_key"
```

Example response:

```json
{
  "database_version": "2026.07.31",
  "total_domains": 120482391,
  "file_size_mb": 3847,
  "last_updated": "2026-07-31T04:30:00Z",
  "categories_count": 57,
  "subcategories_count": 165,
  "format": "csv",
  "columns": ["domain", "categories", "subcategories", "confidence"]
}
```

### Download category taxonomy (Node.js)

```javascript
const CIPAWebFilter = require('cipawebfiltering');
const fs = require('fs');

const filter = new CIPAWebFilter('your_api_key');

filter.downloadCategories().then(data => {
  fs.writeFileSync('cipa_categories.csv', data);
  console.log('Categories downloaded successfully');
});
```

### Download full database (Node.js)

```javascript
const https = require('https');
const fs = require('fs');

const apiKey = 'your_api_key';
const url = `https://www.cipawebfiltering.com/api/?action=download_database&api_key=${apiKey}`;

const file = fs.createWriteStream('cipa_domains.csv');
https.get(url, (response) => {
  response.pipe(file);
  file.on('finish', () => {
    file.close();
    console.log('Database downloaded, 120M+ domains');
  });
});
```

### Check subscription status (Python)

```python
import http.client
import json

conn = http.client.HTTPSConnection("www.cipawebfiltering.com")
conn.request("GET", "/api/?action=status&api_key=your_api_key")

response = conn.getresponse()
data = json.loads(response.read().decode())
print(json.dumps(data, indent=2))
conn.close()
```

### Download database (Python)

```python
import http.client

conn = http.client.HTTPSConnection("www.cipawebfiltering.com")
conn.request("GET", "/api/?action=download_database&api_key=your_api_key")

response = conn.getresponse()
with open("cipa_domains.csv", "wb") as f:
    f.write(response.read())

print("Database downloaded successfully")
conn.close()
```

## Database Schema

The downloaded CSV contains four columns per domain:

| Column | Description |
|--------|-------------|
| `domain` | The fully qualified domain name |
| `categories` | Pipe-separated list of category labels |
| `subcategories` | Pipe-separated list of subcategory labels |
| `confidence` | Classification confidence score (0.0-1.0) |

Example rows from the database:

```
domain,categories,subcategories,confidence
reddit.com,"Social Media|User-Generated Content|Forums","Social Networking|Content Sharing|Discussion Boards",0.97
chatgpt.com,"AI Tools|Conversational AI","AI Chatbots|Text Generation",0.99
roblox.com,"Gaming|Social Media|User-Generated Content","Online Games|Virtual Worlds|Game Chat",0.96
pornhub.com,"Adult Content|Obscene Material","Pornography|Explicit Video",0.99
khanacademy.org,"Education|Reference","Online Learning|Educational Video|Tutorials",0.98
discord.com,"Social Media|Messaging|Gaming","Chat Applications|Voice Chat|Game Communities",0.95
```

## Content Categories

The database organizes domains into 57+ categories specifically aligned to CIPA compliance requirements. The three CIPA-mandated filtering categories, obscene material, child sexual abuse material, and content harmful to minors, are mapped directly, while additional categories provide granular control for school acceptable-use policies.

Core categories include:

- **Adult Content / Obscene Material**, pornography, explicit imagery, adult services
- **Violence / Gore**, graphic violence, weapons, self-harm content
- **Drugs / Alcohol / Tobacco**, substance-related content, drug marketplaces
- **Gambling**, online casinos, sports betting, lottery sites
- **Malware / Phishing**, confirmed malicious domains, credential harvesting
- **Proxies / Anonymizers**, VPN services, web proxies, circumvention tools
- **Social Media**, social networking platforms, messaging services
- **Video / Streaming**, video hosting, live streaming, media platforms
- **Gaming**, online games, game downloads, gaming communities
- **AI Tools**, chatbots, code generators, image generators, deepfake tools
- **Education / Reference**, schools, libraries, educational resources
- **News / Media**, news outlets, journalism, opinion sites
- **Shopping / E-Commerce**, online retail, marketplaces, product listings
- **User-Generated Content**, wikis, forums, content-sharing platforms

The full taxonomy with all 57 categories and 165+ subcategories is available via the `download_categories` API endpoint.

## AI Tools Filtering

A dedicated section of the database tracks over 16,900 AI-tool domains organized into 18 functional categories. Schools implementing AI acceptable-use policies can write granular rules, for example, allowing educational AI tutors while blocking consumer chatbots, image generators, and deepfake tools.

AI tool subcategories include:

- **AI Chatbots**, consumer conversational AI services
- **AI Code Assistants**, code generation and completion tools
- **AI Image Generators**, text-to-image and image editing services
- **AI Video Generators**, text-to-video and video editing AI
- **AI Voice Cloning**, voice synthesis and cloning services (250+ domains)
- **AI Companions**, AI companion and character chat apps (470+ domains)
- **AI Essay Writers**, essay generation and paraphrasing tools
- **AI Homework Solvers**, automated homework and test answer services
- **Deepfake Tools**, face-swap and deepfake generation services (200+ domains)

This AI-tool classification is rebuilt daily, screening approximately 300,000 newly registered domains every 24 hours to capture new tools as they launch. The same AI tools data is available as a standalone product, our [database of AI SaaS domains and their data-use terms](https://www.aitoolsblocklist.com), for organizations that need AI filtering without the full CIPA database. The [NIST AI Risk Management Framework](https://www.nist.gov/artificial-intelligence) provides additional guidance on governing AI tool usage within organizations.

## Integration Patterns

The database supports multiple deployment models depending on your filtering infrastructure:

**DNS-Level Filtering**, Import the CSV into your DNS resolver (BIND RPZ, Pi-hole, AdGuard, Technitium) to block categorized domains at the network level before traffic reaches student devices.

**Proxy Server / Secure Web Gateway**, Load the database into Squid, Blue Coat, Zscaler, or other proxy solutions to enforce per-category policies with full URL visibility.

**Firewall EDL (External Dynamic List)**, Palo Alto Networks, Fortinet, and other NGFW platforms support external domain lists that can be refreshed daily from the database download.

**On-Device Filtering**, For 1:1 Chromebook or laptop deployments, the PAC file and hosts file formats enable filtering that follows students off-campus, meeting CIPA requirements for devices used outside the school network. The [FCC's E-Rate program](https://www.fcc.gov/consumers/guides/childrens-internet-protection-act) requires that CIPA protections extend to all school-owned devices regardless of location.

## Delivery Formats

| Format | Use Case |
|--------|----------|
| REST API | Real-time domain lookups for inline filtering |
| CSV Download | Bulk import into local databases, firewalls, DNS resolvers |
| DNS/RPZ Blocklist | Direct import into BIND, Unbound, Pi-hole, AdGuard |
| PAC File | Browser-level filtering for managed devices |
| Hosts File | OS-level blocking for endpoint devices |
| Firewall EDL | External dynamic list for Palo Alto, Fortinet NGFW |

## Multi-Label Classification

Unlike binary blocklists that return a single allow/block verdict, this database assigns every applicable category to each domain. This enables school districts to implement differentiated policies by grade level.

For example, YouTube might carry labels "Video", "Streaming", "User-Generated Content", and "Education". An elementary school policy could block all four labels, while a high school policy allows "Education"-labeled streaming during class hours. The filtering decision is made by the local policy engine, the database provides the classification data.

## CIPA Compliance Checklist

Schools and libraries must meet several requirements to maintain CIPA compliance and E-Rate eligibility:

1. **Internet Safety Policy**, Adopt and enforce a policy that addresses access to visual depictions that are obscene, constitute child pornography, or are harmful to minors
2. **Technology Protection Measures**, Deploy filtering technology on all computers with internet access, including staff machines
3. **Public Notice and Hearing**, Provide reasonable public notice and hold at least one public hearing before adopting the internet safety policy
4. **Off-Campus Coverage**, For 1:1 device programs, filtering must extend to school-owned devices used outside the school network
5. **Monitoring of Online Activities**, For schools receiving E-Rate discounts, monitor the online activities of minors
6. **Annual Certification**, File annual CIPA certifications with the FCC to maintain E-Rate eligibility

This database directly supports requirements 2 and 4 by providing the classification data that technology protection measures need to make filtering decisions. The multi-format delivery (DNS RPZ, PAC files, hosts files) ensures coverage both on-campus and off-campus.

## Frequently Asked Questions

**How often is the database updated?**
The database rebuilds daily at 04:30 UTC, screening approximately 300,000 newly registered domains in each cycle.

**Can I use this for a single school or does it require a district license?**
Both single-school and district-wide licenses are available. Single schools get API access, while district licenses include full database downloads for on-premise deployment.

**Does the database cover AI tools?**
Yes, 18 dedicated AI-tool categories track 16,900+ domains including chatbots, code generators, image generators, deepfake tools, voice cloners, and AI companion apps, updated daily as new tools launch.

**What deployment formats are supported?**
CSV, REST API, DNS/RPZ blocklists, PAC files, hosts files, and firewall EDL formats. The database integrates with BIND, Pi-hole, AdGuard, Squid, Palo Alto, Fortinet, and other standard filtering infrastructure.

## Update Frequency

The database is rebuilt daily at 04:30 UTC. Each rebuild:

- Screens approximately 300,000 newly registered domains
- Reclassifies domains whose content has changed
- Removes domains that are no longer active
- Adds newly discovered AI tools, proxies, and circumvention services

Daily updates are critical for CIPA compliance because circumvention tools and proxy services appear constantly, a student-shared anonymizer domain registered today will be classified and available in the next database update.

## Our Other Services

For broader URL categorization needs beyond CIPA compliance, including IAB content taxonomy, ad-tech verticals, and e-commerce classifications, the [Website Categorization API](https://www.websitecategorizationapi.com) provides real-time classification across 7 taxonomies with 15+ enrichment data fields.

Organizations that need a general-purpose [enterprise web filtering](https://www.webfilteringdatabase.com) database for corporate environments can deploy 100 million domains across 59 categories in firewalls, secure web gateways, and proxy servers, covering productivity risks and security threats without CIPA-specific category mappings.

A comprehensive [URL categorization database](https://www.urlcategorizationdatabase.com) covering IAB content taxonomy is available for contextual targeting, brand safety, and large-scale analytics where coverage across millions of domains is required.

Security teams can complement content filtering with a [phishing detection API](https://www.phishingdetectionapi.com) that maintains 390,000+ DNS-verified active phishing domains, providing instant verdicts through a single REST endpoint. The [Anti-Phishing Working Group (APWG)](https://apwg.org/trendsreports/) tracks the latest phishing trends and the [CISA phishing guidance](https://www.cisa.gov/secure-our-world/recognize-and-report-phishing) recommends layered domain-level defenses as a frontline measure.

District HR departments hiring teachers and staff can automate applicant intake with our [resume parsing API](http://resumereaderapi.com), an AI document-understanding service that converts PDF and DOCX resumes into structured JSON, contact details, work history, education, and skills, ready for import into applicant tracking systems.

## Useful Resources

- [FCC CIPA Compliance Guide](https://www.fcc.gov/consumers/guides/childrens-internet-protection-act), Federal requirements for E-Rate recipients
- [ALA CIPA Resource Page](https://www.ala.org/advocacy/advleg/federallegislation/cipa), Implementation guidance for schools and libraries
- [NIST AI Risk Management Framework](https://www.nist.gov/artificial-intelligence), Governance framework for AI tool usage
- [OECD AI Policy Observatory](https://oecd.ai/en/), Global AI regulation tracker
- [CoSN (Consortium for School Networking)](https://www.cosn.org), K-12 education technology leadership
- [ISTE (International Society for Technology in Education)](https://www.iste.org), Standards for technology in education

Private equity firms and corporate development teams applying domain classification at scale use our [acquisition target screening platform](https://www.acquisitionuniverse.com) to screen 100M+ domains against custom acquisition theses. The pipeline extracts 15 operational signals per company, leadership bench, recurring revenue indicators, geographic footprint, and compliance readiness, directly from public web presence, surfacing acquisition candidates that keyword-based company databases miss entirely.

The same domain classification infrastructure that powers CIPA filtering also enables privacy-first contextual advertising. Our [cookieless audience intelligence platform](https://www.cookielessaudiences.com) provides pre-classified domain datasets organized by IAB content taxonomy, allowing ad-tech platforms and publishers to build contextual audience segments without relying on third-party cookies. With browser vendors phasing out tracking cookies, contextual classification at the domain level is becoming the primary signal for programmatic targeting and brand-safety enforcement.

## Package Available On

- [https://www.npmjs.com/package/cipawebfiltering](https://www.npmjs.com/package/cipawebfiltering)
- [https://github.com/explainableaixai/cipawebfiltering](https://github.com/explainableaixai/cipawebfiltering)
- [https://www.aitoolsblocklist.com](https://www.aitoolsblocklist.com), AI tools blocklist for school web filtering

Content filtering and data protection usually land on the same desk: the schools and enterprises that filter web traffic under CIPA also handle student and employee records that must stay free of exposed identifiers. A [PII detection API](https://www.piidetectionapi.com) can scan proxy logs, support tickets, and incident reports for names, email addresses, phone numbers, and IDs before they are archived or shared with third parties. Pairing domain filtering with automated [PII redaction](https://www.piidetectionapi.com) lets IT teams cover both content-safety and privacy obligations in a single pipeline.

Content filtering is expanding beyond human users: schools and enterprises now deploy AI agents that browse on their behalf, and those agents need boundaries too. An [AI agent allowlist](https://www.aiagentallowlist.com) provides verified page-type URLs (login, checkout, payment, pricing) so policy engines can decide which pages an autonomous agent may enter before the click. Pairing user-facing web filtering with [agent guardrails data](https://www.aiagentallowlist.com) covers both kinds of traffic your network now carries.

## Related packages

Schools that filter under CIPA usually need three more answers about AI, and there is a package for each:

1. Which AI tools should the filter block? [`aiblocklist`](https://www.npmjs.com/package/aiblocklist) ([PyPI](https://pypi.org/project/aiblocklist/)) queries the [K-12 AI tool blocklist profile](https://www.aitoolsblocklist.com): 20,000+ domains in 18 categories, with a K-12 policy profile that separates approved AI tutors from essay generators, deepfake tools and companion chatbots.
2. Which AI tools are students and staff already reaching? [`shadowaitools`](https://www.npmjs.com/package/shadowaitools) ([PyPI](https://pypi.org/project/shadowaitools/)) reads the filter's own log export and lists them; the hosted [shadow AI discovery tool](https://www.shadowaitools.com) produces the per-user report and PDF for the board.
3. What may the district's own AI assistants open on the web? [`aiagentallowlist`](https://www.npmjs.com/package/aiagentallowlist) ([PyPI](https://pypi.org/project/aiagentallowlist/)) enforces the [AI agent allow list](https://www.aiagentallowlist.com), per-URL verdicts built from verified page types across 40 million+ domains.

See also [`phishingdetectionapi`](https://www.npmjs.com/package/phishingdetectionapi) and [`webfilteringdatabase`](https://www.npmjs.com/package/webfilteringdatabase). Source: [github.com/explainableaixai](https://github.com/explainableaixai), [gitlab.com/url-classifications](https://gitlab.com/url-classifications).
