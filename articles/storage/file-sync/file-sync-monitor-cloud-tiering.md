---
title: Supervisión de la nube por niveles de Azure File Sync | Microsoft Docs
description: Detalles de las métricas que se van a usar para supervisar las directivas de la nube por niveles.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 04/13/2021
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: cbdde16ff214077d7fe4814c8841d805fe05c37c
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107796255"
---
# <a name="monitor-cloud-tiering"></a>Supervisión de la nube por niveles
Existen dos maneras de supervisar la directiva de la nube por niveles: la hoja propiedades del punto de conexión de servidor y Azure Monitor.

## <a name="monitoring-via-server-endpoint"></a>Supervisión mediante el punto de conexión de servidor

Inicie sesión en [Azure Portal](https://portal.azure.com/), vaya a la hoja **Propiedades del punto de conexión de servidor** correspondiente al punto de conexión de servidor que quiere supervisar y desplácese hacia abajo hasta la sección de la nube por niveles. 

![Captura de pantalla de la sección de nube por niveles de las propiedades del punto de conexión de servidor.](media/storage-sync-monitoring-cloud-tiering/cloud-tiering-monitoring-5.png)

El **ahorro de espacio** es la cantidad de espacio que se ahorra al habilitar la nube por niveles. Si el tamaño del volumen local es lo suficientemente grande como para contener todos los datos, tendrá un ahorro del espacio del 0 %. Un ahorro de espacio del 0 % o un porcentaje bajo podría ser indicativo de que la nube por niveles no es beneficiosa con el tamaño actual de la caché local. 

La **frecuencia de aciertos de caché** es el porcentaje de archivos que se abren en la caché local. La frecuencia de aciertos de caché se calcula como los archivos abiertos directamente desde la caché/el número total de archivos abiertos. Una frecuencia de aciertos de caché del 100 % significa que todos los archivos abiertos en la última hora ya estaban en la caché local. Si su objetivo es reducir la salida, este resultado indicaría que la directiva actual funciona bien.

> [!NOTE]
> Las cargas de trabajo con patrones de acceso aleatorio suelen tener una menor frecuencia de aciertos de caché. 

El **tamaño total (nube)** es el tamaño de los archivos que se sincronizaron con la nube. 

El **tamaño almacenado en caché (servidor)** es el tamaño total de los archivos del servidor, descargados y almacenados por capas. A veces, el tamaño almacenado en caché puede ser mayor que el tamaño total de los archivos en la nube. Variables como el tamaño de clúster del volumen en el servidor pueden ser el motivo. Si el tamaño almacenado en caché es mayor de lo que se desea, considere la posibilidad de aumentar la directiva de espacio disponible del volumen. 

La **directiva de volumen efectiva** se usa en Azure File Sync para determinar la cantidad de espacio libre que se va a dejar en el volumen en el que se encuentra el punto de conexión de servidor. Cuando hay varios puntos de conexión de servidor en el mismo volumen, la directiva más alta de espacio disponible del volumen entre los puntos de conexión de servidor se convierte en la directiva de volumen efectiva para todos los puntos de conexión de servidor de ese volumen. Por ejemplo, si hay dos puntos de conexión de servidor en el mismo volumen, uno con el 30 % de espacio disponible en el volumen y otro con el 50 %, la directiva de espacio disponible del volumen efectiva para ambos puntos de conexión del servidor será la del 50 %.

El **espacio libre en el volumen actual** es el espacio libre en el volumen actualmente disponible en el servidor local. Si tiene una salida alta, pero hay más espacio libre en el volumen disponible antes de que se aplique la directiva de espacio disponible del volumen, considere la posibilidad de deshabilitar la directiva de fecha. Otro problema podría ser que los archivos se encuentren almacenados por niveles y que el archivo al que se accede con mayor frecuencia tenga un tamaño mayor que el resto del espacio libre en el volumen antes de que se aplique la directiva. En este caso, considere la posibilidad de aumentar el tamaño del volumen local. 

## <a name="monitoring-via-azure-monitor"></a>Supervisión con Azure Monitor

Vaya al **servicio de sincronización de almacenamiento** y seleccione **Métricas** en el panel de navegación lateral. 

Hay tres métricas diferentes que se pueden usar para supervisar la salida específica de la nube por niveles:

- Cloud Tiering Recall Size (Tamaño de recuperación de nube por niveles)
    - Es el tamaño total de los datos recuperados, en bytes, y se puede usar para identificar la cantidad de datos que se van a recuperar.
- Cloud Tiering Recall Size By Application (Tamaño de recuperación de nube por niveles por aplicación)
    - Es el tamaño total de los datos recuperados, en bytes, por una aplicación y se puede usar para identificar lo que recuperan los datos.
- Cloud Tiering Recall Throughput (Rendimiento de recuperación de la nube por niveles)
    - Mide la rapidez con la que se recuperan los datos, en bytes, y debe usarse si tiene dudas sobre el rendimiento. 

Para ser más específico sobre lo que quiere que muestren los grafos, considere la posibilidad de usar **Agregar filtro** y **Apply Splitting** (Aplicar división).
 
Para más información sobre los distintos tipos de métricas de Azure File Sync y cómo usarlos, consulte [Supervisión de Azure File Sync](file-sync-monitoring.md).

Para más información sobre el uso de métricas, consulte [Introducción al Explorador de métricas de Azure](../../azure-monitor/essentials/metrics-getting-started.md).

Si quiere cambiar la directiva de la nube por niveles, consulte [Elección de las directivas de nube por niveles](file-sync-choose-cloud-tiering-policies.md).

## <a name="next-steps"></a>Pasos siguientes

* [Planeamiento de una implementación de Azure File Sync](file-sync-planning.md)