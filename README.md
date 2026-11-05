# WildPokeBattle (Battle Palace)

> Simulación de combates Pokémon autónomos en tiempo real, en escenarios físicos construidos con la propia web.

| | |
|---|---|
| **Estado** | Fases 1 y 2 terminadas: web estática en HTML y CSS. **Todavía no hay JavaScript, motor físico, combate ni Pokémon en movimiento** |
| **Siguiente paso** | Fase 3: motor físico con una caja de prueba |
| **Plataforma** | Web primero. Escritorio como evolución futura, sin iniciar |
| **Versión del documento** | 2.0 (5 nov 2026) |

## Índice

1. [Presentación](#1-presentación)
2. [Características y visión](#2-características-y-visión)
3. [Decisiones de diseño confirmadas](#3-decisiones-de-diseño-confirmadas)
4. [Modos de juego](#4-modos-de-juego)
5. [Arquitectura prevista](#5-arquitectura-prevista)
6. [Estado actual](#6-estado-actual)
7. [Próximas fases](#7-próximas-fases)
8. [Decisiones pendientes](#8-decisiones-pendientes)
9. [Recursos y licencias](#9-recursos-y-licencias)
10. [Documentación](#10-documentación)

---

## 1. Presentación

WildPokeBattle, también denominado Battle Palace, es un proyecto de simulación de combates Pokémon autónomos en tiempo real.

Los Pokémon se mueven por escenarios bidimensionales de perspectiva lateral, con gravedad, saltos, plataformas y colisiones. El jugador prepara, configura y optimiza sus equipos, pero **no controla a los Pokémon durante la batalla**.

Su rasgo central es que los Pokémon interactúan físicamente con elementos reales de la interfaz web, y los usan como superficies, plataformas y obstáculos.

**Cómo leer este README.** Las secciones 2 a 5 describen lo que se quiere construir, no lo que ya existe. Lo implementado está únicamente en la [sección 6](#6-estado-actual).

| Etiqueta | Significado |
|---|---|
| **Implementado** | Existe en el repositorio |
| **Confirmado** | Decisión tomada, todavía sin construir |
| **Propuesta** | Idea de diseño que puede cambiar |
| **Pendiente** | Sin decidir |

---

## 2. Características y visión

Todas estas características son objetivos del proyecto.

- **Combates autónomos en tiempo real**, sin turnos.
- **La web como escenario:** cada página es un terreno físico.
- **Preparación de equipos:** configurar y optimizar es una parte importante del juego.
- **Inteligencia de los Pokémon:** influye en cómo actúan y en qué deciden.
- **Movimientos con comportamiento propio**, más allá de sus datos oficiales.
- **Combates salvajes, entrenadores NPC y multijugador online.**
- **Cámara de batalla** con los momentos más destacados.
- **Wiki integrada:** la web es también la fuente de información de Pokémon, movimientos, equipos y documentación.

**A largo plazo** se quiere una aplicación de escritorio en la que los Pokémon interactúen con el escritorio, las ventanas y las aplicaciones. Esa etapa llegará después de completar la experiencia web.

---

## 3. Decisiones de diseño confirmadas

El detalle y las propuestas relacionadas están en [SPEC_GAMEPLAY.md](docs/SPEC_GAMEPLAY.md).

### Pokémon y equipos

- Todos los Pokémon están al **nivel 100**. La subida de nivel tradicional no es una mecánica central.
- Se pueden configurar estadísticas, naturaleza, movimientos, objetos y otros parámetros de combate.
- Los equipos tienen hasta **seis Pokémon**.
- Los combates son automáticos.

### Inteligencia

El sistema de inteligencia se inspira en parte en Pokémon Mundo Misterioso. Debe influir en:

- Elegir el momento adecuado para atacar.
- Gestionar los movimientos que requieren preparación.
- Adaptarse a la situación del combate.
- Aprovechar oportunidades y posicionamiento.
- Tomar decisiones estratégicas.

Los legendarios tienen inteligencia máxima y ventajas o comportamientos personalizados. La escala de inteligencia está pendiente de concretar.

<a id="45-combate"></a>

### Movimientos

La precisión oficial **no se traduce en una probabilidad tradicional de acertar**. Sirve como referencia para definir la dificultad de ejecutar correctamente un movimiento. Por ejemplo, Rayo Solar tiene una precisión oficial del 100 %, pero requiere preparación y exposición antes de ejecutarse.

Cada movimiento tiene comportamiento propio: dos movimientos con la misma precisión no tienen por qué funcionar igual. Además, se distinguen estos conceptos:

| Concepto | Qué es |
|---|---|
| **PP** | Número de usos disponibles |
| **Cooldown** | Tiempo de espera antes de volver a usar el movimiento |
| **Preparación o carga** | Fase previa al ataque |
| **Ejecución** | El ataque en sí |
| **Recuperación** | Fase posterior al ataque |

Cada movimiento tiene su propio cooldown, además de sus PP. La propuesta para derivar la forma física de los movimientos a partir de los datos está en la [especificación](docs/SPEC_GAMEPLAY.md#43-forma-de-un-movimiento-propuesta).

### Escenarios web

- Las páginas web son escenarios físicos, con distribuciones y ambientes diferentes.
- Los elementos de la interfaz actúan como plataformas, superficies y obstáculos.
- El campo de batalla se adapta al espacio del navegador: una ventana más pequeña implica un campo más pequeño.
- El sistema identifica qué elementos participan en la simulación. Ese marcado (`data-wpb-body`) ya existe en el prado, aunque aún no hay código que lo lea.

### Cámara de batalla

Se quiere una cámara que guarde **localmente** hasta dos minutos de los momentos más destacados de una partida, seleccionados principalmente por el daño por segundo acumulado. Así se conservan los momentos importantes sin almacenar toda la partida.

No está implementada, y su funcionamiento técnico y sus criterios exactos están pendientes de diseño.

---

## 4. Modos de juego

Ningún modo está implementado.

### Combates salvajes

- Pokémon seleccionados o aleatorios.
- Formatos 1 contra 1 y 2 contra 2.
- Filtros combinables por tiers, legendarios, tipos, Pokémon concretos y otras categorías.
- Los filtros de tiers pertenecen exclusivamente a este modo.

### Entrenadores NPC

- Entrenadores clásicos de Pokémon, conservando en lo posible sus equipos originales.
- Si un entrenador tiene equipos distintos según su aparición, se pueden registrar variantes.
- No se inventan Pokémon para completar equipos que originalmente tenían menos integrantes.

### Multijugador online

- Cuentas de usuario y juego online.
- Los dos jugadores comparten un **enfrentamiento lógico verificable**. No se depende de que cada navegador calcule resultados independientes sin sincronización.
- Se contempla la sincronización de datos y la aleatoriedad compartida.
- Desafíos pendientes que se juegan cuando cada jugador quiera.
- Historial de combates por jugador y por equipo.
- Se plantea un ranking mundial y regional.

**Revanchas tras un empate.** Cada jugador decide por su cuenta si quiere revancha:

| Decisión | Resultado |
|---|---|
| Ambos aceptan | Juegan sus respectivas revanchas |
| Solo uno acepta | Ese jugador puede obtener un resultado definitivo para sí mismo, y el otro conserva el empate sin resolver |
| Ambos rechazan | El empate sigue sin resolverse |

Si vuelve a producirse un empate, se pueden ofrecer nuevas revanchas. La puntuación de los empates no resueltos está sin definir.

---

## 5. Arquitectura prevista

**Es una propuesta de diseño, no una implementación existente.** La idea fundamental es compartir el núcleo de simulación entre web y escritorio.

| Capa | Responsabilidad |
|---|---|
| **Battle Palace Core** | Lógica de combate, Pokémon, estadísticas, IA, movimientos, PP, cooldown y simulación |
| **Web Runtime / Web Adapter** | Adaptación del DOM y de los elementos HTML al escenario físico |
| **Renderer** | Sprites, animaciones y representación visual |
| **Backend** | Cuentas, equipos, historial, matchmaking, rankings y sincronización online |
| **Desktop Runtime / Desktop Adapter** | Futura integración con el escritorio y aplicaciones externas |

Del stack solo están confirmados, porque existen en el repositorio, HTML y CSS sin dependencias. El diseño anterior proponía JavaScript vanilla con un motor propio y datos de Pokémon Showdown y PokéAPI. Sigue siendo la propuesta de partida, pero debe revisarse con el nuevo alcance. Ninguna otra tecnología está decidida.

**Escritorio.** No se ha iniciado, y la prioridad es terminar la web. Habrá que estudiar las APIs de accesibilidad del sistema operativo, porque no todas las aplicaciones exponen su interfaz de la misma forma.

Todo el detalle está en [ARCHITECTURE.md](docs/ARCHITECTURE.md).

---

## 6. Estado actual

| Área | Estado |
|---|---|
| Páginas | **Implementado:** portada, [prado](pages/prado.html), [Pokédex](pages/pokedex.html) y [ayuda](pages/ayuda.html), con HTML semántico y sin JavaScript |
| Diseño visual | **Implementado:** CSS con tokens, componentes y escena del prado, adaptable de 360 px a escritorio. Estilo de cuaderno de campo |
| Superficies físicas | **Solo marcado:** el prado declara 9 elementos con `data-wpb-body` (2 `solid`, 6 `platform`, uno de ellos desactivado, y 1 `wall`). Nada los lee todavía |
| Capa de Pokémon | **Solo estructura:** `#wpb-layer` existe y está vacía |
| Selector | **Parcial:** se abre y se cierra, pero sus 12 especies están desactivadas |
| Editor | **Maqueta:** visible en `prado.html#editor`, con los campos desactivados |
| Pokédex | **Contenido estático:** las 12 especies propuestas, sin sprites |
| Motor, combate, IA y datos | No existen |
| Modos de juego, wiki real, cuentas y cámara | No existen |
| Backend y escritorio | No existen |
| Sprites | No hay ninguno en el repositorio. El arte actual es CSS y SVG propio |

Tampoco existen aún `js/`, `tests/`, `scripts/`, `package.json`, `.gitignore`, `LICENSE` ni `CREDITS.md`.

La [chuleta](docs/CHULETA.md) resume la web construida y registra sus comprobaciones: validación W3C, enlaces, contraste y anchos, hechas solo en Edge.

> **Textos desactualizados en la web.** Las páginas conservan mensajes del concepto anterior: Pokémon con estadísticas aleatorias, un máximo de 2 activos o nivel 50 en la maqueta del editor. Prevalecen las decisiones de este README, y esos textos se actualizarán en una tarea aparte.

**Probarlo:** abre la carpeta con Live Server o ejecuta `npx serve`, y entra en `index.html`.

---

## 7. Próximas fases

La web va antes que el escritorio. **El orden de las fases aún no terminadas es una propuesta.** Se conservan los números del plan anterior, y las fases nuevas llevan letra.

| Fase | Contenido | Estado |
|---|---|---|
| 1 | Estructura HTML | Terminada |
| 2 | Diseño visual con CSS | Terminada |
| 3 | Motor físico con caja de prueba y lectura de `data-wpb-body` | **Siguiente** |
| 4 | Datos Pokémon, selector, sprites y prueba de ritmo | Sin empezar |
| 5 | Movimiento autónomo y base de la inteligencia | Sin empezar |
| 6 | Combate: daño, tipos, PP, cooldown y final de combate | Sin empezar |
| 6B | Preparación de equipos e importación de sets | Sin empezar |
| 7 | Combate avanzado y nuevos escenarios | Sin empezar |
| 7B | Combates salvajes, entrenadores NPC y wiki | Sin empezar |
| 7C | Multijugador online | Sin empezar |
| 7D | Cámara de batalla | Sin empezar |
| 8 | Aplicación de escritorio, cuando la web esté completa | Sin empezar |
| 9 | Ampliación continua: especies, escenarios, IA y rendimiento | Sin empezar |

Los criterios de «Hecho cuando» de cada fase están en [ARCHITECTURE.md](docs/ARCHITECTURE.md#11-plan-técnico-por-fases).

---

## 8. Decisiones pendientes

**Juego**

- Escala de inteligencia y ventajas concretas de los legendarios.
- Conversión de la precisión oficial en dificultad de ejecución.
- Valores de PP y cooldown, y qué ocurre al agotarse los PP.
- Cuántos Pokémon de cada equipo combaten a la vez y cómo entra el siguiente, fuera del formato salvaje.
- Formatos de los combates contra entrenadores y online.
- Qué objetos hay y cómo funcionan en tiempo real.
- Cómo se generan los Pokémon aleatorios de los combates salvajes.
- Qué sistema de tiers se usa.
- Si la pausa con `Alt + P`, el editor en pausa y el modo espectador móvil del diseño anterior siguen vigentes.

**Online**

- Modelo de sincronización y quién calcula o verifica el resultado.
- Cómo compartir un enfrentamiento verificable si el campo depende del tamaño de cada ventana.
- Cómo encajan los desafíos pendientes con el enfrentamiento compartido.
- Puntuación de empates no resueltos y funcionamiento del ranking.
- Datos personales de las cuentas.

**Cámara de batalla**

- Funcionamiento técnico, formato de guardado y criterios exactos de selección.

**Técnica y publicación**

- Lenguaje, motor físico y tecnología del backend.
- Sistema operativo inicial y tecnología de escritorio.
- Fuentes definitivas de sprites, y cómo publicar una versión online si los sprites no se redistribuyen.
- Licencia del código propio.
- Nombre público del proyecto: WildPokeBattle o Battle Palace.

---

<a id="114-recursos-y-licencias"></a>

## 9. Recursos y licencias

**Prioridad visual.** Los sprites y animaciones de Pokémon Mundo Misterioso: Exploradores del Cielo, o recursos similares que cubran el mayor número de especies posible. Se podrán estudiar otras fuentes:

- Otros juegos de Pokémon Mundo Misterioso.
- Recursos comunitarios de estilo similar.
- Sprites de generaciones anteriores.
- Recursos originales para cubrir los huecos.

PMD SpriteCollab es la **fuente candidata** estudiada hasta ahora. El análisis comparativo está en el [informe](docs/INFORME_INVESTIGACION.md#3-sprites-y-licencias).

**Reglas**

- Hay que mantener un inventario con autor, procedencia y licencia de cada recurso, en [assets/ASSETS.md](assets/ASSETS.md).
- No todos los recursos de un repositorio comunitario tienen la misma licencia. Hay que revisarlos uno a uno. Parte del material de SpriteCollab procede de los juegos oficiales, y su licencia no puede autorizar ese material.
- El uso académico no implica permiso para redistribuir o monetizar recursos protegidos.
- Los sprites no se suben al repositorio. Se prevé descargarlos en local, y sin ellos el juego usaría siluetas.
- Proyecto de fans sin uso comercial: sin anuncios, donaciones ni ventas. Puede tener que cerrarse si los titulares lo piden. Los antecedentes y las reglas de publicación están en el [informe](docs/INFORME_INVESTIGACION.md#7-riesgo-legal-y-publicación).

---

## 10. Documentación

| Documento | Contenido |
|---|---|
| [docs/SPEC_GAMEPLAY.md](docs/SPEC_GAMEPLAY.md) | Reglas de juego, propuestas heredadas y cambios frente al diseño anterior |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Arquitectura propuesta, adaptador web, datos, online, escritorio y plan por fases |
| [docs/INFORME_INVESTIGACION.md](docs/INFORME_INVESTIGACION.md) | APIs, licencias, juegos similares y datos reales. Su análisis usa el diseño anterior, con nivel 50 |
| [docs/CHULETA.md](docs/CHULETA.md) | Resumen para clase de la web construida |
| [assets/ASSETS.md](assets/ASSETS.md) | Registro de recursos |
| [AGENTS.md](AGENTS.md) | Reglas para agentes de programación |

El README anterior (v1.3) se conserva en el historial: `git show 6ecb531:README.md`.

---

**Aviso legal:** WildPokeBattle es un proyecto de fans, gratuito y sin ánimo de lucro. No está afiliado, patrocinado ni aprobado por Nintendo, Game Freak, Creatures Inc. ni The Pokémon Company. Pokémon y los nombres de sus personajes son marcas de sus respectivos titulares.
