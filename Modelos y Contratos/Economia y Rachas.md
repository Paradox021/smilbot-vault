# 💰 Economía Virtual y Rachas

Lógica en `services/userService.js` (`dailyBalance`) y métodos de `models/user.js`. Endpoint: `POST /user/:id/dailyBalance` ([Endpoints Usuarios y Stats](../Backend%20API/Endpoints%20Usuarios%20y%20Stats.md)).

---

## 1. Reglas del Sistema de Rachas (`dailyBalance`)

Los dos umbrales están definidos una sola vez en `models/user.js`: `DAILY_COOLDOWN_HOURS = 23` y `STREAK_WINDOW_HOURS = 48`.

El comando diario otorga **100 monedas** y actualiza el contador de racha consecutiva. Sea `h` las horas desde `lastDaily`:

- **Cooldown (`h < 23`):** no se reclama. `400` con `diffHours` y `diffMinutes` restantes.
- **Racha continúa (`23 ≤ h ≤ 48`):**
  $$	ext{dailyStreak} = 	ext{previousStreak} + 1$$
- **Racha rota (`h > 48`):**
  $$	ext{dailyStreak} = 1$$
  y, si había racha, se consolida el récord anterior: `previousMaxStreak = maxDailyStreak`.
- **Récord histórico:**
  $$	ext{maxDailyStreak} = \max(	ext{dailyStreak}, 	ext{maxDailyStreak})$$
- **Nuevo récord (`isNewRecord`):** `true` solo el día en que la racha actual supera por primera vez el récord consolidado: `previousMaxStreak > 0` y `dailyStreak = previousMaxStreak + 1`.

Cada reclamo suma `totalCoinsEarned += 100` y `totalDailiesClaimed += 1`, actualiza `lastDaily` y registra un `DAILY_CLAIM` en el Ledger.

> Un usuario nuevo tiene `lastDaily` = hace 24 h, así que puede reclamar en cuanto se crea y su primera racha es 1.

---

## 2. Racha Efectiva

El `dailyStreak` **guardado** solo se recalcula al reclamar. Los endpoints de lectura calculan la **racha efectiva** sin modificar el dato:

| Método de `User` | Devuelve |
| :--- | :--- |
| `isStreakAlive()` | `true` si `h ≤ 48`. |
| `getEffectiveStreak()` | `dailyStreak` si es `> 0` y la racha sigue viva; si no, `0`. |
| `getNextDailyAt()` | `lastDaily + 23 h`. Si ya pasó, se puede reclamar. |
| `getStreakExpiresAt()` | `lastDaily + 48 h`, o `null` si la racha efectiva es `0`. |

| Endpoint | Racha que devuelve |
| :--- | :--- |
| `GET /user/:id/stats` | Efectiva, más `lastDaily`, `nextDailyAt` y `streakExpiresAt`. Ver [Endpoints Usuarios y Stats](../Backend%20API/Endpoints%20Usuarios%20y%20Stats.md). |
| `GET /leaderboard/streaks?type=current` | Solo usuarios con `dailyStreak > 0` y `lastDaily` en las últimas 48 h. Ver [Endpoints Leaderboard](../Backend%20API/Endpoints%20Leaderboard.md). |
| `GET /user/:id`, `GET /auth/me` | El valor **guardado** (documento `User` sin transformar). |

---

## 3. Respuesta del Reclamo

```json
{
  "ok": true,
  "balance": 650,
  "dailyStreak": 7,
  "previousStreak": 6,
  "maxDailyStreak": 14,
  "previousMaxStreak": 14,
  "isNewRecord": false,
  "totalDailiesClaimed": 42
}
```
`previousStreak` es la racha guardada antes de este reclamo.

---

## 4. Feedback Visual en el Bot (`.dailybalance`)
- 🔥 **Racha activa (≥ 2 días):** muestra los días y, si no es récord, el récord entre paréntesis.
- 🎉 **Nuevo récord (`isNewRecord`):** felicitación indicando el récord superado.
- 💔 **Racha rota (`dailyStreak = 1` y `previousStreak ≥ 2`):** avisa de que se perdió la racha anterior.

Véase también:
- [[Backend API/Endpoints Usuarios y Stats|Endpoints Usuarios y Stats]]
