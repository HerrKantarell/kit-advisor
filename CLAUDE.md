# Kit Advisor — CLAUDE.md

> **Läs alltid KANTARELL_DESIGN_SYSTEM_v2.md och index.html innan du gör något.**

## Vad projektet är
Väderbaserad cykelklädsel-rådgivare. Single-file HTML-app hostad på GitHub Pages.
Användaren klickar på en karta, får aktuellt väder via Open-Meteo och
rekommendationer baserade på effektiv temperatur (vindkyla + ansträngning).

## Stack
- Ren HTML/CSS/JS, ingen build-step
- Leaflet 1.9.4 (karta, CDN)
- Open-Meteo API (väder, gratis, ingen nyckel)
- Nominatim (reverse geocode, gratis)
- CartoDB Positron tiles
- Phosphor Icons 2.1.1 (CDN)
- Playfair Display (Google Fonts)

## Hosting
GitHub Pages — HerrKantarell/kit-advisor
Allt lever i index.html. Inga byggsteg, inga beroenden att installera.

## Design
Kantarell v2 — se KANTARELL_DESIGN_SYSTEM_v2.md för fullständig specifikation.
Kortversion:
- Bakgrund: #F7F7F5 (kall, inte varm)
- Accent: #D96B1F (orange, används sparsamt)
- Typsnitt UI: -apple-system (system sans)
- Typsnitt rubriker: Playfair Display, kursiv
- Kantlinjer: 0.5px solid #E2E2DF
- Manicula SVG används i callouts — aldrig som navigation

## Formellogik
Effektiv temp = Vindkyla(lufttemp, omgivningsvind + cykelhastighet) + ansträngning + turlängdsstraff + fuktighetsstraff
- Ansträngning: lugn +4°C, tempo +7°C, hård +11°C
- Turlängd >3h: -2°C, >1.5h: -1°C
- Fuktighet >85%: -2°C
- Vindkyla-formel: giltig under 10°C och vind >4.8 km/h

## State-objekt
Allt state hålls i S = { lang, tempUnit, windUnit, intensity, fcOffset, lat, lng, locName, wx, wxLive, hourly, isShared, shareName, shareDate }
Hastighet visas alltid i km/h oavsett vindenhets-inställning.

## i18n
Fullt stöd för SV och EN via T-objekt. Alla strängar ska läggas in i båda språken.

## Implementerat

### Fas 1 — Delningsbar turlink (commit 1f75636, 58dce91)
URL-parametrar: ?name=&date=&lat=&lng=&dur=&speed=&int=&loc=
- "Dela tur"-knapp längst ned i vänsterpanelen (visas efter platsvalet)
- Modal med turnamn, datum, tid — längd/hastighet/intensitet följer med automatiskt
- Genererar länk + kopieringsknapp
- Delat läge: UI låst (sliders/intensity disabled), karta ej klickbar
- Header visar turnamn + formaterat datum istället för koordinater
- Tidshorisontsmeddelanden: > 10 dagar, 3–10 dagar, passerat
- Open-Meteo hämtas med forecast_days=16 i delat läge
- CONFIG.baseUrl = null — ändras till 'https://kitadvisor.cc' när domän finns
- "Skapa din egen →"-länk i delat läge

## Roadmap (prioritetsordning)
1. ~~Delningsbar turlink~~ — klar
2. Delningsbar kit-bild — Canvas API, transparent PNG, Instagram-ratio (1080x1920)
3. Affiliate-sektion — Velodrom.cc via Awin, kontextuell, tydlig etikett
4. Gratis MCP — Cloudflare Workers eller Railway

## Känt beteende / quirks
- Safari kräver explicit pixelhöjd på #map (inte flex-baserat) + invalidateSize() x2
- Mobilvy: karta är 150px hög strip, stadsnamn visas i separat banner
- Hastighetsslidern är alltid km/h — följer INTE windUnit-toggle
- Canvas FontFace-loading måste ske asynkront innan ctx.fillText (Playfair Display)
- navigator.clipboard.write kräver HTTPS — visa fallback (enbart nedladdning) på HTTP

## Filer
index.html — hela applikationen
CLAUDE.md — denna fil
KANTARELL_DESIGN_SYSTEM_v2.md — Kantarell v2 designsystem (fullständig spec)
DESIGN.md — äldre kopia av designsystemet (ersatt av ovanstående)
