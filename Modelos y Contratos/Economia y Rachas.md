# 💰 Economía Virtual y Rachas

---

## 1. Reglas del Sistema de Rachas (`dailyBalance`)

El comando diario otorga **100 monedas** y actualiza el contador de racha consecutiva:

- **Cooldown mínimo:** **23 horas** desde el último reclamo (`lastDaily`).
- **Ventana de Racha (23h a 48h):**
  $$\text{dailyStreak} = \text{previousStreak} + 1$$
- **Racha Rota (> 48h):**
  $$\text{dailyStreak} = 1$$
- **Récord Histórico:**
  $$\text{maxDailyStreak} = \max(\text{dailyStreak}, \text{previousMaxStreak})$$

---

## 2. Respuestas y Formatos

```json
{
  "ok": true,
  "balance": 650,
  "dailyStreak": 7,
  "previousStreak": 6,
  "maxDailyStreak": 14,
  "previousMaxStreak": 14,
  "totalDailiesClaimed": 42
}
```

---

## 3. Feedback Visual en el Bot
- 🔥 **Racha activa (≥ 2 días):** Muestra el fuego y los días consecutivos.
- 🎉 **Nuevo récord (≥ 2 días y supera récord):** Felicitación destacada.
- 💔 **Racha rota:** Avisa de forma amigable que la racha previa se perdió.

Véase también:
- [[Backend API/Endpoints Usuarios y Stats|Endpoints Usuarios y Stats]]
