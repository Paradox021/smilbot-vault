# 🐞 Problemas Conocidos (Backend y Bot)

Fallos reales detectados al contrastar la bóveda con el código del backend y del bot (2026-10-10). No son errores de documentación: el comportamiento descrito en las notas es el real. Cuando se arregle uno, se quita de aquí y se actualiza la nota afectada.

> ✅ Resueltos el 2026-10-10: compra del mercado no atómica, mercado del bot roto (compra imposible, campos incorrectos, sin publicar ni cancelar), validación al publicar, errores del mercado como `500` y datos completos del vendedor en el listado. Ver [Endpoints Mercado](./Endpoints%20Mercado.md).
>
> ✅ Resueltos el 2026-10-10 (segunda tanda): racha mostrada desactualizada ([Economía y Rachas](../Modelos%20y%20Contratos/Economia%20y%20Rachas.md)), referencias huérfanas al borrar una carta ([Endpoints Cartas](./Endpoints%20Cartas.md)), `500` en `GET /user/:id/stats` con usuario inexistente, `200 null` en `GET /card/:id`, `DELETE /user/:id` con `_id` de MongoDB, errores de cartas como `500`, imágenes locales sobrescritas con el mismo nombre ([Hosting de Imágenes](./Hosting%20de%20Imagenes.md)) y comentario de `metadata` desactualizado en `models/transaction.js`.

---

## 🟠 Prioridad Media

*(ninguno abierto)*

## 🟡 Prioridad Baja

| Problema | Dónde | Efecto |
| :--- | :--- | :--- |
| Nombres de imagen en Supabase sin sanear | `supabaseProvider.upload` | La ruta es `cards/<Date.now()>-<originalname>`: el nombre original no se sanea (espacios y caracteres especiales acaban en la ruta y la URL) y, con `upsert: false`, dos subidas con el mismo nombre en el mismo milisegundo hacen fallar la segunda. No llega a sobrescribir. El proveedor local ya usa `uniqueImageFilename`. |
