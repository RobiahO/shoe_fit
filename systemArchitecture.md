ShoeFit — Technical Blueprint
 1. Overview
ShoeFit is a web-first, mobile-friendly app that uses a phone camera to capture foot images, runs measurement extraction, and returns recommended sizes across brands. The system is designed to be **offline-friendly**, **scalable**, and **easily integratable** by vendors (widget / API).

2. Core Components & Tech Stack

 Frontend

- Framework: React with Next.js (SSR where useful for SEO; client-side for camera/scan flows).
- UI library:TailwindCSS or simple component library (keeps bundle small).
- Language: TypeScript.
- Key responsibilities:
- Authentication UI (email / Google / Apple).
- Guided camera flow for scanning feet (in-browser camera API).
-Offline-first behavior (Service Worker + IndexedDB/localStorage).
- Displaying measurement results and brand-size recommendations.
- Export/share (QR/link) and vendor widget embedding.

Backend (API)
- Runtime / Framework: Node.js + Express (TypeScript).
- Authentication:JWT for session tokens; OAuth2 for Google/Apple.
-Endpoints :
- User signup & login
- Camera access / foot scanning`
-Displaying recommended sizes
- Saving and viewing user size histor
-vendor integration register

Key responsibilities:
- Validate incoming requests and forward images to the ML measurement service.
- Apply business logic (combine measurements + brand rules).
- Persist profiles and scan metadata.
- Provide vendor integration endpoints and analytics.

Database Layer
- **MongoDB** (Cloud DB e.g., MongoDB Atlas)
- Stores:
- User profile
- Foot measurement data
- Size recommendations
- vendor integrations

3. Component Communication & Sequence
 Typical scan flow
1. User clicks **Start Scan** → frontend prompts camera permission.
2. Frontend captures images and either:
   - (Client inference) runs TF.js model → produces `measurements` → `POST /api/scan` with JSON measurements.
   - (Server inference) uploads images → `POST /api/scan` with images.
3. Backend receives payload → stores scan metadata → forwards images (if present) to ML service (synchronous job or queued).
4. ML service returns measurement results → Backend stores results in `scans` collection.
5. Backend calls Size Mapping Service to convert measurements to brand sizes.
6. Backend returns `results` to frontend; frontend displays Fit Profile and brand suggestions.
 Asynchronous / Queue
- For server inference, use a job queue (RabbitMQ / Bull + Redis) to process images and avoid long HTTP waits. Notify user via WebSocket or polling when ready.

 Offline-first & Mobile Constraints
- Client-side:Save scan images/measurements to IndexedDB when offline. Sync when connection returns.
- For resource-constrained areas (Nigeria): Prefer client-side inference (TensorFlow.js) to avoid heavy uploads and support offline scanning.
- Progressive Web App (PWA): Service Worker to cache assets and saved profiles.

Vendor Integration / Widget
- Widget: small JS snippet vendors embed (`<script src="...">`) that opens a modal for quick scan or pastes a user’s fit profile into checkout.
- API keys: Vendor registers via dashboard to get API key; calls backend to verify size profiles.
- Security: vendor webhook signature verification and per-vendor rate limits.

Security & Privacy
- Data minimization: store measurements and a link to image; allow users to delete scans/profiles.
- Encryption: HTTPS everywhere; at-rest encryption for S3/MongoDB.
- Auth: JWT tokens with short expiry + refresh tokens.
- GDPR/Local compliance: user consent during scan; clear privacy policy; ability to export/delete user data.

Why this approach is technically feasible
- Our approach is realistic because we are not trying to build a complex system from scratch,  we are combining tools and technologies that already exist and are proven to work.
- Smartphone cameras are already powerful enough to take clear images needed for measurement.
- TensorFlow.js allows us to run simple machine-learning calculations directly on the user’s device. This means we don’t need expensive servers to process every scan.
- React + Next.js makes it easy to build a responsive web app that works on both phones and laptops.
- Node.js (backend) and MongoDB (database) are widely used and make it simple to store user profiles, measurements, and size histories.
- The app works even with poor internet or no internet because some of the processing happens on the device (offline-first approach).
- Shoe brands already use size conversion charts, so connecting measurements to size recommendations is straightforward, we just store their sizing rules in the database and apply them.
By leveraging existing technologies instead of inventing new ones, we reduce cost, complexity, and development time. This makes ShoeFit practical, achievable, and easy to improve over time.

**User flow**
![User Flow](./assets/shoefituseflow.png)

### UI Screens

**Welcome**
![Welcome](./assets/welcome.jpg)

**Onboarding**
![Onboarding](./assets/onboarding.jpg)

**Log in/ Sign up**
![Login/ Sign up](./assets/loginsignup.jpg)

**Camera Permission**
![Camera Permission](./assets/camerapermission.jpg)

**Foot Scan**
![Foot Scan](./assets/footscan.jpg)

**Review Measurements**
![Review Measurements](./assets/reviewmeasurements.jpg)

**Size Recommendation**
![Size Recommendation](./assets/sizerecommendation.jpg)

**Profile**
![Profile](./assets/profile.jpg)
