# 💡 Propuesta: Colecciones de Cartas (Tags) 📚

> [!NOTE]
> **Estado:** 💡 *Idea Futura / Pendiente de Implementación*
> **Alcance (Scope):** 🌐 *Ecosistema Completo (Backend API + Bot Discord)*
> - 🔌 **Backend:** Campo `collections: string[]` en `Card`, nuevo modelo `Collection`, endpoints de progreso y reclamo de recompensa.
> - 🤖 **Bot Discord:** Comandos `.collections` / `.collection <nombre>` con barras de progreso, y soporte de tags en `.createcard`.
> **Aviso para Agentes:** Esta mecánica NO está en el código actual de producción.

---

## 1. Concepto
Agrupar las cartas en **colecciones temáticas** (ej. *Gatos Naranjas*, *Gatos Negros*, *Siameses*, *Gatos Dormilones*, *Halloween 2026*) para dar a los jugadores **metas de largo plazo** más allá de la rareza.

Una carta funciona con **tags**: puede pertenecer a **varias colecciones a la vez** (relación muchos a muchos). Por ejemplo, un gato negro con sombrero de bruja puede estar en *Gatos Negros* y en *Halloween 2026*.

---

## 2. Modelo de Datos Propuesto

### A. Tags en la carta (`Card`)
```typescript
interface CardSchema {
  // ...campos actuales (ver [[Modelos y Contratos/Cartas y Rarezas|Cartas y Rarezas]])
  collections: string[];   // Slugs de colecciones, ej. ['gatos-negros', 'halloween-2026'] (default: [])
}

cardSchema.index({ collections: 1 }); // Índice multikey para filtrar por colección
```
Guardar los tags en la carta (y no la lista de cartas en la colección) evita documentos que crezcan sin límite y hace trivial "¿a qué colecciones pertenece esta carta?".

### B. Metadatos de la colección (`Collection`)
```typescript
interface CollectionSchema {
  _id: ObjectId | string;
  slug: string;            // Identificador único (unique), ej. 'gatos-negros'
  name: string;            // Nombre visible, ej. 'Gatos Negros'
  description?: string;
  emoji?: string;          // Ej. '🐈‍⬛'
  isLimited?: boolean;     // Colecciones de evento (no se añaden cartas tras finalizar)
  reward?: {
    coins?: number;        // Recompensa única al completarla
    badge?: string;        // Insignia cosmética para el perfil
  };
  createdAt: Date;
  updatedAt: Date;
}
```

### C. Registro de colecciones completadas (`User`)
```typescript
interface UserSchema {
  // ...campos actuales
  completedCollections: { slug: string; completedAt: Date }[]; // Evita cobrar la recompensa dos veces
}
```

---

## 3. Cálculo del Progreso
- **Progreso** = cartas **distintas** de la colección que el usuario posee (`count > 0`) / total de cartas con ese tag.
- Las copias repetidas no suman: tener 5 copias de la misma carta cuenta como 1.
- Si existen [[Propuestas y Roadmap/Variantes Shiny|Variantes Shiny]], cualquier variante cuenta como poseída. Opcional a futuro: un logro extra de *"Colección Shiny"*.
- Se calcula al vuelo con una agregación (`User.cards` ⨝ `Card` filtrando por tag); no hace falta persistir el progreso.

### Casos borde
| Situación | Comportamiento propuesto |
| :--- | :--- |
| Se añade una carta nueva a una colección ya completada | El progreso vuelve a mostrar la carta faltante, pero la recompensa y la insignia ya obtenidas **se conservan**. |
| El usuario vende o intercambia una carta tras completar | La recompensa no se revoca (ya se registró en `completedCollections`). |
| Farmeo con cuentas alternativas pasándose cartas | Recompensas **modestas o cosméticas** (insignias) y reclamo **una sola vez por usuario**. |

---

## 4. Endpoints Propuestos
| Método | Ruta | Descripción |
| :---: | :--- | :--- |
| `GET` | `/collections` | Lista todas las colecciones con su número de cartas. |
| `GET` | `/collections/:slug` | Detalle de una colección y sus cartas. |
| `GET` | `/user/:id/collections` | Progreso del usuario en cada colección (`owned`, `total`, `completed`). |
| `POST` | `/user/:id/collections/:slug/claim` | Reclama la recompensa si la colección está completa (idempotente). |
| `PUT` | `/card/:id/collections` | *(Admin)* Asigna o modifica los tags de una carta. |
| `POST` | `/collections` | *(Admin)* Crea una colección. |

---

## 5. Experiencia en Discord
- **`.collections`**: lista paginada con barras de progreso:
  ```text
  🐈‍⬛ Gatos Negros      ▰▰▰▰▰▰▱▱▱▱  6/10
  🍊 Gatos Naranjas    ▰▰▰▰▰▰▰▰▰▰  8/8 ✅
  🎃 Halloween 2026    ▰▰▱▱▱▱▱▱▱▱  1/5
  ```
- **`.collection <nombre>`**: muestra las cartas obtenidas y las faltantes como `❔ ???` (silueta), para mantener la sorpresa.
- **Botón "Reclamar recompensa"** cuando la colección esté completa.
- **`.createcard`**: argumento opcional de tags al crear una carta.
- **`.show`**: muestra las colecciones a las que pertenece la carta.

---

## 6. Sinergias
- **Wishlist:** botón "Añadir faltantes a mi wishlist" desde `.collection`.
- **Banners de evento:** una colección limitada (ej. *Halloween 2026*) puede ir ligada a un banner con rate-up.
- **Mercado:** las cartas que faltan para completar colecciones populares ganan demanda de forma natural.
