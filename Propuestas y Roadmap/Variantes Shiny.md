# 💡 Propuesta: Variantes de Cartas "Shiny" ✨

> [!NOTE]
> **Estado:** 💡 *En Exploración / Pendiente de Implementación*
> **Aviso para Agentes:** Esta mecánica NO está en el código actual de producción. Representa el diseño técnico acordado para su futura integración.

---

## 1. Concepto y Justificación
Para enriquecer el coleccionismo sin alterar las reglas de combate ni inventarios funcionales, se introduce una variante estética ultra-rara: la versión **✨ Shiny / Foil ✨** de cualquier carta existente.

---

## 2. Probabilidad y Mecánica
1. **Doble Tirada (RNG Adicional):**
   - Cuando se genera una carta en un sobre, el backend ejecuta una segunda tirada de dados para determinar si es Shiny:
     - Probabilidad sugerida: **2.0% a 3.0%** (1 de cada 33 a 50 cartas).
2. **Variante en Inventario:**
   - Una carta Shiny se almacena con la bandera `isShiny: true`.
   - En el inventario del usuario, se diferencia de la versión estándar (ej: posee 3 copias normales y 1 copia Shiny).

---

## 3. Experiencia Visual en Discord
- **Embed:** Borde con color brillante, destellos en el título y marca especial:
  ```text
  ✨ Pop Cat [SHINY] ✨
  Rareza: Rare | Artista: Creator
  ```
- **Comando `.show` y `.mycards`:** Icono decorativo `✨` junto al nombre en el listado y visor.

---

## 4. Impacto Económico
- **Mercado Libre P2P (`.market`):** Los jugadores pueden poner a la venta cartas Shiny a precios significativamente mayores, dinamizando la economía de coleccionistas de alto nivel.
- **Sin ventajas jugables:** Mantiene la equidad competitiva.
