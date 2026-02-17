# axis-resources.md
# AxisX Master Resource Index — Axis Communications Developer Ecosystem
# Last reviewed: February 2026

## PURPOSE

This file tells Claude (and every AxisX developer) exactly where to find
authoritative information on Axis APIs, SDKs, products, and developer tools.

**The rule: Before writing any Axis-related code or answering any product
question, consult the appropriate resource below. Do NOT guess. Do NOT
hallucinate API endpoints. Always verify against these canonical sources.**

---

## TIER 1: Primary Developer Documentation (Authoritative, Always Current)

### VAPIX® API Reference
**URL:** https://developer.axis.com/vapix/
**What it covers:**
- Complete VAPIX API reference organized by category
- Authentication: Basic Auth, Digest Auth, Bearer Token (VAPIX 4.0+)
- Network video APIs (streaming, recording, PTZ)
- Device configuration APIs
- Audio systems
- Physical access control integration
- Radar integration
- Intercom
- Body worn systems

**When to use:** Any time writing or validating VAPIX API calls.
Always check here before writing endpoint URLs or parameters.
**Last updated:** Aug 18, 2025. Verify when working with firmware 10.x+ features.

---

### AXIS Developer Documentation Hub
**URL:** https://developer.axis.com/
**What it covers:**
- VAPIX full reference
- ACAP SDK documentation
- Analytics APIs
- AXIS Scene Metadata developer docs
- Radar integration guide
- Computer vision development guides
- Metadata Monitor tool

**When to use:** Starting point for any Axis development task.

---

### ACAP SDK Documentation (GitHub Pages)
**URL:** https://axiscommunications.github.io/acap-documentation/
**What it covers:**
- ACAP Native SDK (C/C++) full API reference
- All ACAP interface documentation
- Application examples for every major API
- Build system, packaging, and deployment guides
- Tutorials for ARTPEC-8 and ARTPEC-9
- Curated ACAP FAQ

**When to use:** Any ACAP application development. This is the bible.
Always check before writing ACAP C code. The official docs are more
reliable than training data for specific API signatures and constants.

---

## TIER 2: Product & Platform Pages

### Axis For Developers Portal
**URL:** https://www.axis.com/en-us/for-developers
**What it covers:**
- Developer news and announcements
- Links to all developer resources
- New releases (ACAP 12.8 is current as of February 2026)
- AXIS OS 13 breaking changes documentation
- Computer vision development guides
- Analytics integration resources
- Metadata Monitor tool

**When to use:** Check for announcements, new releases, breaking changes.
Run this before any major development sprint to catch anything that changed.
**Key news as of Feb 2026:** ACAP 12.8 released. AXIS OS 13 affects 200+ smart devices.

---

### ACAP Platform Overview
**URL:** https://www.axis.com/en-us/for-developers/acap
**Key facts:**
- ACAP is forward compatible: new minor AXIS OS versions are compatible
  with apps built for prior versions
- ACAP apps interface via: VAPIX network interface, C libraries in SDK,
  MQTT, webhooks
- Apps can be commercialized and sold independently
- Free to develop; deployment licensing depends on application and scale
- Primary language: C/C++ via ACAP Native SDK
- Hardware spec reference: Use Product Selector for SoC/DLPU details

**When to use:** Product decisions about ACAP support, capability planning,
understanding ACAP lifecycle and compatibility commitments.

---

### AXIS Scene Metadata Integration
**URL:** https://www.axis.com/en-us/for-developers/scene-metadata-integration
**Developer Docs:** https://developer.axis.com/analytics/axis-scene-metadata/

**What it covers:**
- Real-time structured metadata from Axis AI cameras
- Object detection and classification output format
- Frame-by-frame vs consolidated metadata consumption modes
- Multi-object tracking with persistent IDs across frames
- Best snapshot feature for detected objects
- Multi-sensor fusion (radar + video)
- Edge, VMS, IoT/BI, and second analytics layer patterns

**Objects supported:** Humans, vehicles (type + color), attributes
(hats, bags, etc.), movement data (position, direction, speed,
timestamps), geo-coordinates (requires radar integration)

**Two consumption modes:**
- **Frame-by-frame**: Real-time, high temporal precision. Use for
  queue monitoring, loitering detection, traffic management.
- **Consolidated**: Entry-to-exit summary per object. Use for forensic
  search and statistical analysis without per-frame overhead.

**When to use:** Building any analytics integration, metadata consumption,
or scene intelligence feature in AxisX.

---

## TIER 3: GitHub Repositories

### Axis Communications Official GitHub
**URL:** https://github.com/AxisCommunications
**What it covers:**
- Official Axis code repositories
- ACAP application examples
- ACAP SDK tools and utilities
- Computer vision SDK examples (Docker + Python)
- Key repos: acap-native-sdk, acap-computer-vision-sdk, acap-documentation

**When to use:** Code examples, reference implementations, SDK containers.
Always check here before writing new ACAP code from scratch.
Someone has probably already done a version of what you need.

---

### Fred Juhlin — pandosme (Axis Sweden / Lund)
**URL:** https://github.com/pandosme
**Who:** Fred Juhlin, Axis Communications Sweden team, Lund HQ
**Character:** Forward-thinking, likes to test, builds practical reference
code that bridges official SDK and real-world deployment

**Key repositories:**

| Repo | Description | AxisX Use Case |
|------|-------------|----------------|
| `pandosme/make_acap` | Demo ACAPs using various Axis camera services. Designed as base for new ACAP development. | Reference before writing new ACAP |
| `pandosme/DetectX` | Custom YOLOv5 models running on Axis cameras | Real-world ML inference pattern for ARTPEC |
| `pandosme/node-red-contrib-axis-com` | Node-RED integration nodes for Axis cameras | Integrator automation workflow reference |
| `pandosme/Timelapse2` | ACAP updated for AXIS OS 11/12 | OS compatibility patterns |
| `pandosme/node-red-axis-discovery` | UPnP and Bonjour-based Axis device discovery | Auto-discovery feature reference |
| `pandosme/privateCA` | Docker-based private CA with Node-RED UI | Certificate management in Axis deployments |

**Why Fred matters:**
Fred's repos are the unofficial "how Axis engineers actually build it" guide.
His code reflects how ARTPEC hardware actually behaves — not just how the
docs say it should. He updates for current AXIS OS, so his patterns are not
rotting legacy code.

**AxisX usage rule:** Before writing ACAP C code, check `pandosme/make_acap`
for an existing reference pattern. Treat his repos as semi-official
insider knowledge from Axis Sweden engineering.

---

## TIER 4: Partner & Support Tools

### Product Selector (Spec Authority)
**URL:** https://www.axis.com/en-us/support/tools/product-selector
**What it covers:**
- All Axis products with full filterable spec comparison
- Filter by: ACAP support, SoC type, ARTPEC generation, resolution,
  form factor, operating temperature, IP/IK rating, PoE class,
  codec support (H.264/H.265/AV1), DLPU/MLPU TOPS, analytics support
- Product interface guide: SoC details, compute capabilities (MLPU/DLPU),
  flash and RAM per specific product

**When to use:** Answering any partner question about hardware capabilities.
Verifying hardware before recommending ACAP deployment.

**CRITICAL RULE: DO NOT guess RAM, storage, or DLPU specs from memory.
Always verify via Product Selector. Specs vary significantly by product
even within the same camera family.**

---

### Virtual Loan Tool (ADP)
**URL:** https://www.axis.com/partner_pages/adp_virtual_loan_tool/common#/home
**Access:** Requires Axis partner account
**What it covers:**
- Online access to real Axis devices for integration testing
- Test VAPIX API calls against actual devices on actual firmware
- ACAP deployment and testing without physical hardware
- Compatibility validation before customer deployment

**When to use:** Testing AxisX integrations before field deployment.
Validating VAPIX endpoint behavior on specific firmware versions.
This is how you catch the "it works in the docs but not on the device"
class of bugs before they hit a customer site.

---

### Developer News & Release Notes
**URL:** https://www.axis.com/en-us/for-developers/news
**Monitor for:**
- ACAP version releases (current: 12.8)
- AXIS OS version releases and breaking changes
- New API capabilities
- New product launches with ACAP support

**AXIS OS 13 note:** Major release affecting 200+ smart devices.
Reference: https://www.axis.com/en-us/for-developers/news/AXIS-OS-13-breaking-changes
Always check firmware version when writing device-specific code.

---

## QUICK REFERENCE: RIGHT RESOURCE FOR THE JOB

| I need to... | Go to |
|--------------|-------|
| Write a VAPIX API call | developer.axis.com/vapix/ |
| Write ACAP C code | axiscommunications.github.io/acap-documentation/ |
| Find an ACAP code example | github.com/AxisCommunications |
| Find Fred's practical ACAP patterns | github.com/pandosme |
| Check if a camera supports ACAP | axis.com Product Selector |
| Check DLPU/RAM for a specific camera | axis.com Product Selector |
| Test against a real device remotely | Axis Virtual Loan Tool |
| Check for breaking changes | axis.com/en-us/for-developers/news |
| Understand Scene Metadata format | developer.axis.com/analytics/axis-scene-metadata/ |
| Get the latest firmware | axis.com/en-us/support/device-software |

---

## OVERNIGHT AGENT MONITORING TARGETS

The AxisX overnight agent should watch these sources for changes:

1. `https://www.axis.com/en-us/for-developers/news` — New releases, breaking changes
2. `https://github.com/AxisCommunications` — New repos, major commits
3. `https://github.com/pandosme` — Fred's new repos or updates
4. `https://axiscommunications.github.io/acap-documentation/` — SDK updates
5. `https://www.axis.com/en-us/support/device-software` — New firmware versions

When changes are detected:
- Open a GitHub issue in this repo flagging what changed
- Generate a summary of AxisX impact
- Flag any breaking changes that require knowledge base updates
