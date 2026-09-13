# 👤 Endpoints: Usuarios y Estadísticas

Host base: Variable de entorno `BACKEND_URL`

---

## 1. Crear Usuario
- **Ruta:** `POST /user`
- **Body:** `{ "discordId": string, "username": string }`
- **Respuesta:** `{ "ok": true }` (409 si ya existe)

---

## 2. Obtener Usuario
- **Ruta:** `GET /user/:discordId`
- **Respuesta:**
```json
{
  "_id": "67...",
  "discordId": "123456789012345678",
  "username": "Gamer123",
  "balance": 650,
  "dailyStreak": 7,
  "maxDailyStreak": 14,
  "lastDaily": "2026-09-12T10:00:00.000Z"
}
```

---

## 3. Obtener Usuario con Cartas
- **Ruta:** `GET /user/:discordId/cards`
- **Respuesta:** Objeto usuario con el array `cards` poblado con instancias de `Card`.

---

## 4. Obtener Estadísticas y Suerte
- **Ruta:** `GET /user/:discordId/stats`
- **Respuesta:**
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
  "luck": {
    "totalCards": 35,
    "luckPercentage": 138.5,
    "luckDelta": "+38.5%",
    "tier": "Lucky",
    "tierCode": "LUCKY",
    "eligibleForLeaderboard": true,
    "breakdown": {
      "common": 18,
      "rare": 11,
      "epic": 4,
      "legendary": 2,
      "mythic": 0
    }
  }
}
```
