---
title: Flujo de trabajo y recursos del servicio Batch
description: Obtenga información sobre las características del servicio Batch y su flujo de trabajo general desde el punto de vista del desarrollo.
ms.topic: conceptual
ms.date: 06/11/2021
ms.openlocfilehash: c525b635bbda4284c876fa092c285e884a33db73
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112007002"
---
# <a name="batch-service-workflow-and-resources"></a>Flujo de trabajo y recursos del servicio Batch

En este artículo se proporciona información general sobre los componentes principales del servicio Azure Batch, se describe el flujo de trabajo general que pueden usar los desarrolladores de Batch para crear soluciones de proceso en paralelo a gran escala, junto con los recursos de servicio principal que se usan.

Usará muchos de los recursos y las características que se describen en este artículo, tanto si va a desarrollar una aplicación informática distribuida o un servicio que emita llamadas [API REST](/rest/api/batchservice/) directas, como si usa uno de los [SDK de Batch](batch-apis-tools.md#batch-service-apis).

> [!TIP]
> Para ver una introducción de nivel superior sobre el servicio Batch, consulte [¿Qué es Azure Batch?](batch-technical-overview.md). Consulte también la versión más reciente de las [actualizaciones del servicio Batch](https://azure.microsoft.com/updates/?product=batch).

## <a name="basic-workflow"></a>Flujo de trabajo básico

El siguiente flujo de trabajo de alto nivel es típico de casi todas las aplicaciones y los servicios que usan el servicio Batch para procesar cargas de trabajo paralelas:

1. Cargue los **archivos de datos** que quiera procesar en una cuenta de [Azure Storage](../storage/index.yml). Batch incluye compatibilidad integrada con el acceso a Azure Blob Storage y, cuando se ejecutan, las tareas pueden descargar estos archivos a [nodos de proceso](nodes-and-pools.md#nodes) .
2. Cargue los **archivos de la aplicación** que las tareas ejecutarán. Estos archivos pueden ser binarios o scripts y sus dependencias, y los ejecutan las tareas de los trabajos. Las tareas pueden descargar estos archivos desde la cuenta de Almacenamiento, o bien puede usar la característica de [paquetes de aplicación](nodes-and-pools.md#application-packages) en Batch para la implementación y la administración de aplicaciones.
3. Cree un [grupo](nodes-and-pools.md#pools) de nodos de proceso. Cuando cree un grupo, especifique el número de nodos de proceso para el grupo, su tamaño y el sistema operativo. Cuando se ejecutan las tareas de su trabajo, estas se asignan para que se ejecuten en uno de los nodos del grupo.
4. Creación de un [trabajo](jobs-and-tasks.md#jobs). Un trabajo administra una colección de tareas. Se asocia cada trabajo a un grupo específico donde se ejecutarán las tareas de dicho trabajo.
5. Agregue [tareas](jobs-and-tasks.md#tasks) al trabajo. Cada tarea ejecuta la aplicación o el script que haya cargado para procesar los archivos de datos que descarga de la cuenta de almacenamiento. A medida que se complete cada tarea, puede cargar su resultado a Azure Storage.
6. Supervise el progreso del trabajo y recupere el resultado de la tarea de Azure Storage.

> [!NOTE]
> Necesitará una [cuenta de Batch](accounts.md) para utilizar este servicio. La mayoría de las soluciones de Batch usan una cuenta de [Azure Storage](../storage/index.yml) asociada para el almacenamiento y la recuperación de archivos.

## <a name="batch-service-resources"></a>Recursos del servicio Batch

En los temas siguientes se describen los recursos de Batch que habilitan escenarios de cálculo distribuido.

- [Cuentas de Batch y de almacenamiento](accounts.md)
- [Nodos y grupos](nodes-and-pools.md)
- [Trabajos y tareas](jobs-and-tasks.md)
- [Archivos y directorios](files-and-directories.md)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información acerca de las [API y herramientas de Batch](batch-apis-tools.md) disponibles para la creación de soluciones de Batch.
- Para conocer los aspectos básicos del desarrollo de una aplicación habilitada para Batch, consulte la [biblioteca de cliente de Batch para .NET](quick-run-dotnet.md) o [Python](quick-run-python.md). Estas guías de inicio rápido le guían a través de una aplicación de ejemplo que usa el servicio de Batch para ejecutar una carga de trabajo en varios nodos de proceso e incluye el uso de Azure Storage para el almacenamiento provisional y la recuperación del archivo de la carga de trabajo.
- Descargue e instale la utilidad [Batch Explorer](https://azure.github.io/BatchExplorer/) para usarla durante el desarrollo de sus soluciones de Batch. Use Batch Explorer para crear, depurar y supervisar aplicaciones de Azure Batch.
- Vea los recursos de la comunidad entre los que se encuentran [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-batch), el [repositorio de la Comunidad de Batch](https://github.com/Azure/Batch) y el [foro de Azure Batch](/answers/topics/azure-batch.html).
