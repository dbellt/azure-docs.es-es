---
title: 'ML Studio (clásico): Migración a Azure Machine Learning - Recompilación de experimentos'
description: Recompile los experimentos de Studio (clásico) en el diseñador de Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio-classic
ms.topic: how-to
author: xiaoharper
ms.author: zhanxia
ms.date: 03/08/2021
ms.openlocfilehash: 35a68d1a2883d0631f90cd8009e272bca0c66278
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107311845"
---
# <a name="rebuild-a-studio-classic-experiment-in-azure-machine-learning"></a>Recompilación de un experimento de Studio (clásico) en Azure Machine Learning

En este artículo, aprenderá a recompilar un experimento de Studio (clásico) en Azure Machine Learning. Para obtener más información sobre cómo migrar desde Studio (clásico), consulte el [artículo de información general sobre la migración](migrate-overview.md).

Los **experimentos** de Studio (clásico) son similares a las **canalizaciones** de Azure Machine Learning. Sin embargo, en Azure Machine Learning las canalizaciones se compilan en el mismo back-end que sustenta el SDK. Esto significa que tiene dos opciones para el desarrollo del aprendizaje automático: el diseñador de arrastrar y colocar o los SDK de código primero.

Para obtener más información sobre cómo compilar canalizaciones con el SDK, consulte [¿Qué son las canalizaciones de Azure Machine Learning](../concept-ml-pipelines.md#building-pipelines-with-the-python-sdk).


## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un área de trabajo de Azure Machine Learning. [Cree un área de trabajo de Azure Machine Learning](../how-to-manage-workspace.md#create-a-workspace).
- El experimento de Studio (clásico) que desea migrar.
- [Cargue el conjunto de datos](migrate-register-dataset.md) en Azure Machine Learning.

## <a name="rebuild-the-pipeline"></a>Recompilación de la canalización

Después de [migrar el conjunto de datos a Azure Machine Learning](migrate-register-dataset.md), ya puede volver a crear el experimento.

En Azure Machine Learning, el gráfico visual se denomina **borrador de la canalización**. En esta sección, volverá a crear el experimento clásico como un borrador de canalización.

1. Vaya a Estudio de Azure Machine Learning en [ml.azure.com](https://ml.azure.com).
1. En el panel de navegación izquierdo, seleccione **Diseñador** > **Módulos creados previamente fáciles de usar**. ![Captura de pantalla que muestra cómo crear un nuevo borrador de canalización.](../media/tutorial-designer-automobile-price-train-score/launch-designer.png)

1. Recompile manualmente el experimento con los módulos del diseñador.
    
    Consulte la [tabla de asignación de módulos](migrate-overview.md#studio-classic-and-designer-module-mapping) para localizar los módulos de reemplazo. Muchos de los módulos más populares de Studio (clásico) tienen versiones idénticas en el diseñador.

    > [!Important]
    > Si el experimento usa el módulo Execute R Script (Ejecutar script R), debe realizar pasos adicionales para migrar el experimento. Para obtener más información, consulte [Migración de módulos Execute R Script (Ejecutar script R) en Studio (clásico)](migrate-execute-r-script.md).

1. Ajuste los parámetros.
    
    Seleccione cada módulo y ajuste los parámetros en el panel de configuración del módulo a la derecha. Use los parámetros para volver a crear la funcionalidad del experimento de Studio (clásico). Para más información sobre cada módulo, consulte la página de [referencia de módulos](../algorithm-module-reference/module-reference.md).

## <a name="submit-a-run-and-check-results"></a>Envío de una ejecución y comprobación de los resultados

Después de volver a crear el experimento de Studio (clásico), es el momento de enviar una **ejecución de canalización**.

La ejecución de canalización se ejecuta en un **destino de proceso** conectado al área de trabajo. Puede establecer un destino de proceso predeterminado para toda la canalización o bien puede especificar destinos de proceso por módulo.

Una vez que se envía una ejecución desde el borrador de canalización, se convierte en una **ejecución de canalización**. Cada ejecución de canalización se graba y se registra en Azure Machine Learning.

Para establecer un destino de proceso predeterminado para toda la canalización, siga estos pasos:
1. Seleccione el **icono de engranaje** ![icono de engranaje en el diseñador](../media/tutorial-designer-automobile-price-train-score/gear-icon.png) junto al nombre de la canalización.
1. Elija **Seleccione un destino de proceso**.
1. Seleccione un proceso existente o bien cree un nuevo proceso siguiendo las instrucciones en pantalla.

Ahora que ya ha configurado el destino de proceso, puede enviar una ejecución de canalización:

1. En la parte superior del lienzo, seleccione **Enviar**.
1. Seleccione **Crear nuevo** para crear un nuevo experimento.
    
    Los experimentos organizan las ejecuciones de canalización similares juntas. Si ejecuta una canalización varias veces, puede seleccionar el mismo experimento para ejecuciones sucesivas. Esto resulta útil para el registro y el seguimiento.
1. Escriba un nombre para el experimento. Seleccione **Enviar**.

    La primera ejecución puede tardar hasta 20 minutos. Dado que la configuración del proceso predeterminada tiene un tamaño de nodo mínimo de 0, el diseñador debe asignar recursos después de estar inactivo. Las ejecuciones posteriores llevan menos tiempo, puesto que los nodos ya están asignados. Para acelerar el tiempo de ejecución, puede crear recursos de proceso con un tamaño mínimo de nodo de 1 o superior.

Una vez finalizada la ejecución, puede comprobar los resultados de cada módulo:

1. Haga clic con el botón derecho en el módulo cuya salida quiere ver.
1. Seleccione **Visualizar**, **View Output** (Ver salida) o **Ver registro**.

    - **Visualizar**: muestra una vista previa del conjunto de datos de resultado.
    - **View Output** (Ver salida): abre un vínculo a la ubicación de almacenamiento de salida. Use esta opción para explorar o descargar la salida. 
    - **Ver registro**: permite ver los registros del controlador y el sistema. Use **70_driver_log** para ver información relacionada con el script enviado por el usuario, como errores y excepciones.

> [!IMPORTANT]
> Los módulos del diseñador usan paquetes de Python de código abierto, a diferencia de Studio (clásico), que usa paquetes de C#. Esto hace que el resultado del módulo pueda variar ligeramente entre el diseñador y Studio (clásico). 


## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a recompilar un experimento de Studio (clásico) en Azure Machine Learning. El siguiente paso consiste en [recompilar los servicios web en Azure Machine Learning](migrate-rebuild-web-service.md).


Consulte los demás artículos de la serie de migración de Studio (clásico):

1. [Información general sobre la migración](migrate-overview.md).
1. [Migración de un conjunto de datos de Studio a Azure Machine Learning](migrate-register-dataset.md).
1. **Recompilación de una canalización de entrenamiento de Studio (clásico)**.
1. [Recompilación de un servicio web de Studio (clásico)](migrate-rebuild-web-service.md).
1. [Integración de un servicio web de Azure Machine Learning con aplicaciones cliente](migrate-rebuild-integrate-with-client-app.md).
1. [Migración de los módulos Execute R Script (Ejecutar script R) de Studio (Clásico)](migrate-execute-r-script.md).
