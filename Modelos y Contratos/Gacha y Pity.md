# 🔮 Gacha, Pity y Variantes Shiny (Roadmap & Diseño)

Propuestas de diseño consensuadas para la evolución del sistema gacha en Smilbot:

---

## 1. Piedad Mítica (Pity System a 250 Tiradas)

- **Objetivo:** Proteger a los jugadores contra rachas estadísticas de mala suerte extrema en cartas **Míticas**.
- **Contador:** Lleva la cuenta de tiradas consecutivas sin obtener una carta Mítica (`pityCount`).
- **Garantía en 250:** Si el contador alcanza **250**, la siguiente carta entregada es **100% garantizada Mítica**.
- **Reinicio:** En el instante en que sale una carta Mítica (sea por suerte natural o por pity garantizado), el contador vuelve a `0`.
- **Visibilidad:** El usuario puede consultar su avance en `.stats` (ej: `🔮 Piedad Mítica: 184/250`).

---

## 2. Variantes "Shiny" ✨ (Cosmético Raro)

- **Roll adicional:** Al salir cualquier carta en un sobre, hay un **2% a 3%** de probabilidad de que sea versión **✨ Shiny**.
- **Presentación:**
  - Embed con borde brillante y título decorado.
  - Nombre con prefijo: `✨ [Nombre de la Carta]`.
  - Distinción visual en `.mycards` y `.show`.
- **Economía:** No altera estadísticas funcionales; se convierte en un ítem de colección con alto valor de reventa en el mercado libre.

---

## 3. Reciclaje por Polvo Arcano (Sinking de Duplicadas)

- **Problema que resuelve:** Duplicadas comunes sin demanda en el mercado.
- **Mecanismo:** Desencantar cartas (`.scrap` / `.recycle`) otorga **Polvo Arcano**.
- **Regla de Oro:** El polvo **NO** permite comprar cartas directas (para evitar fijar un precio techo o suelo a las míticas).
- **Consumibles de Polvo:**
  - *Amuleto Shiny:* Aumenta la probabilidad de Shiny en el siguiente sobre.
  - *Acelerador de Pity:* Suma +5 o +10 tiradas al contador de piedad mítica.
  - *Seguro de Racha:* Protege la racha diaria de perderse si transcurren más de 48 horas.
