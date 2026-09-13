# 🃏 Endpoints: Cartas y Gacha

Rutas gestionadas por `routers/cardRouter.js` y `controllers/cardController.js`.

---

## 1. Catálogo Completo
- **Ruta:** `GET /card`
- **Respuesta:** Array con todas las cartas ordenadas por rareza (`type: 1`).

---

## 2. Obtener Carta por ID
- **Ruta:** `GET /card/:id`
- **Respuesta:** Objeto `Card` individual.

---

## 3. Obtener Carta Aleatoria por Rareza
Permite obtener una carta aleatoria de una rareza específica:
- `GET /card/mythic`
- `GET /card/legendary`
- `GET /card/epic`
- `GET /card/rare`
- `GET /card/common`

---

## 4. Tirada de Gacha del Usuario
- **Ruta:** `POST /user/:discordId/card/random`
- **Coste:** 100 monedas.
- **Acción:**
  1. Valida que `user.balance >= 100`.
  2. Descuenta 100 monedas (`balance -= 100`).
  3. Suma estadísticas: `totalCoinsSpent += 100`, `cardsOpenedCount += 1`.
  4. Realiza el roll ($0-999$) y añade la carta a la colección del usuario.
  5. Registra la transacción inmutable en el Ledger (`CARD_BUY`) con `metadata: { cardId, cardType, cardName, roll }`.
- **Respuesta:** Objeto `Card` obtenido.

---

## 5. Crear Nueva Carta (Admin / Multipart)
- **Ruta:** `POST /card`
- **Headers:** `Content-Type: multipart/form-data`
- **Campos:**
  - `data`: JSON string con:
    ```json
    {
      "name": "Nombre de la Carta",
      "type": "mythic",
      "description": "Lore o historia",
      "author": "Nombre o Discord Tag"
    }
    ```
  - `image`: Archivo binario de imagen (procesado por Multer e `imageService`).
- **Respuesta (201 Created):** Objeto `Card` creado con `imageUrl` y `imagePublicId`.

---

## 6. Eliminar Carta
- **Ruta:** `DELETE /card/:id`
- **Efecto:** Elimina la carta de MongoDB y borra automáticamente su imagen en el proveedor de hosting configurado (Cloudinary / Supabase / Local).
