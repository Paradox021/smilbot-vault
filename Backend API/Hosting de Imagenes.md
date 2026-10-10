# 🖼️ Arquitectura: Hosting y Almacenamiento de Imágenes

El backend implementa una capa de abstracción con el patrón **Provider / Factory** (`libs/imageHosting/`) expuesta como `imageService`. La subida de ficheros la gestiona Multer (`libs/storage.js`): en disco para `local` y en memoria para los proveedores en la nube.

---

## 1. Proveedores Soportados (`IMAGE_PROVIDER`)

| Proveedor | Valor en `.env` | Destino | Variables |
| :--- | :--- | :--- | :--- |
| **Local** (default) | `IMAGE_PROVIDER=local` | `storage/images/`, servido en `/public/` | `IMAGE_HOST` (opcional; default `http://localhost:<PORT>`) |
| **Cloudinary** | `IMAGE_PROVIDER=cloudinary` | Carpeta remota (default `smilbot`) | `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET`, `CLOUDINARY_FOLDER` (opcional) |
| **Supabase Storage** | `IMAGE_PROVIDER=supabase` | Bucket de Supabase (default `smilbot-images`) | `SUPABASE_URL`, `SUPABASE_KEY`, `SUPABASE_BUCKET` (opcional) |

- `imageUrl` siempre se guarda como **URL absoluta** (en `local`, `IMAGE_HOST + /public/<publicId>`). Si cambias `IMAGE_HOST`, las cartas ya creadas conservan la URL antigua.

### Nombres de fichero

| Proveedor | `publicId` / nombre guardado |
| :--- | :--- |
| **Local** | `<Date.now()>-<8 hex aleatorios>-<nombre original saneado>` (`uniqueImageFilename` en `localProvider.js`). Del nombre original se toma solo el *basename* y cualquier carácter fuera de `[A-Za-z0-9._-]` se sustituye por `_`. Dos imágenes con el mismo nombre ya no se sobrescriben. |
| **Supabase** | `cards/<Date.now()>-<originalname>`, sin sanear y con `upsert: false`. Ver [Problemas Conocidos](./Problemas%20Conocidos.md). |
| **Cloudinary** | Lo genera Cloudinary dentro de `CLOUDINARY_FOLDER`. |

- Las imágenes locales guardadas antes de este cambio conservan su nombre original y su URL.
- Si la creación de una carta falla después de guardar la imagen, el backend la borra (`POST /card`, ver [Endpoints Cartas](./Endpoints%20Cartas.md)).

---

## 2. Interfaz del Servicio (`imageService`)
- `upload(file)`: sube la imagen y retorna `{ url, publicId }`. Se guardan en la carta como `imageUrl` e `imagePublicId`.
- `delete(publicId)`: elimina la imagen del proveedor cuando se borra la carta.
- `getUrl(publicId)`: URL pública a partir del `publicId`.

Para pasar las imágenes locales a la nube, ver `scripts/migrateImages.js` en [Scripts y Mantenimiento](./Scripts%20y%20Mantenimiento.md).
