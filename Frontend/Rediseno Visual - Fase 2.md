# 🐾 Rediseño Visual, Fase 2: Personalidad y Experiencia

Segunda fase del rediseño. La [fase 1](./Rediseno%20Visual.md) cambió el **aspecto**: paleta, tipografía, formas, mascota en login, 404 y logo, y un login nuevo. Las pantallas siguen teniendo el mismo layout y la tirada la misma animación, así que la web todavía se siente como un panel de datos con otros colores.

Esta fase cambia **cómo se vive**: la estructura de las pantallas, el movimiento y la voz de Smilbot.

> Decidido el 2026-10-11. **Estado: implementado (etapas 1 a 5) el 2026-10-11** y fusionado en `master` del frontend; los sonidos (etapa 6, opcional) quedan aplazados. Detalle técnico en [Arquitectura Frontend](./Arquitectura%20Frontend.md#identidad-visual) y referencia en el `DESIGN.md` del frontend. Los ajustes respecto a lo definido están en la sección 13. Se recomienda hacerla **antes de la fase 3 (Colección)** del [Plan Frontend](./Plan%20Frontend.md), para que la Colección nazca ya con este lenguaje (sistema de movimiento, cartas y mascota).

---

## 1. Objetivo

Que la web se sienta como **el juego de Smilbot** y no como un dashboard. Para eso:

1. **Smilbot habla.** Es el guía de la app, no solo el logo.
2. **Inicio es un "lobby" de juego:** una acción del día clara y el progreso como objetos visuales, no como cifras sueltas.
3. **La tirada es una ceremonia:** se abre un sobre de verdad.
4. **Todo responde.** Cada acción tiene una respuesta física pequeña (monedas que vuelan, botones que se hunden, la llama que se aviva).

Se mantienen las reglas de la fase 1: el naranja es escaso, los colores fuertes son de las rarezas, el naranja de marca nunca marca una carta, solo hay modo oscuro y la animación usa `transform` y `opacity` con respeto a `prefers-reduced-motion`.

---

## 2. Decisiones

| Tema | Decisión |
| :--- | :--- |
| Smilbot en las pantallas de datos | **Sí, en Inicio y en la tirada**, como guía con un bocadillo de texto. Cambia la regla de la fase 1, que lo limitaba al login, los estados vacíos y los errores. En Perfil, Colección y Rankings sigue sin aparecer salvo en sus estados vacíos. |
| Tirada | **Se abre un sobre** con la marca de Smilbot, en lugar de girar un dorso suelto. |
| Sonidos | **Opcionales y apagados por defecto.** Van en la última etapa y se pueden dejar fuera sin afectar a lo demás. |
| Vibración | En el móvil, solo en revelados de Épica o mejor y solo donde el navegador la soporte (`navigator.vibrate`; iOS Safari no la tiene). |
| Logros en el perfil | **Fuera de alcance:** el backend no tiene logros. Se valorará como propuesta aparte. |

---

## 3. Smilbot como Guía

Componente nuevo `SmilbotSays`: la mascota pequeña (unos 56 px, en círculo) con un bocadillo al lado. Un solo mensaje cada vez, elegido por prioridad según el estado del usuario.

**En Inicio (de más a menos prioridad):**

| Situación | Mensaje (ejemplo) |
| :--- | :--- |
| La racha caduca en menos de 6 h | "¡Tu racha de 11 días caduca en 3 h! Reclama tu recompensa." |
| Daily disponible | "Tu recompensa diaria está lista. ¡100 monedas te esperan!" |
| Mítica garantizada en 10 sobres o menos | "Solo te quedan 6 sobres para la mítica garantizada." |
| Sin cartas todavía | "¡Abre tu primer sobre y empieza tu colección!" |
| Por defecto | Un saludo que rota ("¡Hola de nuevo, Gamer123!", "¿Probamos suerte hoy?") |

**En la tirada:** reacciona al resultado solo en Épica o mejor ("¡Una épica! Bien hecho", "¡¿Una mítica?! ¡Miau!"), y en las repetidas sin emoción ("Esta ya la tenías").

**Reglas:**
- El bocadillo entra con un pequeño rebote al cargar o al cambiar de mensaje. No hay animación en bucle.
- Para el lector de pantalla es texto normal, sin `aria-live`, salvo la reacción de la tirada, que ya se anuncia en el resultado.
- El tono es el de la fase 1: cercano, de tú, un guiño gatuno de vez en cuando y nunca en errores ni en textos de dinero.
- Con la foto actual solo hay una pose. Las poses distintas son de la etapa con ilustración (sección 9).

---

## 4. Inicio como Lobby

Estructura nueva, de arriba abajo:

1. **Saludo y Smilbot:** "Hola, {nombre}" con `SmilbotSays` debajo.
2. **Acción del día** (tarjeta grande y destacada, la única con botón naranja):
   - Daily disponible: "Recompensa diaria", la cantidad grande en Nunito, el aviso de racha si toca y el botón "Reclamar".
   - Daily reclamado: "Abrir un sobre", con un **montón de sobres** dibujado, el coste y el botón "Abrir sobre", más la cuenta atrás del próximo daily en pequeño.
3. **Progreso** (dos tarjetas, en columna en móvil):
   - **Racha:** una tira de 7 huellas. Las huellas de la racha actual están rellenas y la de hoy se marca. Debajo, el récord. Si la racha es de más de 7 días, se muestra "Semana 2", "Semana 3"...
   - **Mítica garantizada:** un medidor que se llena hacia el **color mítico** (está permitido porque mide una rareza), con "6 sobres" grande en Nunito.
4. **Tus últimas cartas** (cuando exista la fase 3): una tira horizontal con las últimas cartas conseguidas.
5. Enlace discreto "Ver todas tus estadísticas".

---

## 5. La Tirada con Sobre

### El sobre
- Es un objeto de marca, no una carta, así que **puede ser naranja**. Lleva la huella, la palabra "Smilbot" en Nunito y un brillo suave de papel metalizado. Es la única excepción a la regla de "sin degradados de marca".
- Proporción 5:7, algo más ancho que la carta para que la carta "quepa" dentro.
- Se dibuja con CSS o SVG a partir de los tokens: nada de imágenes. Así se puede rasgar en dos piezas.

### Secuencia nueva

| Fase | Qué pasa |
| :--- | :--- |
| Reposo | El sobre en el centro, con una leve respiración (sube y baja unos px). Es el único bucle permitido, y se quita con movimiento reducido. |
| Petición en vuelo | Al pulsar "Abrir sobre", el sobre **tiembla** y disimula la latencia, como hoy el barajado. |
| Anticipación | Por la línea de corte del sobre se escapa **luz del color de la rareza**, con los mismos pulsos que hoy. La anticipación nunca miente. |
| Apertura | La parte de arriba del sobre se rasga y sale volando (unos 400 ms). |
| Revelado | La carta **sube desde dentro** del sobre y gira, con los efectos actuales por rareza (destello, rayos, partículas, tinte y holo). El sobre se desvanece. |
| Resultado | Igual que hoy, más la reacción de Smilbot si es Épica o mejor, la vibración y, en repetidas, "Ya la tenías ×N" (el contador sale del inventario en caché). |

- **Los tiempos por rareza no cambian** (`features/pull/reveal.ts`); la apertura añade unos 400 ms a todas.
- **Saltar** sigue igual: tocar, pulsar una tecla o el botón llevan al resultado al instante.
- **Interacción opcional en el móvil:** deslizar el dedo por la línea de corte rasga el sobre. El botón sigue siendo el camino principal y accesible.
- **Movimiento reducido:** sin temblor, rasgado ni giro. El sobre se funde y aparece la carta con su marco.
- "Abrir otro" vuelve a poner un sobre nuevo en reposo con una entrada corta.

---

## 6. Microinteracciones

| Dónde | Qué |
| :--- | :--- |
| Reclamar el daily | Unas monedas salen del botón y **vuelan a la píldora del saldo**, que da un pequeño salto al recibirlas. Ya existe el contador animado. |
| Abrir un sobre | Aparece "−100" junto al saldo y se desvanece hacia arriba. |
| Racha que sube | La huella nueva se rellena con un rebote y la llama se aviva un momento. |
| Botones | Al pulsar se hunden un poco (`scale` de 0,97). |
| Carta "Nueva" | La insignia entra con un rebote y unas pocas huellas pequeñas como confeti, solo la primera vez que se ve. |
| Comprobando la sesión (carga de página completa) | Smilbot con "Smilbot se está despertando…" en lugar de solo texto. |
| Errores de pantalla | Smilbot con el mensaje y "Reintentar". |

Los skeletons se quedan para las cargas de datos dentro de una pantalla, porque mantienen la forma del contenido.

---

## 7. Perfil como Ficha de Jugador

- **Cabecera de ficha:** avatar grande con anillo, nombre en Nunito, el **nivel de suerte como título** (chip con su icono, p. ej. "Afortunado") y "Miembro desde marzo de 2025".
- **Fila de cifras clave:** cartas, sobres abiertos y mejor racha, grandes y con icono.
- **Desglose por rareza** como una **barra apilada** con los colores de rareza y una leyenda con nombre y cantidad (la rareza nunca solo con color).
- El resto de stats (economía, pity) en tarjetas más sencillas debajo.

---

## 8. Sistema de Movimiento

Para que toda la app se mueva igual, se definen tokens en `shared/config/motion.ts`:

| Token | Valor orientativo | Uso |
| :--- | :--- | :--- |
| `fast` | 150 ms | Hover, pulsado |
| `base` | 250 ms | Entradas de elementos, cambios de estado |
| `slow` | 400 ms | Apertura del sobre, transiciones grandes |
| `spring.snappy` | Muelle rígido, sin rebote | Botones, pestañas |
| `spring.bouncy` | Muelle con un rebote | Bocadillo de Smilbot, insignias, huellas |

**Reglas:**
- Sin bucles infinitos, salvo la respiración del sobre en reposo.
- Todo se desactiva o se cambia por un fundido con `prefers-reduced-motion`.
- Motion solo en páginas diferidas. Lo que va en la cabecera (saldo, monedas que llegan) usa CSS o `requestAnimationFrame`, como hoy, para no cargar Motion en el bundle principal.

---

## 9. Lo que Espera a la Ilustración

Con la foto, Smilbot tiene una sola pose. Si se encarga la ilustración ([fase 1, opción C](./Rediseno%20Visual.md#la-foto-original)):
- **Poses:** dormido (estados vacíos), sorprendido (404, mítica), celebrando (carta nueva), esperando (carga).
- El sobre puede llevar la ilustración en lugar de la huella.
- `SmilbotSays` recibirá una pose además del mensaje. Se diseña ya con ese parámetro opcional para que el cambio sea solo de ficheros.

---

## 10. Sonidos (Opcional)

- Apagados por defecto. Interruptor en el menú de usuario, guardado en `localStorage`.
- Pocos y cortos: monedas, rasgado del sobre y un acorde por rareza a partir de Épica.
- Ficheros pequeños (menos de 30 KB cada uno), con licencia libre y cargados solo al activarlos.

---

## 11. Qué Cambia en el Código

| Dónde | Cambio |
| :--- | :--- |
| `shared/config/motion.ts` | Tokens de movimiento (nuevo). |
| `shared/ui/` | `SmilbotSays` (mascota con bocadillo) y efecto de pulsado en `Button`. |
| `features/home` | Layout de lobby: tarjeta de acción del día, tira de huellas de la racha, medidor de la mítica y elección del mensaje de Smilbot (lógica pura con tests). |
| `features/pull` | Componente `Pack` (sobre), fases nuevas en el reductor de la secuencia (apertura) y reacciones de Smilbot. |
| `features/stats` | Monedas que vuelan a la píldora del saldo y "−100" al tirar. |
| `features/profile` | Cabecera de ficha y barra apilada por rareza. |
| `features/auth` | Carga de sesión con Smilbot. |
| `DESIGN.md` | Secciones de movimiento, `SmilbotSays` y el sobre; nueva regla de la mascota en Inicio y en la tirada. |

## 12. Orden de Implementación

Cada etapa es una rama propia y se puede probar por separado:

1. **Base:** tokens de movimiento, pulsado de botones y `SmilbotSays`.
2. **Inicio como lobby**, con los mensajes de Smilbot.
3. **Tirada con sobre:** sobre, apertura, reacciones, "Ya la tenías ×N" y vibración.
4. **Microinteracciones del saldo y de la racha**, más la carga y los errores con Smilbot.
5. **Perfil como ficha.**
6. **Sonidos** (opcional).

Al terminar cada etapa se actualizan `DESIGN.md`, [Arquitectura Frontend](./Arquitectura%20Frontend.md) y, si cambia el flujo de la tirada, la sección 5 del [Plan Frontend](./Plan%20Frontend.md).


---

## 13. Ajustes al Implementar

- **`SmilbotSays` sin parámetro de pose.** Con una sola foto sería código muerto; se añadirá junto con las poses de la ilustración (sección 9).
- **Botón del daily:** "Reclamar" (la cantidad ya se ve en grande al lado).
- **Huellas de la racha:** rellenas en crema (como el progreso) y la de hoy con anillo; si aún no se ha reclamado, discontinua (en ámbar si la racha está en peligro).
- **Con el daily disponible**, la tarjeta de la mítica garantizada lleva un acceso secundario a la tirada, para no perderlo de vista.
- **En móvil, el botón de la tirada queda fijo** sobre la barra inferior: con la reacción de Smilbot el resultado ya no cabe entero en pantalla.
- **Perfil:** la ficha mantiene el ID de Discord (en pequeño) y la tarjeta "Colección" desaparece (sus cifras pasan a la fila de cifras clave). El pity usa el mismo medidor mítico que Inicio.
- **"−100" y "+100"** flotan junto a la píldora del saldo; las monedas del daily salen cuando el backend ha confirmado el reclamo.
- **Luz de la rareza (2026-10-11, tras probarla):** la primera versión era una franja de luz encima de la línea de corte que se salía del sobre y parecía un fallo. Ahora la luz sale del propio sobre (halo alrededor y ranura que brilla por la perforación) y al rasgarlo un haz corto sale por la boca. El sobre cae y se pierde por el borde inferior antes de que la carta gire (antes se superponían), el sobre ya no parpadea en la segunda tirada y siguientes (el bucle del temblor alcanzaba a la opacidad), y se corrigió y que los rayos del resultado se cortaran en móvil.

Véase también:
- [[Frontend/Rediseno Visual|Rediseño Visual (fase 1)]]
- [[Frontend/Plan Frontend|Plan Frontend]]
- [[Frontend/Arquitectura Frontend|Arquitectura Frontend]]
