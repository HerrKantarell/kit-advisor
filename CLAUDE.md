# Kit/Advisor — CLAUDE.md

> **Läs alltid index.html innan du gör något.**

## Vad projektet är
Väderbaserad cykelklädsel-rådgivare. Single-file HTML-app hostad på GitHub Pages (kitadvisor.cc).
Användaren klickar på en karta, får aktuellt väder via Open-Meteo och
rekommendationer baserade på effektiv temperatur (vindkyla + ansträngning).

## Stack
- Ren HTML/CSS/JS, ingen build-step
- Leaflet 1.9.4 (karta, CDN)
- Open-Meteo API (väder, gratis, ingen nyckel)
- Nominatim (reverse geocode, gratis)
- Esri World Light Gray Base tiles (server.arcgisonline.com, attribution "Tiles © Esri")
- Phosphor Icons 2.1.1 (CDN, endast regular-vikten: src/regular/style.css, laddas icke-blockerande)
- Playfair Display (Google Fonts)
- Manufacturing Consent (blackletter, CDN jsdelivr — används i logotypen)
- Cloudflare Web Analytics (cookiefri besöksstatistik, beacon-script före </body>, dashboard på dash.cloudflare.com — inga custom events)

## Hosting
GitHub Pages — HerrKantarell/kit-advisor
Allt lever i index.html. Inga byggsteg, inga beroenden att installera.
Custom domän: kitadvisor.cc — registrerad och DNS-hanterad hos Porkbun (porkbun.com), ägare Leo Genbergs AB
DNS (i Porkbun): ALIAS + CNAME → herrKantarell.github.io. DNS går INTE via Cloudflare. HTTPS via Let's Encrypt (GitHub hanterar automatiskt).

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
- lang defaultar till 'en' (engelska)
- Hastighet visas alltid i km/h oavsett windUnit-inställning
- thermoType sparas i localStorage (ka_thermo), ingår INTE i share-URL
- wx-objektet innehåller nu även windDir (grader, meteorologisk konvention — varifrån vinden blåser)

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
- #adSlotSec i vänsterpanelen: 300×250 placeholder — för tillfället dold (display:none)

### Fas 4 — Vindriktningsuppmaning
- API-anropet hämtar wind_direction_10m (current + hourly)
- #windHintSec i höger panel (mellan analys och utrustning), dold om vind < 15 km/h
- Kollapserad som default, expanderas med toggleWindHint()
- ph-arrow-up roteras till windDir-grader (pekar mot varifrån vinden kommer)
- Strängar: T[lang].windHintLabel(dir), T[lang].windHintTip(dir), T[lang].windDirs[]

### Fas 5 — Namn, domän, SEO
- Namn: Kit/Advisor (var Field/Trip)
- Domän: kitadvisor.cc — Porkbun, Leo Genbergs AB
- CONFIG.baseUrl = 'https://kitadvisor.cc'
- SEO: meta description, OG-taggar, twitter:card, canonical, JSON-LD (WebApplication)
- og:image: og-image.jpg (1200×630, JPG ~160 KB). Källbilden och den gamla og-image.svg är borttagna (SVG stöds inte av Facebook/LinkedIn/X)
- llms.txt i repo-roten (för AI-crawlers: Perplexity, ChatGPT m.fl.)
- Dold SEO-text: <section class="sr-only"> direkt efter <body> med <h1>, "How it works", kit-trösklar och FAQ. Syns inte men läses av crawlers (även de som inte kör JS) och skärmläsare. Håll texten i synk med formeln/trösklarna i koden — FAQ-exemplen är räknade med 28 km/h, 10 km/h vind, lugn, 2 h
- FAQPage JSON-LD i <head> speglar samma frågor/svar som den dolda sektionen
- robots.txt (släpper in alla, inkl. GPTBot/ClaudeBot/PerplexityBot m.fl.) + sitemap.xml
- Favicon: blackletter-"K" (Manufacturing Consent, utritad som path) i #F7F7F5 på grön rundad ruta — favicon.svg, favicon.ico (16/32/48), apple-touch-icon.png (180), icon-192/512.png, site.webmanifest
- <html lang="en"> (default engelska)

### Fas 6 — Besöksstatistik
- Cloudflare Web Analytics (gratis, cookiefri → ingen cookie-banner behövs)
- JS-snippet (beacon.min.js, token i data-cf-beacon) ligger precis före </body> i index.html, enligt Cloudflares instruktion
- Kräver inte Cloudflare-DNS — domänen ligger kvar hos Porkbun
- Statistik: dash.cloudflare.com → Analytics & Logs → Web Analytics → kitadvisor.cc
- Begränsning: inga custom events (affiliate-klick, "Dela tur" m.m. mäts inte).
  Om det behövs: redirect-räknare för affiliate-länkar eller komplettera med GoatCounter/Plausible

### Fas 7 — SEO-automation
- `.github/workflows/indexnow.yml`: vid push till main som ändrar index.html/llms.txt → sätter sitemap.xml <lastmod> till dagens datum (bot-commit), väntar 2 min på Pages-deploy, pingar IndexNow (Bing/Yandex m.fl., inte Google). Nyckelfil: `576294f82ad8ed52e0685dad366b5ccf.txt` i roten — ta inte bort
- `.github/workflows/lighthouse.yml` + `lighthouserc.json`: Lighthouse CI på varje PR. SEO < 0.9 = fel, övrigt varningar
- Search Console-rapport: privat repo HerrKantarell/kit-advisor-seo (GitHub Action varje måndag, secret GSC_CREDENTIALS = tjänstkonto-JSON). Sökdata ska INTE läggas i detta publika repo
- Schemalagd Claude-körning läser rapporten och öppnar PR med innehållsförslag — mergas aldrig automatiskt

## Roadmap (prioritetsordning)
1. ~~Delningsbar turlink~~ — klar
2. ~~Delningsbar kit-bild~~ — klar
3. ~~Affiliate-sektion~~ — klar (Velodrom.cc, kontextuell, aff=1 i URL)
4. **Namn + domän** — bör lösas innan affiliates formaliseras
   - ~~Appen heter "Field/Trip" i UI men repot är kit-advisor~~
   - ~~Bestäm ett namn, köp domän, uppdatera CONFIG.baseUrl~~
   - Namn: Kit/Advisor, domän: kitadvisor.cc — klart
5. **Affiliate-expansion** — efter namn/domän är klart
   - Velodrom.cc: maila info@velodrom.cc, formalisera befintlig ref-länk
   - Rapha (4%, 30 dagar): skapa publisher-konto på AWIN → ansök till Rapha
   - Sigma Sports (2,8%, 90 dagar): via Avelon (lämnade AWIN) — bär PNS, MAAP, CdC
6. Gratis MCP — Cloudflare Workers eller Railway

## Känt beteende / quirks
- Safari kräver explicit pixelhöjd på #map (inte flex-baserat) + invalidateSize() x2
- Mobilvy: karta är 150px hög strip, stadsnamn visas i separat banner
- Hastighetsslidern är alltid km/h — följer INTE windUnit-toggle
- Canvas FontFace-loading måste ske asynkront innan ctx.fillText (Playfair Display)
- navigator.clipboard.write kräver HTTPS — visa fallback (enbart nedladdning) på HTTP
- CSS-variabeln --k-orange heter så av historiska skäl men är numera grön (#347962)
- windChill-formeln är matematiskt giltig även för T>10 men blend:as ned mot 0 vid 33°C
- LCP-elementet är en kartbild (Esri). Därför: preconnect till server.arcgisonline.com, Leaflet fadeAnimation:false, och Phosphor laddas som icke-blockerande CSS (media="print" onload). Ladda INTE Phosphors index.js i <head> — den blockerar rendering och drar in alla sex vikter
- Annonsblockerare (uBlock m.fl.) blockerar Cloudflare-beacon — egna besök syns ofta inte i statistiken

## Filer
index.html — hela applikationen
CLAUDE.md — denna fil
DESIGN.md — designsystemreferens (Kantarell v2)
README.md — projektöversikt
og-image.jpg — Open Graph-bild (1200×630) för länkförhandsvisningar
llms.txt — maskinläsbar projektbeskrivning för AI-crawlers
robots.txt, sitemap.xml — för sökmotorer
favicon.svg, favicon.ico, apple-touch-icon.png, icon-192.png, icon-512.png, site.webmanifest — ikoner
CNAME — custom domän för GitHub Pages
576294f82ad8ed52e0685dad366b5ccf.txt — IndexNow-nyckel (måste ligga i roten)
lighthouserc.json — Lighthouse CI-gränser
.github/workflows/ — indexnow.yml (IndexNow + sitemap lastmod), lighthouse.yml (Lighthouse CI)
.gitignore — OS-/editorfiler
