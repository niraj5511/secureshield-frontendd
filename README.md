# SecureShield Frontend

The web frontend of SecureShield, a phishing and scam detection app. You paste a URL or a message, the backend model analyses it, and the app shows the verdict, the reasons, and a risk badge. Built with React and Vite.

This repository contains the frontend only. The detection model and the REST API run in a separate backend, which is not included here.

**Live demo:** https://secureshielddetection.netlify.app/

<!-- Add screenshots here: ![Dashboard](screenshots/dashboard.png) -->

## Features

- **URL Scanner:** scan a link and see the verdict, reasons, and a colour-coded risk badge
- **Message Scanner:** scan an SMS or email text (10 to 1000 characters). URLs found in the message are listed with their own verdicts, as returned by the backend
- **PDF report:** download a report for any scan, generated in the browser
- **Feedback:** mark a scan as accurate or not accurate, and send a written comment
- **Dashboard:** stat cards (total scans, phishing URLs, scam messages, safe detections), a 7-day chart, a pie chart, and recent scans
- **History** (login required): search, date filters, scan details, delete, and export the filtered list to a styled Excel file
- **Guest mode:** scan without an account. Guest scans are saved only in that browser
- **Accounts:** register, log in, log out, edit your name and email, and change your password
- **Command palette:** press `Ctrl+K` (or `Cmd+K` on Mac) to jump between pages and run quick actions
- **Light, dark, and system themes**, remembered between visits
- Responsive layout, skip-to-content link, and `prefers-reduced-motion` support

## Tech Stack

- React 18 and React Router 6
- Vite 4
- Axios for API calls
- Recharts for charts
- jsPDF and jspdf-autotable for PDF reports
- xlsx-js-style for Excel export
- react-hot-toast and react-icons
- Plain CSS (no UI framework)

## Project Structure

```
secureshield-frontend/
├── index.html
├── vite.config.js
├── public/
│   ├── _redirects              # Netlify single-page-app fallback
│   ├── manifest.webmanifest
│   └── favicon.svg
└── src/
    ├── main.jsx
    ├── App.jsx                 # Routes and layout
    ├── pages/                  # Dashboard, URLScanner, MessageScanner, History, Profile, Login, Register, NotFound
    ├── components/common/      # Navbar, Footer, CommandPalette, ProtectedRoute, ErrorBoundary, ThemeToggle, ...
    ├── context/                # AuthContext, GuestContext, ThemeContext
    ├── services/               # api.js, pdfGenerator.js, loadingTracker.js
    ├── utils/                  # exportExcel.js, feedback.js, formatters.js, ...
    └── styles/                 # globals.css, components.css, mobile.css
```

For a longer walkthrough of the screens and files, see [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md).

## Getting Started

You need Node.js 16 or newer (18 or newer is recommended) and a running SecureShield backend.

1. Clone the repository:
   ```bash
   git clone https://github.com/niraj5511/secureshield-frontend.git
   cd secureshield-frontend
   ```
2. Install dependencies:
   ```bash
   npm install
   ```
3. Create a `.env` file in the project root and set the backend address:
   ```
   VITE_API_BASE_URL=https://your-backend-url
   ```
4. Start the dev server:
   ```bash
   npm run dev
   ```
5. Open http://localhost:5173.

### Scripts

```bash
npm run dev       # Development server
npm run build     # Production build in dist/
npm run preview   # Preview the production build
```

## Backend API

All calls go through `src/services/api.js`. The frontend expects these endpoints:

| Method | Endpoint | Used for |
|---|---|---|
| POST | `/auth/register`, `/auth/login`, `/auth/logout` | Account access |
| PUT | `/auth/profile`, `/auth/change-password` | Profile updates |
| POST | `/scans/url`, `/scans/messages` | Run a scan |
| GET | `/scans`, `/scans/{reference}` | History and scan details |
| DELETE | `/scans/{reference}` | Delete a scan |
| GET | `/scans/{reference}/report` | Server-side report file |
| GET | `/dashboard/stats` | Dashboard numbers |
| POST | `/feedback`, `/feedback/accuracy` | Comments and accuracy ratings |

The request layer sends the saved JWT as a bearer token, shows a loading bar, retries failed GET requests up to twice (for network errors and 429, 502, 503, and 504 responses), and turns errors into readable messages. On load, the app sends a background request to wake a sleeping free-tier backend.

## Browser Storage

| Key | Contents |
|---|---|
| `accessToken` | JWT of the logged-in user |
| `user` | Name, email, and role of the logged-in user |
| `guest_data` | Guest session ID and guest scans |
| `scan_feedback` | Feedback given per scan |
| `appTheme` | Theme choice |

## Deployment

The build works on Netlify. Set `VITE_API_BASE_URL` in the site's environment variables, use `npm run build` as the build command, and publish the `dist` folder. `public/_redirects` makes page routes like `/history` work on refresh.

## Known Limitations

- Scanning needs the backend. If it is asleep or down, scans fail with an error message.
- Guest scans are stored only in the browser and are lost if site data is cleared. Guests cannot open the History page.
- Feedback comments are sent to the backend, but they are also stored locally by scan reference. Only the accurate/not accurate rating is linked to the scan on the server side.
- The app has a web app manifest but no service worker, so it does not work offline.
- There are no automated tests.
