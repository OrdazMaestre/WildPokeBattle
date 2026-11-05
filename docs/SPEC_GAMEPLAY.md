# Especificación de juego de WildPokeBattle

Detalle de las reglas de juego. La visión, las decisiones principales y los modos están en el [README](../README.md). La parte técnica está en [ARCHITECTURE.md](ARCHITECTURE.md).

**Nada de este documento está implementado.** El repositorio solo contiene la web estática en HTML y CSS.

## Cómo leer este documento

| Etiqueta | Significado |
|---|---|
| **Confirmada** | Decisión actual del proyecto. Todavía sin construir |
| **Heredada** | Viene del diseño v1.3 y nada la contradice, pero debe revisarse con el nuevo alcance |
| **Propuesta** | Punto de partida para discutir o calibrar |
| **Pendiente** | Sin decidir |

Si hay conflicto, mandan las decisiones del README. El diseño v1.3 completo se puede recuperar con `git show 6ecb531:README.md`.

---

## 1. Cambios frente al diseño v1.3

| Tema | Diseño v1.3 | Decisión actual |
|---|---|---|
| Concepto | Mascota virtual que combate | Simulación de combates autónomos en tiempo real, con preparación de equipos |
| Nivel | Fijo a 50 | **Todos a nivel 100** |
| Pokémon en juego | Máximo 2 activos | Equipos de hasta 6 y formatos salvajes 1 contra 1 y 2 contra 2 |
| Configuración | Generación aleatoria al aparecer, retoques en pausa | La preparación y optimización de equipos es parte central del juego |
| Objetos | No contemplados | Configurables |
| PP | No había: el límite eran los enfriamientos | **PP y cooldown conviven**, y cada movimiento tiene el suyo |
| Precisión | Probabilidad de impacto | Referencia de la **dificultad de ejecutar** el movimiento |
| IA | Personalidad derivada de las estadísticas | Sistema de inteligencia inspirado en parte en Mundo Misterioso |
| Modos | Prado libre | Combates salvajes, entrenadores NPC y multijugador online |
| Web | Wiki como ambientación | Plataforma jugable y wiki real |
| Escritorio | Electron como plataforma objetivo | Evolución futura, con sistema operativo y tecnología sin decidir |
| Sprites | PMD SpriteCollab como propuesta | Prioridad al estilo de Exploradores del Cielo, con inventario de recursos y fuentes alternativas |

---

## 2. Pokémon y equipos

**Confirmada**

- Todos los Pokémon están al nivel 100. La subida de nivel tradicional no es una mecánica central.
- Se configuran estadísticas, naturaleza, movimientos, objetos y otros parámetros de combate.
- Un equipo tiene hasta seis Pokémon.
- Los combates son automáticos: el jugador prepara y optimiza, pero no da órdenes durante la batalla.

**Heredada**

- Estadísticas, tipos, habilidades y movimientos influyen en el comportamiento, no solo se muestran.
- Un Pokémon debilitado vuelve a su Poké Ball.
- Dos Pokémon chocan entre sí con un empuje suave para no solaparse.

**Pendiente**

- Cuántos Pokémon de cada equipo están activos a la vez y cómo entra el siguiente.
- Qué objetos existen y cómo funcionan en tiempo real.
- Qué parámetros son editables además de los citados.

### 2.1. Generación aleatoria de sets (heredada)

Los combates salvajes admiten Pokémon aleatorios. **Falta decidir** si su configuración se genera con este procedimiento de v1.3:

| Dato | Cómo se genera |
|---|---|
| Estadísticas | Stats base de la especie (Showdown) + IV aleatorios 0–31 + EV 0 + naturaleza aleatoria |
| Movimientos | Ver el procedimiento de abajo |
| Habilidad | Una al azar entre las del rol elegido o, si no hay rol, las de la especie |

1. Si la especie tiene set de Random Battle, se sortea **un rol** y se usa solo su `movepool`. Mezclar roles rompe la coherencia del set.
2. Se retiran los movimientos marcados como `excluded` (ver [4.5](#45-de-turnos-a-tiempo-real)).
3. Se eligen hasta 4 sin repetir. Muchos `movepool` tienen 4 o menos, así que el azar real suele estar en el rol.
4. Si faltan movimientos, se completa desde el conjunto aprendible con movimientos no excluidos.
5. Debe quedar al menos un movimiento de daño. Si no, se cambia uno por un ataque aprendible.

Las especies sin set (Bulbasaur, Eevee o Magikarp) saltan al paso 4 con un mínimo de 2 ataques de daño, uno con STAB si existe.

Todo sale de un RNG con semilla. La generación, el combate y la IA usan flujos separados, para que regenerar o editar un Pokémon no altere los demás resultados.

---

## 3. Inteligencia artificial

**Confirmada.** La inteligencia influye en cómo un Pokémon ejecuta sus movimientos y toma decisiones:

- Elegir el momento adecuado para atacar.
- Gestionar los movimientos que requieren preparación.
- Adaptarse a la situación del combate.
- Aprovechar oportunidades y posicionamiento.
- Tomar decisiones estratégicas.

La inspiración es parcial: el sistema de inteligencia de Pokémon Mundo Misterioso. Los legendarios tienen inteligencia máxima y ventajas o comportamientos personalizados.

**Pendiente:** la escala de inteligencia. No hay ninguna escala numérica definida.

**Heredada**

- La IA evalúa posiciones, distancia y altura, PS, movimientos disponibles, cooldowns, tipos y efectividades, obstáculos y opciones de esquiva.
- Cada Pokémon decide si avanza, retrocede, se detiene, salta, cambia de plataforma, ataca, esquiva o explora.
- El movimiento es libre: no está obligado a perseguir al rival, y ninguna distancia global cancela el combate.
- La personalidad de combate se deriva de los datos (alta Velocidad, hostigador; alta Defensa, aguanta y contraataca).
- La IA no conoce el futuro ni tiene puntería perfecta.

**Propuesta:** modelar la decisión como una puntuación por distancia, como Monster Rancher, antes de probar algo más complejo ([informe, sección 4](INFORME_INVESTIGACION.md#4-juegos-y-proyectos-similares)).

---

## 4. Movimientos

### 4.1. Precisión

**Confirmada.** La precisión oficial no se traduce en una probabilidad tradicional de acertar. Sirve como referencia para definir la dificultad de ejecutar correctamente el movimiento.

Dos movimientos con la misma precisión no tienen por qué funcionar igual. Rayo Solar tiene una precisión oficial del 100 %, pero requiere preparación y exposición antes de ejecutarse.

**Pendiente:** cómo se convierte la precisión en dificultad.

### 4.2. Ciclo de uso

**Confirmada.** Estos conceptos son distintos y no se mezclan:

| Concepto | Qué es |
|---|---|
| **PP** | Número de usos disponibles |
| **Cooldown** | Tiempo de espera antes de poder repetir el movimiento |
| **Preparación o carga** | Fase previa al ataque, en la que el Pokémon queda expuesto |
| **Ejecución** | El ataque en sí |
| **Recuperación** | Fase posterior, antes de poder actuar con normalidad |

Cada movimiento tiene sus propios PP y su propio cooldown.

**Propuesta:** como punto de partida, `cooldown = base_por_potencia × factor_por_velocidad`, ajustando después por movimiento. `factor_por_velocidad` es una idea propia del proyecto: las fuentes no aclaran si Legends: Z-A hace algo parecido.

**Propuesta:** un ataque básico sin cooldown, para evitar huecos muertos, y enseñar la preparación con una pose o un aviso ([informe, sección 4.1](INFORME_INVESTIGACION.md#41-combate-en-tiempo-real)).

### 4.3. Forma de un movimiento (propuesta)

Ningún juego define la forma física de un movimiento, así que se deriva de los datos de Showdown. Las reglas se aplican en orden y gana la primera que coincide:

| # | Condición en Showdown | Forma |
|---|---|---|
| 1 | Está en `js/data/move-overrides.json` | La indicada ahí |
| 2 | `target` `self`, `allies`, `allyTeam`, `adjacentAllyOrSelf` o `adjacentAlly` | `self` |
| 3 | `target` `all`, `allySide` o `foeSide` | `global` |
| 4 | `target` `allAdjacent` o `allAdjacentFoes` | `area` |
| 5 | flag `contact` | `melee` |
| 6 | flag `bullet` | `projectile` |
| 7 | flag `pulse` | `beam` |
| 8 | flag `sound` | `area` |
| 9 | Categoría Estado con un objetivo | `projectile` |
| 10 | Físico → `melee`; Especial → `projectile` | fallback |

La categoría Estado no significa «sobre sí mismo»: Tóxico u Onda Trueno son Estado y van contra el rival. Ningún flag identifica Hiperrayo, Rayo Hielo, Rayo Solar o Rayo como `beam`, y por eso hace falta la tabla de excepciones. El reparto real está en el [informe](INFORME_INVESTIGACION.md#5-resultados-con-datos-reales).

| Forma | Regla |
|---|---|
| `melee` | Requiere proximidad |
| `projectile` | Trayectoria y alcance |
| `beam` | Gran distancia, línea recta |
| `area` | Afecta una región |
| `global` | Afecta a todo el escenario o a un bando (clima, trampas, campos) |
| `self` | Efecto sobre el propio Pokémon; no necesita alcance ni puntería |

La prioridad reduce o aumenta el tiempo de ejecución. Los efectos secundarios (estados, cambios de stats) se reutilizan.

### 4.4. Daño (propuesta)

Daño y física son sistemas distintos pero conectados: un ataque puede producir retroceso, desplazamiento, caída, interrupción, explosión o efectos sobre el escenario.

```text
daño = ((2·Nivel/5 + 2) · Potencia · Ataque/Defensa) / 50 + 2
       × STAB (1,5) × efectividad de tipo × aleatorio(0,85–1) × battleScale
```

- Los ataques físicos usan Ataque y Defensa; los especiales, Ataque Esp. y Defensa Esp.
- `battleScale` es un multiplicador global de ritmo.
- **Las cifras del informe se calcularon a nivel 50.** Hay que repetir el cálculo a nivel 100 antes de fijar `battleScale`.

### 4.5. De turnos a tiempo real

**Propuesta.** Los datos de Showdown describen un juego por turnos. Cada movimiento lleva un nivel de soporte: `supported`, `adapted` o `excluded`.

| Grupo | Total | En sets Gen 9 | Tratamiento inicial |
|---|---:|---:|---|
| Daño directo con potencia fija | La mayoría | La mayoría | `supported` |
| Daño variable o fijo (`basePower` 0) | 41 | 11 | `excluded` hasta tener su función en `move-overrides.json` |
| Dos turnos (flag `charge`) | 17 | 2 | `adapted`: preparación visible. Vuelo y Excavar esperan a la Fase 7 |
| Recarga (flag `recharge`) | 10 | 1 | `adapted`: inmóvil durante un tiempo tras el ataque |
| Multigolpe | 31 | 11 | `adapted`: ráfaga de impactos |
| KO directo | 4 | 0 | `excluded` |
| Autodestrucción | 7 | 2 | `excluded` hasta la Fase 7 |
| Prioridad distinta de 0 | 55 | 25 | `adapted`: reduce o aumenta el tiempo de ejecución |
| Dependen de turnos o del cambio de Pokémon (Protección, Sustituto, Ida y Vuelta, Mofa…) | Revisión manual | 18 | `excluded`, salvo los que tengan regla propia en `move-overrides.json` |

- Todo lo que los juegos miden en turnos (estados, clima, daño residual) se convierte con una constante única, `TURN_MS`. Una quemadura hace 1/16 de los PS cada `TURN_MS`.
- En v1.3 no existía el cambio de Pokémon, y los movimientos que cambian al usuario solo hacían su daño. Hay que revisarlo cuando se decida cómo funcionan los equipos.

### 4.6. Capacidades físicas (heredada)

Son reglas de movimiento o de escenario que algunos ataques activan. **No son las Habilidades de Pokémon** (Espesura, Intimidación). Pueden alterar temporalmente las reglas, pero la gravedad sigue siendo la norma.

| Capacidad | Efecto | Ejemplo |
|---|---|---|
| Vuelo | Sustentación y desplazamiento aéreo | Vuelo |
| Salto potenciado | Modifica el impulso vertical | Bote |
| Excavación | Interacción con ciertos terrenos | Excavar |
| Surf | Desplazamiento sobre superficies acuáticas | Surf |
| Teletransporte | Cambio instantáneo de posición | Teletransporte |
| Sacudida | Afecta a superficies y entidades | Terremoto |
| Conducción | Los ataques eléctricos se propagan por superficies | Rayo |

---

## 5. Movimiento físico de los Pokémon

**Confirmada:** escenarios bidimensionales de perspectiva lateral, con gravedad, saltos, plataformas, colisiones y elementos interactivos.

**Heredada**

- Plano 2D: X horizontal, Y vertical (hacia abajo positivo, como el DOM).
- Estado mínimo de cada Pokémon: posición, velocidad, aceleración, masa, fricción, fuerza de salto, velocidad máxima, `onGround`, contactos y modo de movimiento (`ground` o `fly`). Los valores varían por especie.
- El vuelo es un modo de movimiento, no «apagar la física»: reduce la gravedad y permite controlar la aceleración vertical.
- Colisiones: Pokémon contra suelo, plataformas, paredes, obstáculos y otro Pokémon; ataques contra Pokémon y superficies.
- La caja física es independiente del sprite: la animación cambia de pose, y el cuerpo representa el espacio real.

Las reglas de estabilidad física están en [ARCHITECTURE.md](ARCHITECTURE.md#4-física-propuesta).

---

## 6. Final del combate (heredada)

**Requisito:** todo combate termina, sin cancelarlo por distancia. Sin estas reglas podría no acabar nunca: Normal contra Fantasma, dos Pokémon solo con movimientos de Estado, o un `melee` que no alcanza la plataforma del rival.

| Situación | Regla |
|---|---|
| Un Pokémon lleva `STRUGGLE_MS` sin ningún movimiento que pueda hacer daño | Usa un ataque de último recurso (inspirado en Forcejeo): `melee`, sin tipo y con daño propio por retroceso |
| El combate supera `SUDDEN_DEATH_MS` | Muerte súbita: ambos pierden un porcentaje creciente de PS máximos |
| Ambos caen en el mismo tick | Doble KO: empate |

**Pendiente:** adaptar estas reglas a equipos completos y al agotamiento de PP. Las revanchas tras un empate online están en el [README](../README.md#4-modos-de-juego).

---

## 7. Prado, pausa y editor (heredada)

Es el flujo de v1.3. La interfaz ya tiene su maqueta, pero **falta decidir cómo encaja con la preparación de equipos y con el modo online**, donde una pausa individual no tendría sentido.

### 7.1. Flujo del prado

1. El usuario entra en el prado y pulsa **«Seleccionar Pokémon»**.
2. Elige un Pokémon en el selector.
3. El Pokémon sale de una Poké Ball, cae por gravedad, aterriza y empieza a moverse.
4. Con un segundo Pokémon empieza el combate.

### 7.2. Pausa por teclado

| Acción | Tecla |
|---|---|
| Pausar (abre el editor) o reanudar | `Alt + P` |
| Cerrar el editor sin reanudar | `Esc` |

- Sin botón de pausa en pantalla. Hay un indicador «Pausa» y una línea con el atajo en el selector y en la ayuda.
- Con el editor cerrado, el indicador muestra el enlace «Abrir editor», que no pausa ni reanuda.
- La pausa congela física, IA, cooldowns y animaciones. El render sigue mostrando el último estado.
- Se detecta con `event.code === "KeyP"`, `event.altKey` y `!event.ctrlKey`, con `preventDefault()`. En Windows, AltGr se notifica como Ctrl + Alt: sin esa exclusión, AltGr + P también pausaría.
- Funciona con el foco en un campo del editor, porque lleva modificador.
- La combinación vive en un único sitio de configuración (`js/ui/shortcuts.js`).

### 7.3. Editor

Panel de la capa web, solo en escritorio porque se abre con teclado. En móvil se mantiene el **modo espectador**: se puede elegir Pokémon y mirar.

- **Estadísticas:** IV, EV y naturaleza como en Showdown, con recálculo inmediato. El campo de nivel de la maqueta sobra, porque el nivel es siempre 100.
- **Movimientos:** cambiar cualquiera de los 4 por otro aprendible. Los `excluded` aparecen desactivados.
- **Habilidad:** elegir entre las de la especie.
- **Regenerar:** volver a sortear todo o un bloque, con semilla nueva o la actual.
- **Retirar:** devuelve el Pokémon a su Poké Ball sin combatir.
- **Importar y exportar** en el formato de texto de Showdown. Un set con movimientos `excluded` se rechaza con un aviso.

Reglas:

- Se edita solo en pausa, y los cambios se aplican todos a la vez al reanudar.
- Si cambian los PS máximos, los actuales se ajustan en proporción.
- Cambiar movimientos reinicia sus cooldowns.
- El editor modifica el estado en el núcleo; nunca escribe directamente en física ni en render.
- La exportación solo garantiza compatibilidad con Showdown en modo legal. Un valor final editado a mano no cabe en ese formato, y el editor avisa de lo que se pierde.

---

## 8. Valores y preguntas abiertas

| Pregunta | Propuesta heredada |
|---|---|
| ¿Cuánto debe durar un combate? | 30–90 s de media entre dos Pokémon. Hay que revisarlo para equipos de hasta seis |
| ¿Cuánto valen `TURN_MS`, `STRUGGLE_MS` y `SUDDEN_DEATH_MS`? | Partir de 2 s, 15 s y 3 min, y ajustar con el spike de ritmo |
| ¿El ganador se cura? | En v1.3, sí, para que el siguiente combate sea justo. Depende del formato |
| ¿Configuración libre o dentro de los límites legales? | Por defecto, legal según Showdown (IV 0–31, EV ≤ 252 por stat y 510 en total, movimientos aprendibles), con un «modo libre» |
| ¿`Alt + P` choca con algún atajo? | No figura en la lista oficial de Chrome para Windows. Falta probar Firefox, Edge y macOS (`Option + P` escribe «π») |
| ¿Qué especies forman la primera versión? | Venusaur, Charizard, Blastoise, Pikachu, Gengar, Snorlax, Dragonite, Scizor, Tyranitar, Gardevoir, Garchomp y Lucario. No incluye legendarios |
| ¿Siguen las evoluciones y la colección? | Estaban en la antigua Fase 9. Hay que revisarlas, porque no hay subida de nivel como mecánica central |

---

## 9. Referencias e inspiración

Son inspiración: no se copian recursos, código ni elementos protegidos. El análisis está en el [informe](INFORME_INVESTIGACION.md#4-juegos-y-proyectos-similares).

| Referencia | Qué se toma |
|---|---|
| Pokémon Mundo Misterioso | Sistema de inteligencia (en parte) y estilo de sprites y animaciones |
| Pokémon Legends: Z-A, Pokémon UNITE | Movimientos adaptados a tiempo real con cooldowns |
| Monster Rancher | Combate autónomo con técnicas condicionadas por la distancia |
| Plataformas 2D | Gravedad, salto, colisiones, desplazamiento lateral |
| WikiDex | Estructura de la web: cabecera, navegación, artículos, listas |
| eSheep, Shimeji | Criaturas que caminan sobre las ventanas (escritorio futuro) |
| VS Code Pets, VS Code Pokémon | Mascotas dentro de una interfaz de trabajo |
| DesktopInvasion *(sin verificar)* | No se ha localizado ningún título con este nombre |
| Pokémon Apex Arena *(sin verificar)* | No se ha localizado ningún título con este nombre |

---

## 10. Glosario

| Término | Significado |
|---|---|
| Arena | Escenario físico donde se mueven los Pokémon |
| Capacidad física | Regla de movimiento o escenario que activa un ataque (vuelo, excavación…) |
| Conjunto aprendible | Movimientos que una especie puede aprender según los learnsets de Showdown |
| Cooldown o enfriamiento | Tiempo antes de reutilizar un movimiento. No sustituye a los PP |
| Forma | Cómo se comporta un movimiento en el espacio: `melee`, `projectile`, `beam`, `area`, `global` o `self` |
| Habilidad | Rasgo de la especie en los juegos Pokémon (Espesura, Intimidación…) |
| Muerte súbita | Pérdida progresiva de PS que garantiza el final de un combate largo |
| Nivel de soporte | Si un movimiento funciona tal cual (`supported`), con reglas propias (`adapted`) o no está disponible (`excluded`) |
| Rol | Variante de set de Random Battle con sus propios movimientos |
| Semilla | Número que fija el resultado del generador aleatorio |
| Set | Configuración completa de un Pokémon, en el formato de texto de Showdown |
