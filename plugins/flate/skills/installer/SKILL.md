---
name: installer
description: Legg Flate-SDK-en inn i dette repoet (Next.js, Expo Router eller Vite/React) så kunder kan peke og kommentere i appen. Bruk når brukeren kjører /flate:installer eller ber om å «koble appen til Flate».
disable-model-invocation: true
---

# Flate: installer SDK-en i appen

Portal-origin er `https://portal-production-c530.up.railway.app` med mindre brukeren sier noe annet.

1. Finn rammeverk: `app/layout.tsx` eller `pages/_app.tsx` → Next.js; `app/_layout.tsx` + `expo` i package.json → Expo Router; `vite.config.*` → Vite/React. Monorepo: gjør det per app brukeren peker på.
2. `npm install @flateapp/sdk` i riktig workspace (eller pnpm/yarn hvis repoet bruker det).
3. Monter `<FlateBridge origin={process.env.NEXT_PUBLIC_FLATE_ORIGIN ?? "<portal-origin>"} />`:
   - Next app router: inni `<body>` i `app/layout.tsx`, etter `{children}`.
   - Next pages: i `pages/_app.tsx` ved siden av `<Component />`.
   - Expo Router: i `app/_layout.tsx` som søsken til rot-navigatoren (`EXPO_PUBLIC_FLATE_ORIGIN`).
   - Vite: i `main.tsx` under `<App />`.
   Import: `import { FlateBridge } from "@flateapp/sdk";`. SDK-en er no-op utenfor portalens iframe og på native.
4. CSP: appen må kunne vises i iframe fra portalen. Next: i `headers()` i `next.config.*`, legg `frame-ancestors 'self' <portal-origin>` inn i eksisterende `Content-Security-Policy` (erstatt `frame-ancestors 'none'`; fjern `X-Frame-Options: DENY`). Behold alle andre direktiver. Andre rammeverk: forklar hvor headeren settes (hosting/proxy).
5. Legg `NEXT_PUBLIC_FLATE_ORIGIN` / `EXPO_PUBLIC_FLATE_ORIGIN` i `.env.example` med portal-origin som verdi.
6. Kjør typecheck. Oppsummer endringene og si at appen må deployes (preview) før den kan brukes i en runde – deretter `/flate:ny-runde`.
