# Explicación de reglas DMN y conexión con el proceso BPMN

Documento que explica cada regla/fórmula usada en las tablas de decisión (`.dmn`) de esta carpeta y cómo se conectan al proceso BPMN principal.

> Justificación con datos reales de cada regla: ver [`02-dataset/calculos-justificacion-reglas.md`](../02-dataset/calculos-justificacion-reglas.md).

## Fase 3 — Conexión de tablas DMN al proceso BPMN

Actualización: 4 de septiembre, 2026

### Componentes construidos

- [`Prueba1.bpmn`](Prueba1.bpmn): proceso "Pedido a Entrega" con 2 pools:
  - **Pool 1 "MaxiMundo"** (lanes: Sistema de pagos, Logística) — proceso principal (`Process_0s5oq2r`)
  - **Pool 2 "MaxiMundo - Atención al Cliente"** (lane: Atención al Cliente) — proceso secundario relacionado por secuencia (`Process_1lwe2il`)
- [`determinarConfirmacionPago.dmn`](determinarConfirmacionPago.dmn): tabla DMN, hit policy **Unique** (`Decision_04dp310`)
  - Input: `metodoPago` (string) — Output: `requiereEspera` (boolean)
  - Regla: `"boleto"` → `true`; `"credit_card"`, `"debit_card"`, `"voucher"` → `false`
- [`determinarRiesgoCuotas.dmn`](determinarRiesgoCuotas.dmn): tabla DMN, hit policy **Unique** (`Decision_0jivutp`)
  - Input: `numeroCuotas` (number) — Output: `requiereRevision` (boolean)
  - Regla: `> 6` → `true`; `<= 6` → `false`

### Conexión BPMN ↔ DMN

- Se usó una **Business Rule Task** (Implementation: "DMN decision") para invocar cada tabla.
- Se configuró el **Result variable** de cada tarea para capturar el output de la tabla (`requiereEspera`, `requiereRevision`).
- Los gateways posteriores usan expresiones **FEEL** sobre esas variables (ej. `=requiereEspera = true`), no comparaciones directas contra el texto original del método de pago.

### Errores encontrados y resueltos

1. **"Process has multiple blank start events"** — ocurrió por modelar dos procesos independientes (principal y atención al cliente) como *lanes* dentro del mismo pool.
   **Solución:** separarlos en dos pools distintos — cada pool es un `<bpmn:Process>` independiente en BPMN.

2. **"A Sequence Flow must have a defined Condition expression"** — ocurrió por escribir el texto de la condición como simple etiqueta de la flecha, en vez de configurarlo en el campo "Condition expression" del panel Properties.
   **Solución:** usar ese campo real, y marcar el camino restante de cada gateway como "flujo por defecto" cuando aplica.

3. **"Extract value error" / "NO_VARIABLE_FOUND"** al invocar la segunda tabla DMN — causado por escribir un Decision ID incorrecto en el campo de la Business Rule Task: primero se usó por error el ID del archivo completo (`Definitions_...`) en vez del ID de la decisión específica (`Decision_...`); luego, aun con el ID correcto, el campo parecía estar en modo "expresión FEEL" en vez de "valor fijo", causando que tratara el ID como nombre de variable a buscar.
   **Solución:** verificar que el campo Decision ID esté en modo valor fijo, no en modo expresión (ícono "fx").

### Prueba de extremo a extremo

4 instancias de prueba ejecutadas con variables `metodoPago` y `numeroCuotas` variadas — todas completadas exitosamente sin incidentes, confirmando el funcionamiento correcto de ambas tablas DMN conectadas al proceso.

### Próximo paso

Preparar una muestra de datos reales del dataset de Olist para usarla como variables de instancias representativas del negocio (no solo pruebas técnicas), y correr esas instancias para responder las preguntas de negocio planteadas.

## Fase 4 — Compuerta paralela (AND) + corrección de deadlock

Actualización: 13 de septiembre, 2026

### Cambio de diseño

Se agregó una bifurcación paralela (Parallel Gateway / AND) para que "Preparar pedido" y "Generar factura" se ejecuten simultáneamente, convergiendo antes de "Despachar pedido" — patrón: AND-split → [Preparar pedido | Generar factura] → AND-join → Despachar pedido.

### Error encontrado y resuelto: deadlock por confundir merge (XOR) con split (AND)

**Síntoma:** la instancia quedaba "viva" indefinidamente en el gateway AND, sin marcar ningún incidente ni error visible en Operate — el Instance History simplemente no avanzaba más allá de ese punto, y el diagrama mostraba el camino recorrido en azul deteniéndose justo al llegar al símbolo "+".

**Causa:** el mismo Parallel Gateway estaba siendo usado a la vez como punto de convergencia de dos caminos *mutuamente excluyentes* provenientes de un XOR anterior (los caminos "sí"/"no" de "¿cuotas mayores a 6?") y como punto de bifurcación hacia las dos tareas paralelas. Un AND-join espera recibir un token por *cada* entrada antes de continuar; como los dos caminos de entrada eran alternativos (nunca llegan ambos en la misma instancia), el segundo token nunca llegaba, causando un bloqueo permanente sin generar ningún error explícito.

**Solución:** se insertó un Exclusive Gateway (XOR) adicional para converger primero los caminos alternativos ("no" y "Revisar riesgo de cuotas"), y desde ese XOR una única flecha alimenta el Parallel Gateway (AND), que ahora actúa exclusivamente como punto de bifurcación (una entrada, dos salidas), sin mezclar semánticas de convergencia y paralelismo en el mismo elemento.

> **Nota pedagógica:** este es un error de diseño BPMN documentado en la literatura (mezclar semántica de gateways de distinto tipo en un mismo nodo) — vale la pena incluirlo explícitamente en el documento técnico como ejemplo de decisión técnica corregida durante la construcción, tal como pide el rubro de "Demostración técnica".

### Prueba de validación

Instancia ejecutada con camino "no" (`requiereRevision = false`) tras la corrección: el proceso avanzó correctamente a través del nuevo XOR de convergencia, el AND split, ambas tareas paralelas, el AND join, y llegó a "Pedido despachado" sin incidentes.

### Próximo pendiente (opcional, según tiempo disponible)

- [ ] Agregar objetos de datos (Pedido, Factura) y almacén de datos (Base de datos de pedidos) como notación visual — sin funcionalidad de ejecución real en Camunda 8, sirve para completitud de la notación BPMN según el estándar visto en el curso (Tema 2.3)
- [ ] Evaluar subproceso colapsado y evento de timer si sobra tiempo antes del 28 de septiembre

### Formulario de la tarea de revisión manual

[`revisarRiesgoCuotas.form`](revisarRiesgoCuotas.form): formulario de la User Task que recibe los pedidos marcados con `requiereRevision = true` (más de 6 cuotas). Muestra el número de cuotas solicitadas y ofrece al revisor tres opciones (`aprobar`, `rechazar`, `mas_info`) más un campo de comentario libre.
