---
name: installer
description: Koble dette repoet til Flate så kunder kan peke og kommentere i den ekte appen. Dekker Next.js, Expo Router, Vite/React, monorepo med både web- og expo-app, CSP (connect-src), innloggingsvegg og feilsøking. Bruk når brukeren kjører /flate:installer eller ber om å «koble appen til Flate».
disable-model-invocation: true
---

# Flate: koble appen til

Portal-origin: `https://portal-production-c530.up.railway.app` (med mindre brukeren sier annet).
SDK-en `@flateapp/sdk` (krev `^0.3.0`) ligger på npm.

**Slik virker det:** kunden åpner delingslenken i portalen, skriver navnet sitt, og sendes videre
til den ekte appen med et kortlevd token i URL-fragmentet (`#flate=…`). SDK-en leser tokenet,
fjerner det fra adressen, og tegner gjennomgangs-UI-et (kommandobar, pinner, kommentarpanel) i
shadow-DOM oppå appen. Appen kjører på toppnivå — ingen iframe — så innlogging, også Vipps,
BankID og Google, virker som vanlig. Uten token, og utenfor portalens ramme, er SDK-en **no-op**
(og alltid no-op på native), så den er trygg å committe til `main`.

## Steg

1. **Finn rammeverk(ene).** `app/layout.tsx` eller `pages/_app.tsx` → Next.js; `app/_layout.tsx` +
   `expo` i package.json → Expo Router; `vite.config.*` → Vite/React. **Sjekk om det er flere apper**
   som krysslenker: ett repo kan ha BÅDE en Next-web-app OG en Expo-web-app der ruter sender brukeren
   mellom appene (f.eks. Next serverer `/hjem`, Expo serverer `/app/*` og innlogging). Da må bridgen
   inn i BEGGE — se «Krysslenkede apper».

2. **Installer:** `npm install @flateapp/sdk@^0.3.0` i riktig workspace (pnpm/yarn hvis repoet bruker
   det). Merk: `npm install` bygger lockfilen på nytt. Er en avhengighet importert i koden men bare lå
   i lockfilen (ikke deklarert i noen `package.json`), kan den bli pruned og bygget ryke. Verifiser
   bygget i steg 6; må en manglende deklarasjon legges til, gjør det eksplisitt.

3. **Monter `<FlateBridge/>`.** Import: `import { FlateBridge } from "@flateapp/sdk";`
   - **Next app router:** inni `<body>` i `app/layout.tsx`, etter `{children}`:
     `<FlateBridge origin={process.env.NEXT_PUBLIC_FLATE_ORIGIN ?? "<portal-origin>"} />`
   - **Next pages:** i `pages/_app.tsx` ved siden av `<Component />`.
   - **Expo Router:** i `app/_layout.tsx` som søsken til rot-navigatoren, **gated på web** så native
     aldri får den: `{Platform.OS === "web" && <FlateBridge origin={process.env.EXPO_PUBLIC_FLATE_ORIGIN ?? "<portal-origin>"} />}`
   - **Vite:** i `main.tsx` under `<App />`.
   `origin` er adressen overlayet snakker med (fetch + EventSource), så den må være portal-origin.

4. **CSP — slipp portalen gjennom `connect-src`.** Har appen en `Content-Security-Policy` med
   `connect-src`, MÅ portal-origin legges til der, ellers får kunden «Appen slipper ikke Flate
   gjennom (connect-src)». Next: i `headers()` i `next.config.*`, f.eks.
   `connect-src 'self' <eksisterende kilder> <portal-origin>`. Behold alle andre direktiver;
   overlayet trenger ingen endring i `style-src`/`script-src`/`img-src` (stilene ligger i shadow-DOM).
   Har appen ingen CSP, trengs ingenting. Andre rammeverk: sett headeren der hosting/proxy setter den.
   *Valgfritt:* portalens gamle rammevisning (`?visning=ramme` på delingslenken) krever i tillegg
   `frame-ancestors 'self' <portal-origin>` og at `X-Frame-Options: DENY` fjernes. Ikke nødvendig
   for vanlig bruk.

5. **env-eksempler:** legg `NEXT_PUBLIC_FLATE_ORIGIN` / `EXPO_PUBLIC_FLATE_ORIGIN` i `.env.example`
   med portal-origin som verdi. (Koden faller tilbake på portal-origin, så variablene er valgfrie.)

6. **Verifiser.** Kjør repoets typecheck, OG bygg flaten kunden faktisk ser. Bygger appen en
   Expo-web-eksport inn i Next-appens `public/` (ofte gitignorert, regenerert i CI/bygget), er en
   **kildeendring nok** — ikke rediger den bygde bundelen; kjør web-eksporten/bygget og bekreft at
   `FlateBridge` er i den ferske bundelen. Etter deploy: `curl -sI <url> | grep -i content-security-policy`
   skal vise portal-origin i `connect-src` (hvis appen har CSP).

7. **Oppsummer.** Si at appen må deployes (til den URL-en runden skal peke på) før den kan brukes,
   deretter `/flate:ny-runde` (eller `/flate:del` for å dele en lokal app uten deploy).

## Krysslenkede apper (Next + Expo i samme repo)

Peker runden på en Next-web-rute (f.eks. `/hjem`) som redirecter uinnloggede til Expo-appens
innloggingsside (`/app/login`), er det Expo-siden kunden faktisk lander på. Tokenet ligger i
`sessionStorage` på appens opphav og overlever navigasjon mellom appene, men overlayet må monteres
på nytt av bridgen i appen kunden står i. Derfor må bridgen være montert i **begge** appene, ellers
vises ingenting på den ene siden. CSP-en i Next-appen dekker også statisk servering av `/app`, så
`connect-src` settes kun ett sted.

## Innloggingsvegg

Krever mål-rutene innlogging, logger kunden inn i appen som vanlig — også med Vipps, BankID eller
Google — fordi appen kjører på toppnivå. Tokenet ligger i `sessionStorage` og overlever
innloggingsflyten så lenge den skjer i samme fane. Kunden trenger fortsatt en konto: avklar med
brukeren om kunden får en test-/kundekonto, eller om runden skal peke på offentlige ruter, og nevn
det i oppsummeringen. Åpner innloggingen et nytt vindu/fane og lander der, mangler tokenet — kunden
åpner da delingslenken på nytt (den gir nytt token).

## Feilsøking

Vanlige symptomer, sjekk i rekkefølge:

1. **«Appen slipper ikke Flate gjennom (connect-src)».** CSP-en mangler portal-origin i
   `connect-src`. Sjekk: `curl -sI <url> | grep -i content-security-policy`. Se steg 4.
2. **Ingenting vises i appen.** (a) Bridgen er ikke montert i akkurat den appen/ruten kunden lander
   på — husk redirects til en annen app (se «Krysslenkede apper»). (b) SDK-en er eldre enn 0.3.0 —
   sjekk `package.json` og den deployede bundelen. (c) Kunden åpnet app-URL-en direkte i stedet for
   delingslenken, så det finnes ikke noe token — be dem åpne delingslenken.
3. **«Lenken er utløpt. Åpne delingslenken på nytt.»** Tokenet varer 12 timer. Delingslenken gir nytt.
4. **Rammevisning (`?visning=ramme`) viser «Appen svarer ikke».** Kun for den gamle iframe-modusen:
   `frame-ancestors` mangler portalen eller `X-Frame-Options: DENY` står igjen. Vanlig bruk trenger
   ikke dette — be kunden bruke delingslenken uten `?visning=ramme`.
