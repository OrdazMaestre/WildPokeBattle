# Reglas para agentes de programación

**Antes de tocar nada**

1. Leer el [README](README.md), [SPEC_GAMEPLAY](docs/SPEC_GAMEPLAY.md), [ARCHITECTURE](docs/ARCHITECTURE.md), el [informe](docs/INFORME_INVESTIGACION.md) y la documentación de la tarea.
2. Comprobar en qué fase está el proyecto. No implementar funciones de fases posteriores.
3. Comprobar el código antes de afirmar que algo está implementado. Que esté documentado no significa que exista.

**Decisiones de diseño (no se cambian sin ADR)**

4. Sin combate por turnos ni distancia global que lo cancele.
5. Combates automáticos: el jugador prepara equipos, pero no controla a los Pokémon durante la batalla.
6. Movimiento autónomo, perspectiva lateral 2D, gravedad como regla general.
7. El escenario es interactivo, nunca una imagen plana.
8. Todos los Pokémon están al nivel 100.
9. PP y cooldown son conceptos distintos, y cada movimiento tiene los suyos.
10. La precisión no es una probabilidad de acierto: es una referencia de la dificultad de ejecución.
11. Los filtros de tiers pertenecen solo a los combates salvajes.
12. No inventar Pokémon para completar equipos de entrenadores NPC.
13. La aleatoriedad pasa siempre por el RNG con semilla del núcleo (nunca `Math.random()` directo).
14. Pausa solo por teclado, sin botón en pantalla, mientras no se revise esa decisión heredada.

**Código**

15. Priorizar la versión web. No empezar el escritorio.
16. No añadir dependencias, frameworks ni pasos de build sin explicar qué necesidad resuelven. La tecnología no confirmada se trata como propuesta.
17. Los datos de mecánicas salen de Showdown. No copiar su simulador por turnos ni mezclar su lógica con el motor en tiempo real.
18. Battle Palace Core no toca el DOM ni APIs del navegador.
19. No confundir caja visual con cuerpo físico. Cero lógica física en CSS.
20. No duplicar lógica entre web y escritorio. Rutas relativas.
21. No asumir que un recurso gráfico es libre, ni que todo un repositorio comunitario tiene la misma licencia. Registrar autor, procedencia y licencia en `assets/ASSETS.md`.
22. **No subir sprites al repositorio** ni enlazar en caliente sprites de terceros.
23. Toda excepción de movimientos va en `js/data/move-overrides.json`, no repartida por el código.

**Ante dudas**

24. Si una decisión no está definida, documentar la duda y proponer opciones. No inventarla como aprobada.
25. No sustituir una decisión aprobada por algo más sencillo sin justificarlo.

## HTML y CSS

- Las páginas no llevan JS ni física hasta la fase que los introduce.
- No simular el combate con animaciones.
- Priorizar semántica, navegación y organización.
- Preparar el marcado (`data-wpb-body`, `id`, clases) para las capas futuras sin implementarlas.
- No dar por terminada una funcionalidad que solo tenga estructura HTML.
- Las superficies físicas no usan `transform`, `translate`, `scale`, `rotate`, `fixed` ni `sticky`, tampoco en `:hover`. Solo se animan las capas decorativas sin `data-wpb-body`.

## Informe al terminar cada tarea

```markdown
**Archivos:** creados / modificados
**Hecho:** funcionalidad implementada
**Decisiones:** técnicas tomadas (¿requiere ADR?)
**Dependencias:** añadidas / retiradas
**Pruebas:** qué se probó y resultado
**Limitaciones y pendientes:** bloqueos, documentación a actualizar
```

Un ADR es un registro breve de una decisión de arquitectura y su porqué. Irá en `docs/decisions/`, que todavía no existe. Los ADR recomendados están en el [informe](docs/INFORME_INVESTIGACION.md#8-adr-y-cuestiones-abiertas).
