# 👤 Endpoints: Usuarios, Economía, Inventario y Ledger

Rutas gestionadas por `routers/userRouter.js` y `controllers/userController.js`.

---

## 1. Directorio Completo de Rutas (`/user`)

| Método | Ruta | Descripción |
| :--- | :--- | :--- |
| `GET` | `/user` | Lista todos los usuarios registrados en el sistema. |
| `GET` | `/user/:id` | Obtiene los datos de un usuario por su `discordId`. |
| `POST` | `/user` | Registra un nuevo usuario (`{ discordId, username }`). 409 si ya existe. |
| `POST` | `/user/id` | Obtiene o crea un usuario automáticamente a partir del body. |
| `DELETE` | `/user/:id` | Elimina un usuario del sistema. |
| `POST` | `/user/:id/dailyBalance` | Reclamo diario de 100 monedas y cálculo de racha. |
| `POST` | `/user/:id/card/random` | Tirada de sobre gacha (100 monedas, RNG 0-999). |
| `GET` | `/user/:id/cards` | Inventario completo agrupado por carta con contador (`count: N`). |
| `GET` | `/user/:id/cards/number` | Conteo de cartas del usuario agrupado por rareza (`type`). |
| `POST` | `/user/:id/card/:cardId` | Añade una carta manualmente al inventario del usuario. |
| `DELETE` | `/user/:id/card/:cardId` | Remueve una carta del inventario del usuario. |
| `POST` | `/user/:id/balance/:amount` | Ajuste administrativo: añade saldo (`ADMIN_ADJUST`). |
| `DELETE` | `/user/:id/balance/:amount` | Ajuste administrativo: descuenta saldo (`ADMIN_ADJUST`). |
| `GET` | `/user/:id/stats` | Perfil económico, métricas agregadas y suerte en gacha. |
| `GET` | `/user/:id/transactions` | Historial paginado de movimientos del Ledger (`?page=1&limit=10`). |

---

## 2. Esquema de Usuario (`User`)

```typescript
interface UserCard {
  cardId: ObjectId | string;   // Ref a 'Card'
  count: number;               // Cantidad de copias en posesión (min: 0)
}

interface UserSchema {
  _id: ObjectId | string;
  discordId: string;           // ID único en Discord (index: 1, unique: true)
  username: string;            // Nombre de usuario en Discord
  balance: number;             // Saldo de monedas virtuales (default: 0)
  cards: UserCard[];           // Array de cartas agrupadas
  lastDaily: Date;             // Última fecha de reclamo del daily
  lastTimeCommand: Date;       // Alias retrocompatible de lastDaily

  // --- Métricas y Telemetría ---
  dailyStreak: number;         // Racha consecutiva actual
  maxDailyStreak: number;      // Récord histórico alcanzado
  previousMaxStreak: number;   // Marca de récord consolidada previa
  totalDailiesClaimed: number; // Total de dailies reclamados de por vida
  totalCoinsEarned: number;    // Monedas ganadas en toda la historia
  totalCoinsSpent: number;     // Monedas gastadas en toda la historia
  cardsOpenedCount: number;    // Cartas obtenidas por gacha
  pityCount: number;           // Tiradas consecutivas sin mítica actuales (min: 0)
  pityMythicsCount: number;    // Total histórico de míticas obtenidas vía pity (min: 0)
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

### B. Estadísticas y Suerte (`GET /user/:id/stats`)
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
    "totalCards": 35,
    "luckPercentage": 138.5,
    "luckDelta": "+38.5%",
    "tier": "Lucky",
    "tierCode": "LUCKY",
    "pityMythicsCount": 0,
    "eligibleForLeaderboard": true,
    "breakdown": {
      "common": 18,
      "rare": 11,
      "epic": 4,
      "legendary": 2,
      "mythic": 0,
      "pityMythics": 0
    }
  }
}
```

### C. Historial de Transacciones (`GET /user/:id/transactions`)
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
        "isNewRecord": true
      },
      "createdAt": "2026-08-14T11:00:00.000Z"
    }
  ],
  "total": 52,
  "page": 1,
  "totalPages": 6
}
```
