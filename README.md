# SyncBoard

**SyncBoard** is a high-performance, real-time collaboration platform designed for instant text, code, and file sharing across devices. Built with speed and simplicity in mind, it eliminates the overhead of user accounts and complex configurations, providing a frictionless digital workspace for developers, content creators, and teams.

[![Status](https://img.shields.io/badge/Status-Production--Ready-success?style=flat)](https://isyncboard.vercel.app/)
[![Live Demo](https://img.shields.io/badge/Demo-isyncboard.vercel.app-0070F3?style=flat&logo=vercel&logoColor=white)](https://isyncboard.vercel.app/)
[![Privacy](https://img.shields.io/badge/Privacy-Zero_PII-blue?style=flat)](PRIVACYPOLICY.md)
[![Security](https://img.shields.io/badge/Security-Salted_SHA--256-blueviolet?style=flat)](#board-security--pin-lock)
[![PWA](https://img.shields.io/badge/PWA-Installable-5A0FC8?style=flat&logo=pwa&logoColor=white)](#progressive-web-app-pwa--offline-resilience)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat)](LICENSE)

Developed with ☕

[![Vite](https://img.shields.io/badge/Vite-7.3.3-B73BFE?style=flat&logo=vite&logoColor=white)](https://vitejs.dev/)
[![React](https://img.shields.io/badge/React-19.2.5-20232A?style=flat&logo=react&logoColor=61DAFB)](https://react.dev/)
[![MUI](https://img.shields.io/badge/MUI-9.0.0-007FFF?style=flat&logo=mui&logoColor=white)](https://mui.com/)
[![Firebase](https://img.shields.io/badge/Firebase-12.9.0-FFCA28?style=flat&logo=firebase&logoColor=black)](https://firebase.google.com/)
[![Cloudinary](https://img.shields.io/badge/Storage-Cloudinary-3448C5?style=flat&logo=cloudinary&logoColor=white)](https://cloudinary.com/)
[![Language](https://img.shields.io/badge/Language-JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![Version](https://img.shields.io/badge/Version-1.3.0-orange?style=flat)](package.json)

---

## Core Features

### Real-Time Debounced Synchronization
- **Zero-Latency Collaboration**: Text entered into a board is instantly synchronized across all active participants using a unique **5-character board code**.
- **400ms Debounce Engine**: Atomic debouncing ensures database write stability and smooth typing performance even at high typing speeds.
- **Atomic Text Updates**: Updates the text node directly in Firebase Realtime Database without rewriting the entire board payload.

### Collaborative Presence & Obfuscated History
- **Live User Tracking**: Active collaborator count is tracked in real time using Firebase `.info/connected` presence and `onDisconnect` listeners.
- **Obfuscated Local History**: Remembers your last 5 visited boards in your browser using **Base64-encoded local storage** (`btoa`), allowing 1-click access from the homepage without exposing room codes in plain text.
- **QR Code Sharing**: Generate scannable QR codes on the fly using `qrcode.react` to instantly open boards on mobile devices.

### Universal File Gallery
- **Up to 5 Files Simultaneously**: Share up to 5 multi-format assets per board (Images, PDFs, Documents, Videos, Audio, and Code files up to 10MB each).
- **Cloudinary Integration**: Fast asset uploads using unsigned presets with automated image thumbnail optimization (`c_thumb,w_96,h_96,q_auto,f_auto`).
- **Hardened Blob Downloads**: Direct fetch-based blob download system ensures reliable file downloads with accurate file names and extensions.
- **Drag & Drop**: Seamlessly drag and drop files onto the board for instant uploading.

### Self-Destruct Countdown Timer
- **Configurable Ephemeral Lifetimes**: Set a board self-destruct countdown timer (**10 Minutes**, **1 Hour**, or **24 Hours**) directly from the board navbar.
- **Automated Destruction**: When the timer expires, the board's text, files, and PIN hash are automatically purged from the database upon the next access.
- **Live Navbar Badge**: Displays a live minute-by-minute countdown chip on active boards.

### Syntax Highlighting & Heuristic Detection
- **Heuristic Auto-Detection**: Custom pattern scoring engine ([useLanguageDetection.js](src/hooks/useLanguageDetection.js)) automatically detects code language from syntax patterns.
- **15+ Highlighting Modes**: Highlighting powered by PrismJS for JavaScript, Python, Java, C++, Go, PHP, SQL, HTML, CSS, JSON, YAML, Ruby, and Markdown.
- **Manual Language Selector**: Easily switch between Auto-Detect and specific language modes from the editor toolbar.
- **Editor Actions & Metrics**: Real-time word count, letter count (excluding whitespace), line counter, Select All, Copy All, Paste from Clipboard, and Download as File.
- **One-Tap Wipe**: Dedicated "Clear Board" action with Sonner toast verification.

### Board Security & PIN Lock
- **Salted SHA-256 PIN Lock**: Secure any board with a **6-digit PIN** hashed using Web Crypto API `crypto.subtle.digest("SHA-256")` salted with the room code (`${code}:${pin}`).
- **Access Enforcement**: Once locked, guests without the PIN are strictly placed in read-only mode (editing text, uploading files, deleting files, and clearing the board are disabled).
- **Silent Anonymous Auth**: Every visitor is silently authenticated via Firebase Anonymous Authentication, satisfying database security rules (`auth != null`) without collecting emails or passwords.

### Progressive Web App (PWA) & Offline Resilience
- **Installable Desktop & Mobile App**: Powered by `vite-plugin-pwa` and Workbox with standalone display mode.
- **Offline Detection**: Real-time network listener (`navigator.onLine`) alerts users when connection is interrupted.
- **Service Worker Updates**: Automatic update notification toast prompting the user to reload when a new version is deployed.
- **Layout Stability**: Styled with CSS `scrollbar-gutter: stable` and unified container geometry to eliminate layout shift during navigation.

---

## Project Architecture

The codebase follows a modular structure organized by responsibility:

```text
SyncBoard/
├── public/                       # Static assets and PWA configuration
│   ├── assets/                   # App icons, avatars, and illustrations
│   ├── favicon/                  # Cross-platform icons and PWA assets
│   ├── _redirects                # SPA routing rewrites
│   ├── offline.html              # PWA offline fallback page
│   ├── robots.txt                # Search engine crawler policies
│   └── sitemap.xml               # Production SEO sitemap
├── src/
│   ├── components/
│   │   ├── common/               # Global components (Navbar, ClipNavbar, Footer, QRCodeModal, etc.)
│   │   └── features/             # Feature modules (TextEditor, FilePanel, Hero, SelfDestructTimer, etc.)
│   ├── config/                   # Central configuration (firebase.js)
│   ├── constants/                # Static data (developers.js, faqs.js, termsContent.js)
│   ├── hooks/                    # Reusable React hooks (useLanguageDetection.js)
│   ├── pages/                    # Route pages (Home, Clipboard, Explore, Feedback, PrivacyPolicy, TermsConditions)
│   ├── services/                 # Backend API services (firebaseService, cloudinaryService, analyticsService, authService)
│   ├── utils/                    # Utility functions (sanitize.js, userIdentifier.js)
│   ├── App.css                   # App layout styles
│   ├── App.jsx                   # Route configurations and code splitting
│   ├── index.css                 # Global theme variables, typography, and scrollbar styles
│   ├── main.jsx                  # React DOM entry point, MUI ThemeProvider, HelmetProvider
│   └── prism.css                 # PrismJS syntax highlighting theme
├── validator/                    # Maintenance & data integrity tooling
│   └── script.py                 # Python validator & archiver for Firebase RTDB exports
├── .env                          # Local environment secrets (not committed)
├── .eslintrc.cjs                 # ESLint rules
├── example.env                   # Template for environment configuration
├── index.html                    # HTML entry point with meta tags and schema markup
├── LICENSE                       # MIT License
├── package.json                  # Dependencies and build scripts
├── vercel.json                   # Vercel deployment rewrites
└── vite.config.js                # Vite build, PWA plugin, and manual chunking rules
```

---

## Technology Stack

| Layer | Technology | Version | Purpose |
| :--- | :--- | :--- | :--- |
| **Framework** | [React](https://react.dev/) | `19.2.5` | Core UI library |
| **Bundler** | [Vite](https://vitejs.dev/) | `7.3.3` | Fast development server & production bundler |
| **Routing** | [React Router DOM](https://reactrouter.com/) | `7.15.0` | Client-side route management |
| **UI Components** | [Material UI (MUI)](https://mui.com/) | `9.0.0` | Modern responsive UI system & dark/light theme |
| **Styling** | [Emotion](https://emotion.sh/) | `11.14` | CSS-in-JS engine for MUI components |
| **Icons** | [Lucide React](https://lucide.dev/) & MUI Icons | `^1.14.0` / `9.0.0` | Vector icons across desktop and mobile UI |
| **Database** | [Firebase Realtime Database](https://firebase.google.com/) | `12.9.0` | Low-latency atomic text sync & presence tracking |
| **Authentication** | [Firebase Anonymous Auth](https://firebase.google.com/) | `12.9.0` | Silent authentication ensuring secure database rules |
| **Storage** | [Cloudinary](https://cloudinary.com/) | REST API | Multi-format asset storage & CDN delivery |
| **PWA & Caching** | [Vite PWA](https://vite-pwa-org.netlify.app/) & [Workbox](https://developer.chrome.com/docs/workbox/) | `1.3.0` | Offline service worker & web app manifest |
| **Syntax Highlighting** | [PrismJS](https://prismjs.com/) | `1.30.0` | Multi-language syntax formatting |
| **Notifications** | [Sonner](https://sonner.emilkowal.ski/) | `2.0.7` | Toast notification system |
| **Form Management** | [React Hook Form](https://react-hook-form.com/) & [Yup](https://github.com/jquense/yup) | `7.52.2` / `1.4.0` | Feedback form validation |
| **SEO & Head** | [React Helmet Async](https://github.com/staylor/react-helmet-async) | `3.0.0` | Document head and OpenGraph meta management |
| **QR Generation** | [qrcode.react](https://github.com/zpao/qrcode.react) | `4.2.0` | Dynamic SVG/Canvas QR code rendering |
| **Contact Delivery** | [EmailJS](https://www.emailjs.com/) | `4.1.0` | Direct feedback email submission |

---

## Performance Optimizations

- **Route-Level Code Splitting**: Main views are loaded dynamically via `React.lazy()` with a lightweight fallback loader.
- **Idle Route Prefetching**: Secondary routes (`/explore`, `/feedback`) are pre-fetched during idle time using `requestIdleCallback`.
- **Manual Rollup Chunks**: `vite.config.js` splits third-party dependencies into separate chunks (`mui`, `firebase`, `router`, `react-core`), cutting initial bundle payload by over **85%**.
- **Debounced Pattern Scoring**: Language detection runs with a 300ms debounce on input changes to eliminate UI thread lockup.
- **Zero PII Collection**: Removed IP geolocation and third-party tracking scripts to maximize privacy and eliminate external latency.

---

## Getting Started

### Prerequisites
- **Node.js**: v18.0.0 or higher
- **npm**: v9.0.0 or higher
- (Optional) **Python 3.8+** for the database validator utility

### 1. Clone the Repository
```bash
git clone https://github.com/i-viki/sync-board.git
cd SyncBoard
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Environment Configuration
Create a `.env` file in the root directory based on `example.env`:
```bash
cp example.env .env
```

Populate the required credentials in `.env`:
```env
# Firebase Configuration
VITE_APP_APIKEY=your_firebase_api_key
VITE_APP_AUTHDOMAIN=your_project.firebaseapp.com
VITE_APP_PROJECTID=your_project_id
VITE_APP_STORAGEBUCKET=your_project.firebasestorage.app
VITE_APP_MESSAGINGSENDERID=your_messaging_sender_id
VITE_APP_APPID=your_app_id
VITE_APP_DATABASEURL=https://your_project-default-rtdb.asia-southeast1.firebasedatabase.app/
VITE_APP_MEASUREMENT_ID=your_measurement_id

# EmailJS Configuration
VITE_APP_EMAILJSAPIKEY=your_emailjs_public_key
VITE_APP_EMAILJSSERVICE_KEY=your_service_id
VITE_APP_EMAILJSTEMPLATE_KEY=your_template_id

# Cloudinary Configuration
VITE_APP_CLOUDINARY_CLOUD_NAME=your_cloud_name
VITE_APP_CLOUDINARY_CLOUD_UPLOAD_PRESET_NAME=your_unsigned_preset
```

### 4. Firebase Backend Configuration
1. **Enable Anonymous Authentication**:
   - Go to **Firebase Console** → **Authentication** → **Sign-in method**.
   - Enable **Anonymous** authentication.
2. **Configure Realtime Database Security Rules**:
   - Go to **Firebase Console** → **Realtime Database** → **Rules**.
   - Apply the following rules requiring authentication:
   ```json
   {
     "rules": {
       "rooms": {
         "$roomId": {
           ".read": "auth != null",
           ".write": "auth != null"
         }
       }
     }
   }
   ```

### 5. Run Development Server
```bash
npm run dev
```
The application will launch at `http://localhost:3000`.

---

## Available Scripts

| Script | Command | Purpose |
| :--- | :--- | :--- |
| `dev` | `npm run dev` | Runs the Vite dev server with host binding (`--host`) on port `3000` |
| `build` | `npm run build` | Compiles production assets into `dist/` with PWA service worker |
| `preview` | `npm run preview` | Starts a local server to preview the production build on port `3000` |
| `lint` | `npm run lint` | Runs ESLint across all `.js` and `.jsx` files |

---

## Database Validator & Maintenance Utility

SyncBoard includes a dedicated Python maintenance script in [`validator/script.py`](validator/script.py) for auditing Firebase Realtime Database exports:
- Validates 5-character board code formatting.
- Verifies database schema conformance (`text`, `images`, `lastUpdated`, `users`, `expirationTime`, `passwordHash`).
- Computes platform metrics (total boards, unique active users, total files, locked boards, ephemeral boards).
- Automatically archives audited export files into `validator/passed/` or `validator/failed/` with IST timestamps.

To run the validator:
```bash
python validator/script.py
```

---

## Maintainers & Contributors

<table>
  <tr>
    <td align="center" width="200">
      <a href="https://jayavignesh.dev">
        <img src="https://isyncboard.vercel.app/assets/avatar-Jv.webp" width="80px;" alt="Jayavignesh" style="border-radius: 50%;" /><br />
        <sub><b>Jayavignesh</b></sub>
      </a><br />
      <sub>Superuser & Lead Developer</sub><br />
      <a href="https://jayavignesh.dev">Portfolio</a> • <a href="https://www.linkedin.com/in/jayavigneshj/">LinkedIn</a>
    </td>
    <td align="center" width="200">
      <a href="https://github.com/Karthick-1501">
        <img src="https://isyncboard.vercel.app/assets/avatar-sk.png" width="80px;" alt="Karthick S" style="border-radius: 50%;" /><br />
        <sub><b>Karthick S</b></sub>
      </a><br />
      <sub>QA & Content / Contributor</sub><br />
      <a href="https://karthick.at">Portfolio</a> • <a href="https://linkedin.com/in/karthicks1520">LinkedIn</a> • <a href="https://github.com/Karthick-1501">GitHub</a>
    </td>
  </tr>
</table>

---

## Contributing

Contributions, feedback, and pull requests are welcome!
1. Fork the repository.
2. Create your branch (`git checkout -b feature/your-feature`).
3. Commit your changes (`git commit -m 'feat: add your feature'`).
4. Push to your branch (`git push origin feature/your-feature`).
5. Open a Pull Request.

Please review our [Code of Conduct](CODE_OF_CONDUCT.md) before contributing.

---

## Security & Contact

For support, feedback, or vulnerability disclosures, reach out via the in-app [Feedback Form](https://isyncboard.vercel.app/feedback) or email [hello@jayavignesh.dev](mailto:hello@jayavignesh.dev). See our [Privacy Policy](PRIVACYPOLICY.md) and [Terms and Conditions](TERMSCONDITIONS.md).

---

## License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.

---

**SyncBoard** — *Collaborate at the speed of thought.*
