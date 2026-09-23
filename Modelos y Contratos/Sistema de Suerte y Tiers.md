# 🍀 Sistema de Suerte y Tiers

---

## 1. Neutralización Contra el Mercado

Para evitar que los usuarios compren cartas de alta rareza en el mercado para inflar artificialmente su suerte, el backend calcula:

$$\text{Cartas Reales de Gacha} = \text{Inventario} - \text{Compras en Mercado} + \text{Ventas en Mercado}$$

---


## 2. Neutralización Contra el Sistema de Pity

Para que las cartas míticas deterministas obtenidas mediante la red de seguridad de pity (a las 250 tiradas) no distorsionen artificialmente el ranking ni penalicen al usuario:

$\\text{Míticas Evaluadas} = \\max(0, \\text{MíticasGacha} - \\text{pityMythicsCount})$
$\\text{Tiradas Totales Evaluadas} = \\max(0, \\text{TotalTiradasGacha} - \\text{pityMythicsCount})$

Se descuenta cada mítica de pity tanto del acumulador de míticas como del total de cartas evaluadas, dejando que el ranking refleje exclusivamente el azar puro.

## 3. Escala de Tiers de Suerte

| Tier Code | Nombre | Icono | Rango Delta (%) |
| :--- | :--- | :--- | :--- |
| `GODLY` | **Godly Luck** | 🌟 | Mayor a `+40.0%` |
| `LUCKY` | **Lucky** | 🍀 | `+15.0%` a `+40.0%` |
| `AVERAGE` | **Average** | ⚖️ | `-15.0%` a `+15.0%` |
| `UNLUCKY` | **Unlucky** | 🌧️ | `-30.0%` a `-15.0%` |
| `CURSED` | **Cursed** | 💀 | Menor a `-30.0%` |

---

## 4. Requisito de Calificación
- **Mínimo de tiradas:** Se requieren al menos **20 aperturas** (`minPulls = 20`) para que el usuario sea elegible en el Leaderboard (`.top luck`).

Véase también:
- [[Backend API/Endpoints Leaderboard|Endpoints Leaderboard]]
