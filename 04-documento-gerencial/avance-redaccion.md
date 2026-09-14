# Documento gerencial — Avance de redacción

Actualización: 13 de septiembre, 2026

> El documento gerencial se redacta directamente en [`DocumentoGerencial.docx`](DocumentoGerencial.docx). Este archivo lleva el registro del avance, los insumos recolectados para las secciones aún no redactadas y el estado de las referencias APA7.

## Estado: secciones completadas hasta "Justificación" (punto 4 de 10)

### Estructura completa del documento

1. Portada
2. Introducción — ✅ redactada (borrador)
3. Contexto — ✅ redactada (borrador)
4. Justificación — ✅ redactada (borrador completo, 3 puntos + tablas)
5. Ventajas y desventajas — ⏳ pendiente
6. Planes y costos — ⏳ pendiente (insumos ya recolectados, ver abajo)
7. Casos de éxito — ⏳ pendiente (insumos ya recolectados, ver abajo)
8. Descripción del proceso de negocio a automatizar — ⏳ pendiente
9. Conclusiones — ⏳ pendiente
10. Referencias (APA7) — ⏳ pendiente, consolidar todas las usadas

---

## 1. Introducción (redactada, borrador)

Camunda es una plataforma de orquestación de procesos de negocio (Business Process Management) que permite modelar, automatizar y monitorear procesos empresariales utilizando el estándar internacional BPMN 2.0 (Business Process Model and Notation) [CITA 1]. A través de su motor de ejecución (Zeebe), Camunda permite que los procesos diseñados visualmente se conviertan en flujos de trabajo reales que se ejecutan, se monitorean y se automatizan dentro de una organización.

Por su parte, bpmn.io es la librería de código abierto sobre la cual está construido el Camunda Desktop Modeler, la herramienta de modelado visual utilizada para diseñar los diagramas BPMN y DMN (Decision Model and Notation) que alimentan el motor de ejecución [CITA 2]. En conjunto, Camunda y bpmn.io ofrecen una solución integral que cubre tanto el diseño como la ejecución y el monitoreo de procesos de negocio, sin requerir herramientas adicionales para cada etapa.

Desde una perspectiva comercial, Camunda se posiciona como una alternativa de automatización de procesos que combina flexibilidad técnica (al ser de código abierto en su edición de desarrollo) con capacidad de escalar a un entorno empresarial mediante sus ediciones de producción y su servicio en la nube (Camunda SaaS) [CITA 3]. Esto la hace atractiva tanto para equipos técnicos que buscan control total sobre su infraestructura, como para organizaciones que prefieren delegar la operación a un proveedor especializado.

En el presente documento se analiza la herramienta Camunda + bpmn.io aplicada a un caso hipotético de implementación en MaxiMundo, una empresa ficticia de comercio electrónico tipo marketplace, con el fin de demostrar su potencial para automatizar procesos reales de negocio.

**Citas pendientes de verificar por el equipo:**
- [CITA 1]: Camunda / BPMN 2.0 — camunda.com o docs.camunda.io
- [CITA 2]: bpmn.io — verificado: Camunda Services GmbH. (2026). *Web-based tooling for BPMN, DMN, CMMN, and Forms*. bpmn.io. https://bpmn.io/
- [CITA 3]: planes/SaaS — verificado: Camunda. (s.f.). *Camunda pricing*. https://camunda.com/pricing/

---

## 2. Contexto (redactado, borrador)

MaxiMundo es una empresa ficticia que opera como un marketplace de comercio electrónico multi-categoría en Latinoamérica, conectando a múltiples vendedores con compradores finales a través de una plataforma en línea, de manera similar a marketplaces reconocidos como Amazon o Temu. Como parte de su operación diaria, MaxiMundo genera un volumen considerable de pedidos que involucran múltiples etapas: recepción del pedido, confirmación del pago, preparación logística y entrega al cliente final.

Para efectos de este análisis, se utiliza como base el histórico de pedidos (log de datos) contenido en el *Brazilian E-Commerce Public Dataset by Olist* [CITA DATASET — pendiente verificar en Kaggle, ver nota abajo], un conjunto de datos públicos que documenta aproximadamente 99,000 pedidos reales de un marketplace de e-commerce, incluyendo información sobre métodos de pago, tiempos de aprobación, entregas y calificaciones de los clientes. Este histórico se utiliza como un caso hipotético equivalente al comportamiento operativo que tendría MaxiMundo, permitiendo fundamentar el análisis en datos reales de la industria en lugar de supuestos arbitrarios.

Actualmente, el proceso de gestión de pedidos de MaxiMundo se ejecuta de manera manual y dispersa entre los equipos de logística, pagos y atención al cliente, sin un mecanismo centralizado que permita monitorear en tiempo real el estado de los pedidos ni automatizar decisiones operativas recurrentes (como la validación de métodos de pago o la identificación de pedidos en riesgo de incumplir los tiempos de entrega). Ante este panorama, la gerencia de MaxiMundo evalúa la implementación de Camunda como plataforma de orquestación de procesos, con el fin de automatizar y dar visibilidad al proceso de "pedido a entrega".

> **Nota:** cita del dataset de Olist pendiente de verificar directamente en Kaggle (el navegador no pudo acceder a kaggle.com en esta sesión). Formato aproximado esperado:
> ```
> Olist, & Sionek, A. (Año). Brazilian E-Commerce Public Dataset by Olist [Conjunto de datos]. Kaggle. https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
> ```
> El equipo debe confirmar año y autoría exacta en la página real antes de finalizar.

---

## 3. Justificación (redactada completa, 3 puntos + tablas)

La decisión de MaxiMundo de evaluar la implementación de Camunda no responde a una tendencia tecnológica genérica, sino a hallazgos concretos identificados a partir del análisis del histórico de pedidos de la empresa. Se identificaron tres situaciones operativas que justifican la necesidad de automatizar el proceso de "pedido a entrega":

**1. Diferencias significativas en el tiempo de confirmación según el método de pago.**

El análisis del histórico de pedidos reveló que los pedidos pagados mediante boleto bancario tardan en promedio 33.1 horas en confirmarse, frente a 4.6 horas en el caso de pagos con tarjeta de crédito — una diferencia de aproximadamente 7.3 veces.

**Tabla 1**
*Tiempo de aprobación por método de pago*

| Método de pago | Tiempo promedio de aprobación (horas) |
|---|---|
| Boleto | 33.12 |
| Credit card | 4.56 |
| Debit card | 9.55 |
| Voucher | 8.35 |

*Nota. Cálculo propio a partir de Olist (s.f.), mediante el cruce de las tablas de pedidos y pagos del histórico de MaxiMundo.*

Actualmente, el proceso de MaxiMundo no distingue de forma sistemática entre estos dos escenarios, lo que genera inconsistencia en los tiempos de respuesta hacia el cliente. Automatizar esta distinción mediante reglas de decisión permitiría que el proceso reaccione de forma diferenciada según el método de pago, sin intervención manual.

**2. Exposición financiera no monitoreada en pedidos con financiamiento extendido.**

El análisis de la distribución de cuotas de pago en el histórico de pedidos de MaxiMundo (Tabla 2) muestra que, si bien el 50.58% de los pedidos se paga de contado (una sola cuota), un 11.76% se financia en más de 6 cuotas — un segmento no trivial que representa mayor riesgo financiero para la empresa, al implicar un compromiso de pago extendido en el tiempo.

**Tabla 2**
*Distribución de pedidos por número de cuotas*

| Cuotas | % de pedidos |
|---|---|
| 1 | 50.58% |
| 2 a 6 | 37.66% |
| 7 o más | 11.76% |

*Nota. Cálculo propio a partir de Olist (s.f.). Valores agrupados a partir del detalle completo por número de cuotas disponible en el Anexo [X — pendiente crear].*

Actualmente, MaxiMundo no cuenta con un mecanismo diferenciado que identifique estos pedidos de mayor exposición financiera antes de aprobarlos, tratándolos de la misma forma que los pedidos de contado o de pocas cuotas. La automatización de una regla de negocio que identifique cuándo un pedido supera el umbral de 6 cuotas permitiría dirigir únicamente esos casos hacia una revisión de riesgo adicional, sin ralentizar el 88.24% restante de los pedidos que no lo requiere — optimizando así el uso del tiempo del equipo de revisión y manteniendo la agilidad operativa para la mayoría de las transacciones.

**3. Incumplimiento no gestionado del tiempo estimado de entrega.**

El análisis del cumplimiento del plazo de entrega estimado en el histórico de pedidos de MaxiMundo (Tabla 3) muestra que, si bien el 89.15% de los pedidos se entrega dentro del plazo estimado o antes, un 7.87% incumple dicho plazo — es decir, llega tarde al cliente. Un 2.98% adicional corresponde a pedidos sin dato de entrega registrado, generalmente asociados a cancelaciones u otros estados incompletos del proceso.

**Tabla 3**
*Cumplimiento del plazo estimado de entrega*

| Estado de entrega | % de pedidos |
|---|---|
| Cumple (a tiempo o antes) | 89.15% |
| No cumple (tardío) | 7.87% |
| Sin dato | 2.98% |

*Nota. Cálculo propio a partir de Olist (s.f.).*

Aunque el 7.87% de incumplimiento representa una minoría dentro del total de pedidos, al proyectarlo sobre el volumen operativo histórico de aproximadamente 99,000 pedidos, equivale a más de 7,700 casos de incumplimiento — un volumen considerable para una operación de este tamaño. Actualmente, MaxiMundo no cuenta con un mecanismo que le permita identificar estos casos en tiempo real ni anticiparse a ellos; el incumplimiento generalmente se detecta únicamente cuando el cliente se comunica para reportarlo, lo que implica una gestión reactiva del problema. La implementación de un mecanismo de monitoreo activo, como el que ofrece Camunda a través de Operate, permitiría a MaxiMundo visualizar en tiempo real qué pedidos están en riesgo de incumplir el plazo estimado, habilitando una gestión proactiva — por ejemplo, notificando al cliente con anticipación o escalando el caso al equipo de logística antes de que el retraso ocurra.

**Párrafo de cierre de la Justificación:**

Estas tres situaciones tienen en común que actualmente dependen de la revisión manual de los equipos de logística, pagos y atención al cliente, lo que introduce inconsistencia, retrasos y falta de trazabilidad. Camunda, mediante su motor de orquestación de procesos (Zeebe), su capacidad de automatizar reglas de decisión (DMN) y su módulo de monitoreo en tiempo real (Operate), permite atender estas tres situaciones dentro de un mismo proceso automatizado, ofreciendo a MaxiMundo trazabilidad, consistencia y visibilidad operativa que actualmente no posee.

---

## Insumos ya recolectados para secciones pendientes (no redactados aún como prosa)

### Para "Planes y costos" (verificado en camunda.com/pricing)

- Self-Managed Development: gratis, sin licencia
- Self-Managed Production: requiere licencia Enterprise (precio no público, contactar ventas)
- Camunda SaaS: prueba gratis 30 días, luego contactar ventas

### Para "Casos de éxito" (verificado en camunda.com/customers)

- Vodafone Alemania (testimonio: Michael Voeller, Head of Project and Demand Management)
- athenahealth (testimonio: Vamsi Krishna, Director of Engineering)
- Sugerencia: filtrar por industria "Retail/Consumer" + región "Latin America" para un caso más cercano al contexto de MaxiMundo

### Referencias APA7 ya verificadas

```
Camunda. (s.f.). Camunda pricing. https://camunda.com/pricing/
Camunda. (s.f.). Our customers. https://camunda.com/about/customers/
Camunda Services GmbH. (2026). Web-based tooling for BPMN, DMN, CMMN, and Forms. bpmn.io. https://bpmn.io/
```

### Referencia pendiente de verificar por el equipo

```
Olist, & Sionek, A. (Año — verificar). Brazilian E-Commerce Public Dataset by Olist [Conjunto de datos]. Kaggle. https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
```

## Próximo paso al retomar

Continuar redacción con "Ventajas y desventajas", luego "Planes y costos" y "Casos de éxito" (usando los insumos ya recolectados arriba), y finalmente "Descripción del proceso de negocio", "Conclusiones" y consolidación de "Referencias".
