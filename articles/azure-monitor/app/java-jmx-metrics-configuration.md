---
title: 'Configuración de métricas de JMX: Azure Monitor Application Insights para Java'
description: Configuración de la recopilación adicional de métricas de JMX para el agente de Java de Azure Monitor Application Insights
ms.topic: conceptual
ms.date: 03/16/2021
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 8b5db7861a3488fb71e709d17679e8d54911b0c9
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110069191"
---
# <a name="configuring-jmx-metrics"></a>Configuración de métricas de JMX

El agente de Java 3.x de Application Insights recopila algunas de las métricas de JMX de forma predeterminada, pero en muchos casos esto no es suficiente. En este documento se describe la opción de configuración de JMX en detalle.

## <a name="how-do-i-collect-additional-jmx-metrics"></a>¿Cómo se recopilan métricas de JMX adicionales?

La recopilación de métricas de JMX se puede configurar agregando una sección ```"jmxMetrics"``` al archivo applicationinsights.json. Puede especificar el nombre de la métrica de la forma en que desea que aparezca en Azure Portal en el recurso Application Insights. Debe definir el nombre de objeto y el atributo para cada una de las métricas que desea recopilar.

## <a name="how-do-i-know-what-metrics-are-available-to-configure"></a>¿Cómo puedo saber qué métricas están disponibles para configurar?

Ha dado en el clavo: debe conocer los nombres de los objetos y los atributos, esas propiedades son diferentes para varias bibliotecas, marcos y servidores de aplicaciones, y a menudo no están bien documentadas. Para obtener los nombres de objeto y los atributos, debe ver el árbol de MBean. Un MBean es un objeto Java administrado que puede representar un dispositivo, una aplicación o un recurso, y tiene un conjunto de atributos. 

Para ver las métricas disponibles y examinarlas, se recomienda usar [Java Mission Control](https://www.oracle.com/java/technologies/jdk-mission-control.html).

### <a name="how-to-navigate-the-java-mission-control-to-get-to-the-right-metrics"></a>¿Cómo navegar por Java Mission Control para llegar a las métricas correctas?

Al ejecutar la herramienta Java Mission Control, tendrá una selección de JVM disponible en el lado izquierdo, haga clic en el proceso correspondiente en la pestaña del explorador de JVM. Espere hasta que JMC cargue el panel para el proceso y seleccione la pestaña del explorador de MBean en la parte inferior (vea más abajo). El JMC debe estar ubicado en la misma carpeta que JVM y el proceso o la aplicación deben estar en funcionamiento.

![Captura de pantalla del explorador de MBean de JMC](media/java-ipa/jmx/jmc-mbean-browser.png)

### <a name="how-to-get-to-the-metrics-i-want-and-the-necessary-attributes"></a>¿Cómo puedo llegar a las métricas que quiero y los atributos necesarios?

El explorador de MBean abre el árbol de MBean con la lista de categorías que se pueden expandir. Al seleccionar una categoría a la izquierda, se abre la lista de atributos de la derecha. A continuación se muestra un ejemplo de una métrica, su nombre de objeto y los atributos. Los atributos pueden estar anidados, como en el ejemplo siguiente.

![Captura de pantalla del árbol de MBean de JMC](media/java-ipa/jmx/jmc-metric-sample.png)

### <a name="configuration-example"></a>Ejemplo de configuración

A partir de la selección, como se muestra en la imagen anterior, vamos a configurar algunas métricas. El primero es un ejemplo de una métrica anidada `LastGcInfo`, que tiene varias propiedades, y queremos capturar `GcThreadCount`.

```json
"jmxMetrics": [
      {
        "name": "Demo - GC Thread Count",
        "objectName": "java.lang:type=GarbageCollector,name=PS MarkSweep",
        "attribute": "LastGcInfo.GcThreadCount"
      },
      {
        "name": "Demo - GC Collection Count",
        "objectName": "java.lang:type=GarbageCollector,name=PS MarkSweep",
        "attribute": "CollectionCount"
      },
      {
        "name": "Demo - Thread Count",
        "objectName": "java.lang:type=Threading",
        "attribute": "ThreadCount"
      }
],
```

### <a name="types-of-collected-metrics-and-available-configuration-options"></a>¿Cuáles son los tipos de métricas recopiladas y las opciones de configuración disponibles?

Se admiten métricas de JMX numéricas y booleanas, mientras que otros tipos no se admiten y se omitirán. 

Actualmente, los comodines y los atributos agregados no se admiten, por lo que cada par "nombre de objeto"/"atributo" debe configurarse por separado. 


## <a name="where-do-i-find-the-jmx-metrics-in-application-insights"></a>¿Dónde encuentro las métricas de JMX en Application Insights?

A medida que la aplicación se ejecuta y se recopilan las métricas de JMX, puede verlas yendo a Azure Portal y navegando a su recurso de Application Insights. En la pestaña Métricas, seleccione la lista desplegable que se muestra a continuación para ver las métricas.

![Captura de pantalla de las métricas en el portal](media/java-ipa/jmx/jmx-portal.png)
