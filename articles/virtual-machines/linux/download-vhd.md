---
title: Descarga de un VHD de Linux desde Azure
description: Descarga de un VHD de Linux mediante la CLI de Azure y Azure Portal.
author: cynthn
ms.service: virtual-machines
ms.subservice: disks
ms.collection: linux
ms.topic: how-to
ms.date: 08/03/2020
ms.author: cynthn
ms.openlocfilehash: 8def06990b72d6e08127e8c4f16e0dfd87905d4f
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107565192"
---
# <a name="download-a-linux-vhd-from-azure"></a>Descarga de un VHD de Linux desde Azure

En este artículo aprenderá a descargar un archivo de disco duro virtual (VHD) de Linux desde Azure mediante Azure Portal. 

## <a name="stop-the-vm"></a>Parada de la máquina virtual

No se puede descargar un VHD desde Azure si está conectado a una máquina virtual en ejecución. Si quiere mantener la VM en ejecución, puede [crear una instantánea y, a continuación, descargarla](#alternative-snapshot-the-vm-disk).

Para parar la VM:

1.  Inicie sesión en [Azure Portal](https://portal.azure.com/).
2.  En el menú de la izquierda, seleccione **Máquinas virtuales**.
3.  Seleccione la máquina virtual en la lista.
4.  En la página de la máquina virtual, seleccione **Detener**.

    :::image type="content" source="./media/download-vhd/export-stop.PNG" alt-text="Muestra el botón de menú para detener la máquina virtual.":::

### <a name="alternative-snapshot-the-vm-disk"></a>Alternativa: Instantánea del disco de VM

Tome una instantánea del disco para descargarla.

1. Seleccione la VM en el [portal](https://portal.azure.com).
2. Seleccione **Discos** en el menú izquierdo y, a continuación, seleccione el disco del que quiera crear una instantánea. Se mostrarán los detalles del disco.  
3. En el menú de la parte superior de la página, seleccione **Crear instantánea**. Se abrirá la página **Crear instantánea**.
4. En **Nombre**, escriba un nombre para la instantánea. 
5. En **Tipo de instantánea**, seleccione **Completo** o **Incremental**.
6. Cuando haya terminado, seleccione **Revisar y crear**.

La instantánea se creará en breve y, a continuación, se puede usar para descargar o crear otra VM.

> [!NOTE]
> Si no detiene primero la VM, la instantánea no estará limpia. La instantánea estará en el mismo estado que si la VM se hubiera encendido o se bloqueara en el momento en que se realizó la instantánea.  Aunque normalmente es seguro, esto podría causar problemas si las aplicaciones que se ejecutan a la hora no son resistentes a bloqueos.
>  
> Este método solo se recomienda para las VM con un único disco del sistema operativo. Las VM con uno o varios discos de datos deben detenerse antes de la descarga o antes de crear una instantánea del disco del sistema operativo y de cada disco de datos.

## <a name="generate-sas-url"></a>Generación de dirección URL de SAS

Para descargar el archivo de VHD, debe generar una dirección URL de [firma de acceso compartido (SAS)](../../storage/common/storage-sas-overview.md?toc=/azure/virtual-machines/windows/toc.json). Una vez generada la dirección URL, se asigna un tiempo de expiración a la dirección URL.

1. En el menú de la página de la máquina virtual, seleccione **Discos**.
2. Seleccione el disco de sistema operativo de la máquina virtual y luego seleccione **Exportación del disco**.
1. Si es necesario, actualice el valor de **La dirección URL expira en (segundos)** para proporcionar suficiente tiempo para completar la descarga. El valor predeterminado es 3600 segundos (una hora).
3. Seleccione **Generar dirección URL**.
 
      
## <a name="download-vhd"></a>Descarga de VHD

1.  En la dirección URL generada, seleccione **Descargar el archivo VHD**.

    :::image type="content" source="./media/download-vhd/export-download.PNG" alt-text="Muestra el botón para descargar el VHD.":::

2.  Es posible que tenga que seleccionar **Guardar** en el explorador para iniciar la descarga. El nombre predeterminado del archivo de VHD es *abcd*.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [cargar y crear una máquina virtual de Linux desde un disco personalizado con la CLI de Azure](upload-vhd.md). 
- [Administración de discos de Azure con la CLI de Azure](tutorial-manage-disks.md).
