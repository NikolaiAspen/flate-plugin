---
name: del
description: Del appen du utvikler LOKALT med en kunde – uten å deploye. Tunnelerer localhost til en offentlig URL og lager en Flate-runde i én kommando. Bruk når brukeren kjører /flate:del eller sier «del appen jeg kjører lokalt».
disable-model-invocation: true
argument-hint: "[port] [rundenavn]"
---

# Flate: del en app du utvikler lokalt

Poenget: under utvikling er ingenting deployet. Denne skillen gjør den lokale appen delbar for en kunde uten Vercel/Netlify/Railway – ved å åpne en midlertidig offentlig tunnel til `localhost` og lage en Flate-runde mot den.

## Forutsetninger (sjekk kort)
- Appen må allerede kjøre lokalt (dev-server). Gjør den ikke det: be brukeren starte den og oppgi porten.
- `@flateapp/sdk` må være montert (`/flate:installer`) og portalen tillatt i `frame-ancestors`. I dev er dette som regel på plass; hvis appen ikke rammes inn i portalen, kjør `/flate:installer`.

## Slik gjør du det
1. **Finn porten.** Første argument er porten hvis oppgitt. Ellers: prøv å oppdage dev-serveren (`lsof -iTCP -sTCP:LISTEN -P -n | grep -E 'node|next|vite|expo'` eller sjekk vanlige porter 3000/5173/8080/19006/4321). Er det tvetydig, spør brukeren kort hvilken port appen kjører på.
2. **Sørg for cloudflared.** `command -v cloudflared` – finnes den ikke: installer med `brew install cloudflared` (macOS) eller last ned binæren fra https://github.com/cloudflare/cloudflared/releases. Ingen Cloudflare-konto trengs for en quick tunnel.
3. **Start tunnelen i bakgrunnen** og fang den offentlige URL-en:
   `cloudflared tunnel --url http://localhost:<port>` – kjør som bakgrunnsprosess. URL-en dukker opp i stderr som `https://<tilfeldig>.trycloudflare.com` (kan ta 3–10 sek). Les output til du ser den; ikke gå videre uten en URL.
4. **Verifiser at URL-en svarer:** `curl -sI https://<...>.trycloudflare.com` skal gi en HTTP-status (200/3xx). Får du feil, vent noen sekunder og prøv igjen; vedvarer det, meld fra og stopp.
5. **Enhet:** utled fra hva brukeren deler (mobilapp→iphone, nettbrett→ipad, ellers desktop). Spør kort hvis uklart; default iphone for Expo/React Native, desktop ellers.
6. **Lag runden:** `mcp__flate__list_projects` → velg prosjektet som matcher repoet (opprett med `mcp__flate__create_project` og mappenavnet hvis det mangler). `mcp__flate__create_round` med `project_id`, et auto-navn («Runde N») eller andre argument som navn, og én flate `{ name: "<branch/forsiden>", url: "<tunnel-url>", device }`. Sett `brief` hvis brukeren sa hva de vil ha tilbakemelding på.
7. **Delingslenke:** `mcp__flate__create_share_link` med `round_id`. Skriv den ut tydelig.
8. **Oppsummer for brukeren:**
   - Delingslenken (send til kunden – ingen konto: «Åpne lenken, skriv navnet ditt, slå på Kommentér, pek på det du vil endre»).
   - **VIKTIG:** tunnelen (og dermed previewen) lever bare så lenge `cloudflared`-prosessen kjører og appen kjører lokalt. Be brukeren la terminalen stå åpen mens kunden kommenterer. Lukkes den, dør previewen – kjør `/flate:del` på nytt for en ny lenke.
   - Når kunden har kommentert: `/flate:runde` for å hente og implementere kommentarene.

## Merknad om levetid
En trycloudflare quick tunnel er midlertidig og får ny URL hver gang. Fint for en live gjennomgang mens du sitter og jobber; ikke egnet for asynkron kommentering over dager. For varig preview: koble på GitHub-repo (`Prosjektinnstillinger → Koble til GitHub`) og velg branch, eller deploy appen.
