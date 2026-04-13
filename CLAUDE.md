# Kit Advisor — CLAUDE.md

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
Kantarell v2 — se DESIGN.md för fullständig specifikation.
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
Allt state hålls i S = { lang, tempUnit, windUnit, intensity, fcOffset, lat, lng, locName, wx, wxLive, hourly }
Hastighet visas alltid i km/h oavsett vindenhets-inställning.

## i18n
Fullt stöd för SV och EN via T-objekt. Alla strängar ska läggas in i båda språken.

## Roadmap (prioritetsordning)
1. Delningsbar turlink — URL-parametrar, inget backend
2. Delningsbar kit-bild — Canvas API, transparent PNG, Instagram-ratio (1080x1920)
3. Affiliate-sektion — Velodrom.cc via Awin, kontextuell, tydlig etikett
4. Gratis MCP — Cloudflare Workers eller Railway

## Känt beteende / quirks
- Safari kräver explicit pixelhöjd på #map (inte flex-baserat) + invalidateSize() x2
- Mobilvy: karta är 150px hög strip, stadsnamn visas i separat banner
- Hastighetsslidern är alltid km/h — följer INTE windUnit-toggle

## Filer
index.html — hela applikationen
CLAUDE.md — denna fil
DESIGN.md — Kantarell v2 designsystem (fullständig spec)
