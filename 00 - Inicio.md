# 🌌 Ecosistema Smilbot - Documentación Centralizada

Bienvenido a la bóveda de documentación técnica compartida de **Smilbot**. Esta bóveda es la **Única Fuente de Verdad** compartida entre el repositorio del **Bot de Discord** y el repositorio del **Backend API**.

---

## 🔗 Repositorios
- 🤖 **Bot Discord:** [Paradox021/Smilbot](https://github.com/Paradox021/Smilbot)
- 🔌 **Backend API:** [Paradox021/backendSmilbot](https://github.com/Paradox021/backendSmilbot)

---

## 🗺️ Mapa de Navegación

### 📐 [[Modelos y Contratos]]
- [[Modelos y Contratos/Cartas y Rarezas|Cartas y Rarezas]]: Esquema `Card`, tabla de probabilidades reales (Mythic 0.5% a Common 57.5%).
- [[Modelos y Contratos/Economia y Rachas|Economía y Rachas]]: Balance de monedas, regla de cooldown de 23h-48h, rachas y récords.
- [[Modelos y Contratos/Sistema de Suerte y Tiers|Sistema de Suerte y Tiers]]: Neutralización contra mercado, delta de suerte y tiers.
- [[Modelos y Contratos/Mercado y Transacciones|Mercado y Transacciones]]: Ofertas `ACTIVE`, `SOLD`, `CANCELLED` y libro mayor (`transactions`).
- [[Modelos y Contratos/Gacha y Pity|Gacha, Pity y Variantes Shiny]]: Sistema de piedad a las 250 tiradas, cartas Shiny ✨ y tienda de consumibles.

### 🔌 [[Backend API]]
- [[Backend API/Endpoints Usuarios y Stats|Endpoints Usuarios y Stats]]: `GET /user/:id`, `GET /stats`, `GET /transactions`, `POST /dailyBalance`.
- [[Backend API/Endpoints Cartas|Endpoints Cartas y Gacha]]: `GET /card`, `GET /card/:rarity`, `POST /card` (multipart), `POST /user/:id/card/random`.
- [[Backend API/Endpoints Mercado|Endpoints Mercado P2P]]: `GET /offers`, `POST /offers` (publicar), `POST /buy`, `POST /cancel`.
- [[Backend API/Endpoints Leaderboard|Endpoints Leaderboards]]: Rankings de suerte, rachas (`streaks`), riqueza (`wealth`) y coleccionistas (`cards`).
- [[Backend API/Hosting de Imagenes|Hosting de Imágenes]]: Abstracción de proveedores Local, Cloudinary y Supabase Storage.

### 🤖 [[Bot Discord]]
- [[Bot Discord/Arquitectura y Eventos|Arquitectura y Eventos]]: Flujo de middlewares, handlers y listeners.
- [[Bot Discord/Componentes de UI|Componentes de UI]]: Embeds de cartas, botones de paginación y protección de sesión.
- [[Bot Discord/Comandos de Economia|Comandos de Economía]]: `.balance`, `.dailybalance`, `.mycards`, `.getcard`, `.market`, `.stats`, etc.
- [[Bot Discord/Comandos de Musica|Comandos de Música]]: `.play`, `.stop` y reproductor con `discord-player`.
- [[Bot Discord/Comandos de Utilidad|Comandos de Utilidad]]: `.help`, `.ping`.
