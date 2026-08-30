# Guía Maestra: Investigación Camunda + bpmn.io
### Informática Aplicada a los Negocios II — UCR 2026-G02

---

## 0. Resumen de decisiones (basado en tu contexto)

| Decisión | Recomendación | Por qué |
|---|---|---|
| **Motor/plataforma** | Camunda 8 (Zeebe) vía **"Camunda 8 Run"** con Docker | Es la versión que Camunda promueve activamente hoy (cloud-native), tiene mejores "casos de éxito" recientes para citar, y con su experiencia en Docker el setup es manejable en los 15 min asignados a consultas de instalación. |
| **Modelador (bpmn.io)** | **Camunda Desktop Modeler** | Es la aplicación oficial que empaqueta la librería bpmn.io — cumple el requisito explícito de "Camunda + bpmn.io" del enunciado sin que ustedes tengan que integrar la librería a mano. |
| **Monitoreo/"mining-lite"** | **Operate** (incluido en Camunda 8 Run) | Permite mostrar métricas de instancias de proceso (tiempos, cuellos de botella), lo cual da algo de profundidad analítica aunque Camunda no es una herramienta de minería pura. |
| **Automatización de decisiones** | **DMN (Decision Model and Notation)**, incluido en Camunda | Les permite automatizar reglas de negocio (ej. aprobación de crédito, escalamiento de pedidos) usando datos reales del dataset — esto cubre el criterio de "profundidad en funcionalidades" y "automatización de procesos" del rubro. |

> ⚠️ Nota importante: mi conocimiento sobre versiones exactas de Camunda puede estar desactualizado (mi corte de entrenamiento es enero 2026). **Antes de grabar el video de instalación, verifiquen los pasos exactos en `docs.camunda.io`** porque los comandos de instalación cambian entre versiones menores.

---

## 1. Dataset y proceso de negocio sugerido

**Propuesta: Order-to-Cash / Cumplimiento de pedidos e-commerce**, usando el dataset público:

> **Brazilian E-Commerce Public Dataset by Olist** (Kaggle, ~99,441 pedidos, datos reales anonimizados de una plataforma brasileña de e-commerce)

**Por qué este dataset:**
- Es real, verificable, tiene procedencia clara (empresa Olist, publicado en Kaggle) — fácil de citar correctamente en APA7.
- Incluye pedidos, pagos, entregas, reseñas → encaja directo con los ejemplos del enunciado ("compras, pedidos, entregas, facturación").
- Tiene suficiente volumen y variables (fechas de aprobación, envío, entrega, cancelaciones, métodos de pago) para justificar un proceso *"suficientemente complejo"* como exige el rubro.

**Caso hipotético sugerido:** una empresa ficticia de e-commerce ("TicoShop" o el nombre que definan) implementa Camunda para modelar y ejecutar su proceso de **"Pedido a Entrega"**, con puntos de decisión automatizados:
- Validación de pago (tarea de servicio / DMN con reglas basadas en montos y método de pago reales del dataset).
- Escalamiento si el envío se demora más de X días (basado en percentiles reales de tiempos de entrega del dataset).
- Cancelación/reembolso si el pago es rechazado.

**Preguntas de negocio que el workshop debe responder** (ejemplos, ajústenlas):
1. ¿Cuánto tiempo promedio transcurre entre aprobación de pago y entrega, y cuántos pedidos exceden el SLA definido?
2. ¿Qué porcentaje de pedidos requiere escalamiento manual según las reglas automatizadas?
3. ¿Cómo cambia el flujo cuando se automatiza la validación de pago vs. un proceso manual?

**Alternativas por si el dataset de Olist no las convence:**
- *Procure-to-Pay*: dataset sintético de órdenes de compra (deben generarlo o buscar uno en Kaggle tipo "Purchase Order Data").
- *Aprobación de crédito/préstamos*: buenos para lucir DMN, pero menos alineado a los ejemplos del enunciado.

---

## 2. Cronograma retroactivo (exposición: **lunes 28 de septiembre**)

| Fecha límite | Entregable | Nota del enunciado |
|---|---|---|
| Cuanto antes (idealmente esta semana) | Enviar **propuesta de contenidos del workshop** por correo institucional a la profesora | Se envía por correo, no por Mediación Virtual |
| ~1–2 semanas antes | Tener el **proceso BPMN modelado** y el **dataset limpio/analizado** | Base para todo lo demás |
| **Jueves 25 de septiembre** (3 días antes) | Enviar **video de instalación/configuración** (vía TEAMS o correo con enlace) | Obligatorio, no negociable según el enunciado |
| Días previos | Grabar **video corto de explicación de opciones** de la herramienta (menú, importar/exportar, etc., SIN mostrar el desarrollo del workshop) | Documentación a distribuir |
| Antes del 28 | Terminar **documento gerencial**, **guía del taller**, **documento técnico**, **presentación**, **tarea corta para compañeros** | Ver checklist detallado abajo |
| **Lunes 28 de septiembre, 8:00 a.m.** | Entrega final: **archivo comprimido** con nombre de la herramienta, vía Mediación Virtual | "No se dará más plazo" — literal del enunciado |
| Día de la exposición | Presentación (gerencial 20 min + workshop 80 min + dudas 20 min) | Tiempos medidos |

---

## 3. Checklist maestro por entregable

### ✅ A. Propuesta de contenidos (1%)
- [ ] Redactar un resumen de 1 página: qué proceso de negocio usarán, qué dataset, qué funcionalidades de Camunda/bpmn.io planean mostrar.
- [ ] Enviar por correo institucional a la profesora (no Mediación Virtual).

### ✅ B. Manual de instalación + video (2%)
- [ ] Instalar Camunda 8 Run + Camunda Modeler en un ambiente limpio (idealmente una VM o carpeta nueva) para poder documentar el proceso real, sin pasos "de memoria".
- [ ] Escribir manual paso a paso con capturas de pantalla (requisitos, comandos, verificación de que corrió bien).
- [ ] Grabar sesión de instalación/configuración (15 min de tiempo asignado a consultas — practíquenla antes).
- [ ] Publicar el video 3 días antes (25 de septiembre) vía TEAMS o correo con enlace.

### ✅ C. Documentos y archivos para ejecutar el workshop (1%)
- [ ] Archivo(s) con el dataset ya preparado (CSV/Excel limpio).
- [ ] Archivo(s) `.bpmn` del proceso modelado.
- [ ] Si usan fórmulas/reglas DMN, documento `.txt` explicando cada una.

### ✅ D. Documento gerencial (2%) — puntos mínimos exigidos
- [ ] **Contexto:** descripción del "log de datos" (dataset) y caso hipotético de implementación en la empresa ficticia.
- [ ] **Justificación:** por qué esta empresa adquiriría e implementaría Camunda.
- [ ] **Ventajas y desventajas.**
- [ ] **Planes y costos** (Camunda tiene ediciones open-source/community y planes pagos enterprise — investigar y citar fuente oficial).
- [ ] **Casos de éxito** (buscar en el sitio oficial de Camunda, sección de clientes/casos de estudio).
- [ ] **Referencias en APA7** (ver sección 5).

### ✅ E. Workshop: demostración técnica (6%)
- [ ] Ejercicio con complejidad real (no un "hola mundo"): debe usar varias funcionalidades (modelado + ejecución + al menos una automatización tipo DMN o tarea de servicio).
- [ ] Explicar las decisiones técnicas tomadas (por qué modelaron así, por qué esas reglas de negocio).
- [ ] Basar el ejercicio en el proceso de negocio ya definido (contexto, participantes, actividades, objetivo).
- [ ] Ejecutar el ejercicio paso a paso y guiado en clase.

### ✅ F. Tarea corta para compañeros (1%)
- [ ] Diseñar 1–2 consignas que extiendan lo visto en el workshop (ej. "agreguen una nueva regla de escalamiento" o "modelen una variante del proceso").
- [ ] Preparar presentación breve de la pregunta de negocio + resultado esperado.
- [ ] Si hay fórmulas, documento `.txt` con explicación.

### ✅ G. Presentación y exposición (3% + 2%)
- [ ] Presentación clara, dentro del tiempo, con participación de todos.
- [ ] Practicar transiciones entre "gerencial" y "técnico" para no perder tiempo.
- [ ] Preparar respuestas anticipadas a preguntas típicas (seguridad, escalabilidad, costos, diferencias vs. otras herramientas BPM).

### ✅ H. Presentación de tareas asignadas (2%)
- [ ] Elegir 2 de las 3 tareas cortas creadas por otros grupos de investigación.
- [ ] Resolverlas y grabar video explicativo (participan todos los integrantes).
- [ ] Documento de máximo 1 página por tarea resuelta.

---

## 4. Instalación paso a paso (borrador — verificar versión actual en docs.camunda.io)

1. **Requisitos previos:** Docker Desktop instalado y corriendo; al menos 4–8 GB de RAM libres asignados a Docker.
2. **Descargar Camunda 8 Run:** desde el repositorio oficial de GitHub de Camunda (`camunda/camunda-platform` o la página de descargas de `camunda.com/download`), obtener la distribución "Camunda 8 Run" más reciente.
3. **Iniciar el entorno:** ejecutar el script de arranque incluido (`start.sh` en Linux/Mac o `start.bat` en Windows) o el `docker-compose` correspondiente si usan esa variante.
4. **Verificar servicios activos:**
   - Operate: normalmente en `http://localhost:8081`
   - Tasklist: normalmente en `http://localhost:8082`
5. **Instalar Camunda Desktop Modeler:** descargar desde `camunda.com/download/modeler/` e instalar como aplicación de escritorio.
6. **Conectar el Modeler al motor local** (deployment del diagrama BPMN vía el botón de despliegue en el Modeler).
7. **Prueba de humo:** desplegar un proceso simple ("Hola mundo") y verificar que aparece una instancia en Operate.

> Documenten cada paso con captura de pantalla real de **su** instalación — esto también protege contra sospechas de plagio/IA, porque son evidencias únicas de su propio proceso.

---

## 5. Fuentes confiables sugeridas para citar en APA7

**Importante:** yo no tengo acceso a internet en este momento, así que **no puedo darles URLs o datos exactos verificados**; lo que sigue son *puntos de partida* que ustedes deben localizar, leer y citar directamente desde la fuente original. Nunca usen una cita que no hayan verificado abriendo la fuente.

- **Documentación oficial de Camunda** — `docs.camunda.io` (para conceptos técnicos, arquitectura, funcionalidades).
- **Sitio de casos de clientes de Camunda** — sección "Customers" o "Case Studies" en `camunda.com` (para "casos de éxito").
- **Especificación oficial de BPMN** — Object Management Group (OMG), `omg.org/spec/BPMN` (para fundamentar el enfoque de modelado).
- **Dataset:** Olist. *Brazilian E-Commerce Public Dataset by Olist* [Data set]. Kaggle. (Busquen la página exacta en Kaggle y citen con la fecha de acceso).
- **Libro académico de referencia del curso:** Dumas, M., La Rosa, M., Mendling, J., & Reijers, H. A. (2018). *Fundamentals of Business Process Management* (2nd ed.). Springer. — útil para fundamentar teóricamente BPM/BPMN sin depender solo de fuentes comerciales.
- **Material del curso:** las diapositivas de "Introducción a BPM" que su profesora ya compartió (citar como material de clase según formato APA7 para comunicaciones/presentaciones no publicadas, o preguntarle a la profesora el formato preferido).

**Formato de referencia APA7 (plantilla genérica):**
```
Apellido, A. A. (Año). Título del recurso en cursiva. Nombre del sitio. URL
```

---

## 6. Buenas prácticas para evitar problemas de integridad académica

1. **Usen esta guía como andamiaje, no como texto final.** Redacten cada sección del documento gerencial y técnico con sus propias palabras y su propio análisis del dataset.
2. **Guarden evidencia de su proceso real:** capturas propias, historial de versiones del documento, grabaciones de su propia instalación. Esto es lo que más los protege ante sospechas de IA/plagio.
3. **Verifiquen cada cita abriendo la fuente original** antes de ponerla en el documento — no confíen en citas generadas por IA (incluida esta conversación) sin comprobarlas.
4. **Documenten sus decisiones técnicas explicando el "por qué"**, no solo el "qué" — eso es difícil de plagiar y es justo lo que pide el rubro.
5. Si usan IA como esta para organizarse o entender conceptos, verifiquen las políticas de su curso sobre divulgación de uso de IA y sigan lo que la profesora indique.

---

## Próximos pasos inmediatos que sugiero

1. Definan el nombre de su empresa ficticia y confirmen el dataset (Olist u otro).
2. Instalen Camunda 8 Run + Modeler en un ambiente de prueba esta semana.
3. Avísenme cuando tengan el proceso BPMN en borrador — puedo ayudarles a revisarlo, sugerir mejoras de notación BPMN, o a diseñar las reglas DMN basadas en los datos reales.
