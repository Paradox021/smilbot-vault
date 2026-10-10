# 📋 Pendientes del Frontend

Cosas detectadas al construir el frontend que quedan **por resolver o revisar**. Cuando una se resuelva, se elimina de aquí y se refleja en la nota correspondiente (contrato del endpoint, modelo, etc.).

> El frontend no rodea lo que falta en la API: si una pantalla necesita un dato que no existe, se apunta aquí y se propone el cambio en el backend.

---

## 1. Necesidades del Backend

Endpoints o cambios que el frontend necesita y aún no existen o no están documentados.

| Necesidad | Endpoint | Impacto | Propuesta |
| :--- | :--- | :--- | :--- |
| **`createdAt` en stats** *(opcional)* | `GET /user/:id/stats` | El perfil no puede mostrar "Miembro desde", porque `/auth/me` queda solo para la identidad. | Añadir `createdAt` a la respuesta si se quiere ese dato. |

> ✅ Resueltas el 2026-10-10: racha efectiva en `/stats` (con `lastDaily`, `nextDailyAt` y `streakExpiresAt`) y avatar de Discord (`avatarUrl`). Ver [Endpoints Usuarios y Stats](../Backend%20API/Endpoints%20Usuarios%20y%20Stats.md).

---

## 2. Incoherencias en la Bóveda

*(ninguna abierta)* Las detectadas (ruta de la tirada, `Card.imageUrl` opcional, forma de `/auth/me`, `luck` sin tiradas y el ejemplo de stats) se corrigieron en sus notas el 2026-10-10 tras revisar el código del backend y del bot. Los fallos reales encontrados en esa revisión están en [Problemas Conocidos](../Backend%20API/Problemas%20Conocidos.md).

---

## 3. Pendientes Propios del Frontend

- **Dependencias del stack aún sin usar:** `motion`, `react-hook-form` y `@hookform/resolvers`. Se usarán en el mercado y en los formularios.
- **Página 404 sin carga diferida.** La regla de `lazy` se aplica a las páginas de las features; la 404 vive en `app/` y es mínima.

> ✅ Resueltos el 2026-10-10: volver a la ruta original tras el login (`returnTo`) y CSS de shadcn copiado a mano (ahora se importa del paquete). Ver [Arquitectura Frontend](./Arquitectura%20Frontend.md).

Véase también:
- [[Frontend/Arquitectura Frontend|Arquitectura Frontend]]
