# 🐞 Problemas Conocidos (Backend y Bot)

Fallos reales detectados al contrastar la bóveda con el código del backend y del bot (2026-10-10). No son errores de documentación: el comportamiento descrito en las notas es el real. Cuando se arregle uno, se quita de aquí y se actualiza la nota afectada.

> ✅ Resueltos el 2026-10-10: compra del mercado no atómica, mercado del bot roto (compra imposible, campos incorrectos, sin publicar ni cancelar), validación al publicar, errores del mercado como `500` y datos completos del vendedor en el listado. Ver [Endpoints Mercado](./Endpoints%20Mercado.md).

---

## 🟠 Prioridad Media

| Problema | Dónde | Efecto |
| :--- | :--- | :--- |
| **Racha mostrada desactualizada** | `getUserStats`, `getLeaderboardStreaks` | `dailyStreak` solo se recalcula al reclamar: perfiles y `/leaderboard/streaks?type=current` muestran rachas ya rotas. Ver [Economía y Rachas](../Modelos%20y%20Contratos/Economia%20y%20Rachas.md) y [Pendientes Frontend](../Frontend/Pendientes%20Frontend.md). |
| **Borrar carta deja referencias huérfanas** | `DELETE /card/:id` | Los inventarios y ofertas siguen apuntando a la carta borrada. Falta decidir si se bloquea el borrado o se limpian las referencias. |

## 🟡 Prioridad Baja

| Problema | Dónde | Efecto |
| :--- | :--- | :--- |
| `GET /user/:id/stats` con usuario inexistente | `getUserStats` | Responde `500` en vez de `404`. |
| `GET /card/:id` inexistente | `cardController.getCard` | Responde `200 null` en vez de `404`. |
| `DELETE /user/:id` usa el `_id` de MongoDB | `userService.deleteUser` | Es la única ruta de `/user/:id` donde `:id` no es el `discordId`. |
| Errores de cartas como `500` | `cardController` | Carta inexistente al borrar, rareza sin cartas o `type` desconocido al crear devuelven `500` en lugar de `400`/`404`. |
| Imágenes locales con el mismo nombre | `localProvider` | Se sobrescriben entre cartas. |
| Comentario de `metadata` desactualizado | `models/transaction.js` | Describe campos de mercado que el código no escribe. |
