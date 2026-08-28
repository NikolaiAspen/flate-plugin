---
name: ny-runde
description: Opprett en ny Flate-gjennomgangsrunde med flater (URL-er) og få delingslenken til kunden. Bruk når brukeren vil dele en preview med en kunde eller kjører /flate:ny-runde.
disable-model-invocation: true
argument-hint: "<rundenavn> <url> [url ...]"
---

# Flate: ny runde

1. Argumenter: første ord er rundenavnet (bruk «Runde N – <dato>» hvis tomt), resten er URL-er. Mangler URL-er: spør etter preview-adressen(e).
2. Enhet per URL: sti som inneholder `/app` → én flate `iphone` og én `ipad` med samme URL; ellers `desktop`. Si hva du valgte; brukeren kan overstyre.
3. `mcp__flate__list_projects` → velg prosjektet som matcher dette repoet; finnes ingen: `mcp__flate__create_project` med repoets mappenavn.
4. `mcp__flate__create_round` med `project_id`, navn og flater.
5. `mcp__flate__create_share_link` med `round_id` (passord/utløp bare hvis brukeren ber om det).
6. Skriv ut delingslenken tydelig, og to setninger kunden kan få: «Åpne lenken, skriv navnet ditt, slå på Kommentér og pek på det du vil endre.»
7. Minn om at appen på URL-en må ha `@flateapp/sdk` montert (`/flate:installer`) og tillate portalen i `frame-ancestors`.
