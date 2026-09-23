# 🎰 Gacha y Sistema de Pity (Piedad)

El sistema de **Pity (Piedad)** es una red de seguridad determinista que garantiza que los usuarios no sufran rachas desmedidas de mala suerte al tirar por cartas Míticas (`type: 4`).

---

## 1. Reglas Fundamentales

- **Umbral de Activación:** **250 tiradas** consecutivas sin obtener una carta Mítica.
- **Tirada Garantizada (`pityCount >= 250`):**
  - El backend omite el roll aleatorio usual y selecciona directamente una carta Mítica (`type: 4`) al azar entre el catálogo disponible.
  - La respuesta marca `isPity: true` y `roll: null`.
  - Incrementa `pityMythicsCount += 1`.
  - Reinicia `pityCount = 0`.
- **Tirada Regular (`pityCount < 250`):**
  - Si el roll RNG produce una Mítica natural (`roll < 5`): se entrega la carta, `isPity: false` y `pityCount` se reinicia a `0`.
  - Si el roll produce cualquier otra rareza (Común, Rara, Épica, Legendaria): se entrega la carta, `isPity: false` y `pityCount` se incrementa en `+1`.

---

## 2. Persistencia en Base de Datos

### Modelo `User` (`models/user.js`)
```typescript
interface UserSchema {
  // ... campos previos
  pityCount: number;         // Racha actual de tiradas consecutivas sin mítica (default: 0, min: 0)
  pityMythicsCount: number;  // Total acumulado histórico de míticas obtenidas por pity (default: 0, min: 0)
}
```

### Modelo `Transaction` (`models/transaction.js`)
En cada compra de carta (`CARD_BUY`), el objeto `metadata` almacena la auditoría:
```typescript
metadata: {
  cardId: ObjectId;
  cardType: number;
  cardName: string;
  roll: number | null;
  isPity: boolean;
  pityCountBefore: number;
  pityCountAfter: number;
}
```

---

## 3. Neutralización en el Cálculo de Suerte

Las cartas míticas obtenidas por pity **no deben inflar la suerte** del jugador ni su ausencia de azar debe castigarle con una tirada de 0 puntos:
- Se descuenta `user.pityMythicsCount` tanto de las míticas efectivas (`breakdown.mythic`) como de las tiradas totales evaluadas (`totalCards`).
- De este modo, el ranking de suerte evalúa de forma 100% pura y neutral únicamente las tiradas verdaderamente aleatorias.

---

## 4. Endpoints y Payloads

- **Tirada:** `POST /user/:id/cards/roll`
  Retorna la carta junto con `roll`, `isPity` y el objeto de progreso `pity: { current, threshold: 250, remaining }`.
- **Estadísticas:** `GET /user/:discordId/stats`
  Incluye el bloque `pity: { pullsSinceLastMythic, pityThreshold: 250, pullsUntilGuaranteed, pityMythicsCount }`.

Véase también:
- [[Backend API/Endpoints Cartas|Endpoints Cartas]]
- [[Modelos y Contratos/Cartas y Rarezas|Cartas y Rarezas]]
- [[Modelos y Contratos/Sistema de Suerte y Tiers|Sistema de Suerte y Tiers]]
