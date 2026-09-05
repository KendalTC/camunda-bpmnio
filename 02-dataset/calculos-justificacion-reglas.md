# Cálculos de justificación — Reglas de negocio (dataset Olist)

Actualización: 4 de septiembre, 2026

> Ver reglas descritas en [`docs/decisiones.md`](../docs/decisiones.md#4-reglas-de-negocio-priorizadas-para-bpmn--dmn).

## Metodología

### Preparación de datos

- Se trabajó con `olist_orders_dataset` y `olist_order_payments_dataset` combinados en un mismo libro de Excel (como pestañas separadas), para poder usar BUSCARV entre ambas tablas.
- En `olist_order_payments_dataset`, se filtró `payment_sequential = 1` para quedarnos con el pago principal de cada pedido (un mismo `order_id` puede tener varios pagos, ej. voucher + tarjeta).
- Configuración regional de Excel detectada: separador de argumentos de función = **coma** `,` (no punto y coma, aunque el Excel esté en español). Nombres de función en español (`SI.ERROR` en vez de `IFERROR`, `BUSCARV` en vez de `VLOOKUP`).

### Fórmulas usadas (columnas agregadas en `olist_orders_dataset`)

**Columna `tiempo_aprobacion_horas`:**

```
=SI(E2="","",(E2-D2)*24)
```

> Nota: la primera versión de esta fórmula (`SI.ERROR((E2-D2)*24,"")`) daba resultados incorrectos (negativos, del orden de -100 años) porque Excel trata una celda de fecha vacía como 0 (año 1900) en vez de generar un error — hubo que reemplazarla por una validación explícita de celda vacía con `SI(E2="",...)`.

**Columna `metodo_pago`:**

```
=SI.ERROR(BUSCARV(A2,olist_order_payments_dataset!A:C,3,FALSO),"sin dato")
```

> Nota: al principio se intentó cruzar los datos estando `orders` y `payments` en archivos de Excel separados, lo cual generaba un diálogo de "buscar archivo externo" en Excel. Se resolvió copiando la hoja de `payments` como pestaña dentro del mismo libro que `orders`.

## Resultado — Regla 1: Tiempo de aprobación por método de pago

Tabla dinámica: `metodo_pago` en Filas, Promedio de `tiempo_aprobacion_horas` en Valores.

| Método de pago | Tiempo promedio de aprobación |
|---|---|
| boleto | 33.12 horas (~1.4 días) |
| credit_card | 4.56 horas |
| debit_card | 9.55 horas |
| voucher | 8.35 horas |
| not_defined | sin datos válidos (división por cero — casi ningún pedido con este método llega a aprobarse) |

**Conclusión:** boleto tarda ~7.3x más que credit_card en aprobarse. Justifica que el proceso BPMN de MaxiMundo bifurque el camino según método de pago: boleto espera confirmación bancaria (evento intermedio de espera); credit_card avanza automáticamente.

## Resultado — Regla 2: Distribución de cuotas (`payment_installments`)

Tabla dinámica: `payment_installments` en Filas, Cuenta de `order_id` en Valores, mostrado como % del total.

| Cuotas | % de pedidos |
|---|---|
| 1 | 50.58% |
| 2–6 | ~37.7% (acumulado) |
| 7+ | ~11.76% (acumulado) |

**Observación adicional:** pico inusual en 10 cuotas (5.13%, más alto que 7, 8 o 9 cuotas) — probablemente asociado a promociones tipo "10x sin intereses", común en e-commerce brasileño.

**Conclusión:** el 11.76% de los pedidos supera las 6 cuotas — segmento no trivial que justifica una regla DMN de revisión de riesgo cuando `payment_installments` > 6.

## Resultado — Regla 3: Cumplimiento de SLA de entrega

Actualización: 4 de septiembre, 2026

### Metodología

- Columna agregada `dias_diferencia_entrega` en `olist_orders_dataset`:

```
=SI(G2="","",(G2-H2))
```

  (G = `order_delivered_customer_date`, H = `order_estimated_delivery_date`)

> Nota técnica: al restar fechas, Excel hereda formato de fecha en el resultado en vez de mostrar el número de días — hubo que cambiar manualmente el formato de la columna a "Número" para poder leer el resultado correctamente.

- Columna `cumple_sla`:

```
=SI(K2="","sin dato",SI(K2<=0,"cumple","no cumple"))
```

  (negativo o cero = entregado a tiempo o antes; positivo = entregado tarde)

### Resultado

| cumple_sla | % de pedidos |
|---|---|
| cumple | 89.15% |
| no cumple | 7.87% |
| sin dato | 2.98% |

### Conclusión

7.87% de los pedidos incumple el SLA de entrega estimado. Sobre una base de ~99,000 pedidos históricos, equivale a más de 7,700 casos — volumen suficiente para justificar un mecanismo de monitoreo/alerta automática dentro del proceso (ej. vía Operate) y, opcionalmente, una tarea de notificación proactiva al cliente cuando se detecte riesgo de atraso.

## Estado

Las 3 reglas de negocio quedan justificadas con datos reales. Listo para iniciar el modelado BPMN.

## Pendiente para la próxima sesión

- [ ] Modelado BPMN real del proceso "Pedido a Entrega" de MaxiMundo usando estas reglas
- [ ] Construcción de tablas DMN en Camunda con estos umbrales (boleto vs. credit_card; >6 cuotas; SLA de entrega)
