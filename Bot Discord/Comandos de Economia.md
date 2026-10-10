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
| `.market` | `.shop` | `checkUser` | `GET /market/:serverId/offers` | Abre el mercado del servidor. |
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
2. **🔥 Daily Streaks:** racha actual, récord y dailies totales.
3. **🃏 Cards & Market:** cartas en inventario, sobres abiertos y ventas en el mercado.
4. **✨ Gacha Pity:** `Progress: X/250`, `Guaranteed In: Y pulls` y barra de progreso de 10 segmentos.
5. **🎲 Gacha Luck:** porcentaje, tier, desviación y desglose por rareza. Si hay míticas por pity se muestra `N🔴 (+M Pity)`.

### 🏆 `.top luck`
- Top 5 con `minPulls = 20`.
- Si un jugador tiene míticas por pity, se añade `(+M Pity)` a su conteo de míticas.

### 🏪 `.market`
- Muestra cuántas ofertas hay y un botón para abrir el listado (menú de selección + botón de compra).
- **El bot no permite publicar ni cancelar ofertas**: `marketService.addOffer` y `removeOffer` existen pero ningún comando los usa.
- ⚠️ **La compra no funciona.** Antes de comprar, el bot pide la oferta a `GET /market/:serverId/offers/:offerId`, que no existe en el backend; recibe `404` y responde *"This offer no longer exists."*. Además, `marketService` envía `buyerDiscordId` / `sellerDiscordId` / `cardId`, cuando el backend espera `discordId`, `username` y `cardName`. Ver [Endpoints Mercado](../Backend%20API/Endpoints%20Mercado.md).
