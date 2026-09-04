---
name: issues
description: Gjør åpne Flate-kommentarer om til issues/oppgaver i din egen issue-tracker (Linear, GitHub Issues, Azure DevOps eller annet med MCP), via dine egne MCP-koblinger. Bruk når brukeren kjører /flate:issues eller sier «lag oppgaver av kommentarene».
disable-model-invocation: true
argument-hint: "[rundenavn eller runde-id] [tracker]"
---

# Flate: kommentarer → issues i din tracker

Poenget: Flate holder ikke integrasjoner mot Linear/GitHub/Azure selv. Du har allerede
issue-trackeren din koblet som MCP-server i Claude Code — denne skillen lar AGENTEN opprette
issues der via DINE koblinger. Fungerer for enhver tracker med en MCP-server.

## Før du starter
1. Finn trackeren: er «tracker» oppgitt som argument, bruk den. Ellers se hvilke issue-tracker-MCP-servere som er tilgjengelige (verktøy som `mcp__linear__*`, `mcp__github__*`/`create_issue`, `mcp__azure*__*`/work items). Finnes flere, spør kort hvilken. Finnes ingen: si at brukeren må koble en tracker som MCP-server først (`/mcp`), og stopp.
2. Bekreft hvilket team/prosjekt/repo issues skal havne i (Linear: team; GitHub: eier/repo; Azure: organisasjon/prosjekt). Spør én gang hvis uklart; husk svaret for resten av kjøringen.

## Slik gjør du det
1. `mcp__flate__list_rounds` (får du feil om `project_id`: `mcp__flate__list_projects` og velg prosjektet som matcher repoet). Bruk rundenavn/-id fra argument, ellers den åpne runden med flest åpne kommentarer; si hvilken du valgte.
2. `mcp__flate__get_round` med `round_id`. For hver kommentar med status `open` eller `planned` som IKKE allerede har et issue (se punkt 5 – sjekk svartråden for en «issue: <url>»-lenke fra en tidligere kjøring, hopp over de som har det):
   - Bygg issue-innholdet: **tittel** = en kort oppsummering av kommentaren; **beskrivelse** = kommentarteksten + «Flate-kommentar #<nummer> av <authorName>» + ankeret (`anchor.route` = siden, `anchor.testId`/`anchor.component`/`anchor.text`/`anchor.cssPath` = elementet, `surface.device` = enhet) + fokusteksten (`round.brief`) hvis satt + skjermbilde-lenke (`screenshotUrl`, allerede absolutt) hvis satt + en lenke til Flate-runden.
3. Opprett issuet via trackerens MCP-verktøy (f.eks. Linear `create_issue`, GitHub `create_issue`, Azure work-item-opprettelse) i det avklarte team/prosjekt/repo. Fang issue-URL-en fra svaret.
4. Feiler oppretting for én kommentar: si det, hopp videre — ikke stopp hele kjøringen.
5. **Koble tilbake:** for hver opprettet issue, `mcp__flate__reply` på kommentaren med teksten `issue: <url>`, og sett `mcp__flate__update_comment` status `planned` (arbeidet er planlagt, ikke gjort). Dette gjør kjøringen idempotent – neste `/flate:issues` hopper over kommentarer som alt har en «issue: …»-lenke.
6. Avslutt med en kort oppsummering: hvor mange issues opprettet, i hvilken tracker/prosjekt, med lenkene – og hvilke kommentarer som ble hoppet over (alt hadde issue) eller feilet.

## Merknad
Denne skillen oppretter issues når DU kjører den (ikke i det sekundet kunden kommenterer).
Vil du at agenten skal implementere kommentarene med én gang i stedet for å lage issues, bruk
`/flate:runde`. De to kan kombineres: lag issues nå, løs dem senere.
