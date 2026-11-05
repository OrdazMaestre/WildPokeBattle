# Arquitectura de WildPokeBattle

**Este documento es una propuesta de diseño, no una descripción de código existente.** Hoy el repositorio solo contiene páginas HTML y CSS. Lo único implementado de esta arquitectura es el marcado de la [sección 3.1](#31-marcado-implementado) y la capa vacía `#wpb-layer`.

Las reglas de juego están en [SPEC_GAMEPLAY.md](SPEC_GAMEPLAY.md). El diseño v1.3 completo se puede recuperar con `git show 6ecb531:README.md`.

---

## 1. Capas previstas

La idea fundamental es **compartir el núcleo de simulación entre web y escritorio**.

| Capa | Responsabilidad |
|---|---|
| **Battle Palace Core** | Lógica de combate, Pokémon, estadísticas, IA, movimientos, PP, cooldown y simulación |
| **Web Runtime / Web Adapter** | Adaptar el DOM y los elementos HTML al escenario físico |
| **Renderer** | Sprites, animaciones y representación visual |
| **Backend** | Cuentas, equipos, historial, matchmaking, rankings y sincronización online |
| **Desktop Runtime / Desktop Adapter** | Futura integración con el escritorio y aplicaciones externas |

```text
Web Adapter ─────┐
Renderer ────────┼──► Battle Palace Core
Desktop Adapter ─┘
```

El Core no conoce el DOM, el navegador ni el sistema operativo: recibe superficies y datos como parámetros. Así se puede probar sin navegador y reutilizar en escritorio. Falta decidir qué papel tiene el Backend en la simulación ([sección 8](#8-online)).

### 1.1. Equivalencia con los módulos de v1.3

| Módulo v1.3 | Capa actual | ¿Toca el DOM? |
|---|---|---|
| `core`, `physics`, `battle`, `ai`, `data` | Battle Palace Core | No |
| `bridge` (DOM Bridge) | Web Adapter | Sí |
| `ui` (selector, editor, atajos, menú contextual) | Web Runtime | Sí |
| `render` | Renderer | Sí |
| `desktop` | Desktop Adapter | Plataforma |

---

## 2. Tecnología

**Confirmado por el repositorio:** HTML semántico y CSS con custom properties, sin dependencias ni compilación.

**Sin confirmar.** Lo siguiente viene del diseño v1.3 y debe revisarse con el nuevo alcance. Nada de ello existe en el repositorio.

| Tecnología | Uso previsto |
|---|---|
| JavaScript vanilla con módulos ES | Motor, física, IA, combate, datos y adaptador web |
| JSDoc + `// @ts-check` | Tipado sin compilación |
| `node --test` | Tests del núcleo |
| Pokémon Showdown (Gen 9) | Datos de mecánicas, solo en scripts de build |
| PokéAPI (CSV) | Nombres en español, solo en scripts de build |
| PMD SpriteCollab | Candidato principal de sprites |

**Motivos dados en v1.3 para evitar un motor gráfico:**

- Pintar los Pokémon en el DOM y no en un `<canvas>` hace que mundo físico y página compartan las coordenadas de documento.
- La física necesaria eran 2 cuerpos contra unas decenas de rectángulos. Un motor AABB propio de unas 200 líneas parecía más sencillo que sincronizar otro motor con el DOM. El formato 2 contra 2 sube el número de cuerpos, pero sigue siendo poco.
- Sin build para jugar: basta un servidor estático (Live Server, `npx serve`), porque los módulos ES no funcionan desde `file://`.

**Sin decidir:** motor propio o librería (Phaser, Matter.js), tecnología del backend y de la sincronización (por ejemplo, PostgreSQL o WebSockets) y tecnología de escritorio (Electron, Tauri u otra). Son ejemplos, no decisiones.

---

## 3. Web Adapter

### 3.1. Marcado (implementado)

Los elementos físicos se declaran con atributos `data-` en el HTML. Es marcado, no lógica: **todavía no hay código que lo lea.**

```html
<header data-wpb-body="solid">…</header>
<article class="card" data-wpb-body="platform">…</article>
<aside data-wpb-body="wall">…</aside>
<button data-wpb-body="platform" data-wpb-enabled="false">…</button>
```

| Valor | Comportamiento previsto |
|---|---|
| `solid` | Colisiona por todos los lados |
| `platform` | Se aterriza encima y se atraviesa desde abajo |
| `wall` | Solo bloqueo lateral |
| *(sin atributo)* | No participa |

Orientación: contenedores, pared o sólido; tarjetas, plataforma; botones, plataforma u obstáculo.

El marcado está en [pages/prado.html](../pages/prado.html), y los detalles de la implementación actual en la [chuleta](CHULETA.md).

### 3.2. Tres conceptos que no se mezclan

| Concepto | Qué es | Quién lo gestiona |
|---|---|---|
| **Contenido** | Texto, imagen o control | HTML |
| **Caja visual** | Geometría que pinta el navegador | CSS |
| **Caja física** | Rectángulo con el que colisiona el motor | Web Adapter y física |

Tener contenido no implica tener caja física. Si un elemento desactiva su caja, sigue viéndose pero deja de existir para la simulación.

### 3.3. Capa de sprites

`<div id="wpb-layer">` ya existe, vacío, con `pointer-events: none`, para que los Pokémon no tapen los clics. Los sprites se moverían con `transform: translate3d()`. Si un sprite necesita ser interactivo, solo ese elemento activa `pointer-events: auto`.

### 3.4. DOM Bridge (propuesta)

1. Buscar elementos `[data-wpb-body]` con caja activa.
2. Medir todos en una sola pasada de lectura (`getBoundingClientRect` + scroll) para evitar *layout thrashing*.
3. Crear, actualizar o retirar los cuerpos estáticos.
4. Volver a medir solo ante cambios: `ResizeObserver`, `MutationObserver` (atributos `data-wpb-*`, nodos), `resize` y carga de fuentes o imágenes.
5. Ignorar elementos ocultos (`display: none`, tamaño 0).

Casos que hay que cubrir:

- `ResizeObserver` avisa de cambios de tamaño, no de posición. Se observa también el contenedor de la página y se vuelve a medir todo el conjunto.
- Los elementos `fixed` o `sticky` cambian de coordenadas de documento con el scroll.
- Una animación CSS con `transform` no dispara observadores. Por eso las superficies físicas no usan `transform`, `translate`, `scale`, `rotate`, `fixed` ni `sticky`, tampoco en `:hover`.

### 3.5. Tamaño del campo de batalla

**Confirmado:** el campo se adapta al espacio disponible en el navegador, y una ventana más pequeña implica un campo más pequeño. Falta decidir cómo se reconcilia con el online ([sección 8](#8-online)).

### 3.6. Configuración por el usuario (propuesta, Fase 7)

Activar y desactivar cajas, cambiar tamaños y tipo de superficie desde un menú de clic derecho. El marcado `data-wpb-*` ya lo permite sin rehacer las páginas.

---

## 4. Física (propuesta)

- **Sin atravesar superficies finas:** la velocidad por tick nunca supera el grosor mínimo de una superficie. Si un retroceso lo exige, se usa barrido (*swept AABB*).
- **Plataforma de un sentido:** solo se aterriza si la base del cuerpo estaba por encima en el tick anterior y baja.
- **Superficie que cambia:** si desaparece, el Pokémon cae. Si se mueve, se recalcula el contacto, y si el cuerpo queda dentro de otra caja se le expulsa por el lado de menor penetración.
- **Salto base:** el desnivel máximo del prado es de unos 140 px en escritorio y 160 px a 360 px de ancho, y el salto debe alcanzarlo.

---

## 5. Bucle de juego (propuesta)

- `requestAnimationFrame` con **paso fijo de física** (1/60 s con acumulador) para obtener resultados estables con cualquier tasa de refresco.
- El acumulador tiene tope (por ejemplo, 5 pasos por frame), para evitar una espiral de actualizaciones.
- Orden por tick: IA decide, física integra (X, resolver; Y, resolver), combate aplica impactos, eventos.
- El render va fuera del paso fijo: una vez por frame, interpolando entre los dos últimos estados.
- Las animaciones avanzan con el reloj del motor, no con CSS: el render cambia de frame según las duraciones de `AnimData.xml`, en 1/60 s y variables por frame. Así la pausa las congela sin más código, y `HitFrame` puede sincronizar el impacto.
- Cada subsistema usa su propio flujo de RNG con semilla. La reproducibilidad exacta se exige en Node con superficies fijas.
- **Pausa:** el bucle guarda un conjunto de motivos (`pauseReasons`) y solo avanza si está vacío. `manual` lo controla `Alt + P`; `hidden`, `visibilitychange`. Volver a la pestaña no reanuda una pausa manual.
- Al reanudar se descarta el tiempo acumulado, y las ediciones encoladas se aplican en bloque en el primer tick.

---

## 6. Datos de combate (propuesta)

[Pokémon Showdown](https://github.com/smogon/pokemon-showdown) (código MIT) se usaría **como fuente de datos, no como simulador**: su motor es por turnos.

| Dato de Showdown | Uso |
|---|---|
| Pokédex (stats base, tipos, habilidades) | Estadísticas de cada especie |
| Moves (potencia, precisión, categoría, prioridad, objetivo, flags, efectos) | Daño, dificultad de ejecución y forma física |
| Learnsets | Movimientos configurables |
| Sets de Random Battle (`data/random-battles/gen9/sets.json`) | Roles coherentes. Su nivel por especie se ignora: aquí siempre es 100 |
| Typechart y naturalezas | Efectividades y modificadores |
| Fórmula de stats | Valores finales a partir de base, IV, EV, nivel y naturaleza |
| Formato de exportación de sets | Importar y exportar Pokémon |

Integración prevista:

- Solo en `scripts/build-data.js` (Node). Se leen los datos con `@pkmn/dex` y `@pkmn/data` (MIT) y se genera JSON estático en `js/data/generated/`, versionado para poder jugar sin ejecutar el build.
- El conjunto aprendible se calcula con `canLearn` de `@pkmn/data`.
- En el navegador no hay dependencias: `main.js` carga el JSON y lo inyecta en el núcleo.
- El parser de sets es propio y pequeño; los tests lo comparan con `@pkmn/sets`.
- Se registra la versión de los paquetes y el commit de Showdown usados.
- Cambiar de generación requiere un ADR.

Los PP, los tiers y los equipos originales de entrenadores no se han estudiado todavía como datos de entrada. Los detalles de las fuentes están en el [informe](INFORME_INVESTIGACION.md#2-apis-y-fuentes-de-datos).

---

## 7. Renderer (propuesta)

- Sprites y animaciones en la capa `#wpb-layer`, con una caja física independiente del dibujo.
- Sprite sheets y no GIF, para que la pausa pueda congelarlos.
- Sin sprites disponibles, el juego usa siluetas.
- La cámara de batalla está pendiente de diseño técnico: no se sabe aún en qué capa vive ni qué guarda.

---

## 8. Online

**Requisitos confirmados** (el diseño completo está pendiente):

- Cuentas de usuario.
- Los dos jugadores comparten un enfrentamiento lógico verificable.
- No se depende de que cada navegador calcule resultados independientes sin sincronización.
- Se contempla sincronización de datos y aleatoriedad compartida.
- Desafíos pendientes que se juegan cuando cada jugador quiera.
- Historial por jugador y por equipo. Ranking mundial y regional, como planteamiento.

**Cuestiones técnicas abiertas:**

- **Campo y sincronía.** La física depende de las superficies, y estas del tamaño de la ventana de cada jugador. Hay que decidir cómo se comparte un enfrentamiento verificable entre ventanas distintas. La reproducibilidad planteada en v1.3 solo valía con superficies fijas.
- **Autoridad.** Quién calcula y quién verifica el resultado: servidor, clientes con comprobación mutua u otra opción.
- **Desafíos asíncronos.** Cómo encajan con un enfrentamiento compartido.
- **Datos personales.** Las cuentas y el ranking regional obligan a definir qué se guarda y cómo se protege.
- **Sprites en un servicio público.** Ver los [recursos del README](../README.md#9-recursos-y-licencias).

---

## 9. Desktop Adapter (futuro)

Es una evolución de la web. **No se ha iniciado**, y la prioridad es terminar la versión web. No están decididos el sistema operativo inicial ni la tecnología.

La idea es sustituir o ampliar **la fuente de superficies**: en lugar de leer el DOM, un adaptador lee ventanas y elementos del sistema y entrega los mismos rectángulos al Core. El adaptador convierte todo a un único sistema de coordenadas, teniendo en cuenta varios monitores y el escalado de pantalla.

Hay que estudiar las **APIs de accesibilidad** de cada sistema: no todas las aplicaciones exponen sus elementos de interfaz de la misma forma. La comparación por sistema y el alcance propuesto están en el [informe](INFORME_INVESTIGACION.md#6-escritorio).

**Notas si se estudia Electron** (candidato, no decisión):

- Ventana transparente, sin bordes, siempre visible y con paso de clics (`setIgnoreMouseEvents(true, { forward: true })`; `forward` solo existe en macOS y Windows).
- `globalShortcut` retiene la combinación para las demás aplicaciones, y si otra la tiene el registro falla sin error claro. Debe ser configurable y comprobarse.
- Con paso de clics la ventana no recibe teclado: hay que desactivarlo al pausar y restaurarlo al reanudar.
- `backgroundThrottling` debe desactivarse para no ralentizar la simulación sin foco.

---

## 10. Estructura de carpetas

Lo marcado como *previsto* aún no existe. Cada carpeta se crea al llegar a su fase.

```text
WildPokeBattle/
├── index.html
├── pages/                     prado, pokedex, ayuda
├── css/                       tokens, base, components, scenes/
├── assets/
│   ├── ASSETS.md              origen y licencia de cada recurso
│   └── sprites/               previsto: local, fuera de Git
├── docs/
│   ├── ARCHITECTURE.md
│   ├── SPEC_GAMEPLAY.md
│   ├── INFORME_INVESTIGACION.md
│   ├── CHULETA.md
│   └── decisions/             previsto: ADR, una decisión por archivo
├── js/                        previsto: main.js, core/, physics/, battle/, ai/,
│                              data/, bridge/, render/, ui/
├── scripts/                   previsto: build-data.js, fetch-assets.js
├── tests/                     previsto
├── desktop/                   previsto
├── package.json               previsto
├── .gitignore                 previsto: node_modules/, assets/sprites/
├── CREDITS.md                 previsto: autores de sprites
├── LICENSE                    previsto: solo el código propio
├── AGENTS.md
└── README.md
```

---

## 11. Plan técnico por fases

El resumen está en el [README](../README.md#7-próximas-fases). Se conserva la numeración de v1.3 porque el informe y la chuleta la citan. Las fases nuevas llevan letra.

Cada fase tiene un «Hecho cuando» comprobable. No se empieza una fase sin cerrar la anterior.

### Fase 1 — Estructura HTML ✅

Web navegable y semántica, sin JS: `index.html`, prado, Pokédex y ayuda, selector con `<dialog popover>`, maqueta del editor en `prado.html#editor` y superficies marcadas.

**Hecho cuando:** se navega sin enlaces rotos, el HTML valida en el W3C y el prado tiene al menos un suelo, dos plataformas y una pared marcados.

### Fase 2 — Diseño visual ✅

Tokens, layout responsive, componentes, identidad del prado y capa reservada. Estilo de cuaderno de campo, con fuentes del sistema y arte propio en CSS y SVG.

**Hecho cuando:** el prado es reconocible, funciona de 360 px a escritorio y el CSS no contiene lógica física.

### Fase 3 — Motor físico con caja de prueba ⬅ siguiente

Va antes que los datos a propósito: valida lo más arriesgado con un rectángulo. Bucle con paso fijo, pausa, gravedad, aceleración, fricción, salto, colisiones AABB, plataformas de un sentido, DOM Bridge, vuelo básico y tests en Node.

**Hecho cuando:** un rectángulo cae, aterriza, salta y choca en el prado; al redimensionar, desplazar un elemento o desactivar un `data-wpb-body`, la física se actualiza; la pausa lo congela y lo reanuda sin saltos; volver a la pestaña no reanuda una pausa manual; pasan los tests de física.

### Fase 4 — Datos Pokémon, selector y ritmo

`scripts/build-data.js` genera JSON para un conjunto acotado de especies. `scripts/fetch-assets.js` descarga sprites y créditos. Selector funcional con Pokémon a nivel 100, sprite o silueta.

Incluye un spike de ritmo sin interfaz: un script en Node enfrenta especies con daño, PP y cooldowns, sin física, para fijar `battleScale`, `TURN_MS` y los primeros cooldowns.

**Hecho cuando:** se elige un Pokémon y se comporta como la caja de la Fase 3 con su sprite; funciona sin conexión con los sprites descargados y muestra siluetas sin ellos; el spike da una duración dentro del objetivo.

### Fase 5 — Movimiento autónomo

Caminar, detenerse, girar, saltar obstáculos, explorar y no atascarse. IA separada de la física, con la base del sistema de inteligencia.

**Hecho cuando:** un Pokémon solo explora el prado 5 minutos sin atascarse y dos especies se mueven de forma visiblemente distinta.

### Fase 6 — Combate

Objetivo, posicionamiento, ataques, PS y daño, tipos, PP, cooldown, preparación, ejecución, recuperación, retroceso, debilitamiento, final de combate y victoria o derrota.

**Hecho cuando:** dos Pokémon combaten sin turnos hasta que uno cae, los tipos alteran el resultado, un combate con la misma semilla es reproducible en tests y un combate entre Pokémon que no pueden dañarse también termina.

### Fase 6B — Preparación de equipos

Configuración de estadísticas, naturaleza, movimientos, objetos y demás parámetros, en equipos de hasta seis. Importación y exportación en formato Showdown. Sustituye al antiguo «editor en pausa», cuyo papel debe revisarse.

**Hecho cuando:** se prepara un equipo y combate con esa configuración; un set exportado en modo legal se pega en Showdown sin errores; un set con movimientos excluidos se rechaza con un aviso.

### Fase 7 — Combate avanzado y escenarios

Proyectiles, área, esquivas, estados alterados, capacidades físicas, superficies destruibles, menú de clic derecho y nuevos escenarios.

### Fase 7B — Modos locales y wiki

Combates salvajes (1 contra 1 y 2 contra 2, filtros combinables), entrenadores NPC con sus equipos originales y wiki de Pokémon, movimientos, equipos y documentación.

### Fase 7C — Online

Cuentas, equipos guardados, desafíos, enfrentamiento verificable, historial, revanchas y ranking. Requiere cerrar antes las [cuestiones abiertas](#8-online).

### Fase 7D — Cámara de batalla

Guardado local de hasta dos minutos de momentos destacados. Requiere su diseño técnico.

### Fase 8 — Escritorio

Solo cuando la experiencia web esté completa. Primero, investigación de plataforma y APIs de accesibilidad.

### Fase 9 — Ampliación

Más especies, más escenarios, mejoras de IA, rendimiento y estabilidad. Puede avanzar en paralelo a otras fases.

### Criterios globales

- [ ] Gravedad, salto, plataformas, paredes y límites funcionan desde la primera aparición.
- [ ] El vuelo está contemplado en la arquitectura.
- [ ] Los Pokémon pueden separarse sin que se cancele el combate.
- [ ] Todo combate termina, incluso entre Pokémon que no pueden dañarse.
- [ ] La IA actúa sola, y estadísticas y tipos cambian el comportamiento.
- [ ] Los combates respetan los límites de su formato.
- [ ] Los ataques producen efectos físicos coherentes.
- [ ] Los elementos marcados actúan como superficies, y desactivar su caja los saca de la física sin ocultarlos.
- [ ] Los cambios de tamaño y posición se reflejan en la simulación.
- [ ] La página sigue siendo usable (clics, selección de texto) con los Pokémon encima.
- [ ] El Core funciona fuera del navegador y se reutiliza en escritorio sin cambios.
- [ ] Cada recurso usado está registrado, con autor, procedencia y licencia.

---

## 12. Riesgos técnicos

- **Sincronización DOM y física:** scroll, zoom, redimensionado, elementos dinámicos y cambios de visibilidad. Se mitiga con coordenadas de documento y observadores, y necesita pruebas manuales en la Fase 3.
- **Rendimiento:** el riesgo real es medir el DOM en cada frame. Hay que medir solo ante cambios y limitar la física a elementos marcados. Objetivo: 60 fps estables en un portátil medio.
- **Equilibrio:** las estadísticas oficiales son referencia; tiempos, cooldowns, alcance y daño necesitan reglas propias y pruebas.
- **Aplicaciones externas:** el navegador solo ve su propio DOM. En escritorio harán falta APIs del sistema y permisos, y la compatibilidad variará.
