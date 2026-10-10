# 🛠️ Scripts, Mantenimiento y Migraciones (Backend)

Scripts de mantenimiento en la carpeta `scripts/` del repo del backend. Todos leen `URL_DB` del `.env`.

---

## 1. Comandos del Servidor

```bash
# Desarrollo con recarga automática (nodemon)
npm run dev

# Inicio estándar en producción
npm run start
```

---

## 2. Backfill de Estadísticas y Ledger (`scripts/backfillStatsAndLedger.js`)

Migración de datos antiguos:
- Calcula y rellena las estadísticas históricas de los usuarios (`totalCoinsEarned`, `cardsOpenedCount`, etc.) a partir de sus balances e inventarios.
- Normaliza el inventario al formato `[{ cardId, count }]`.
- Migra las ofertas del antiguo modelo `Market` (embebidas) a la colección independiente `market_offers`.

```bash
# Simulación (dry run): muestra lo que cambiaría sin tocar MongoDB
npm run backfill:dry

# Ejecución real
npm run backfill
```

---

## 3. Backfill de Pity (`scripts/backfillPity.js`)

Calcula el contador de tiradas sin mítica de cada usuario combinando el volcado cronológico original (`test.users.json`) con las transacciones `CARD_BUY` posteriores del Ledger.

```bash
# Simulación sin persistir cambios
npm run backfill:pity:dry

# Ejecución real
npm run backfill:pity
```

---

## 4. Migración de Imágenes a la Nube (`scripts/migrateImages.js`)

Sube a la nube las imágenes locales (`storage/images/`) de las cartas y actualiza `imageUrl` e `imagePublicId`. Solo migra **de local al proveedor configurado**; no acepta parámetros de origen y destino.

```bash
# 1. En .env: IMAGE_PROVIDER=cloudinary (o supabase) y sus credenciales
# 2. Ejecutar
node scripts/migrateImages.js
```

- Salta las cartas que ya tienen `imagePublicId` (ya migradas), así que se puede ejecutar varias veces.
- No tiene script en `package.json` ni modo dry run.
