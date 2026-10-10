# 👤 Endpoints: Usuarios, Economía, Inventario y Ledger

Rutas gestionadas por `routers/userRouter.js` y `controllers/userController.js`.

> 🔐 Todas las rutas requieren autenticación. Permisos por ruta en [Autenticación](./Autenticacion.md#4-matriz-de-permisos).

En todas las rutas `:id` es el **`discordId`**, salvo en `DELETE /user/:id` (ver la tabla).

---

## 1. Directorio Completo de Rutas (`/user`)

| Método | Ruta | Descripción |
| :--- | :--- | :--- |
| `GET` | `/user` | Lista todos los usuarios registrados en el sistema. |
| `GET` | `/user/:id` | Documento `User` completo. `404` si no existe. |
| `POST` | `/user` | Registra un nuevo usuario (`{ discordId, username }`). `201`; `409` si ya existe. |
| `POST` | `/user/id` | Obtiene o crea un usuario a partir del body (`{ discordId, username }`). Siempre `200`. |
| `DELETE` | `/user/:id` | Elimina un usuario. ⚠️ Aquí `:id` es el **`_id` de MongoDB**, no el `discordId`. |
| `POST` | `/user/:id/dailyBalance` | Reclamo diario de 100 monedas y cálculo de racha. |
| `POST` | `/user/:id/card/random` | Tirada de sobre gacha (100 monedas, RNG 0-999, pity). Ver [Endpoints Cartas](./Endpoints%20Cartas.md#4-tirada-de-gacha-del-usuario). |
| `GET` | `/user/:id/cards` | El `User` con `cards` sustituido por las cartas completas + `count`. |
| `GET` | `/user/:id/cards/number` | Conteo de cartas agrupado por rareza: `[{ "_id": <type>, "count": N }]`. |
| `POST` | `/user/:id/card/:cardId` | Añade una copia de una carta al inventario. Devuelve el `User`. |
| `DELETE` | `/user/:id/card/:cardId` | Quita una copia de una carta del inventario. Devuelve el `cardId`. |
| `POST` | `/user/:id/balance/:amount` | Ajuste administrativo: añade saldo (`ADMIN_ADJUST`). Devuelve el `User`. |
| `DELETE` | `/user/:id/balance/:amount` | Ajuste administrativo: descuenta saldo (`ADMIN_ADJUST`). Devuelve el `User`. |
| `GET` | `/user/:id/stats` | Perfil económico, métricas agregadas, pity y suerte en gacha. |
| `GET` | `/user/:id/transactions` | Historial paginado del Ledger (`?page=1&limit=10`, `limit` máx. 100). |

---

## 2. Esquema de Usuario (`User`)

```typescript
interface UserCard {
  cardId: ObjectId | string;   // Ref a 'Card'
  count: number;               // Cantidad de copias en posesión (min: 0)
}

interface UserSchema {
  _id: ObjectId | string;
  discordId: string;           // ID único en Discord (unique). Siempre string: es un snowflake de 64 bits
  username: string;            // Nombre de usuario en Discord
  balance: number;             // Saldo de monedas virtuales (default: 0)
  cards: UserCard[];           // Array de cartas agrupadas
  lastDaily: Date;             // Último reclamo del daily (default: hace 24 h)
  lastTimeCommand: Date;       // Alias retrocompatible de lastDaily

  // --- Métricas y Telemetría ---
  dailyStreak: number;         // Racha guardada en el último reclamo (ver "Racha mostrada" en Economía y Rachas)
  maxDailyStreak: number;      // Récord histórico alcanzado
  previousMaxStreak: number;   // Récord consolidado cuando se rompió la última racha
  totalDailiesClaimed: number; // Total de dailies reclamados de por vida
  totalCoinsEarned: number;    // Monedas ganadas en toda la historia
  totalCoinsSpent: number;     // Monedas gastadas en toda la historia
  cardsOpenedCount: number;    // Sobres gacha abiertos
  pityCount: number;           // Tiradas consecutivas sin mítica actuales (min: 0)
  pityMythicsCount: number;    // Total histórico de míticas obtenidas vía pity (min: 0)
  tokenVersion: number;        // Versión de sesión para revocar refresh tokens (select: false, no se devuelve)
  createdAt: Date;
  updatedAt: Date;
}
```

---

## 3. Detalle de Respuestas Clave

### A. Reclamo Diario (`POST /user/:id/dailyBalance`)
```json
{
  "ok": true,
  "balance": 650,
  "dailyStreak": 11,
  "previousStreak": 10,
  "maxDailyStreak": 11,
  "previousMaxStreak": 10,
  "isNewRecord": true,
  "totalDailiesClaimed": 42
}
```
- **Cooldown activo (< 23 h):** `400 { "error": "You can't use this command yet...", "diffHours": 5, "diffMinutes": 12 }`, con el tiempo que falta.
- Reglas de racha y récord en [Economía y Rachas](../Modelos%20y%20Contratos/Economia%20y%20Rachas.md).

### B. Inventario (`GET /user/:id/cards`)
Devuelve el documento `User` completo, pero con `cards` reemplazado por las cartas pobladas (solo las que tienen `count > 0`), ordenadas por `type` ascendente. No está paginado.
```json
{
  "discordId": "123456789012345678",
  "username": "Gamer123",
  "balance": 650,
  "cards": [
    { "_id": "642d...", "name": "SwimmingLarry", "description": "...", "type": 4, "imageUrl": "https://...", "author": "...", "count": 2 }
  ]
}
```

### C. Estadísticas y Suerte (`GET /user/:id/stats`)
```json
{
  "discordId": "123456789012345678",
  "username": "Gamer123",
  "balance": 650,
  "dailyStreak": 11,
  "maxDailyStreak": 11,
  "previousMaxStreak": 10,
  "totalDailiesClaimed": 42,
  "totalCoinsEarned": 5400,
  "totalCoinsSpent": 4750,
  "cardsCount": 35,
  "cardsOpenedCount": 30,
  "marketSalesCount": 5,
  "pity": {
    "pullsSinceLastMythic": 244,
    "pityThreshold": 250,
    "pullsUntilGuaranteed": 6,
    "pityMythicsCount": 0
  },
  "luck": {
    "totalCards": 28,
    "luckPercentage": 97.4,
    "luckDelta": "-2.6%",
    "tier": "Average",
    "tierCode": "AVERAGE",
    "pityMythicsCount": 0,
    "eligibleForLeaderboard": true,
    "breakdown": {
      "common": 15,
      "rare": 9,
      "epic": 3,
      "legendary": 1,
      "mythic": 0,
      "pityMythics": 0
    }
  }
}
```

**Qué cuenta cada campo de cartas:**
- `cardsCount`: copias en el inventario ahora mismo (incluye las compradas en el mercado y las añadidas por un admin).
- `cardsOpenedCount`: sobres gacha abiertos en toda la historia.
- `luck.totalCards`: cartas que **cuentan para la suerte**. Por cada carta: `max(0, inventario − compradas en mercado) + vendidas en mercado`; después se restan las míticas de pity. Normalmente es ≤ `cardsOpenedCount`, pero puede superarlo si hay cartas añadidas por un admin o anteriores a la telemetría. Ver [Sistema de Suerte y Tiers](../Modelos%20y%20Contratos/Sistema%20de%20Suerte%20y%20Tiers.md).

**Casos especiales:**
- **Sin tiradas válidas (`luck.totalCards = 0`):** `luckPercentage: 100`, `luckDelta: "+0.0%"`, `tier: "Average"`, `tierCode: "AVERAGE"`. No significa suerte media, sino que no hay datos.
- **No elegible para el leaderboard (menos de 20 cartas válidas):** `eligibleForLeaderboard: false` y se añade `cardsNeeded` con las que faltan.
- **No incluye** `lastDaily` ni `createdAt`.
- ⚠️ **Usuario inexistente:** responde `500 { "error": "Usuario no encontrado" }` en lugar de `404`, porque el servicio lanza el error antes de que el controlador lo compruebe.

### D. Historial de Transacciones (`GET /user/:id/transactions`)
```json
{
  "transactions": [
    {
      "_id": "66bcde123...",
      "discordId": "123456789012345678",
      "type": "DAILY_CLAIM",
      "amount": 100,
      "balanceBefore": 550,
      "balanceAfter": 650,
      "metadata": {
        "streakAtClaim": 11,
        "previousStreak": 10,
        "previousMaxStreak": 10,
        "isNewRecord": true,
        "streakBroken": false
      },
      "createdAt": "2026-08-14T11:00:00.000Z",
      "updatedAt": "2026-08-14T11:00:00.000Z"
    }
  ],
  "total": 52,
  "page": 1,
  "totalPages": 6
}
```
Orden: más reciente primero. El contenido de `metadata` por tipo está en [Mercado y Transacciones](../Modelos%20y%20Contratos/Mercado%20y%20Transacciones.md).

---

## 4. Errores
Salvo los casos indicados (`404` en `GET /user/:id`, `/cards`, `/cards/number` y `/transactions`; `409` al crear; `400` en el daily y en la tirada), los errores devuelven `500 { "error": "<mensaje>" }`.
