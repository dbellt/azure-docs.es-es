---
title: Configuración de directivas de laboratorio en Azure DevTest Labs | Microsoft Docs
description: Obtenga información sobre cómo configurar una directiva de retención, limpiar la fábrica y retirar imágenes antiguas de DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 4e642d7dc0733e55caa4ed62e3382ef4422030f9
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105727542"
---
# <a name="set-up-retention-policy-in-azure-devtest-labs"></a>Configuración de la directiva de retención en Azure DevTest Labs
Este artículo abarca el establecimiento de una directiva de retención, la limpieza de la fábrica y la retirada de las imágenes anteriores de todos los otros laboratorios de DevTes Labs de la organización. 

## <a name="prerequisites"></a>Requisitos previos
Asegúrese de que ha seguidos los artículos que tratan los temas siguientes:

- [Creación de una fábrica de imágenes](image-factory-create.md)
- [Ejecución de una fábrica de imágenes desde AzureDevOps](image-factory-set-up-devops-lab.md)
- [Guardar imágenes personalizadas y distribuirlas a varios laboratorios](image-factory-save-distribute-custom-images.md)

Los siguientes elementos ya deben estar implementados:

- Un laboratorio para el generador de imágenes en Azure DevTest Labs.
- Una o varias instancias de Azure DevTest Labs donde el generador distribuirá las imágenes maestras.
- Un proyecto de Azure DevOps que se usa para automatizar el generador de imágenes.
- Ubicación del código fuente que contiene los scripts y la configuración (en nuestro ejemplo, en el mismo proyecto de DevOps usado anteriormente).
- Una definición de compilación para coordinar las tareas de Azure Powershell.
 
## <a name="setting-the-retention-policy"></a>Establecimiento de la directiva de retención
Antes de configurar los pasos de limpieza, defina cuántas imágenes históricas desea conservar en DevTest Labs. Si ha seguido el artículo sobre la [ejecución de un generador de imágenes desde Azure DevOps](image-factory-set-up-devops-lab.md), ha configurado varias variables de compilación. Una de ellas era **ImageRetention**. Esta variable se establece en `1`, lo que significa que DevTest Labs no conservará un historial de imágenes personalizadas. Solo las imágenes distribuidas más recientes estará disponibles. Si cambia esta variable a `2`, se conservarán la versión distribuida más reciente y las anteriores. Puede establecer este valor para definir el número de imágenes históricas que desea mantener en su instancia de DevTest Labs.

## <a name="cleaning-up-the-factory"></a>Limpieza de la fábrica
El primer paso para limpiar la fábrica es quitar las máquinas virtuales de la imagen maestra del generador de imágenes. Hay un script para realizar esta tarea al igual que nuestros scripts anteriores. El primer paso es agregar otra tarea de **Azure Powershell** a la definición de compilación tal como se muestra en la siguiente imagen:

![Paso de PowerShell](./media/set-retention-policy-cleanup/powershell-step.png)

Una vez que tenga la nueva tarea en la lista, seleccione el elemento y rellene todos los detalles, como se muestra en la siguiente imagen:

![Tarea de PowerShell de limpieza de imágenes anteriores](./media/set-retention-policy-cleanup/configure-powershell-task.png)

Los parámetros de script son: `-DevTestLabName $(devTestLabName)`.

## <a name="retire-old-images"></a>Retirada de imágenes anteriores 
Esta tarea elimina las imágenes anteriores, manteniendo solo un historial que coincide con la variable de compilación **ImageRetention**. Agregue otra tarea de compilación de **Azure Powershell** a nuestra definición de compilación. Una vez agregada, seleccione la tarea y rellene los detalles como se muestra en la siguiente imagen: 

![Tarea de PowerShell de retirada de imágenes anteriores](./media/set-retention-policy-cleanup/retire-old-image-task.png)

Los parámetros de script son: `-ConfigurationLocation $(System.DefaultWorkingDirectory)$(ConfigurationLocation) -SubscriptionId $(SubscriptionId) -DevTestLabName $(devTestLabName) -ImagesToSave $(ImageRetention)`.

## <a name="queue-the-build"></a>Puesta en cola de la compilación
Ahora que ha completado la definición de compilación, ponga en cola una nueva compilación para asegurarse de que todo funciona. Una vez que la compilación se complete correctamente, las nuevas imágenes personalizadas se muestran en el laboratorio de destino y, si consulta el laboratorio del generador de imágenes, no verá ninguna máquina virtual aprovisionada. Además, si pone en cola nuevas compilaciones, verá que las tareas de limpieza retiran antiguas imágenes personalizadas de los laboratorios de DevTest Labs de acuerdo con el valor de retención establecido en las variables de compilación.

> [!NOTE]
> Si ha ejecutado la canalización de compilación al final del último artículo de la serie, elimine manualmente las máquinas virtuales que se crearon en el laboratorio del generador de imágenes antes de poner en cola una compilación nueva.  El paso de limpieza manual solo es necesario mientras configuramos todo y comprobamos que funciona.



## <a name="summary"></a>Resumen
Ahora tiene un generador de imágenes en ejecución que puede crear y distribuir imágenes personalizadas para los laboratorios a petición. En este punto, lo que tiene que hacer es configurar adecuadamente sus imágenes e identificar los laboratorios de destino. Como se mencionó en el artículo anterior, el archivo **Labs.json** ubicado en la carpeta **Configuración** especifica las imágenes que deben estar disponibles en cada uno de los laboratorios de destino. Para sumar otros laboratorios de DevTest Labs a su organización, solo tiene que agregar una entrada en Labs.json para el nuevo laboratorio.

También es sencillo agregar una nueva imagen al generador. Cuando quiera incluir una nueva imagen en el generador, abra [Azure Portal](https://portal.azure.com), vaya al generador de DevTest Labs, seleccione el botón para agregar una máquina virtual y elija la imagen y los artefactos del Marketplace deseados. En lugar de seleccionar el botón **Crear** para la nueva VM, seleccione **Ver plantilla de Azure Resource Manager.** y guárdela como un archivo .json en algún lugar dentro de la carpeta **GoldenImages** en el repositorio. La próxima vez que ejecute el generador de imágenes, creará la imagen personalizada.


## <a name="next-steps"></a>Pasos siguientes
1. [Programe la compilación o versión](/azure/devops/pipelines/build/triggers?tabs=designer) para ejecutar periódicamente el generador de imágenes. Actualiza las imágenes creadas por el generador de forma periódica.
2. Cree más imágenes maestras para el generador. También puede considerar la [creación de artefactos](devtest-lab-artifact-author.md) para aplicar scripts a otras piezas de las tareas de configuración de la máquina virtual e incluir los artefactos en las imágenes del generador.
4. Cree [otra compilación/versión](/azure/devops/pipelines/overview) para ejecutar el script **DistributeImages** por separado. Puede ejecutar este script al realizar cambios en Labs.json y obtener las imágenes copiadas en los laboratorios de destino sin tener que volver a crear todas las imágenes de nuevo.

