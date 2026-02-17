# axis-acap.md
# ARTPEC Developer Bible — AxisX ACAP Reference
# Source: https://axiscommunications.github.io/acap-documentation/
#         https://www.axis.com/en-us/for-developers/acap
#         https://github.com/AxisCommunications
#         https://github.com/pandosme (Fred Juhlin, Axis Sweden)
# Last reviewed: February 2026

## PURPOSE

This file makes Claude write correct ACAP code for ARTPEC-8/9 hardware
without hallucinating APIs, inventing memory limits, or recommending
patterns that get killed by the OS watchdog.

**The authoritative reference is always:**
https://axiscommunications.github.io/acap-documentation/

Before writing any ACAP C code, check `github.com/pandosme/make_acap`
for an existing reference pattern. Fred Juhlin (Axis Sweden/Lund)
builds close to the metal and his patterns reflect actual hardware behavior.

---

## ACAP OVERVIEW

ACAP (Axis Camera Application Platform) is Axis's open platform for
deploying custom applications on-device. Applications run partially
or entirely at the edge on AXIS OS-based devices.

**Current version:** ACAP 12.8 (February 2026)
**Primary language:** C/C++ via ACAP Native SDK
**Also supported:** Python via Computer Vision SDK containers
**Forward compatibility:** New minor AXIS OS versions are compatible
with apps built for prior ACAP versions. Major versions (like AXIS OS 13)
may have breaking changes — always test after major OS upgrades.

**ACAP is commercially viable:**
- Developers can sell ACAP applications independently
- Deployment licensing required (varies by app and scale)
- Free to develop and test
- Axis Technology Integration Partner Program available for deeper partnership

---

## ARTPEC CHIP GENERATIONS

Understanding which ARTPEC generation you're targeting changes
everything about what's possible and how you build it.

### ARTPEC-6 (Legacy)
- **DLPU:** None
- **Target arch:** armv7hf
- **ACAP support:** Limited
- **Vision Transformers:** NO
- **Recommendation:** Steer customers toward ARTPEC-8/9 upgrades.
  Don't invest significant development effort here.

### ARTPEC-7 (Active Legacy)
- **DLPU:** Basic inference, low TOPS
- **Target arch:** armv7hf
- **Max concurrent VAPIX connections:** ~10
- **Vision Transformers:** NO
- **Status:** Many active deployments in the field.
  Support existing ARTPEC-7 deployments but don't target new ones.
- **ML models:** Lightweight MobileNet-class only

### ARTPEC-8 (Current Mainstream) ← Primary AxisX Target
- **DLPU:** ~1.2 TOPS
- **Max concurrent ACAP applications:** 4
- **RAM available to ACAP:** ~256MB
  (verify specific product in Product Selector — varies by model)
- **Storage:** ~512MB eMMC shared with OS
- **Target arch:** aarch64
- **ACAP SDK:** Native SDK 4.x
- **Supported ML frameworks:** TensorFlow Lite, ONNX (quantized, INT8)
- **Recommended max model size:** ~50MB; hard ceiling ~100MB
- **Vision Transformers:** NO — quantized CNNs only
- **Max concurrent VAPIX connections:** ~20 (undocumented increase vs ARTPEC-7)
- **Model constraints:** MobileNet, EfficientNet-nano, YOLOv5-nano-scale

### ARTPEC-9 (Current Premium) ← Premium AxisX Target
- **DLPU:** ~4.0 TOPS (3.3x improvement over ARTPEC-8)
- **Max concurrent ACAP applications:** 6
- **RAM available to ACAP:** ~512MB
  (verify specific product in Product Selector)
- **Target arch:** aarch64
- **Vision Transformers:** YES — INT8 ViTs viable at edge
- **Key unlock:** Enables transformer-based analytics previously requiring cloud
- **Model options:** YOLOv8, ViT-Tiny, larger EfficientNet variants
- **Use this for all premium AxisX deployments**

**ARTPEC Generation Rule:**
When a partner asks about AI analytics on Axis cameras:
1. Ask which camera model (determines ARTPEC generation)
2. Check Product Selector for confirmed DLPU spec
3. ARTPEC-8 = CNNs only, simpler models
4. ARTPEC-9 = Vision Transformers viable, more sophisticated analytics

---

## DLPU SCHEDULING — THE MOST IMPORTANT THING IN THIS FILE

The DLPU (Deep Learning Processing Unit) is a **shared resource**.
Getting DLPU scheduling wrong is how ACAP apps get throttled, cause
performance issues, or get killed by the watchdog.

### Priority Order (Highest → Lowest)
```
1. OS operations
2. Video encoding pipeline  ← This ALWAYS wins over your app
3. ACAP applications (round-robin among competing apps)
```

### The Wrong Pattern — Don't Do This
```c
// WRONG: Continuous max-rate inference
// This starves other apps and gets throttled by the OS
while (running) {
    inference_result_t result = run_inference(current_frame);
    process_result(&result);
    // No yield, no sleep = DLPU monopolization
}
```

### The Right Pattern — Always Do This
```c
// CORRECT: Event-triggered inference
// Register callback, don't poll
// Only run inference when something worth analyzing happens

// 1. Set up motion or object detection trigger
axevent_handler_t *handler;
axevent_subscribe(
    event_handler,
    AX_EVENT_TOPIC_MOTION_ALARM,
    on_motion_detected,
    &app_context,
    &handler,
    NULL
);

// 2. In callback: trigger inference
void on_motion_detected(guint subscription, AXEvent *event, gpointer data) {
    app_context_t *ctx = (app_context_t *)data;
    
    // Queue inference request via ACAP Runtime API
    // Do NOT block here
    acap_runtime_request_inference_async(
        ctx->model_handle,
        ctx->current_frame,
        on_inference_complete,
        ctx
    );
}

// 3. Handle result in async callback
void on_inference_complete(inference_result_t *result, gpointer data) {
    // Process result
    // Publish via MQTT or event system
    // Do NOT run another inference from here synchronously
}
```

### DLPU Rules for AxisX Development
```
✓ Use async inference callbacks — never blocking calls
✓ Design for event-triggered inference bursts + idle periods
✓ Implement graceful degradation when DLPU is saturated
✓ Yield DLPU within 100ms maximum
✓ Target inference latency < 200ms end-to-end
✓ Monitor DLPU load via /axis-cgi/systemlog.cgi
✓ Test under load: behavior at 1 fps is different from 25 fps

✗ Never poll for frames and infer on every frame
✗ Never use blocking inference calls in main thread
✗ Never assume DLPU is always available
✗ Never ignore graceful degradation
```

---

## MEMORY MANAGEMENT

ARTPEC devices have **no swap**. What you allocate is what you get.
The OS watchdog kills ACAP apps that exceed their allocation.

### Rules
```c
// Always implement memory pressure callback
static void on_memory_pressure(gpointer user_data) {
    app_context_t *ctx = (app_context_t *)user_data;
    
    // Reduce buffer sizes
    // Clear non-essential caches
    // Log the event for monitoring
    g_warning("Memory pressure event — reducing buffers");
    reduce_inference_buffer_pool(ctx);
}

// Register it
acap_memory_register_pressure_callback(on_memory_pressure, app_ctx);
```

### Memory Profiling
```bash
# Profile your app on device
acap-sdk-utils --memory-profile {app-package-name}

# Monitor in real-time via VAPIX
GET /axis-cgi/systemlog.cgi
# Look for memory-related entries
```

### Memory Rules
- Never assume available RAM — check Product Selector for your target
- Allocate conservatively; leave headroom for video encoding pipeline
- Test under production load: memory leaks invisible at low frame rate
  become fatal under sustained production conditions
- Profile before declaring production-ready
- Set maximum buffer limits and enforce them

---

## BUILD SYSTEM

### Docker Container (Cross-Compilation)
```bash
# Primary build container
docker pull axisecp/acap-native-sdk:latest

# Build your application
docker run --rm \
  -v $(pwd):/opt/app \
  axisecp/acap-native-sdk:latest \
  make

# Output: your-app.eap file
```

### Architecture Targets
```
aarch64  → ARTPEC-8 and ARTPEC-9 (all current mainstream devices)
armv7hf  → ARTPEC-6 and ARTPEC-7 (legacy devices only)
```

**Always cross-compile. Never try to build natively on the device.**

### Build → Deploy → Test Cycle
Target: **< 90 seconds from code change to running on device**

```bash
# 1. Build
docker run --rm -v $(pwd):/opt/app axisecp/acap-native-sdk:latest make

# 2. Deploy via VAPIX
curl -X POST \
  --digest -u root:password \
  -F packfil=@your-app.eap \
  http://{device-ip}/axis-cgi/applications/upload.cgi

# 3. Start
curl --digest -u root:password \
  "http://{device-ip}/axis-cgi/applications/control.cgi?package=your-app&action=start"

# 4. Check status
curl --digest -u root:password \
  "http://{device-ip}/axis-cgi/applications/control.cgi?package=your-app&action=status"

# 5. Check logs
curl --digest -u root:password \
  "http://{device-ip}/axis-cgi/applications/control.cgi?package=your-app&action=log"
```

---

## ACAP INTEGRATION METHODS

Ranked by AxisX use case:

### 1. ACAP Runtime gRPC API (Primary — Video + Inference)
Use for: Real-time video capture, inference scheduling, result publishing
Key APIs: Video capture, DLPU inference, event publishing
This is what most AxisX ACAP development uses.

### 2. VAPIX CGI Interface (Device Control + Config)
Use for: Remote configuration, status queries, event subscriptions
Works over network — accessible from AxisX cloud layer
Reference: `api/axis-vapix.md`

### 3. MQTT (Event Publishing)
Use for: Publishing analytics results to cloud or VMS
Good for: Lightweight, high-frequency event streams from edge to cloud
Pattern: ACAP runs on device, publishes inference results via MQTT,
AxisX cloud layer subscribes and processes

### 4. Webhooks (Simple Notifications)
Use for: Simple trigger-based notifications to external systems
Lowest overhead, least flexible

---

## AXISX ACAP PATTERNS

### Pattern 1: Answering "Can this camera run X analytics?"
```
When an integrator asks about deploying analytics on specific hardware:

1. Identify target camera model
2. Check ARTPEC generation (Product Selector)
3. Check DLPU TOPS (Product Selector — do not guess)
4. Assess competing ACAP apps (each consumes DLPU budget)
5. Match model complexity to ARTPEC capability:
   - ARTPEC-8 (1.2 TOPS): MobileNet, EfficientNet-nano, YOLO-nano
   - ARTPEC-9 (4.0 TOPS): YOLOv8, ViT-Tiny, larger models
6. Give realistic expectations:
   - ARTPEC-8: Max 4 concurrent ACAP apps
   - ARTPEC-9: Max 6 concurrent ACAP apps
   - Multiple analytics apps = DLPU contention = latency increases
```

### Pattern 2: Writing New ACAP Code
```
Before writing any ACAP C code:
1. Check pandosme/make_acap for existing reference pattern
2. Check github.com/AxisCommunications for official examples
3. Review ACAP SDK docs for specific API being used
4. Never use blocking inference calls
5. Always implement memory pressure callback
6. Always test on physical ARTPEC hardware before production claim
7. Use Axis Virtual Loan Tool if no physical hardware available
```

### Pattern 3: AxisX Edge-to-Cloud Architecture
```
Design pattern for AxisX hybrid deployments:

ARTPEC-8/9 Camera
    ↓ (ACAP Runtime gRPC)
ACAP Application (lightweight metadata extraction)
    ↓ (MQTT or VAPIX event)
AxisX Cloud (Claude Sonnet performs complex reasoning)
    ↓
Integrator voice interface

Key principle: Camera handles lightweight, fast operations.
Claude handles complex reasoning and synthesis.
Never try to run transformer-scale reasoning on the edge.
```

---

## COMMUNITY RESOURCES — FRED JUHLIN (PANDOSME)

Fred Juhlin works at Axis Sweden (Lund HQ). His GitHub repos at
`github.com/pandosme` are practical reference implementations that
reflect how ARTPEC hardware actually behaves.

**Before writing new ACAP code:** Check `pandosme/make_acap` first.

Key repos:
- `pandosme/make_acap` — Demo ACAPs using various Axis camera services.
  Designed as base for new ACAP development. This is the practical
  complement to the official SDK docs.
- `pandosme/DetectX` — Custom YOLOv5 models running on Axis cameras.
  Real-world ML inference pattern for ARTPEC hardware. Shows the
  complete pipeline from model to deployment.
- `pandosme/node-red-contrib-axis-com` — Node-RED integration for
  integrators using automation workflows.

Fred's code is updated for current AXIS OS versions (Timelapse2
targets AXIS OS 11/12). Not rotting legacy code.

---

## QUICK REFERENCE: ACAP GOTCHA LIST

```
1. No swap on ARTPEC — memory allocation failures are fatal
2. DLPU is shared — design for event-triggered inference always
3. Memory watchdog kills leaking apps — implement pressure callbacks
4. Build container must match target arch (aarch64 vs armv7hf)
5. ACAP 12.8 is current — build against latest unless targeting legacy
6. AXIS OS 13 has breaking changes — test before upgrading production
7. Vision Transformers only on ARTPEC-9 — do not recommend for ARTPEC-8
8. Max 4 concurrent apps on ARTPEC-8, 6 on ARTPEC-9 — design for sharing
9. Always use Virtual Loan Tool before claiming device compatibility
10. Cross-compile always — never native build on device
```
