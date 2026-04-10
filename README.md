# Technical Specification: Smart TAC Shield

## 1. Overview
The Smart TAC Shield is a security module integrated into the mobile banking application and backend Risk Engine. It evaluates device telemetry and environmental context in real-time to detect social engineering and scam indicators prior to generating or approving a Transaction Authorization Code (TAC).

## 2. System Architecture
The system consists of two primary components:
1. **Client-Side Security Module (Mobile SDK):** Collects local device telemetry (call status, screen recording status, sideloaded apps).
2. **Backend Risk Engine:** Receives telemetry, cross-references with transaction history and anomaly detection models, and returns a risk decision (Allow, Friction, Block).

## 3. Client-Side Telemetry & OS Permissions

### 3.1 Android Implementation
* **Active Call Detection:** * API: `TelephonyManager.getCallState()`
  * Permission: `READ_PHONE_STATE`
* **Screen Sharing / Remote Control Detection:**
  * API: Monitor `MediaProjection` service state and check for active `DisplayManager` virtual displays.
* **Malware / Overlay Detection:**
  * API: Check for apps with `SYSTEM_ALERT_WINDOW` (Draw over other apps) permission active.
  * API: Utilize Google Play Integrity API for device attestation.

### 3.2 iOS Implementation
* **Active Call Detection:**
  * API: `CallKit` -> `CXCallObserver` to check `hasConnected` and `hasEnded` properties.
* **Screen Recording / AirPlay Detection:**
  * API: `UIScreen.main.isCaptured` (Returns a boolean indicating if the screen is being recorded, mirrored, or shared via SharePlay).

## 4. API Specifications

### 4.1 Evaluate Risk Endpoint
Called by the mobile client immediately after the user initiates a transfer but *before* the TAC prompt is rendered.

* **Endpoint:** `POST /api/v1/security/tac/evaluate`
* **Headers:** `Authorization: Bearer <token>`, `X-Device-ID: <uuid>`
* **Request Payload (JSON):**
  ```json
  {
    "transaction_id": "tx_8932479823",
    "amount": 5000.00,
    "payee_account": "1234567890",
    "telemetry": {
      "is_active_call": true,
      "is_screen_captured": false,
      "active_overlays": [],
      "device_uptime_seconds": 86400
    }
  }
