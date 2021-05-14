---
title: Virtualization anidada en Azure IoT Edge para Linux en Windows | Microsoft Docs
description: Obtenga información sobre cómo navegar por la virtualización anidada en Azure IoT Edge para Linux en Windows.
author: fcabrera
manager: kgremban
ms.author: fcabrera
ms.date: 2/24/2021
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
monikerRange: =iotedge-2018-06
ms.openlocfilehash: 4f01362fd9342c1f508f165b34e121a11e8d07e2
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107496595"
---
# <a name="nested-virtualization-for-azure-iot-edge-for-linux-on-windows"></a>Virtualización anidada en Azure IoT Edge para Linux en Windows
Hay dos formas de hacer que la virtualización anidada sea compatible con Azure IoT Edge para Linux en Windows. Los usuarios pueden implementar una máquina virtual local o una máquina virtual de Azure. En este artículo se explica a los usuarios cuál es la opción más adecuada en su caso y se proporciona información sobre los requisitos de configuración.

> [!NOTE]
>
> Asegúrese de habilitar una [opción de red](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization#networking-options) para la virtualización anidada. De no hacerlo, se producirán errores de instalación de EFLOW. 

## <a name="deployment-on-local-vm"></a>Implementación en una máquina virtual local
Este es el enfoque de base de referencia para cualquier máquina virtual de Windows que hospede Azure IoT Edge para Linux en Windows. En este caso, la virtualización anidada deberá habilitarse antes de iniciar la implementación. Lea el artículo [Ejecución de Hyper-V en una máquina virtual con la virtualización anidada](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization) para obtener más información sobre cómo configurar este escenario.

Si usa Windows Server, asegúrese de [instalar el rol de Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/get-started/install-the-hyper-v-role-on-windows-server).

## <a name="deployment-on-azure-vms"></a>Implementación en máquinas virtuales de Azure
Si decide hacer la implementación en máquinas virtuales de Azure, tenga en cuenta que no hay ningún conmutador externo como parte del diseño. Azure IoT Edge para Linux en Windows tampoco es compatible con una máquina virtual de Azure que ejecuta la SKU del servidor, a menos que se utilice un script específico que obtiene un modificador predeterminado. Para más información sobre cómo obtener un modificador predeterminado, consulte la sección sobre [Windows Server](#windows-server) más adelante. 

> [!NOTE]
>
> Cualquier máquina virtual de Azure que supuestamente hospede EFLOW debe [admitir la virtualización anidada](../virtual-machines/acu.md).


## <a name="limited-use-of-external-switch"></a>Uso limitado del conmutador externo
En cualquier escenario en el que la máquina virtual no pueda obtener una dirección IP mediante un conmutador externo, la funcionalidad de implementación utiliza automáticamente el modificador predeterminado interno para la implementación. Esto significa que la administración de la máquina virtual de Azure IoT Edge para Linux solo se puede llevar a cabo en el propio dispositivo de destino (es decir, la conexión de la máquina virtual de Azure IoT Edge para Linux mediante la extensión WAC PowerShell SSH solo es posible en localhost).

## <a name="windows-server"></a>Windows Server
En el caso de los usuarios de Windows Server, tenga en cuenta que Azure IoT Edge para Linux en Windows no admite automáticamente el modificador predeterminado.

* Repercusiones en la máquina virtual local: asegúrese de que la máquina virtual de EFLOW puede obtener una dirección IP a través del conmutador externo.

* Repercusiones en la máquina virtual de Azure: dado que no hay ningún conmutador externo en las máquinas virtuales de Azure, no es posible implementar EFLOW antes de configurar un conmutador interno en el servidor.

No hay ningún conmutador predeterminado en las SKU de servidor (independientemente de si la SKU del servidor se ejecuta en una máquina virtual de Azure o no). Al hacer una implementación en una máquina virtual de Azure en la que no se pueda usar el conmutador externo, el conmutador predeterminado deberá instalarse y configurarse manualmente antes de la implementación de Azure IoT Edge para Linux en Windows. Nuestra funcionalidad de implementación abarca este escenario, ya que requiere la configuración de IP para el conmutador interno, una configuración de NAT, y la instalación y configuración de un servidor DHCP. Nuestra funcionalidad de implementación en la versión preliminar pública indica que no se juega con estos valores para no perjudicar las configuraciones de red en implantaciones productivas.

* Encontrará información de interés sobre la configuración manual del conmutador predeterminado en este artículo: [Procedimientos para habilitar la virtualización anidada en máquinas virtuales de Azure](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization)
* La documentación sobre cómo configurar un servidor DHCP para este escenario está disponible en: [Implementación de DHCP mediante Windows PowerShell](https://docs.microsoft.com/windows-server/networking/technologies/dhcp/dhcp-deploy-wps).
