# 🪙 Comandos de Economía y Cartas

Prefijo: `.` · Código en `src/commands/economy/`.

| Comando | Alias | Middlewares | Endpoint | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `.balance` | `.bal` | `checkUser` | `GET /user/:id` | Muestra el saldo actual. |
| `.dailybalance` | `.db` | `checkUser` | `POST /user/:id/dailyBalance` | Reclama las 100 monedas diarias y gestiona la racha. |
| `.getcard` | `.buycard`, `.gc` | `checkUser` | `POST /user/:id/card/random` | Compra 1 sobre gacha por 100 monedas (con pity). |
| `.mycards` | — | `checkUser` | `GET /user/:id/cards` | Abre el navegador de tu inventario. |
| `.show @usuario` | — | `checkUser` | `GET /user/:id/cards` | Abre el inventario de otro usuario. |
| `.allcards` | `.cards` | — | `GET /card` | Catálogo de todas las cartas del juego. |
| `.market` | `.shop` | `checkUser` | `GET /market/:serverId/offers`, `GET .../:offerId`, `POST .../:offerId/buy` | Abre el mercado del servidor y permite comprar. |
| `.sell <carta> <precio>` | `.offer` | `checkUser` | `GET /user/:id/cards`, `POST /market/:serverId/offers` | Pone a la venta 1 copia de una carta propia. |
| `.cancelsale [carta]` | `.unsell`, `.myoffers` | `checkUser` | `GET /market/:serverId/offers`, `DELETE .../:offerId` | Sin argumentos lista tus ofertas; con nombre cancela esa oferta y te devuelve la carta. |
| `.stats [@usuario]` | `.profile`, `.estadisticas`, `.mystats` | `checkUser` | `GET /user/:id/stats` | Perfil, rachas, pity y suerte (propio o del usuario mencionado). |
| `.top luck [worst]` | `.topluck`, `.lucktop`, `.leaderboard`, `.ranking` | `checkUser` | `GET /leaderboard/luck` | Top 5 de suerte. `worst`, `bad`, `cursed` o `asc` muestran los menos afortunados. |
| `.createcard` | `.newcard`, `.addcard` | `checkAdmin` | `POST /card` | Crea una carta con imagen adjunta o URL al final. |

El bot solo usa el leaderboard de suerte; los de rachas, fortuna y coleccionistas no tienen comando.

---

## Detalle de Interacciones Clave

### 🃏 `.getcard` (Sistema de Pity)
- **Coste:** 100 monedas.
- **Tirada regular:** el embed muestra la carta de forma limpia (`Rarity: <Rarity> | Author: <Author>`).
- **Tirada por pity (`isPity: true`):** se activa tras 250 tiradas seguidas sin mítica. El bot envía como texto encima del embed:
  `🌟 **Smilbot took pity on your bad luck and granted you a guaranteed Mythic card!**`
  El embed de la carta no cambia.

### 🛠️ `.createcard`
```text
.createcard <nombre | "nombre con espacios"> <rareza> <descripción> [url_imagen]
```
- La imagen puede ir adjunta al mensaje o como URL al final de la descripción.
- `rareza`: `common`, `rare`, `epic`, `legendary` o `mythic`.

### 📊 `.stats` (Perfil Económico y Pity)
Cinco bloques en el embed:
1. **💰 Economy:** saldo actual, total ganado y total gastado.
2. **🔥 Daily Streaks:** racha actual (efectiva, ya calculada por el backend), récord y dailies totales. Además `Next Daily` (`available now` o tiempo relativo `<t:…:R>` desde `nextDailyAt`) y `Streak Ends` (desde `streakExpiresAt`, solo si hay racha activa).
3. **🃏 Cards & Market:** cartas en inventario, sobres abiertos y ventas en el mercado.
4. **✨ Gacha Pity:** `Progress: X/250`, `Guaranteed In: Y pulls` y barra de progreso de 10 segmentos.
5. **🎲 Gacha Luck:** porcentaje, tier, desviación y desglose por rareza. Si hay míticas por pity se muestra `N🔴 (+M Pity)`.

### 🏆 `.top luck`
- Top 5 con `minPulls = 20`.
- Si un jugador tiene míticas por pity, se añade `(+M Pity)` a su conteo de míticas.

### 🏪 Mercado (`.market`, `.sell`, `.cancelsale`)
El mercado es **por servidor**: cada comando usa el `guildId` del mensaje y no funciona por mensaje directo. Contrato en [Endpoints Mercado](../Backend%20API/Endpoints%20Mercado.md).

- **`.market`:** muestra cuántas ofertas hay y un botón para abrir el listado (menú de selección + botón de compra, 10 por página).
  - Al comprar, el bot vuelve a pedir la oferta (`GET .../:offerId`) y avisa si ya no está activa.
  - Comprar tu propia oferta se rechaza en el bot antes de llamar al backend.
  - Los errores muestran el mensaje del backend (`You don't have enough money`, etc.).
- **`.sell <carta> <precio>`:** el último argumento es el precio (entero positivo) y el resto el nombre de la carta, que puede tener espacios y no distingue mayúsculas. El bot busca la carta en tu inventario y la publica por `cardId`. La copia queda retenida hasta que se vende o se cancela.
  ```text
  .sell Gato Pan 150
  ```
- **`.cancelsale [carta]`:** sin argumentos lista tus ofertas activas en el servidor. Con el nombre de la carta cancela tu oferta más reciente de esa carta y te la devuelve.
