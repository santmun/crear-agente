# Walkthrough 99 — Errores comunes y soluciones

> Cuando el alumno se atora en cualquier fase, buscar aquí el error.

---

## Errores de instalación

### `command not found: node` (Mac/Linux) o `'node' is not recognized` (Windows)

**Causa**: Node no está en el PATH del sistema.

**Fix Mac**:
```bash
# Reinicia Terminal y prueba otra vez
# Si sigue, agrega manualmente al PATH:
echo 'export PATH="/usr/local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**Fix Windows**:
1. Cierra TODAS las ventanas de PowerShell / CMD / Git Bash
2. Reinicia tu compu
3. Si después de reiniciar sigue, reinstala Node con instalador oficial dejando "Add to PATH" marcado

---

### `npx wrangler login` no abre el navegador

**Fix**:
```
Copia la URL que aparece en tu terminal (empieza con
https://dash.cloudflare.com/oauth...) y pégala manualmente en tu
navegador. Autoriza ahí.
```

---

### Error "EACCES: permission denied" al instalar npm packages

**Causa**: el alumno está intentando instalar packages globalmente sin permisos.

**Fix Mac/Linux**:
```bash
# NO uses sudo. En vez de eso, instala en el folder del proyecto:
cd ~/Desktop/mi-agente
npm install [paquete]
```

**Fix Windows**:
```powershell
# Primero confirma que estás dentro del folder de tu agente
cd $HOME\Desktop\mi-agente
npm install [paquete]
```

Evita abrir PowerShell "como administrador" para instalar paquetes del agente. Si npm
sigue pidiendo permisos, cierra la terminal, abre una PowerShell normal nueva y
reintenta. Si el error aparece al instalar un paquete global (`npm install -g ...`),
reinstala Node.js con el instalador oficial y deja marcada la opción "Add to PATH".

---

## Errores de Cloudflare / Wrangler

### `Unable to detect Cloudflare account` o `not logged in`

**Fix**:
```bash
npx wrangler logout
npx wrangler login
# autoriza en el browser
npx wrangler whoami
```

---

### `Workers not enabled on this account`

**Fix**:
1. Ve a https://dash.cloudflare.com
2. Tap "Workers & Pages" en el menú izquierdo
3. Cloudflare te va a pedir un "subdomain" tuyo (ej. tu-nombre.workers.dev)
4. Confírmalo
5. Vuelve a la terminal y reintenta `npx wrangler deploy`

---

### Error `Adding a payment method is required to use Workers`

**Causa**: Cloudflare a veces pide tarjeta para activar Workers, aunque uses el plan gratis.

**Fix**:
1. Agrega una tarjeta en https://dash.cloudflare.com → Billing
2. Cloudflare NO te va a cobrar nada si quedas dentro del free tier (100K req/día)
3. Hay un límite mensual de gasto que puedes poner para seguridad

---

### `Durable Object class not found` al hacer wrangler deploy

**Causa**: el `wrangler.jsonc` no tiene la migración del DO.

**Fix**: verificar que `wrangler.jsonc` tenga:

```jsonc
{
  "durable_objects": {
    "bindings": [{ "name": "MiAgente", "class_name": "MiAgente" }]
  },
  "migrations": [
    { "tag": "v1", "new_sqlite_classes": ["MiAgente"] }
  ]
}
```

---

## Errores de OpenAI

### `429: Quota exceeded`

**Causa**: el saldo de OpenAI llegó a $0 o pasaste el límite mensual.

**Fix**:
1. Ve a https://platform.openai.com/account/billing
2. Tap "Add to credit balance"
3. Carga $5-10 más
4. Espera 1 minuto y vuelve a intentar

---

### `401: Incorrect API key`

**Causa**: la API key copiada está mal o expiró.

**Fix**: crear una nueva en https://platform.openai.com/api-keys y actualizar en `.dev.vars` + en Cloudflare secrets (`npx wrangler secret put OPENAI_API_KEY`).

---

### `model_not_found: gpt-4o-mini` o similar

**Causa**: la cuenta de OpenAI es muy nueva y aún no tiene acceso a ese modelo.

**Fix**: usar `gpt-3.5-turbo` por ahora — todas las cuentas tienen acceso desde el día 1. Cuando OpenAI verifique la cuenta (1-3 días), gpt-4o-mini se desbloquea.

---

## Errores del pipeline del agente

### El agente no devuelve nada / `noResults`

**Posibles causas**:
- El scraping de Apify no encontró nada (no hay tweets en el rango de fechas)
- El query está mal formado
- Pasaste un cap de API

**Fix**: revisar los logs de `wrangler dev` (terminal donde corre) o `wrangler tail` (si está deployado). Ahí sale el error real.

---

### El agente corre pero no manda notificación

**Posibles causas**:
- `PUSHOVER_USER_KEY` o `PUSHOVER_APP_TOKEN` mal seteados
- Pushover está caído (raro pero pasa)

**Fix manual de Pushover**:
```bash
curl -s "https://api.pushover.net/1/messages.json" \
  --data-urlencode "token=$PUSHOVER_APP_TOKEN" \
  --data-urlencode "user=$PUSHOVER_USER_KEY" \
  --data-urlencode "title=Test" \
  --data-urlencode "message=Test"
```

Si responde `{"status":1,...}` → configuración OK, el problema está en el código del agente.

---

### Test local funciona, pero en producción falla

**Posibles causas**:
- Olvidaste subir un secret a Cloudflare con `npx wrangler secret put`
- El secret subido tiene typo o espacio extra

**Fix**: verifica todos los secrets:
```bash
npx wrangler secret list
```

Debe aparecer el listado completo. Si falta alguno, súbelo.

---

## Errores de Notion (si el agente guarda en Notion)

### `object_not_found` al crear página

**Causa**: la integration de Notion no está compartida con la database donde quieres guardar.

**Fix**:
1. Abre la página/database en Notion
2. Tap "..." (top right) → "Connections" → "+ Add connections"
3. Busca el nombre de tu integration → Add
4. Reintenta

---

### `validation_error: property X does not exist`

**Causa**: el nombre del campo en el código no coincide con el de tu Notion DB.

**Fix**: revisa el schema de tu DB con:

```bash
curl -s "https://api.notion.com/v1/databases/[tu-db-id]" \
  -H "Authorization: Bearer [tu-notion-token]" \
  -H "Notion-Version: 2022-06-28" \
  | jq '.properties | keys'
```

Compara nombres exactos. Capitalización y espacios importan.

---

## Errores generales / cuando no sabes qué pasa

Decirle al alumno:

```
Tranquilo, esto se puede ver complicado pero TODO error tiene fix. Para
ayudarte mejor, necesito 3 cosas:

1. Screenshot del error completo (incluyendo lo que sale ANTES del error,
   no solo el último mensaje rojo)
2. Qué paso estabas haciendo cuando salió
3. Tu sistema operativo y versión

Pásamelo y vemos qué hacemos.

Si nada funciona en este chat, en mi comunidad de Skool tenemos un canal
#soporte-tecnico donde 600+ alumnos también ayudan. Ahí seguramente
encontramos solución.
```
