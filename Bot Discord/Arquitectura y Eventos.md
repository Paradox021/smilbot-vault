# 🤖 Bot Discord: Arquitectura y Eventos

---

## 1. Stack Tecnológico
- **Framework:** Discord.js v14
- **Lenguaje:** TypeScript 5 (estricto)
- **Ejecución y Build:** `tsx` para desarrollo, `tsc` + `tsc-alias` para producción.
- **Audio:** `discord-player` v7 con extractores y FFmpeg.

---

## 2. Pipeline de Comandos y Middlewares
Cada comando pasa por una cadena de middlewares antes de ejecutarse:

```text
Mensaje -> Prefijo '.' -> Parseo -> checkUser -> checkAdmin -> Comando.execute()
```

- **`checkUser`:** Verifica que el usuario exista en la base de datos a través de `userService.ensureUser()`. Si no existe, lo crea de forma transparente.
- **`checkAdmin`:** Requiere permisos nativos de Administrador de Discord o un rol con nombre `admin` o `ADMIN`.
