# 🍀 Sistema de Suerte y Tiers

---

## 1. Neutralización Contra el Mercado

Para evitar que los usuarios compren cartas de alta rareza en el mercado para inflar artificialmente su suerte, el backend calcula:

$$\text{Cartas Reales de Gacha} = \text{Inventario} - \text{Compras en Mercado} + \text{Ventas en Mercado}$$

---

## 2. Escala de Tiers de Suerte

| Tier Code | Nombre | Icono | Rango Delta (%) |
| :--- | :--- | :--- | :--- |
| `GODLY` | **Godly Luck** | 🌟 | Mayor a `+40.0%` |
| `LUCKY` | **Lucky** | 🍀 | `+15.0%` a `+40.0%` |
| `AVERAGE` | **Average** | ⚖️ | `-15.0%` a `+15.0%` |
| `UNLUCKY` | **Unlucky** | 🌧️ | `-30.0%` a `-15.0%` |
| `CURSED` | **Cursed** | 💀 | Menor a `-30.0%` |

---

## 3. Requisito de Calificación
- **Mínimo de tiradas:** Se requieren al menos **20 aperturas** (`minPulls = 20`) para que el usuario sea elegible en el Leaderboard (`.top luck`).

Véase también:
- [[Backend API/Endpoints Leaderboard|Endpoints Leaderboard]]
