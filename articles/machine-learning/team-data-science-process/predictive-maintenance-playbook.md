---
title: 'Guía de Azure AI para soluciones de mantenimiento predictivo: proceso de ciencia de los datos en equipos'
description: Una descripción completa de la ciencia de datos que ofrece soluciones de mantenimiento predictivo en varios sectores verticales.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: previous-author=fboylu, previous-ms.author=fboylu
ms.openlocfilehash: 5a8b722b7bef393422eec395f16de0f38e806152
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108141764"
---
# <a name="azure-ai-guide-for-predictive-maintenance-solutions"></a>Guía de Azure AI para soluciones de mantenimiento predictivo

## <a name="summary"></a>Resumen

El mantenimiento predictivo (**PdM**) es una aplicación popular de análisis predictivo que puede ayudar a que empresas de varios sectores logren una alta utilización de los recursos y ahorros en los costos operativos. En esta guía se combinan las directrices analíticas y empresariales y los procedimientos recomendados para desarrollar e implementar las soluciones de mantenimiento predictivo con la tecnología de la [plataforma de Microsoft Azure AI](https://azure.microsoft.com/overview/ai-platform).

Para empezar, en esta guía se presentan escenarios empresariales específicos del sector y el proceso de calificar estos escenarios de mantenimiento predictivo. También se proporcionan los requisitos de datos y las técnicas de modelado para compilar soluciones de mantenimiento predictivo. El contenido principal de la guía trata del proceso de ciencia de datos, incluidos los pasos de preparación de datos, la ingeniería de características, la creación de modelos y la operacionalización del modelo. Para complementar estos conceptos claves, en esta guía se muestra un conjunto de plantillas de soluciones para ayudar a acelerar el desarrollo de aplicaciones de mantenimiento predictivo. La guía también apunta a los recursos de aprendizaje útiles para el profesional obtenga más información sobre la inteligencia artificial que está detrás de la ciencia de datos. 

### <a name="data-science-guide-overview-and-target-audience"></a>Información general sobre la guía de ciencia de datos y público objetivo
La primera mitad de esta guía describe los problemas empresariales habituales, las ventajas de la implementación del mantenimiento predictivo para solucionar estos problemas y enumera algunos casos de uso comunes. Los responsables de la toma de decisiones empresariales (BDM) se beneficiarán de este contenido. La segunda mitad explica la ciencia de datos detrás del mantenimiento predictivo y proporciona una lista de las soluciones de PdM compiladas con los principios que se describen en esta guía. También proporciona rutas de aprendizaje y punteros para el material de aprendizaje. Los responsables de la toma de decisiones técnicas (TDM) considerarán que este contenido es útil.

| Comience con... | Si es... |
|:---------------|:---------------|
| [Caso empresarial para mantenimiento predictivo](#business-case-for-predictive-maintenance) |responsable de la toma de decisiones empresariales (BDM) que desea disminuir los costos operativos y el tiempo de inactividad, además de mejorar el uso de los equipos |
| [Ciencia de datos para mantenimiento predictivo](#data-science-for-predictive-maintenance) |responsable de la toma de decisiones técnicas (TDM) que evalúa las tecnologías de PdM para entender los requisitos únicos de procesamiento de datos e inteligencia artificial para el mantenimiento predictivo |
| [Plantillas de soluciones para mantenimiento predictivo](#solution-templates-for-predictive-maintenance)|arquitecto de software o desarrollador de inteligencia artificial que busca crear rápidamente una demostración o prueba de concepto |
| [Recursos de aprendizaje para mantenimiento predictivo](#training-resources-for-predictive-maintenance) | cualquiera o todas las opciones anteriores y desea conocer los conceptos fundamentales detrás de la ciencia de datos, herramientas y técnicas

### <a name="prerequisite-knowledge"></a>Conocimiento de los requisitos previos
El contenido para BDM no espera que el lector tenga ningún conocimiento de ciencia de datos previo. En el caso del contenido para TDM, resulta útil contar con un conocimiento básico de estadística y ciencia de datos. Se recomienda tener conocimientos de los servicios de inteligencia artificial y datos de Azure, Python, R, XML y JSON. Las técnicas de inteligencia artificial se implementan en paquetes de Python y R. Las plantillas de soluciones se implementan mediante servicios de Azure, herramientas de desarrollo y SDK.

## <a name="business-case-for-predictive-maintenance"></a>Caso empresarial para mantenimiento predictivo

Las empresas requieren que el equipo crítico se ejecute al máximo nivel de eficacia y utilización para capitalizar la rentabilidad de sus inversiones de capital. Estos recursos podrían abarcar desde motores de aviones, turbinas, ascensores o enfriadores industriales (cuyo costo es millonario) hasta dispositivos cotidianos como fotocopiadoras, máquinas de café o refrigeradores de agua.
- De manera predeterminada, la mayoría de las empresas confían en el _mantenimiento correctivo_, en el que las piezas se reemplazan una vez que presentan errores. El mantenimiento correctivo garantiza que las piezas se usen por completo (por lo tanto, no se desperdicia la vida útil del componente), pero para el negocio implica costos en tiempo de inactividad, trabajo y requisitos de mantenimiento no programado (horas de inactividad o ubicaciones poco prácticas).
- En el nivel siguiente, las empresas practican el _mantenimiento preventivo_, en el que determinan el tiempo de vida útil de alguna pieza y realizan su mantenimiento o reemplazo antes de que ocurra algún error. El mantenimiento preventivo evita errores catastróficos y no programados. Pero los altos costos del tiempo de inactividad programado, la infrautilización de los componentes durante la vigencia de uso y el trabajo todavía se mantienen.
- El objetivo del _mantenimiento predictivo_ es optimizar el equilibrio entre el mantenimiento correctivo y el preventivo, mediante la habilitación del reemplazo _justo a tiempo_ de los distintos componentes. En este enfoque, los componentes solo se reemplazan cuando estén próximos a presentar errores. Al extender la vida útil de los componentes (en comparación con el mantenimiento preventivo) y reducir los costos laborales y el mantenimiento no programado (en comparación con el mantenimiento correctivo), las empresas pueden obtener un mayor ahorro y mayores ventajas competitivas.

## <a name="business-problems-in-pdm"></a>Problemas empresariales en el mantenimiento predictivo
Las empresas enfrentan un alto riesgo operativo debido a errores inesperados y no tienen mucha información sobre la causa principal de los problemas en sistemas complejos. Algunas de las cuestiones de negocio clave son:

- Detectar anomalías en el rendimiento o la funcionalidad del sistema o los equipos.
- Predecir si un recurso puede presentar un error en un futuro próximo.
- Calcular la vida útil restante de un recurso.
- Identificar las principales causas del error de un recurso.
- Identificar qué acciones de mantenimiento se deben llevar a cabo en un recurso y en qué plazo.

Las declaraciones de propósito típicas del mantenimiento predictivo son:

- Reducir el riesgo operativo del equipo crítico.
- Aumentar la tasa de rentabilidad en los recursos mediante la previsión de los errores antes de que se produzcan.
- Controlar el costo de mantenimiento mediante la habilitación de operaciones de mantenimiento Just-In-Time.
- Disminuir el desgaste de los clientes, mejorar la imagen de la marca y la pérdida de ventas.
- Disminuir los costos de inventario mediante la reducción de los niveles de este último gracias a la previsión del punto en el que se debe hacer un nuevo pedido.
- Detectar patrones asociados a diversos problemas de mantenimiento.
- Proporcionar KPI (indicadores clave de rendimiento) como puntuaciones del estado para las condiciones de los recursos.
- Calcular la vida útil que le queda a los recursos.
- Recomendar actividades de mantenimiento oportunas.
- Habilitar el inventario Just-In-Time mediante el cálculo de las fechas de pedido para el reemplazo de las piezas.

Estas declaraciones de propósito son los puntos de partida para que:

- _los científicos de datos_ analicen y resuelvan problemas predictivos específicos.
- _los desarrolladores y arquitectos de la nube_ creen una solución completa.

## <a name="qualifying-problems-for-predictive-maintenance"></a>Calificación de los problemas para el mantenimiento predictivo
Es importante destacar que el mantenimiento predictivo no puede resolver eficazmente todos los casos de uso o problemas empresariales. Existen tres criterios de calificación importantes que deben tenerse en cuenta durante la selección del problema:

- La naturaleza del problema debe ser predictiva; es decir, debe haber un destino o un resultado para predecir. El problema también debe tener una ruta de acción clara para evitar errores cuando se detecten.
- El problema debe tener un registro del historial de operaciones del equipo que contenga tanto _resultados buenos como malos_. El conjunto de acciones llevadas a cabo para mitigar los resultados incorrectos también debe estar disponible como parte de estos registros. También son importantes los informes de errores, registros de mantenimiento de la degradación del rendimiento, registros de reemplazos y reparación. Además, también son útiles las reparaciones que se realizan para mejorarlos y los registros de reemplazos.
- El historial registrado debe reflejarse en los datos _relevantes_ que sean de una calidad _suficiente_ para respaldar el caso de uso. Para más información sobre de la suficiencia y la relevancia de los datos, consulte el artículo sobre los [requisitos de datos para el mantenimiento predictivo](#data-requirements-for-predictive-maintenance).
- Por último, la empresa debe tener expertos del dominio que tengan una idea clara del problema. Deben ser conscientes de las prácticas y los procesos internos que puedan ayudar a los analistas a comprender e interpretar los datos. También deben poder realizar los cambios necesarios en los procesos de negocio existentes como ayuda para recopilar los datos correctos relacionados con los problemas, si es necesario.

## <a name="sample-pdm-use-cases"></a>Casos de uso de mantenimiento predictivo de ejemplo
Esta sección se centra en una colección de casos de uso de mantenimiento predictivo de varios sectores, como el aeroespacial, el de servicios públicos y el de transporte. Cada sección comienza con un problema empresarial y describe las ventajas del mantenimiento predictivo, los datos relevantes que rodean al problema empresarial y, por último, las ventajas de una solución de mantenimiento predictivo.

| Problema empresarial | Ventajas del mantenimiento predictivo |
|:-----------------|-------------------|
|**Sector aeronáutico**      |                   |
|_Retraso y cancelaciones de vuelos_ debido a problemas mecánicos. Los errores que no se pueden reparar a tiempo pueden provocar cancelaciones de vuelos e interrumpir los horarios y las operaciones. |Las soluciones de mantenimiento predictivo pueden predecir la probabilidad de que un avión se retrase o que se cancele un vuelvo debido a errores mecánicos.|
|_Errores de las piezas del motor de un avión_: los reemplazos de piezas del motor de un avión se encuentran entre las tareas de mantenimiento más comunes del sector aeronáutico. Las soluciones de mantenimiento requieren una cuidadosa administración de la disponibilidad de existencias de los componentes, su entrega y la planificación.|El mero hecho de poder recopilar información sobre la confiabilidad de los componentes genera una reducción considerable en los costos de inversión.|
|**Sector financiero** |                         |
|Los _errores de un cajero automático_ son un problema común del sector bancario. Aquí, el problema es informar la probabilidad de que una transacción de retiro de efectivo de un cajero automático se interrumpe debido a un error en las piezas o un atasco de papel en el dispensador de efectivo. En función de las predicciones de los errores en las transacciones, se puede realizar un mantenimiento proactivo de los cajeros automáticos para evitar errores.| En lugar de permitir que la máquina presente errores en medio de una transacción, la alternativa deseable es programar la máquina para denegar el servicio en función de la predicción.|
|**Sector energético** |                          |
|_Error en turbinas eólicas_: las turbinas eólicas son la principal fuente energética en países o regiones respetuosos con el medio ambiente y significan altos costos de capital. Un componente clave de las turbinas eólicas es el motor del generador, cuyo error hace que la turbina resulte ineficaz. También es muy costoso de arreglar.|Predecir KPI como el MTTF (tiempo medio entre errores) puede ayudar a las empresas generadoras de energía eviten los errores de las turbinas y garanticen que el tiempo de inactividad sea mínimo. Las probabilidades de error informarán a los técnicos que supervisen las turbinas que probablemente pronto presenten errores y programen regímenes de mantenimiento en función del tiempo. Los modelos predictivos proporcionan conclusiones sobre los distintos factores que contribuyen a los errores, lo que ayuda a los técnicos a comprender mejor que las causas principales de los problemas.|
|_Errores de disyuntores_: la distribución de la electricidad a hogares y empresas requiere que las redes eléctricas siempre funcionen para garantizar la entrega de la energía. Los disyuntores ayudar a limitar o evitar daños en las redes eléctricas durante alguna sobrecarga o condiciones meteorológicas adversas. El problema empresarial aquí es predecir los errores de los disyuntores.| Las soluciones de mantenimiento predictivo ayudan a reducir los costos de reparación y aumentan el ciclo de vida del equipo, como los disyuntores. Ayudan a mejorar la calidad de la red energética mediante la reducción de interrupciones del servicio y errores inesperados.|
|**Transporte y logística** |    |
|_Errores en puertas de ascensores_: las grandes empresas de ascensores proporcionan un servicio completo para millones de ascensores en funcionamiento en todo el mundo. Las principales preocupaciones para esos clientes son la seguridad, la confiabilidad y el tiempo de actividad de los ascensores. Estas empresas realizan el seguimiento de estos y otros atributos a través de sensores, los que ayudarán a realizar el mantenimiento correctivo y preventivo. En un ascensor, el principal problema para los clientes es el funcionamiento incorrecto de las puertas de los ascensores. El problema empresarial de este caso es proporcionar una aplicación predictiva de la base de conocimientos que prediga las posibles causas de los errores en las puertas.| Los ascensores son inversiones de capital posiblemente para un período de 20 a 30 años. Por tanto, cada venta potencial puede ser muy competitiva, por lo que las expectativas de servicio y soporte técnico son altas. El mantenimiento predictivo puede brindar a estas empresas una ventaja con respecto a la competencia en cuanto a sus ofertas de productos y servicios.|
|_Errores de las ruedas_: los errores de las ruedas representan la mitad de todos los descarrilamientos de trenes y cuestan miles de millones al sector ferroviario mundial. Los errores de las ruedas también generar deterioro en los rieles, por lo que a veces se rompen antes de tiempo. Las roturas de los rieles generan eventos catastróficos, como descarrilamientos. Para evitar esas instancias, los ferrocarriles supervisan el rendimiento de las ruedas y las reemplazan de manera preventiva. El problema comercial aquí es predecir los errores de las ruedas.| El mantenimiento predictivo de las ruedas ayudará a reemplazarlas justo a tiempo. |
|_Errores en las puertas del metro_: un motivo importante de los retrasos en las operaciones del metro son los errores de las puertas de los carros. El problema empresarial aquí es predecir los errores de las puertas de los carros.|Conocer por anticipado el error en una puerta, o bien el número de días que faltan para que se produzca, ayudará a la empresa a optimizar las programaciones de mantenimiento de las puertas de los carros.|

La sección siguiente se adentra en los detalles de cómo materializar las ventajas del mantenimiento predictivo descritas anteriormente.

## <a name="data-science-for-predictive-maintenance"></a>Ciencia de datos para mantenimiento predictivo

Esta sección proporciona directrices generales de los principios de ciencia de datos y la práctica del mantenimiento predictivo. Está diseñada para ayudar a un TDM, un arquitecto de soluciones o un desarrollador a entender los requisitos previos y el proceso para compilar aplicaciones integrales de inteligencia artificial para el mantenimiento predictivo. Puede leer esta sección junto con una revisión de las demostraciones y las plantillas de prueba de concepto que aparecen en [Plantillas de soluciones para mantenimiento predictivo](#solution-templates-for-predictive-maintenance). Luego puede usar estos principios y procedimientos recomendados para implementar la solución de mantenimiento predictivo en Azure.

> [!NOTE]
> Esta guía no está diseñada para enseñar ciencia de datos al lector. Se proporcionan varios orígenes útiles para obtener más información en la sección [Recursos de aprendizaje para mantenimiento predictivo](#training-resources-for-predictive-maintenance). Las [plantillas de soluciones](#solution-templates-for-predictive-maintenance) que aparecen en la guía muestran algunas de estas técnicas de inteligencia artificial para problemas de mantenimiento predictivo específicos.

## <a name="data-requirements-for-predictive-maintenance"></a>Requisitos de datos para mantenimiento predictivo

El éxito de todo aprendizaje depende de (a) la calidad de lo que se enseña y (b) la capacidad del aprendiz. Los modelos predictivos aprenden patrones de los datos históricos y predicen resultados futuros con cierta probabilidad en función de estos patrones observados. La precisión de la predicción de un modelo depende de la relevancia, suficiencia y calidad de los datos de aprendizaje y prueba. Los nuevos datos que se "puntúan" mediante este modelo deben tener las mismas características y el mismo esquema que los datos de aprendizaje y prueba. Las características de la función (tipo, densidad, distribución, etc.) de los datos nuevos deben coincidir con las de los conjuntos de datos de aprendizaje y prueba. El objetivo de esta sección está en esos requisitos de datos.

### <a name="relevant-data"></a>Datos relevantes

En primer lugar, los datos deben ser _relevantes para el problema_. Considere el caso de uso de los _errores de las ruedas_ que ya se analizó: los datos de aprendizaje deben contener características relacionadas con el funcionamiento de las ruedas. Si el problema era predecir el error del _sistema de tracción_, los datos de aprendizaje deben abarcar todos los distintos componentes del sistema de tracción. El primer caso apunta a un componente específico, mientras que el segundo caso apunta al error de un subsistema más grande. La recomendación general es diseñar sistemas de predicción sobre componentes específicos en lugar de subsistemas más grandes, ya que este último tendrá datos más dispersos. El experto del dominio (consulte [Calificación de los problemas para el mantenimiento predictivo](#qualifying-problems-for-predictive-maintenance)) ayudará a seleccionar los subconjuntos de datos para análisis más relevantes. Los orígenes de datos relevantes se describen con más detalle en [Preparación de los datos para mantenimiento predictivo](#data-preparation-for-predictive-maintenance).

### <a name="sufficient-data"></a>Datos suficientes
Se suelen formular dos preguntas con respecto a los datos del historial de errores: (1) "¿Cuántos eventos de errores son necesarios para entrenar a un modelo?" (2) "¿Cuántos registros se consideran "suficientes"?" No hay ninguna respuesta definitiva, sino solo reglas generales. Para la pregunta (1), cuanto más eventos de error hay, mejor será el modelo. Para la pregunta (2), el número exacto de eventos de error depende de los datos y el contexto del problema que se está solucionando. Pero, por otro lado, si una máquina presenta errores con demasiada frecuencia, la empresa la reemplazará, lo que disminuirá las instancias de error. Una vez más, la orientación del experto del dominio es importante. Pero existen métodos para el problema de los _eventos inusuales_. Se describen en la sección [Control de datos desequilibrados](#handling-imbalanced-data).

### <a name="quality-data"></a>Datos de calidad
La calidad de los datos es fundamental, por lo que cada valor de atributo de predicción debe ser _preciso_ junto con el valor de la variable de destino. La calidad de los datos es un área bastante estudiada en la administración de estadísticas datos y, por tanto, fuera queda fuera del ámbito de esta guía.

> [!NOTE]
> Hay varios recursos y productos empresariales para brindar datos de calidad. A continuación, un ejemplo de las referencias:
> - Dasu, T, Johnson, T., Exploratory Data Mining and Data Cleaning, Wiley, 2003.
> - [Exploratory Data Analysis, Wikipedia](https://en.wikipedia.org/wiki/Exploratory_data_analysis)
> - [Hellerstein, J, Quantitative Data Cleaning for Large Databases](http://db.cs.berkeley.edu/jmh/papers/cleaning-unece.pdf)
> - [de Jonge, E, van der loo, M, Introduction to Data Cleaning with R](https://cran.r-project.org/doc/contrib/de_Jonge+van_der_Loo-Introduction_to_data_cleaning_with_R.pdf)

## <a name="data-preparation-for-predictive-maintenance"></a>Preparación de los datos para mantenimiento predictivo

### <a name="data-sources"></a>Orígenes de datos

Los orígenes de datos relevantes para el mantenimiento predictivo incluyen, pero no se limitan a:
- Historial de errores
- Historial de mantenimiento o reparaciones
- Condiciones de funcionamiento de la máquina
- Metadatos de equipo

#### <a name="failure-history"></a>Historial de errores
Los eventos de error son inusuales en las aplicaciones de mantenimiento predictivo. Pero cuando se compilan modelos de predicción, el algoritmo debe conocer el patrón de funcionamiento normal de un componente, así como sus patrones de error. Por tanto, los datos de aprendizaje deben incluir un número suficiente de ejemplos de ambas categorías. Los registros de mantenimiento y el historial de reemplazo de piezas son buenas fuentes para buscar eventos de error. Con ayuda de algunos conocimientos de dominio, las anomalías de los datos de aprendizaje también se pueden definir como errores.

#### <a name="maintenancerepair-history"></a>Historial de mantenimiento o reparaciones
El historial de mantenimiento de un recurso incluye detalles sobre los componentes reemplazados, las actividades de reparación hechas, etc. Estos eventos registran los patrones de degradación. Si no se cuenta con esta información fundamental en los datos de aprendizaje se pueden obtener resultados engañosos para el modelo. El historial de errores también se puede encontrar dentro del historial de mantenimiento como códigos de error especiales o las fechas de pedidos de las piezas. Los expertos del dominio deben investigar y proporcionar orígenes de datos adicionales que influyen en los patrones de error.

#### <a name="machine-operating-conditions"></a>Condiciones de funcionamiento de la máquina
Los datos de streaming basados en sensor (u otro) del equipo en funcionamiento están un origen de datos importante. Un supuesto clave del mantenimiento predictivo es que el estado de mantenimiento de una máquina se degrada con el tiempo durante su funcionamiento de rutina. Se espera que los datos contengan características que varían con el tiempo y que capturan este patrón de vencimiento, además de todas las anomalías que llevan a la degradación. El aspecto temporal de los datos se requiere para que el algoritmo aprenda los patrones de errores y no de errores en el tiempo. En función de estos puntos de datos, el algoritmo aprende a predecir cuántas unidades de tiempo más puede seguir funcionando una máquina antes de que se produzca un error.

#### <a name="static-feature-data"></a>Datos de características estáticas
Las características estáticas son metadatos sobre el equipo. Algunos ejemplos son la marca del equipo, el modelo, la fecha de fabricación, la fecha de inicio del servicio, la ubicación del sistema y otras especificaciones técnicas.

A continuación, se tabulan ejemplos de datos relevantes para los [casos de uso de mantenimiento predictivo de ejemplo](#sample-pdm-use-cases):

| Caso de uso | Ejemplos de datos relevantes |
|:---------|---------------------------|
|_Retrasos y cancelaciones de vuelos_ | Información de ruta de vuelo en forma de escalas de vuelo y los registros de incidencias. Los datos de las escalas incluyen detalles sobre las rutas, como la fecha y hora de salida y llegada, el aeropuerto, las escalas, etc. Los registros de incidencias incluyen una serie de códigos de error y de mantenimiento que registra el personal de mantenimiento en tierra.|
|_Errores de las piezas del motor de un avión_ | Datos obtenidos de los sensores del avión que proporciona información sobre la condición de las distintas piezas. Los registros de mantenimiento ayudan a identificar cuándo se produjeron errores de componentes y cuándo se reemplazaron.|
|_Errores en un cajero automático_ | Lecturas del sensor en cada transacción (depósito de efectivo/cheque) y despacho de efectivo. Información de medición de la distancia entre billetes, el grosos de los billetes, la distancia que deben recorrer los billetes, los atributos de los cheques, etc. Registros de mantenimiento que proporcionan códigos de error, información de reparación, la última hora en que se rellenó el dispensador de efectivo.|
|_Error en turbinas eólicas_ | Los sensores supervisan las condiciones de las turbinas, como temperatura, dirección del viento, la energía generada, la velocidad del generador, etc. Los datos se recopilan de varias turbinas de parques eólicos ubicados en distintas regiones. Por lo general, cada turbina tendrá las lecturas de varios sensores para transmitir las medidas según un intervalo de tiempo fijo.|
|_Errores de disyuntores_ | Registros de mantenimiento que incluyen las acciones correctivas, preventivas y sistemáticas. Datos operativos que incluyen comandos manuales y automáticos enviados a los disyuntores como para las acciones de apertura y cierre. Metadatos del dispositivo, como la fecha de fabricación, la ubicación, el modelo, etc. Especificaciones de los disyuntores, como los niveles de voltaje, la geolocalización y las condiciones ambientales.|
|_Errores en puertas de ascensores_| Metadatos de los ascensores, como el tipo de ascensor, la fecha de fabricación, la frecuencia de mantenimiento, el tipo de edificio, etc. Información operativa como el número de ciclos de las puertas, el tiempo promedio de cierre de las puertas. Historial de errores con causas.|
|_Errores de las ruedas_ | Datos de sensores que miden la aceleración de las ruedas, las instancias de frenado, la distancia de conducción, la velocidad, etc. Información estática sobre las ruedas, como el fabricante y la fecha de fabricación. Datos de error inferidos a partir de la base de datos de pedidos de piezas que hacen seguimiento de las fechas de pedidos y cantidades.|
|_Errores en las puertas del metro_ | Tiempos de apertura y cierre de las puertas y otros datos operativos, como la condición actual de las puertas de los carros. Los datos estáticos incluirían las columnas del valor de condición, el tiempo y el identificador del recurso.|

### <a name="data-types"></a>Tipos de datos
Dados los orígenes de datos anteriores, los dos tipos de datos principales observados en el dominio de mantenimiento predictivo son:

- _Datos temporales_: telemetría operativa, condiciones de la máquina, los tipos de órdenes de trabajo, los códigos de prioridad que tendrán marcas de tiempo en el momento del registro. Los historiales de errores, de mantenimiento o reparaciones y de uso también tendrán marcas de tiempo asociadas con cada evento.
- _Datos estáticos_: en general, tanto las características de la máquina como las características del operador son estáticas, porque describen las especificaciones técnicas de las máquinas o los atributos del operador. Si estas características pudieran cambiar con el tiempo, también deberían tener asociadas marcas de tiempo.

Las variables de predicción y de destino se deben preprocesar o transformar en [tipos de datos numéricos, de categorías y otros tipos](https://www.statsdirect.com/help/basics/measurement_scales.htm) en función del algoritmo que se va a usar.

### <a name="data-preprocessing"></a>Preprocesamiento de datos
Como requisito previo para la _ingeniería de características_, prepare los datos provenientes de varios flujos para componer un esquema desde donde es sencillo crear características. Visualice los datos primero como una tabla de registros. Cada fila de la tabla representa una instancia de aprendizaje y las columnas representan las características de _predicción_ (también denominadas variables o atributos independientes). Organice los datos de manera que las últimas columnas sean el _destino_ (variable dependiente). Para cada instancia de aprendizaje, asigne una _etiqueta_ como el valor de esta columna.

En el caso de los datos temporales, divida la duración de los datos de sensor en unidades de tiempo. Cada registro debe pertenecer a una unidad de tiempo para un recurso, _y debe ofrecer información distinta_. Las unidades de tiempo se definen en función de las necesidades de negocio en múltiplos de segundos, minutos, horas, días, meses, etc. La unidad de tiempo _no tiene que ser la misma frecuencia de la recopilación de datos_. Si la frecuencia es alta, los datos pueden no mostrar ninguna diferencia significativa de una unidad a la otra. Por ejemplo, supongamos que la temperatura ambiente se recopiló cada 10 segundos. Usar ese mismo intervalo para los datos de aprendizaje solo infla el número de ejemplos sin proporcionar información adicional alguna. En este caso, una mejor estrategia sería usar el promedio de los datos más de 10 minutos, o bien una hora en función de la justificación comercial.

Para los datos estáticos:
- _Registros de mantenimiento_: los datos de mantenimiento sin formato tienen un identificador de recurso y una marca de tiempo con información sobre las actividades de mantenimiento que se han realizado en un momento dado en el tiempo. Transforme las actividades de mantenimiento en columnas de _categoría_, donde cada descriptor de categoría se asigna de manera única a una acción de mantenimiento específica. El esquema de los registros de mantenimiento incluye el identificador del recurso, tiempo y acción de mantenimiento.

- _Registros de errores_: los errores o motivos de error se pueden registrar como códigos de error específicos o eventos de error definidos por condiciones de negocio específicas. En casos donde el equipo tiene varios códigos de error, el experto del dominio debe ayudar a identificar los que son pertinentes para la variable de destino. Use el resto de los códigos de error o las condiciones para construir características de _predicción_ que se correlacionan con estos errores. El esquema de los registros de error incluye el identificador de recurso, tiempo, error o motivo del error, si está disponible.

- _Metadatos de máquina y operador_: combine los datos de máquina y operador en un esquema para asociar un recurso con su operador, junto con sus atributos respectivos. El esquema de las condiciones de la máquina incluye el identificador de recurso, las características del recurso, el identificador del operador y las características del operador.

Otros pasos del preprocesamiento de los datos incluyen el _control de los valores que faltan_ y la _normalización_ de los valores de atributo. Un debate detallado escapa del ámbito de esta guía: consulte la sección siguiente para ver algunas referencias útiles.

Con los orígenes de datos preprocesados anteriores implementados, la transformación final antes de la ingeniería de características es combinar las tablas anteriores según el identificador del recurso y la marca de tiempo. La tabla resultante tendrá valores NULL para la columna de error cuando la máquina funciona correctamente. Un indicador puede imputar estos valores NULL para el funcionamiento normal. Use esta columna de error para crear _etiquetas para el modelo predictivo_. Para más información, consulte la sección sobre las [técnicas de modelado para el mantenimiento predictivo](#modeling-techniques-for-predictive-maintenance).

## <a name="feature-engineering"></a>Ingeniería de características
La ingeniería de características es el primer paso antes de modelar los datos. [Aquí se describe](./create-features.md) su rol en el proceso de ciencia de datos. Una _característica_ es un atributo predictivo del modelo, como la temperatura, presión, vibración, etc. Para el mantenimiento predictivo, la ingeniería de características implica la abstracción del estado de una máquina respecto de los datos históricos recopilados durante un tiempo considerable. En ese sentido, es distinto de las características del mismo nivel, como la supervisión remota, la detección de anomalías y la detección de errores. 

### <a name="time-windows"></a>Ventanas de tiempo
La supervisión remota implica informar los eventos que se producen a partir de _momentos específicos_. Los modelos de detección de anomalías evalúan (puntúan) flujos de datos entrantes para marcar las anomalías a partir de momentos específicos. La detección de errores clasifica los errores para que sean de tipos específicos, dado que se producen en momentos específicos. En cambio, el mantenimiento predictivo implica predecir errores durante un _período de tiempo futuro_, según las características que representan el comportamiento de la máquina durante un _período de tiempo histórico_. Para el mantenimiento predictivo, los datos de características a partir de momentos específicos individuales tienen demasiada interferencia como para ser predictivos. Por tanto, los datos de cada característica se deben _suavizar_, agregándoles puntos de datos durante ventanas de tiempo.

### <a name="lag-features"></a>Características de retraso
Los requisitos empresariales definen hasta qué punto el modelo debe predecir en el futuro. A su vez, esta duración ayuda a definir "hasta qué punto el modelo debe mirar" para hacer estas predicciones. Este período "retrospectivo" se denomina _retraso_ y las características diseñadas durante este período de retraso se denominan _características de retraso_. En esta sección se describen las características de retraso que se pueden construir a partir de orígenes de datos con marcas de tiempo y la creación de características de orígenes de datos estáticos. Por lo general, las características de retraso son _numéricas_ por naturaleza.

> [!IMPORTANT]
> El tamaño de la ventana se determina mediante la experimentación y se debe finalizar con ayuda de un experto del dominio. La misma salvedad se aplica a la selección y la definición de características de retraso, sus agregaciones y el tipo de ventanas.

#### <a name="rolling-aggregates"></a>Agregados acumulados
En cada registro de un recurso, elegimos una ventana móvil de tamaño "W", que es el número de unidades de tiempo para las cuales calcular los agregados. A continuación, se calculan las características de retraso mediante los períodos de W _anteriores a la fecha_ de dicho registro. En la ilustración 1, las líneas azules muestran los valores de sensor registrados para un recurso para cada unidad de tiempo. Indican una media móvil de los valores de característica para una ventana de tamaño W=3. La media móvil se calcula sobre todos los registros con marcas de tiempo en el intervalo t<sub>1</sub> (en color naranjo) para t<sub>2</sub> (en color verde). Habitualmente, el valor de W es en minutos u horas, según la naturaleza de los datos. Pero para ciertos problemas, elegir un valor alto para W (digamos de 12 meses) puede proporcionar el historial completo de un recurso hasta el momento del registro.

![Figura 1. Características de agregado en ventanas acumuladas](./media/predictive-maintenance-playbook/rolling-aggregate-features.png)

Figura 1. Características de agregado en ventanas acumuladas

Ejemplos de agregados acumulados en una ventana de tiempo son recuento, promedio, medidas CUMSUM (suma acumulativa), valores mínimos y máximos. Además, a menudo se usa la varianza, la desviación estándar y el recuento de valores atípicos más allá de N desviaciones estándar. A continuación, se muestran ejemplos de los agregados que se pueden aplicar a los [casos de uso](#sample-pdm-use-cases) que aparecen en esta guía. 
- _Retraso de vuelos_: recuento de los códigos de error durante el último día o semana.
- _Errores en las piezas del motor de un avión_: medias acumuladas, desviación estándar y suma sobre el último día, semana, etc. Esta métrica se debe determinar junto con el experto del dominio empresarial.
- _Errores de los cajeros automáticos_: medias acumuladas, mediana, intervalo, desviaciones estándar, recuento de valores atípicos más allá de tres desviaciones estándar, CUMSUM superior e inferior.
- _Errores en las puertas del metro_: recuento de los eventos del último día, semana, dos semanas, etc.
- _Errores de disyuntores_: número de errores durante la última semana, año, tres años, etc.

Otra técnica útil en el mantenimiento predictivo es capturar los cambios de tendencia, picos y cambios de nivel mediante algoritmos que detectan anomalías en los datos.

#### <a name="tumbling-aggregates"></a>Agregados de saltos de tamaño constante
En cada registro etiquetado de un recurso, se define una ventana de tamaño _W-<sub>k</sub>_ , donde _k_ es el número de ventanas de tamaño _W_. Luego se crean agregados en _k_ _ventanas de saltos de tamaño constante_ _W-k, W-<sub>(k-1)</sub>, …, W-<sub>2</sub>, W-<sub>1</sub>_ para los períodos anteriores a la marcha de tiempo de un registro. _k_ puede ser un número pequeño para capturar efectos a corto plazo o un número grande para capturar patrones de degradación a largo plazo. (Consulte la ilustración 2).

![Ilustración 2. Características de agregados de saltos de tamaño constante](./media/predictive-maintenance-playbook/tumbling-aggregate-features.png)

Ilustración 2. Características de agregados de saltos de tamaño constante

Por ejemplo, las características de retraso del caso de uso de las turbinas eólicas se pueden crear con W=1 y k=3. Implican el retraso para cada uno de los tres últimos meses con valores atípicos superior e inferior.

### <a name="static-features"></a>Características estáticas

Las especificaciones técnicas del equipo, como la fecha de fabricación, el número de modelo, la ubicación, son algunos ejemplos de las características estáticas. Se consideran variables de _categoría_ para el modelado. Algunos ejemplos para el caso de uso de los disyuntores son el voltaje, la corriente, la potencia instalada, el tipo de transformador y la fuente de alimentación. En el caso de los errores de las ruedas, un ejemplo es el tipo de neumáticos (aleación o acero).

Los esfuerzos de preparación de los datos descritos hasta el momento deben implicar que los datos se organicen como se muestra a continuación. Los datos de aprendizaje, prueba y validación deben tener este esquema lógico (en este ejemplo se muestra el tiempo en unidades de días).

| Identificador de recurso | Time | \<Feature Columns> | Etiqueta |
| ---- | ---- | --- | --- |
| A123 |Día 1 | . . . | . |
| A123 |Día 2 | . . . | . |
| ...  |...   | . . . | . |
| B234 |Día 1 | . . . | . |
| B234 |Día 2 | . . . | . |
| ...  |...   | . . . | . |

El último paso en la ingeniería de características es el **etiquetado** de la variable de destino. Este proceso depende de la técnica de modelado. A su vez, la técnica de modelado depende del problema empresarial y de la naturaleza de los datos disponibles. El etiquetado se describe en la sección siguiente.

> [!IMPORTANT]
> La preparación de los datos y la ingeniería de características son tan importantes como las técnicas de modelado para lograr soluciones correctas de mantenimiento predictivo. El experto del dominio y el profesional deben invertir mucho tiempo para llegar a las características y los datos correctos para el modelo. A continuación, una pequeña muestra de los diversos libros que existen sobre la ingeniería de características:
> - Pyle, D. Data Preparation for Data Mining (The Morgan Kaufmann Series in Data Management Systems), 1999
> - Zheng, A., Casari, A. Feature Engineering for Machine Learning: Principles and Techniques for Data Scientists, O'Reilly, 2018.
> - Dong, G. Liu, H. (Editors), Feature Engineering for Machine Learning and Data Analytics (Chapman & Hall/CRC Data Mining and Knowledge Discovery Series), CRC Press, 2018.

## <a name="modeling-techniques-for-predictive-maintenance"></a>Técnicas de modelado para el mantenimiento predictivo

En esta sección se describen las principales técnicas de modelado para los problemas del mantenimiento predictivo, junto con sus métodos de construcción de etiquetas específicos. Tenga en cuenta que una técnica de modelado única se puede usar en distintos sectores. La técnica de modelado está emparejada con el problema de la ciencia de datos, en lugar del contexto de los datos en cuestión.

> [!IMPORTANT]
> La elección de las etiquetas de los casos de error y la estrategia de etiquetado  
> se debe determinar en consulta con el experto del dominio.

### <a name="binary-classification"></a>Clasificación binaria
La clasificación binaria se usa para _predecir la probabilidad de que una pieza de equipo presente un error en un período futuro_, que se conoce como el _período de horizonte futuro X_. La X la determina el problema empresarial y los datos en cuestión, en consulta con el experto del dominio. Algunos ejemplos son:
- _el plazo mínimo_ necesario para reemplazar componentes, implementar recursos de mantenimiento, realizar mantenimiento para evitar un problema que es probable que se produzca en ese período.
- _el recuento mínimo de eventos_ que pueden pasar antes de que se produzca un problema.

En esta técnica, se identifican dos tipos de ejemplos de aprendizaje. Un ejemplo positivo, _que indica un error_, con la etiqueta = 1. Un ejemplo negativo, que indica un funcionamiento normal, con la etiqueta = 0. La variable de destino, y por tanto, los valores de etiqueta, son _categorías_. El modelo debe identificar cada ejemplo nuevo que es probable que presente errores o funcione con normalidad durante las próximas X unidades de tiempo.

#### <a name="label-construction-for-binary-classification"></a>Construcción de etiquetas para la clasificación binaria
Aquí la pregunta es: "¿Cuál es la probabilidad de que el recurso presente un error en las próximas X unidades de tiempo?" Para responder esta pregunta, etiquete X registros antes del error de un recurso como "a punto del error" (etiqueta = 1) y etiquete todos los demás registros como "normales" (etiqueta =0). (Consulte la ilustración 3).

![Figura 3. Etiquetado para clasificación binaria](./media/predictive-maintenance-playbook/labelling-for-binary-classification.png)

Figura 3. Etiquetado para clasificación binaria

A continuación, se muestran ejemplos de estrategia de etiquetado para algunos de los casos de uso.
- _Retrasos de vuelos_: Se puede elegir X como un día para predecir los retrasos en las 24 horas siguientes. Todos los vuelos que están dentro de las 24 horas antes de los errores están etiquetados como 1.
- _Errores en la dispensación de efectivo en los cajeros automáticos_: un objetivo puede ser determinar la probabilidad de error de una transacción en la hora siguiente. En ese caso, todas las transacciones realizadas dentro de la última hora del error se etiquetan como 1. Para predecir la probabilidad de error sobre los próximos N billetes dispensados, todos los billetes dispensados dentro de los últimos N billetes de un error se etiquetan como 1.
- _Errores de disyuntores_: el objetivo puede ser predecir el próximo error del mando del disyuntor. En ese caso, X será un mando futuro.
- _Errores en las puertas del metro_: X se puede elegir como dos días.
- _Error en turbinas eólicas_: X se puede elegir como dos meses.

### <a name="regression-for-predictive-maintenance"></a>Regresión para mantenimiento predictivo
Se usan modelos de regresión para _calcular la vida útil restante (RUL) de un recurso_. La vida útil restante se define como la cantidad de tiempo que un recurso está operativo antes de que se produzca el error siguiente. Cada ejemplo de aprendizaje es un registro que pertenece a una unidad de tiempo _nY_ de un recurso, donde _n_ es el múltiplo. El modelo debe calcular la RUL de cada ejemplo nuevo como un _número continuo_. Este número indica el período de tiempo que queda antes del error.

#### <a name="label-construction-for-regression"></a>Construcción de etiquetas de la regresión
Aquí la pregunta es: "¿Cuál es la vida útil restante (RUL) del equipamiento?" Para cada registro antes del error, calcule la etiqueta para que sea el número de unidades de tiempo restantes antes del error siguiente. En este método, las etiquetas son variables continuas. (Consulte la ilustración 4).

![Figura 4. Etiquetado de la regresión](./media/predictive-maintenance-playbook/labelling-for-regression.png)

Figura 4. Etiquetado de la regresión

Para la regresión, el etiquetado se hace haciendo referencia a un punto de error. No se puede calcular sin saber cuánto tiempo ha sobrevivido el recurso antes de un error. Por tanto, y a diferencia de la clasificación binaria, los recursos sin errores en los datos no se pueden usar para el modelado. Este problema se aborda mejor con otra técnica estadística llamada [Análisis de supervivencia](https://en.wikipedia.org/wiki/Survival_analysis). Pero pueden surgir complicaciones al aplicar esta técnica a casos de uso de mantenimiento predictivo que implican datos variables en el tiempo a intervalos frecuentes. Para más información sobre Análisis de supervivencia, consulte [este documento de una página](https://www.cscu.cornell.edu/news/news.php/stnews78.pdf).

### <a name="multi-class-classification-for-predictive-maintenance"></a>Clasificación multiclase para mantenimiento predictivo
Las técnicas de clasificación multiclase se pueden usar en las soluciones de mantenimiento predictivo en dos escenarios:
- Predecir _dos resultados futuros_: el primer resultado es _un intervalo de tiempo de error_ de un recurso. El recurso se asigna a uno de varios posibles períodos de tiempo. El segundo resultado es la probabilidad de error en un período futuro debido a _una de las diferentes causas principales_. Esta predicción permite que el personal de mantenimiento inspeccione los síntomas y planifique programaciones de mantenimiento.
- Prediga la _causa principal más probable_ de un error determinado. Este resultado recomienda el conjunto correcto de acciones de mantenimiento para corregir un error. Una lista clasificada de las causas principales y las reparaciones recomendadas puede ayudar a los técnicos a establecer las prioridades de sus acciones de reparación después de un error.

#### <a name="label-construction-for-multi-class-classification"></a>Construcción de etiquetas para la clasificación multiclase
Aquí la pregunta es: "¿Cuál es la probabilidad de que un recurso presente un error en las próximas _nZ_ unidades de tiempo, donde _n_ es el número de períodos?" Para responder esta pregunta, etiquete nZ registros antes del error de un recurso mediante depósitos de tiempo (3Z, 2Z, Z). Etiquete todos los otros registros como "normales" (etiqueta = 0). En este método, la variable de destino contiene los valores de _categoría_. (Consulte la ilustración 5).

![Figura 5. Etiquetas de predicción del momento del error para la clasificación multiclase](./media/predictive-maintenance-playbook/labelling-for-multiclass-classification-for-failure-time-prediction.png)

Figura 5. Etiquetado de la clasificación multiclase para predecir el momento del error

Aquí la pregunta es: "¿Cuál es la probabilidad de que el recurso presente un error en las próximas X unidades de tiempo debido a la causa principal/problema _P <sub>i</sub>_ ?" donde _i_ es el número de las causas principales posibles. Para responder esta pregunta, etiquete X registros antes del error de un recurso como "a punto del error debido a la causa principal _P <sub>i</sub>_ " (etiqueta = _P <sub>i</sub>_ ). Etiquete todos los otros registros como "normales" (etiqueta = 0). En este método, las etiquetas también son de categoría (consulte la ilustración 6).

![Figura 6. Etiquetas de predicción de la causa principal para la clasificación multiclase](./media/predictive-maintenance-playbook/labelling-for-multiclass-classification-for-root-cause-prediction.png)

Figura 6. Etiquetado de la clasificación multiclase para predecir la causa del error

El modelo asigna una probabilidad de error debido a cada _P <sub>i</sub>_ , así como la probabilidad de que no se produzca ningún error. Estas probabilidades se pueden ordenar por magnitud para permitir la predicción de los problemas que es más probable que ocurran en el futuro.

Aquí la pregunta es: "¿Qué acciones de mantenimiento se recomiendan después de un error?" Para responder esta pregunta, el etiquetado _no requiere elegir un horizonte futuro_, porque el modelo no predice ningún error en el futuro. Simplemente predice la causa principal más probable _una vez que ya se produjo el error_.

## <a name="training-validation-and-testing-methods-for-predictive-maintenance"></a>Métodos de aprendizaje, validación y prueba para mantenimiento predictivo
El [proceso de ciencia de datos en equipo](./overview.md) proporciona una completa cobertura del ciclo de aprendizaje-prueba-validación del modelo. En esta sección se analizan los aspectos exclusivos del mantenimiento predictivo.

### <a name="cross-validation"></a>Validación cruzada
El objetivo de la [validación cruzada](https://en.wikipedia.org/wiki/Cross-validation_(statistics)) es definir un conjunto de datos para "probar" el modelo en la fase de aprendizaje. Este conjunto de datos se conoce como el _conjunto de validación_. Esta técnica ayuda a limitar problemas como el _sobreajuste_ y ofrece una idea de cómo se generalizará el modelo para un conjunto de datos independiente. Es decir, un conjunto de datos desconocido que podría surgir de un problema real. La rutina de aprendizaje y prueba de mantenimiento predictivo debe considerar los aspectos que cambian con el tiempo generalizar mejor en datos futuros no vistos.

Muchos algoritmos de aprendizaje automático dependen de diferentes hiperparámetros que pueden cambiar significativamente el rendimiento del modelo. Los valores óptimos de estos hiperparámetros no se calculan automáticamente al entrenar el modelo. Los deben especificar los científicos de datos. Hay varias formas de buscar los valores correctos para los hiperparámetros.

La más común es la _validación cruzada con k iteraciones_ que divide los ejemplos aleatoriamente en _k_ iteraciones. Para cada conjunto de valores de hiperparámetro, ejecute el algoritmo de aprendizaje _k_ veces. En cada iteración, use los ejemplos de la iteración actual como un conjunto de validación y use el resto de los ejemplos como un conjunto de aprendizaje. Entrene el algoritmo con los ejemplos de aprendizaje y las calcule las métricas de rendimiento con los ejemplos de validación. Al final de este bucle, calcule el promedio de _k_ métricas de rendimiento. Para cada conjunto de valores de hiperparámetros, elija los que tengan un mejor rendimiento promedio. La tarea de elegir hiperparámetros suele ser experimental por naturaleza.

En los problemas de mantenimiento predictivo, los datos se registran como una serie temporal de eventos procedentes de diversos orígenes de datos. Estos registros se pueden ordenar según la hora de etiquetado. Por lo tanto, si se divide el conjunto de datos _aleatoriamente_ en el conjunto de aprendizaje y el de validación, _algunos de los ejemplos de aprendizaje podrán ser posteriores a algunos de los ejemplos de validación_. Se calculará el rendimiento futuro de los valores de hiperparámetro en función de algunos datos que llegaron _antes_ de que el modelo se entrenara. Estas estimaciones podrían resultar extremadamente optimistas, especialmente si las series temporales no son fijas y evolucionan con el tiempo. Como resultado, los valores de hiperparámetro elegidos podrían no ser los óptimos.

El método recomendado es dividir los ejemplos en conjuntos de aprendizaje y validación establecidos de manera _dependiente del tiempo_, donde todos los ejemplos de validación sean posteriores en el tiempo a todos los ejemplos de aprendizaje. Para cada conjunto de valores de hiperparámetro, entrene el algoritmo en el conjunto de datos de aprendizaje. Mida el rendimiento del modelo en el mismo conjunto de validación. Elija los valores de hiperparámetro que muestra el mejor rendimiento. Los valores de hiperparámetro elegidos por la división en aprendizaje y validación producen un mejor rendimiento futuro del modelo que los que la validación cruzada elige aleatoriamente.

El modelo final puede generarse mediante el entrenamiento de un algoritmo de aprendizaje sobre los datos de aprendizaje completos con los mejores valores de hiperparámetros.

### <a name="testing-for-model-performance"></a>Pruebas de rendimiento del modelo
Una vez que se genera un modelo, se requiere una estimación de su rendimiento en el futuro en datos nuevos. Una estimación adecuada es la métrica de rendimiento de los valores de hiperparámetro calculada con el conjunto de validación o una métrica de rendimiento promedio calculada mediante validación cruzada. Estos cálculos suelen ser demasiado optimistas. A menudo, la empresa podría tener algunas instrucciones adicionales sobre cómo desean probar el modelo.

El método recomendado para mantenimiento predictivo es dividir los ejemplos en conjuntos de datos de aprendizaje, validación y prueba de una manera _dependiente del tiempo_. Todos los ejemplos de prueba deben ser posteriores en el tiempo a todos los ejemplos de aprendizaje y validación. Después de la división, genere el modelo y mida su rendimiento, tal como se describió anteriormente.

Cuando las series temporales son fijas y fáciles de predecir, ambos enfoques aleatorios y dependientes del tiempo generan estimaciones similares del rendimiento futuro. Pero cuando las series temporales no son fijas o son difíciles de predecir, el enfoque dependiente del tiempo generará cálculos más realistas del rendimiento futuro.

### <a name="time-dependent-split"></a>División dependiente del tiempo
En esta sección se describen los procedimientos recomendados para implementar una división dependiente del tiempo. A continuación, se describe una división bidireccional dependiente del tiempo entre los conjuntos de aprendizaje y prueba.

Supongamos que tenemos una secuencia de eventos con marca de fecha y hora, tales como medidas de diferentes sensores. Defina las características y las etiquetas de los ejemplos de aprendizaje y prueba en períodos de tiempo que contienen varios eventos. Por ejemplo, para la clasificación binaria, cree características en función de los eventos pasados y cree etiquetas en función de los eventos futuros con "X" unidades de tiempo en el futuro (consulte las secciones sobre [ingeniería de características](#feature-engineering) y técnicas de modelado). Por lo tanto, el período de tiempo del etiquetado de un ejemplo es posterior al período de tiempo de sus características.

Para la división dependiente del tiempo, seleccione un _tiempo límite de aprendizaje T <sub>c</sub>_ en el que se entrenará un modelo, con hiperparámetros optimizados mediante el uso de datos históricos hasta T <sub>c</sub>. Para evitar que etiquetas futuras que están mas allá de T<sub>c</sub> se fuguen a los datos de aprendizaje, elija el período de tiempo más reciente para etiquetar los ejemplos de aprendizaje en X unidades antes de T<sub>c</sub>. En el ejemplo que se muestra en la ilustración 7, cada cuadrado representa un registro del conjunto de datos donde las características y las etiquetas se calculan como se describió anteriormente. La ilustración muestra los registros que deben ir en los conjuntos de aprendizaje y de prueba para X=2 y W=3:

![Ilustración 7. División dependiente del tiempo para la clasificación binaria](./media/predictive-maintenance-playbook/time-dependent-split-for-binary-classification.png)

Ilustración 7. División dependiente del tiempo para la clasificación binaria

Los cuadrados verdes representan los registros que pertenecen a las unidades de tiempo que pueden usarse para el aprendizaje. Cada ejemplo de aprendizaje se genera considerando los últimos tres períodos para la generación de características y dos períodos futuros para el etiquetado antes de T<sub>c</sub>. Cuando cualquier parte de los dos períodos futuros vaya más allá de T<sub>c</sub>, excluya ese ejemplo del conjunto de datos de aprendizaje porque no se supone visibilidad ninguna más allá de T<sub>c</sub>.

Los cuadrados negros representan los registros del conjunto de datos etiquetado final que no se deben usar en el conjunto de datos de aprendizaje, dada la restricción mencionada. Estos registros tampoco se usarán en los datos de prueba, porque son anteriores a T<sub>c</sub>. Además, los períodos de tiempo del etiquetado dependen parcialmente del período de tiempo del aprendizaje, lo que es no la situación ideal. Los datos de aprendizaje y de prueba deben tener períodos de tiempo de etiquetado independientes para impedir la fuga de información del etiquetado.

La técnica descrita hasta este momento permite la superposición de los ejemplos de aprendizaje y prueba que tienen marcas de tiempo cerca de T<sub>c</sub>. Una solución para lograr mayor separación es excluir ejemplos que estén dentro de W unidades de tiempo de T<sub>c</sub> del conjunto de prueba. Pero esta división agresiva depende de la amplia disponibilidad de los datos.

Los modelos de regresión que se usan para predecir la RUL se ven más gravemente afectados por el problema de la fuga. Usar el método de división aleatoria lleva a un sobreajuste extremo. Para los problemas de regresión, la división debe ser tal que los registros que pertenecen a los recursos con errores antes de T<sub>c</sub> vayan en el conjunto de aprendizaje. Los registros de los recursos que tienen errores después de la fecha límite entran en el conjunto de prueba.

Otro procedimiento recomendado para dividir los datos para aprendizaje y prueba es usar una división por identificador de recurso. La división debe ser tal que ninguno de los recursos que se usen en el conjunto de aprendizaje se use para probar el rendimiento del modelo. Con este enfoque, un modelo tiene más posibilidades de proporcionar resultados más realistas con nuevos recursos.

### <a name="handling-imbalanced-data"></a>Control de datos desequilibrados
En los problemas de clasificación, si hay más ejemplos de una clase que de otra, se dice que el conjunto de datos está _desequilibrado_. Lo ideal es tener suficientes representantes de cada clase en los datos de aprendizaje para permitir diferenciar entre las distintas clases. Si una clase tiene menos del 10 % de los datos, los datos se consideran como desequilibrados. La clase insuficientemente representada se denomina una _clase minoritaria_.

Muchos problemas de mantenimiento predictivo enfrentan esos conjuntos de datos desequilibrados, donde una clase es demasiado insuficientemente representada en comparación con la otra clase o clases. En algunas situaciones, la clase minoritaria puede constituir solo 0,001 % del total de puntos de datos. El desequilibrio de clases no es exclusivo del mantenimiento predictivo. Otros dominios donde los errores y las anomalías son situaciones inusuales enfrentan un problema similar, por ejemplo, la detección de fraudes e intrusión en la red. Estos errores conforman los ejemplos de la clase minoritaria.

Con un desequilibrio de clases en los datos, el rendimiento de la mayoría de los algoritmos de aprendizaje estándar se ve comprometido, ya que intentan minimizar la tasa general de errores. Para un conjunto de datos con un 99 % de ejemplos negativos y un 1 % de ejemplos de positivos, podemos obtener un 99 % de precisión simplemente etiquetando todas las instancias como negativas. Pero el modelo clasificará de manera incorrecta todos los ejemplos positivos, por tanto, aunque la precisión sea alta, el algoritmo no es útil. Por lo tanto, las métricas de evaluación convencionales tales como la _precisión global en la tasa de errores_ no son suficientes en el caso del aprendizaje desequilibrado. Cuando uno se enfrenta con conjuntos de datos desequilibrados, se usan otras métricas para evaluar el modelo:
- Precision
- Recuperación
- Puntuaciones F1
- ROC con ajuste del costo (características operativas del receptor)

Para más información sobre estas métricas, consulte la [evaluación de modelos](#model-evaluation).

Sin embargo, hay algunos métodos que ayudan a solucionar el problema de desequilibrio de clases. Las dos más importantes son las _técnicas de muestreo_ y el _aprendizaje sensible al costo_.

#### <a name="sampling-methods"></a>Métodos de muestreo
El aprendizaje desequilibrado implica el uso de métodos de muestreo para modificar el conjunto de datos de aprendizaje a un conjunto de datos equilibrado. Los métodos de muestreo no se deben aplicar al conjunto de prueba. Aunque hay muchas técnicas de muestreo, las más directas son el _sobremuestreo aleatorio_ y el _submuestreo aleatorio_.

El _sobremuestreo aleatorio_ consiste en seleccionar una muestra aleatoria de una clase minoritaria, replicar estos ejemplos y agregarlos al conjunto de datos de aprendizaje. Como consecuencia, aumenta el número de ejemplos en la clase minoritaria y, a la larga, se equilibraría el número de ejemplos de las diferentes clases. Una desventaja del sobremuestreo es que varias instancias de ciertos ejemplos pueden hacer que el clasificador sea demasiado específico y provocar un sobreajuste. El modelo puede mostrar una gran precisión del aprendizaje, pero su rendimiento en los datos de prueba no vistos puede ser muy deficiente.

Por el contrario, el _submuestreo aleatorio_ consiste en seleccionar una muestra aleatoria de una clase mayoritaria y quitar esos ejemplos del conjunto de datos de aprendizaje. Sin embargo, quitar ejemplos de la clase de mayoría podría provocar que el clasificador pase por alto conceptos importantes relacionados con la clase mayoritaria. Otro enfoque viable es un _muestreo híbrido_ que consiste en realizar un sobremuestreo de una clase minoritaria y un submuestreo de una clase mayoritaria.

Hay muchas técnicas de muestreo sofisticadas. La técnica elegida depende de las propiedades de datos y los resultados de experimentos iterativos por los científicos de datos.

#### <a name="cost-sensitive-learning"></a>Aprendizaje sensible al costo
En el mantenimiento predictivo, los errores que constituyen la clase minoritaria son más interesantes que los ejemplos normales. Por tanto, el foco principalmente está en el rendimiento del algoritmo en errores. Predecir incorrectamente una clase como una clase positiva como negativa puede ser más costoso que a la inversa. Esta situación se conoce normalmente como pérdida desigual o costos asimétricos de la clasificación errónea de elementos de diferentes clases. El clasificador ideal debe ser capaz de proporcionar una gran precisión de predicción en la clase minoritaria sin poner en riesgo la precisión de la clase de mayoría.

Hay varias formas de alcanzar este equilibrio. Para mitigar el problema de la pérdida desigual, asigne un alto costo a la clasificación errónea de la clase minoritaria y trate de minimizar el costo general. Los algoritmos como _SVM (máquinas de vectores de soporte)_ adoptan este método de manera inherente, al permitir especificar el costo de los ejemplos positivos y negativos durante el aprendizaje. De manera similar, los métodos de incremento, como los _árboles de decisiones incrementados_, habitualmente muestran un buen rendimiento con los datos desequilibrados.

## <a name="model-evaluation"></a>Evaluación del modelo
La clasificación errónea es un problema importante en escenarios de mantenimiento predictivo donde el costo de alarmas falsas es alto para el negocio. Por ejemplo, una decisión de inmovilizar un avión debido a una predicción incorrecta de un error en el motor puede interrumpir no solo horarios, sino también planes de viajes. Sacar una máquina de la línea de ensamblado puede generar pérdidas de ingresos. Por tanto, es fundamental la evaluación de modelos con las métricas de rendimiento correctas respecto de los datos de prueba nuevos.

A continuación, se describen las métricas de rendimiento típicas que se usan para evaluar los modelos de mantenimiento predictivo:

- La [precisión](https://en.wikipedia.org/wiki/Accuracy_and_precision) es la métrica más popular que se usa para describir el rendimiento de un clasificador. Pero la precisión es importante para las distribuciones de datos y es una medida ineficaz para escenarios con conjuntos de datos desequilibrados. En su lugar, se usan otras métricas. Herramientas como la [matriz de confusión](https://en.wikipedia.org/wiki/Confusion_matrix) se usan para calcular y razonar sobre la precisión del modelo.
- La [precisión](https://en.wikipedia.org/wiki/Precision_and_recall) de los modelos de mantenimiento predictivo se relacionan con la tasa de alarmas falsas. La menor precisión del modelo suele corresponder a una tasa mayor de alarmas falsas.
- La tasa de [recuperación](https://en.wikipedia.org/wiki/Precision_and_recall) indica cuántos errores del conjunto de prueba identificó correctamente el modelo. Las tasas de recuperación mayores indican que el modelo pudo identificar correctamente los errores verdaderos.
- La [puntuación F1](https://en.wikipedia.org/wiki/F1_score) es el promedio armónico de precisión y recuperación, con su valor entre 0 (peor) a 1 (mejor).

Para la clasificación binaria:
- Las [curvas de características operativas del receptor (ROC)](https://en.wikipedia.org/wiki/Receiver_operating_characteristic) también son una métrica popular. En las curvas ROC, el rendimiento del modelo se interpreta en función de un punto operativo fijo en la curva ROC.
- Pero para los problemas de mantenimiento predictivo, las _tablas de deciles_ y los _gráficos de elevación_ proporcionan más información. Se centran solo en la clase positiva (errores) y proporcionan una imagen más compleja del rendimiento del algoritmo que las curvas ROC.
  - Las _tablas de deciles_ se crean mediante el uso de ejemplos de prueba en orden descendente de las probabilidades de error. Los ejemplos ordenados se agrupan en deciles (el 10 % de ejemplos con mayor probabilidad, después el 20 %, el 30 %, etc.). La relación (tasa de positivos verdaderos)/(línea base aleatoria) de cada decil ayuda a calcular el rendimiento del algoritmo en cada decil. La línea base aleatoria toma los valores 0,1, 0,2, 0,3, etc.
  - Los b[gráficos de elevación](http://www2.cs.uregina.ca/~dbd/cs831/notes/lift_chart/lift_chart.html) trazan la tasa de positivos verdaderos del decil en comparación con la tasa de positivos verdaderos aleatoria de todos los deciles. Los primeros deciles suelen ser el foco de los resultados porque muestran las mayores ganancias. Los primeros deciles también pueden verse como representativos de "en riesgo" cuando se usan para el mantenimiento predictivo.

## <a name="model-operationalization-for-predictive-maintenance"></a>Operacionalización del modelo para mantenimiento predictivo

La ventaja del ejercicio de la ciencia de datos solo se percibe cuando el modelo entrenado entra en funcionamiento. Es decir, el modelo se debe implementar en los sistemas de la empresa para crear predicciones en función de los datos nuevos, no vistos anteriormente.  Los datos nuevos deben cumplir exactamente con la _firma_ del modelo entrenado de dos maneras:
- todas las características deben estar presentes en cada instancia lógica (como una fila de una tabla) de los datos nuevos.
- los datos nuevos se deben preprocesar, y cada una de las características diseñadas, exactamente del mismo modo que los datos de aprendizaje.

El proceso mencionado se indica de muchas maneras en la literatura académica y especializada del sector. Pero todas las afirmaciones siguientes significan lo mismo:
- _Asignar puntuación a los datos nuevos_ con el modelo
- _Aplicar el modelo_ a los datos nuevos
- _Poner en marcha_ el modelo
- _Implementar_ el modelo
- _Ejecutar el modelo_ con datos nuevos

Como se mencionó anteriormente, la operacionalización del modelo para el mantenimiento predictivo es distinta de otros de su mismo nivel. Los escenarios que implican la detección de anomalías y la detección de errores suelen implementar la _puntuación en línea_ (que también se conoce como _puntuación en tiempo real_). Aquí, el modelo _asigna una puntuación_ a cada registro entrante y devuelve una predicción. En el caso de la detección de anomalías, la predicción es un indicador de que se produjo una anomalía (ejemplo: SVM de una clase). En el caso de la detección de errores, sería el tipo o la clase de error.

Por el contrario, el mantenimiento predictivo implica la _puntuación de Batch_. Para cumplir con la firma del modelo, las características de los datos nuevos se deben diseñar del mismo modo que los datos de aprendizaje. En el caso de los conjuntos de datos de gran tamaño típicos para los datos nuevos, las características se agregan durante períodos de tiempo y se les asigna una puntuación en Batch. Por lo general, la puntuación de Batch se hace en sistemas distribuidos como [Spark](https://spark.apache.org/) o [Azure Batch](../../batch/batch-service-workflow-features.md). Hay un par de alternativas, aunque ambas son insuficientes:
- Los motores de datos de streaming admiten la agregación de las ventanas en memoria. Por tanto, se podría argumentar que admiten la puntuación en línea. Pero estos sistemas son adecuados para los datos densos en ventanas de tiempo acotadas o elementos dispersos en ventanas más amplias. Es posible que no se ajusten correctamente a los datos denso en ventanas de tiempo más amplias, como se ve en los escenarios de mantenimiento predictivo.
- Si la puntuación de Batch no está disponible, la solución adaptará la puntuación en línea para administrar los datos nuevos en pequeños lotes a la vez.

## <a name="solution-templates-for-predictive-maintenance"></a>Plantillas de soluciones de mantenimiento predictivo

En la parte final de esta guía se proporciona una lista de plantillas de soluciones de mantenimiento predictivo, tutoriales y experimentos implementados en Azure. Las aplicaciones de mantenimiento predictivo se pueden implementar en una suscripción de Azure, en cuestión de minutos en algunos casos. Se pueden usar como demostraciones de prueba de concepto, espacios aislados para experimentar con alternativas o aceleradores para implementaciones de producción reales. Estas plantillas están en [Azure AI Gallery](https://gallery.azure.ai) o en [Azure GitHub](https://github.com/Azure). Estos ejemplos distintos se acumularán en esta plantilla de soluciones en el tiempo.

| # | Título | Descripción |
|--:|:------|-------------|
| 2 | [Plantilla de soluciones de mantenimiento predictivo de Azure](https://github.com/Azure/AI-PredictiveMaintenance) | Una plantilla de solución de código abierto que demuestra el modelado de Azure ML y una infraestructura completa de Azure que es capaz de admitir escenarios de mantenimiento predictivo en el contexto de supervisión remota de IoT. |
| 3 | [Aprendizaje profundo para mantenimiento predictivo](https://github.com/Azure/MachineLearningSamples-DeepLearningforPredictiveMaintenance) | Notebook de Azure con una solución de demostración sobre el uso de redes con memoria a corto y largo plazo (LSTM) (una clase de redes neuronales recurrentes) para el mantenimiento predictivo, con una [entrada de blog sobre este ejemplo](https://azure.microsoft.com/blog/deep-learning-for-predictive-maintenance).|
| 4 | [Mantenimiento predictivo de Azure para el sector aeroespacial](https://gallery.azure.ai/Solution/Predictive-Maintenance-for-Aerospace-1) | Una de las primeras plantillas de soluciones de mantenimiento predictivo basada en Azure ML v1.0 para el mantenimiento aeronáutico. Esta guía se originó a partir de este proyecto. |
| 5 | [Kit de herramientas de Azure AI para IoT Edge](https://github.com/Azure/ai-toolkit-iot-edge) | Inteligencia artificial en IoT Edge con TensorFlow; modelos de aprendizaje profundo con paquetes del kit de herramientas en contenedores Docker compatibles con Azure IoT Edge y exposición de esos modelos como API REST.
| 6 | [Mantenimiento predictivo de Azure IoT](https://github.com/Azure/azure-iot-predictive-maintenance) | PCS de Azure IoT Suite: solución preconfigurada. Plantilla de mantenimiento predictivo para el mantenimiento aeronáutico con IoT Suite. [Otro documento](/previous-versions/azure/iot-accelerators/about-iot-accelerators) y [tutorial](/previous-versions/azure/iot-accelerators/iot-accelerators-predictive-walkthrough) relacionados con el mismo proyecto. |
| 7 | [Plantilla de mantenimiento predictivo con SQL Server R Services](https://gallery.azure.ai/Tutorial/Predictive-Maintenance-Template-with-SQL-Server-R-Services-1) | Demostración del escenario de vida útil restante basado en R Services. |
| 8 | [Guía de modelado de mantenimiento predictivo](https://gallery.azure.ai/Collection/Predictive-Maintenance-Modelling-Guide-1) | Características del conjunto de datos de mantenimiento aeronáutico diseñadas a través de R con [experimentos](https://gallery.azure.ai/Experiment/Predictive-Maintenance-Modelling-Guide-Experiment-1) y [conjuntos de datos](https://gallery.azure.ai/Experiment/Predictive-Maintenance-Modelling-Guide-Data-Sets-1) y un cuaderno de Azure Notebooks, y [experimentos](https://gallery.azure.ai/Experiment/Predictive-Maintenance-Step-1-of-3-data-preparation-and-feature-engineering-2) en AzureML versión 1.0|

## <a name="training-resources-for-predictive-maintenance"></a>Recursos de aprendizaje para un mantenimiento predictivo

Microsoft Azure ofrece rutas de aprendizaje de los conceptos fundamentales sobre las técnicas de mantenimiento predictivo, además de contenido y formación sobre los conceptos sobre inteligencia artificial generales y prácticas.

| Recursos de aprendizaje  | Disponibilidad |
|:-------------------|--------------|
| [Microsoft Docs: rol Científico de datos](/learn/roles/data-scientist) | Público |
| [Microsoft Docs: rol Ingeniero de IA](/learn/roles/ai-engineer) | Público |
| [Microsoft Docs: rol Ingeniero de datos](/learn/roles/data-engineer) | Público |
| [Microsoft AI School](https://www.microsoft.com/ai/ai-school) | Público |
| [LinkedIn Learning](https://www.linkedin.com/learning) | Público |
| [Microsoft: Listas de reproducción en YouTube para inteligencia artificial y análisis](https://www.youtube.com/c/MicrosoftAzure/playlists?view=50&sort=dd&shelf_id=7) | Público |
| [Presentación de Microsoft AI](https://channel9.msdn.com/Shows/AI-Show) | Público |
| [Introducción a la plataforma de IA](https://azure.microsoft.com/overview/ai-platform/) | Público |
| [Laboratorio de inteligencia artificial](https://www.microsoft.com/ai/ai-lab) | Público |
| [IA de Microsoft](https://www.microsoft.com/AI) | Público |
| [Microsoft Partner Network](https://partner.microsoft.com/training/training-center) | Asociados |

Además, instituciones académicas, como Stanford y MIT, además de otras empresas educativas, ofrecen cursos en línea abiertos y masivos (MOOC) gratis sobre la inteligencia artificial.