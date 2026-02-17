# axis-vapix.md
# VAPIX® API Reference — AxisX Developer Bible
# Source: https://developer.axis.com/vapix/
# Last reviewed: February 2026

## PURPOSE

This file makes Claude stop guessing about VAPIX API calls and start
writing production-correct code on the first pass. Every pattern here
is sourced from official Axis documentation.

**Before writing any VAPIX code, read this file. Before answering any
VAPIX question, read this file. Do not invent endpoints.**

The authoritative reference is always: https://developer.axis.com/vapix/

---

## WHAT IS VAPIX

VAPIX (Video Application Programming Interface for IP cameras by Axis)
is Axis's open API that uses standard HTTP/HTTPS protocols to provide
direct access and control of Axis devices.

VAPIX is organized into categories:
- Authentication
- Network video (streaming, PTZ, recording)
- Device configuration
- Applications (ACAP management)
- Audio systems
- Physical access control
- Radar
- Intercom
- Body worn systems

---

## AUTHENTICATION

VAPIX supports three authentication methods. Choosing wrong is the
most common integration failure point.

### Method 1: Basic Auth (Legacy)
```
Authorization: Basic base64(username:password)
```
- Still common on older firmware and legacy integrations
- **NOT recommended for new development**
- Some endpoints reject Basic Auth even when HTTP is enabled globally
- Works on most ARTPEC-6/7 devices by default

### Method 2: Digest Auth (Default)
```
# First request returns 401 with WWW-Authenticate: Digest header
# Client computes MD5 hash of credentials + nonce
# Second request includes Authorization: Digest header
```
- Default auth method on most modern Axis devices
- Required for many sensitive endpoints
- Libraries: Python `requests` handles this with `HTTPDigestAuth`
- **Use this unless you have a specific reason not to**

```python
import requests
from requests.auth import HTTPDigestAuth

response = requests.get(
    'http://{device-ip}/axis-cgi/basicdeviceinfo.cgi',
    auth=HTTPDigestAuth('root', 'password')
)
```

### Method 3: Bearer Token (VAPIX 4.0+)
```
Authorization: Bearer {token}
```
- Preferred for new integrations on current firmware
- Requires AXIS OS with VAPIX 4.0 support
- Token obtained via VAPIX authentication API
- **Use this for all new AxisX cloud integrations**

### Auth Decision Rule
```
If firmware >= AXIS OS 10.x AND new development:
    Use Bearer Token
Elif device is Axis and you have credentials:
    Use Digest Auth
Elif legacy system or ARTPEC-6/7:
    Use Basic Auth (fallback)
```

---

## DEVICE CAPABILITY — START HERE

Always query device capabilities before making feature-specific calls.
This prevents calling endpoints that don't exist on the firmware version.

### Basic Device Info
```
GET http://{device-ip}/axis-cgi/basicdeviceinfo.cgi
Auth: Digest or Bearer
Response: XML (default) or JSON (with Accept: application/json)
```

Returns: Model, firmware version, serial number, hardware ID, ARTPEC generation

```python
# Get firmware version to gate feature availability
import requests
from requests.auth import HTTPDigestAuth
import xml.etree.ElementTree as ET

def get_device_info(ip, username, password):
    resp = requests.get(
        f'http://{ip}/axis-cgi/basicdeviceinfo.cgi',
        auth=HTTPDigestAuth(username, password)
    )
    root = ET.fromstring(resp.text)
    return {
        'firmware': root.find('.//FirmwareVersion').text,
        'model': root.find('.//ProdShortName').text,
        'serial': root.find('.//SerialNumber').text
    }
```

---

## CRITICAL GOTCHAS

These are the bugs that bite everyone. Read them. Remember them.

```
1. VAPIX responses are XML by default.
   JSON requires explicit Accept: application/json header.
   Missing this = parse errors that look like device problems.

2. PTZ speed values are NOT normalized across product families.
   Speed "50" on a Q61 PTZ is not the same as speed "50" on a P55.
   Always test PTZ speed values on target hardware.

3. Event subscription TTL defaults to 60 seconds.
   Must heartbeat or re-subscribe before TTL expires.
   Failure = silent subscription death, missed events.

4. Metadata stream format changed between firmware 9.x and 10.x.
   If parsing metadata and it breaks after firmware upgrade, this is why.
   Check firmware version and branch your parser.

5. ARTPEC-7 devices: max concurrent VAPIX connections ~10.
   ARTPEC-8 raised this to ~20 (undocumented).
   ARTPEC-9: check Product Selector for current limit.
   Exceeding this = connection refused, looks like network issue.

6. CGI parameter names are case-sensitive.
   resolution=1920x1080 works. Resolution=1920x1080 fails silently.
   Always match exact case from documentation.

7. Some endpoints require HTTPS even when HTTP is globally enabled.
   Especially: certificate management, user management, firmware upgrade.
   Test with HTTPS if HTTP returns 403 unexpectedly.

8. AXIS OS 13 introduced breaking changes on 200+ devices.
   Reference: axis.com/en-us/for-developers/news/AXIS-OS-13-breaking-changes
   Always check firmware version before deploying to production.
```

---

## CORE ENDPOINT MAP

### Video Streaming
```
RTSP (H.264/H.265):
  rtsp://{username}:{password}@{device-ip}/axis-media/media.amp

RTSP with params:
  rtsp://{device-ip}/axis-media/media.amp?videocodec=h265&resolution=1920x1080

MJPEG (HTTP):
  http://{device-ip}/axis-cgi/mjpg/video.cgi

Still image (JPEG):
  GET http://{device-ip}/axis-cgi/jpg/image.cgi?resolution=1920x1080
  Auth: Digest
```

### Device Information
```
Basic device info:
  GET /axis-cgi/basicdeviceinfo.cgi

System log (includes DLPU load):
  GET /axis-cgi/systemlog.cgi

Parameter list (full device config):
  GET /axis-cgi/param.cgi?action=list

Specific parameter:
  GET /axis-cgi/param.cgi?action=list&group=Network.eth0
```

### PTZ Control
```
Absolute move (go to exact position):
  GET /axis-cgi/com/ptz.cgi?pan={-180 to 180}&tilt={-90 to 90}&zoom={1-9999}

Relative move (move from current position):
  GET /axis-cgi/com/ptz.cgi?rpan={delta}&rtilt={delta}&rzoom={delta}

Continuous move (keep moving until stopped):
  GET /axis-cgi/com/ptz.cgi?continuouspantiltmove={pan_speed},{tilt_speed}
  # Stop: continuouspantiltmove=0,0

PTZ preset go-to:
  GET /axis-cgi/com/ptz.cgi?gotoserverpresetname={preset_name}

Get PTZ position:
  GET /axis-cgi/com/ptz.cgi?query=position
```

**PTZ Speed Warning:** Speed values vary by product family. Always test
on target hardware. Document the speed values that work for each deployment.

### Event System
```
List available event topics:
  GET /axis-cgi/eventnotification.cgi?action=getlist

Subscribe to events (WS-BaseNotification):
  POST /vapix/ws-notifications
  Content-Type: application/soap+xml
  Body: WS-BaseNotification subscription XML

Event subscription TTL: default 60 seconds
Heartbeat endpoint: Send subscription renewal before TTL expires
```

### ACAP Application Management
```
List installed applications:
  GET /axis-cgi/applications/list.cgi

Upload application (.eap file):
  POST /axis-cgi/applications/upload.cgi
  Content-Type: multipart/form-data
  Body: EAP file

Start application:
  GET /axis-cgi/applications/control.cgi?package={package_name}&action=start

Stop application:
  GET /axis-cgi/applications/control.cgi?package={package_name}&action=stop

Get application status:
  GET /axis-cgi/applications/control.cgi?package={package_name}&action=status
```

### Analytics & Metadata
```
AXIS Object Analytics data:
  GET /axis-cgi/analytics.cgi

Scene Metadata stream:
  Via RTSP with metadata channel
  Or via AXIS Scene Metadata API
  Reference: developer.axis.com/analytics/axis-scene-metadata/

Motion detection configuration:
  GET /axis-cgi/param.cgi?action=list&group=Motion
```

---

## FIRMWARE COMPATIBILITY MATRIX

Feature availability by AXIS OS version. Always check firmware before
recommending or implementing these features.

| Feature | Minimum AXIS OS | Notes |
|---------|----------------|-------|
| VAPIX 4.0 / Bearer Token | 10.x | Check specific model |
| AXIS Scene Metadata | 10.x | Requires ARTPEC-8+ for full feature set |
| H.265 streaming | 9.x on ARTPEC-7+ | Not available on ARTPEC-6 |
| AV1 codec | AXIS OS 11.x | ARTPEC-9 only |
| Vision Transformer ACAP | AXIS OS 11.x | ARTPEC-9 only |
| AXIS OS 13 features | 13.x | Breaking changes — test before upgrading |

**Rule:** Before using any firmware-specific feature, query
`/axis-cgi/basicdeviceinfo.cgi` and check `FirmwareVersion`.
Gate your feature usage accordingly.

---

## AXISX-SPECIFIC VAPIX PATTERNS

### Pattern 1: Pre-flight Device Check
Always run this before any AxisX operation against a device.

```python
def axisx_device_preflight(ip, username, password):
    """
    AxisX standard pre-flight check before any device operation.
    Returns device context dict or raises with clear error.
    """
    try:
        info = get_device_info(ip, username, password)
        firmware_major = int(info['firmware'].split('.')[0])
        
        return {
            'model': info['model'],
            'firmware': info['firmware'],
            'firmware_major': firmware_major,
            'supports_vapix4': firmware_major >= 10,
            'supports_scene_metadata': firmware_major >= 10,
            'supports_av1': firmware_major >= 11,
        }
    except Exception as e:
        raise RuntimeError(f"AxisX preflight failed for {ip}: {e}")
```

### Pattern 2: Safe VAPIX Call Wrapper
```python
def vapix_get(device_ip, endpoint, auth, params=None, json_response=False):
    """
    Standard AxisX VAPIX GET wrapper.
    Handles auth, JSON vs XML, and common error cases.
    """
    headers = {}
    if json_response:
        headers['Accept'] = 'application/json'
    
    url = f'http://{device_ip}{endpoint}'
    
    response = requests.get(
        url,
        auth=auth,
        params=params,
        headers=headers,
        timeout=10
    )
    
    response.raise_for_status()
    
    if json_response:
        return response.json()
    return response.text  # XML — parse as needed
```

### Pattern 3: Event Subscription with Auto-Renew
```python
import threading
import time

class VAPIXEventSubscription:
    """
    AxisX event subscription with automatic TTL renewal.
    Prevents silent subscription death.
    """
    TTL_SECONDS = 60
    RENEW_AT = 45  # Renew 15 seconds before expiry
    
    def __init__(self, device_ip, auth, topic, callback):
        self.device_ip = device_ip
        self.auth = auth
        self.topic = topic
        self.callback = callback
        self.subscription_id = None
        self._renewal_thread = None
        self._active = False
    
    def subscribe(self):
        # Implementation: POST WS-BaseNotification subscription
        # Store subscription_id
        self._active = True
        self._start_renewal_thread()
    
    def _start_renewal_thread(self):
        def renew_loop():
            while self._active:
                time.sleep(self.RENEW_AT)
                if self._active:
                    self._renew_subscription()
        
        self._renewal_thread = threading.Thread(
            target=renew_loop, daemon=True
        )
        self._renewal_thread.start()
```

---

## RESPONSE FORMAT REFERENCE

### XML (default)
Most VAPIX endpoints return XML by default.
```xml
<root>
  <basicdeviceinfo>
    <ProdShortName>AXIS P3245-V</ProdShortName>
    <FirmwareVersion>10.12.101</FirmwareVersion>
    <SerialNumber>ACCC8E123456</SerialNumber>
  </basicdeviceinfo>
</root>
```

### JSON (with Accept header)
```json
{
  "apiVersion": "1.0",
  "data": {
    "prodShortName": "AXIS P3245-V",
    "firmwareVersion": "10.12.101",
    "serialNumber": "ACCC8E123456"
  }
}
```

**Always request JSON for new integrations.** Cleaner parsing,
better error messages, future-proof.

---

## SECURITY HARDENING NOTES

For enterprise deployments (Stone persona especially):

```
1. Always use HTTPS in production. HTTP is for lab only.
2. Use Digest or Bearer Auth. Never Basic Auth in production.
3. Create dedicated API accounts with minimum required privileges.
   Do not use root account for API integrations.
4. Rotate credentials regularly. VAPIX has no automatic expiry.
5. Reference: Axis Hardening Guide for enterprise deployments.
   Available via Axis partner portal.
6. NDAA compliance: All ARTPEC-based Axis devices are NDAA Section 889
   compliant. Document this for government/enterprise customers.
```
