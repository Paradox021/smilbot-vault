# 🌐 Arquitectura Frontend

SPA web de Smilbot para que los usuarios inicien sesión con Discord y consulten su perfil, inventario, mercado y leaderboards.
Las reglas de código y estructura detalladas están en el `AGENTS.md` del repo del frontend; esta nota recoge lo que el **resto del ecosistema** necesita saber.

- **Repo local:** `smilbot-project/smilbot-frontend`
- **Estado:** sesión, perfil, fases 1 y 2 del [Plan Frontend](./Plan%20Frontend.md) (Inicio con daily y tirada) e identidad visual de [Rediseño Visual](./Rediseno%20Visual.md) (fases 1 y 2, sin los sonidos opcionales). Sin desplegar.

---

## 1. Stack

Vite · React · TypeScript · React Router (SPA) · TanStack Query · Tailwind CSS + shadcn/ui · Motion · Vitest + MSW.

- **CSS de shadcn:** se importa del paquete (`@import 'shadcn/tailwind.css'` en `src/index.css`), con `shadcn@4.21.4` como `devDependency`. Ya no hay copia manual en `src/shared/ui/`.

---

## 2. Integración con el Backend

| Aspecto | Decisión |
| :--- | :--- |
| Autenticación | Cookies httpOnly del backend (`smilbot_access`, `smilbot_refresh`). El frontend no guarda tokens. Ver [Autenticación](../Backend%20API/Autenticacion.md). |
| Peticiones | `fetch` con `credentials: 'include'` contra `VITE_API_URL`. |
| Token expirado | `401 TOKEN_EXPIRED` → un único `POST /auth/refresh` → reintento. |
| Login | Navegación a `/auth/discord?returnTo=<ruta>` (`getDiscordLoginUrl`); vuelve con `?login=success\|error`. Ver "Volver a la ruta original tras el login". |
| Usuario actual | `GET /auth/me` (incluye `isAdmin` y `avatarUrl`). |
| Avatar | `avatarUrl` de `/auth/me` y `/user/:id/stats` (tipos `CurrentUser` y `UserStats`). |

### Configuración que debe coincidir

| Frontend | Backend | Desarrollo |
| :--- | :--- | :--- |
| Origen de la app | `FRONTEND_URL` | `http://localhost:5173` |
| `VITE_API_URL` | Dirección del servidor (`PORT`) | `http://localhost:<PORT>` |
| — | `DISCORD_REDIRECT_URI` (registrada en el Developer Portal) | `http://localhost:<PORT>/auth/discord/callback` |

- En desarrollo, frontend y API están ambos en `localhost` (mismo *site*): las cookies funcionan con `SameSite=lax` sin HTTPS.
- En producción conviene servir frontend y API bajo el mismo dominio (p. ej. `app.dominio` y `api.dominio`). Con dominios distintos hacen falta `COOKIE_SAMESITE=none` y `COOKIE_SECURE=true`.

### Rutas de la SPA

| Ruta | Acceso | Descripción |
| :--- | :--- | :--- |
| `/` | Pública | Portada con "Entrar con Discord". Recibe `?login=success\|error` (es el destino de `FRONTEND_URL`). Con sesión redirige a la ruta de retorno o a `/inicio`. |
| `/inicio` | Privada | Tablero: reclamo del daily (`POST /user/:id/dailyBalance`) con cuenta atrás y aviso de racha en peligro, racha y progreso de pity. Ver [Plan Frontend](./Plan%20Frontend.md#4-inicio-fase-1). |
| `/tirar` | Privada | Abrir un sobre (`POST /user/:id/card/random`) con la animación escalada por rareza. Ver [Plan Frontend](./Plan%20Frontend.md#5-tirada-fase-2) y "Tirada" más abajo. |
| `/perfil` | Privada | Identidad de `GET /auth/me` y stats completas de `GET /user/:id/stats`. Se abre desde el menú de usuario. |

Las rutas privadas sin sesión redirigen a `/` guardando la ruta de retorno.

### Cabecera y Datos Compartidos
- **Cabecera (con sesión):** navegación principal (solo los destinos ya implementados, ver `NAV_ITEMS`), **saldo** siempre visible y menú de usuario desplegable (Perfil, Cerrar sesión).
- **Móvil:** la navegación principal se oculta en la cabecera y pasa a una barra inferior fija (`BottomNav`) con los mismos `NAV_ITEMS`.
- **Stats del usuario:** `GET /user/:id/stats` vive en la feature `stats` (query key `['stats', 'user', discordId]`) y la comparten la cabecera, Inicio y Perfil: una sola petición en caché.
- **Daily:** la mutación espera al refetch de las stats antes de terminar, así el saldo, la racha y la cuenta atrás ya están al día cuando se muestra el resultado.
- **Inventario:** `GET /user/:id/cards` vive en la feature `inventory` (query key `['inventory', 'user', discordId]`).
- **Rendimiento:** la cabecera está en el bundle principal, así que no usa Motion (el contador del saldo usa `requestAnimationFrame` y las monedas que vuelan, Web Animations). Motion solo se carga en las páginas diferidas.
- **Efectos del saldo** (`shared/lib/balanceFx.ts`): las features que cambian el saldo emiten un efecto (`gain` con el rectángulo de origen al reclamar el daily, `spend` al tirar) y `BalancePill` lo pinta: monedas que vuelan hasta la píldora o "−100" que flota. Así `daily` y `pull` no importan `stats` para animar.

### Identidad Visual
La decisión está en [Rediseño Visual](./Rediseno%20Visual.md) y la referencia de implementación en el `DESIGN.md` del frontend.
- **Tokens:** en `src/index.css` (variables de shadcn/ui). `--radius` es 16 px (`rounded-lg`, tarjetas); las cartas y los campos usan `rounded-md` (12 px); botones, insignias y pestañas, `rounded-full`. `--ring` ya incluye la opacidad.
- **Fuentes:** Nunito (`@fontsource-variable/nunito`) en `font-heading`, aplicada por defecto a `h1`-`h3`; Inter en el resto.
- **Mascota:** componente `Mascot` (`shared/ui/mascot.tsx`), con `srcset` sobre `src/assets/mascot/smilbot-{128,320,640}.webp`. Favicon en `public/favicon-32.png` y `public/apple-touch-icon.png`. Los ficheros fuente están en la bóveda (`Frontend/assets/`).
- **Una acción principal por pantalla:** en Inicio, la tarjeta de hoy (`TodayCard`) muestra el daily si está disponible y, si no, "Abrir un sobre"; es la única con botón naranja. Con el daily disponible, la mítica garantizada ofrece un acceso secundario a la tirada.
- **Movimiento:** tokens en `shared/config/motion.ts` (Motion) y sus equivalentes CSS en `src/index.css` (`--ease-*`, `--animate-*`). Botones con pulsado (`scale` 0,97).
- **Smilbot como guía:** `SmilbotSays` (`shared/ui/smilbot-says.tsx`), mascota con bocadillo. En Inicio el mensaje sale de `features/home/smilbotMessage.ts` (racha en peligro > daily > mítica cerca > primer sobre > saludo del día); en la tirada, de `features/pull/smilbotReaction.ts` (Épica o mejor y repetidas).
- **Errores y carga de sesión:** `ErrorState` (`shared/ui/error-state.tsx`) con la mascota y "Reintentar"; la comprobación de sesión muestra a Smilbot "despertándose".
- **Login:** mascota grande con halo y motas fijas, texto de bienvenida y "Continuar con Discord" con el logo de Discord (`DiscordIcon`, de Simple Icons).

### Tirada
- **Secuencia** (`features/pull/sequence.ts`, un reducer puro): `idle` (sobre en reposo) → `shuffling` (petición en vuelo, el sobre tiembla) → `anticipation` (luz de la rareza por la línea de corte) → `opening` (la solapa se rasga, 400 ms) → `revealing` (la carta sube y gira) → `result`. Los tiempos y efectos por rareza están en `features/pull/reveal.ts`.
- **Sobre:** `PackArt` (`features/cards`, SVG con los tokens) y su geometría en `features/cards/pack.ts`; en la tirada se pinta dos veces recortado por la línea de corte (solapa y cuerpo). Se puede rasgar deslizando el dedo por la línea de corte (`useSwipeToTear`); el botón sigue siendo el camino principal.
- **Vibración:** una vez por tirada al revelar Épica o mejor, donde exista `navigator.vibrate`.
- **Saltar:** el botón "Saltar", tocar el escenario o cualquier tecla llevan al resultado. Si se salta antes de que llegue la respuesta, la carta se muestra en cuanto llega.
- **Movimiento reducido:** sin temblor, rasgado, giro ni efectos; el sobre se funde y aparece la carta.
- **"Nueva" y repetidas:** se calculan con el inventario en caché antes de tirar (la página lo precarga): `copies` es el número de copias tras la tirada ("Ya la tenías · ×N"). Si el inventario no ha cargado, no se muestra ninguna de las dos.
- **Tras tirar:** se añade la carta al inventario en caché y se invalidan el inventario y las stats (saldo y pity). Los errores `400` (sin saldo) y `404` (rareza sin cartas, no se cobra) vuelven al estado inicial con su mensaje.
- **Accesibilidad:** la cara de la carta no está en el DOM hasta el giro; el resultado (con la reacción de Smilbot) se anuncia en una región `aria-live` y el bocadillo es solo visual.
- **Cartas:** las caras (`CardFront` con imagen de reserva, `CardBack`) son de la feature `cards` y las reutilizará la colección. `CardFront` muestra la imagen entera a su proporción original (máx. 18 rem de alto y 20 rem de ancho), así que el sobre, el dorso (5:7) y la cara se centran en una caja fija para que nada cambie de sitio al abrir o girar.
- **Precarga de la imagen:** la mutación espera a que la imagen de la carta cargue (máx. 3 s) antes de terminar, mientras el sobre tiembla; así la cara aparece ya con su tamaño final.

### Volver a la ruta original tras el login
1. `RequireAuth` manda al usuario sin sesión a `/` con `state.returnTo` (ruta + query actuales, sin el parámetro `login`).
2. `LoginPage` / `LoginButton` construyen `/auth/discord?returnTo=<codificada>` con `getDiscordLoginUrl`.
3. El backend valida `returnTo` y, tras el login, redirige a `FRONTEND_URL` + esa ruta con `?login=success` ([Autenticación](../Backend%20API/Autenticacion.md#volver-a-la-ruta-original-returnto)).
4. Si un usuario con sesión llega a `/`, va a `returnTo` o, si no hay, a `/inicio`.

`src/features/auth/returnTo.ts` sanea la ruta en el cliente: solo rutas relativas que empiezan por `/` (no `//` ni `/\`) y nunca la raíz `/`. La validación completa (longitud, caracteres de control, normalización) la hace el backend.

### Avatar
`UserAvatar` (`src/features/auth/components/`, sobre el `Avatar` de shadcn) muestra `avatarUrl` con las iniciales del usuario como respaldo (sin URL o si la imagen falla). Se usa en el menú de usuario de la cabecera (`UserMenu`) y en `ProfileHeader`. Aún no hay vistas de leaderboard.

---

## 3. Alcance

El alcance y el orden de las pantallas están en [Plan Frontend](./Plan%20Frontend.md).

---

## 4. Pendientes

Las necesidades del backend, las incoherencias de la bóveda y los pendientes propios del frontend se recogen en [Pendientes Frontend](./Pendientes%20Frontend.md). Añade ahí cualquier endpoint o cambio que el frontend necesite antes de implementarlo en el backend.
