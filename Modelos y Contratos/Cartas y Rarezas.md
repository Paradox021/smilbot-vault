# 🃏 Cartas y Rarezas (Especificación Real del Backend)

Documento de referencia para el modelo de datos de cartas y el sistema de probabilidades exacto implementado en el backend.

---

## 1. Esquema de Carta (Card Model / Mongoose)

```typescript
interface CardSchema {
  _id: ObjectId | string;
  name: string;             // Nombre único de la carta
  description: string;      // Descripción o lore
  type: number;             // 0: Common, 1: Rare, 2: Epic, 3: Legendary, 4: Mythic
  imageUrl: string;         // URL pública de la imagen
  imagePublicId?: string;   // ID del proveedor de imágenes (para borrado remoto)
  author?: string;          // Creador o artista de la carta
  createdAt: Date;
  updatedAt: Date;
}
```

---

## 2. Tabla de Rarezas y Probabilidades del Gacha

El backend utiliza un generador criptográficamente seguro (`crypto.randomInt(0, 1000)`) que asigna un roll entero entre **0** y **999**:

| Tipo (`type`) | Rareza | Probabilidad | Rango de Tirada (`roll`) | Color Representativo |
| :---: | :--- | :---: | :---: | :--- |
| **4** | **Mythic** (Mítica) | **0.5%** | `0 – 4` | 🔴 Rojo / Arcoíris (`#c45039`) |
| **3** | **Legendary** (Legendaria) | **2.0%** | `5 – 24` | 🟠 Dorado / Amarillo (`#ff8000`) |
| **2** | **Epic** (Épica) | **10.0%** | `25 – 124` | 🟣 Púrpura / Morado (`#a335ee`) |
| **1** | **Rare** (Rara) | **30.0%** | `125 – 424` | 🔵 Azul (`#0070dd`) |
| **0** | **Common** (Común) | **57.5%** | `425 – 999` | ⚪ Gris / Blanco (`#808080`) |

*Suma total de probabilidades: $0.5\% + 2.0\% + 10.0\% + 30.0\% + 57.5\% = 100.0\%$.*

### Algoritmo de Tirada (`getRandomCard`)
1. Genera un número entero $roll \in [0, 999]$.
2. Determina el `type` de rareza según el rango alcanzado.
3. Selecciona una carta aleatoria entre todas las cartas existentes de esa rareza en la base de datos.
4. Retorna la carta seleccionada junto con el valor exacto del `roll`.

Véase también:
- [[Modelos y Contratos/Gacha y Pity|Gacha y Pity]]
- [[Backend API/Endpoints Cartas|Endpoints Cartas]]
