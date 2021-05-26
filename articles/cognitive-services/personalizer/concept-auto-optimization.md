---
title: Optimización automática - Personalizer
description: En este artículo se proporciona información general conceptual de la característica de optimización automática del Personalizer de Azure.
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: conceptual
ms.date: 03/08/2021
ms.openlocfilehash: d2e8e73e47ebef566bed098e911b372aad6ee1e5
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382933"
---
# <a name="personalizer-auto-optimize-preview"></a>Optimización automática de Personalizer (versión preliminar)


## <a name="introduction"></a>Introducción
La optimización automática de Personalizer ahorra esfuerzo manual a la hora de mantener un rendimiento óptimo del aprendizaje automático de un bucle de Personalizer, ya que busca automáticamente una configuración de aprendizaje mejorada con el fin de usarla para entrenar sus modelos y aplicarlos. Personalizer tiene criterios estrictos para aplicar una nueva configuración de aprendizaje, con el fin de reducir las probabilidades de que las mejoras generen pérdidas en las recompensas.

La optimización automática de Personalizer está en versión preliminar pública, y sus características, enfoques y procesos cambiarán en función de los comentarios de los usuarios.

## <a name="when-to-use-auto-optimize"></a>Cuándo usar la optimización automática
En la mayoría de los casos, la mejor opción es tener activada la optimización automática. La optimización automática está *activada* de manera predeterminada para nuevos bucles de Personalizer.

La optimización automática puede ser de ayuda en las siguientes situaciones:
* Compila aplicaciones que usan muchos inquilinos, cada uno de los cuales obtiene sus propios bucles de Personalizer; por ejemplo, si hospeda varios sitios de comercio electrónico. Con la optimización automática puede ahorrarse el esfuerzo manual necesario para ajustar la configuración de aprendizaje para un gran número de bucles de Personalizer.
* Ha implementado Personalizer y validado que funciona bien tras obtener buenas recompensas, y se ha asegurado de que no haya errores ni problemas en sus características.

> [!NOTE]
> La optimización automática sobrescribirá periódicamente la configuración de aprendizaje de Personalizer. Si su caso de uso o sector requiere una auditoría y un archivo de modelos y configuraciones, o bien si necesita copias de seguridad de configuraciones anteriores, puede usar la API de Personalizer para recuperar configuraciones de aprendizaje o descargarlas a través de Azure Portal.

## <a name="how-to-enable-and-disable-auto-optimize"></a>Cómo habilitar y deshabilitar la optimización automática
Para habilitar la optimización automática, use el botón de alternancia de la hoja "Configuración de modelo y aprendizaje" de Azure Portal. 

También puede activarla mediante la API `/configurations/service` de Personalizer.

Para deshabilitar la optimización automática, desactive el botón de alternancia.

## <a name="auto-optimize-reports"></a>Informes de la optimización automática

En la hoja "Configuración de modelo y aprendizaje" puede ver el historial de ejecuciones de la optimización automática y la acción realizada en cada caso. 

En la tabla se muestra lo siguiente:
* Cuándo se ha ejecutado la optimización automática.
* Qué ventana de datos se ha incluido.
* Cuál ha sido el rendimiento en términos de recompensas de las configuraciones de aprendizaje en línea y de línea de base, así como de la mejor encontrada.
* Acciones realizadas: si se ha actualizado o no la configuración de aprendizaje.

El rendimiento en términos de recompensa de diferentes configuraciones de aprendizaje en cada fila del historial de optimización automática se muestra en números absolutos y como porcentajes relativos al rendimiento de línea de base. 

**Ejemplo**: si se calcula que su recompensa media de línea de base es de 0,20, y el comportamiento en línea de Personalizer obtiene 0,30, estos datos se mostrarán como 100 % y 150 % respectivamente. Si la optimización automática ha encontrado una configuración de aprendizaje capaz de lograr una recompensa media de 0,40, se mostrará como 200 % (0,40 es el 200 % de 0,20). Suponiendo que los márgenes de confianza lo permitan, se aplicaría la nueva configuración, que haría que Personalizer fuese la configuración en línea hasta la siguiente ejecución.

Se conserva un historial de hasta 24 ejecuciones anteriores de la optimización automática a efectos de análisis. Puede obtener más detalles sobre las evaluaciones sin conexión e informes con relación a cada una de ellas. Además, los informes contienen las configuraciones de aprendizaje que figuran en el historial, por lo que puede buscarlas para descargarlas o aplicarlas.

## <a name="how-it-works"></a>Cómo funciona
Personalizer entrena constantemente los modelos de IA que usa en función de las recompensas. Este entrenamiento se realiza siguiendo *parámetros de aprendizaje*, que contienen hiperparámetros y otros valores usados en el proceso de entrenamiento. Esta configuración de aprendizaje se puede "afinar" de acuerdo con la instancia específica de Personalizer. 

Personalizer también tiene la capacidad de realizar *evaluaciones sin conexión*. Las evaluaciones sin conexión analizan datos pasados y pueden generar un cálculo estadístico de la recompensa media que podrían haber obtenido los distintos algoritmos y modelos de Personalizer. Durante este proceso, Personalizer también buscará mejores configuraciones de aprendizaje, calculando su rendimiento (número de recompensas que habría obtenido) durante ese período de tiempo pasado.

#### <a name="auto-optimize-frequency"></a>Frecuencia de la optimización automática
La optimización automática se ejecutará periódicamente en función de datos pasados.
* Si su aplicación envía a Personalizer aproximadamente más de 20 MB de datos en las últimas dos semanas, usará las dos últimas semanas de datos.
* Si su aplicación envía un volumen inferior, Personalizer agregará datos de días anteriores hasta que haya un volumen suficiente para optimizar o hasta llegar a los primeros datos almacenados (hasta el número de días de retención de datos).

El servicio Personalizer determina las horas y los días exactos en los que se ejecuta la optimización automática, que fluctúan con el tiempo.

#### <a name="criteria-for-updating-learning-settings"></a>Criterios para actualizar la configuración de aprendizaje

Personalizer usa estos cálculos de recompensas para decidir si se debe cambiar la configuración de aprendizaje vigente por otra. Cada cálculo es una curva de distribución, con límites de confianza superior e inferior del 95 %. Personalizer solo aplicará una nueva configuración de aprendizaje si se cumplen las dos condiciones siguientes:
  * Ha mostrado recompensas medias más altas en el período de evaluación.
  * Tiene un límite inferior del intervalo de confianza del 95 % que es *superior* al límite inferior del intervalo de confianza del 95 % de la configuración de aprendizaje en línea.
Estos criterios diseñados para maximizar la mejora de las recompensas, a la vez que se intenta eliminar la probabilidad de que se pierdan otras futuras, se administran mediante Personalizer y se basan en los estudios sobre [algoritmos seldonianos](https://aisafety.cs.umass.edu/overview.html) y seguridad de la inteligencia artificial.

#### <a name="limitations-of-auto-optimize"></a>Limitaciones de la optimización automática

La optimización automática de Personalizer se basa en una evaluación de un período pasado para calcular el rendimiento futuro. Debido a factores externos en el mundo, a su aplicación y a sus usuarios, es posible que estos cálculos y predicciones que se realizan sobre modelos de Personalizer con relación al período anterior no sean representativos del futuro.

La versión preliminar de la optimización automática no está disponible para bucles de Personalizer que hayan habilitado la funcionalidad de la API de personalización de varias ranuras (versión preliminar). 

## <a name="next-steps"></a>Pasos siguientes

* [Evaluaciones sin conexión](concepts-offline-evaluation.md)
* [Directiva y configuración de aprendizaje](concept-active-learning.md)
* [Cómo analizar Personalizer con una evaluación sin conexión](how-to-offline-evaluation.md) 
* [Estudios sobre seguridad de la inteligencia artificial](https://aisafety.cs.umass.edu/overview.html) 
