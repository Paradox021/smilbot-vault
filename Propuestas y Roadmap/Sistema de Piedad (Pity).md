# 💡 Propuesta: Sistema de Piedad Mítica (Pity 250)

> [!NOTE]
> **Estado:** 💡 *En Exploración / Pendiente de Implementación*
> **Alcance (Scope):** 🌐 *Ecosistema Completo (Backend API + Bot Discord)*
> - 🔌 **Backend:** Campo `pityCount` en `User`, lógica forzada a las 250 en `cardController.js`, auditoría en `Transaction`.
> - 🤖 **Bot Discord:** Barra de progreso en `.stats` y mensaje especial en `.getcard`.
> **Aviso para Agentes:** Esta mecánica NO está en el código actual de producción.

---

## 1. Concepto y Justificación
En el sistema gacha actual, las cartas **Míticas** tienen una probabilidad del **0.5%** (1 de cada 200 sobres). 
Para proteger a los jugadores de rachas extremas de mala suerte (por ejemplo, abrir 500 sobres sin ver una sola Mítica), se propone un sistema de **Piedad Dura (*Hard Pity*) a las 250 tiradas**.

---

## 2. Mecánica de Juego y Algoritmo
1. **Contador de Piedad (`pityCount`):**
   - Cada sobre abierto por el usuario que **no** entregue una carta Mítica incrementa el contador:
     $$\text{pityCount} = \text{pityCount} + 1$$
2. **Garantía en 250:**
   - Si un usuario alcanza `pityCount === 250`, el roll de rareza es forzado directamente a **Mythic (`type: 4`)**.
3. **Reinicio de Contador:**
   - En el instante exacto en que el usuario obtiene una carta Mítica (sea por suerte natural o por haber alcanzado el límite de 250), el contador vuelve inmediatamente a **0**:
     $$\text{pityCount} = 0$$

---

## 3. Impacto en el Backend
- **Modelo `User`:** Añadir campo `pityCount: { type: Number, default: 0 }`.
- **Controlador `cardController.js` (`rollRandomCard`):**
  - Evaluar si `user.pityCount >= 249`. Si es verdadero, forzar rareza Mítica y reiniciar contador.
  - De lo contrario, tirar RNG normal. Si sale Mítica, reiniciar a 0; si sale otra rareza, `pityCount++`.
- **Auditoría (`Transaction`):** Registrar en la metadata si la Mítica fue obtenida por suerte (`natural`) o por piedad (`pity_guaranteed`).

---

## 4. Impacto en el Bot de Discord
- **Comando `.stats`:** Mostrar barra de progreso de piedad:
  ```text
  🔮 Piedad Mítica: 184 / 250 tiradas
  ```
- **Mensaje de Apertura (`.getcard`):** Notificación especial si la carta fue entregada por haber alcanzado el hito de 250 sobres.
