---
title: 'Azure VMware Solution by CloudSimple: creación de una máquina virtual en Azure con plantillas de máquina virtual'
description: Describe cómo crear máquinas virtuales en Azure mediante plantillas de máquina virtual en la infraestructura de VMware para la nube privada de CloudSimple.
author: shortpatti
ms.author: v-patsho
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: ad998c1def91a0e2a6ff74c25e4e1cc972df1474
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108182667"
---
# <a name="create-a-virtual-machine-in-azure-using-vm-templates-on-the-vmware-infrastructure"></a>Creación de una máquina virtual en Azure mediante plantillas de máquina virtual en la infraestructura de VMware

Puede crear una máquina virtual en Azure Portal mediante el uso de plantillas de máquina virtual en la infraestructura de VMware que el administrador de CloudSimple haya habilitado para su suscripción.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-cloudsimple-virtual-machine"></a>Creación de una máquina virtual de CloudSimple

1. Seleccione **Todos los servicios**.

2. Busque **CloudSimple Virtual Machines**.

3. Haga clic en **Agregar**.

    ![Creación de una máquina virtual de CloudSimple](media/create-cloudsimple-virtual-machine.png)

4. Escriba la información básica y haga clic en **Siguiente: Tamaño**.

    > [!NOTE]
    > La creación de máquinas virtuales de CloudSimple en Azure requiere una plantilla de máquina virtual.  Esta plantilla de máquina virtual debería existir en la instancia de vCenter de la nube privada.  Cree una máquina virtual en la nube privada desde la interfaz de usuario de vCenter con las configuraciones y el sistema operativo que desee.  Para crear una plantilla, siga las instrucciones de [Clonar una máquina virtual en una plantilla en vSphere Web Client](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-FE6DE4DF-FAD0-4BB0-A1FD-AFE9A40F4BFE_copy.html).

    ![Creación de una máquina virtual de CloudSimple: Conceptos básicos](media/create-cloudsimple-virtual-machine-basic-info.png)

    | Campo | Descripción |
    | ------------ | ------------- |
    | Subscription | Suscripción de Azure asociada a la nube privada.  |
    | Grupo de recursos | Grupo de recursos al que se va a asignar la máquina virtual. Puede seleccionar un grupo existente o crear uno. |
    | Nombre | Nombre para identificar la máquina virtual.  |
    | Location | Región de Azure en la que se hospeda la máquina virtual.  |
    | Nube privada | Nube privada de CloudSimple en la que quiere crear la máquina virtual. |
    | Grupo de recursos | Grupo de recursos asignado para la máquina virtual. Seleccione entre los grupos de recursos disponibles. |
    | Plantilla de vSphere | Plantilla de vSphere para la máquina virtual.  |
    | Nombre de usuario | Nombre de usuario del administrador de la máquina virtual (para plantillas de Windows)|
    | Contraseña <br>Confirmar contraseña | Contraseña del administrador de la máquina virtual (para plantillas de Windows).  |

5. Seleccione el número de núcleos y la capacidad de memoria de la máquina virtual y haga clic en **Siguiente: Configuraciones**. Active la casilla si quiere exponer la virtualización completa de la CPU al sistema operativo invitado para que las aplicaciones que requieren virtualización de hardware se puedan ejecutar en máquinas virtuales sin traducción binaria o paravirtualización. Para más información, consulte el artículo de VMware [Exponer la virtualización asistida por hardware de VMware](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-2A98801C-68E8-47AF-99ED-00C63E4857F6.html).

    ![Creación de una máquina virtual de CloudSimple: Tamaño](media/create-cloudsimple-virtual-machine-size.png)

6. Configure las interfaces de red y los discos como se describe en las tablas siguientes y haga clic en **Revisar y crear**.

    ![Creación de una máquina virtual de CloudSimple: Configuraciones](media/create-cloudsimple-virtual-machine-configurations.png)

    En las interfaces de red, haga clic en **Add network interface** (Agregar interfaz de red) y configure los siguientes valores.

    | Control | Descripción |
    | ------------ | ------------- |
    | Nombre | Escriba un nombre para identificar la interfaz.  |
    | Red | Seleccione en la lista de grupos de puertos distribuidos configurados de la instancia de vSphere de la nube privada.  |
    | Adapter (Adaptador) | Seleccione un adaptador de vSphere en la lista de tipos disponibles configurados para la máquina virtual. Para más información, consulte el artículo de la base de conocimiento de VMware [Elegir un adaptador de red para la máquina virtual](https://kb.vmware.com/s/article/1001805). |
    | Power on at Boot (Encender al arrancar) | Elija si quiere habilitar el hardware de NIC al arrancar la máquina virtual. El valor predeterminado es **Habilitar**. |

    En los discos, haga clic en **Add disk** (Agregar disco) y configure los siguientes valores.

    | Elemento | Descripción |
    | ------------ | ------------- |
    | Nombre | Escriba un nombre para identificar el disco.  |
    | Size | Seleccione uno de los tamaños disponibles.  |
    | Controladora SCSI | Seleccione una controladora SCSI para el disco.  |
    | Mode | Determina cómo participa el disco en las instantáneas. Elija una de estas opciones: <br> - Independent persistent (Independiente persistente): todos los datos escritos en el disco se escriben de forma permanente.<br> - Independent non-persistent (Independiente no persistente): los cambios que se escriben en el disco se descartan al apagar o restablecer la máquina virtual.  El modo independiente no persistente permite reiniciar siempre la máquina virtual en el mismo estado. Para más información, consulte la [documentación de VMware](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-8B6174E6-36A8-42DA-ACF7-0DA4D8C5B084.html).

7. Cuando finalice la validación, revise la configuración y haga clic en **Create** (Crear). Para realizar cambios, haga clic en las pestañas de la parte superior.

    ![Creación de una máquina virtual de CloudSimple: Revisión](media/create-cloudsimple-virtual-machine-review.png)

## <a name="view-list-of-cloudsimple-virtual-machines"></a>Visualización de la lista de máquinas virtuales de CloudSimple

1. Seleccione **Todos los servicios**.

2. Busque **CloudSimple Virtual Machines**.

3. Seleccione aquella en la que se creó la nube privada.

    ![Lista de máquinas virtuales de CloudSimple](media/list-cloudsimple-virtual-machines.png)

La lista de máquinas virtuales de CloudSimple incluye máquinas virtuales creadas a partir de Azure Portal.  Las máquinas virtuales creadas en la instancia de vCenter de la nube privada en el grupo de recursos asignado aparecerán en la lista.  
