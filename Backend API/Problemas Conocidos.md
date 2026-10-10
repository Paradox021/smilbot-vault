# 🐞 Problemas Conocidos (Backend y Bot)

Fallos reales detectados al contrastar la bóveda con el código del backend y del bot (2026-10-10). No son errores de documentación: el comportamiento descrito en las notas es el real. Cuando se arregle uno, se quita de aquí y se actualiza la nota afectada.

---

## 🔴 Prioridad Alta

| Problema | Dónde | Efecto |
| :--- | :--- | :--- |
| **La compra en el mercado no es atómica** | `marketController.buyOffer` | Oferta, carta, saldos y Ledger se actualizan en pasos separados sin sesión transaccional (el `AGENTS.md` del backend lo exige). Un fallo a mitad deja datos inconsistentes. Ver [Mercado y Transacciones](../Modelos%20y%20Contratos/Mercado%20y%20Transacciones.md). |
| **El mercado del bot no funciona** | Bot: `marketService.ts`, `MarketInteractions.ts` | La compra consulta `GET /market/:serverId/offers/:offerId` (no existe → siempre "This offer no longer exists"). El body usa `buyerDiscordId` / `sellerDiscordId` / `cardId` en vez de `discordId` / `username` / `cardName`. No hay comandos para publicar ni cancelar. |

## 🟠 Prioridad Media

| Problema | Dónde | Efecto |
| :--- | :--- | :--- |
| **Racha mostrada desactualizada** | `getUserStats`, `getLeaderboardStreaks` | `dailyStreak` solo se recalcula al reclamar: perfiles y `/leaderboard/streaks?type=current` muestran rachas ya rotas. Ver [Economía y Rachas](../Modelos%20y%20Contratos/Economia%20y%20Rachas.md). |
| **Publicar oferta sin validar** | `marketController.addOffer` | No valida `price` (negativo, 0 o no numérico) ni la presencia de `discordId`. La respuesta no incluye el `_id` de la oferta. |
| **Borrar carta deja referencias huérfanas** | `DELETE /card/:id` | Los inventarios y ofertas siguen apuntando a la carta borrada. |

## 🟡 Prioridad Baja

| Problema | Dónde | Efecto |
| :--- | :--- | :--- |
| `GET /user/:id/stats` con usuario inexistente | `getUserStats` | Responde `500` en vez de `404`. |
| `GET /card/:id` inexistente | `cardController.getCard` | Responde `200 null` en vez de `404`. |
| `DELETE /user/:id` usa el `_id` de MongoDB | `userService.deleteUser` | Es la única ruta de `/user/:id` donde `:id` no es el `discordId`. |
| Errores de negocio como `500` | Mercado (publicar, cancelar), cartas | "No tienes la carta", "la oferta no es tuya" o "no está activa" devuelven `500` en lugar de `400`/`403`/`404`. |
| `GET /market/:serverId/offers` devuelve el `User` completo del vendedor | `marketService.getAllMarketOffers` | Expone saldo, inventario y estadísticas del vendedor; bastaría con `discordId` y `username`. |
| Imágenes locales con el mismo nombre | `localProvider` | Se sobrescriben entre cartas. |
| Comentario de `metadata` desactualizado | `models/transaction.js` | Describe campos de mercado que el código no escribe. |
