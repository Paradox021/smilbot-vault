# 🏆 Endpoints: Rankings y Clasificación

---

## 1. Ranking de Suerte Gacha
- **Ruta:** `GET /leaderboard/luck`
- **Query Params:**
  - `order`: `'desc'` (los más suertudos) o `'asc'` (los más desafortunados / malditos).
  - `minPulls`: Mínimo de aperturas de sobre requeridas (default: `20`).
  - `limit`: Cantidad máxima de registros devueltos (default: `5`).
- **Respuesta:**
```json
{
  "order": "desc",
  "minPulls": 20,
  "leaderboard": [
    {
      "rank": 1,
      "discordId": "111222333444555666",
      "username": "LuckyGamer",
      "totalCards": 45,
      "luckPercentage": 145.2,
      "luckDelta": "+45.2%",
      "tier": "Godly Luck",
      "tierCode": "GODLY",
      "breakdown": {
        "common": 20,
        "rare": 14,
        "epic": 7,
        "legendary": 3,
        "mythic": 1
      }
    }
  ]
}
```
