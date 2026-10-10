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
| `GET` | `/auth/discord` | — | Genera `state` (cookie `smilbot_oauth_state`, 10 min) y redirige a Discord (`scope=identify`). Acepta `?returnTo=/ruta` opcional (ver abajo). |
| `GET` | `/auth/discord/callback` | — | Valida `state`, intercambia el `code`, crea el usuario si no existe (o actualiza su `username` y `avatar` desde `/users/@me`), emite cookies y redirige a `FRONTEND_URL` + `returnTo` con `?login=success` (o a `FRONTEND_URL?login=error`). |
| `POST` | `/auth/refresh` | Refresh token | Rota el par de tokens. Con cookie → renueva cookies y responde `200 { message }`. Con body `{ refreshToken }` → `200 { accessToken, refreshToken }`. `401` si expirado/revocado. |
| `POST` | `/auth/logout` | Usuario | Revoca todas las sesiones y borra cookies. `204`. |
| `GET` | `/auth/me` | Usuario | Perfil del usuario autenticado + `isAdmin` (ver abajo). |

### Volver a la ruta original (`returnTo`)
`GET /auth/discord?returnTo=/perfil?tab=cartas` permite que el frontend vuelva a la página privada que el usuario intentaba abrir.
- **Validación** (`sanitizeReturnTo` en `libs/authCookies.js`): no vacío, como máximo 512 caracteres, empieza por `/` pero no por `//` ni `/\`, sin `\` ni caracteres de control, y debe seguir siendo una ruta del mismo origen tras normalizarla. Si no cumple, se **ignora en silencio** (el login sigue y vuelve a la raíz).
- Se guarda en la cookie httpOnly `smilbot_oauth_return` (path `/auth/discord`, 10 min, `SameSite=lax`), junto al `state`. El callback la borra siempre.
- **Login correcto:** redirige a `FRONTEND_URL` + `returnTo` añadiendo `login=success` (p. ej. `http://localhost:5173/perfil?tab=cartas&login=success`). Sin `returnTo` válido → `FRONTEND_URL?login=success`.
- **Error:** siempre `FRONTEND_URL?login=error`, sin `returnTo`.

### Respuesta de `GET /auth/me`
Objeto **plano**: la forma pública del usuario (la misma que `GET /user/:id`, sin `tokenVersion`) con `isAdmin` añadido en la raíz. Incluye `avatarUrl` y `dailyStreak` es la racha **efectiva** (ver [Economía y Rachas](../Modelos%20y%20Contratos/Economia%20y%20Rachas.md#2-racha-efectiva)). `404` si el usuario ya no existe.
```json
{
  "_id": "66bc...",
  "discordId": "123456789012345678",
  "username": "Gamer123",
  "avatar": "a_1b2c3d4e5f60718293a4b5c6d7e8f901",
  "balance": 650,
  "cards": [{ "cardId": "642d...", "count": 2 }],
  "lastDaily": "2026-10-09T18:00:00.000Z",
  "dailyStreak": 11,
  "...": "resto de campos de User",
  "avatarUrl": "https://cdn.discordapp.com/avatars/123456789012345678/a_1b2c3d4e5f60718293a4b5c6d7e8f901.gif?size=128",
  "isAdmin": false
}
```
Los clientes deberían usarlo solo para la identidad (`discordId`, `username`, `avatarUrl`, `isAdmin`); los datos que cambian se leen de sus endpoints (`/user/:id/stats`, `/user/:id/cards`). Cómo se construye `avatarUrl`: [Endpoints Usuarios y Stats](./Endpoints%20Usuarios%20y%20Stats.md).

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
{ "error": "Requiere la API key del bot" }
```

---

## 4. Matriz de Permisos

- **Cualquiera autenticado:** lecturas de catálogo, perfiles, inventarios, stats, mercado y leaderboards.
- **Self:** el usuario solo actúa sobre su propio `discordId`; el bot y los admins sobre cualquiera.
- **Admin:** bot o `ADMIN_DISCORD_IDS` (`requireAdmin`).
- **Solo bot:** únicamente la API key del bot (`requireBot`, comprueba `req.auth.type === 'bot'`). Cualquier JWT de usuario recibe `403 Requiere la API key del bot`, aunque sea admin o actúe sobre sí mismo.

| Ruta | Permiso | `discordId` comprobado en |
| :--- | :--- | :--- |
| `GET /card`, `GET /card/:rarity`, `GET /card/:id` | Autenticado | — |
| `POST /card`, `DELETE /card/:id` | Admin | — |
| `GET /user`, `POST /user`, `POST /user/id`, `DELETE /user/:id` | Admin | — |
| `POST/DELETE /user/:id/card/:cardId`, `POST/DELETE /user/:id/balance/:amount` | Admin | — |
| `PATCH /user/:id/profile` | Solo bot | — |
| `GET /user/:id`, `/cards`, `/cards/number`, `/stats` | Autenticado | — |
| `POST /user/:id/dailyBalance`, `POST /user/:id/card/random`, `GET /user/:id/transactions` | Self | `:id` |
| `GET /market/:serverId/offers`, `GET /market/:serverId/offers/:offerId` | Autenticado | — |
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
