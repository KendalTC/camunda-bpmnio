# Decisiones del proyecto — MaxiMundo (Camunda 8 + bpmn.io)

Última actualización: 13 de septiembre, 2026 (tarde)

## 1. Stack técnico

- **Motor/plataforma:** Camunda 8 Run v8.9.17 (Zeebe), instalado y verificado el 29/08/2026. Ver [manual de instalación](../01-instalacion/manual-instalacion.md).
- **Modelador (bpmn.io):** Camunda Desktop Modeler v5.50.1, columna "Camunda 8".
- **Monitoreo:** Operate, incluido en Camunda 8 Run (`http://localhost:8080/operate`).
- **Automatización de decisiones (DMN):** tablas de decisión de Camunda, integradas al proceso BPMN en los puntos descritos en la sección 4.

## 2. Empresa ficticia

- **Nombre:** MaxiMundo
- **Tipo de negocio:** Marketplace multi-categoría (estilo Amazon/Temu), múltiples vendedores, operación en LatAm.
- **Contexto/problema actual:** el seguimiento de pedidos está disperso entre logística, pagos y atención al cliente, sin automatización ni visibilidad centralizada. La empresa busca implementar Camunda para automatizar el proceso de "Pedido a Entrega" y ganar visibilidad en tiempo real (vía Operate).

## 3. Dataset seleccionado

- **Fuente:** Brazilian E-Commerce Public Dataset by Olist (Kaggle)
- **Tablas usadas:** `olist_orders_dataset`, `olist_order_payments_dataset`, `olist_order_items_dataset`, `olist_order_reviews_dataset`, `olist_products_dataset`, `olist_customers_dataset`, `olist_sellers_dataset`, `product_category_name_translation`
- **Decisión sobre categorías de producto:** se usará el dataset completo (todas las categorías), no un nicho específico — coherente con el modelo de negocio "vende de todo" de MaxiMundo.
- **Nota pendiente:** el dataset viene en portugués (nombres de columnas ya identificados en inglés gracias a los nombres de campo, pero valores de texto como `order_status` y ciudades pueden estar en portugués) — pendiente armar diccionario de datos completo (ver [`02-dataset/diccionario-datos.md`](../02-dataset/diccionario-datos.md)).

## 4. Reglas de negocio priorizadas (para BPMN + DMN)

### Regla 1 (protagonista): Confirmación de pago según método

- **Columnas:** `payment_type`, `order_purchase_timestamp`, `order_approved_at`
- **Lógica:** `boleto` → esperar confirmación antes de continuar (proceso más lento, depende del banco); `credit_card` → avanzar automáticamente.
- **Justificación con datos:** ✅ calculada — boleto tarda ~33.12 h en promedio vs. ~4.56 h de credit_card (~7.3x más lento). Ver [`02-dataset/calculos-justificacion-reglas.md`](../02-dataset/calculos-justificacion-reglas.md#resultado--regla-1-tiempo-de-aprobación-por-método-de-pago).
- **Implementación BPMN:** compuerta exclusiva (XOR gateway) como punto central del proceso. ✅ conectada a la tabla DMN [`determinarConfirmacionPago.dmn`](../03-modelado/determinarConfirmacionPago.dmn) — ver [`03-modelado/reglas-dmn.md`](../03-modelado/reglas-dmn.md).

### Regla 2 (anidada dentro de la Regla 1): Riesgo por número de cuotas

- **Columna:** `payment_installments`
- **Lógica:** más de 6 cuotas → marcar para revisión de riesgo; 6 o menos → aprobación automática.
- **Justificación con datos:** ✅ calculada — 11.76% de los pedidos supera las 6 cuotas (segmento no trivial), con un pico inusual en 10 cuotas. Ver [`02-dataset/calculos-justificacion-reglas.md`](../02-dataset/calculos-justificacion-reglas.md#resultado--regla-2-distribución-de-cuotas-payment_installments).
- **Implementación BPMN:** ✅ conectada a la tabla DMN [`determinarRiesgoCuotas.dmn`](../03-modelado/determinarRiesgoCuotas.dmn) — ver [`03-modelado/reglas-dmn.md`](../03-modelado/reglas-dmn.md).

### Regla 3 (métrica, no bifurcación): SLA de entrega

- **Columnas:** `order_delivered_customer_date` vs. `order_estimated_delivery_date`
- **Uso:** no se modela como gateway del proceso principal, sino como métrica de monitoreo mostrada en Operate/dashboard (% de pedidos que incumplen el SLA). Opcional: disparar tarea de "notificar al cliente" si se detecta riesgo de atraso.
- **Justificación con datos:** ✅ calculada — 7.87% de los pedidos incumple el SLA (~7,700 de ~99,000 pedidos históricos), volumen suficiente para justificar monitoreo/alerta automática. Ver [`02-dataset/calculos-justificacion-reglas.md`](../02-dataset/calculos-justificacion-reglas.md#resultado--regla-3-cumplimiento-de-sla-de-entrega).

### Regla 4 (proceso secundario): Escalamiento por satisfacción

- **Columna:** `review_score` (opcionalmente cruzado con si hubo retraso en la entrega)
- **Lógica:** `review_score` ≤ 2 → crear tarea de "contacto prioritario de servicio al cliente".
- **Nota:** se modela como un proceso relacionado/secundario, no como parte del flujo principal "Pedido a Entrega", porque la reseña llega después de completado el proceso principal.

## 5. Enfoque de ejecución del workshop ("vivo vs. preconstruido")

Enfoque híbrido, basado en los requisitos del enunciado ("se ejecuta paso a paso en la clase de manera guiada" + "explicar las decisiones técnicas tomadas durante la construcción"):

- **Preparado de antemano (no se construye en vivo):** BPMN completo y reglas DMN ya construidas, probadas y funcionando — sirve como respaldo si algo falla en vivo.
- **Construido/reconstruido en vivo durante el workshop:** los puntos de valor pedagógico — agregar la compuerta de decisión de pago, conectar la tabla DMN, desplegar, iniciar instancia, ver resultado en Operate — explicando el "por qué" de cada decisión técnica mientras se construye.
- **Los compañeros replican en sus propias máquinas** en tiempo real (por eso el video de instalación debe estar disponible 3 días antes).
- **Al final:** cada compañero entrega su propia versión de manera individual vía Mediación Virtual.
- **Pendiente:** armar el guion cronometrado del workshop (qué se construye en qué orden, con qué timing, para caber en los 80 minutos asignados) — ver [`05-workshop/guia-taller.md`](../05-workshop/guia-taller.md).

## 6. Pendientes generales

> Recordatorio, no exhaustivo — ver checklist maestro completo en [`Guia_Camunda_bpmnio_Checklist.md`](../Guia_Camunda_bpmnio_Checklist.md).

- [ ] Guion + grabación del video de instalación (debe enviarse 3 días antes de la exposición: 25 de septiembre)
- [ ] Diccionario de datos traducido (dataset en portugués)
- [x] Cálculos reales de las métricas que justifican las Reglas 1, 2 y 3 (ver [`02-dataset/calculos-justificacion-reglas.md`](../02-dataset/calculos-justificacion-reglas.md))
- [x] Modelado BPMN inicial del proceso "Pedido a Entrega" de MaxiMundo (`Prueba1.bpmn`, con pool secundario de Atención al Cliente)
- [x] Construcción y conexión de las tablas DMN de las Reglas 1 y 2 (ver [`03-modelado/reglas-dmn.md`](../03-modelado/reglas-dmn.md))
- [x] Bifurcación paralela (AND) para "Preparar pedido" + "Generar factura", con corrección de un deadlock por mezclar semántica de gateway XOR/AND (ver [`03-modelado/reglas-dmn.md`](../03-modelado/reglas-dmn.md#fase-4--compuerta-paralela-and--corrección-de-deadlock))
- [x] Subproceso colapsado "Validar pago" + Boundary Timer con ruta de escalamiento ("Pago sin confirmar") para pedidos boleto que nunca confirman el pago (ver [`03-modelado/reglas-dmn.md`](../03-modelado/reglas-dmn.md#fase-5--subproceso-colapsado--boundary-timer))
- [ ] Correr instancias con datos reales del dataset de Olist (no solo pruebas técnicas) para responder las preguntas de negocio
- [ ] (Opcional, descartado por tiempo) Objetos de datos/almacén de datos visuales; patrón multi-instancia en "Preparar pedido" (no se encontró la opción en el Modeler; alternativa no explorada: editar el XML directamente)
- [ ] Guion cronometrado del workshop (80 min)
- [ ] Documento gerencial: redactado hasta "Justificación" (4 de 10 secciones), faltan Ventajas y desventajas, Planes y costos, Casos de éxito, Descripción del proceso, Conclusiones y Referencias — ver [`04-documento-gerencial/avance-redaccion.md`](../04-documento-gerencial/avance-redaccion.md)
- [ ] Documento técnico, guía del taller, tarea corta para compañeros, presentación final
