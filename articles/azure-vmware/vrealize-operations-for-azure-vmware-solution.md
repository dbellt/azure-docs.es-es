---
title: Configuración de vRealize Operations para Azure VMware Solution
description: Aprenda a configurar vRealize Operations para una nube privada de Azure VMware Solution.
ms.topic: how-to
ms.date: 01/26/2021
ms.openlocfilehash: 127280d22e26a88b81016aeb53432dade45dc010
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108769316"
---
# <a name="configure-vrealize-operations-for-azure-vmware-solution"></a>Configuración de vRealize Operations para Azure VMware Solution


vRealize Operations Manager es una plataforma de administración de operaciones que permite a los administradores de infraestructura de VMware supervisar recursos del sistema. Estos recursos del sistema podrían ser objetos de nivel de aplicación o de nivel de infraestructura (ambos físicos y virtuales). La mayoría de los administradores de VMware han usado vRealize Operations para supervisar y administrar los componentes de nube privada de VMware: vCenter, ESXi, NSX-T, vSAN y VMware HCX.  Cada nube privada de Azure VMware Solution aprovisionada incluye una implementación dedicada de vCenter, NSX-T, vSAN y HCX. 

Revise primero detenidamente las secciones [Antes de comenzar](#before-you-begin) y [Requisitos previos](#prerequisites). Después, se le guiará a través de las dos topologías de implementación típicas:

> [!div class="checklist"]
> * [vRealize Operations locales que administran la implementación de Azure VMware Solution](#on-premises-vrealize-operations-managing-azure-vmware-solution-deployment)
> * [vRealize Operations que se ejecutan en la implementación de Azure VMware Solution](#vrealize-operations-running-on-azure-vmware-solution-deployment)

## <a name="before-you-begin"></a>Antes de empezar
* Revise la [documentación del producto vRealize Operations Manager](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) para obtener más información sobre la implementación de vRealize Operations. 
* Revise la [serie de tutoriales](tutorial-network-checklist.md) básicos de Software Defined Datacenter (SDDC) de Azure VMware Solution.
* Opcionalmente, revise la documentación del producto [vRealize Operations Remote Controller](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-263F9219-E801-4383-8A59-E84F3D01ED6B.html) para la opción de administración de vRealize Operations local de la implementación de Azure VMware Solution. 


## <a name="prerequisites"></a>Requisitos previos
* [vRealize Operations Manager](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) instalado.
* Una VPN o Azure ExpressRoute configurados entre el SDDC de Azure VMware Solution y el entorno local.
* Una nube privada de Azure VMware Solution se ha implementado en Azure.



## <a name="on-premises-vrealize-operations-managing-azure-vmware-solution-deployment"></a>vRealize Operations locales que administran la implementación de Azure VMware Solution
La mayoría de los clientes tienen una implementación local existente de vRealize Operations para administrar uno o varios dominios de vCenters locales. Cuando aprovisionan una nube privada de Azure VMware Solution, conectan su entorno local con su nube privada mediante Azure ExpressRoute o una solución de VPN de nivel 3.  

:::image type="content" source="media/vrealize-operations-manager/vrealize-operations-deployment-option-1.png" alt-text="vRealize Operations locales que administran la implementación de Azure VMware Solution" border="false":::

Para ampliar las funciones de vRealize Operations a la nube privada de Azure VMware Solution, cree una [instancia de adaptador para los recursos de la nube privada](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.config.doc/GUID-640AD750-301E-4D36-8293-1BFEB67E2600.html). Recopila datos de la nube privada de Azure VMware Solution y los transfiere a la instancia local de vRealize Operations. La instancia local de vRealize Operations Manager se puede conectar directamente a vCenter y NSX-T Manager en Azure VMware Solution. Opcionalmente, puede implementar un recopilador remoto de vRealize Operations en la nube privada de Azure VMware Solution. El recopilador comprime y cifra los datos recopilados de la nube privada antes de enviarlos a través de la red VPN o de ExpressRoute a la instancia de vRealize Operations Manager que se ejecuta en el entorno local. 

> [!TIP]
> Consulte la [documentación de VMware](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.vapp.doc/GUID-7FFC61A0-7562-465C-A0DC-46D092533984.html) y obtenga una guía paso a paso de cómo instalar vRealize Operations Manager. 



## <a name="vrealize-operations-running-on-azure-vmware-solution-deployment"></a>vRealize Operations que se ejecutan en la implementación de Azure VMware Solution

Otra opción consiste en implementar una instancia de vRealize Operations Manager en un clúster de vSphere de la nube privada. 

>[!IMPORTANT]
>Esta opción no es compatible actualmente con VMware.

:::image type="content" source="media/vrealize-operations-manager/vrealize-operations-deployment-option-2.png" alt-text="vRealize Operations que se ejecutan en Azure VMware Solution" border="false":::

Una vez que se ha implementado la instancia, puede configurar vRealize Operations para recopilar datos de vCenter, ESXi, NSX-T, vSAN y HCX. 



## <a name="known-limitations"></a>Limitaciones conocidas

- El usuario **cloudadmin\@vsphere.local** en Azure VMware Solution tiene [privilegios limitados](concepts-identity.md).  Las máquinas virtuales de Azure VMware Solution no admiten la recopilación de la memoria de invitado con las herramientas de VMware.  El uso de memoria activa y consumida continúa funcionando en este caso.
- La optimización de la carga de trabajo para la intención empresarial basada en host no funciona porque Azure VMware Solution administra configuraciones de clúster, incluida la configuración de DRS.
- La optimización de la carga de trabajo para la colocación entre clústeres dentro de SDDC mediante la intención empresarial basada en clúster es totalmente compatible con vRealize Operations Manager 8.0 y versiones posteriores. Pero la optimización de la carga de trabajo no tiene en cuenta los grupos de recursos y coloca las máquinas virtuales en el nivel de clúster. Un usuario puede corregirlo manualmente en la interfaz de vCenter Server de Azure VMware Solution.
- No se puede iniciar sesión en vRealize Operations Manager con las credenciales de vCenter Server de Azure VMware Solution. 
- Azure VMware Solution no admite el complemento vRealize Operations Manager.

Al conectar vCenter de Azure VMware Solution a vRealize Operations Manager mediante una cuenta de nube vCenter Server, verá una advertencia:

:::image type="content" source="./media/vrealize-operations-manager/warning-adapter-instance-creation-succeeded.png" alt-text="La instancia del adaptador de advertencia se creó correctamente":::

La advertencia se produce porque el usuario **cloudadmin\@vsphere.local** de Azure VMware Solution no tiene privilegios suficientes para realizar todas las acciones de vCenter Server necesarias para el registro. Sin embargo, los privilegios son suficientes para que la instancia de adaptador realice la recopilación de datos, como se muestra a continuación:

:::image type="content" source="./media/vrealize-operations-manager/adapter-instance-to-perform-data-collection.png" alt-text="Instancia de adaptador para realizar la recopilación de datos":::

Para obtener más información, consulte [Privilegios necesarios para configurar una instancia del adaptador de vCenter](https://docs.vmware.com/en/vRealize-Operations-Manager/8.1/com.vmware.vcom.core.doc/GUID-3BFFC92A-9902-4CF2-945E-EA453733B426.html).

<!-- LINKS - external -->


<!-- LINKS - internal -->




