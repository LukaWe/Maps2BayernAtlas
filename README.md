# Maps2BayernAtlas
🗺️ Konvertiere Google Maps und OpenStreetMap (WGS84) URLs zu [BayernAtlas](https://atlas.bayern.de) (UTM Zone 32N) Links mit präziser Koordinatentransformation.

![PHP](https://img.shields.io/badge/PHP-8.0+-blue)
![Lizenz](https://img.shields.io/badge/Lizenz-MIT-green)
![Keine Abhängigkeiten](https://img.shields.io/badge/Abhängigkeiten-keine-brightgreen)

![Demo](screenshots/demo.webp)

## Technische Details
### Warum ist eine Koordinatentransformation notwendig?
Google Maps und OpenStreetMap verwenden das **WGS84**-Koordinatensystem (EPSG:4326) mit Längen- und Breitengraden (z.B. `48.137°N, 11.575°E`). Dies ist das weltweit gebräuchlichste Format für GPS und Webkarten.

Der **BayernAtlas** hingegen verwendet das **UTM Zone 32N** Koordinatensystem basierend auf dem europäischen Referenzsystem **ETRS89** (EPSG:25832). Dieses System verwendet metrische Rechts- und Hochwerte (z.B. `691567, 5334734`) und ist optimiert für präzise Vermessungsarbeiten in Mitteleuropa.

**Beispiel:**
| System | Darstellung |
|--------|-------------|
| WGS84 (Google Maps) | `48.137000°N, 11.575000°E` |
| UTM Zone 32N (BayernAtlas) | `Rechtswert: 691567, Hochwert: 5334734` |

Diese Anwendung führt die mathematische Transformation zwischen beiden Systemen durch, sodass du einfach einen Google Maps oder OSM Link eingeben kannst und direkt den korrekten BayernAtlas-Link erhältst.

### Bayern-Grenze
Die Anwendung verwendet eine Polygon-Approximation der bayerischen Grenzen mit ~5km Pufferzone. Dies gewährleistet vollständige Abdeckung einschließlich grenznaher Gebiete, für die BayernAtlas noch Daten bereitstellt.

## Funktionen
- ✅ **Multi-Quellen Support** - Google Maps & OpenStreetMap URLs
- ✅ **Koordinatentransformation** - WGS84 → UTM Zone 32N (ETRS89)
- ✅ **Bayern-Grenzprüfung** - Validiert, dass Koordinaten innerhalb Bayerns liegen
- ✅ **REST API** - Programmatischer Zugriff für Automatisierung
- ✅ **Mehrsprachig** - Deutsche & englische Benutzeroberfläche
- ✅ **Keine Abhängigkeiten** - Reines PHP, keine externen Bibliotheken
- ✅ **Rate Limiting** - Eingebauter Schutz vor Missbrauch

## Unterstützte URL-Formate
| Quelle | Format | Beispiel |
|--------|--------|----------|
| Google Maps | Gekürzt | `https://maps.app.goo.gl/xyz123` |
| Google Maps | Ort/Marker | `https://www.google.de/maps/place/.../@lat,lon/data=!3d...!4d...` |
| Google Maps | Koordinaten | `https://www.google.com/maps/@48.137,11.575,15z` |
| OpenStreetMap | Hash-Format | `https://www.openstreetmap.org/#map=19/49.021931/12.081882` |
| OpenStreetMap | Marker | `https://www.openstreetmap.org/?mlat=49.02&mlon=12.08` |

## Einschränkungen
- ⚠️ **Nur Bayern** - Koordinaten außerhalb Bayerns werden abgelehnt
- ⚠️ **Rate Limit** - 10 Anfragen/Minute pro IP (konfigurierbar)
- ⚠️ **Origin-Whitelist** - API ist auf konfigurierte Domains beschränkt

## Installation
Repository klonen (oder als ZIP herunterladen und entpacken).
Für Produktiveinsatz auf einem PHP-fähigen Webserver bereitstellen (Apache, Nginx mit PHP-FPM, etc.)

**Voraussetzungen:** PHP 8.0+ with cURL extension

## Konfiguration

Bearbeite `includes/config.php`:

```php
// Rate Limiting (Anfragen pro Minute)
const API_RATE_LIMIT = 10;

// Erlaubte Origins für API-Zugriff (leer = alle erlauben)
const API_ALLOWED_ORIGINS = [
    'https://deinedomain.de',
    'https://www.deinedomain.de',
];

// IPs, die Rate Limiting umgehen
const IP_WHITELIST = [
    '127.0.0.1',
    '::1',
];
```

## API Dokumentation

### Endpunkt

```
POST /api/convert
Content-Type: application/json
```

### Anfrage

```json
{
  "gmaps_url": "https://www.openstreetmap.org/#map=19/49.021931/12.081882"
}
```

### Antwort (Erfolg)

```json
{
  "success": true,
  "bayernatlas_url": "https://atlas.bayern.de/?c=725303,5434470&z=16&r=0&l=atkis&crh=true&mid=1",
  "coordinates": {
    "lat": 49.021931,
    "lon": 12.081882,
    "easting": 725303,
    "northing": 5434470
  }
}
```

### Antwort (Fehler)

```json
{
  "success": false,
  "message": "Location is outside Bavaria"
}
```

### HTTP Status Codes

| Code | Beschreibung |
|------|--------------|
| 200 | Erfolgreiche Konvertierung |
| 400 | Ungültige Anfrage (fehlende/ungültige URL) |
| 403 | Origin nicht erlaubt |
| 422 | Standort außerhalb Bayerns |
| 429 | Rate Limit überschritten |

### cURL Beispiel

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"gmaps_url":"https://maps.app.goo.gl/xyz123"}' \
  https://deinserver.de/api/convert
```

### JavaScript Beispiel

```javascript
const response = await fetch('/api/convert', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ 
    gmaps_url: 'https://www.openstreetmap.org/#map=19/49.02/12.08' 
  })
});
const data = await response.json();
console.log(data.bayernatlas_url);
```

## Projektstruktur

```
maps-to-bayernatlas/
├── assets/
│   ├── css/style.css      # Styles
│   └── js/app.js          # Frontend-Logik
├── includes/
│   ├── config.php         # Konfiguration & Bayern-Polygon
│   ├── classes.php        # PHP-Kernklassen
│   └── api.php            # API-Handler
├── lang/
│   ├── de.json            # Deutsche Übersetzungen
│   └── en.json            # Englische Übersetzungen
└── index.php              # Einstiegspunkt
```

## Lizenz

MIT-Lizenz - siehe [LICENSE](LICENSE) für Details.

---

# English Version

## Maps2BayernAtlas (English)

🗺️ Convert Google Maps and OpenStreetMap URLs to [BayernAtlas](https://atlas.bayern.de) links with precise coordinate transformation.

### Features

- ✅ **Multi-Source Support** - Google Maps & OpenStreetMap URLs
- ✅ **Coordinate Transformation** - WGS84 → UTM Zone 32N (ETRS89)
- ✅ **Bavaria Boundary Check** - Validates coordinates are within Bavaria
- ✅ **REST API** - Programmatic access for automation
- ✅ **Multi-Language** - German & English interface
- ✅ **No Dependencies** - Pure PHP, no external libraries required

### Supported URL Formats

| Source | Format | Example |
|--------|--------|---------|
| Google Maps | Shortened | `https://maps.app.goo.gl/xyz123` |
| Google Maps | Place/Marker | `https://www.google.de/maps/place/.../@lat,lon/data=!3d...!4d...` |
| Google Maps | Coordinates | `https://www.google.com/maps/@48.137,11.575,15z` |
| OpenStreetMap | Hash format | `https://www.openstreetmap.org/#map=19/49.021931/12.081882` |

### Limitations

- ⚠️ **Bavaria Only** - Coordinates outside Bavaria are rejected
- ⚠️ **Rate Limited** - 10 requests/minute per IP (configurable)
- ⚠️ **Origin Whitelist** - API restricted to configured domains in production

### Installation

```bash
git clone https://github.com/yourusername/Maps2BayernAtlas.git
cd Maps2BayernAtlas
php -S localhost:8080
```

**Requirements:** PHP 8.0+ with cURL extension

### API Usage

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"gmaps_url":"https://maps.app.goo.gl/xyz123"}' \
  https://yourserver.com/api/convert
```

**Response:**
```json
{
  "success": true,
  "bayernatlas_url": "https://atlas.bayern.de/?c=725303,5434470&z=16...",
  "coordinates": { "lat": 49.02, "lon": 12.08, "easting": 725303, "northing": 5434470 }
}
```

### Status Codes

| Code | Description |
|------|-------------|
| 200 | Successful conversion |
| 400 | Bad request (missing/invalid URL) |
| 403 | Origin not allowed |
| 422 | Location outside Bavaria |
| 429 | Rate limit exceeded |

### Why Coordinate Transformation?

Google Maps and OpenStreetMap use **WGS84** coordinates (EPSG:4326) - latitude and longitude (e.g., `48.137°N, 11.575°E`). This is the global standard for GPS and web maps.

**BayernAtlas** uses **UTM Zone 32N** based on the European reference system **ETRS89** (EPSG:25832). This system uses metric easting/northing values (e.g., `691567, 5334734`) optimized for precise surveying in Central Europe.

This application performs the mathematical transformation between both systems automatically.

---

Made with ❤️ in Bavaria from a Bavarian 🏔️

If you find this useful and want to support:
**Bitcoin (BTC):** `bc1q3lz8vxpk0rchqn6dq8g08rkcqts425csuvnjr2477uzdenak5n8sfds2ke`
