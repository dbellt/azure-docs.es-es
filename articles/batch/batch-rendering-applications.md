---
title: Aplicaciones de representación
description: Todas las aplicaciones de representación se pueden usar con Azure Batch. Sin embargo, las imágenes de máquina virtual de Azure Marketplace están disponibles con aplicaciones comunes preinstaladas.
ms.date: 03/12/2021
ms.topic: how-to
ms.openlocfilehash: 7dee2806aafe34edee8dcb3dc3577def9c4c01da
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107987513"
---
# <a name="pre-installed-applications-on-batch-rendering-vm-images"></a>Aplicaciones preinstaladas en imágenes de máquina virtual de representación de Batch

Todas las aplicaciones de representación se pueden usar con Azure Batch. Sin embargo, las imágenes de máquina virtual de Azure Marketplace están disponibles con aplicaciones comunes preinstaladas.

Si procede, hay licencias de pago por uso disponibles para las aplicaciones de representación preinstaladas. Cuando se crea un grupo de Batch, se pueden especificar las aplicaciones necesarias y el costo tanto de la máquina virtual como de las aplicaciones se facturará por minuto. Los precios de las aplicaciones se muestran en la [página de precios de Azure Batch](https://azure.microsoft.com/pricing/details/batch/#graphic-rendering).

Algunas aplicaciones solo admiten Windows, pero la mayoría son compatibles con Windows y con Linux.

> [!IMPORTANT]
> Las imágenes de VM de representación y las licencias de pago por uso han [quedado en desuso y se retirarán el 29 de febrero de 2024](https://azure.microsoft.com/updates/azure-batch-rendering-vm-images-licensing-will-be-retired-on-29-february-2024/). Para usar Batch para la representación, [se debe usar una imagen de máquina virtual personalizada y una licencia de aplicación estándar.](batch-rendering-functionality.md#batch-pools-using-custom-vm-images-and-standard-application-licensing)

## <a name="applications-on-latest-centos-7-rendering-image"></a>Aplicaciones en la imagen de representación de CentOS 7 más reciente

La lista siguiente se aplica a la imagen de representación de CentOS, versión 1.2.0.

* Autodesk Maya I/O 2020 Actualización 4.6
* Autodesk Arnold para Maya 2020 (Arnold versión 6.2.0.0) MtoA-4.2.0-2020
* Chaos Group V-Ray para Maya 2020 (versión 5.00.21)
* Blender (2.80)
* AZ 10

## <a name="applications-on-latest-windows-server-rendering-image"></a>Aplicaciones en la imagen de representación más reciente de Windows Server

La lista siguiente se aplica a la imagen de representación de Windows Server, versión 1.5.0.

* Autodesk Maya I/O 2020 Actualización 4.4
* Autodesk 3ds Max I/O 2021 Update 3
* Autodesk Arnold para Maya 2020 (Arnold versión 6.1.0.1) MtoA-4.1.1.1-2020
* Autodesk Arnold para 3ds Max 2021 (versión de Arnold 6.1.0.1) MAXtoA-4.2.2.20-2021
* Chaos Group V-Ray para Maya 2020 (versión 5.00.21)
* Chaos Group V-Ray para 3ds Max 2021 (versión 5.00.05)
* Blender (2.79)
* Blender (2.80)
* AZ 10

> [!IMPORTANT]
> Para ejecutar V-Ray con Maya fuera de las [plantillas de extensión de Azure Batch](https://github.com/Azure/batch-extension-templates), inicie `vrayses.exe` antes de ejecutar la representación. Para iniciar el archivo vrayses.exe fuera de las plantillas, puede usar el siguiente comando `%MAYA_2020%\vray\bin\vrayses.exe"`.
>
> Para ver un ejemplo, consulte la tarea de inicio de la [plantilla de Maya y V-Ray](https://github.com/Azure/batch-extension-templates/blob/master/templates/maya/render-vray-windows/pool.template.json) en GitHub.

## <a name="applications-on-previous-windows-server-rendering-images"></a>Aplicaciones en imágenes de representación de Windows Server

La lista siguiente se aplica a las imágenes de representación de Windows Server 2016, versión 1.3.8.

* Autodesk Maya I/O 2017 Actualización 5 (versión 17.4.5459)
* Autodesk Maya I/O 2018 Update 6 (versión 18.4.0.7622)
* Autodesk Maya I/O 2019
* Autodesk 3ds Max I/O 2018 Actualización 4 (versión 20.4.0.4254)
* Autodesk 3ds Max I/O 2019 Actualización 1 (versión 21.2.0.2219)
* Autodesk 3ds Max I/O 2020 Update 2
* Autodesk Arnold para Maya 2017 (versión de Arnold 5.3.0.2) MtoA-3.2.0.2-2017
* Autodesk Arnold para Maya 2018 (versión de Arnold 5.3.0.2) MtoA-3.2.0.2-2018
* Autodesk Arnold para Maya 2019 (Arnold versión 5.3.0.2) MtoA-3.2.0.2-2019
* Autodesk Arnold para 3ds Max 2018 (versión de Arnold 5.3.0.2) (versión 1.2.926)
* Autodesk Arnold para 3ds Max 2019 (versión de Arnold 5.3.0.2) (versión 1.2.926)
* Autodesk Arnold para 3ds Max 2020 (versión de Arnold 5.3.0.2) (versión 1.2.926)
* Chaos Group V-Ray para Maya 2017 (versión 4.12.01)
* Chaos Group V-Ray para Maya 2018 (versión 4.12.01)
* Chaos Group V-Ray para Maya 2019 (versión 4.04.03)
* Chaos Group V-Ray para 3ds Max 2018 (versión 4.20.01)
* Chaos Group V-Ray para 3ds Max 2019 (versión 4.20.01)
* Chaos Group V-Ray para 3ds Max 2020 (versión 4.20.01)
* Blender (2.79)
* Blender (2.80)
* AZ 10

La lista siguiente se aplica a las imágenes de representación de Windows Server 2016, versión 1.3.7.

* Autodesk Maya I/O 2017 Actualización 5 (versión 17.4.5459)
* Autodesk Maya I/O 2018 Actualización 4 (versión 18.4.0.7622)
* Autodesk 3ds Max I/O 2019 Actualización 1 (versión 21.2.0.2219)
* Autodesk 3ds Max I/O 2018 Actualización 4 (versión 20.4.0.4254)
* Autodesk Arnold para Maya 2017 (Arnold versión 5.2.0.1) MtoA-3.1.0.1-2017
* Autodesk Arnold para Maya 2018 (Arnold versión 5.2.0.1) MtoA-3.1.0.1-2018
* Autodesk Arnold para 3ds Max 2018 (Arnold versión 5.0.2.4) (versión 1.2.926)
* Autodesk Arnold para 3ds Max 2019 (Arnold versión 5.0.2.4) (versión 1.2.926)
* Chaos Group V-Ray para Maya 2018 (versión 3.52.03)
* Chaos Group V-Ray para 3ds Max 2018 (versión 3.60.02)
* Chaos Group V-Ray para Maya 2019 (versión 3.52.03)
* Chaos Group V-Ray para 3ds Max 2019 (versión 4.10.01)
* Blender (2.79)

> [!NOTE]
> Chaos Group V-Ray para 3ds Max 2019 (versión 4.10.01) introduce cambios importantes en V-ray. Para usar la versión anterior (versión 3.60.02), use los nodos de representación de Windows Server 2016, versión 1.3.2.

## <a name="applications-on-previous-centos-rendering-images"></a>Aplicaciones en imágenes de representación de CentOS anteriores

La lista siguiente se aplica a las imágenes de representación de CentOS 7.6, versión 1.1.6.

* Autodesk Maya I/O 2017 Actualización 5 (versión 201708032230)
* Autodesk Maya I/O 2018 Actualización 2 (versión 201711281015)
* Autodesk Maya I/O 2019 Actualización 1
* Autodesk Arnold para Maya 2017 (Arnold versión 5.3.1.1) MtoA-3.2.1.1-2017
* Autodesk Arnold para Maya 2018 (Arnold versión 5.3.1.1) MtoA-3.2.1.1-2018
* Autodesk Arnold para Maya 2019 (Arnold versión 5.3.1.1) MtoA-3.2.1.1-2019
* Chaos Group V-Ray para Maya 2017 (versión 3.60.04)
* Chaos Group V-Ray para Maya 2018 (versión 3.60.04)
* Blender (2.68)
* Blender (2.8)

## <a name="next-steps"></a>Pasos siguientes

Para usar las imágenes de máquina virtual de representación, deben especificarse en la configuración del grupo cuando se crea; consulte las [funcionalidades del grupo de Batch para la representación](./batch-rendering-functionality.md).
