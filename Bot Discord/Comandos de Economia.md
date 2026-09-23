# 🪙 Comandos de Economía y Cartas

Prefijo por defecto: `.`

| Comando | Alias | Permisos | Descripción |
| :--- | :--- | :--- | :--- |
| `.balance` | `.bal`, `.coins` | Todos | Muestra el saldo actual del usuario. |
| `.dailybalance` | `.db`, `.daily` | Todos | Reclama las 100 monedas diarias y gestiona racha. |
| `.getcard` | `.gc`, `.buycard` | Todos | Compra 1 sobre gacha aleatorio por 100 monedas (con sistema de pity). |
| `.mycards` | `.mc`, `.inventory` | Todos | Abre el navegador de cartas del inventario propio. |
| `.show` | `.card` | Todos | Muestra una carta específica del inventario en detalle. |
| `.allcards` | `.ac` | Todos | Catálogo de todas las cartas existentes en el juego. |
| `.market` | `.mkt`, `.shop` | Todos | Abre el mercado libre de ofertas del servidor. |
| `.stats` | `.profile` | Todos | Muestra perfil, rachas, pity, analíticas de balance y suerte. |
| `.top` | `.topluck`, `.leaderboard` | Todos | Muestra el ranking de suerte (`.top luck` o `.top luck worst`). |
| `.createcard` | `.newcard` | **Admin** | Crea una nueva carta con imagen adjunta o URL. |

---

## Detalle de Interacciones Clave

### 🃏 `.getcard` (Sistema de Pity)
- **Coste:** 100 monedas.
- **Tirada Regular:**
  - El embed muestra la carta obtenida de forma limpia y coleccionable (`Rarity: <Rarity> | Author: <Author>`).
- **Tirada por Pity (`isPity: true`):**
  - Se activa al alcanzar 250 tiradas consecutivas sin obtener una carta mítica.
  - El bot envía como mensaje de texto superior:
    `🌟 **Smilbot took pity on your bad luck and granted you a guaranteed Mythic card!**`
  - El embed de la carta permanece 100% limpio y auténtico, preservando el protagonismo del arte de la carta.

### 📊 `.stats` (Perfil Económico y Pity)
Incluye cinco bloques informativos en el embed:
1. **💰 Economy:** Saldo actual, total ganado y total gastado.
2. **🔥 Daily Streaks:** Racha actual, récord histórico y dailies totales.
3. **🃏 Cards & Market:** Cartas en inventario, sobres abiertos y ventas en mercado.
4. **✨ Gacha Pity:**
   - `• Progress: X/250`
   - `• Guaranteed In: Y pulls`
   - `• ▰▰▱▱▱▱▱▱▱▱ Z%` (Barra de progreso visual animada en 10 segmentos).
5. **🎲 Gacha Luck:** Rating de suerte, tier (`GODLY`, `LUCKY`, `AVERAGE`, `UNLUCKY`, `CURSED`), desviación de media y desglose por rareza.
   - Si el usuario obtuvo cartas míticas por pity (`pityMythics > 0`), se refleja en el breakdown al final de las míticas como `... N🔴 (+M Pity)` para cuadrar el inventario total sin inflar la puntuación de suerte.

### 🏆 `.top luck` (Leaderboard de Suerte)
- Muestra el Top 5 de jugadores más afortunados o desafortunados (`.top luck worst`).
- En el desglose de rarezas de cada jugador, si posee míticas otorgadas por pity (`pityMythics > 0`), se añade `(+M Pity)` al final del conteo de míticas para máxima transparencia en la competición.
