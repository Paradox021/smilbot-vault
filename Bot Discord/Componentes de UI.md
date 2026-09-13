# 🎨 Bot Discord: Componentes de UI e Interacciones

---

## 1. Embeds Estándar
- **`createCardEmbed(card)`:** Muestra la carta a pantalla completa con el color de borde correspondiente a su rareza y el autor.
- **`createTextEmbed(color, message)`:** Notificaciones de error (rojo), éxito (verde) o aviso (amarillo).

---

## 2. Paginación Interactiva (`CardNavigator`)
- Utiliza `ActionRowBuilder` con botones (`◀️ Anterior`, `Siguiente ▶️`, `🔍 Alternar Vista`).
- **Seguridad:** Solo el usuario que invocó el comando puede interactuar con los botones.
- **Timeout:** Los colectores tienen un tiempo de vida (ej. 120s) tras el cual se deshabilitan los botones para no saturar memoria.
