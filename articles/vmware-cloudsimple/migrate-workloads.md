---
title: 'Azure VMware Solution by CloudSimple: migración de máquinas virtuales con carga de trabajo a nubes privadas'
description: Describe cómo migrar máquinas virtuales desde la versión local de vCenter a vCenter en la nube privada de CloudSimple
author: shortpatti
ms.author: v-patsho
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: d95257d0b3c031747ca7ff4f359398938f9bde61
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108181929"
---
# <a name="migrate-workload-vms-from-on-premises-vcenter-to-private-cloud-vcenter-environment"></a>Migración de máquinas virtuales con carga de trabajo desde la versión local de vCenter a un entorno de vCenter en una nube privada

Para migrar máquinas virtuales desde un centro de centros local a una nube privada de CloudSimple, hay varias opciones disponibles.  La nube privada proporciona acceso nativo a VMware vCenter y las herramientas compatibles con VMware se pueden usar para la migración de cargas de trabajo. En este artículo se describen algunas de las opciones de migración de vCenter.

## <a name="prerequisites"></a>Requisitos previos

La migración de máquinas virtuales y datos desde un centro de datos local requiere conectividad de red desde el centro de datos a un entorno de nube privada.  Use cualquiera de estos métodos para establecer la conectividad de red:

* [Conexión VPN de sitio a sitio](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway) entre el entorno local y la nube privada.
* Conexión ExpressRoute Global Reach entre el circuito de ExpressRoute local y un circuito ExpressRoute de CloudSimple.

La ruta de acceso de red desde el entorno de la versión local de vCenter a la nube privada debe estar disponible para la migración de máquinas virtuales mediante vMotion.  La red vMotion de la versión local de vCenter debe tener capacidades de enrutamiento.  Compruebe que el firewall permite todo el tráfico de vMotion entre la versión local de vCenter y vCenter de la nube privada (en la nube privada, el enrutamiento en la red vMotion está configurado de forma predeterminada).

## <a name="migrate-isos-and-templates"></a>Migración de archivos ISO y plantillas

Para crear máquinas virtuales en una nube privada, use archivos ISO y plantillas de máquina virtual.  Para cargar ambos en el vCenter de la nube privada y que estén disponibles, use el siguiente método.

1. Cargue el archivo ISO en el vCenter de la nube privada desde la interfaz de usuario de vCenter.
2. [Publique una biblioteca de contenido](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-2A0F1C13-7336-45CE-B211-610D39A6E1F4.html) en el vCenter de la nube privada:

    1. Publique su biblioteca de contenido local.
    2. Cree una biblioteca de contenido en el vCenter de la nube privada.
    3. Suscríbase a la biblioteca de contenido local publicada.
    4. Sincronice la biblioteca de contenido para acceder al contenido suscrito.

## <a name="migrate-vms-using-powercli"></a>Migración de máquinas virtuales mediante PowerCLI

Para migrar máquinas virtuales desde la versión local de vCenter al vCenter de la nube privada, use VMware PowerCLI o la utilidad para la migración de cargas de trabajo entre vCenter disponible en VMware Labs.  En el siguiente script de ejemplo se muestran los comandos de migración de PowerCLI.

```
$sourceVC = Connect-VIServer -Server <source-vCenter name> -User <source-vCenter user name> -Password <source-vCenter user password>
$targetVC = Connect-VIServer -Server <target-vCenter name> -User <target-vCenter user name> -Password <target-vCenter user password>
$vmhost = <name of ESXi host on destination>
$vm = Get-VM -Server $sourceVC <name of VM>
Move-VM -VM $vm -VMotionPriority High -Destination (Get-VMhost -Server $targetVC -Name $vmhost) -Datastore (Get-Datastore -Server $targetVC -Name <name of tgt vc datastore>)
```

> [!NOTE]
> Para usar los nombres de los hosts de ESXi y del servidor vCenter de destino, configure el reenvío de DNS desde la ubicación local a la nube privada.

## <a name="migrate-vms-using-nsx-layer-2-vpn"></a>Migración de máquinas virtuales una VPN de nivel 2 de NSX

Esta opción permite la migración en vivo de cargas de trabajo desde un entorno de VMware local a la nube privada de Azure.  Con esta red de capa 2 extendida, la subred del entorno local estará disponible en la nube privada.  Después de la migración, no es preciso asignar nuevas direcciones IP a las máquinas virtuales.

En [Migración de cargas de trabajo mediante redes extendidas de nivel 2](migration-layer-2-vpn.md) se describe cómo usar una VPN de nivel 2 para extender una red de nivel 2 desde un entorno local a una nube privada.

## <a name="migrate-vms-using-backup-and-disaster-recovery-tools"></a>Migración de máquinas virtuales mediante herramientas de copia de seguridad y de recuperación ante desastres

La migración de máquinas virtuales a una nube privada se puede realizar mediante herramientas tanto de copia de seguridad y restauración como de recuperación ante desastres.  Use la nube privada como destino para la restauración de copias de seguridad creadas mediante herramientas de terceros.  La nube privada también se puede usar como destino para la recuperación ante desastres mediante VMware SRM o una herramienta de terceros.

Para más información sobre el uso de estas herramientas, consulte los siguientes temas:

* [Copia de seguridad de máquinas virtuales de carga de trabajo en la nube privada de CloudSimple mediante Veeam B&R](backup-workloads-veeam.md)
* [Configuración de la nube privada de CloudSimple como un sitio de recuperación ante desastres de las cargas de trabajo locales de VMware](disaster-recovery-zerto.md)
