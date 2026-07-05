# Guía de gestión de proyectos y productos digitales

## Objetivo

Esta guía resume buenas prácticas para gestionar proyectos y productos, especialmente en contextos de tecnología, software, datos, plataformas digitales y servicios web. Cubre desde la creación de productos y el contacto con clientes hasta presentaciones ejecutivas, seguimiento, gestión de derivas, marketing, Scrum y metodologías ágiles.

Está organizada en tres niveles:

```text
1. Básico: fundamentos, roles, objetivos, clientes, alcance, planificación, comunicación y seguimiento.
2. Intermedio: producto, discovery, roadmaps, métricas, presentaciones, marketing, Scrum, Kanban y gestión de riesgos.
3. Avanzado: portafolio, gobernanza, escalamiento, gestión de stakeholders, derivas complejas, métricas ejecutivas, operación, cambio organizacional y mejora continua.
```

La guía distingue dos conceptos que suelen mezclarse:

```text
Proyecto:
Esfuerzo temporal para entregar un resultado específico.

Producto:
Activo vivo que evoluciona para resolver problemas de usuarios y generar valor de negocio.
```

Un proyecto puede crear o mejorar un producto, pero no todo trabajo de producto debería gestionarse como proyecto cerrado.

---

# Parte I: fundamentos básicos

## 1. Qué es gestión de proyectos

La gestión de proyectos coordina personas, tiempo, presupuesto, alcance, riesgos y comunicación para entregar un resultado.

Un proyecto tiene normalmente:

```text
Objetivo.
Inicio y fin.
Stakeholders.
Restricciones.
Entregables.
Riesgos.
Plan de trabajo.
Seguimiento.
Cierre.
```

Ejemplo:

```text
Proyecto:
Implementar un sistema de autenticación para una plataforma web antes del 30 de septiembre.

Entregables:
- Login.
- Registro.
- Recuperación de contraseña.
- MFA.
- Documentación.
- Pruebas.
- Despliegue.
```

---

## 2. Qué es gestión de producto

La gestión de producto se enfoca en descubrir, construir, medir y evolucionar una solución que crea valor para usuarios y negocio.

Un producto tiene:

```text
Usuarios.
Problemas.
Propuesta de valor.
Roadmap.
Métricas.
Feedback.
Ciclo de vida.
Estrategia.
```

Ejemplo:

```text
Producto:
Plataforma de gestión de tareas para equipos técnicos.

Objetivo de producto:
Reducir el tiempo de coordinación y aumentar visibilidad del trabajo.

Métricas:
- Tareas completadas por semana.
- Tiempo promedio de ciclo.
- Usuarios activos.
- Retención.
- NPS.
```

---

## 3. Proyecto vs producto

| Aspecto | Proyecto | Producto |
|---|---|---|
| Horizonte | Temporal | Continuo |
| Éxito | Entregar alcance acordado | Crear valor sostenido |
| Gestión | Plan, hitos, riesgos | Discovery, roadmap, métricas |
| Cambio | Controlado | Esperado |
| Cliente | Stakeholder del proyecto | Usuario/mercado permanente |
| Resultado | Entregable | Evolución de solución |

Regla práctica:

```text
Si el trabajo termina al entregar, piensa como proyecto.
Si la solución seguirá evolucionando con usuarios, piensa como producto.
```

---

## 4. Principios de buena gestión

Principios básicos:

```text
Claridad de objetivos.
Comunicación frecuente.
Responsabilidades explícitas.
Priorización.
Transparencia.
Gestión de riesgos.
Validación temprana.
Seguimiento basado en evidencia.
Aprendizaje continuo.
Cierre formal.
```

Antipatrones:

```text
Empezar sin objetivo claro.
Prometer fechas sin entender alcance.
No hablar con usuarios.
No registrar decisiones.
No gestionar riesgos.
Confundir actividad con avance.
Usar reuniones como reemplazo de decisiones.
```

---

## 5. Las tres restricciones clásicas

Todo proyecto balancea:

```text
Alcance.
Tiempo.
Costo.
```

También debes considerar:

```text
Calidad.
Riesgo.
Satisfacción del cliente.
Capacidad del equipo.
Mantenibilidad.
Seguridad.
Operación.
```

Ejemplo:

```text
Si el cliente exige adelantar la fecha,
normalmente debes reducir alcance, aumentar capacidad o aceptar más riesgo.
```

No prometas cambiar una restricción sin negociar las demás.

---

## 6. Definir éxito

Antes de comenzar, define qué significa éxito.

Ejemplo deficiente:

```text
Hacer una app moderna.
```

Mejor:

```text
Lanzar una aplicación web que permita a 500 usuarios internos crear, buscar y cerrar solicitudes, reduciendo en 40% el tiempo promedio de gestión durante los primeros tres meses.
```

Componentes de un buen objetivo:

```text
Resultado esperado.
Usuario o cliente.
Métrica.
Horizonte temporal.
Restricción relevante.
```

---

## 7. Roles básicos

Roles comunes:

```text
Sponsor:
Persona que financia o habilita el proyecto.

Cliente:
Persona u organización que recibe valor.

Usuario:
Persona que usa la solución.

Project Manager:
Coordina plan, riesgos, comunicación, dependencias y seguimiento.

Product Manager / Product Owner:
Prioriza valor, problema, roadmap y backlog.

Tech Lead:
Define dirección técnica.

Equipo de desarrollo:
Construye, prueba y entrega.

QA:
Valida calidad.

UX/UI:
Investiga usuarios y diseña experiencia.

Marketing / Growth:
Comunica, posiciona y activa adopción.

Operaciones / DevOps:
Despliegue, monitoreo y continuidad.
```

En equipos pequeños, una persona puede cubrir varios roles. Lo importante es que las responsabilidades existan.

---

## 8. Matriz RACI

RACI aclara responsabilidades.

```text
R = Responsible     ejecuta el trabajo.
A = Accountable     responde finalmente por el resultado.
C = Consulted       debe ser consultado.
I = Informed        debe ser informado.
```

Ejemplo:

| Actividad | PM | PO | Tech Lead | Cliente | QA |
|---|---|---|---|---|---|
| Definir alcance | A | R | C | C | I |
| Diseñar arquitectura | I | C | A/R | I | C |
| Priorizar backlog | C | A/R | C | C | I |
| Ejecutar pruebas | I | C | C | I | A/R |
| Aprobar release | R | A | C | C | C |

Regla:

```text
Cada actividad debe tener un solo Accountable.
```

---

# Parte II: inicio del proyecto

## 9. Project charter

El project charter formaliza el inicio.

Debe incluir:

```text
Nombre del proyecto.
Problema.
Objetivos.
Alcance inicial.
Fuera de alcance.
Stakeholders.
Sponsor.
Equipo.
Restricciones.
Riesgos iniciales.
Criterios de éxito.
Presupuesto.
Fechas clave.
```

Plantilla:

```markdown
# Project Charter

## Proyecto

Nombre:

## Problema

Qué problema se busca resolver.

## Objetivo

Resultado esperado y métrica de éxito.

## Alcance

Qué se incluye.

## Fuera de alcance

Qué no se incluye.

## Stakeholders

Sponsor:
Cliente:
Usuarios:
Equipo:

## Restricciones

Tiempo:
Presupuesto:
Tecnología:
Regulatorio:

## Riesgos iniciales

1.
2.
3.

## Criterios de éxito

1.
2.
3.
```

---

## 10. Kickoff

El kickoff alinea expectativas.

Agenda recomendada:

```text
1. Contexto y problema.
2. Objetivos.
3. Alcance y fuera de alcance.
4. Roles.
5. Forma de trabajo.
6. Calendario inicial.
7. Canales de comunicación.
8. Riesgos conocidos.
9. Próximos pasos.
```

Resultado esperado:

```text
Todos entienden qué se hará, por qué, quién decide y cómo se medirá avance.
```

---

## 11. Alcance

Define el alcance con claridad.

Ejemplo:

```text
Incluye:
- Registro de usuarios.
- Login.
- Panel de tareas.
- CRUD de tareas.
- Búsqueda básica.
- Despliegue en staging y producción.

No incluye:
- App móvil.
- Integración con ERP.
- Analítica avanzada.
- Modo offline.
```

Regla:

```text
El fuera de alcance es tan importante como el alcance.
```

---

## 12. Criterios de aceptación

Un entregable está completo solo si cumple criterios verificables.

Formato útil:

```text
Dado...
Cuando...
Entonces...
```

Ejemplo:

```text
Dado un usuario autenticado,
cuando crea una tarea con título válido,
entonces la tarea queda registrada,
aparece en la lista
y se muestra mensaje de confirmación.
```

Criterios técnicos:

```text
Tests unitarios pasan.
Tests de integración pasan.
Logs estructurados.
Endpoint documentado.
No hay errores críticos de seguridad.
Despliegue reproducible.
```

---

## 13. Definition of Done

Definition of Done define cuándo algo se considera terminado.

Ejemplo:

```text
[ ] Código implementado.
[ ] Revisión de código aprobada.
[ ] Tests agregados.
[ ] Validaciones backend.
[ ] Manejo de errores.
[ ] Logs relevantes.
[ ] Documentación actualizada.
[ ] Desplegado en staging.
[ ] Validado por QA.
[ ] Aprobado por Product Owner.
```

Regla:

```text
Si no hay Definition of Done, cada persona inventará su propia definición de terminado.
```

---

# Parte III: clientes y stakeholders

## 14. Identificar stakeholders

Stakeholder es cualquier persona que puede afectar o ser afectada por el proyecto.

Tipos:

```text
Sponsor.
Cliente comprador.
Usuario final.
Equipo interno.
Soporte.
Legal.
Seguridad.
Operaciones.
Marketing.
Ventas.
Finanzas.
Proveedores.
Reguladores.
```

Preguntas:

```text
¿Quién paga?
¿Quién aprueba?
¿Quién usa?
¿Quién opera?
¿Quién da soporte?
¿Quién puede bloquear?
¿Quién se verá afectado?
```

---

## 15. Mapa de stakeholders

Clasifica por poder e interés.

| Poder | Interés | Estrategia |
|---|---|---|
| Alto | Alto | Gestionar de cerca |
| Alto | Bajo | Mantener satisfecho |
| Bajo | Alto | Mantener informado |
| Bajo | Bajo | Monitorear |

Ejemplo:

```text
Sponsor:
Alto poder, alto interés.

Área legal:
Alto poder, bajo o medio interés.

Usuarios frecuentes:
Bajo poder formal, alto interés.

Equipo externo:
Medio poder, alto interés operativo.
```

---

## 16. Contacto con clientes

El contacto con clientes debe ser sistemático, no improvisado.

Buenas prácticas:

```text
Define un punto de contacto principal.
Agenda reuniones regulares.
Documenta acuerdos.
Confirma decisiones por escrito.
Separa opinión de requerimiento.
Pregunta por problema antes que por solución.
Gestiona expectativas temprano.
No ocultes riesgos.
```

Antipatrón:

```text
Aceptar cambios por conversación informal y luego sorprender al equipo.
```

Mejor:

```text
Registrar el cambio, evaluar impacto y confirmar prioridad.
```

---

## 17. Entrevistas con clientes

Objetivo:

```text
Entender problemas, contexto, restricciones y valor esperado.
```

Preguntas útiles:

```text
¿Qué problema intenta resolver?
¿Cómo lo resuelve hoy?
¿Qué pasa si no se resuelve?
¿Quién lo sufre?
¿Cuánto cuesta el problema?
¿Cómo mediría una mejora?
¿Qué restricciones existen?
¿Qué intentos anteriores fallaron?
¿Qué sería inaceptable?
```

Evita preguntar solo:

```text
¿Qué funcionalidad quieres?
```

Mejor:

```text
¿Qué objetivo necesitas lograr?
¿Qué obstáculo te lo impide hoy?
```

---

## 18. Registro de decisiones

Mantén un decision log.

Plantilla:

```markdown
# Decisión

Fecha:
Proyecto:
Decisión:
Contexto:
Opciones consideradas:
Criterio:
Impacto:
Responsable:
```

Ejemplo:

```markdown
# Decisión

Fecha: 2026-07-05
Decisión: Usar PostgreSQL como base principal.
Contexto: El sistema requiere transacciones y reporting.
Opciones: PostgreSQL, MySQL, DynamoDB.
Impacto: El equipo usará migraciones SQL y backups relacionales.
Responsable: Tech Lead.
```

Beneficio:

```text
Reduce discusiones repetidas.
Ayuda a onboardear.
Permite auditar cambios.
```

---

# Parte IV: creación de productos

## 19. Discovery y delivery

Producto necesita dos ciclos:

```text
Discovery:
Entender problema, usuario, valor y solución posible.

Delivery:
Construir, probar, lanzar y operar.
```

Antipatrón:

```text
Saltar directo a construir sin validar problema.
```

Riesgos que discovery reduce:

```text
Construir algo que nadie usa.
Resolver el problema equivocado.
Sobreinvertir en una funcionalidad.
Ignorar restricciones reales.
```

---

## 20. Problema, solución y resultado

No confundas:

```text
Problema:
Los usuarios pierden tiempo buscando documentos.

Solución:
Buscador con filtros y ranking.

Resultado:
Reducir tiempo promedio de búsqueda de 8 minutos a 2 minutos.
```

Regla:

```text
Prioriza resultados, no solo funcionalidades.
```

---

## 21. Product vision

Una visión de producto responde:

```text
Para quién es.
Qué problema resuelve.
Qué valor entrega.
Cómo se diferencia.
Qué impacto busca.
```

Plantilla:

```markdown
Para [usuario objetivo],
que necesita [necesidad/problema],
el producto [nombre]
es una [categoría]
que permite [beneficio principal],
a diferencia de [alternativa],
porque [diferenciador].
```

Ejemplo:

```text
Para equipos técnicos que gestionan tareas operativas,
TaskOps es una plataforma de coordinación
que permite priorizar, ejecutar y medir trabajo recurrente,
a diferencia de hojas de cálculo,
porque integra responsables, estados, alertas y métricas de flujo.
```

---

## 22. Propuesta de valor

Debe conectar problema, usuario y beneficio.

Ejemplo débil:

```text
Nuestra app tiene dashboard, filtros y notificaciones.
```

Mejor:

```text
Ayudamos a equipos de soporte a reducir tiempos de respuesta centralizando solicitudes, priorizando urgencias y automatizando notificaciones.
```

Elementos:

```text
Segmento.
Dolor.
Beneficio.
Diferenciador.
Prueba o métrica.
```

---

## 23. MVP

MVP no significa producto incompleto o mal hecho. Significa versión mínima para aprender o entregar valor real.

Un MVP debe:

```text
Resolver un problema concreto.
Ser usable.
Ser medible.
Reducir incertidumbre.
Evitar trabajo innecesario.
```

Preguntas:

```text
¿Qué hipótesis queremos validar?
¿Cuál es la funcionalidad mínima para validarla?
¿Qué métrica confirmará aprendizaje?
¿Qué haremos si la hipótesis falla?
```

---

## 24. Roadmap

Un roadmap comunica dirección, no una lista rígida de promesas.

Formato recomendado:

```text
Now:
Lo que estamos haciendo.

Next:
Lo que probablemente sigue.

Later:
Oportunidades futuras.
```

Ejemplo:

| Horizonte | Objetivo | Iniciativas |
|---|---|---|
| Now | Reducir fricción de onboarding | Registro, tutorial, emails |
| Next | Mejorar retención | Recordatorios, plantillas |
| Later | Monetización | Planes, facturación, límites |

Regla:

```text
Mientras más lejano el horizonte, menor precisión.
```

---

## 25. Backlog de producto

El backlog contiene trabajo priorizado.

Items comunes:

```text
Features.
Bugs.
Deuda técnica.
Investigación.
Experimentos.
Mejoras UX.
Seguridad.
Observabilidad.
Documentación.
```

Un buen backlog debe:

```text
Tener prioridad.
Tener criterios de aceptación.
Ser refinado regularmente.
Eliminar items obsoletos.
Conectar con objetivos.
```

Antipatrón:

```text
Backlog como basurero infinito.
```

---

## 26. Priorización

Métodos útiles:

```text
MoSCoW.
RICE.
ICE.
WSJF.
Impacto vs esfuerzo.
Cost of Delay.
```

### MoSCoW

```text
Must have:
Imprescindible.

Should have:
Importante, pero no bloqueante.

Could have:
Deseable.

Won't have:
No se hará por ahora.
```

### RICE

```text
Reach:
A cuántos usuarios afecta.

Impact:
Cuánto impacto genera.

Confidence:
Qué tan seguros estamos.

Effort:
Cuánto esfuerzo requiere.
```

Fórmula:

```text
RICE = Reach × Impact × Confidence / Effort
```

Regla:

```text
La priorización no elimina juicio; lo hace visible.
```

---

# Parte V: planificación y seguimiento

## 27. Plan de proyecto

Un plan básico debe incluir:

```text
Objetivo.
Alcance.
Fases.
Entregables.
Responsables.
Fechas.
Dependencias.
Riesgos.
Métricas.
Comunicación.
```

Ejemplo de fases:

```text
1. Discovery.
2. Diseño.
3. Desarrollo.
4. QA.
5. Piloto.
6. Producción.
7. Seguimiento post-lanzamiento.
```

---

## 28. WBS

Work Breakdown Structure divide el trabajo en partes manejables.

Ejemplo:

```text
Proyecto: Plataforma de tareas

1. Producto
   1.1 Entrevistas
   1.2 Roadmap
   1.3 Backlog

2. UX/UI
   2.1 Wireframes
   2.2 Prototipo
   2.3 Diseño visual

3. Backend
   3.1 Modelo de datos
   3.2 API tareas
   3.3 Autenticación

4. Frontend
   4.1 Layout
   4.2 Lista tareas
   4.3 Formulario tareas

5. QA
   5.1 Unit tests
   5.2 Integration tests
   5.3 E2E

6. Deploy
   6.1 Staging
   6.2 Producción
```

---

## 29. Seguimiento semanal

Un seguimiento útil responde:

```text
¿Qué se completó?
¿Qué está en curso?
¿Qué está bloqueado?
¿Qué cambió?
¿Qué riesgo apareció?
¿Qué decisión se necesita?
¿Qué sigue?
```

Plantilla:

```markdown
# Estado semanal

## Resumen

Estado: Verde / Amarillo / Rojo

## Avances

- 

## Próximos pasos

- 

## Bloqueos

- 

## Riesgos

- 

## Decisiones requeridas

- 

## Cambios de alcance

- 

## Métricas

- 
```

---

## 30. Semáforo de estado

```text
Verde:
Avanza según plan.

Amarillo:
Hay riesgo, pero se puede corregir.

Rojo:
Hay desviación relevante que requiere decisión.
```

No uses amarillo o verde por optimismo. Usa evidencia.

Ejemplo:

```text
Rojo:
La integración con proveedor externo no está disponible y bloquea la fecha comprometida.

Decisión requerida:
Mover fecha, cambiar proveedor o reducir alcance.
```

---

## 31. Métricas de seguimiento

Métricas de proyecto:

```text
Avance por entregables.
Hitos cumplidos.
Riesgos abiertos.
Bloqueos.
Velocidad del equipo.
Defectos abiertos.
Tiempo de ciclo.
Burnup.
Burndown.
Lead time.
```

Métricas de producto:

```text
Usuarios activos.
Retención.
Conversión.
Uso de features.
NPS.
Churn.
Revenue.
Activation rate.
Tiempo a valor.
```

Regla:

```text
Mide resultado, no solo actividad.
```

---

## 32. Gestión de riesgos

Riesgo = evento incierto con impacto posible.

Plantilla:

```text
Riesgo:
Probabilidad:
Impacto:
Severidad:
Mitigación:
Contingencia:
Responsable:
Fecha de revisión:
```

Ejemplo:

```text
Riesgo:
El proveedor de pagos retrasa certificación.

Probabilidad:
Media.

Impacto:
Alto.

Mitigación:
Iniciar integración temprana y solicitar ambiente sandbox.

Contingencia:
Lanzar sin pagos automáticos usando facturación manual temporal.

Responsable:
PM + Tech Lead.
```

---

## 33. RAID log

RAID registra:

```text
Risks.
Assumptions.
Issues.
Dependencies.
```

Ejemplo:

| Tipo | Descripción | Responsable | Estado |
|---|---|---|---|
| Risk | API externa puede cambiar | Tech Lead | Abierto |
| Assumption | Cliente entregará datos de prueba | PM | Validar |
| Issue | Staging caído | DevOps | En curso |
| Dependency | Legal debe aprobar términos | Sponsor | Pendiente |

---

# Parte VI: derivas y control de cambios

## 34. Qué es una deriva

Una deriva ocurre cuando el proyecto se aleja del plan, objetivo o alcance original.

Tipos:

```text
Scope creep:
Aumento no controlado de alcance.

Deriva de objetivos:
Se empieza a resolver otro problema.

Deriva técnica:
La solución se vuelve más compleja de lo necesario.

Deriva de calendario:
Fechas se mueven sin decisión explícita.

Deriva de calidad:
Se recorta calidad para sostener fecha.

Deriva de stakeholders:
Aparecen nuevos decisores tarde.

Deriva de comunicación:
Personas clave dejan de estar alineadas.
```

---

## 35. Señales tempranas de deriva

```text
“Aprovechemos de agregar...”
“Esto es pequeño.”
“No hace falta documentarlo.”
“Después lo arreglamos.”
“El cliente lo pidió por WhatsApp.”
“No sabemos quién decide.”
“Todo es prioridad.”
“El backlog crece, pero la fecha no cambia.”
```

Regla:

```text
La deriva no siempre es mala.
Lo peligroso es que sea invisible.
```

---

## 36. Control de cambios

Plantilla de cambio:

```markdown
# Solicitud de cambio

## Descripción

Qué se quiere cambiar.

## Motivo

Por qué se solicita.

## Impacto

Alcance:
Tiempo:
Costo:
Riesgo:
Calidad:
Operación:

## Opciones

1.
2.
3.

## Recomendación

## Decisión

Aprobado / Rechazado / Diferido

## Responsable

## Fecha
```

---

## 37. Negociación de alcance

Cuando aparece un cambio, pregunta:

```text
¿Qué objetivo habilita?
¿Qué pasa si no se incluye?
¿Qué reemplaza?
¿Afecta fecha?
¿Afecta costo?
¿Afecta calidad?
¿Es requisito o preferencia?
```

Frase útil:

```text
Podemos incluirlo, pero necesitamos decidir qué se mueve: alcance, fecha, costo o prioridad.
```

---

## 38. Baseline

Una baseline es una versión acordada del plan.

Incluye:

```text
Alcance.
Fechas.
Costo.
Entregables.
Supuestos.
```

Cada cambio relevante se compara contra la baseline.

Regla:

```text
Sin baseline no hay deriva medible.
```

---

# Parte VII: comunicación y presentaciones

## 39. Plan de comunicación

Define:

```text
Quién necesita información.
Qué información necesita.
Con qué frecuencia.
Por qué canal.
Con qué formato.
Quién la envía.
```

Ejemplo:

| Audiencia | Frecuencia | Canal | Contenido |
|---|---|---|---|
| Sponsor | Semanal | Email | Estado ejecutivo |
| Equipo | Diario | Standup | Coordinación |
| Cliente | Quincenal | Reunión | Demo y decisiones |
| Soporte | Por release | Documento | Cambios operativos |

---

## 40. Presentaciones ejecutivas

Una presentación ejecutiva debe ser breve y orientada a decisiones.

Estructura:

```text
1. Objetivo.
2. Estado actual.
3. Avances.
4. Riesgos.
5. Decisiones requeridas.
6. Próximos pasos.
```

Evita:

```text
Muchos detalles técnicos.
Listas extensas de tareas.
Ocultar problemas.
Slides sin mensaje.
```

Mejor:

```text
Cada slide debe responder una pregunta.
```

---

## 41. Slide de estado

Formato:

```text
Título:
Estado general del proyecto

Contenido:
- Semáforo: Verde/Amarillo/Rojo.
- Avance principal.
- Riesgo principal.
- Decisión requerida.
- Próximo hito.
```

Ejemplo:

```text
Estado: Amarillo

Avance:
MVP completado al 70%.

Riesgo:
Integración con proveedor externo se retrasó 5 días.

Decisión:
Aprobar lanzamiento parcial sin integración automática.

Próximo hito:
Demo de staging el 15 de julio.
```

---

## 42. Storytelling para presentaciones

Estructura simple:

```text
Contexto:
Dónde estamos.

Conflicto:
Qué problema o riesgo existe.

Resolución:
Qué proponemos hacer.

Decisión:
Qué necesitamos aprobar.
```

Ejemplo:

```text
Contexto:
El MVP está funcional en staging.

Conflicto:
El módulo de reportes no cumple performance mínima.

Resolución:
Lanzar MVP sin reportes avanzados y mantener reportes básicos.

Decisión:
Aprobar ajuste de alcance para proteger fecha.
```

---

## 43. Demos

Una demo no debe ser improvisada.

Checklist:

```text
[ ] Ambiente listo.
[ ] Datos de prueba preparados.
[ ] Flujo ensayado.
[ ] Riesgos conocidos.
[ ] Plan B si falla internet o ambiente.
[ ] Objetivo de la demo claro.
[ ] Feedback registrado.
```

Estructura:

```text
1. Recordar objetivo.
2. Mostrar flujo.
3. Explicar qué está listo.
4. Explicar qué no está listo.
5. Pedir feedback concreto.
6. Confirmar próximos pasos.
```

---

# Parte VIII: marketing y go-to-market

## 44. Rol del marketing en proyectos de producto

Marketing no es solo publicidad. En producto digital ayuda a:

```text
Entender mercado.
Definir posicionamiento.
Comunicar valor.
Activar usuarios.
Lanzar producto.
Medir adquisición.
Medir conversión.
Recoger feedback.
```

Debe entrar antes del lanzamiento, no al final.

---

## 45. Posicionamiento

Define:

```text
Categoría.
Usuario objetivo.
Problema.
Beneficio.
Diferenciador.
Prueba.
```

Ejemplo:

```text
TaskOps es una plataforma de coordinación operativa para equipos técnicos que necesitan reducir trabajo invisible, a diferencia de hojas de cálculo, porque combina backlog, responsables, alertas y métricas de flujo en un solo lugar.
```

---

## 46. Mensaje principal

Un buen mensaje responde:

```text
¿Para quién?
¿Qué problema?
¿Qué resultado?
¿Por qué confiar?
```

Ejemplo:

```text
Reduce el tiempo de coordinación operativa de tu equipo centralizando tareas, responsables y alertas en una plataforma diseñada para operaciones técnicas.
```

---

## 47. Lanzamiento

Tipos de lanzamiento:

```text
Soft launch:
Lanzamiento controlado a pocos usuarios.

Beta:
Usuarios tempranos con feedback frecuente.

General Availability:
Disponible para todos.

Internal launch:
Lanzamiento interno.

Pilot:
Prueba con cliente o área específica.
```

Checklist:

```text
[ ] Segmento objetivo definido.
[ ] Mensaje preparado.
[ ] Landing o documentación lista.
[ ] Soporte preparado.
[ ] Métricas configuradas.
[ ] Feedback channel definido.
[ ] Riesgos y rollback claros.
```

---

## 48. Funnel básico

Modelo:

```text
Awareness:
Usuario conoce el producto.

Acquisition:
Usuario llega.

Activation:
Usuario obtiene primer valor.

Retention:
Usuario vuelve.

Revenue:
Usuario paga o genera valor económico.

Referral:
Usuario recomienda.
```

Métricas:

```text
Visitas.
Conversiones.
Registros.
Activación.
Retención.
Churn.
Ingresos.
Referidos.
```

---

## 49. Marketing y feedback loop

El marketing debe alimentar producto con evidencia:

```text
Qué mensaje convierte.
Qué segmento responde.
Qué objeciones aparecen.
Qué features se piden.
Qué casos de uso emergen.
Qué competidores se mencionan.
```

Regla:

```text
Marketing no solo comunica el producto; también ayuda a descubrirlo.
```

---

# Parte IX: metodologías ágiles

## 50. Qué significa ágil

Ágil no significa ausencia de planificación. Significa capacidad de adaptarse aprendiendo frecuentemente.

Principios prácticos:

```text
Entrega temprana y frecuente.
Colaboración con cliente.
Equipos autónomos.
Feedback continuo.
Software funcionando como medida de avance.
Mejora continua.
Adaptación al cambio.
```

Ágil es útil cuando:

```text
Hay incertidumbre.
El problema evoluciona.
El usuario necesita validar.
El alcance no está totalmente claro.
La tecnología tiene riesgo.
```

No es excusa para:

```text
No documentar.
No planificar.
No estimar.
No gestionar riesgos.
No cumplir compromisos.
```

---

## 51. Scrum

Scrum es un marco de trabajo ágil para desarrollar productos complejos.

Elementos principales:

```text
Roles:
Product Owner.
Scrum Master.
Developers.

Eventos:
Sprint.
Sprint Planning.
Daily Scrum.
Sprint Review.
Sprint Retrospective.

Artefactos:
Product Backlog.
Sprint Backlog.
Increment.

Compromisos:
Product Goal.
Sprint Goal.
Definition of Done.
```

---

## 52. Roles de Scrum

### Product Owner

Responsable de maximizar valor.

Responsabilidades:

```text
Gestionar Product Backlog.
Definir prioridades.
Alinear stakeholders.
Clarificar objetivos.
Aceptar o rechazar trabajo según criterios.
```

### Scrum Master

Responsable de la efectividad de Scrum.

Responsabilidades:

```text
Facilitar eventos.
Eliminar impedimentos.
Ayudar al equipo a mejorar.
Proteger foco.
Promover empirismo.
```

### Developers

Personas que construyen el incremento.

Responsabilidades:

```text
Crear plan del Sprint.
Construir incremento.
Asegurar calidad.
Adaptar trabajo diario.
Cumplir Definition of Done.
```

---

## 53. Eventos de Scrum

### Sprint

Contenedor temporal, usualmente de 1 a 4 semanas.

Objetivo:

```text
Crear un incremento usable.
```

### Sprint Planning

Define:

```text
Por qué este Sprint es valioso.
Qué se hará.
Cómo se hará.
```

Resultado:

```text
Sprint Goal.
Sprint Backlog.
Plan inicial.
```

### Daily Scrum

Reunión breve para inspeccionar avance hacia Sprint Goal.

No es reporte al jefe.

Preguntas útiles:

```text
¿Estamos avanzando hacia el Sprint Goal?
¿Qué bloquea?
¿Qué ajustamos hoy?
```

### Sprint Review

Inspección del incremento con stakeholders.

Objetivo:

```text
Recoger feedback y ajustar Product Backlog.
```

### Sprint Retrospective

El equipo inspecciona su forma de trabajar.

Resultado:

```text
Acciones de mejora concretas.
```

---

## 54. Artefactos de Scrum

### Product Backlog

Lista ordenada de trabajo para el producto.

Debe estar alineado al Product Goal.

### Sprint Backlog

Trabajo seleccionado para el Sprint más el plan para entregarlo.

Debe estar alineado al Sprint Goal.

### Increment

Resultado terminado y usable.

Debe cumplir Definition of Done.

Regla:

```text
Si no cumple Definition of Done, no es incremento terminado.
```

---

## 55. Sprint Goal

Un Sprint Goal da foco.

Malo:

```text
Completar tickets 123, 124, 125 y 126.
```

Mejor:

```text
Permitir que usuarios creen y organicen tareas básicas desde la web.
```

Beneficio:

```text
El equipo puede adaptar detalles sin perder objetivo.
```

---

## 56. Scrum bien aplicado

Scrum requiere:

```text
Transparencia.
Inspección.
Adaptación.
Backlog ordenado.
Sprint Goal claro.
Incremento usable.
Retrospectivas honestas.
Stakeholders disponibles.
```

Scrum mal aplicado se ve así:

```text
Daily como reporte de estado.
Sprint sin objetivo.
PO ausente.
Backlog enorme sin prioridad.
Retros sin acciones.
Trabajo no terminado arrastrado siempre.
No se muestra producto funcionando.
```

---

## 57. Kanban

Kanban se basa en visualizar trabajo, limitar trabajo en progreso y mejorar flujo.

Elementos:

```text
Tablero.
Columnas.
WIP limits.
Políticas explícitas.
Métricas de flujo.
Mejora continua.
```

Ejemplo:

```text
Backlog -> Ready -> In Progress -> Review -> QA -> Done
```

WIP limit:

```text
In Progress máximo 3.
Review máximo 2.
QA máximo 2.
```

Métricas:

```text
Cycle time.
Lead time.
Throughput.
Work in progress.
Cumulative flow diagram.
```

Kanban es útil para:

```text
Soporte.
Mantenimiento.
Operaciones.
Flujos continuos.
Equipos con prioridades cambiantes.
```

---

## 58. Scrum vs Kanban

| Aspecto | Scrum | Kanban |
|---|---|---|
| Cadencia | Sprints | Flujo continuo |
| Roles | Definidos | No prescribe roles |
| Planificación | Por Sprint | Continua |
| Cambio | Controlado dentro del Sprint | Puede entrar si hay capacidad |
| Métrica | Velocidad, goal, incremento | Cycle time, WIP, throughput |

Regla práctica:

```text
Usa Scrum cuando necesitas cadencia de producto y aprendizaje por incrementos.
Usa Kanban cuando necesitas flujo continuo y gestión de demanda variable.
```

---

## 59. Scrumban

Scrumban combina elementos:

```text
Cadencia de reuniones de Scrum.
Tablero y WIP limits de Kanban.
Planificación flexible.
Métricas de flujo.
```

Útil para equipos que:

```text
Tienen producto, pero también mucho soporte.
No pueden proteger Sprints completamente.
Necesitan visualizar bloqueos.
```

---

## 60. Estimación

Métodos:

```text
Story points.
T-shirt sizes.
Estimación por horas.
No estimates.
Rangos.
```

Story points miden esfuerzo relativo, no horas exactas.

Ejemplo:

```text
1, 2, 3, 5, 8, 13
```

Reglas:

```text
Estima para conversar, no para castigar.
Actualiza planes con datos reales.
No compares velocidad entre equipos.
```

---

## 61. Planning poker

Proceso:

```text
1. Se lee item.
2. Se aclaran dudas.
3. Cada persona estima en secreto.
4. Se revelan estimaciones.
5. Se discuten diferencias.
6. Se acuerda estimación.
```

El valor está en la conversación, no en el número.

---

## 62. Velocity

Velocity es cuánto trabajo termina un equipo por Sprint.

Uso razonable:

```text
Forecast interno.
Capacidad aproximada.
Tendencia histórica.
```

Mal uso:

```text
Comparar equipos.
Presionar para subir puntos.
Convertir puntos en horas rígidas.
```

---

## 63. Burnup y burndown

Burndown:

```text
Muestra trabajo restante.
```

Burnup:

```text
Muestra trabajo completado y alcance total.
```

Para proyectos con alcance cambiante, burnup suele mostrar mejor derivas de alcance.

---

# Parte X: gestión avanzada

## 64. Gobernanza de proyectos

Gobernanza define cómo se toman decisiones.

Debe incluir:

```text
Quién aprueba presupuesto.
Quién aprueba cambios de alcance.
Quién prioriza.
Quién acepta entregables.
Cómo se escalan riesgos.
Cómo se reporta avance.
Qué métricas se usan.
Qué documentación es obligatoria.
```

Sin gobernanza, los proyectos dependen de relaciones informales.

---

## 65. Comité de proyecto

Útil para proyectos grandes.

Agenda:

```text
1. Estado general.
2. Hitos.
3. Riesgos.
4. Presupuesto.
5. Cambios de alcance.
6. Decisiones requeridas.
7. Próximos pasos.
```

Participantes:

```text
Sponsor.
PM.
Product Owner.
Tech Lead.
Representante cliente.
Operaciones/seguridad si aplica.
```

Regla:

```text
Un comité sin decisiones es solo una reunión de reporte.
```

---

## 66. Gestión de dependencias

Dependencias típicas:

```text
Equipo externo.
Proveedor.
Legal.
Seguridad.
Infraestructura.
Datos.
Diseño.
Aprobación cliente.
Integración técnica.
```

Registro:

```text
Dependencia:
Responsable:
Fecha requerida:
Impacto si falla:
Estado:
Plan alternativo:
```

---

## 67. Gestión de proveedores

Buenas prácticas:

```text
Contrato claro.
SLA si aplica.
Responsables definidos.
Canal de soporte.
Ambiente de pruebas.
Documentación técnica.
Fechas de entrega.
Penalidades o criterios de aceptación si corresponde.
```

Riesgos:

```text
Proveedor no responde.
API cambia.
Sandbox no refleja producción.
Costos ocultos.
Dependencia crítica sin alternativa.
```

---

## 68. Gestión de calidad

Calidad no debe aparecer al final.

Incluye:

```text
Definition of Done.
QA plan.
Pruebas automatizadas.
Revisión de código.
Pruebas de seguridad.
Pruebas de performance.
Revisión UX.
Monitoreo post-release.
```

Métricas:

```text
Defectos por release.
Defectos reabiertos.
Tiempo de resolución.
Cobertura de pruebas.
Incidentes post-release.
Satisfacción del usuario.
```

---

## 69. Gestión de deuda técnica

Deuda técnica no siempre es mala. Es mala cuando no se reconoce ni paga.

Tipos:

```text
Código difícil de mantener.
Falta de tests.
Arquitectura temporal.
Dependencias obsoletas.
Documentación inexistente.
Observabilidad débil.
Seguridad pendiente.
```

Gestión:

```text
Registrar deuda.
Clasificar impacto.
Incluir en backlog.
Reservar capacidad.
Pagar deuda antes de que bloquee.
```

Regla:

```text
Si la deuda aumenta el costo de cambio, debe hacerse visible.
```

---

## 70. Gestión de incidentes

En productos vivos, incidentes son parte de la operación.

Proceso:

```text
Detección.
Clasificación.
Contención.
Resolución.
Comunicación.
Postmortem.
Acciones preventivas.
```

Postmortem sin culpa:

```text
Qué ocurrió.
Impacto.
Línea de tiempo.
Causas contribuyentes.
Qué funcionó.
Qué no funcionó.
Acciones.
Responsables.
Fechas.
```

---

## 71. SLO, SLA y KPI

```text
SLO:
Objetivo interno de nivel de servicio.

SLA:
Compromiso formal con cliente.

KPI:
Indicador de desempeño de negocio o proceso.
```

Ejemplo:

```text
SLO:
99.9% disponibilidad mensual.

SLA:
99.5% disponibilidad comprometida contractualmente.

KPI:
Tiempo promedio de resolución de solicitudes.
```

---

## 72. Gestión de portafolio

Cuando hay muchos proyectos, necesitas decidir dónde invertir.

Criterios:

```text
Valor estratégico.
Retorno esperado.
Riesgo.
Costo.
Capacidad.
Dependencias.
Urgencia.
Cumplimiento legal.
Impacto cliente.
```

Matriz simple:

| Proyecto | Valor | Esfuerzo | Riesgo | Prioridad |
|---|---:|---:|---:|---:|
| A | Alto | Medio | Medio | Alta |
| B | Medio | Alto | Alto | Baja |
| C | Alto | Bajo | Bajo | Muy alta |

---

# Parte XI: herramientas

## 73. Herramientas comunes

Gestión de trabajo:

```text
Jira.
Trello.
Asana.
Linear.
Azure DevOps.
GitHub Projects.
ClickUp.
Notion.
Monday.
```

Documentación:

```text
Confluence.
Notion.
Google Docs.
Markdown en Git.
SharePoint.
```

Comunicación:

```text
Slack.
Microsoft Teams.
Email.
Reuniones.
Loom.
```

Producto y diseño:

```text
Figma.
Miro.
FigJam.
Productboard.
Jira Product Discovery.
Amplitude.
Mixpanel.
PostHog.
Google Analytics.
```

Regla:

```text
La herramienta debe soportar el proceso.
No debe reemplazar claridad de prioridades y decisiones.
```

---

## 74. Tablero mínimo

Columnas:

```text
Backlog.
Ready.
In Progress.
Review.
QA.
Done.
```

Campos por item:

```text
Título.
Descripción.
Prioridad.
Responsable.
Estado.
Criterios de aceptación.
Fecha objetivo.
Dependencias.
Riesgo.
Link a diseño/documentación.
```

---

## 75. Dashboard de proyecto

Debe mostrar:

```text
Estado.
Avance.
Riesgos.
Bloqueos.
Hitos.
Trabajo en curso.
Defectos.
Decisiones pendientes.
```

Dashboard de producto:

```text
Usuarios activos.
Retención.
Conversión.
Uso de features.
Feedback.
Ingresos si aplica.
Churn.
```

---

# Parte XII: plantillas prácticas

## 76. Plantilla de minuta

```markdown
# Minuta

Fecha:
Proyecto:
Participantes:

## Objetivo de la reunión

## Temas tratados

1.
2.
3.

## Decisiones

| Decisión | Responsable | Fecha |
|---|---|---|

## Acciones

| Acción | Responsable | Fecha |
|---|---|---|

## Riesgos o bloqueos

## Próxima reunión
```

---

## 77. Plantilla de status report

```markdown
# Status Report

Proyecto:
Fecha:
Responsable:
Estado: Verde / Amarillo / Rojo

## Resumen ejecutivo

## Avances desde el último reporte

## Próximos hitos

## Riesgos

## Bloqueos

## Cambios de alcance

## Decisiones requeridas

## Métricas

## Comentarios
```

---

## 78. Plantilla de caso de negocio

```markdown
# Caso de negocio

## Problema

## Oportunidad

## Usuarios afectados

## Solución propuesta

## Beneficios esperados

## Costos estimados

## Riesgos

## Alternativas

## Métricas de éxito

## Recomendación
```

---

## 79. Plantilla de PRD

PRD = Product Requirements Document.

```markdown
# Product Requirements Document

## Contexto

## Problema

## Objetivos

## No objetivos

## Usuarios

## Casos de uso

## Requerimientos funcionales

## Requerimientos no funcionales

## Criterios de aceptación

## Métricas

## Riesgos

## Dependencias

## Diseño

## Plan de lanzamiento
```

---

## 80. Plantilla de retrospectiva

```markdown
# Retrospectiva

Sprint / Periodo:
Fecha:

## Qué funcionó bien

## Qué no funcionó

## Qué aprendimos

## Acciones de mejora

| Acción | Responsable | Fecha |
|---|---|---|

## Experimento para el próximo ciclo
```

---

# Parte XIII: checklists

## 81. Checklist de inicio

```text
[ ] Problema definido.
[ ] Objetivo definido.
[ ] Sponsor identificado.
[ ] Cliente identificado.
[ ] Usuarios identificados.
[ ] Alcance inicial definido.
[ ] Fuera de alcance definido.
[ ] Criterios de éxito definidos.
[ ] Riesgos iniciales registrados.
[ ] Equipo asignado.
[ ] Canales de comunicación definidos.
[ ] Kickoff realizado.
```

---

## 82. Checklist de producto

```text
[ ] Usuario objetivo claro.
[ ] Propuesta de valor clara.
[ ] Problema validado.
[ ] Métricas de producto definidas.
[ ] Roadmap definido.
[ ] Backlog priorizado.
[ ] MVP definido.
[ ] Feedback loop activo.
[ ] Plan de lanzamiento definido.
[ ] Marketing alineado.
```

---

## 83. Checklist de seguimiento

```text
[ ] Estado actualizado.
[ ] Avances registrados.
[ ] Bloqueos visibles.
[ ] Riesgos revisados.
[ ] Dependencias revisadas.
[ ] Cambios de alcance registrados.
[ ] Decisiones pendientes identificadas.
[ ] Próximos pasos claros.
[ ] Stakeholders informados.
```

---

## 84. Checklist Scrum

```text
[ ] Product Owner disponible.
[ ] Scrum Master o facilitador claro.
[ ] Developers definidos.
[ ] Product Goal definido.
[ ] Product Backlog ordenado.
[ ] Sprint Goal definido.
[ ] Sprint Backlog claro.
[ ] Daily orientada a adaptación.
[ ] Sprint Review con stakeholders.
[ ] Retrospective con acciones.
[ ] Definition of Done aplicada.
```

---

## 85. Checklist de presentación

```text
[ ] Audiencia clara.
[ ] Objetivo de la presentación claro.
[ ] Mensaje principal definido.
[ ] Estado resumido.
[ ] Riesgos visibles.
[ ] Decisiones requeridas explícitas.
[ ] Datos actualizados.
[ ] Demo ensayada si aplica.
[ ] Próximos pasos claros.
```

---

## 86. Checklist de cierre

```text
[ ] Entregables aceptados.
[ ] Documentación entregada.
[ ] Operación transferida.
[ ] Soporte definido.
[ ] Métricas post-lanzamiento configuradas.
[ ] Riesgos residuales comunicados.
[ ] Lecciones aprendidas registradas.
[ ] Contratos cerrados si aplica.
[ ] Presupuesto cerrado.
[ ] Cierre comunicado.
```

---

# Parte XIV: resumen de reglas principales

```text
1. Define problema antes de solución.
2. Distingue proyecto de producto.
3. Especifica alcance y fuera de alcance.
4. Define éxito con métricas.
5. Identifica stakeholders temprano.
6. Documenta decisiones.
7. Mantén comunicación regular y objetiva.
8. Mide avance con evidencia, no con percepción.
9. Gestiona riesgos antes de que sean incidentes.
10. Haz visibles las derivas.
11. Cambios de alcance deben tener evaluación de impacto.
12. Roadmap no es promesa rígida.
13. Backlog no debe ser basurero.
14. Marketing debe participar antes del lanzamiento.
15. Scrum requiere foco, incremento y retrospectiva real.
16. Kanban sirve para flujo continuo y trabajo variable.
17. La herramienta no reemplaza la gestión.
18. Toda reunión debe producir claridad, decisión o avance.
19. Producto exitoso se mide por valor, no por cantidad de features.
20. Cerrar bien un proyecto es parte de gestionarlo bien.
```

---

# Fuentes de referencia recomendadas

```text
- Agile Manifesto.
- Principles behind the Agile Manifesto.
- The Scrum Guide 2020.
- Project Management Institute, PMBOK Guide.
- PMI Project Performance Domains.
- Kanban Guide.
- Atlassian Agile Coach.
- Scrum.org resources.
- Product management discovery and delivery literature.
```
