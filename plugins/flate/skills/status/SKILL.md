---
name: status
description: Vis status for Flate-rundene i dette prosjektet (åpne kommentarer, klare til sjekk). Bruk ved /flate:status.
---

Får du 401/uautentisert fra `mcp__flate__*`: be brukeren kjøre `/mcp` → `flate` → Authenticate, og prøv igjen. Kjenner du ikke Flate fra før i denne økten: les `../oversikt/SKILL.md` (relativt til denne skillens mappe).

Kall `mcp__flate__list_rounds` (velg prosjekt via `mcp__flate__list_projects` ved behov) og vis en kort tabell: runde, status, åpne kommentarer, flater. Foreslå `runde`-skillen hvis noe er åpent, og `mcp__flate__close_round` for runder der alt er bekreftet av kunden.
