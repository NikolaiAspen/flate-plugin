---
name: ny-runde
description: Opprett en ny Flate-gjennomgangsrunde med flater (URL-er) og få delingslenken til kunden. Bruk når brukeren vil dele en preview med en kunde eller kjører /flate:ny-runde.
argument-hint: "<rundenavn> <url> [url ...]"
---

# Flate: ny runde

Får du 401/uautentisert fra `mcp__flate__*`: be brukeren kjøre `/mcp` → `flate` → Authenticate, og prøv igjen. Kjenner du ikke Flate fra før i denne økten: les `../oversikt/SKILL.md` (relativt til denne skillens mappe).

1. Argumenter: første ord er rundenavnet (bruk «Runde N – <dato>» hvis tomt), resten er URL-er. Mangler URL-er: sjekk om prosjektet er koblet til GitHub (`githubRepo` i `mcp__flate__list_projects`). Er det det, kan du lage runden fra branchen du står på (`git branch --show-current`) — se steg 4b. Ellers: spør etter preview-adressen(e).
2. Enhet per URL: sti som inneholder `/app` → én flate `iphone` og én `ipad` med samme URL; ellers `desktop`. Si hva du valgte; brukeren kan overstyre.
3. `mcp__flate__list_projects` → velg prosjektet som matcher dette repoet (`githubRepo` mot `git remote get-url origin`, ellers mappenavn). Finnes ingen: `mcp__flate__create_project` med repoets mappenavn som `name` og `eier/repo` fra origin-remoten som `github_repo`, så senere runder matcher repoet.
4. `mcp__flate__create_round` med `project_id`, navn og `surfaces` (URL + enhet).
   4b. **Branch i stedet for URL:** har prosjektet `githubRepo`, og brukeren nevner en branch eller ikke oppga URL, kall `create_round` med `branch` (og `device`, standard desktop) i stedet for `surfaces`. Flate finner deploy-previewen som Vercel/Netlify/Railway har postet som GitHub Deployment. Svarer verktøyet «ikke koblet til GitHub»: be brukeren koble til i portalen (prosjektet → Innstillinger → Koble til GitHub) eller oppgi URL. Svarer det «ingen deploy-preview»: branchen må deployes først, eller oppgi URL.
5. `mcp__flate__create_share_link` med `round_id` (passord/utløp bare hvis brukeren ber om det).
6. Skriv ut delingslenken tydelig, og to setninger kunden kan få: «Åpne lenken, skriv navnet ditt, slå på Kommentér og pek på det du vil endre.»
7. Minn om at appen på URL-en må ha `@flateapp/sdk` (`^0.3.0`) montert (`/flate:installer`) og tillate portalen i `connect-src` hvis appen har CSP. Krever URL-en(e) innlogging, logger kunden inn i appen som vanlig (også Vipps/BankID) — men kunden trenger en konto; nevn det, eller pek runden på offentlige ruter.
