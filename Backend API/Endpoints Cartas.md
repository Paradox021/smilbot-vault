# 🃏 Endpoints: Cartas y Gacha

---

## 1. Catálogo de Todas las Cartas
- **Ruta:** `GET /card`
- **Respuesta:** Array `Card[]`

---

## 2. Apertura de Sobre Gacha
- **Ruta:** `POST /user/:discordId/card/random`
- **Requisitos:** El usuario debe tener al menos 100 monedas.
- **Acción:** Descuenta 100 monedas, realiza la tirada RNG, guarda la carta en la colección del usuario y registra la transacción `CARD_BUY`.
- **Respuesta:** Objeto `Card` obtenido.

---

## 3. Creación Administrativa de Cartas
- **Ruta:** `POST /card`
- **Formato:** `multipart/form-data`
  - Campo `image`: Archivo binario de imagen (PNG/JPG).
  - Campo `data`: JSON stringificado:
    ```json
    {
      "name": "Nombre",
      "type": "mythic",
      "description": "Lore",
      "author": "Creador#1234"
    }
    ```
- **Respuesta:** Objeto `Card` recién creado.
