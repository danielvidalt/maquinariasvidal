# Hand-off — Maquinarias y Construcciones Vidal (maquinariasvidal.cl)

**Última actualización:** 2026-06-10

## Datos del proyecto
- Repo: `danielvidalt/maquinariasvidal`, branch `main` (auto-deploy a GitHub Pages al hacer push)
- Dominio: `maquinariasvidal.cl` (CNAME en repo) — **ahora detrás de Cloudflare** (proxy activo)
- Sitio estático, sin build, todo en `index.html` (HTML+JS inline) + `styles.css`

## Cambios hechos en esta sesión (orden cronológico, todos commiteados y pusheados)

1. **`7499378`** — Fix overlapping subtitle lines en sección "Maquinarias disponibles"
   - `styles.css`: agregado `.subtitulo-seccion + .subtitulo-seccion{margin-top:0}` (el margen negativo de `.subtitulo-seccion` colapsaba con el siguiente elemento)

2. **`5f5a6d1`** — Botones de WhatsApp e Instagram en el hero
   - Agregado `.redes` div con 3 íconos SVG inline: 2 WhatsApp (`+56966068918` y `+56966375925`) + Instagram (`@myc_vidal`)
   - Todos con `target="_blank" rel="noopener noreferrer"`
   - JSON-LD `sameAs` actualizado con los 3 links

3. **`d0a0223`** — Navbar con logo
   - Navbar sticky agregado al inicio del `<body>` con logo + links de navegación (Maquinarias, Servicios, Nosotros, Reserva)
   - Logo procesado desde `Logo.jpeg` (carpeta padre "website Alexander") → `img/logo.jpeg` (480x362, calidad 90) y `img/logo.webp`

4. **`f53e68a`** / **`c39374f`** — Ajuste de color del navbar para matchear el logo
   - Iteración 1: color promedio de esquina (#f0eeeb) — el usuario indicó que se veía blanco vs el logo
   - **Fix**: se calculó el color *dominante* (más frecuente) vía PIL + `Counter` sobre grid de píxeles → `#eae9e5`, verificado pixel-exacto (234,233,229) con screenshot headless Chrome

5. **`c44f082`** — Cambio de logo a "Logo blanco.png"
   - Reemplazo de `img/logo.jpeg` y `img/logo.webp` por la versión de fondo blanco (1444x1089 → 480x362, calidad 90, ~24.5KB)
   - Navbar `background` ajustado a `#fefefe` para matchear el nuevo fondo blanco

6. **`977f523`** — Conversión de imágenes a WebP
   - 19 imágenes de producto (arrays JS de `maquinarias`/`servicios`) cambiadas de `img/NAME.jpg|jpeg` → `webp/NAME.webp` (22 reemplazos, algunas imágenes compartidas)
   - Imagen hero de Unsplash: agregado `&fm=webp` a la URL (preload + `<img>`)
   - Reducción de peso ~71% (1.9MB → 551KB) en el grid de productos

7. **`bd59f44`** — Hardening de seguridad (cyber security)
   - Meta tags agregados: `Content-Security-Policy` (CSP), `referrer` (`strict-origin-when-cross-origin`), `robots`
   - Función `escapeHTML()` agregada y aplicada en `pintarMaquinarias()`, `pintarServicios()`, `actualizarReserva()`, `abrirModal()` (modalSpecs) — previene DOM XSS
   - Inputs del formulario de reserva: agregado `maxlength` y `autocomplete` apropiados (nombre, rut, telefono, direccionCliente, comentario)
   - Inputs de cantidad: agregado `max="999"`
   - **Fix de vulnerabilidad real**: el mensaje de WhatsApp se armaba con `%0A` literal y se insertaba sin encodear en `window.open()` → posible inyección/corrupción de URL. Se cambió a `\n` reales + `encodeURIComponent(msg)` aplicado al mensaje completo, y `window.open(url, "_blank", "noopener")`

## Configuración de Cloudflare (fuera del repo, hecha vía dashboard)

El dominio `maquinariasvidal.cl` quedó detrás de Cloudflare (plan Free) para poder agregar headers HTTP que GitHub Pages no soporta:

- **Nameservers** apuntando a Cloudflare (`simon.ns.cloudflare.com`, `vera.ns.cloudflare.com`)
- **DNS**: los 4 registros `A` (185.199.108-111.153) y el `CNAME` de `www` puestos en **Proxied** (nube naranja). El registro `TXT` de google-site-verification quedó en DNS only (correcto)
- **SSL/TLS**: modo **Full**
- **Edge Certificates**: 
  - Always Use HTTPS: ON
  - HSTS: habilitado, max-age 6 meses, sin includeSubDomains, sin preload
  - No-Sniff Header (X-Content-Type-Options: nosniff): ON
- **Rules → Transform Rules → "Security Headers"** (Response Header Transform, aplica a "All incoming requests", todos "Set static"):
  - `X-Frame-Options: DENY`
  - `Permissions-Policy: geolocation=(), microphone=(), camera=()`
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' https://images.unsplash.com data:; font-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests`

**Resultado verificado en securityheaders.com: nota A** (capada en A por `'unsafe-inline'` en `script-src`, necesario porque todo el JS del sitio está inline en `index.html`).

## Errores encontrados y solución
- **Color matching**: promedio de píxeles ≠ percepción visual → usar color dominante (moda) en vez de promedio
- **Edit tool "file modified since read"**: re-leer el archivo antes de reintentar el Edit
- **Headless Chrome screenshot**: con `.hero{min-height:100vh}`, viewports grandes hacen que el hero ocupe toda la pantalla y tape el resto — usar `--dump-dom` + grep o snippets aislados en vez de screenshot full-page
- **Cloudflare**: el botón de nube en la tabla de DNS records a veces no responde al clic directo → usar el botón **"Edit"** de cada fila, que abre un panel con el toggle de Proxy status
- **DNS cache local**: tras activar el proxy, `curl`/`dig` locales seguían devolviendo IPs viejas de GitHub Pages por caché → verificar con `dig @1.1.1.1` / `dig @8.8.8.8`, o forzar con `curl --resolve dominio:443:IP_CLOUDFLARE`

## Pendiente / opcional (no solicitado aún)
- **Bug pre-existente** (anterior a esta sesión): overflow horizontal en mobile (~390px) — el `<h1>` "Maquinarias y Construcciones Vidal" y los botones se salen del viewport. Confirmado vía `git stash` que ya existía antes de los cambios de esta sesión. El usuario aún no ha pedido arreglarlo.
- **A+ en securityheaders.com**: requeriría mover todo el JS inline de `index.html` a un archivo `.js` externo y quitar `'unsafe-inline'` de `script-src` (en el meta CSP y en la Transform Rule de Cloudflare). Es un refactor grande, no urgente — se le explicó el trade-off al usuario y quedó pendiente de decisión.

## Último estado
Todo lo anterior está commiteado y pusheado a `origin/main` (último commit: `bd59f44`). Cloudflare configurado y verificado funcionando (headers de seguridad activos, nota A). No hay cambios sin commitear en el código del sitio (solo `.DS_Store` y carpeta `Info Maquinarias/` sin trackear, no relacionados).
