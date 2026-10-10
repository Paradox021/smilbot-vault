# 🃏 Endpoints: Cartas y Gacha

Rutas gestionadas por `routers/cardRouter.js` y `controllers/cardController.js`.

> 🔐 Todas las rutas requieren autenticación. Permisos por ruta en [Autenticación](./Autenticacion.md#4-matriz-de-permisos).

- `POST /card` y `DELETE /card/:id` requieren **Admin** (bot o `ADMIN_DISCORD_IDS`).

---

## 1. Catálogo Completo
- **Ruta:** `GET /card`
- **Respuesta:** Array con todas las cartas ordenadas por rareza (`type: 1`).

---

## 2. Obtener Carta por ID
- **Ruta:** `GET /card/:id`
- **Respuesta:** Objeto `Card` individual.
- ⚠️ Si no existe responde `200` con `null` (no `404`). Un `id` con formato inválido da `500`.

---

## 3. Obtener Carta Aleatoria por Rareza
Permite obtener una carta aleatoria de una rareza específica:
- `GET /card/mythic`
- `GET /card/legendary`
- `GET /card/epic`
- `GET /card/rare`
- `GET /card/common`

Si no hay ninguna carta de esa rareza responde `500 { "error": "No cards found for this type" }`.

---

## 4. Tirada de Gacha del Usuario
- **Ruta:** `POST /user/:id/card/random` (`:id` = `discordId`). Es la única ruta de tirada; no existe `/cards/roll`.
- **Coste:** 100 monedas.
- **Errores:** `404` si el usuario no existe; `400 { "error": "No tienes suficientes monedas..." }` si el saldo no llega.
- **Acción y Sistema de Pity:**
  1. Valida que `user.balance >= 100`.
  2. Comprueba el estado de pity (`user.pityCount >= 250`):
     - **Si es Pity (`isPity: true`):** Selecciona directamente una carta Mítica aleatoria (`type: 4`), incrementa `pityMythicsCount += 1` y reinicia `pityCount = 0`.
     - **Si es Tirada Regular (`isPity: false`):** Genera un roll (0-999). Si sale Mítica natural (`roll < 5`), reinicia `pityCount = 0`. Si sale otra rareza, incrementa `pityCount += 1`.
  3. Descuenta 100 monedas (`balance -= 100`).
  4. Suma estadísticas: `totalCoinsSpent += 100`, `cardsOpenedCount += 1`.
  5. Añade la carta al inventario `user.cards` (incrementando `count` si ya existe).
  6. Registra la transacción inmutable en el Ledger (`CARD_BUY`) con `metadata: { cardId, cardType, cardName, roll, isPity, pityCountBefore, pityCountAfter }`.
- **Respuesta:** todos los campos de la `Card` obtenida más `roll`, `isPity` y `pity`:
  ```json
  {
    "_id": "642dbd17fc0fd3e62bde6659",
    "name": "SwimmingLarry",
    "description": "Una carta mítica acuática",
    "type": 4,
    "imageUrl": "https://...",
    "roll": null,
    "isPity": true,
    "pity": {
      "current": 0,
      "threshold": 250,
      "remaining": 250
    }
  }
  ```

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
  - `image`: Archivo binario de imagen (procesado por Multer e `imageService`). Es obligatorio en la práctica: sin él la subida falla con `500`.
- `type` se envía como **nombre** (`common`, `rare`, `epic`, `legendary`, `mythic`) y el backend lo convierte a número (0-4). Un nombre desconocido da `500`.
- `name` debe ser único.
- **Respuesta (201 Created):** Objeto `Card` creado con `imageUrl` y `imagePublicId`.

---

## 6. Eliminar Carta
- **Ruta:** `DELETE /card/:id`
- **Efecto:** Elimina la carta de MongoDB y borra automáticamente su imagen en el proveedor de hosting configurado (Cloudinary / Supabase / Local).
- **Respuesta:** la `Card` eliminada. `500 { "error": "Card not found" }` si no existe.
- ⚠️ No quita la carta de los inventarios de los usuarios ni de las ofertas del mercado: esas referencias quedan apuntando a una carta inexistente (el inventario las filtra al poblarlas).
