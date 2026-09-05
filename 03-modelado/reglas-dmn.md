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
