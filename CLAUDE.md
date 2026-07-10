# Finca — Sistema de Gestión Económica

Este proyecto nace de una sesión de planeación con Claude (chat) que cubrió, siguiendo un proceso formal de ingeniería de software, la definición del proyecto y el levantamiento de requisitos. Este archivo es el punto de entrada para continuar el trabajo. No se ha escrito código todavía.

## Léeme primero

Antes de hacer o proponer cualquier cosa, lee completo **`docs/1-definicion-y-requisitos/definicion_proyecto_requisitos.md`**. Contiene el contexto del negocio, el problema, los objetivos, el alcance dentro/fuera, las restricciones y supuestos, los requisitos funcionales y no funcionales priorizados (MoSCoW), las categorías económicas ya cerradas con el cliente, y un registro de decisiones (D1–D5) con su justificación. No lo repitas ni lo resumas en las respuestas — ya está documentado ahí; consúltalo cuando necesites justificar una decisión de diseño.

## Contexto de una línea

App para que el administrador de una finca ganadera lechera (25 cabezas, Colombia) registre ingresos/egresos y consulte su balance financiero desde el celular — hoy lo lleva todo a mano en un cuaderno.

## Estructura del repositorio

Cada carpeta de `docs/` corresponde a un paso del roadmap. Guarda cada entregable en su carpeta correspondiente a medida que se completa — no los dejes sueltos en la raíz de `docs/`:

| Carpeta | Paso | Contenido |
|---|---|---|
| `docs/1-definicion-y-requisitos/` | 1-2 | Definición del proyecto + RF/RNF (ya cerrado) |
| `docs/2-historias-usuario/` | 3 | Historias de usuario + backlog priorizado ⟵ **siguiente** |
| `docs/3-casos-uso-uml/` | 4 | Casos de uso, diagramas de clases/secuencia |
| `docs/4-arquitectura/` (+ `adrs/`) | 5 | Estilo, diagramas C4, stack, decisiones de arquitectura |
| `docs/5-modelo-datos/` | 6 | Modelo de dominio / entidad-relación |
| `docs/6-plan-scrum/` | 7 | Roles, sprints, ceremonias, DoD/DoR |
| `docs/7-cronograma/` | 8 | Sprints e hitos con fechas |
| `docs/8-plan-pruebas/` | 9 | Estrategia de testing |
| `docs/9-documentacion-final/` | 10 | Documento consolidado para la entrega |
| `src/` | — | Código. Estructura interna pendiente de la Arquitectura (Paso 5); ver `src/README.md` |

## Estado del proceso

- [x] Paso 1 — Definición del proyecto (contexto, problema, alcance)
- [x] Paso 2 — Requisitos funcionales y no funcionales (cerrados con el cliente)
- [ ] Paso 3 — Historias de usuario + criterios de aceptación (Gherkin) + backlog priorizado ⟵ **siguiente tarea**
- [ ] Paso 4 — Casos de uso y diagramas UML (clases, secuencia)
- [ ] Paso 5 — Arquitectura de software (estilo, diagramas C4, stack tecnológico, ADRs)
- [ ] Paso 6 — Modelo de datos
- [ ] Paso 7 — Plan Scrum (roles, sprints, ceremonias, Definition of Done/Ready)
- [ ] Paso 8 — Cronograma (sprints + hitos)
- [ ] Paso 9 — Plan de pruebas
- [ ] Paso 10 — Documentación final
- [ ] Implementación

## Restricciones críticas (no negociables)

- **Plazo:** entrega entre el 27 de julio y el 10 de agosto de 2026. Un solo desarrollador.
- **Plataforma:** Android nativo. iOS fuera de alcance. El framework (Kotlin nativo vs. Flutter/React Native) todavía **no** está decidido — es tarea del Paso 5, no lo asumas.
- **Sin entrenamiento de modelos propios de ML.** Si se implementa el reconocimiento de escritura (RF-09/10/11, prioridad Could), es vía API de un modelo multimodal existente, nunca fine-tuning ni entrenamiento propio (decisión D2). Y va al final del backlog, después de que el núcleo financiero (RF-01 a RF-08) esté sólido.
- **Offline-first:** el registro de transacciones debe funcionar sin conexión y sincronizar después.
- Backend/stack de datos: sin definir todavía — decisión del Paso 5.

## Cómo trabajar en este proyecto

- Sigue el roadmap en orden. No saltes pasos ni los fusiones para "ir más rápido" — cada uno es un entregable real del proceso (aplica conceptos de Análisis y Diseño de Software / Fundamentos de Ingeniería de Software), no un trámite.
- Explicaciones, comentarios de diseño y documentación en **español**. Código, nombres de variables/funciones/clases y mensajes de commit en **inglés**.
- Antes de tomar una decisión de arquitectura no trivial (framework móvil, base de datos, proveedor de IA, etc.), preséntala como una pregunta corta con 2-3 opciones y su trade-off — no decidas en silencio. Regístrala luego como ADR en `docs/4-arquitectura/adrs/`.
- Si falta información de negocio que no está en el documento de referencia (un caso borde, una categoría nueva, etc.), pregunta al usuario — no la inventes.
- `.github/workflows/ci.yml`, `scripts/*.sh` y `src/` son placeholders — complétalos cuando el Paso 5 defina el stack, no antes.
- Entorno del desarrollador: Arch Linux.

## Siguiente tarea inmediata

Construir las **historias de usuario** a partir de los RF del documento de referencia, en formato "Como/quiero/para", cada una con sus criterios de aceptación en Gherkin (Dado/Cuando/Entonces), y organizarlas en un backlog priorizado (usando la misma prioridad MoSCoW ya asignada a cada RF). Guardarlas en `docs/2-historias-usuario/`. Al terminar, proponer sprints tentativos solo como referencia — el cronograma formal es el Paso 8.
