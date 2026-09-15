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
- [x] Evaluar subproceso colapsado y evento de timer si sobra tiempo antes del 28 de septiembre — ver Fase 5

### Formulario de la tarea de revisión manual

[`revisarRiesgoCuotas.form`](revisarRiesgoCuotas.form): formulario de la User Task que recibe los pedidos marcados con `requiereRevision = true` (más de 6 cuotas). Muestra el número de cuotas solicitadas y ofrece al revisor tres opciones (`aprobar`, `rechazar`, `mas_info`) más un campo de comentario libre.

## Fase 5 — Subproceso colapsado + Boundary Timer

Actualización: 13 de septiembre, 2026

### Subproceso colapsado "Validar pago"

Se agrupó todo el tramo de validación de pago (Validar método pago, ambas tablas DMN, evento de confirmación bancaria, gateway de cuotas, revisión manual, XOR de convergencia) dentro de un Sub-process (collapsed) llamado "Validar pago". El diagrama principal queda simplificado a:

```
Pedido recibido → [Validar pago] → AND split → (Preparar pedido | Generar factura) → AND join → Despachar pedido → Pedido despachado
```

> **Nota técnica:** en esta versión del Modeler no se encontró una función de "agrupar selección existente en subproceso" vía clic derecho; se realizó creando primero la caja de "Sub-process (collapsed)" vacía y reconstruyendo los elementos en su interior.

### Boundary Timer sobre el subproceso "Validar pago"

Se agregó un evento de límite (boundary event) de tipo Timer, adjunto al borde del subproceso "Validar pago", con duración de prueba de `PT30S` (30 segundos — en un escenario de negocio real este valor sería considerablemente mayor, ej. `PT24H`).

**Comportamiento:** si el subproceso "Validar pago" no se completa dentro del tiempo definido, se interrumpe automáticamente y el flujo salta a una nueva actividad "Escalar seguimiento de pago pendiente", terminando en un End Event alternativo "Pago sin confirmar" — distinto del camino normal hacia "Pedido despachado".

**Justificación de negocio:** modela el caso real de que un pago vía boleto nunca llegue a confirmarse por parte del banco, evitando que el proceso quede esperando indefinidamente sin ninguna acción de seguimiento.

**Prueba de validación:** instancia iniciada con `metodoPago = "boleto"` sin completar la confirmación bancaria manualmente; tras superar el tiempo del timer, la instancia tomó correctamente la ruta de escalamiento hacia "Pago sin confirmar".

### Pendiente descartado por esta sesión: patrón multi-instancia

Se intentó configurar la tarea "Preparar pedido" como actividad multi-instancia (para repetirse una vez por cada vendedor del pedido, aprovechando que un mismo `order_id` de Olist puede involucrar múltiples sellers). No se logró ubicar la opción en la interfaz gráfica de esta versión del Modeler. Se descarta por restricción de tiempo, documentado como mejora identificada pero no implementada. Alternativa no explorada: edición directa del XML del diagrama (bloque `<bpmn:multiInstanceLoopCharacteristics>`).

### Estado general del modelo

Complejidad técnica ampliada y validada: subproceso colapsado, boundary timer con ruta de escalamiento alternativa, gateways XOR/AND correctamente diferenciados, dos tablas DMN conectadas y una tarea de revisión manual con formulario propio.

> ⚠️ **Actualización (Fase 6):** el subproceso colapsado descrito arriba se descartó por preferencia de legibilidad del equipo — ver Fase 6. El boundary timer se conserva, pero ahora cuelga directamente del tramo plano de validación de pago en vez de una caja colapsada.

## Fase 6 — Versión final: gateway del revisor + Consigna 2 (SLA de entrega)

Actualización: 14 de septiembre, 2026

### Decisión de diseño: se descartó el subproceso colapsado

Tras evaluarlo, el equipo decidió eliminar el subproceso colapsado "Validar pago" implementado en la Fase 5 y volver a mostrar todo el flujo de validación de pago de forma plana (sin colapsar), por preferencia de legibilidad del equipo. El resto de la lógica (2 tablas DMN, evento de confirmación bancaria, Boundary Timer sobre validación) se mantiene igual, solo que visible directamente en el diagrama principal en vez de dentro de una caja colapsada.

### Corrección del bug original: la decisión del revisor ahora sí afecta el proceso

Se agregó el gateway **"¿Pedido aprobado?"** inmediatamente después de la convergencia del flujo de validación de pago, con tres salidas:

- **"rechazar"** (`=decisionRevision = "rechazar"`) → tarea **"Notificar rechazo"** → End Event **"Pedido rechazado por riesgo"**
- **"mas_info"** (`=decisionRevision = "mas_info"`) → tarea **"Solicitar información adicional al cliente"** → regresa (bucle) a **"Revisar riesgo de cuotas"**, permitiendo una nueva ronda de revisión
- **"aprobar"** (aprobado explícitamente, o pedidos que nunca requirieron revisión por tener ≤6 cuotas) → tarea **"Aprobar pedido"** → continúa hacia el AND split (Generar Factura / Preparar pedido)

> **Nota importante (bug de despliegue):** el camino "aprobar" originalmente se dejó como flujo por defecto sin condición explícita; en el proceso de ajuste se le agregó por error una condición explícita `=decisionRevision = "aprobar"` con espacios extra al inicio del string, causando un error de despliegue (`failed to parse expression`, FEEL es estricto con que "=" sea el primer carácter sin espacios). Se corrigió reescribiendo la expresión manualmente sin espacios.

### Verificación técnica de los pendientes de la sesión anterior

Revisando el XML de [`Workshop_Final.bpmn`](Workshop_Final.bpmn) (idéntico en contenido a `Prueba1.bpmn`):

1. **Condición de "aprobar":** quedó como **condición explícita** `=decisionRevision = "aprobar"` (`Flow_17sucfb`), **no** como flujo por defecto — el gateway `Gateway_0c46t8c` no tiene ningún flujo marcado `default`. Esto significa que un pedido que **nunca pasó por revisión manual** (≤6 cuotas, `decisionRevision` sin definir) no cumple ninguna de las tres condiciones (`rechazar` / `mas_info` / `aprobar`) al llegar a este gateway, lo que provocaría un error de ejecución ("no matching sequence flow") en vez de avanzar. **Pendiente real de corregir:** o bien marcar el flujo "aprobar" como flujo por defecto, o bien inicializar `decisionRevision = "aprobar"` como valor por defecto para los pedidos que no requieren revisión.
2. **Tipo de "Entrega escalada a logística":** el elemento (`Event_1r1qlc8`) es en realidad un **Intermediate Throw Event** con `escalationEventDefinition`, no un End Event, y **no tiene flujo saliente** — queda como un punto muerto al final de esa rama. Para que sea correcto según el estándar BPMN, debería convertirse en un **End Event de tipo Escalation** (o agregarle un flujo saliente hacia un End Event real).
3. **Detalle menor:** la etiqueta del flujo hacia "Solicitar información adicional al cliente" tiene un typo: `"solitirar info"` (falta la "c").

### Limitación conocida y documentada intencionalmente (Consigna 1 — versión completa del equipo)

El camino "mas_info" hace un bucle real de vuelta a "Revisar riesgo de cuotas" **sin límite de intentos ni contador**. Fue una decisión explícita del equipo (se evaluó agregar un contador con Output Mapping en FEEL — `=intentosInformacion + 1` — pero se descartó por simplicidad). **Riesgo reconocido:** si un revisor selecciona "Solicitar más información" repetidamente, la instancia podría permanecer en bucle indefinido sin ningún mecanismo de corte. Se documenta como limitación conocida del prototipo, no como un error no identificado.

### Consigna 2 (SLA de entrega) — implementada como parte del modelo final

Se agregó una nueva etapa después de "Despachar pedido":

```
Despachar pedido → Esperar confirmación de entrega ──(a tiempo)──> Pedido despachado (fin)
                          │
                          ⏱️ Boundary Timer "Posible atraso en entrega"
                          ▼
                    Notificar posible atraso → End Event "Entrega con atraso reportado"
                          │
                          ⏱️ (segundo Boundary Timer, más corto)
                          ▼
                    Escalar el caso → "Entrega escalada a logística" (ver hallazgo #2 arriba)
```

**Decisión de diseño clave:** inicialmente se consideró poner el Boundary Timer directamente sobre "Despachar pedido", pero se identificó que esa tarea representa solo la acción interna de MaxiMundo (empacar/entregar al transportista), no el tiempo de tránsito real hasta el cliente — que es lo que mide el SLA real del dataset de Olist (`order_estimated_delivery_date` vs. `order_delivered_customer_date`). Por eso se agregó la tarea intermedia "Esperar confirmación de entrega" como representación del tiempo de tránsito, y el timer se colgó sobre esa tarea en su lugar. Esta es una buena decisión técnica a explicar en el documento técnico.

### Archivos de esta fase

- [`Workshop_Final.bpmn`](Workshop_Final.bpmn): versión completa/clave de respuesta (idéntica a `Prueba1.bpmn`), con las Consignas 1 y 2 ya resueltas por el equipo.
- [`Workshop_V1.bpmn`](Workshop_V1.bpmn): versión anterior, sin "Solicitar información adicional al cliente" ni "Escalar el caso" — candidata a servir de plantilla **incompleta** para entregar a los compañeros (ver [`05-workshop/tarea-companeros.md`](../05-workshop/tarea-companeros.md)).

### Pendiente para la próxima sesión

- [ ] Corregir el gateway "¿Pedido aprobado?" para que cubra el caso de pedidos sin `decisionRevision` definido (flujo por defecto o valor inicial) — ver hallazgo #1 arriba
- [ ] Convertir "Entrega escalada a logística" en un End Event de tipo Escalation real (o agregarle flujo saliente) — ver hallazgo #2 arriba
- [ ] Corregir el typo "solitirar info" → "solicitar info"
- [ ] Probar despliegue completo de punta a punta sin errores
- [ ] Decidir qué tramos del modelo se reconstruyen EN VIVO durante el workshop (80 min) vs. cuáles se muestran ya construidos — el modelo actual es demasiado extenso para reconstruirlo completo en el tiempo disponible
- [ ] Confirmar si `Workshop_V1.bpmn` es la base definitiva de la plantilla incompleta para compañeros, o si falta despojarla de más elementos
- [ ] Redactar el texto formal de ambas consignas para el documento de "Tarea para la clase" (ver [`05-workshop/tarea-companeros.md`](../05-workshop/tarea-companeros.md))
