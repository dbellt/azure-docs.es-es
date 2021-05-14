---
title: Descargar un VHD de Windows desde Azure
description: Descargue un VHD de Windows mediante Azure Portal.
author: cynthn
manager: gwallace
ms.service: virtual-machines
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 01/13/2019
ms.author: cynthn
ms.openlocfilehash: 32b9753b79273ce747d00cba077dd8a5ee6d724d
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107565294"
---
# <a name="download-a-windows-vhd-from-azure"></a>Descargar un VHD de Windows desde Azure

En este artículo aprenderá a descargar un archivo de disco duro virtual (VHD) de Windows desde Azure mediante Azure Portal.

## <a name="optional-generalize-the-vm"></a>Opcional: Generalización de la máquina virtual

Si quiere usar el VHD como una [imagen](tutorial-custom-images.md) para crear otras máquinas virtuales, debe usar [Sysprep](/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation) para generalizar el sistema operativo. De lo contrario, tendrá que realizar una copia del disco para cada máquina virtual que quiera crear.

Para usar el VHD como imagen para crear otras máquinas virtuales, generalice la máquina virtual.

1. Si aún no lo ha hecho, inicie sesión en el [Azure Portal](https://portal.azure.com/).
2. [Conecte a la máquina virtual](connect-logon.md). 
3. En la máquina virtual, abra la ventana del símbolo del sistema como administrador.
4. Cambie el directorio a *%windir%\system32\sysprep* y ejecute sysprep.exe.
5. En el cuadro de diálogo Herramienta de preparación del sistema, seleccione **Iniciar la configuración rápida (OOBE) del sistema** y asegúrese de que **Generalizar** está activado.
6. En Opciones de apagado, seleccione **Apagar** y luego haga clic en **Aceptar**. 

Si no desea generalizar la máquina virtual actual, puede crear una imagen generalizada: cree primero [una instantánea del disco del sistema operativo,](#alternative-snapshot-the-vm-disk) después cree una nueva máquina virtual a partir de la instantánea y, a continuación, generalice la copia.

## <a name="stop-the-vm"></a>Parada de la máquina virtual

No se puede descargar un VHD desde Azure si está conectado a una máquina virtual en ejecución. Si quiere mantener la máquina virtual en ejecución, puede [crear una instantánea y, a continuación, descargarla](#alternative-snapshot-the-vm-disk).

1. En el menú Concentrador de Azure Portal, haga clic en **Máquinas virtuales**.
1. Seleccione la máquina virtual en la lista.
1. En la hoja de la máquina virtual, haga clic en **Detener**.

### <a name="alternative-snapshot-the-vm-disk"></a>Alternativa: Instantánea del disco de máquina virtual

Tome una instantánea del disco para descargarla.

1. Seleccione la máquina virtual en el [portal](https://portal.azure.com).
2. Seleccione **Discos** en el menú izquierdo y, a continuación, seleccione el disco del que quiera crear una instantánea. Se mostrarán los detalles del disco.  
3. En el menú de la parte superior de la página, seleccione **Crear instantánea**. Se abrirá la página **Crear instantánea**.
4. En **Nombre**, escriba un nombre para la instantánea. 
5. En **Tipo de instantánea**, seleccione **Completo** o **Incremental**.
6. Cuando haya terminado, seleccione **Revisar y crear**.

La instantánea se creará en breve y, a continuación, se puede usar para descargar o crear otra máquina virtual.

> [!NOTE]
> Si no detiene primero la máquina virtual, la instantánea no estará limpia. La instantánea estará en el mismo estado que si la máquina virtual se hubiera encendido o se bloqueara en el momento en que se realizó la instantánea.  Aunque normalmente es seguro, esto podría causar problemas si las aplicaciones que se ejecutan a la hora no son resistentes a bloqueos.
>  
> Este método solo se recomienda para las máquinas virtuales con un único disco del sistema operativo. Las máquinas virtuales con uno o varios discos de datos deben detenerse antes de la descarga o antes de crear una instantánea del disco del sistema operativo y de cada disco de datos.

## <a name="generate-download-url"></a>Generación de la dirección URL de descarga

Para descargar el archivo de VHD, debe generar una dirección URL de [firma de acceso compartido (SAS)](../../storage/common/storage-sas-overview.md?toc=/azure/virtual-machines/windows/toc.json). Una vez generada la dirección URL, se asigna un tiempo de expiración a la dirección URL.

1. En la página de la máquina virtual, haga clic en **Discos** en el menú de la izquierda.
1. Seleccione el disco de sistema operativo de la máquina.
1. En la página del disco, seleccione **Exportación del disco** en el menú de la izquierda.
1. El tiempo de expiración predeterminado de la dirección URL es de *3600* segundos (una hora). Es posible que tenga que aumentar este valor para discos del sistema operativo Windows o discos de datos más grandes. Normalmente, **36 000** segundos (10 horas) son suficientes.
1. Haga clic en **Generar dirección URL**.

> [!NOTE]
> El tiempo de expiración se aumenta con respecto al valor predeterminado para proporcionar tiempo suficiente para descargar el archivo de VHD grande de un sistema operativo Windows Server. Los discos duros virtuales grandes pueden tardar varias horas en descargarse según la conexión y el tamaño de la máquina virtual. 
> 
> 

## <a name="download-vhd"></a>Descarga de VHD

1. En la dirección URL generada, haga clic en Descargar el archivo VHD.
1. Es posible que tenga que hacer clic en **Guardar** en el explorador para iniciar la descarga. El nombre predeterminado del archivo de VHD es *abcd*.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre cómo [cargar un archivo de VHD en Azure](upload-generalized-managed.md). 
- [Create managed disks from unmanaged disks in a storage account](attach-disk-ps.md) (Creación de discos administrados a partir de discos no administrados en una cuenta de almacenamiento).
- [Manage Azure disks with PowerShell](tutorial-manage-data-disk.md) (Administrar discos de Azure con PowerShell).
