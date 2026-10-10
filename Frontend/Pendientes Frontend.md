# 📋 Pendientes del Frontend

Cosas detectadas al construir el frontend que quedan **por resolver o revisar**. Cuando una se resuelva, se elimina de aquí y se refleja en la nota correspondiente (contrato del endpoint, modelo, etc.).

> El frontend no rodea lo que falta en la API: si una pantalla necesita un dato que no existe, se apunta aquí y se propone el cambio en el backend.

---

## 1. Necesidades del Backend

Endpoints o cambios que el frontend necesita y aún no existen o no están documentados.

| Necesidad | Endpoint | Impacto | Propuesta |
| :--- | :--- | :--- | :--- |
| **Racha efectiva** | `GET /user/:id/stats` | `dailyStreak` solo se recalcula al reclamar el daily ([Economía y Rachas](../Modelos%20y%20Contratos/Economia%20y%20Rachas.md)). Si han pasado más de 48 h desde `lastDaily`, el perfil muestra una racha que ya está rota, y `/stats` no incluye `lastDaily` para detectarlo. | Devolver la racha efectiva (0 si está rota) o, como mínimo, `lastDaily` + `streakActive: boolean`. Para la futura pantalla de daily, añadir también `nextDailyAt` (fin del cooldown de 23 h). |
| **`createdAt` en stats** *(opcional)* | `GET /user/:id/stats` | El perfil no puede mostrar "Miembro desde", porque `/auth/me` queda solo para la identidad. | Añadir `createdAt` a la respuesta si se quiere ese dato. |
| **Avatar de Discord** *(opcional)* | `GET /auth/me` | El login usa `scope=identify`, que da acceso al avatar, pero no se guarda ni se devuelve. | Guardar el hash del avatar y devolverlo (o la URL completa del CDN de Discord). |

---

## 2. Incoherencias en la Bóveda

*(ninguna abierta)* Las detectadas (ruta de la tirada, `Card.imageUrl` opcional, forma de `/auth/me`, `luck` sin tiradas y el ejemplo de stats) se corrigieron en sus notas el 2026-10-10 tras revisar el código del backend y del bot. Los fallos reales encontrados en esa revisión están en [Problemas Conocidos](../Backend%20API/Problemas%20Conocidos.md).

---

## 3. Pendientes Propios del Frontend

- **Volver a la ruta original tras el login.** El backend siempre redirige a la raíz de `FRONTEND_URL`, así que se pierde la página privada que el usuario intentaba abrir. Se podría guardar la ruta antes de ir a `/auth/discord` (no es un dato sensible) o que el backend acepte un parámetro de retorno validado.
- **Dependencias del stack aún sin usar:** `motion`, `react-hook-form` y `@hookform/resolvers`. Se usarán en el mercado y en los formularios.
- **Página 404 sin carga diferida.** La regla de `lazy` se aplica a las páginas de las features; la 404 vive en `app/` y es mínima.
- **CSS de shadcn copiado a mano** (`src/shared/ui/shadcn.css`) para no depender del paquete de la CLI. Si shadcn lo actualiza, hay que volver a copiarlo.

Véase también:
- [[Frontend/Arquitectura Frontend|Arquitectura Frontend]]
