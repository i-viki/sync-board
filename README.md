# SyncBoard Workspace

[![React](https://img.shields.io/badge/React-19.2-18181b?logo=react&logoColor=61DAFB&labelColor=101012)](https://react.dev)
[![Vite](https://img.shields.io/badge/Vite-7.3-18181b?logo=vite&logoColor=9135FF&labelColor=101012)](https://vitejs.dev)
[![MUI](https://img.shields.io/badge/MUI-9.0-18181b?logo=mui&logoColor=007FFF&labelColor=101012)](https://mui.com)
[![Firebase](https://img.shields.io/badge/Firebase-12.9-18181b?logo=firebase&logoColor=FFCA28&labelColor=101012)](https://firebase.google.com)
[![Cloudinary](https://img.shields.io/badge/Storage-Cloudinary-18181b?logo=cloudinary&logoColor=3448C5&labelColor=101012)](https://cloudinary.com)
[![PWA](https://img.shields.io/badge/PWA-Installable-18181b?logo=pwa&logoColor=5A0FC8&labelColor=101012)](https://web.dev/progressive-web-apps/)
[![PrismJS](https://img.shields.io/badge/PrismJS-Syntax_Engine-18181b?logo=javascript&logoColor=F7DF1E&labelColor=101012)](https://prismjs.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A high-performance, **zero-account real-time collaboration workspace and PWA clipboard**, engineered for frictionless text, code, and asset synchronization across devices. Powered by **React 19**, **Vite 7**, **Firebase Realtime Database**, and **Cloudinary CDN**, the platform coordinates **400ms atomic debouncing**, **salted SHA-256 room cryptographic locking**, ephemeral self-destruct timers, and offline-first PWA caching without collecting any personally identifiable information (Zero-PII).

---

## Key Architecture Pillars

- **Atomic Debounced Synchronization (`src/services/firebaseService.js`):**
  400ms atomic debounce engine writing directly to isolated Firebase `/rooms/{roomId}/text` paths, guaranteeing database stability and zero input lag under typing bursts.
- **Zero-PII Presence & Obfuscated History (`src/hooks/` & `src/utils/`):**
  Tracks live collaborator presence via Firebase `.info/connected` and `onDisconnect` triggers without capturing IP addresses, storing the last 5 accessed boards in Base64-obfuscated local storage.
- **Cryptographic Salted SHA-256 Room Locking (`src/components/features/PinModal.jsx`):**
  Enforces room read/write isolation using Web Crypto API SHA-256 derivation salted with the room code ($H = \text{SHA-256}(\text{RoomCode} \parallel \text{PIN})$), placing unauthenticated guests in strict read-only mode.
- **Ephemeral Lifecycle & Self-Destruct Automation (`src/components/features/SelfDestructTimer.jsx`):**
  Configurable board lifetimes (10 minutes, 1 hour, 24 hours) verified and purged upon expiration directly from the client and backend validation scripts.
- **Heuristic Multi-Language Syntax Detection (`src/hooks/useLanguageDetection.js`):**
  In-memory regex pattern scoring engine identifying 15+ programming languages dynamically and formatting code via PrismJS without UI thread blocking.
- **Offline PWA Architecture & Asset Pipeline (`vite.config.js`):**
  Fully installable PWA utilizing Workbox service workers with offline fallback pages, paired with Cloudinary CDN direct unsigned asset uploads up to 10MB.

---

## System Architecture

```mermaid
flowchart TB
    subgraph CLIENT["Consumer Tier (React 19 / Vite PWA)"]
        APP["SyncBoard Client<br/>Port: 3000 / Vercel Edge"]
        SW["Workbox Service Worker<br/>• Offline Cache<br/>• App Manifest"]
        APP <--> SW
    end

    subgraph SECURITY["Client-Side Security Perimeter"]
        direction TB
        PIN_ENGINE["Salted SHA-256 Derivation<br/>crypto.subtle.digest"]
        AUTH_ENGINE["Firebase Anonymous Auth<br/>• auth != null Rule Compliance"]
        PIN_ENGINE --> AUTH_ENGINE
    end

    subgraph SYNC_MESH["Real-Time State Mesh (Firebase RTDB)"]
        ROOMS[("Firebase Realtime Database<br/>Node: /rooms/{roomId}<br/>• text (Atomic 400ms Debounce)<br/>• lastUpdated<br/>• passwordHash (SHA-256 Salted)")]
        PRESENCE[("Presence Subsystem<br/>Node: /.info/connected<br/>• onDisconnect Trigger<br/>• Zero-PII Active User Count")]
    end

    subgraph STORAGE_CDN["Asset Storage & Delivery (Cloudinary)"]
        CLOUDINARY["Cloudinary Media Gateway<br/>• Direct Unsigned Preset<br/>• Automatic WebP Thumbnailing<br/>• Max 10MB Multi-Format"]
    end

    subgraph MAINTENANCE["Platform Maintenance Suite"]
        VALIDATOR["Database Validator Utility<br/>validator/script.py<br/>• Schema Integrity Check<br/>• Expired Room Purge"]
    end

    APP --> SECURITY
    SECURITY --> ROOMS
    APP <--> PRESENCE
    APP -->|Multipart Direct Upload| CLOUDINARY
    VALIDATOR -.->|Audit & Purge| ROOMS
```

---

## Cryptographic Salt & Debounce Invariants

SyncBoard enforces strict user anonymity and cryptographic room boundaries without server-side user tables:

$$H_{\text{room}} = \operatorname{SHA-256}(\text{RoomCode}_{5\text{-char}} \parallel \text{PIN}_{6\text{-digit}})$$

$$T_{\text{debounce}} = 400\text{ms} \implies \text{Max Database Write Rate} = 2.5 \text{ ops/sec per user}$$

| Security Invariant | Technical Implementation | Privacy Guarantee |
| :--- | :--- | :--- |
| **Room Code Space** | 5-character alphanumeric ($[A-Z0-9]^5 \approx 60.4\text{M}$ rooms) | Frictionless sharing with bounded collision space |
| **Salted PIN Lock** | `SHA-256(code + ":" + pin)` | Rainbow table immune; client enforces read-only mode |
| **Anonymous Session** | Firebase Anonymous Token | Database rules require authentication without identity tracking |
| **Zero-PII Presence** | Temporary session UUIDs | IP addresses and geolocation are never collected |
| **Ephemeral Destruction**| Timestamp verification upon read | Expired records purged permanently from database |

---

## Quickstart & Platform Orchestration

### Prerequisites
- Node.js 18.0+ & npm 9+
- Firebase Project with Realtime Database enabled ([Firebase Console](https://console.firebase.google.com))
- Cloudinary Account for file uploads ([Cloudinary Free Tier](https://cloudinary.com))
- Python 3.8+ (Optional, for database maintenance utilities)

---

### Option A: Local Development Launch (Recommended)

Run the full platform locally with live Vite hot-reloading:
```bash
# Clone repository
git clone https://github.com/i-viki/sync-board.git
cd SyncBoard

# Install dependencies
npm install

# Configure environment secrets
cp example.env .env
# Edit .env with your Firebase and Cloudinary credentials

# Launch dev server
npm run dev
```
*Application available immediately at `http://localhost:3000`.*

---

### Option B: Production Build & Local Preview

```bash
# Compile optimized production bundle with PWA assets
npm run build

# Preview production build locally
npm run preview
```

---

## Decoupled Environment Strategy

All credentials and API keys are abstracted into environment templates loaded at build time:

| Service / Layer | Config File | Template Reference | Key Variables |
| :--- | :--- | :--- | :--- |
| **Firebase Realtime DB** | `src/config/firebase.js` | [`example.env`](example.env) | `VITE_APP_APIKEY`, `VITE_APP_DATABASEURL`, `VITE_APP_PROJECTID` |
| **Cloudinary CDN** | `src/services/cloudinaryService.js` | [`example.env`](example.env) | `VITE_APP_CLOUDINARY_CLOUD_NAME`, `VITE_APP_CLOUDINARY_CLOUD_UPLOAD_PRESET_NAME` |
| **Email Feedback** | `src/components/common/Feedback.jsx` | [`example.env`](example.env) | `VITE_APP_EMAILJSAPIKEY`, `VITE_APP_EMAILJSSERVICE_KEY` |
| **PWA & Offline** | `vite.config.js` | [Default Manifest] | Workbox precache manifest, runtime caching rules |

---

## Live Telemetry & Observability Hubs

| Hub / Service | Port | URL | Role & Live Features |
| :--- | :--- | :--- | :--- |
| **SyncBoard Web App** | `3000` | [http://localhost:3000](http://localhost:3000) | Core collaboration workspace, real-time board editor & QR sharing. |
| **Production Deployment**| Cloud | [https://isyncboard.vercel.app](https://isyncboard.vercel.app) | Production Vercel Edge deployment. |
| **Firebase RTDB Console**| Cloud | [Firebase Console](https://console.firebase.google.com) | Realtime JSON tree monitoring and rule verification. |

---

## Testing & Quality Assurance Suite

### Code Quality & Linter
```bash
npm run lint
# Executes ESLint across all React JSX and utility modules.
```

### Database Validator & Schema Audit Utility (Python 3)
```bash
python validator/script.py
# Audits Realtime Database JSON exports, validates room schemas, and archives corrupt nodes.
```

---

## Author

**Jayavignesh**
- **Website:** [jayavignesh.dev](https://jayavignesh.dev)
- **GitHub:** [@i-viki](https://github.com/i-viki)
- **Email:** [hello@jayavignesh.dev](mailto:hello@jayavignesh.dev)

---

## License

This platform is distributed under the **MIT License**. See the [`LICENSE`](LICENSE) file for complete terms.
