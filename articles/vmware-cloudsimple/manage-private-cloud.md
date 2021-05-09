---
title: Administrar una nube privada de Azure VMware Solution by CloudSimple
description: Describe las funcionalidades disponibles para administrar la actividad y los recursos de nube privada CloudSimple.
author: shortpatti
ms.author: v-patsho
ms.date: 06/10/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 204f259285578a8af697833e2c9b210c231d62ef
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108184395"
---
# <a name="manage-private-cloud-resources-and-activity"></a>Administrar la actividad y los recursos de la nube privada

Las nubes privadas se administran desde el portal de CloudSimple.  Compruebe el estado, los recursos disponibles, la actividad en la nube privada y otras opciones de configuración desde el portal de CloudSimple.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="access-the-cloudsimple-portal"></a>Acceso al portal de CloudSimple

Acceda al [portal de CloudSimple](access-cloudsimple-portal.md).

## <a name="view-the-list-of-private-clouds"></a>Ver la lista de nubes privadas

En la pestaña **Nubes privadas** en la página **Recursos**, se enumeran todas las nubes privadas en su suscripción. La información incluye el nombre, el número de clústeres de vSphere, la ubicación, el estado actual de la nube privada e información de los recursos.

![Página Nube privada](media/manage-private-cloud.png)

Seleccione una nube privada para acciones e información adicionales.

## <a name="private-cloud-summary"></a>Resumen de nube privada

Vea un resumen completo de la nube privada seleccionada.  La página Resumen incluye los servidores DNS que se han implementado en la nube privada.  Puede configurar el reenvío DNS desde los servidores DNS locales hacia los servidores DNS de la nube privada.  Para más información sobre el reenvío de DNS, consulte [Configure DNS for name resolution for Private Cloud vCenter from on-premises](./on-premises-dns-setup.md) (Configurar DNS para la resolución de nombres para la nube privada de vCenter local).

![Resumen de nube privada](media/private-cloud-summary.png)

### <a name="available-actions"></a>Acciones disponibles

* [Inicie el cliente de vSphere](./vcenter-access.md). Acceda a vCenter para esta nube privada.
* [Compre nodos](create-nodes.md). Agregue nodos a esta nube privada.
* [Expanda](expand-private-cloud.md). Agregue nodos a esta nube privada.
* **Actualice**. Actualice la información de esta página.
* **Eliminar**. Puede eliminar la nube privada en cualquier momento. **Antes de eliminar, asegúrese de haber hecho copias de seguridad de todos los sistemas y datos.** Al eliminar una nube privada, se eliminan todas las VM, la configuración de vCenter y los datos. Haga clic en **Eliminar** en la sección del resumen para la nube privada seleccionada. Después de su eliminación, todos los datos de la nube privada se borran en un proceso de eliminación seguro y altamente compatible.
* [Cambie los privilegios de vSphere](escalate-private-cloud-privileges.md).  Escale sus privilegios en esta nube privada.

## <a name="private-cloud-vlanssubnets"></a>VLAN y subredes de nubes privadas

Vea la lista de VLAN y subredes definidas para la nube privada seleccionada.  En la lista se incluye la administración de VLAN y subredes que se crearon cuando se creó la nube privada.

![VLAN y subredes de nubes privadas](media/private-cloud-vlans-subnets.png) 

### <a name="available-actions"></a>Acciones disponibles

* [Agregue las VLAN y subredes](./create-vlan-subnet.md). Agregue una VLAN o subconjunto a esta nube privada.

Seleccione una VLAN o subred para las acciones siguientes:
* [Adjuntar una tabla de firewall](./firewall.md). Adjunte una tabla de firewall a esta nube privada.
* **Edición**
* **Eliminar** (solo VLAN y subredes definidas por el usuario)

## <a name="private-cloud-activity"></a>Actividad de nube privada

Ver la siguiente información para la nube privada seleccionada.  La información de actividades es una lista filtrada de todas las actividades de la nube privada seleccionada.  Esta página muestra hasta 25 actividades recientes.

* Alertas recientes
* Eventos recientes
* Tareas recientes
* Auditoría reciente

![Actividad de nube privada](media/private-cloud-activity.png)

## <a name="cloud-racks"></a>Bastidores de nube

Los bastidores de nube son los componentes básicos de la nube privada. Cada bastidor proporciona una unidad de capacidad. CloudSimple configura automáticamente los bastidores de nube según las selecciones hechas al crear o expandir una nube privada.  Vea la lista completa de bastidores de nube, incluida la nube privada a la que está asignado cada uno.

![Bastidores de nube de nube privada](media/private-cloud-cloudracks.png)

## <a name="vsphere-management-network"></a>Red de administración de vSphere

Lista de máquinas virtuales y recursos de administración de VMware que están configurados actualmente en la nube privada. La información incluye la versión de software, el nombre de dominio completo (FQDN) y la dirección IP de los recursos.

![Red de administración de vSphere de nube privada](media/private-cloud-vsphere-management-network.png)

## <a name="next-steps"></a>Pasos siguientes

* [Uso de máquinas virtuales de VMware en Azure](quickstart-create-vmware-virtual-machine.md)
* Más información sobre las [nubes privadas](cloudsimple-private-cloud.md)
