+++
date = '2025-11-16T13:00:00-04:00'
draft = true
title = 'Paradigmas de Desarrollo de Software Explicados (Sin la Jerga de Consultoría)'
tags = ['agile', 'scrum', 'kanban', 'devops', 'tdd', 'software-engineering']
categories = ['tech']
+++

Alguien me pidió que explicara la diferencia entre TDD, BDD y CDD, y luego entre Agile, Scrum y Kanban, y luego cómo CI/CD encaja en todo esto.

Resulta que estas son en realidad dos categorías diferentes de conceptos que la gente mezcla constantemente. Aquí está lo que realmente significan, sin la tontería de consultoría empresarial.

## Dos Conjuntos Diferentes

**Conjunto 1: Prácticas de Pruebas/Desarrollo**

- TDD (Desarrollo Guiado por Pruebas)
- BDD (Desarrollo Guiado por Comportamiento)
- CDD (Desarrollo Guiado por Contratos)

**Conjunto 2: Marcos de Gestión de Proyectos**

- Agile (la filosofía)
- Scrum (el marco)
- Kanban (el sistema)
- Lean Manufacturing (el origen)
- Just-in-Time Manufacturing (la estrategia)

Estos están relacionados pero son distintos. El primer conjunto es sobre *cómo escribes código*. El segundo conjunto es sobre *cómo organizas el trabajo*.

Desglosémoslos.

## Conjunto 1: Prácticas de Pruebas y Desarrollo

### Desarrollo Guiado por Pruebas (TDD)

Escribe pruebas antes de escribir código.

El ciclo:

1. Escribe una prueba que falle
2. Escribe justo el código suficiente para que pase
3. Refactoriza mientras mantienes las pruebas en verde
4. Repite

El punto realmente no es sobre las pruebas—es sobre forzarte a pensar qué debería hacer el código antes de escribirlo. Las pruebas son solo el artefacto de ese pensamiento.

¿Funciona? A veces. Cuando sabes exactamente qué estás construyendo, TDD es genial.
Honestamente nunca lo he hecho, cuando estás explorando y no sabes qué debería hacer el código todavía, TDD puede sentirse como escribir con esposas.

### Desarrollo Guiado por Comportamiento (BDD)

El primo más comunicativo de TDD. Misma idea básica (escribir pruebas primero), pero las pruebas están escritas en un formato que personas no técnicas pueden leer.

En lugar de `assert(user.isValid())`, escribes:

```gherkin
Given a user with a valid email
When they attempt to log in
Then they should be authenticated
```

El beneficio es que gerentes de producto, QA y desarrolladores pueden todos estar de acuerdo en qué significa "comportamiento correcto" antes de que alguien escriba código. La desventaja es mantener otra capa de abstracción más.

BDD es más útil cuando la falta de comunicación es tu mayor problema. Si todos ya están de acuerdo en los requisitos, es solo sobrecarga extra.

### Desarrollo Guiado por Contratos (CDD)

Define el contrato de la API primero, luego impleméntalo.

Si estás construyendo un microservicio, escribes la especificación OpenAPI (o equivalente) antes de escribir cualquier código. El contrato especifica: "Este endpoint toma X, retorna Y, falla con Z."

Luego implementas el servicio para que coincida con el contrato, y pruebas contra ese contrato.

Esto es útil cuando múltiples equipos necesitan integrarse antes de que alguien haya terminado de construir. El equipo de frontend puede desarrollar contra el contrato mientras el equipo de backend lo implementa.

La desventaja es que los contratos son difíciles de cambiar una vez que la gente depende de ellos, así que más vale que lo hagas bien la primera vez.

### Qué Tienen en Común

Los tres son sobre *definir expectativas antes de la implementación*. Difieren en cómo se ven esas expectativas:

- TDD: Pruebas a nivel de código
- BDD: Especificaciones de comportamiento
- CDD: Contratos de API

## Conjunto 2: Gestión de Proyectos y Procesos

### Agile (La Filosofía)

No es un proceso específico—es una filosofía sobre cómo debería funcionar el desarrollo de software.

La idea original (del [Manifiesto Ágil](https://agilemanifesto.org/)):

- Individuos e interacciones sobre procesos y herramientas
- Software funcionando sobre documentación exhaustiva
- Colaboración con el cliente sobre negociación de contratos
- Responder al cambio sobre seguir un plan

Eso es todo. Todo lo demás es interpretación.

Agile dice: Construye en incrementos pequeños, obtén retroalimentación, adáptate. No pases seis meses construyendo la cosa equivocada.

No te dice *cómo* hacer eso. Ahí es donde entran marcos como Scrum y Kanban.

### Scrum (El Marco)

Scrum es una implementación específica de principios Agile.

**Componentes principales:**

- **Sprints**: Cajas de tiempo fijas (usualmente 2 semanas) donde construyes un conjunto de características
- **Roles**: Product Owner (decide qué construir), Scrum Master (remueve obstáculos), Equipo de Desarrollo (lo construye)
- **Ceremonias**:
  - Sprint Planning (decidir qué construir este sprint)
  - Daily Standup (sincronización rápida sobre el progreso)
  - Sprint Review (demo de lo que construiste)
  - Sprint Retrospective (descubrir cómo mejorar)
- **Artefactos**: Product Backlog (todas las cosas), Sprint Backlog (cosas para este sprint), Incremento (software funcionando)

Scrum es prescriptivo. Dice "haz estas cosas, de esta manera, en estos intervalos."

**Cuándo funciona:** Equipos que necesitan estructura, ciclos de entrega predecibles, e involucramiento de stakeholders.

**Cuándo no funciona:** Equipos que odian las reuniones, tienen prioridades constantemente cambiantes, o necesitan despliegue continuo.

### Kanban (El Sistema)

Kanban es sobre visualizar el trabajo y limitar el trabajo en progreso.

**Principios principales:**

- Visualizar el flujo de trabajo (usualmente como un tablero con columnas: Por Hacer, En Progreso, Hecho)
- Limitar TEP (Trabajo en Progreso)—solo X ítems pueden estar "En Progreso" a la vez
- Gestionar el flujo—optimizar para entrega continua y suave
- Hacer las políticas de proceso explícitas
- Mejorar colaborativamente

Kanban es menos prescriptivo que Scrum. Sin sprints, sin reuniones requeridas, sin roles fijos. Solo: "Aquí está el trabajo, aquí está donde está, no comiences cosas nuevas hasta que termines las actuales."

**Cuándo funciona:** Equipos haciendo despliegue continuo, trabajo de soporte/mantenimiento, o cualquier cosa con llegada impredecible de tareas.

**Cuándo no funciona:** Cuando los stakeholders quieren fechas de entrega predecibles, o cuando el equipo necesita más estructura.

### Manufactura Esbelta y Justo a Tiempo

Estos no son conceptos de software originalmente—son de manufactura (Toyota, específicamente).

**Lean**: Elimina el desperdicio. Cualquier cosa que no cree directamente valor para el cliente es desperdicio.

**Justo a Tiempo (JIT)**: Produce exactamente lo que se necesita, exactamente cuando se necesita. Sin inventario quedándose ahí.

El software tomó prestadas estas ideas:

- No construyas características que nadie usará (Lean)
- No despliegues antes de cuando los usuarios lo necesitan (JIT)
- Minimiza el trabajo en progreso (Lean)
- Entrega continuamente en lugar de en lotes (JIT)

Kanban es básicamente JIT aplicado al software.

## Dónde Encaja CI/CD

**Integración Continua (CI)**: Los desarrolladores integran código en un repositorio compartido frecuentemente (múltiples veces al día). Cada integración dispara builds y pruebas automatizadas.

**Despliegue Continuo (CD)**: El código que pasa las pruebas se despliega automáticamente a producción.

CI/CD es una *práctica técnica* que habilita metodologías Agile.

No puedes hacer Scrum bien sin CI—necesitas software funcionando al final de cada sprint, lo cual significa que la integración no puede ser una pesadilla.

Definitivamente no puedes hacer Kanban bien sin CD—el flujo continuo requiere despliegue continuo.

CI/CD automatiza el ciclo de retroalimentación del que Agile depende. No es una metodología en sí; es la infraestructura que hace posibles las metodologías modernas.

## DevOps: El Envoltorio Cultural

DevOps no es un rol o una herramienta—es un cambio cultural.

Modelo tradicional:

- Los desarrolladores escriben código
- El equipo de operaciones lo despliega y mantiene
- Estos equipos tienen incentivos conflictivos (Devs quieren enviar rápido, Ops quiere estabilidad)

Modelo DevOps:

- El mismo equipo maneja desarrollo y operaciones
- Incentivos compartidos (enviar rápido *y* mantenerlo estable)
- Automatización pesada (porque no puedes hacer ambos sin ella)

DevOps se apoya en pipelines de CI/CD, monitoreo, infraestructura como código, y cultura colaborativa.

Se sienta por encima de todos los otros marcos. Puedes hacer DevOps con Scrum, o con Kanban, o con cualquier proceso que quieras. DevOps es sobre romper el muro entre desarrollo y operaciones.

## Cómo Se Relacionan

```
Agile (Filosofía)
├── Scrum (Marco para entrega iterativa)
│   └── Requiere CI para software funcionando cada sprint
├── Kanban (Marco para entrega continua)
│   └── Requiere CD para flujo continuo
└── Lean (Minimizar desperdicio, maximizar valor)
    └── Inspira prácticas de Kanban y DevOps

DevOps (Cultura)
└── Habilitado por CI/CD (Prácticas técnicas)
    ├── Integración Continua
    └── Despliegue Continuo

Prácticas de Pruebas (Nivel de código)
├── TDD (Pruebas primero)
├── BDD (Especificaciones de comportamiento primero)
└── CDD (Contratos primero)
```

Son complementarios. Podrías usar Scrum para gestión de proyectos, TDD para desarrollo, y CI/CD para automatizar pruebas y despliegue, todo bajo una cultura DevOps.

O podrías usar Kanban con BDD y despliegue continuo.

O cualquier otra combinación que encaje con tu contexto.

## Qué Realmente Importa

Aquí está la cosa: ninguno de estos son balas mágicas.

Scrum no salvará a un equipo disfuncional. Kanban no funcionará si tienes 50 ítems en progreso. TDD no ayudará si estás probando las cosas equivocadas. DevOps no arreglará código malo.

Todos son herramientas. Usa las que resuelvan tus problemas reales:

- **¿Demasiado trabajo en progreso?** Prueba Kanban con límites de TEP.
- **¿Los stakeholders quieren entrega predecible?** Prueba Scrum con sprints.
- **¿La integración es dolorosa?** Implementa CI.
- **¿El despliegue da miedo?** Agrega pruebas automatizadas y CD.
- **¿Dev y Ops se odian mutuamente?** Prueba cultura DevOps (y terapia).
- **¿Inseguro sobre qué construir?** Usa BDD para alinearte en requisitos.

No adoptes un marco porque está de moda. Adóptalo porque resuelve un problema que realmente tienes.

## Lectura Adicional

Si quieres el material original en lugar de mis opiniones:

- [Manifiesto Ágil](https://agilemanifesto.org/) (corto, vale la pena leerlo)
- [Guía de Scrum](https://scrumguides.org/) (definición oficial de Scrum)
- [Guía de Kanban](https://kanbanguides.org/) (principios oficiales de Kanban)
- *The Phoenix Project* de Gene Kim (DevOps como una novela—sorprendentemente bueno)
- *Continuous Delivery* de Jez Humble (la biblia de CI/CD)

---

Ahora puedes explicar la diferencia en tu próxima reunión de standup. De nada.
