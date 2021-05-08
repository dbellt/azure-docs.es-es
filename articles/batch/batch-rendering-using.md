---
title: Uso de las funcionalidades de representación
description: Forma de usar las funcionalidades de representación de Azure Batch. Pruebe a usar la aplicación Batch Explorer, ya sea directamente o invocada desde un complemento de aplicación cliente.
author: mscurrell
ms.author: markscu
ms.date: 03/12/2020
ms.topic: how-to
ms.openlocfilehash: df82f51377ac1a585dbfab989ca5386d33d90297
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107988971"
---
# <a name="using-azure-batch-rendering"></a>Uso de la representación de Azure Batch

> [!IMPORTANT]
> Las imágenes de VM de representación y las licencias de pago por uso han [quedado en desuso y se retirarán el 29 de febrero de 2024](https://azure.microsoft.com/updates/azure-batch-rendering-vm-images-licensing-will-be-retired-on-29-february-2024/). Para usar Batch para la representación, [se debe usar una imagen de máquina virtual personalizada y una licencia de aplicación estándar.](batch-rendering-functionality.md#batch-pools-using-custom-vm-images-and-standard-application-licensing)

Hay varias formas de usar la representación de Azure Batch:

* API:
  * Escriba código con cualquiera de las API de Batch.  Los desarrolladores pueden integrar funcionalidades de Azure Batch en sus aplicaciones o su flujo de trabajo existentes, independientemente de que se encuentre en la nube o de forma local.
* Herramientas de línea de comandos:
  * La [línea de comandos de Azure](/cli/azure/) o [PowerShell](/powershell/azure/) se pueden usar para crear un script de uso de Batch.
  * En concreto, la [compatibilidad con plantillas de la CLI de Batch](./batch-cli-templates.md) facilita considerablemente la creación de grupos y el envío de trabajos.
* Interfaz de usuario de Batch Explorer:
  * [Batch Explorer](https://github.com/Azure/BatchLabs) es una herramienta de cliente multiplataforma que también permite la supervisión y administración de cuentas de Batch.
  * Para cada una de las aplicaciones de representación, se proporcionan varias plantillas de grupos y trabajos que se pueden usar tanto para crear fácilmente grupos como para enviar trabajos.  En la interfaz de usuario de la aplicación aparece un conjunto de plantillas con los archivos de plantilla a los que se accede desde GitHub.
  * Las plantillas personalizadas se pueden crear desde cero, o bien se pueden copiar y modificar las plantillas proporcionadas en GitHub.
* Complementos de la aplicación cliente:
  * Hay complementos disponibles que permiten que la representación de Batch se use directamente desde las aplicaciones de modelado y diseño del cliente.  Los complementos principalmente invocan la aplicación Batch Explorer con información contextual acerca del modelo 3D actual e incluye características que ayudan a administrar recursos.

La mejor forma de probar la representación de Azure Batch y la manera más sencilla para los usuarios finales, que no son desarrolladores ni expertos de Azure, es usar la aplicación Batch Explorer, ya sea directamente o bien invocada desde un complemento de la aplicación cliente.

## <a name="using-batch-explorer"></a>Uso de Batch Explorer

Para ver un tutorial paso a paso para usar Batch Explorer para realizar la representación consulte el [tutorial de Blender](./tutorial-rendering-batchexplorer-blender.md).

### <a name="download-and-install"></a>Descarga e instalación

Las [descargas](https://azure.github.io/BatchExplorer/) de Batch Explorer están disponibles para Windows, OSX y Linux.

### <a name="using-templates-to-create-pools-and-run-jobs"></a>Uso de plantillas para crear grupos y ejecutar trabajos

Hay un completo conjunto de plantillas disponible para su uso con Batch Explorer que facilita la creación de grupos y el envío de trabajos para las distintas aplicaciones de representación sin tener que especificar todas las propiedades necesarias para crear grupos, trabajos y tareas directamente con Batch.  Las plantillas disponibles en Batch Explorer se almacenan en [un repositorio de GitHub](https://github.com/Azure/BatchExplorer-data/tree/master/ncj) y se pueden ver en él.

![Galería de Batch Explorer](./media/batch-rendering-using/batch-explorer-gallery.png)

Se proporcionan plantillas que satisfacen las necesidades de todas las aplicaciones presentes en las imágenes de máquina virtual de representación de Marketplace.  Para cada aplicación existen varias plantillas, entre las que se incluyen las plantillas de grupo, que satisfacen los grupos de CPU y GPU, o los grupos de Windows y Linux; las plantillas de trabajo incluyen un marco completo o la representación de Blender en iconos y la representación distribuida de V-Ray. El conjunto de plantillas proporcionadas se ampliará con el paso del tiempo para cumplir los requisitos de otras funcionalidades de Batch, como el escalado automático del grupo.

También es posible producir plantillas personalizadas desde cero o mediante la modificación de las plantillas proporcionadas. Para usar las plantillas personalizadas hay que seleccionar la opción "Plantillas locales" en la sección "Galería" de Batch Explorer.

### <a name="file-system-and-data-movement"></a>Sistema de archivos y movimiento de datos

La sección "Datos" de Batch Explorer permite que se copien archivos entre un sistema de archivos local y cuentas de Azure Storage.

## <a name="next-steps"></a>Pasos siguientes

Para ver ejemplos de representación de Batch consulte los dos tutoriales siguientes:

* [Representación mediante la CLI de Azure](./tutorial-rendering-cli.md)
* [Representación mediante Batch Explorer](./tutorial-rendering-batchexplorer-blender.md)
