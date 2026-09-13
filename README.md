# 🌌 Smilbot Vault - Documentación Centralizada

Esta bóveda de **Obsidian** es la **Única Fuente de Verdad (Single Source of Truth)** para la arquitectura, contratos de API REST, modelos de base de datos y diseño de sistemas del ecosistema **Smilbot**.

---

## 🔗 Repositorios Relacionados

- 🤖 **Bot de Discord:** [Paradox021/Smilbot](https://github.com/Paradox021/Smilbot) *(TypeScript / Discord.js v14)*
- 🔌 **Backend API:** *Repositorio del Backend API de Smilbot*

---

## 🗺️ Mapa de la Documentación

### 📐 [Modelos y Contratos](./Modelos%20y%20Contratos)
Estructuras de datos, esquemas compartidos y reglas de negocio:
- 🃏 [Cartas y Rarezas](./Modelos%20y%20Contratos/Cartas%20y%20Rarezas.md): Esquema `Card`, tabla de probabilidades (Common a Mythic) y colores.
- 💰 [Economía y Rachas](./Modelos%20y%20Contratos/Economia%20y%20Rachas.md): Balance, cooldowns de 23h-48h, rachas diarias y récord.
- 🍀 [Sistema de Suerte y Tiers](./Modelos%20y%20Contratos/Sistema de%20Suerte%20y%20Tiers.md): Fórmula neutralizada contra mercado, delta de suerte y tiers.
- 🏪 [Mercado y Transacciones](./Modelos%20y%20Contratos/Mercado%20y%20Transacciones.md): Ciclo de ofertas (`ACTIVE`, `SOLD`, `CANCELLED`) y libro mayor (*ledger*).
- 🔮 [Gacha, Pity y Variantes Shiny](./Modelos%20y%20Contratos/Gacha%20y%20Pity.md): Piedad a las 250 tiradas, cartas Shiny ✨ y reciclaje de duplicadas.

### 🔌 [Backend API](./Backend%20API)
Especificaciones y contratos de los endpoints REST:
- 👤 [Endpoints Usuarios y Stats](./Backend%20API/Endpoints%20Usuarios%20y%20Stats.md): `GET /user/:id`, `POST /user`, `GET /user/:id/stats`.
- 🃏 [Endpoints Cartas y Gacha](./Backend%20API/Endpoints%20Cartas.md): `GET /card`, `POST /card` (multipart), `POST /user/:id/card/random`.
- 🏪 [Endpoints Mercado](./Backend%20API/Endpoints%20Mercado.md): `GET /market/:id/offers`, `POST /market/:id/offers/:id/buy`.
- 🏆 [Endpoints Leaderboard](./Backend%20API/Endpoints%20Leaderboard.md): `GET /leaderboard/luck` (rankings desc/asc).

### 🤖 [Bot Discord](./Bot%20Discord)
Diseño de la aplicación cliente en Discord.js:
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
Esta bóveda está configurada como servidor MCP vía `@modelcontextprotocol/server-filesystem`, permitiendo a los asistentes de IA de desarrollo leer y actualizar la documentación automáticamente cada vez que se implementan cambios.
