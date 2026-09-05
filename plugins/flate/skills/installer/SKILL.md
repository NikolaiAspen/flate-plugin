---
name: installer
description: Koble dette repoet til Flate så kunder kan peke og kommentere i appen. Dekker Next.js, Expo Router, Vite/React, monorepo med både web- og expo-app, innloggingsvegg og feilsøking. Bruk når brukeren kjører /flate:installer eller ber om å «koble appen til Flate».
disable-model-invocation: true
---

# Flate: koble appen til

Portal-origin: `https://portal-production-c530.up.railway.app` (med mindre brukeren sier annet).
SDK-en `@flateapp/sdk` ligger på npm. Den er **no-op** utenfor portalens iframe og på native, så
den er trygg å committe til `main` — da har alle brancher den.

## Steg

1. **Finn rammeverk(ene).** `app/layout.tsx` eller `pages/_app.tsx` → Next.js; `app/_layout.tsx` +
   `expo` i package.json → Expo Router; `vite.config.*` → Vite/React. **Sjekk om det er flere apper**
   som krysslenker: ett repo kan ha BÅDE en Next-web-app OG en Expo-web-app der ruter sender brukeren
   mellom appene (f.eks. Next serverer `/hjem`, Expo serverer `/app/*` og innlogging). Da må bridgen
   inn i BEGGE — se «Krysslenkede apper».

2. **Installer:** `npm install @flateapp/sdk` i riktig workspace (pnpm/yarn hvis repoet bruker det).
   Merk: `npm install` bygger lockfilen på nytt. Er en avhengighet importert i koden men bare lå i
   lockfilen (ikke deklarert i noen `package.json`), kan den bli pruned og bygget ryke. Verifiser
   bygget i steg 6; må en manglende deklarasjon legges til, gjør det eksplisitt.

3. **Monter `<FlateBridge/>`.** Import: `import { FlateBridge } from "@flateapp/sdk";`
   - **Next app router:** inni `<body>` i `app/layout.tsx`, etter `{children}`:
     `<FlateBridge origin={process.env.NEXT_PUBLIC_FLATE_ORIGIN ?? "<portal-origin>"} />`
   - **Next pages:** i `pages/_app.tsx` ved siden av `<Component />`.
   - **Expo Router:** i `app/_layout.tsx` som søsken til rot-navigatoren, **gated på web** så native
     aldri får den: `{Platform.OS === "web" && <FlateBridge origin={process.env.EXPO_PUBLIC_FLATE_ORIGIN ?? "<portal-origin>"} />}`
   - **Vite:** i `main.tsx` under `<App />`.

4. **CSP — la portalen ramme appen.** Next: i `headers()` i `next.config.*`, i den eksisterende
   `Content-Security-Policy`, erstatt `frame-ancestors 'none'` med
   `frame-ancestors 'self' <portal-origin>`, og **fjern `X-Frame-Options: DENY`** (den kjenner ikke
   «tillat fra dette originet» og kan overstyre CSP i enkelte nettlesere). Behold alle andre
   direktiver. Andre rammeverk: sett headeren der hosting/proxy setter den.

5. **env-eksempler:** legg `NEXT_PUBLIC_FLATE_ORIGIN` / `EXPO_PUBLIC_FLATE_ORIGIN` i `.env.example`
   med portal-origin som verdi. (Koden faller tilbake på portal-origin, så variablene er valgfrie.)

6. **Verifiser.** Kjør repoets typecheck, OG bygg flaten kunden faktisk ser. Bygger appen en
   Expo-web-eksport inn i Next-appens `public/` (ofte gitignorert, regenerert i CI/bygget), er en
   **kildeendring nok** — ikke rediger den bygde bundelen; kjør web-eksporten/bygget og bekreft at
   `FlateBridge` er i den ferske bundelen.

7. **Oppsummer.** Si at appen må deployes (til den URL-en runden skal peke på) før den kan brukes,
   deretter `/flate:ny-runde` (eller `/flate:del` for å dele en lokal app uten deploy).

## Krysslenkede apper (Next + Expo i samme repo)

Peker runden på en Next-web-rute (f.eks. `/hjem`) som redirecter uinnloggede til Expo-appens
innloggingsside (`/app/login`), er det Expo-siden kunden faktisk ser. Da må bridgen være montert i
**begge** appene, ellers får kunden «Appen svarer ikke» på innloggingssiden. CSP-en i Next-appen
dekker også statisk servering av `/app`, så CSP settes kun ett sted.

## Innloggingsvegg

Krever mål-rutene innlogging, ser kunden en innloggingsside i stedet for siden som skal vurderes.
Avklar med brukeren: enten (a) gi kunden en test-/kundekonto så de logger inn i iframen — da må
bridgen også være på innloggingssiden — eller (b) pek runden på offentlige ruter. Nevn dette i
oppsummeringen så brukeren ordner en innlogging til kunden.

## Feilsøking: «Appen svarer ikke»

Tre vanlige årsaker, sjekk i rekkefølge:

1. **Iframe blokkert.** `frame-ancestors` mangler portalen, eller `X-Frame-Options: DENY` står igjen.
   Sjekk svar-headerne: `curl -sI <url> | grep -iE 'content-security-policy|x-frame-options'`.
2. **Ingen bridge på siden som rendres.** Bridgen må være montert i akkurat den appen/ruten kunden
   lander på. Husk redirects til en annen app (f.eks. Expo `/app/login`) — se «Krysslenkede apper».
3. **Ruten krever innlogging.** Kunden bounces til login. Se «Innloggingsvegg».
