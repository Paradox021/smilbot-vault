# 🐾 Rediseño Visual: Identidad de Smilbot (Fase 1)

Definición de la nueva identidad visual de la web, antes de implementarla. Sustituye a la dirección basada en Raycast del `DESIGN.md` actual del frontend.

> Decidido el 2026-10-11. **Estado: implementado** (integrado en `master` del frontend el 2026-10-11). El `DESIGN.md` del frontend se ha reescrito a partir de esta nota y es la referencia para implementar; el detalle técnico está en [Arquitectura Frontend](./Arquitectura%20Frontend.md#identidad-visual). Esta nota conserva el porqué de las decisiones.
>
> Esta fase cambió el **aspecto** (paleta, tipografía, formas, mascota y login). La estructura de las pantallas, el movimiento y la voz de Smilbot son la [fase 2](./Rediseno%20Visual%20-%20Fase%202.md).

---

## 1. Por Qué

El sistema actual (copia del de Raycast: negro neutro, monocromo, CTA blanco, Inter) es limpio y profesional, pero es el de una herramienta de productividad. Smilbot es otra cosa:

- Es un **juego de colección de cartas de gatitos**.
- **Smilbot es un gato naranja**: la mascota y la marca son lo mismo.
- Conseguir una carta debe sentirse como una **pequeña recompensa**.

La personalidad no se consigue añadiendo paneles ni efectos, sino con cuatro cosas: **la mascota**, **una paleta cálida**, **tipografía y formas amables** y **las cartas como protagonistas**.

---

## 2. Principios

1. **Oscuro y cálido.** Fondo azul tinta, texto crema y naranja de marca. Nada de negro neutro ni de blanco puro.
2. **Las cartas mandan.** La interfaz es un escenario sobrio y las ilustraciones y los colores de rareza aportan el color. Los colores fuertes significan siempre rareza.
3. **El naranja es Smilbot.** Se usa para la acción principal, el logo y la mascota. Es escaso a propósito: si todo es naranja, nada destaca.
4. **Amable, no infantil.** Formas redondeadas, títulos con carácter y textos cercanos, pero con una interfaz que se lee y se usa sin esfuerzo.
5. **Mobile first.** La mayoría llega desde Discord en el móvil. La colección aprovecha el ancho en escritorio.

Se mantienen del sistema actual: escalera de superficies en lugar de sombras, animación solo con `transform` y `opacity` y respeto a `prefers-reduced-motion`.

---

## 3. Paleta

### Interfaz

| Token (CSS) | Rol | Color | Antes |
| :--- | :--- | :--- | :--- |
| `--background` | Fondo de la página | `#0D111C` | `#07080a` |
| `--card` | Superficie (tarjetas, nav inferior) | `#171E2D` | `#0d0d0d` |
| `--secondary` / `--popover` | Superficie elevada (menús, diálogos, botón secundario, hover) | `#1F2738` | `#101111` |
| `--muted` / `--accent` | Relleno suave (skeletons, pistas de progreso) | `#252E42` | `#121212` |
| `--border` | Borde sutil | `#283149` | `#242728` |
| `--input` | Borde de campos | `#36405A` | blanco al 16 % |
| `--foreground` | Texto principal | `#F3EEE7` (crema) | `#f4f4f6` |
| `--body` | Texto de párrafo | `#D9D3CA` | `#cdcdcd` |
| `--muted-foreground` | Texto secundario | `#A8A29A` | `#9c9c9d` |
| `--primary` | **Naranja Smilbot**: CTA principal, logo, foco | `#FFAD62` | `#ffffff` |
| `--primary-pressed` | Naranja pulsado o hover | `#F29A4A` | `#e8e8e8` |
| `--primary-foreground` | Texto sobre naranja | `#0D111C` | `#07080a` |
| `--ring` | Anillo de foco | `#FFAD62` al 60 % | `#9c9c9d` |
| `--success` | Éxito, saldo que sube | `#5CD49B` | `#59d499` |
| `--warning` | Racha en peligro | `#F5C451` | `#ffc533` |
| `--destructive` | Errores | `#FF7A70` | `#ff6161` |

**Contraste comprobado (WCAG):**
- Texto crema sobre el fondo: unos 16:1.
- Texto secundario sobre la superficie: unos 6,6:1.
- Texto oscuro sobre el naranja: unos 10,2:1.
- **El texto blanco sobre el naranja no pasa AA (1,8:1).** Por eso el botón principal lleva texto oscuro, no blanco como en el mockup.

### Rarezas (no cambian)

Son los colores que usa el bot en sus embeds de Discord (`src/types/CardType.ts`) y la web debe coincidir con él. Ver [Cartas y Rarezas](../Modelos%20y%20Contratos/Cartas%20y%20Rarezas.md).

| Rareza | Color | Contraste sobre `#0D111C` |
| :--- | :--- | :--- |
| Común | `#808080` | ~4,7:1 |
| Rara | `#0070dd` | ~3,9:1 |
| Épica | `#a335ee` | ~3,9:1 |
| Legendaria | `#ff8000` | ~7,5:1 |
| Mítica | `#c45039` | ~4,1:1 |

Todas superan el 3:1 que pide WCAG para bordes y elementos gráficos. La rareza nunca se comunica solo con color: siempre va también su nombre.

### Reglas de color
- **Sin violeta ni otros acentos de marca.** El violeta se confundiría con Épica. Las acciones secundarias van en superficie elevada con texto crema.
- **El naranja de marca nunca marca una carta.** `#FFAD62` (marca) y `#ff8000` (Legendaria) son de la misma familia, así que el naranja de marca no se usa en marcos, brillos ni insignias de carta. Una legendaria se reconoce por su marco, su nombre de rareza y sus efectos.
- **Sin degradados de marca ni neón.** El único brillo cálido es el halo de la mascota. Los brillos de color son los de las rarezas en la tirada.

---

## 4. Tipografía

| Uso | Fuente | Pesos | Paquete |
| :--- | :--- | :--- | :--- |
| Títulos, logo, cifras destacadas (saldo grande, racha) | **Nunito** (sans redondeada) | 700, 800 | `@fontsource-variable/nunito` |
| Texto, botones, etiquetas, tablas | **Inter** (con `ss03`) | 400, 500, 600 | `@fontsource-variable/inter` (ya instalado) |

- Nunito da el tono amable sin parecer infantil. Inter mantiene la legibilidad en textos pequeños.
- Los números que se comparan (saldo, contadores, cuenta atrás) usan siempre `tabular-nums`.

**Escala:**

| Nivel | Tamaño | Fuente |
| :--- | :--- | :--- |
| Título de login | 40-56 px | Nunito 800 |
| H1 de página | 28-32 px | Nunito 800 |
| H2 de sección o tarjeta | 18-20 px | Nunito 700 |
| Texto | 15-16 px | Inter 400 |
| Secundario | 13-14 px | Inter |
| Etiquetas | 12 px | Inter 500 |

El *eyebrow* en mayúsculas espaciadas del mockup ("TU COLECCIÓN DE GATITOS") solo se usa en el login.

---

## 5. Formas y Profundidad

| Elemento | Radio |
| :--- | :--- |
| Botones, chips, píldora del saldo, insignias, pestañas | Píldora (`9999px`) |
| Tarjetas y paneles de interfaz | 16 px |
| Cartas coleccionables (marco de rareza) | 12 px |
| Campos de formulario | 12 px |
| Menús y diálogos | 16 px |

- **Profundidad:** cada nivel es una superficie más clara (fondo, superficie, superficie elevada), no una sombra. Los bordes son de 1 px y sutiles, y solo se usan donde la superficie no basta.
- **Halo cálido:** se reserva para la mascota (degradado radial naranja muy suave detrás del círculo) y, en la tirada, para el brillo de la rareza.
- **Más aire:** menos tarjetas anidadas. Agrupar con espacio y títulos antes que con cajas.

---

## 6. La Mascota

Smilbot (el gato naranja) aparece donde la app tiene que **tener voz**, no como decoración repetida:

| Dónde | Cómo |
| :--- | :--- |
| Login | Grande, en círculo con halo cálido (pieza principal) |
| Estados vacíos (sin cartas, sin historial) | Mediana, con un texto cercano y una acción ("Abre tu primer sobre") |
| 404 y errores | Mediana ("Smilbot no encuentra esta página") |
| Logo de la cabecera y favicon | Pequeña, en círculo |

No aparece en las pantallas de datos (Inicio, Colección, Rankings) salvo en sus estados vacíos. **La fase 2 cambia esta regla:** Smilbot aparece también en Inicio y en la tirada como guía, con un bocadillo ([fase 2, sección 3](./Rediseno%20Visual%20-%20Fase%202.md#3-smilbot-como-guía)).

### La foto original

La única foto de Smilbot es de baja resolución (unos 490×456 px) y algo blanda. Opciones:

| Opción | Ventajas | Inconvenientes |
| :--- | :--- | :--- |
| **A. Foto tal cual, en tamaños pequeños** | Es el Smilbot real, el mismo avatar que el bot en Discord | Grande se ve pixelada |
| **B. Foto mejorada con un escalado por IA** (p. ej. Upscayl, gratuito y local, o Real-ESRGAN) a 2-4× | Sigue siendo la foto real y aguanta tamaños medianos (login a 240-320 px) | Hay que revisar que no invente rasgos (ojos, bigotes). No sirve para posturas nuevas |
| **C. Ilustración de la mascota** basada en la foto (encargada a un ilustrador o generada y retocada) | Escala a cualquier tamaño, permite poses (dormido, sorprendido, celebrando) para estados vacíos y tirada, y es la identidad más fuerte | Cuesta tiempo o dinero y debe parecerse a Smilbot |

**Decisión:** empezar con **B** (foto mejorada) para tener la identidad ya, y dejar **C** como mejora futura.

**Hecho (2026-10-11):** escalado 4× con Real-ESRGAN (modelo `realesrgan-x4plus`), de 493×456 a 1972×1824 px. Es fiel al original (sin rasgos inventados) y aguanta un círculo de 280 px en pantallas retina. Ficheros fuente en la bóveda:
- `Frontend/assets/smilbot-original.png`: la foto original.
- `Frontend/assets/smilbot-x4.jpg`: la versión mejorada, de la que salen los WebP de la web.

El encuadre del círculo incluye las orejas: los WebP salen de un recorte cuadrado de toda la altura de la foto, centrado en la cara (en una prueba con un recorte más cerrado se cortaban).

**Cómo usar la foto para que no quede rara:**
- **Siempre recortada en círculo** y centrada en la cara (como el avatar de Discord). Nunca a sangre ni en rectángulo grande.
- **Tamaño máximo de 280 px de lado en pantalla** (el doble de píxeles reales con la versión mejorada).
- **Halo naranja suave detrás** y un anillo fino crema u oscuro: integran la foto con el fondo y disimulan la blandura.
- Las versiones pequeñas (cabecera y favicon) salen de la foto original reducida, donde la calidad no se nota.
- **Ficheros:** `src/assets/mascot/` en WebP (`smilbot-128`, `-320`, `-640`) con `srcset`, y el favicon en PNG (32 y 180 px).

La imagen del mockup era un gatito generado por IA, no Smilbot: sirve de referencia de ambiente, pero la mascota es la foto real.

---

## 7. Componentes

| Componente | Definición |
| :--- | :--- |
| **Botón principal** | Píldora naranja `--primary` con texto `--primary-foreground`; hover y pulsado `--primary-pressed`. Uno por pantalla como máximo. |
| **Botón secundario** | Píldora en `--secondary` con texto crema y borde sutil. |
| **Botón fantasma** | Sin fondo, texto secundario; hover en superficie. |
| **Tarjeta de interfaz** | `--card`, radio de 16 px, sin sombra; borde solo si se apoya directamente sobre otra superficie. |
| **Carta coleccionable** | Igual que ahora (marco de 1 px del color de su rareza, imagen entera a su proporción, nombre debajo), con radio de 12 px y fondo `--card`. |
| **Píldora del saldo** | Superficie elevada, icono de moneda en naranja y cifra en Nunito. |
| **Insignias** | "Nueva" en naranja de marca, pero solo **fuera** del marco de la carta. Las demás en superficie elevada. |
| **Navegación** | Escritorio: pestañas en píldora (la activa en superficie elevada con texto crema). Móvil: barra inferior en `--card`, con el icono activo en naranja. |
| **Progreso** (pity, colección) | Pista `--muted` y relleno crema (o el color de la rareza si mide rarezas). |
| **Foco** | Anillo `--ring` (naranja al 60 %) en todos los controles. |

---

## 8. Pantallas

### Login (`/` sin sesión)
Solo cuatro elementos, sin nav, saldo ni cartas:
1. **Logo** (mascota pequeña y "Smilbot" en Nunito) arriba a la izquierda.
2. **Mascota** grande en círculo con halo.
3. **Texto:**
   - Eyebrow: "Tu colección de gatitos".
   - Título: "Un mundo de gatitos te espera."
   - Subtítulo: "Colecciona cartas, descubre rarezas y consigue las más difíciles."
4. **Botón** "Continuar con Discord" (principal, con el logo de Discord) y debajo, en pequeño: "Tu progreso va unido a tu cuenta de Discord."

**Disposición:**
- Escritorio: dos columnas, mascota a la izquierda y texto a la derecha.
- Móvil: mascota arriba (unos 200 px), texto y botón debajo, con el botón al alcance del pulgar.

**Ambiente:** cuatro o cinco motas de luz naranja fijas alrededor de la mascota (decoración estática, sin animación en bucle).

### Inicio
- **Saludo:** "Hola, {nombre}" en Nunito.
- **Daily:** sigue siendo la primera tarjeta y es la acción principal (botón naranja) cuando está disponible.
- **Tirada:** cuando el daily está en cooldown, el acceso "Abrir un sobre" pasa a ser el botón principal.
- **Últimas cartas:** se añade en la fase 3 (con la colección), a partir de los `CARD_BUY` del historial.
- Racha y mítica garantizada como ahora, con el nuevo estilo.

### Tirada
- La secuencia por rareza no cambia ([Plan Frontend](./Plan%20Frontend.md#5-tirada-fase-2)).
- El botón "Abrir sobre" pasa a ser naranja.
- El dorso de la carta lleva una huella en lugar del destello genérico.

### Perfil, Colección y Rankings
Mismas reglas: superficies, tarjetas de 16 px y cifras en Nunito. Los estados vacíos llevan a la mascota.

---

## 9. Tono de los Textos

- **Cercano y breve, en español de tú.** "Abre tu primer sobre", "Smilbot no encuentra esta página".
- **Un guiño gatuno de vez en cuando**, en estados vacíos y celebraciones (p. ej. "¡Miau! Carta nueva"), nunca en errores ni en textos de dinero.
- Los textos funcionales (botones, errores, saldo) siguen siendo claros y directos.

---

## 10. Qué Cambia en el Código

| Archivo | Cambio |
| :--- | :--- |
| `DESIGN.md` | Se reescribe como sistema propio de Smilbot a partir de esta nota (sustituye a la copia de Raycast). |
| `AGENTS.md` (sección "Diseño visual") | Deja de mencionar Raycast. |
| `src/index.css` | Tokens de la tabla de la sección 3 y radios de la sección 5. |
| `package.json` | Añadir `@fontsource-variable/nunito` (justificación: fuente de títulos de la identidad). |
| `shared/ui/button.tsx`, `badge.tsx`, `card.tsx` | Variantes con píldora, naranja y radios nuevos. |
| `app/layout/` | Logo con mascota, pestañas y barra inferior con el activo en naranja. |
| `features/auth/pages/LoginPage.tsx` | Nuevo login (sección 8). |
| `src/assets/mascot/`, `public/` | Mascota en WebP y favicon. |
| Pantallas existentes | Inicio, Perfil y Tirada revisados con el nuevo estilo; 404 y errores con la mascota. |

**Fuera de alcance:** tema claro (la web sigue siendo solo oscura).

### Orden de implementación
1. Tokens, fuentes y primitivas (`button`, `badge`, `card`). Con eso, casi todo cambia solo.
2. Mascota (assets), logo y favicon.
3. Login.
4. Repaso de cabecera, nav, Inicio, Perfil y Tirada, más 404, errores y estados vacíos.
5. Reescribir `DESIGN.md` y actualizar `AGENTS.md` y [Arquitectura Frontend](./Arquitectura%20Frontend.md).


Véase también:
- [[Frontend/Rediseno Visual - Fase 2|Rediseño Visual (fase 2)]]
- [[Frontend/Plan Frontend|Plan Frontend]]
- [[Frontend/Arquitectura Frontend|Arquitectura Frontend]]
- [[Modelos y Contratos/Cartas y Rarezas|Cartas y Rarezas]]
