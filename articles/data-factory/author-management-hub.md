---
title: Centro de administración
description: Administración de las conexiones, la configuración de control de código fuente y las propiedades de creación global en el centro de administración de Azure Data Factory
ms.service: data-factory
ms.topic: conceptual
author: minhe-msft
ms.author: hemin
ms.date: 04/27/2021
ms.openlocfilehash: db87372ab4dd56d8b6a00ca42cb1cf4a8729db69
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108178293"
---
# <a name="management-hub-in-azure-data-factory"></a>Centro de administración de Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

El centro de administración, al que se accede en la pestaña *Administrar* de la UX de Azure Data Factory, es un portal que hospeda acciones de administración globales para la factoría de datos. Aquí puede administrar las conexiones a almacenes de datos y procesos externos, la configuración de control de código fuente y la configuración de desencadenadores.

## <a name="manage-connections"></a>Administración de conexiones

### <a name="linked-services"></a>Servicios vinculados

Los servicios vinculados definen la información de conexión de Azure Data Factory para conectarse a almacenes de datos externos y entornos de proceso. Para más información, consulte los [conceptos sobre servicios vinculados](concepts-linked-services.md). La creación, edición y eliminación de servicios vinculados se realiza en el centro de administración.

![Administración de servicios vinculados](media/author-management-hub/management-hub-linked-services.png)

### <a name="integration-runtimes"></a>Entornos de ejecución de integración

Un entorno de ejecución de integración es la infraestructura de proceso que Azure Data Factory usa para proporcionar funcionalidades de integración de datos en distintos entornos de red. Para más información, consulte los [conceptos sobre entornos de ejecución de integración](concepts-integration-runtime.md). En el centro de administración, puede crear, eliminar y supervisar los entornos de ejecución de integración.

![Administración de entornos de ejecución de integración](media/author-management-hub/management-hub-integration-runtime.png)

## <a name="manage-source-control"></a>Administración del control de código fuente

### <a name="git-configuration"></a>Configuración de Git

Puede ver o editar toda la información relacionada con Git en los valores de configuración de Git en la central de administración. 

La última información de confirmación publicada también se muestra y puede ayudar a comprender la confirmación precisa, que se publicó o implementó por última vez en entornos. También puede ser útil cuando se realizan revisiones en producción.

Para más información, consulte [Control de código fuente en Azure Data Factory](source-control.md).

![Administración del repositorio de Git](media/author-management-hub/management-hub-git.png)

### <a name="parameterization-template"></a>Plantilla de parametrización

Para reemplazar los parámetros de la plantilla de Resource Manager generados al publicar desde la rama de colaboración, puede generar o editar un archivo de parámetros personalizado. Para más información, consulte [Uso de parámetros personalizados en la plantilla de Resource Manager](continuous-integration-deployment.md#use-custom-parameters-with-the-resource-manager-template). La plantilla de parametrización solo está disponible cuando se trabaja en un repositorio de Git. Si el archivo *arm-template-parameters-definition.json* no existe en la rama de trabajo, se generará al editar la plantilla predeterminada.

![Administración de los parámetros personalizados](media/author-management-hub/management-hub-custom-parameters.png)

## <a name="manage-authoring"></a>Administración de la creación

### <a name="triggers"></a>Desencadenadores

Los desencadenadores determinan cuándo se debe iniciar una ejecución de canalización. Actualmente, los desencadenadores pueden estar en una programación de reloj, operar en un intervalo periódico o depender de un evento. Para más información, consulte acerca de la [ejecución de desencadenadores](concepts-pipeline-execution-triggers.md#trigger-execution). En el centro de administración, puede crear, editar, eliminar o ver el estado actual de un desencadenador.

![Captura de pantalla que muestra dónde crear, editar, eliminar o ver el estado actual de un desencadenador.](media/author-management-hub/management-hub-triggers.png)

### <a name="global-parameters"></a>Parámetros globales

Los parámetros globales son constantes en una factoría de datos que una canalización puede consumir en cualquier expresión. Para obtener más información, consulte los [parámetros globales](author-global-parameters.md).

![Creación de parámetros globales](media/author-global-parameters/create-global-parameter-3.png)

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [configurar un repositorio de Git](source-control.md) para su ADF


