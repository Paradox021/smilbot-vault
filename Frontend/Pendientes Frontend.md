# 📋 Pendientes del Frontend

Cosas detectadas al construir el frontend que quedan **por resolver o revisar**. Cuando una se resuelva, se elimina de aquí y se refleja en la nota correspondiente (contrato del endpoint, modelo, etc.).

> El frontend no rodea lo que falta en la API: si una pantalla necesita un dato que no existe, se apunta aquí y se propone el cambio en el backend.

---

## 1. Necesidades del Backend

Endpoints o cambios que el frontend necesita y aún no existen o no están documentados.

| Necesidad | Endpoint | Impacto | Propuesta |
| :--- | :--- | :--- | :--- |
| **Racha efectiva** | `GET /user/:id/stats` | `dailyStreak` solo se recalcula al reclamar el daily ([Economía y Rachas](../Modelos%20y%20Contratos/Economia%20y%20Rachas.md)). Si han pasado más de 48 h desde `lastDaily`, el perfil muestra una racha que ya está rota, y `/stats` no incluye `lastDaily` para detectarlo. | Devolver la racha efectiva (0 si está rota) o, como mínimo, `lastDaily` + `streakActive: boolean`. Para la futura pantalla de daily, añadir también `nextDailyAt` (fin del cooldown de 23 h). |
| **Documentar `/auth/me`** | `GET /auth/me` | [Autenticación](../Backend%20API/Autenticacion.md) solo dice "perfil + `isAdmin`". El frontend lo usa solo para la identidad y asume `{ discordId, username, isAdmin }` en la raíz. | Confirmar la forma exacta (¿plano o `{ user, isAdmin }`?) y escribirla en la nota de Autenticación. |
| **Documentar `luck` sin tiradas** | `GET /user/:id/stats` | No consta qué devuelven `luckPercentage`, `tier` y `tierCode` cuando `totalCards` es `0`. El frontend muestra un estado vacío y no lee esos campos. | Documentar el caso en [Endpoints Usuarios y Stats](../Backend%20API/Endpoints%20Usuarios%20y%20Stats.md). |
| **`createdAt` en stats** *(opcional)* | `GET /user/:id/stats` | El perfil no puede mostrar "Miembro desde", porque `/auth/me` queda solo para la identidad. | Añadir `createdAt` a la respuesta si se quiere ese dato. |
| **Avatar de Discord** *(opcional)* | `GET /auth/me` | El login usa `scope=identify`, que da acceso al avatar, pero no se guarda ni se devuelve. | Guardar el hash del avatar y devolverlo (o la URL completa del CDN de Discord). |

---

## 2. Incoherencias en la Bóveda

| Tema | Detalle | Acción |
| :--- | :--- | :--- |
| Ruta de la tirada | `POST /user/:id/card/random` en [Endpoints Usuarios y Stats](../Backend%20API/Endpoints%20Usuarios%20y%20Stats.md) y [README](../README.md), pero `POST /user/:id/cards/roll` en [Gacha y Pity](../Modelos%20y%20Contratos/Gacha%20y%20Pity.md). | Confirmar cuál es la real en el backend y corregir la otra. |
| `Card.imageUrl` | El esquema de [Cartas y Rarezas](../Modelos%20y%20Contratos/Cartas%20y%20Rarezas.md) lo marca como `string` obligatorio, pero el `AGENTS.md` del frontend dice que puede faltar. | Aclarar si puede faltar. El frontend usará una imagen de reserva igualmente. |
| Ejemplo de stats | En el ejemplo de `GET /user/:id/stats`, `luck.totalCards` (35) es mayor que `cardsOpenedCount` (30). Puede ser correcto por las ventas en el mercado, pero confunde. | Revisar el ejemplo o explicar la relación entre ambos campos. |

---

## 3. Pendientes Propios del Frontend

- **Volver a la ruta original tras el login.** El backend siempre redirige a la raíz de `FRONTEND_URL`, así que se pierde la página privada que el usuario intentaba abrir. Se podría guardar la ruta antes de ir a `/auth/discord` (no es un dato sensible) o que el backend acepte un parámetro de retorno validado.
- **Dependencias del stack aún sin usar:** `motion`, `react-hook-form` y `@hookform/resolvers`. Se usarán en el mercado y en los formularios.
- **Página 404 sin carga diferida.** La regla de `lazy` se aplica a las páginas de las features; la 404 vive en `app/` y es mínima.
- **CSS de shadcn copiado a mano** (`src/shared/ui/shadcn.css`) para no depender del paquete de la CLI. Si shadcn lo actualiza, hay que volver a copiarlo.

Véase también:
- [[Frontend/Arquitectura Frontend|Arquitectura Frontend]]
