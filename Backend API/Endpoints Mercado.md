# 🏪 Endpoints: Mercado Libre

---

## 1. Listar Ofertas Activas
- **Ruta:** `GET /market/:serverId/offers`
- **Parámetros URL:** `serverId` (ID del servidor de Discord)
- **Respuesta:** Array de ofertas activas con carta poblada (`MarketOfferWithCard[]`):
```json
[
  {
    "_id": "offer_123",
    "card": {
      "_id": "card_abc",
      "name": "Dark Magician",
      "type": 3,
      "imageUrl": "https://..."
    },
    "sellerId": "111222333",
    "price": 450,
    "serverId": "999888777",
    "status": "ACTIVE",
    "createdAt": "2026-09-10T14:20:00.000Z"
  }
]
```

---

## 2. Comprar Oferta
- **Ruta:** `POST /market/:serverId/offers/:offerId/buy`
- **Body:** `{ "buyerId": string }`
- **Validaciones:**
  - Comprador no puede ser vendedor.
  - Comprador debe tener saldo suficiente.
  - La oferta debe seguir en estado `ACTIVE`.
- **Respuesta:** `{ "ok": true, "message": "Card bought successfully" }`
