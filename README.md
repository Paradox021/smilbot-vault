# 🌌 Smilbot Vault - Documentación Centralizada

Esta bóveda de **Obsidian** es la **Única Fuente de Verdad (Single Source of Truth)** para la arquitectura, contratos de API REST, modelos de base de datos y diseño de sistemas del ecosistema **Smilbot**.

---

## 🔗 Repositorios Relacionados

- 🤖 **Bot de Discord:** [Paradox021/Smilbot](https://github.com/Paradox021/Smilbot) *(TypeScript / Discord.js v14)*
- 🔌 **Backend API:** [Paradox021/backendSmilbot](https://github.com/Paradox021/backendSmilbot) *(Node.js / Express / MongoDB)*

---

## 🗺️ Mapa de la Documentación

### 📐 [Modelos y Contratos](./Modelos%20y%20Contratos)
- 🃏 [Cartas y Rarezas](./Modelos%20y%20Contratos/Cartas%20y%20Rarezas.md): Esquema `Card`, tabla de probabilidades reales (Mythic 0.5%, Legendary 2%, Epic 10%, Rare 30%, Common 57.5%).
- 💰 [Economía y Rachas](./Modelos%20y%20Contratos/Economia%20y%20Rachas.md): Balance, cooldowns de 23h-48h, rachas diarias y récord.
- 🍀 [Sistema de Suerte y Tiers](./Modelos%20y%20Contratos/Sistema%20de%20Suerte%20y%20Tiers.md): Fórmula neutralizada contra mercado, delta de suerte y tiers.
- 🏪 [Mercado y Transacciones](./Modelos%20y%20Contratos/Mercado%20y%20Transacciones.md): Ciclo de ofertas (`ACTIVE`, `SOLD`, `CANCELLED`) y libro mayor (*ledger*).

### 🔌 [Backend API](./Backend%20API)
- 🛠️ [Scripts y Mantenimiento](./Backend%20API/Scripts%20y%20Mantenimiento.md): Backfill de telemetría y migración de imágenes.
- 👤 [Endpoints Usuarios y Stats](./Backend%20API/Endpoints%20Usuarios%20y%20Stats.md): `GET /user/:id`, `POST /user`, `POST /dailyBalance`, `GET /transactions`.
- 🃏 [Endpoints Cartas y Gacha](./Backend%20API/Endpoints%20Cartas.md): `GET /card`, `GET /card/:rarity`, `POST /card` (multipart), `POST /user/:id/card/random`.
- 🏪 [Endpoints Mercado](./Backend%20API/Endpoints%20Mercado.md): `GET /offers`, `POST /offers` (publicar), `POST /buy`, `POST /cancel`.
- 🏆 [Endpoints Leaderboard](./Backend%20API/Endpoints%20Leaderboard.md): Rankings de suerte, rachas (`streaks`), riqueza (`wealth`) y coleccionistas (`cards`).
- 🖼️ [Hosting de Imágenes](./Backend%20API/Hosting%20de%20Imagenes.md): Abstracción Local, Cloudinary y Supabase Storage.

### 💡 [Propuestas y Roadmap](./Propuestas%20y%20Roadmap)
Diseños conceptuales y futuras mecánicas en fase de exploración:
- 🎯 [Sistema de Piedad (Pity)](./Propuestas%20y%20Roadmap/Sistema%20de%20Piedad%20(Pity).md): Garantía de carta Mítica a las 250 tiradas y control de mala suerte.
- ✨ [Variantes Shiny](./Propuestas%20y%20Roadmap/Variantes%20Shiny.md): Variante cosmética brillante ultra-rara (2%-3% de aparición).
- 🧪 [Reciclaje y Polvo Arcano](./Propuestas%20y%20Roadmap/Reciclaje%20y%20Polvo%20Arcano.md): Destrucción de duplicadas por polvo para comprar consumibles y amuletos.

### 🤖 [Bot Discord](./Bot%20Discord)
- 🏛️ [Arquitectura y Eventos](./Bot%20Discord/Arquitectura%20y%20Eventos.md): Lifecycle, eventos y middlewares (`checkUser`, `checkAdmin`).
- 🎨 [Componentes de UI](./Bot%20Discord/Componentes%20de%20UI.md): Embeds interactivos, botones de navegación y paginación protegida.
- 🪙 [Comandos de Economía](./Bot%20Discord/Comandos%20de%20Economia.md): Referencia y alias (`.balance`, `.getcard`, `.market`, etc.).
- 🎵 [Comandos de Música](./Bot%20Discord/Comandos%20de%20Musica.md): `.play`, `.stop` con `discord-player`.
- 🛠️ [Comandos de Utilidad](./Bot%20Discord/Comandos%20de%20Utilidad.md): `.help`, `.ping`.

---

## 🛠️ Cómo Usar esta Bóveda

### En Obsidian
1. Abre Obsidian.
2. Selecciona **"Abrir carpeta como bóveda"** (*Open folder as vault*).
3. Selecciona la raíz de este repositorio.

### Con Agentes de IA (MCP)
Esta bóveda está conectada vía MCP (`@modelcontextprotocol/server-filesystem`), permitiendo a los asistentes de IA de desarrollo leer y actualizar la documentación automáticamente en ambos repositorios.
