# 🃏 Cartas y Rarezas

Documento de referencia para el modelo de datos de cartas y el sistema de probabilidades.

---

## 1. Esquema de Carta (Card Model)

```typescript
export interface Card {
  _id: string;
  name: string;
  type: CardType;        // Número entero (0 a 4)
  description: string;
  author: string;        // Discord tag o nombre del creador
  imageUrl: string;      // URL de la imagen alojada
}
```

---

## 2. Tabla de Rarezas y Probabilidades

La tirada gacha estándar ejecuta un roll entero entre **0 y 999**:

| Rareza | Type ID | Rango de Roll | Probabilidad | Color Hex | Color Int |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Common** | `0` | 0 – 599 | **60.0%** | `#808080` | `0x808080` |
| **Rare** | `1` | 600 – 849 | **25.0%** | `#0070dd` | `0x0070dd` |
| **Epic** | `2` | 850 – 949 | **10.0%** | `#a335ee` | `0xa335ee` |
| **Legendary** | `3` | 950 – 989 | **4.0%** | `#ff8000` | `0xff8000` |
| **Mythic** | `4` | 990 – 999 | **1.0%** | `#c45039` | `0xc45039` |

---

## 3. Notas de Diseño
- El nombre de las cartas puede contener espacios y caracteres especiales.
- La creación administrativa (`.createcard`) valida que la rareza esté dentro del rango permitido antes de enviarla a `POST /card`.

Véase también:
- [[Modelos y Contratos/Gacha y Pity|Gacha y Pity]]
- [[Backend API/Endpoints Cartas|Endpoints Cartas]]
