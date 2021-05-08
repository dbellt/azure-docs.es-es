---
title: Conceptos sobre nubes privadas y clústeres
description: Obtenga información sobre las funcionalidades clave de los centros de datos definidos por software de Azure VMware Solution y los clústeres de vSphere.
ms.topic: conceptual
ms.date: 04/27/2021
ms.openlocfilehash: 71cd3c80f5ada0d8e21fbf1696896fe03368babe
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108161652"
---
#  <a name="azure-vmware-solution-private-cloud-and-cluster-concepts"></a>Conceptos de nubes privadas y clústeres de Azure VMware Solution

Azure VMware Solution ofrece nubes privadas basadas en VMware en Azure. Las implementaciones de software y hardware en nubes privadas de AVS están totalmente integradas y automatizadas en Azure. La nube privada se implementa y administra a través de Azure Portal, la CLI o PowerShell.  

Una nube privada incluye clústeres con:

- Nodos de servidor sin sistema operativo dedicados aprovisionados con el hipervisor de VMware ESXi 
- vCenter Server para administrar ESXi y vSAN 
- Redes definidas por software de NSX-T de VMware para máquinas virtuales de carga de trabajo de vSphere  
- Almacén de datos de vSAN de VMware para máquinas virtuales de carga de trabajo de vSphere  
- HCX de VMware para la movilidad de la carga de trabajo  
- Recursos del subyacente de Azure (necesarios para la conectividad y para operar la nube privada)

Al igual que con otros recursos, las nubes privadas se instalan y administran desde una suscripción a Azure. El número de nubes privadas dentro de una suscripción es escalable. Inicialmente, hay un límite de una nube privada por suscripción.  Existe una relación lógica entre las suscripciones a Azure, las nubes privadas de Azure VMware Solution, los clústeres de vSAN y los hosts. 

El diagrama muestra una suscripción a Azure única con dos nubes privadas que representan el entorno de desarrollo y producción. En cada una de esas nubes privadas hay dos clústeres. 

:::image type="content" source="media/hosts-clusters-private-clouds-final.png" alt-text="Imagen de dos nubes privadas en la suscripción de un cliente.":::

## <a name="hosts"></a>Hosts

[!INCLUDE [disk-capabilities-of-the-host](includes/disk-capabilities-of-the-host.md)]

## <a name="clusters"></a>Clústeres

[!INCLUDE [hosts-minimum-initial-deployment-statement](includes/hosts-minimum-initial-deployment-statement.md)]

[!INCLUDE [azure-vmware-solutions-limits](includes/azure-vmware-solutions-limits.md)]

## <a name="vmware-software-versions"></a>Versiones de software de VMware

[!INCLUDE [vmware-software-versions](includes/vmware-software-versions.md)]

## <a name="host-maintenance-and-lifecycle-management"></a>Mantenimiento y administración del ciclo de vida de hosts

Una de las ventajas de las nubes privadas de Azure VMware Solution es que la plataforma se mantiene automáticamente.  Microsoft es responsable de la administración del ciclo de vida del software de VMware (ESXi, vCenter y vSAN). Microsoft también se encarga de la administración del ciclo de vida de los dispositivos NSX-T y de la configuración de la red de arranque, como la creación de la puerta de enlace de nivel 0 y la habilitación del enrutamiento vertical de arriba abajo. Usted es responsable de la configuración de SDN de NSX-T, como los segmentos de red, reglas de firewall distribuidas, las puertas de enlace de nivel 1 y equilibradores de carga. 

[!INCLUDE [vmware-software-update-frequency](includes/vmware-software-update-frequency.md)]

## <a name="host-monitoring-and-remediation"></a>Supervisión y corrección de host

Azure VMware Solution supervisa continuamente el estado de los componentes subyacentes y de VMware. Cuando Azure VMware Solution detecta un error, toma medidas para reparar los componentes que no funcionan. Cuando Azure VMware Solution detecta una degradación o un error en un nodo de Azure VMware Solution, desencadena el proceso de corrección del host. 

La corrección del host implica reemplazar el nodo defectuoso por un nuevo nodo en buen estado en el clúster. A continuación, cuando sea posible, el host defectuoso se coloca en el modo de mantenimiento VMware vSphere. VMware vMotion traslada las máquinas virtuales fuera del host defectuoso a otros servidores disponibles en el clúster, lo que puede permitir la migración en vivo de las cargas de trabajo sin tiempo de inactividad. Si el host defectuoso no se puede poner en modo de mantenimiento, se quita del clúster.

Azure VMware Solution supervisa las siguientes condiciones en el host:  

- Estado del procesador 
- Estado de la memoria 
- Estado de conexión y energía 
- Estado del ventilador de hardware 
- Pérdida de conectividad de red 
- Estado de la placa del sistema de hardware 
- Errores en los discos de un host de vSAN 
- Voltaje de hardware 
- Estado de la temperatura de hardware 
- Estado de la energía de hardware 
- Estado de almacenamiento 
- Error de conexión 

> [!NOTE]
> Los administradores de inquilinos de Azure VMware Solution no deben editar ni eliminar las alarmas de VMware vCenter definidas anteriormente, ya que las administra el plano de control de Azure VMware Solution en vCenter. La funcionalidad de supervisión de Azure VMware Solution usa estas alarmas para desencadenar el proceso de corrección del host de Azure VMware Solution.

## <a name="backup-and-restoration"></a>Copia de seguridad y restauración

Las configuraciones de vCenter y NSX-T de la nube privada se encuentran en una programación de copia de seguridad.  Las copias de seguridad se conservan durante tres días. Si tiene que realizar una restauración a partir de una copia de seguridad, abra una [solicitud de soporte técnico](https://rc.portal.azure.com/#create/Microsoft.Support) en Azure Portal para solicitar la restauración.

Azure VMware Solution supervisa continuamente el estado de los componentes subyacentes y de VMware. Cuando Azure VMware Solution detecta un error, toma medidas para reparar los componentes que no funcionan.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha visto los conceptos de nube privada de Azure VMware Solution, puede que quiera obtener información sobre: 

- [Conceptos de interconectividad y redes de Azure VMware Solution](concepts-networking.md)
- [Conceptos de almacenamiento de Azure VMware Solution](concepts-storage.md)
- [Habilitación del recurso de Azure VMware Solution](enable-azure-vmware-solution.md)

<!-- LINKS - internal -->
[concepts-networking]: ./concepts-networking.md

<!-- LINKS - external-->
[VCSA versions]: https://kb.vmware.com/s/article/2143838
[ESXi versions]: https://kb.vmware.com/s/article/2143832
[vSAN versions]: https://kb.vmware.com/s/article/2150753

