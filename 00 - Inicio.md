# 🌌 Ecosistema Smilbot - Documentación Centralizada

Bienvenido a la bóveda de documentación técnica compartida de **Smilbot**. Esta bóveda es la **Única Fuente de Verdad** compartida entre el repositorio del **Bot de Discord** y el repositorio del **Backend API**.

---

## 🗺️ Mapa de Navegación

### 📐 [[Modelos y Contratos]]
Los tipos, interfaces, reglas de negocio y modelos de datos compartidos:
- [[Modelos y Contratos/Cartas y Rarezas|Cartas y Rarezas]]: Estructura de cartas, rarezas (Common a Mythic), colores y pesos.
- [[Modelos y Contratos/Economia y Rachas|Economía y Rachas]]: Balance de monedas, regla de cooldown de 23h-48h, rachas y récords.
- [[Modelos y Contratos/Sistema de Suerte y Tiers|Sistema de Suerte y Tiers]]: Neutralización contra mercado, fórmula de delta de suerte y tiers (GODLY a CURSED).
- [[Modelos y Contratos/Mercado y Transacciones|Mercado y Transacciones]]: Estados de oferta (ACTIVE, SOLD, CANCELLED) y libro mayor (Ledger de transacciones).
- [[Modelos y Contratos/Gacha y Pity|Gacha, Pity y Futuras Mecánicas]]: Sistema de piedad a las 250 tiradas, variantes Shiny y tienda de consumibles.

### 🔌 [[Backend API]]
Especificación de los endpoints REST expuestos por el Backend y consumidos por el Bot:
- [[Backend API/Endpoints Usuarios y Stats|Endpoints Usuarios y Stats]]: `GET /user/:discordId`, `GET /user/:discordId/stats`, `POST /user`.
- [[Backend API/Endpoints Cartas|Endpoints Cartas y Gacha]]: `GET /card`, `POST /card` (multipart), `POST /user/:discordId/card/random`.
- [[Backend API/Endpoints Mercado|Endpoints Mercado]]: `GET /market/:serverId/offers`, `POST /market/:serverId/offers/:offerId/buy`.
- [[Backend API/Endpoints Leaderboard|Endpoints Leaderboard]]: `GET /leaderboard/luck` (rankings 'desc' y 'asc').

### 🤖 [[Bot Discord]]
Diseño de la aplicación cliente en Discord.js:
- [[Bot Discord/Arquitectura y Eventos|Arquitectura y Eventos]]: Flujo de mensajes, middlewares (checkAdmin, checkUser), handlers y eventos.
- [[Bot Discord/Componentes de UI|Componentes de UI]]: Embeds de cartas, botones de paginación y seguridad de sesión.
- [[Bot Discord/Comandos de Economia|Comandos de Economía]]: `.balance`, `.dailybalance`, `.mycards`, `.getcard`, `.market`, `.stats`, `.top`, `.createcard`.
- [[Bot Discord/Comandos de Musica|Comandos de Música]]: `.play`, `.stop` y reproductor con discord-player.
- [[Bot Discord/Comandos de Utilidad|Comandos de Utilidad]]: `.help`, `.ping`.

---

## 🚨 Reglas de Convivencia para Agentes de IA
1. **Modificación en el Backend:** Si cambias un modelo o endpoint, actualiza la nota correspondiente en `Backend API/` o `Modelos y Contratos/`.
2. **Modificación en el Bot:** Antes de cambiar una llamada HTTP o tipado, consulta esta documentación para asegurarte de que coincide con el contrato del backend.
