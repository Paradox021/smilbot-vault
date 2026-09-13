# 👤 Endpoints: Usuarios, Estadísticas y Transacciones

---

## 1. Crear Usuario Inicial
- **Ruta:** `POST /user`
- **Body:** `{ "discordId": string, "username": string }`
- **Respuesta:** `{ "ok": true }` (409 si ya existe).

---

## 2. Obtener Usuario
- **Ruta:** `GET /user/:discordId`
- **Respuesta:** Objeto usuario completo con saldo y rachas.

---

## 3. Obtener Usuario con Cartas Pobladas
- **Ruta:** `GET /user/:discordId/cards`
- **Respuesta:** Objeto usuario con array `cards` con objetos completos de `Card`.

---

## 4. Reclamo Diario (`dailybalance`)
- **Ruta:** `POST /user/:discordId/dailyBalance`
- **Lógica de tiempo:**
  - $\text{diffHours} < 23$: Error 400 con tiempo restante.
  - $23 \le \text{diffHours} \le 48$: Mantiene racha (`dailyStreak = previousStreak + 1`).
  - $\text{diffHours} > 48$: Racha rota (`dailyStreak = 1`, `streakBroken = true`).
- **Respuesta:**
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

---

## 5. Perfil y Estadísticas Globales
- **Ruta:** `GET /user/:discordId/stats`
- **Respuesta:** Contiene `balance`, `dailyStreak`, `maxDailyStreak`, `totalCoinsEarned`, `totalCoinsSpent`, `cardsOpenedCount`, `marketSalesCount` y objeto completo `luck`.

---

## 6. Historial de Transacciones (Ledger)
- **Ruta:** `GET /user/:discordId/transactions?limit=10&page=1`
- **Respuesta:**
```json
{
  "transactions": [
    {
      "_id": "66bcde123...",
      "type": "DAILY_CLAIM",
      "amount": 100,
      "balanceBefore": 550,
      "balanceAfter": 650,
      "createdAt": "2026-08-14T11:00:00.000Z"
    }
  ],
  "total": 52,
  "page": 1,
  "totalPages": 6
}
```
