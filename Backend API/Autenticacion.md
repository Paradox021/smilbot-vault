# 🔐 Autenticación y Autorización

Todas las rutas de la API requieren credenciales salvo `GET /`, `/public/*` (imágenes locales) y `/auth/discord*`.
Implementado en `middleware/auth.js`, `services/authService.js`, `controllers/authController.js` y `libs/authCookies.js`.

---

## 1. Tipos de Credencial

| Cliente | Header | `req.auth` |
| :--- | :--- | :--- |
| 🤖 Bot de Discord | `Authorization: Bot <BOT_API_KEY>` | `{ type: 'bot', isAdmin: true }` |
| 🌐 Frontend (navegador) | Cookie httpOnly `smilbot_access` | `{ type: 'user', discordId, isAdmin }` |
| 📱 Otros clientes de usuario | `Authorization: Bearer <accessToken>` | `{ type: 'user', discordId, isAdmin }` |

- **`BOT_API_KEY`** es un secreto propio de la API compartido con el bot. **No** es el token de Discord del bot.
- **Admin de usuario:** un `discordId` incluido en `ADMIN_DISCORD_IDS` obtiene `isAdmin: true` (mismos permisos que el bot).
- **CSRF:** si el usuario se autentica por cookie, las peticiones que no son `GET/HEAD/OPTIONS` deben traer `Origin` igual a `FRONTEND_URL`; si no → `403 Origen no permitido`.

### Integración en el Bot
```typescript
// src/services/api.ts
export const api = axios.create({
  baseURL: process.env.API_URL,
  headers: { 'Content-Type': 'application/json', Authorization: `Bot ${process.env.BOT_API_KEY}` },
});
```

---

## 2. Tokens de Usuario (JWT HS256)

| Token | Claims | Duración (env) | Dónde viaja |
| :--- | :--- | :--- | :--- |
| Access | `sub=discordId`, `type='access'`, `iss='smilbot-backend'` | `JWT_ACCESS_EXPIRES_IN` (15m) | Cookie `smilbot_access` (path `/`) o header `Bearer` |
| Refresh | `sub`, `type='refresh'`, `ver=tokenVersion` | `JWT_REFRESH_EXPIRES_IN` (30d) | Cookie `smilbot_refresh` (path `/auth`) o body |

- Un refresh token nunca es aceptado como access token (se valida `type`).
- **Revocación:** `User.tokenVersion` (campo `select: false`). `POST /auth/logout` lo incrementa e invalida todos los refresh tokens del usuario. Los access tokens ya emitidos siguen siendo válidos hasta expirar (≤ 15 min).
- El access token de Discord solo se usa durante el callback para leer `/users/@me`; no se guarda ni se reenvía.

---

## 3. Endpoints `/auth`

| Método | Ruta | Auth | Descripción |
| :--- | :--- | :--- | :--- |
| `GET` | `/auth/discord` | — | Genera `state` (cookie `smilbot_oauth_state`, 10 min) y redirige a Discord (`scope=identify`). |
| `GET` | `/auth/discord/callback` | — | Valida `state`, intercambia el `code`, crea el usuario si no existe, emite cookies y redirige a `FRONTEND_URL?login=success` (o `?login=error`). |
| `POST` | `/auth/refresh` | Refresh token | Rota el par de tokens. Con cookie → renueva cookies y responde `200 { message }`. Con body `{ refreshToken }` → `200 { accessToken, refreshToken }`. `401` si expirado/revocado. |
| `POST` | `/auth/logout` | Usuario | Revoca todas las sesiones y borra cookies. `204`. |
| `GET` | `/auth/me` | Usuario | Perfil del usuario autenticado + `isAdmin`. |

### Errores de autenticación
```json
// 401
{ "error": "Token expirado", "code": "TOKEN_EXPIRED" }   // el frontend debe llamar a POST /auth/refresh
{ "error": "Token inválido", "code": "TOKEN_INVALID" }
{ "error": "No autenticado" }
{ "error": "API key del bot inválida" }
// 403
{ "error": "Requiere permisos de administrador" }
{ "error": "No puedes actuar en nombre de otro usuario" }
```

---

## 4. Matriz de Permisos

- **Cualquiera autenticado:** lecturas de catálogo, perfiles, inventarios, stats, mercado y leaderboards.
- **Self:** el usuario solo actúa sobre su propio `discordId`; el bot y los admins sobre cualquiera.
- **Admin:** bot o `ADMIN_DISCORD_IDS`.

| Ruta | Permiso | `discordId` comprobado en |
| :--- | :--- | :--- |
| `GET /card`, `GET /card/:rarity`, `GET /card/:id` | Autenticado | — |
| `POST /card`, `DELETE /card/:id` | Admin | — |
| `GET /user`, `POST /user`, `POST /user/id`, `DELETE /user/:id` | Admin | — |
| `POST/DELETE /user/:id/card/:cardId`, `POST/DELETE /user/:id/balance/:amount` | Admin | — |
| `GET /user/:id`, `/cards`, `/cards/number`, `/stats` | Autenticado | — |
| `POST /user/:id/dailyBalance`, `POST /user/:id/card/random`, `GET /user/:id/transactions` | Self | `:id` |
| `GET /market/:serverId/offers` | Autenticado | — |
| `POST /market/:serverId/offers`, `.../buy`, `DELETE .../:offerId` | Self | `body.discordId` |
| `GET /leaderboard/*` | Autenticado | — |

---

## 5. Variables de Entorno

| Variable | Obligatoria | Descripción |
| :--- | :--- | :--- |
| `JWT_SECRET` | ✅ | Secreto de firma de los JWT. El servidor no arranca sin él. |
| `BOT_API_KEY` | ✅ | API key del bot. El servidor no arranca sin ella. |
| `JWT_ACCESS_EXPIRES_IN` / `JWT_REFRESH_EXPIRES_IN` | — | Por defecto `15m` / `30d`. |
| `ADMIN_DISCORD_IDS` | — | Lista separada por comas. |
| `DISCORD_CLIENT_ID` / `DISCORD_CLIENT_SECRET` | Para login | App OAuth2 del Developer Portal. |
| `DISCORD_REDIRECT_URI` | Para login | Debe coincidir con el registrado en Discord (`.../auth/discord/callback`). |
| `FRONTEND_URL` | Para login | Destino tras el login; activa CORS con credenciales y el chequeo de `Origin`. |
| `COOKIE_SECURE` / `COOKIE_SAMESITE` | — | Por defecto `secure` si `NODE_ENV=production` y `lax`. Frontend y API en dominios distintos → `none` + `secure`. |
