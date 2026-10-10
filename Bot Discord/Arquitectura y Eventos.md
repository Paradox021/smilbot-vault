# 🤖 Bot Discord: Arquitectura y Eventos

- **Repo:** [Paradox021/Smilbot](https://github.com/Paradox021/Smilbot), rama `master`.

---

## 1. Stack Tecnológico
- **Framework:** Discord.js v14
- **Lenguaje:** TypeScript 5 (estricto)
- **Ejecución y Build:** `tsx watch` para desarrollo (`npm run dev`), `tsc` + `tsc-alias` para producción (`npm run build`, `npm start`).
- **Audio:** `discord-player` v7 con extractores, FFmpeg y Opus.
- **Despliegue:** `Dockerfile` (Node 20 con FFmpeg, Opus y `yt-dlp`) y `docker-compose.yml` que carga el `.env`.
- **Intents:** `Guilds`, `GuildMessages`, `MessageContent`, `GuildVoiceStates`.

### Variables de entorno (`.env.sample`)

| Variable | Descripción |
| :--- | :--- |
| `TOKEN` | Token del bot de Discord. |
| `BACKEND_URL` | URL base del Backend API. |
| `BOT_API_KEY` | Debe ser idéntica a `BOT_API_KEY` del backend. El bot no arranca sin ella. Ver [Autenticación](../Backend%20API/Autenticacion.md). |

---

## 2. Cliente HTTP (`src/services/api.ts`)
- Instancia única de axios con `baseURL = BACKEND_URL` y `timeout` de 5 s.
- Añade `Authorization: Bot <BOT_API_KEY>` en cada petición (con un interceptor, no en los defaults, para que la clave no aparezca al imprimir la configuración).
- Oculta la clave en las respuestas y errores de axios para que no se filtre en los logs.
- Registra los `401` (clave incorrecta o ausente) y `403` con la ruta afectada. No reintenta.
- Los servicios (`cardService`, `userService`, `economyService`, `marketService`, `leaderboardService`) usan siempre esta instancia.

---

## 3. Pipeline de Comandos y Middlewares
```text
messageCreate -> prefijo '.' -> nombre o alias -> middlewares globales -> middlewares del comando -> execute()
```

- **Middleware global:** registra en consola cada comando recibido.
- Cada comando declara sus propios middlewares (`middlewares: [...]`); no todos los llevan.
- **`checkUser`:** registra al autor con `POST /user` (`{ discordId, username }`). Un `409` (ya existe) se considera correcto.
- **`checkAdmin`:** requiere el permiso nativo de Administrador de Discord o un rol llamado `admin` (sin distinguir mayúsculas). Solo lo usa `.createcard`.

---

## 4. Interacciones (botones y menús)
`interactionCreate` delega en `src/handlers/InteractionHandler.ts`, que prueba en orden los handlers de ayuda, cartas y mercado según el `customId`. Ver [Componentes de UI](./Componentes%20de%20UI.md).
