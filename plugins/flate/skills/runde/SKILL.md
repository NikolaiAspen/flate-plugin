---
name: runde
description: Hent åpne kundekommentarer fra Flate, implementer dem i dette repoet og meld tilbake status. Bruk når brukeren sier «ta runden», «hent kommentarene fra Flate» eller kjører /flate:runde.
disable-model-invocation: true
argument-hint: "[rundenavn eller runde-id]"
---

# Flate: gjennomfør en gjennomgangsrunde

Du er koblet til Flate via MCP-serveren `flate`. Verktøyene heter `mcp__flate__<navn>`.

## Slik gjør du det
1. Kall `mcp__flate__list_rounds`. Får du feil om `project_id`: kall `mcp__flate__list_projects` og velg prosjektet som matcher dette repoet (mappenavn eller `githubRepo`); spør brukeren hvis det er uklart. Er et rundenavn/-id gitt som argument, bruk den; ellers velg den åpne runden med flest åpne kommentarer, og si hvilken du valgte.
2. Kall `mcp__flate__get_round` med `round_id`. Les hver kommentar med status `open` eller `planned`:
   - `anchor.route` = siden i appen, `anchor.testId` (mest stabil) / `anchor.component` / `anchor.text` / `anchor.cssPath` = elementet. `surface.device` sier hvilken flate kunden så (iphone/ipad/android/desktop).
   - Finn koden som rendrer elementet (søk etter `data-testid`/`testID`, tekst, rutefil).
3. Implementer én kommentar om gangen, med en commit per kommentar. Følg repoets konvensjoner. Kjør repoets typecheck/tester.
4. Etter hver kommentar: `mcp__flate__update_comment` med `status: "implemented"` og `note` på norsk (1–2 setninger, skrevet til kunden – hva ble endret, ikke hvordan). Hvis du må gjøre noe stort som ikke er avgrenset: `status: "planned"` + `note` om hva som trengs.
5. Er kommentaren tvetydig: kall `mcp__flate__reply` med et konkret spørsmål (gjerne to alternativer) og sett `status: "planned"`. Ikke gjett.
6. Sett ALDRI `verified` – det gjør kunden. Kjør ALDRI handlinger med eksterne effekter (sende SMS/e-post, betaling, deploy) som del av en kommentar.
7. Avslutt med en kort oppsummering: hva som ble gjort per kommentar, hva som må deployes for at kunden skal se det, og eventuelle spørsmål som venter på svar.
