# Kit/Advisor

**What to wear on your ride — based on real weather.**

[kitadvisor.cc](https://kitadvisor.cc) · Free · No sign-up · English & Swedish

![Kit/Advisor](og-image.jpg)

Kit/Advisor recommends cycling clothing for any location in the world. Click the map, set how long, how fast and how hard you'll ride, and get a complete kit list — jersey, bibs or tights, gloves, headwear, shoes and rain gear.

## Features

- **Live weather** for any point on the map (Open-Meteo, up to 16 days ahead)
- **Feels-like temperature** that accounts for wind chill at riding speed, effort, ride length, humidity and how easily you get cold
- **Wind advice** — which direction to start so you get a tailwind home
- **Shareable ride links** with name, date and time — the forecast follows the ride
- **Kit card export** — a 1080×1920 image of your kit, ready for stories
- °C / °F and km/h / m/s toggles

## How it works

```
effective temp = windChill(air temp, wind + riding speed)
               + effort          (easy +4 · tempo +7 · hard +11 °C)
               + ride length     (>1.5 h −1 · >3 h −2 °C)
               + humidity        (>85 % −2 °C)
               + personal offset (runs cold −2 · runs warm +2 °C)
```

Wind chill uses the standard North American formula, blended smoothly from full effect at 10 °C to none at 33 °C so the result never jumps. The effective temperature then maps to a kit:

| Effective temp | Kit |
|---|---|
| ≥ 22 °C | Short-sleeve jersey, bib shorts |
| 17–22 °C | + arm warmers, light gloves |
| 12–17 °C | Long-sleeve jersey, bib shorts, thin gloves, ear warmer |
| 7–12 °C | Thermal jersey, bib tights, winter gloves, headband |
| 2–7 °C | Winter jacket, winter tights, heavy gloves, cap, shoe covers |
| < 2 °C | Double layer, extreme gloves, balaclava, winter shoes |

## Running locally

Everything lives in a single `index.html` — no build step, no dependencies.

```sh
python3 -m http.server 8000   # then open http://localhost:8000
```

(Copying the kit card to the clipboard requires HTTPS; downloading works everywhere.)

## Project layout

| Path | What |
|---|---|
| `index.html` | The whole app — HTML, CSS and JS |
| `DESIGN.md` | Design system reference (Kantarell v2) |
| `CLAUDE.md` | Developer notes: formula, state, quirks, roadmap |
| `llms.txt`, `robots.txt`, `sitemap.xml` | For search engines and AI crawlers |
| `favicon.*`, `icon-*.png`, `apple-touch-icon.png`, `site.webmanifest` | Icons |
| `og-image.jpg` | Link preview image |
| `.github/workflows/` | IndexNow ping on deploy, Lighthouse CI on pull requests |
| `CNAME`, `576294f8….txt` | Custom domain and IndexNow key — must stay in the root |

## Built with

[Leaflet](https://leafletjs.com) · [Open-Meteo](https://open-meteo.com) · [Nominatim](https://nominatim.openstreetmap.org) · Esri basemap tiles · [Phosphor Icons](https://phosphoricons.com) · Playfair Display · Manufacturing Consent

Hosted on GitHub Pages.

## License

[MIT](LICENSE) © Leo Genberg AB
