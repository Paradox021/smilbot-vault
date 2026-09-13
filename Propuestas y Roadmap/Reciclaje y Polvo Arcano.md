# 💡 Propuesta: Reciclaje de Duplicadas y Polvo Arcano

> [!NOTE]
> **Estado:** 💡 *En Exploración / Pendiente de Implementación*
> **Aviso para Agentes:** Esta mecánica NO está en el código actual de producción. Representa el diseño técnico acordado para su futura integración.

---

## 1. Concepto y Regla Anticolapso del Mercado
**El problema:** Con el tiempo, los jugadores acumulan decenas de cartas Comunes repetidas que nadie compra en el mercado libre.
**La solución:** Permitir desencantar o reciclar cartas duplicadas a cambio de **Polvo Arcano** (`dust`).

### 🚨 Regla de Oro de Diseño:
> **El Polvo Arcano NUNCA debe permitir comprar cartas directas ni fabricar Míticas.**
> Si permitiéramos canjear polvo por cartas específicas o fusionar hacia arriba sin límite, se fijaría un precio techo/suelo para las cartas raras y se destruiría el mercado libre.

---

## 2. Mecánica de Reciclaje (`.recycle` / `.scrap`)
El usuario selecciona cartas sobrantes de su inventario para destruirlas a cambio de polvo según su rareza:

| Rareza Desencantada | Polvo Obtenido Sugerido |
| :--- | :---: |
| **Common** | 5 de Polvo |
| **Rare** | 20 de Polvo |
| **Epic** | 75 de Polvo |
| **Legendary** | 250 de Polvo |
| **Mythic** | 1,000 de Polvo |

---

## 3. Tienda de Consumibles de Polvo
El polvo se utiliza exclusivamente en una tienda de objetos consumibles de soporte:

1. 🧪 **Amuleto de Fortuna (Shiny Charm):**
   - *Coste sugerido:* 200 de Polvo.
   - *Efecto:* Aumenta la probabilidad de Shiny de tu siguiente sobre (de 2% a 6%).
2. ⏩ **Acelerador de Piedad (Pity Booster):**
   - *Coste sugerido:* 300 de Polvo.
   - *Efecto:* Añade directamente **+5 tiradas** al contador de Piedad Mítica (`pityCount`) sin entregar cartas intermedias.
3. 🛡️ **Gema Protectora de Racha (Streak Shield):**
   - *Coste sugerido:* 150 de Polvo.
   - *Efecto:* Si transcurren más de 48 horas sin reclamar el `dailybalance`, el escudo se consume y rescata tu racha histórica intacta.
