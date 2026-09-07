---
name: oversikt
description: Slik henger Flate sammen — portal, SDK, MCP-verktøy, roller, statuser og livssyklusen til en runde. Les når brukeren spør hvordan Flate fungerer eller brukes, og før du bruker de andre Flate-skillene første gang i en økt.
---

# Flate: slik henger det sammen

Flate er kundegjennomgang for web-apper: kunden peker og kommenterer i den ekte appen, og du får
kommentarene som verktøy i terminalen, fikser dem og melder tilbake. Ingen konto for kunden.

## Delene

| Del | Hva | Hvor |
|---|---|---|
| Portalen | prosjekter, runder, flater, delingslenker, GitHub-kobling, abonnement | `https://portal-production-c530.up.railway.app` (`/app`) |
| SDK-en | `@flateapp/sdk` (`^0.3.0`), montert i kundens app; tegner gjennomgangs-UI-et som overlay | npm |
| MCP-serveren | `flate` — verktøyene `mcp__flate__<navn>` i terminalen | `https://mcp-production-e390.up.railway.app/mcp` |
| Pluginen | skillene installer, ny-runde, del, runde, issues, status, oversikt | denne |

## Hva kunden opplever

1. Åpner delingslenken (`<portal>/r/<slug>`), skriver navnet sitt.
2. Sendes til appen på toppnivå med et 12-timers token i URL-fragmentet (`#flate=…`). SDK-en leser
   tokenet, fjerner det fra adressen og lagrer det i `sessionStorage` for fanen.
3. Ser appen sin som vanlig, med Flate oppå: en flytende kommandobar (kan dras i gripefeltet; «k»
   slår kommentarmodus av og på), pinner på tidligere kommentarer, og et kommentarpanel til høyre.
4. Slår på Kommentér, peker på et element, skriver. Kommentaren får nummer, anker og skjermbilde.
5. Innlogging virker som i appen ellers, også Vipps, BankID og Google, fordi det ikke er noen iframe.
6. Ser svar og statusendringer i sanntid, og bekrefter selv («klar til sjekk» → verified).

Utløpt token gir «Lenken er utløpt» — kunden åpner delingslenken på nytt.

## Hva du gjør, i rekkefølge

1. **Koble appen til** (én gang per app): `installer`-skillen. Deploy etterpå.
2. **Lag en runde**: `ny-runde` (deployet/preview-URL eller branch) eller `del` (lokal app via tunnel).
3. **Send delingslenken** til kunden.
4. **Ta runden**: `runde`-skillen henter kommentarene, fikser én om gangen, setter status med notat.
5. **Kunden bekrefter**. Når alt er verified: `mcp__flate__close_round`.

Alt dette kan du gjøre direkte når brukeren ber om det i prosa («lag en runde», «ta runden»,
«lukk runden») — slash-kommandoene er snarveier, ikke krav.

## Datamodell

- **Prosjekt**: navn, `githubRepo` (`eier/repo`, brukes for å matche repoet du står i), GitHub-kobling.
- **Runde**: navn, `status` `open` | `closed`, valgfri `brief` (hva kunden skal fokusere på).
- **Flate**: `name`, `url`, `device` `iphone` | `ipad` | `android` | `desktop`. Én flate per side/enhet.
- **Kommentar**: `number`, `authorName`, `body`, `status`, `priority`, `screenshotUrl` (absolutt),
  `anchor` med `route` (siden), `testId` (mest stabil), `component`, `text`, `cssPath`, `rect`, og
  `replies` med `authorType` `client` | `developer` | `agent`.

## Statuser og hvem som setter dem

`open` → `planned` → `implemented` → `verified`, eller `rejected`.
Agenten kan sette `planned`, `implemented` og `rejected` fra `open`/`planned`/`implemented`, alltid
med et kort `note` på norsk skrevet til kunden. **Kun kunden setter `verified`.** En lukket runde
avviser nye kommentarer og svar (HTTP 409 `closed`).

## MCP-verktøyene

| Verktøy | Bruk |
|---|---|
| `list_projects` | prosjekter med `githubRepo` og antall åpne runder |
| `create_project` (`name`, `github_repo?`) | send alltid `github_repo` (`eier/repo` fra `git remote get-url origin`) |
| `list_rounds` (`project_id?`) | runder med åpne kommentarer |
| `create_round` (`name`, `surfaces` eller `branch`+`device?`) | `branch` finner deploy-previewen via GitHub-koblingen |
| `create_share_link` (`round_id`, `password?`, `expires_days?`) | lenken kunden får |
| `get_round` / `get_comment` | alt innhold, gruppert per flate |
| `update_comment` (`comment_id`, `status`, `note`) | status + notat til kunden |
| `reply` (`comment_id`, `body`) | spørsmål i tråden |
| `close_round` (`round_id`) | når alt er bekreftet |
| `export_round` (`round_id`) | hele runden som markdown |
| `create_agent_token` (`name`, `project_id?`) | prosjekt-token `fl_…` for CI/Codex (krever OAuth-innlogging) |

Får du **401/uautentisert** fra `mcp__flate__*`: be brukeren kjøre `/mcp`, velge `flate` og
Authenticate (innlogging i nettleseren). Alternativ uten OAuth: prosjekt-token som Bearer.
Får du **plangrense** (Free har én aktiv runde): meldingen inneholder lenken til abonnement; lukk en
runde eller oppgrader.

## Å få appen opp på web

Tre veier, velg den som passer:

1. **Egen deploy eller preview-URL** (Vercel, Netlify, Railway, egen server): `create_round` med
   `surfaces`. Appen på den URL-en må ha SDK-en montert og portalen i `connect-src`.
2. **Branch via GitHub-koblingen**: er prosjektet koblet til GitHub i portalen (Prosjektinnstillinger
   → Koble til GitHub), finner `create_round` med `branch` deploy-previewen som Vercel/Netlify/Railway
   poster som GitHub Deployment. Ingen URL å holde styr på. Mangler preview: deploy branchen først.
   Portalen kan i tillegg bygge enkle apper uten backend på Railway (opt-in i portal-UI-et).
3. **Lokalt uten deploy**: `del`-skillen tunnelerer `localhost` med cloudflared og lager runden.
   Lever så lenge terminalen står åpen.

## Er appen klar? (sjekk før du deler)

```
curl -sI https://<app> | grep -i content-security-policy | grep -o 'connect-src[^;]*'   # portal-origin skal være med hvis appen har CSP
curl -s https://<app> | grep -o '/_next/static/chunks/[^"]*\.js' | while read c; do curl -s "https://<app>$c" | grep -q 'flate:review-token' && echo "SDK i $c"; done
```

Ingen treff på SDK-markøren betyr at bridgen ikke er i bundelen kunden får — se `installer`.
