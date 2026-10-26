# Walkthrough 04 — Crear cuenta Apify (si el agente va a "scrapear")

> Apify es la herramienta que usa el agente cuando necesita información de sitios web que NO tienen API oficial — típicamente Twitter, Instagram, sitios con catálogo, etc.

---

## ¿Cuándo necesito Apify?

Solo si tu agente va a buscar información en:

- ✅ Twitter / X (cuentas que sigues, búsquedas, trending)
- ✅ Instagram
- ✅ LinkedIn
- ✅ Sitios web sin API (catálogos, foros, comparadores)
- ✅ Google Maps (negocios cerca, reviews)

**NO necesitas Apify si**:

- Tu agente solo lee RSS o noticias (usa RSS directo)
- Tu agente solo procesa contenido que TÚ le das
- Solo va a leer APIs oficiales (Notion, GitHub, etc.)

Si tu agente no necesita scraping, **skip este walkthrough** completamente.

---

## ¿Cuánto cuesta?

```
Plan free de Apify: $5 USD/mes gratis para empezar.
Después: $0.10-0.40 USD por cada 1000 resultados scrapeados.

Para un agente personal que corre 1 vez al día:
  ~30 tweets × 30 días = 900 tweets/mes = $0.09 USD/mes 🙃

O sea, gratis casi siempre.
```

---

## Paso 1 — Crear cuenta Apify (3 min)

```
1. Abre tu navegador
2. Ve a https://console.apify.com/sign-up
3. Tap "Sign up with Google" o usa email
4. Confirma email si te lo piden
5. Te lleva al dashboard

¿Ya entraste? Dime cuando estés.
```

---

## Paso 2 — Obtener tu API token

```
1. Una vez dentro del dashboard, tap tu avatar arriba a la derecha
2. Tap "Settings" o "Account settings"
3. En el menú izquierdo busca "Integrations" o "API & Integrations"
4. Vas a ver "Personal API token" con un string que empieza con "apify_api_"
5. Tap el botón "Copy" al lado del token
6. Pégamelo en el chat

(Es un solo token por cuenta, NO necesitas crear varios para distintos agentes.)
```

**Cuando el alumno pegue el token**:
- **NO imprimas el valor** después
- Guardarlo en `.dev.vars` como `APIFY_TOKEN`
- Responder: *"Listo, ya lo guardé."*

---

## Paso 3 — Verificar saldo (opcional pero recomendado)

```
Antes de cerrar, vamos a confirmar que tienes los $5 gratis cargados:

1. En el dashboard, busca arriba el "Billing" o "Usage"
2. Debe decir algo como "$5 free credits this month"
3. Si dice $0, espera 10 minutos y refresca — a veces tarda

Estos $5 te van a durar varios meses si tu agente corre 1-3 veces al día.
Si llegas a topar, hay opción de cargar $5 manualmente cuando pase.
```

---

## Paso 4 — Apify CLI (opcional, no necesario para v1)

Si el alumno es más techy y quiere también el CLI:

```bash
# Mac/Linux:
brew install apify-cli   # si tiene brew
# o:
npm install -g apify-cli

# Windows:
npm install -g apify-cli
```

Pero **NO es obligatorio**. El agente usa solo el token + fetch directo a la API de Apify, sin CLI.

---

## Actores disponibles que tu agente puede usar

(Para tu referencia interna como Claude — qué actor escoger según el caso del alumno)

| Fuente | Actor recomendado | Costo |
|---|---|---|
| Twitter/X — tweets de cuentas | `scrape.badger/twitter-tweets-scraper` | $0.0002/tweet |
| Twitter/X — followers/following | `kaitoeasyapi/premium-x-follower-scraper-following-data` | $0.0001/user |
| Instagram — perfiles | `apify/instagram-profile-scraper` | $0.005/profile |
| Instagram — posts | `apify/instagram-post-scraper` | $0.005/post |
| LinkedIn — perfiles | `johnvc/linkedin-profile-api` | $0.02/1000 profiles |
| Google Maps — negocios | `compass/google-maps-extractor` | $0.001/place |
| Sitios genéricos | `apify/website-content-crawler` | $1/1000 pages |

Estos actores se llaman desde el código del agente con fetch directo (sin SDK):

```typescript
const url = `https://api.apify.com/v2/acts/<actor-id>/run-sync-get-dataset-items?token=${APIFY_TOKEN}`;
const resp = await fetch(url, { method: "POST", body: JSON.stringify(input) });
const items = await resp.json();
```

---

## Reglas del walkthrough

1. **Solo correr esto si el agente del alumno realmente necesita scraping.** Si no, skip.
2. **El plan free de Apify ($5/mes) renueva el día 1 de cada mes.** Si ya gastaron eso, esperar al próximo ciclo o cargar manual.
3. **El token de Apify es ÚNICO por cuenta**, no por proyecto. El mismo se usa en todos los agentes del alumno.
4. **NO imprimir el token** después de recibirlo.
5. **Si el alumno solo necesita scrapear sitios públicos sencillos**, considerar usar `fetch()` directo del worker sin Apify. Es gratis pero más frágil.
