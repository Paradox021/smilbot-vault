# 🏆 Endpoints: Rankings y Clasificaciones (Leaderboards)

El backend provee endpoints agregados de alto rendimiento en `routers/leaderboardRouter.js`:

---

## 1. Top Suerte en Gacha
- **Ruta:** `GET /leaderboard/luck`
- **Query Params:**
  - `order`: `'desc'` (default, más afortunados) o `'asc'` (más desafortunados).
  - `minPulls`: Mínimo de aperturas requeridas (default: `20`).
  - `limit`: Límite de resultados (default: `10`).
- **Respuesta:**
```json
{
  "order": "desc",
  "minPulls": 20,
  "leaderboard": [
    {
      "rank": 1,
      "discordId": "111222333444555666",
      "username": "LuckyPlayer",
      "totalCards": 45,
      "luckPercentage": 138.5,
      "luckDelta": "+38.5%",
      "tier": "Lucky",
      "tierCode": "LUCKY",
      "breakdown": {
        "common": 18,
        "rare": 11,
        "epic": 4,
        "legendary": 2,
        "mythic": 0
      }
    }
  ]
}
```

---

## 2. Top Rachas Diarias
- **Ruta:** `GET /leaderboard/streaks?type=current&limit=10`
- **Query Params:** `type` (`'current'` o `'max'`), `limit` (default: `10`).
- **Respuesta:**
```json
[
  { "discordId": "111...", "username": "Player1", "streak": 45 },
  { "discordId": "222...", "username": "Player2", "streak": 32 }
]
```

---

## 3. Top Fortuna Histórica
- **Ruta:** `GET /leaderboard/wealth?type=earned&limit=10`
- **Query Params:** `type` (`'earned'` para histórico ganado o `'current'` para saldo actual).
- **Respuesta:**
```json
[
  { "discordId": "111...", "username": "Player1", "amount": 25400 },
  { "discordId": "333...", "username": "Player3", "amount": 18200 }
]
```

---

## 4. Top Coleccionistas de Cartas
- **Ruta:** `GET /leaderboard/cards?limit=10`
- **Respuesta:**
```json
[
  { "discordId": "111...", "username": "Player1", "cardsCount": 120 },
  { "discordId": "444...", "username": "Player4", "cardsCount": 98 }
]
```
