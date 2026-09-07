## Flate-gjennomgang

Dette repoet er koblet til Flate via MCP-serveren `flate`. Verktøyene er tilgjengelige som `mcp__flate__<navn>`: `list_projects`, `create_project`, `list_rounds`, `create_round`, `create_share_link`, `get_round`, `get_comment`, `update_comment`, `reply`, `close_round`, `export_round`.

Flate er kundegjennomgang: kunden åpner en delingslenke, havner i den ekte appen med Flate sitt overlay oppå, peker og kommenterer. Kommentarene har anker (`anchor.route` = siden, `anchor.testId`/`component`/`text`/`cssPath` = elementet), skjermbilde og status. Statuser: `open` → `planned` → `implemented` → `verified` (kun kunden) eller `rejected`. Får du 401 fra verktøyene: `codex mcp login flate`.

### Lage en runde

1. `mcp__flate__list_projects` → velg prosjektet som matcher repoet (`githubRepo` mot `git remote get-url origin`). Mangler det: `mcp__flate__create_project` med mappenavn og `github_repo` (`eier/repo`).
2. `mcp__flate__create_round` med navn og enten `surfaces` (`{ name, url, device: iphone|ipad|android|desktop }`) eller `branch` (finner deploy-previewen via prosjektets GitHub-kobling). Appen på URL-en må ha `@flateapp/sdk` (`^0.3.0`) montert og portalen i `connect-src` hvis den har CSP.
3. `mcp__flate__create_share_link` med `round_id`, og gi brukeren lenken: «Åpne lenken, skriv navnet ditt, trykk k eller Kommentér, pek på det du vil endre.»

### Ta runden

1. Kall `mcp__flate__list_rounds`. Får du feil om `project_id`: kall `mcp__flate__list_projects` og velg prosjektet som matcher dette repoet; spør brukeren hvis det er uklart. Er et rundenavn/-id gitt, bruk den; ellers velg den åpne runden med flest åpne kommentarer, og si hvilken du valgte.
2. Kall `mcp__flate__get_round` med `round_id`. Les hver kommentar med status `open` eller `planned`, og finn koden som rendrer elementet (søk etter `data-testid`/`testID`, tekst, rutefil). `surface.device` sier hvilken flate kunden så.
3. Implementer én kommentar om gangen, med en commit per kommentar. Følg repoets konvensjoner. Kjør repoets typecheck/tester.
4. Etter hver kommentar: `mcp__flate__update_comment` med `status: "implemented"` og `note` på norsk (1–2 setninger, skrevet til kunden – hva ble endret, ikke hvordan). Må du gjøre noe stort som ikke er avgrenset: `status: "planned"` + `note` om hva som trengs.
5. Er kommentaren tvetydig: kall `mcp__flate__reply` med et konkret spørsmål (gjerne to alternativer) og sett `status: "planned"`. Ikke gjett.
6. Sett ALDRI `verified` – det gjør kunden. Kjør ALDRI handlinger med eksterne effekter (sende SMS/e-post, betaling, deploy) som del av en kommentar.
7. Avslutt med en kort oppsummering: hva som ble gjort per kommentar, hva som må deployes for at kunden skal se det, og eventuelle spørsmål som venter på svar. Er alt bekreftet av kunden: `mcp__flate__close_round`.
