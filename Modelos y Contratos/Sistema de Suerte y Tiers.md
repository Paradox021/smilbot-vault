# 🍀 Sistema de Suerte y Tiers

---

## 1. Neutralización Contra el Mercado

Para evitar que los usuarios compren cartas de alta rareza en el mercado para inflar artificialmente su suerte, el backend calcula:

Por cada carta del inventario o vendida alguna vez:

$$\text{Cartas Reales de Gacha} = \max(0,\ \text{Inventario} - \text{Compras en Mercado}) + \text{Ventas en Mercado}$$

Solo cuentan las ofertas con `status: 'SOLD'`. Las cartas añadidas por un admin (`POST /user/:id/card/:cardId`) no se distinguen y cuentan como de gacha.

---


## 2. Neutralización Contra el Sistema de Pity

Para que las cartas míticas deterministas obtenidas mediante la red de seguridad de pity (a las 250 tiradas) no distorsionen artificialmente el ranking ni penalicen al usuario:

$\\text{Míticas Evaluadas} = \\max(0, \\text{MíticasGacha} - \\text{pityMythicsCount})$
$\\text{Tiradas Totales Evaluadas} = \\max(0, \\text{TotalTiradasGacha} - \\text{pityMythicsCount})$

Se descuenta cada mítica de pity tanto del acumulador de míticas como del total de cartas evaluadas, dejando que el ranking refleje exclusivamente el azar puro.

## 3. Cálculo del Porcentaje de Suerte

Cada rareza vale unos puntos y se compara con los puntos esperados por carta según las probabilidades del gacha (`calculateLuckScore` en `services/userService.js`):

| Rareza | Peso |
| :--- | :---: |
| Common | 1 |
| Rare | 3 |
| Epic | 10 |
| Legendary | 50 |
| Mythic | 200 |

$$\text{Esperado por carta} = 0.575 \cdot 1 + 0.30 \cdot 3 + 0.10 \cdot 10 + 0.02 \cdot 50 + 0.005 \cdot 200 = 4.475$$

$$\text{luckPercentage} = \frac{\sum (\text{cartas}_r \cdot \text{peso}_r)}{\text{totalCards} \cdot 4.475} \cdot 100$$

- `luckDelta` = `luckPercentage − 100`, como texto con signo y un decimal (`"+38.5%"`).
- Con `totalCards = 0` devuelve `100` / `"+0.0%"` / `AVERAGE`.

## 4. Escala de Tiers de Suerte

| Tier Code | Nombre | Icono | Rango Delta (%) |
| :--- | :--- | :--- | :--- |
| `GODLY` | **Godly Luck** | 🌟 | Mayor a `+40.0%` |
| `LUCKY` | **Lucky** | 🍀 | `+15.0%` a `+40.0%` |
| `AVERAGE` | **Average** | ⚖️ | `-15.0%` a `+15.0%` |
| `UNLUCKY` | **Unlucky** | 🌧️ | `-30.0%` a `-15.0%` |
| `CURSED` | **Cursed** | 💀 | Menor a `-30.0%` |

---

Límites exactos: `GODLY` si delta `> 40`; `LUCKY` si `≥ 15`; `AVERAGE` si `≥ -15`; `UNLUCKY` si `≥ -30`; si no, `CURSED`.

## 5. Requisito de Calificación
- **Mínimo de tiradas:** se requieren al menos **20 cartas válidas** (`luck.totalCards ≥ 20`, ya descontados mercado y pity; no son los sobres abiertos) para ser elegible en el leaderboard (`.top luck`). Es el valor por defecto de `minPulls`.

Véase también:
- [[Backend API/Endpoints Leaderboard|Endpoints Leaderboard]]
