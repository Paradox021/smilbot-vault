# 🎨 Bot Discord: Componentes de UI e Interacciones

Código en `src/components/` (embeds, botones, paginación) y `src/handlers/` (respuesta a interacciones).

---

## 1. Embeds Estándar (`src/components/embeds/`)

- **`createCardEmbed(card)`:** muestra la carta a pantalla completa con el color de su rareza y el autor (`Rarity: <Label> | Author: <Author>`).
  - Mantiene el diseño limpio de la carta, sin banners ni metadatos extra.
  - Los eventos especiales como el **pity** se avisan en el texto del mensaje (`content`), no en el embed.
- **`createCardListEmbed` / `createCardDetailedEmbed`:** vista de lista paginada y vista de carta individual del navegador de cartas.
- **`createMarketListEmbed`, `createEmptyMarketEmbed`, `createPurchaseResultEmbed`:** mercado.
- **`createTextEmbed(color, message)`:** notificaciones de error (rojo), éxito (verde) o aviso (amarillo).
- Colores por rareza en `src/types/CardType.ts` (ver [Cartas y Rarezas](../Modelos%20y%20Contratos/Cartas%20y%20Rarezas.md)).

---

## 2. Navegador de Cartas (`.mycards`, `.show`, `.allcards`)
- Botones creados en `CardButtons.ts` y `PaginationButtons.ts`; la página actual se calcula con `PaginationState.ts` (10 cartas por página).
- Dos vistas: **lista** y **detallada** (una carta a la vez), con botones para cambiar entre ellas y navegar.
- Funciona **sin estado**: el `customId` de cada botón lleva el tipo de listado, el `discordId` cuyas cartas se muestran y la posición. En cada pulsación el bot vuelve a pedir las cartas al backend.
- No hay colectores ni timeout: los botones siguen funcionando mientras exista el mensaje, y **cualquier usuario** puede pulsarlos (navega por el mismo inventario, sin modificar nada).

---

## 3. Mercado (`.market`)
- Botón para abrir el listado, menú de selección de oferta y botón de compra (`MarketButtons.ts`, `MarketInteractions.ts`).
- La selección de oferta se guarda en memoria por usuario y mensaje.
- ⚠️ La compra no funciona actualmente: ver [Comandos de Economía](./Comandos%20de%20Economia.md).

---

## 4. Ayuda (`.help`)
- Menú desplegable de categorías y botones de paginación (`HelpInteractions.ts`).
