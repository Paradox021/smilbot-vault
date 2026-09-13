# 🖼️ Arquitectura: Hosting y Almacenamiento de Imágenes

El backend implementa una capa de abstracción con el patrón **Provider / Factory** (`libs/imageHosting/`) gestionada por `imageService`.

---

## 1. Proveedores Soportados (`IMAGE_PROVIDER`)

| Proveedor | Valor en `.env` | Destino | Variables Requeridas |
| :--- | :--- | :--- | :--- |
| **Local** | `IMAGE_PROVIDER=local` | `storage/images/` | Ninguna adicional (servido en `/public/`). |
| **Cloudinary** | `IMAGE_PROVIDER=cloudinary` | Carpeta remota `smilbot/` | `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` |
| **Supabase Storage**| `IMAGE_PROVIDER=supabase` | Bucket de Supabase | `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_BUCKET_NAME` |

---

## 2. Interfaz del Servicio (`imageService`)
- `upload(file)`: Sube la imagen y retorna `{ url, publicId }`.
- `delete(publicId)`: Elimina la imagen del proveedor remoto cuando se borra la carta.
