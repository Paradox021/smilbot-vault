# 🌌 Smilbot Vault - Documentación Centralizada

Esta bóveda de **Obsidian** es la **Única Fuente de Verdad (Single Source of Truth)** para la arquitectura, contratos de API REST, modelos de base de datos y diseño de sistemas del ecosistema **Smilbot**.

---

## 🔗 Repositorios Relacionados

- 🤖 **Bot de Discord:** [Paradox021/smilbot](https://github.com/Paradox021/smilbot) *(TypeScript / Discord.js v14)*
- 🔌 **Backend API:** [Paradox021/smilbot-backend](https://github.com/Paradox021/smilbot-backend) *(Node.js / Express / MongoDB)*
- 🌐 **Frontend Web:** `smilbot-project/smilbot-frontend` *(React / TypeScript / Vite)*

---

## 🗺️ Mapa de la Documentación

### 📐 [Modelos y Contratos](./Modelos%20y%20Contratos)
- 🃏 [Cartas y Rarezas](./Modelos%20y%20Contratos/Cartas%20y%20Rarezas.md): Esquema `Card`, tabla de probabilidades reales (Mythic 0.5%, Legendary 2%, Epic 10%, Rare 30%, Common 57.5%).
- 💰 [Economía y Rachas](./Modelos%20y%20Contratos/Economia%20y%20Rachas.md): Balance, cooldowns de 23h-48h, rachas diarias, racha efectiva y récord.
- 🍀 [Sistema de Suerte y Tiers](./Modelos%20y%20Contratos/Sistema%20de%20Suerte%20y%20Tiers.md): Fórmula neutralizada contra mercado, delta de suerte y tiers.
- 🏪 [Mercado y Transacciones](./Modelos%20y%20Contratos/Mercado%20y%20Transacciones.md): Ciclo de ofertas (`ACTIVE`, `SOLD`, `CANCELLED`) y libro mayor (*ledger*) con `metadata` por tipo.

### 🔌 [Backend API](./Backend%20API)
- 🔐 [Autenticación](./Backend%20API/Autenticacion.md): API key del bot (`Authorization: Bot`), JWT de usuario, login OAuth2 con Discord (`returnTo`) y matriz de permisos.
- 🛠️ [Scripts y Mantenimiento](./Backend%20API/Scripts%20y%20Mantenimiento.md): Backfills de estadísticas, Ledger y pity, y migración de imágenes a la nube.
- 👤 [Endpoints Usuarios y Stats](./Backend%20API/Endpoints%20Usuarios%20y%20Stats.md): `GET /user/:id`, `POST /user`, `POST /dailyBalance`, `GET /stats` (racha efectiva, `avatarUrl`), `GET /transactions`, `PATCH /profile` (solo bot).
- 🃏 [Endpoints Cartas y Gacha](./Backend%20API/Endpoints%20Cartas.md): `GET /card`, `GET /card/:rarity`, `POST /card` (multipart), `DELETE /card/:id` (limpieza transaccional), `POST /user/:id/card/random`.
- 🏪 [Endpoints Mercado](./Backend%20API/Endpoints%20Mercado.md): `GET /offers`, `POST /offers` (publicar), `POST /buy`, `DELETE /offers/:offerId` (cancelar).
- 🏆 [Endpoints Leaderboard](./Backend%20API/Endpoints%20Leaderboard.md): Rankings de suerte, rachas (`streaks`), riqueza (`wealth`) y coleccionistas (`cards`).
- 🖼️ [Hosting de Imágenes](./Backend%20API/Hosting%20de%20Imagenes.md): Abstracción Local, Cloudinary y Supabase Storage.
- 🐞 [Problemas Conocidos](./Backend%20API/Problemas%20Conocidos.md): Fallos reales del backend y del bot pendientes de arreglar, por prioridad.

### 💡 [Propuestas y Roadmap](./Propuestas%20y%20Roadmap)
Diseños conceptuales y futuras mecánicas en fase de exploración:
- 🎯 [Sistema de Piedad (Pity)](./Propuestas%20y%20Roadmap/Sistema%20de%20Piedad%20(Pity).md): Garantía de carta Mítica a las 250 tiradas y control de mala suerte.
- ✨ [Variantes Shiny](./Propuestas%20y%20Roadmap/Variantes%20Shiny.md): Variante cosmética brillante ultra-rara (2%-3% de aparición).
- 🧪 [Reciclaje y Polvo Arcano](./Propuestas%20y%20Roadmap/Reciclaje%20y%20Polvo%20Arcano.md): Destrucción de duplicadas por polvo para comprar consumibles y amuletos.
- 📚 [Colecciones y Tags](./Propuestas%20y%20Roadmap/Colecciones%20y%20Tags.md): Cartas etiquetadas en varias colecciones temáticas, progreso por colección y recompensas al completarlas.

### 🤖 [Bot Discord](./Bot%20Discord)
- 🏛️ [Arquitectura y Eventos](./Bot%20Discord/Arquitectura%20y%20Eventos.md): Stack, variables de entorno, cliente HTTP con `BOT_API_KEY`, middlewares (`checkUser`, `checkAdmin`), sincronización de perfil e interacciones.
- 🎨 [Componentes de UI](./Bot%20Discord/Componentes%20de%20UI.md): Embeds interactivos, botones de navegación y paginación protegida.
- 🪙 [Comandos de Economía](./Bot%20Discord/Comandos%20de%20Economia.md): Referencia y alias (`.balance`, `.getcard`, `.market`, etc.).
- 🎵 [Comandos de Música](./Bot%20Discord/Comandos%20de%20Musica.md): `.play`, `.stop` con `discord-player`.
- 🛠️ [Comandos de Utilidad](./Bot%20Discord/Comandos%20de%20Utilidad.md): `.help`, `.ping`.

### 🌐 [Frontend](./Frontend)
- 🏗️ [Arquitectura Frontend](./Frontend/Arquitectura%20Frontend.md): Stack, integración con la API (cookies, refresh, login), configuración que debe coincidir con el backend y alcance de la v1.
- 🧭 [Plan Frontend](./Frontend/Plan%20Frontend.md): Fases de las pantallas (Inicio, Tirada, Colección, Rankings, Historial, Mercado, Admin) y decisiones de UX: navegación, animación de la tirada por rareza y colección tipo álbum.
- 🐾 [Rediseño Visual](./Frontend/Rediseno%20Visual.md): Identidad visual de Smilbot (oscura y cálida): paleta, tipografía, formas, mascota, componentes y login. Implementada.
- 📋 [Pendientes Frontend](./Frontend/Pendientes%20Frontend.md): Necesidades del backend, incoherencias de la bóveda y tareas pendientes detectadas desde el frontend.

---

## 🛠️ Cómo Usar esta Bóveda

### En Obsidian
1. Abre Obsidian.
2. Selecciona **"Abrir carpeta como bóveda"** (*Open folder as vault*).
3. Selecciona la raíz de este repositorio.

### Con Agentes de IA (MCP)
Esta bóveda está conectada vía MCP (`@modelcontextprotocol/server-filesystem`), permitiendo a los asistentes de IA de desarrollo leer y actualizar la documentación automáticamente en ambos repositorios.
