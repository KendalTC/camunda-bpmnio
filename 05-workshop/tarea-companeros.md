# Tarea corta para compañeros (1%)

Ambas consignas ya fueron resueltas por el equipo como parte del modelo "versión completa" ([`03-modelado/Workshop_Final.bpmn`](../03-modelado/Workshop_Final.bpmn), documentada en [`03-modelado/reglas-dmn.md`](../03-modelado/reglas-dmn.md#fase-6--versión-final-gateway-del-revisor--consigna-2-sla-de-entrega)). Falta redactar el enunciado formal de cada consigna y preparar la plantilla **incompleta** (sin la solución) que se entregará a los compañeros.

## Consigna 1: desenlace del camino "solicitar más información"

Conectar el camino `mas_info` del gateway "¿Pedido aprobado?" a un desenlace coherente.

- **Solución de referencia del equipo:** bucle de vuelta a "Revisar riesgo de cuotas" (sin límite de intentos — ver limitación conocida documentada en la Fase 6).
- **Pendiente:** decidir si se entrega esa solución como pista parcial, o se deja el camino completamente vacío para que el compañero lo diseñe desde cero.

## Consigna 2: mecanismo de detección y notificación de atraso en la entrega

Diseñar el mecanismo de detección y notificación de atraso en la entrega, justificando el umbral de tiempo con datos reales del dataset.

- **Solución de referencia del equipo:** doble Boundary Timer sobre "Esperar confirmación de entrega" con escalamiento a logística (ver Fase 6).
- **Dato de referencia para justificar el umbral:** 7.87% de los pedidos incumple el SLA estimado (ver [`02-dataset/calculos-justificacion-reglas.md`](../02-dataset/calculos-justificacion-reglas.md#resultado--regla-3-cumplimiento-de-sla-de-entrega)).

## Plantilla para compañeros

- **Base:** [`03-modelado/Workshop_V1.bpmn`](../03-modelado/Workshop_V1.bpmn) — versión anterior a la Fase 6, sin "Solicitar información adicional al cliente" ni "Escalar el caso". Pendiente confirmar si ya está lista tal cual o falta despojarla de más elementos para que ninguna de las dos consignas quede resuelta a medias.

## Pendiente

- [ ] Redactar el texto formal de ambas consignas
- [ ] Decidir el nivel de "pista" que se deja en cada consigna
- [ ] Confirmar/ajustar `Workshop_V1.bpmn` como plantilla incompleta definitiva
- [ ] Preparar presentación breve con la pregunta de negocio y el resultado esperado de cada consigna
