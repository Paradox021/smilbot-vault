# 🛠️ Scripts, Mantenimiento y Migraciones (Backend)

Documentación de los scripts de mantenimiento y tareas administrativas en `D:\codigos\backendSmilbot\scripts\`.

---

## 1. Migración y Backfill de Telemetría (`scripts/backfillTelemetry.js`)

Calcula y rellena las estadísticas históricas de los usuarios (`totalCoinsEarned`, `cardsOpenedCount`, etc.) basándose en sus balances e inventarios existentes:

```bash
# Modo simulación (Dry Run): muestra lo que cambiaría sin tocar MongoDB
npm run backfill:dry

# Modo ejecución real: aplica los cambios en la base de datos
npm run backfill
```

---

## 2. Migración de Imágenes entre Proveedores (`scripts/migrateImages.js`)

Permite migrar de forma masiva los archivos de imagen de las cartas de un proveedor a otro sin perder enlaces ni nombres:

```bash
# Ejemplo: Migrar de almacenamiento local a Supabase Storage
node scripts/migrateImages.js --from=local --to=supabase

# Ejemplo: Migrar de Cloudinary a Supabase
node scripts/migrateImages.js --from=cloudinary --to=supabase
```

---

## 3. Comandos de Ejecución del Servidor

```bash
# Desarrollo con recarga automática (nodemon)
npm run dev

# Inicio estándar en producción
npm run start
```
