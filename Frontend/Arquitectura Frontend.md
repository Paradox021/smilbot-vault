# 🌐 Arquitectura Frontend

SPA web de Smilbot para que los usuarios inicien sesión con Discord y consulten su perfil, inventario, mercado y leaderboards.
Las reglas de código y estructura detalladas están en el `AGENTS.md` del repo del frontend; esta nota recoge lo que el **resto del ecosistema** necesita saber.

- **Repo local:** `smilbot-project/smilbot-frontend`
- **Estado:** base inicializada (stack, cliente HTTP, sesión y perfil mínimo). Sin desplegar.

---

## 1. Stack

Vite · React · TypeScript · React Router (SPA) · TanStack Query · Tailwind CSS + shadcn/ui · Motion · Vitest + MSW.

---

## 2. Integración con el Backend

| Aspecto | Decisión |
| :--- | :--- |
| Autenticación | Cookies httpOnly del backend (`smilbot_access`, `smilbot_refresh`). El frontend no guarda tokens. Ver [Autenticación](../Backend%20API/Autenticacion.md). |
| Peticiones | `fetch` con `credentials: 'include'` contra `VITE_API_URL`. |
| Token expirado | `401 TOKEN_EXPIRED` → un único `POST /auth/refresh` → reintento. |
| Login | Navegación a `/auth/discord`; vuelve con `?login=success\|error`. |
| Usuario actual | `GET /auth/me` (incluye `isAdmin`). |

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
| `/` | Pública | Inicio con "Entrar con Discord". Recibe `?login=success\|error` (es el destino de `FRONTEND_URL`). Con sesión redirige a `/perfil`. |
| `/perfil` | Privada | Identidad de `GET /auth/me` y stats de `GET /user/:id/stats`. Sin sesión redirige a `/`. |

---

## 3. Alcance Inicial (v1)

1. Login / logout con Discord.
2. Mi perfil: balance, racha y stats.
3. Mi inventario de cartas.
4. Leaderboards.

Siguientes fases: catálogo, mercado (publicar, comprar, cancelar), saldo diario y tiradas desde la web, y panel de admin.

---

## 4. Pendientes

Las necesidades del backend, las incoherencias de la bóveda y los pendientes propios del frontend se recogen en [Pendientes Frontend](./Pendientes%20Frontend.md). Añade ahí cualquier endpoint o cambio que el frontend necesite antes de implementarlo en el backend.
