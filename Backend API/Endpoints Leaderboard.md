# 🏆 Endpoints: Rankings y Clasificaciones (Leaderboards)

El backend provee endpoints agregados de alto rendimiento en `routers/leaderboardRouter.js`:

> 🔐 Todas las rutas requieren autenticación. Permisos por ruta en [Autenticación](./Autenticacion.md#4-matriz-de-permisos).

Cada entrada de los rankings incluye `avatarUrl` (URL del CDN de Discord, o el avatar por defecto si el usuario no tiene hash guardado). Ver [Avatar de Discord](./Endpoints%20Usuarios%20y%20Stats.md).

---

## 1. Top Suerte en Gacha
- **Ruta:** `GET /leaderboard/luck`
- **Query Params:**
  - `order`: `'desc'` (default, más afortunados) o `'asc'` (más desafortunados).
  - `minPulls`: Mínimo de cartas válidas para la suerte (`luck.totalCards`, no sobres abiertos). Default `20`, mínimo `1`.
  - `limit`: Límite de resultados (default: `10`, entre `1` y `100`).
- El cálculo de cada jugador es el mismo que `luck` en `GET /user/:id/stats` ([Sistema de Suerte y Tiers](../Modelos%20y%20Contratos/Sistema%20de%20Suerte%20y%20Tiers.md)). Empates: por `username`.
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
      "avatarUrl": "https://cdn.discordapp.com/embed/avatars/3.png",
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
        "mythic": 0,
        "pityMythics": 0
      },
      "pityMythicsCount": 0
    }
  ]
}
```

---

## 2. Top Rachas Diarias
- **Ruta:** `GET /leaderboard/streaks?type=current&limit=10`
- **Query Params:** `type` (`'current'` o `'max'`, default `'current'`), `limit` (default: `10`, máx. `100`).
- **`current`:** solo usuarios con `dailyStreak > 0` **y** `lastDaily` dentro de las últimas 48 h (racha viva). Las rachas ya rotas no aparecen, aunque el valor guardado no se haya recalculado. Ver [Economía y Rachas](../Modelos%20y%20Contratos/Economia%20y%20Rachas.md#2-racha-efectiva).
- **`max`:** usuarios con `maxDailyStreak > 0`, sin filtro de fecha.
- **Respuesta:**
```json
[
  { "discordId": "111...", "username": "Player1", "avatarUrl": "https://cdn.discordapp.com/avatars/111.../<hash>.png?size=128", "streak": 45 },
  { "discordId": "222...", "username": "Player2", "avatarUrl": "https://cdn.discordapp.com/embed/avatars/1.png", "streak": 32 }
]
```

---

## 3. Top Fortuna Histórica
- **Ruta:** `GET /leaderboard/wealth?type=earned&limit=10`
- **Query Params:** `type` (`'earned'` para histórico ganado, default, o `'current'` para saldo actual), `limit` (default: `10`, máx. `100`).
- **Respuesta:**
```json
[
  { "discordId": "111...", "username": "Player1", "avatarUrl": "https://...", "amount": 25400 },
  { "discordId": "333...", "username": "Player3", "avatarUrl": "https://...", "amount": 18200 }
]
```

---

## 4. Top Coleccionistas de Cartas
- **Ruta:** `GET /leaderboard/cards?limit=10` (máx. `100`)
- Cuenta todas las copias del inventario (`cardsCount`), igual que en `/stats`.
- **Respuesta:**
```json
[
  { "discordId": "111...", "username": "Player1", "cardsCount": 120, "avatarUrl": "https://..." },
  { "discordId": "444...", "username": "Player4", "cardsCount": 98, "avatarUrl": "https://..." }
]
```
