# CLAUDE.md — Bore & Barrel PWA Project Context

> **Project:** Bore & Barrel — Gunshop Progressive Web App (PWA) II  
> **Course:** Praktikum Pemrograman Perangkat Bergerak (Prak PPB) — Modul 5 / BAB VI  
> **Student / Author:** Joshpeter14 (`joshpetersirait@gmail.com`)  
> **GitHub Repository:** [https://github.com/Joshpeter14/PWA2_PrakPPB.git](https://github.com/Joshpeter14/PWA2_PrakPPB.git)  
> **Last Updated:** November 2026

---

## 1. Project Overview & Architecture

Bore & Barrel is an offline-first Single Page Application (SPA) built with:
- **Framework:** React 19 + Vite 8
- **PWA Tooling:** `vite-plugin-pwa` (Workbox Service Worker + Web App Manifest)
- **Styling:** Custom CSS design system (`App.css`, `SplashScreen.css`, sans Tailwind dependency)
- **Icons:** `lucide-react`
- **Backend:** Supabase PostgREST RESTful API (`public.products`, `public.reviews`, `public.favorites`)
- **HTTP Client:** Axios with Supabase authorization interceptors
- **Offline / Persistence:** LocalStorage for drafts (`draftStorage.js`), favorites cache, user identity UUID (`userService.js`), and Service Worker precache (15 core assets).

---

## 2. Active Environment & Supabase Credentials

The application connects to Joshpeter14's personal Supabase database instance:

```env
VITE_API_BASE_URL=https://dowfveikyqltstczugcw.supabase.co
VITE_SUPABASE_PUBLISHABLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImRvd2Z2ZWlreXFsdHN0Y3p1Z2N3Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3OTM0OTIwMjEsImV4cCI6MjEwOTA2ODAyMX0.qWxr0oxB7jq2HTg1UVgDPy1HtiouELbOcFgnDsciKaI
```

### Database Schema in Supabase:
- **`public.products`**:
  - `id` (BIGINT PK IDENTITY)
  - `name` (TEXT), `type` (TEXT), `caliber` (TEXT)
  - `price` (NUMERIC), `description` (TEXT)
  - `specs` (JSONB), `image_url` (TEXT)
  - `created_at` (TIMESTAMPTZ), `updated_at` (TIMESTAMPTZ)
- **`public.reviews`**:
  - `id` (BIGINT PK IDENTITY), `product_id` (BIGINT FK -> products.id ON DELETE CASCADE)
  - `user_id` (UUID, client-generated), `rating` (INT 1-5), `comment` (TEXT), `created_at`
- **`public.favorites`**:
  - `user_id` (UUID), `product_id` (BIGINT FK -> products.id ON DELETE CASCADE), PK `(user_id, product_id)`
- **Row Level Security (RLS)**: Enabled with full `ALL` access policies for both `anon` and `authenticated`.
- **Active Seed Products (6 items)**:
  1. Glock 17 (Pistol, 9mm, \$599)
  2. AK-47 (Rifle, 7.62mm, \$899)
  3. Remington 870 (Shotgun, 12 Gauge, \$449)
  4. AR-15 (Rifle, 5.56mm, \$799)
  5. Desert Eagle (Pistol, .50 AE, \$1,599)
  6. Mossberg 500 (Shotgun, 12 Gauge, \$399)

---

## 3. History of Problems Solved

### A. Broken Image Rendering ("beberapa gambar tidak muncul")
1. **Root Cause 1 — Dead Blob URLs:** The lab template saved temporary `blob:http://localhost:5173/...` preview URLs to Supabase whenever storage upload failed. These blob URLs die upon page refresh or across different devices.
   - **Fix:** Added `fileToOptimizedDataUrl(file)` in `src/utils/helpers.js` that resizes canvas to max 800px and compresses images into persistent, lightweight base64 Data URLs (<150KB). Handled in `CreateGunPage.jsx`, `EditGunPage.jsx`, and `uploadService.js`.
2. **Root Cause 2 — Missing `onError` Fallbacks:** Image tags had no fallback handler when images were unreachable.
   - **Fix:** Added `getDefaultGunImage(type)` in `src/utils/helpers.js` returning local SVG silhouettes (`/guns/pistol.svg`, `/guns/rifle.svg`, `/guns/shotgun.svg`). Added declarative `onError` fallbacks in `GunCard.jsx`, `GunDetail.jsx`, `FavoritesPage.jsx`, `ProfilePage.jsx`, `Header.jsx`, and `SplashScreen.jsx`.
3. **Root Cause 3 — `loading="lazy"` Delayed Rendering:**
   - **Fix:** Replaced `loading="lazy"` with `decoding="async"` in `GunCard.jsx` to eliminate mobile scroll delay.
4. **Root Cause 4 — Auto-Sanitizer for Stored Blobs:**
   - **Fix:** In `src/services/gunService.js` (`normalizeProduct`), any lingering `blob:` URL from database records is automatically intercepted and swapped with `getDefaultGunImage(item.type)`.

### B. Git Account & Remote Migration
1. Git commit author was previously configured as `Chalin` (`chalinrizkymalinda1810@gmail.com`).
   - Cleaned up Windows Credential Manager entries for the old user.
   - Updated `git config --global user.name "Joshpeter14"` and `user.email "joshpetersirait@gmail.com"`.
2. Switched remote origin from the shared lab repository to `https://github.com/Joshpeter14/PWA2_PrakPPB.git`.
3. All commits have been pushed and synced with GitHub `origin/main`.

### C. Dedicated Supabase Setup
- Set up a clean PostgreSQL schema on `dowfveikyqltstczugcw.supabase.co` with permissive RLS policies and initial seed data.
- Updated `.env` and `.env.example`.

---

## 4. Key Files & Directory Structure

```text
PWA2_PrakPPB/
├── .env                          # Local Supabase credentials (git-ignored)
├── .env.example                  # Template with current Supabase credentials
├── CLAUDE.md                     # Context for Claude Code (this file)
├── MODUL 5_Prak PPB_PWA 2.docx.pdf # Lab guide PDF (BAB VI)
├── index.html                    # Root HTML with PWA meta tags & splash hooks
├── vite.config.js                # Vite config + VitePWA plugin
├── public/
│   ├── logo.png                  # Brand logo
│   ├── icon-192.png, icon-512.png # PWA install icons
│   └── guns/                     # Fallback SVGs: pistol.svg, rifle.svg, shotgun.svg
└── src/
    ├── App.jsx                   # Main shell, state-based tab router ('Catalog' | 'Favorites' | 'Profile' | 'About' | 'Contact')
    ├── App.css                   # Global styling system
    ├── config/
    │   └── api.js                # Axios instance with Supabase apikey/bearer interceptor
    ├── services/
    │   ├── gunService.js         # Products CRUD + normalizeProduct helper
    │   ├── uploadService.js      # Image upload + canvas base64 fallback
    │   ├── favoriteService.js    # Supabase public.favorites query & toggle
    │   ├── reviewService.js      # Supabase public.reviews query & mutation
    │   └── userService.js        # Stable anonymous client UUID & profile in localStorage
    ├── hooks/
    │   ├── useGuns.js            # useGuns, useGunById
    │   ├── useFavorites.js       # useFavorites, useToggleFavorite, useIsFavorited
    │   └── useReviews.js         # useReviews, useCreateReview
    ├── pages/
    │   ├── Catalog.jsx           # Products listing, search, category pills, sort, CRUD actions
    │   ├── CreateGunPage.jsx     # 2-step register flow (Image upload/preset -> Specs & Steps form)
    │   ├── EditGunPage.jsx       # Edit product specs, steps, image replacement
    │   ├── FavoritesPage.jsx     # Shortlisted guns from Supabase + localStorage fallback
    │   ├── ProfilePage.jsx       # Operator profile, callsign, bio, avatar, draft statistics
    │   ├── About.jsx             # About page
    │   └── Contact.jsx           # Contact page
    ├── components/
    │   ├── Header.jsx            # Top navbar + favorites badge + Add Hardware CTA
    │   ├── Footer.jsx            # Footer
    │   ├── PWABadge.jsx          # PWA install prompt & offline status toast
    │   ├── gun/
    │   │   ├── GunCard.jsx       # Catalog product card with async decoding & onError fallback
    │   │   └── GunDetail.jsx     # Detail view, reviews list, rating submission form
    │   ├── common/
    │   │   ├── AdvancedFilter.jsx # Search bar, category pills, sort dropdown
    │   │   ├── FavoriteButton.jsx # Toggle heart button with stopPropagation
    │   │   ├── SplashScreen.jsx  # Signature intro animation (auto-fades or skip button)
    │   │   └── SplashScreen.css
    │   └── modals/
    │       └── ConfirmModal.jsx  # Native dialog confirmation modal (delete & draft recovery)
    └── utils/
        ├── helpers.js            # getDefaultGunImage, fileToOptimizedDataUrl, formatPrice, search & sort
        └── draftStorage.js       # Auto-save & resume form drafts in localStorage
```

---

## 5. Development & Build Commands

```bash
# Run development server
npm run dev

# Run production build (validates bundle & generates PWA service worker)
npm run build

# Run code linter
npm run lint

# Preview production build locally
npm run preview
```

---

## 6. Next Steps / Lab Roadmap (BAB VI Sub-bab 6.4.3 & 6.4.4)

1. **Deploy to Vercel**:
   - Log in to [vercel.com](https://vercel.com) using GitHub account `Joshpeter14`.
   - Click **Add New...** -> **Project** -> Import `Joshpeter14/PWA2_PrakPPB`.
   - In **Environment Variables**, set:
     - `VITE_API_BASE_URL` = `https://dowfveikyqltstczugcw.supabase.co`
     - `VITE_SUPABASE_PUBLISHABLE_KEY` = `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImRvd2Z2ZWlreXFsdHN0Y3p1Z2N3Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3OTM0OTIwMjEsImV4cCI6MjEwOTA2ODAyMX0.qWxr0oxB7jq2HTg1UVgDPy1HtiouELbOcFgnDsciKaI`
   - Click **Deploy**.
2. **Mobile Testing & PWA Installation**:
   - Open the live Vercel URL in a mobile browser (Chrome / Safari).
   - Tap **"Add to Home Screen"** / **"Install App"**.
   - Test offline capabilities by enabling Airplane mode.
