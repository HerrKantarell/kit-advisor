# Field/Trip — CLAUDE.md

> **Läs alltid index.html innan du gör något.**

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
- Manufacturing Consent (blackletter, CDN jsdelivr — används i logotypen)

## Hosting
GitHub Pages — HerrKantarell/kit-advisor
Allt lever i index.html. Inga byggsteg, inga beroenden att installera.

## Design
Kantarell v2 — se DESIGN.md för designreferens.
Kortversion:
- Bakgrund: #F7F7F5 (kall, inte varm)
- Accent: #347962 (grön, CSS-variabel --k-orange av historiska skäl)
- Typsnitt UI: -apple-system (system sans)
- Typsnitt rubriker: Playfair Display, kursiv
- Typsnitt logotyp: Manufacturing Consent, blackletter
- Kantlinjer: 0.5px solid #E2E2DF
- Manicula SVG används i callouts — aldrig som navigation

## Formellogik
Effektiv temp = windChill(lufttemp, omgivningsvind + cykelhastighet) + ansträngning + turlängdsstraff + fuktighetsstraff + thermoOffset

### windChill(T, V) — kontinuerlig modell (inga hopp)
- V < 4.8 km/h: returnerar T oförändrat
- T ≤ 10°C: standardformeln (13.12 + 0.6215T − 11.37V^0.16 + 0.3965TV^0.16)
- 10 < T < 33°C: blend = (33−T)/23, result = T + (wc−T) × blend
  → full vindkylaeffekt vid 10°C, noll vid 33°C (hudtemperatur)
- T ≥ 33°C: returnerar T

Den gamla modellen hade cutoff `T>=10 → return T`, vilket gav 3°C-hopp i upplevd temp.

### Övriga bidrag
- Ansträngning: lugn +4°C, tempo +7°C, hård +11°C
- Turlängd >3h: −2°C, >1.5h: −1°C
- Fuktighet >85%: −2°C
- Känsla (thermoType): fryser lätt −2°C, normal 0°C, varm lätt +2°C

### calcEff() returnerar
`{ eff, wc, speedEffect, effort, durP, humP, thermoOffset }`
- speedEffect = windChill(T, vind+fart) − windChill(T, vind) — hastighetens isolerade bidrag
- Visas i breakdown när |speedEffect| ≥ 0.5°C

## State-objekt
Allt state hålls i S = { lang, tempUnit, windUnit, intensity, thermoType, fcOffset, lat, lng, locName, wx, wxLive, hourly, isShared, shareName, shareDate }
- Hastighet visas alltid i km/h oavsett windUnit-inställning
- thermoType sparas i localStorage (ka_thermo), ingår INTE i share-URL

## i18n
Fullt stöd för SV och EN via T-objekt. Alla strängar ska läggas in i båda språken.

## Implementerat

### Fas 1 — Delningsbar turlink
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

### Fas 2 — Delningsbar kit-bild
- Canvas API, 1080×1920, pos/neg tema, eget kit
- FontFace-loading asynkront innan ctx.fillText (Playfair Display)

### Fas 3 — Affiliate-sektion + annonsplats
- AFFILIATE-objekt i CONFIG-blocket: winter/midseason/summer/rain per sv/en
- getAffiliateLinks(eff, isRain) → max 2 kontextuella länkar
- renderAffiliateLinks() → populerar #affArea med .aff-row
- affSec visas alltid i normalt läge; i delat läge bara om aff=1 i URL
- generateShareUrl lägger alltid till aff=1
- panel-left breddad 270→320px för annonsplats
- #adSlotSec i vänsterpanelen: 300×250 placeholder under delningsknapparna

## Roadmap (prioritetsordning)
1. ~~Delningsbar turlink~~ — klar
2. ~~Delningsbar kit-bild~~ — klar
3. ~~Affiliate-sektion~~ — klar (Velodrom.cc, kontextuell, aff=1 i URL)
4. Gratis MCP — Cloudflare Workers eller Railway

## Känt beteende / quirks
- Safari kräver explicit pixelhöjd på #map (inte flex-baserat) + invalidateSize() x2
- Mobilvy: karta är 150px hög strip, stadsnamn visas i separat banner
- Hastighetsslidern är alltid km/h — följer INTE windUnit-toggle
- Canvas FontFace-loading måste ske asynkront innan ctx.fillText (Playfair Display)
- navigator.clipboard.write kräver HTTPS — visa fallback (enbart nedladdning) på HTTP
- CSS-variabeln --k-orange heter så av historiska skäl men är numera grön (#347962)
- windChill-formeln är matematiskt giltig även för T>10 men blend:as ned mot 0 vid 33°C

## Filer
index.html — hela applikationen
CLAUDE.md — denna fil
DESIGN.md — designsystemreferens (Kantarell v2)
BUGS.md — kända buggar och quirks
README.md — projektöversikt
