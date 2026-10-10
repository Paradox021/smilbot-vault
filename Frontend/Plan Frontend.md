# 🧭 Plan del Frontend

Hoja de ruta de las pantallas de la web y decisiones de producto y UX tomadas antes de implementarlas. Cuando una fase se termina, su detalle técnico pasa a [Arquitectura Frontend](./Arquitectura%20Frontend.md) y se marca aquí como hecha.

> Decidido el 2026-10-11. La referencia visual sigue siendo el `DESIGN.md` del repo del frontend (oscuro, monocromo, Inter con `ss03`, CTA blanco). El único color fuerte de la interfaz es el de las **rarezas**: así una carta rara destaca sola.

---

## 1. Principios de Producto

1. **La web es un sitio donde jugar, no solo donde consultar.** Discord no puede animar nada; la web sí. Abrir sobres es el momento emocional del juego y es lo que justifica que alguien entre en la web.
2. **Mismo juego, mismas reglas.** El daily y las tiradas de la web usan los mismos endpoints que el bot: el cooldown, el saldo y el pity son compartidos. Reclamar en la web cuenta como haberlo reclamado en Discord.
3. **Mobile first.** La mayoría llegará desde un enlace en Discord en el móvil.
4. **La espera se ajusta a la recompensa.** Una común (57.5 % de las tiradas) se resuelve rápido; la ceremonia crece con la rareza. Todo se puede saltar.

---

## 2. Navegación

| Destino | Ruta | Dónde aparece |
| :--- | :--- | :--- |
| Inicio | `/inicio` | Nav principal |
| Tirar | `/tirar` | Nav principal (acción destacada) |
| Colección | `/coleccion` | Nav principal |
| Rankings | `/rankings` | Nav principal |
| Perfil (stats detalladas) | `/perfil` | Menú de usuario |
| Historial | `/historial` | Menú de usuario |
| Mercado | `/mercado` | Nav principal (cuando se desbloquee, ver fase 6) |

- **Escritorio:** cabecera de 56 px con la nav en una línea, el **saldo siempre visible** (píldora con monedas) y el menú de usuario.
- **Móvil:** barra inferior con 4 pestañas (Inicio, Tirar, Colección, Rankings) en la zona del pulgar; la cabecera conserva el saldo y el avatar.
- Tras el login, el destino por defecto pasa de `/perfil` a `/inicio`.
- El saldo de la cabecera sale de `GET /user/:id/stats`, nunca de `/auth/me`.

---

## 3. Fases

| Fase | Pantalla | Endpoints | Estado |
| :--- | :--- | :--- | :--- |
| 1 | Estructura (nav, saldo) + **Inicio** | `GET /stats`, `POST /dailyBalance` | Pendiente |
| 2 | **Tirada** | `POST /card/random` | Pendiente |
| 3 | **Colección** | `GET /card`, `GET /user/:id/cards` | Pendiente |
| 4 | **Rankings** | `GET /leaderboard/*` | Pendiente |
| 5 | **Historial** | `GET /user/:id/transactions` | Pendiente |
| 6 | **Mercado** | `/market/:serverId/*` | 🔒 Bloqueado: la web no conoce los servidores del usuario. Ver [Pendientes Frontend](./Pendientes%20Frontend.md). |
| 7 | **Admin** (crear y borrar cartas) | `POST /card`, `DELETE /card/:id` | Pendiente |

Fuera de alcance por ahora: tirada múltiple (x10). Necesitaría un endpoint atómico nuevo; no se simula con varias peticiones.

---

## 4. Inicio (fase 1)

Tablero con lo que el usuario puede hacer **hoy**, en este orden de prioridad:

1. **Daily.** Una tarjeta con tres estados:
   - *Disponible:* botón principal "Reclamar 100 monedas".
   - *En cooldown:* cuenta atrás hasta `nextDailyAt`. Sin botón.
   - *Racha en peligro:* si `streakExpiresAt` está a menos de 6 h y el daily está disponible, aviso "Tu racha de N días caduca en X h".
2. **Racha:** días actuales y récord (`maxDailyStreak`).
3. **Tirar:** acceso a `/tirar` con el coste (100) y el progreso de pity (`pullsUntilGuaranteed`).
4. Enlace a la colección con el progreso "N de M descubiertas".

**Feedback al reclamar** (mismos casos que el bot, ver [Economía y Rachas](../Modelos%20y%20Contratos/Economia%20y%20Rachas.md#4-feedback-visual-en-el-bot-dailybalance)):
- El saldo de la cabecera sube con un contador animado (+100).
- Racha que sube: la cifra cambia con una transición corta.
- `isNewRecord`: mensaje de nuevo récord.
- Racha rota (`dailyStreak = 1` y `previousStreak ≥ 2`): aviso de que se perdió la racha anterior.
- `400` por cooldown: no debería ocurrir (el botón no se muestra), pero si llega, se muestra el tiempo restante de la respuesta y se refrescan las stats.

Invalida: `stats` del usuario.

---

## 5. Tirada (fase 2)

Pantalla propia (`/tirar`), a pantalla completa, sin distracciones. Es la pieza con más diseño de la web.

### Flujo
1. **Antes de tirar:** sobre/dorso de carta en el centro, botón "Abrir sobre · 100", saldo y pity visibles. Si el saldo no llega, el botón se desactiva con "Te faltan X monedas" (se sabe por `stats`, sin esperar al `400`).
2. **Petición en vuelo:** el dorso empieza a "barajarse" en cuanto se pulsa. Disimula la latencia: la animación arranca antes de que llegue la respuesta.
3. **Anticipación según la rareza** (cuando ya se conoce el resultado): el dorso se ilumina con el color de la rareza antes de girar.
4. **Revelado:** la carta gira y se muestra con su marco de rareza.
5. **Resultado:** nombre, rareza, insignia **"Nueva"** si es la primera copia, "Garantizada por piedad" si `isPity`, pity actualizado y botón "Abrir otro".

### Escalado por rareza

| Rareza | Duración aprox. | Anticipación | Revelado |
| :--- | :--- | :--- | :--- |
| Común | ~0.8 s | Ninguna: gira directamente | Sin efectos |
| Rara | ~1.2 s | Brillo azul suave | Borde azul |
| Épica | ~1.6 s | Brillo púrpura con un pulso | Borde púrpura y destello |
| Legendaria | ~2.2 s | Dos pulsos dorados crecientes | Destello y rayos de luz |
| Mítica | ~3 s | El fondo de la pantalla se tiñe; tres pulsos | Destello grande, partículas y brillo holográfico en la carta |

- **Siempre se puede saltar:** tocar o pulsar cualquier tecla lleva al resultado al instante.
- **`prefers-reduced-motion`:** sin giros ni partículas; la carta aparece con un fundido y el marco de rareza.
- La anticipación nunca miente: el color que aparece antes del giro es siempre el de la carta que sale.
- Animaciones con Motion, solo `transform` y `opacity`.
- **Errores:** `400` sin saldo y `404 No cards found for this type` (no se cobra) vuelven al estado inicial con el mensaje; la animación se corta.

**"Nueva":** se calcula con el inventario que el cliente ya tiene en caché (la carta no estaba o tenía `count = 0` antes de la tirada). No requiere peticiones extra.

Invalida: `stats` e inventario del usuario.

---

## 6. Colección (fase 3)

Un **álbum**: el catálogo completo (`GET /card`) cruzado con el inventario (`GET /user/:id/cards`). Lo que falta motiva a tirar.

### Cabecera
- "N de M descubiertas" y el desglose por rareza (5 contadores con el color de cada rareza, p. ej. "Míticas 1/4").

### Rejilla
- 2 columnas en móvil, 3-4 en tablet, 5-6 en escritorio. Cartas en proporción **5:7** (carta coleccionable).
- **Conseguida:** imagen completa, marco con el color de la rareza y contador de copias si hay más de una ("×3").
- **No conseguida:** dorso oscuro con el marco de su rareza, nombre oculto ("???") y la imagen sin mostrar. La rareza sí se ve: saber que falta una mítica es parte del gancho.
- Las míticas y legendarias conseguidas tienen un brillo holográfico sutil al pasar el cursor (desactivado con reduced motion).
- Imagen de reserva única cuando falta `imageUrl` (regla del `AGENTS.md`).

### Controles
- Píldoras de rareza (Todas, Común… Mítica), filtro "Todas / Conseguidas / Me faltan", búsqueda por nombre (solo entre las conseguidas, para no destapar nombres) y orden (rareza o nombre).

### Detalle de carta
- Al tocar una carta conseguida: diálogo (hoja inferior en móvil) con la imagen grande, nombre, rareza y su probabilidad ("Mítica · 0.5 %"), descripción, autor y copias.
- Las no conseguidas no abren detalle; muestran "Aún no la tienes" y un acceso a Tirar.
- Hueco reservado para "Vender en el mercado" cuando exista la fase 6.

---

## 7. Rankings (fase 4)

- Pestañas: Suerte, Rachas, Fortuna y Coleccionistas. Cada una con su conmutador (`desc/asc`, `current/max`, `earned/current`).
- Top 3 destacado arriba, el resto en lista con avatar, nombre y valor.
- La fila del propio usuario se resalta si aparece en el top.
- Suerte: si el usuario no es elegible (`eligibleForLeaderboard: false` en sus stats), aviso "Te faltan N cartas para entrar en el ranking".

## 8. Historial (fase 5)

- Lista paginada del Ledger agrupada por día. Cada movimiento con icono por `type`, descripción legible a partir de `metadata` (p. ej. "Tirada: SwimmingLarry (Mítica)") y la variación de saldo en verde o rojo.

---

## 9. Diseño de la Carta (componente compartido)

Un único componente `CardTile` para tirada, colección, detalle, rankings y mercado:
- Marco de 1 px con el color de la rareza (de `shared/config/rarities.ts`), radio de 10 px, fondo `surface`.
- Variantes: `owned`, `locked` (dorso) y `reveal` (para la tirada).
- La rareza nunca se comunica **solo** con color: también con el nombre de la rareza (accesibilidad).

Véase también:
- [[Frontend/Arquitectura Frontend|Arquitectura Frontend]]
- [[Frontend/Pendientes Frontend|Pendientes Frontend]]
- [[Modelos y Contratos/Gacha y Pity|Gacha y Pity]]
