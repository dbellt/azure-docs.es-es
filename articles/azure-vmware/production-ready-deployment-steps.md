---
title: Planificación de la implementación de Azure VMware Solution
description: En este artículo se describe el flujo de trabajo de implementación de Azure VMware Solution.  El resultado final es un entorno listo para la creación y migración de máquinas virtuales (VM).
ms.topic: tutorial
ms.custom: contperf-fy21q4
ms.date: 04/27/2021
ms.openlocfilehash: c9fba2009ec20a613ebf37adc2493145d7471711
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108145400"
---
# <a name="plan-the-azure-vmware-solution-deployment"></a>Planificación de la implementación de Azure VMware Solution

En este inicio rápido, se proporciona el proceso de planeamiento para identificar y recopilar la información que necesitará para la implementación. Cuando planee la implementación, asegúrese de documentar la información que recopila para facilitar la referencia durante la implementación.

Los pasos destacados proporcionan un entorno listo para producción para la creación de máquinas virtuales y la migración. 

>[!TIP]
>Para realizar un seguimiento de los datos que va a recopilar, utilice la [lista de comprobación del planeamiento de VMware HCX](https://www.virtualworkloads.com/2021/04/hcx-planning-checklist/).


## <a name="request-a-host-quota"></a>Solicitud de una cuota de host 

Es importante solicitar una cuota de host al prepararse para crear el recurso Azure VMware Solution. Puede solicitar una cuota de host ahora, de forma que cuando finalice el proceso de planeamiento, estará listo para implementar la nube privada de Azure VMware Solution. Cuando el equipo de soporte técnico reciba su solicitud, tardará un máximo de cinco días laborables en confirmarla y asignar los hosts. Si tiene una nube privada de Azure VMware Solution y desea asignar más hosts, debe realizar el mismo proceso. Para más información, consulte los vínculos siguientes, en función del tipo de suscripción que tenga:
- [Clientes de EA](enable-azure-vmware-solution.md?tabs=azure-portal#request-host-quota-for-ea-customers)
- [Clientes de CSP](enable-azure-vmware-solution.md?tabs=azure-portal#request-host-quota-for-csp-customers)

## <a name="identify-the-subscription"></a>Identificación de la suscripción

Identifique la suscripción que piensa usar para implementar Azure VMware Solution.  Puede crear una nueva suscripción o reutilizar una existente.

>[!NOTE]
>La suscripción debe estar asociada a un plan de Contrato Enterprise de Microsoft o a un plan de Proveedor de soluciones en la nube de Azure. Para más información, consulte [Habilitación del recurso de Azure VMware Solution](enable-azure-vmware-solution.md).

## <a name="identify-the-resource-group"></a>Identificación del grupo de recursos

Identifique el grupo de recursos que quiere usar para Azure VMware Solution.  Por lo general, se crea un grupo de recursos específicamente para Azure VMware Solution, pero puede usar un grupo de recursos existente.

## <a name="identify-the-region-or-location"></a>Identificación de la región o ubicación

Identifique la región en la que desea implementar Azure VMware Solution.  Para más información, consulte la [Guía de productos Azure disponibles por región](https://azure.microsoft.com/en-us/global-infrastructure/services/?products=azure-vmware).

## <a name="identify-the-resource-name"></a>Identificación del nombre del recurso

Defina el nombre del recurso que va a usar durante la implementación.  El nombre del recurso es un nombre descriptivo en el que se le da un título a la nube privada de Azure VMware Solution.

>[!IMPORTANT]
>El nombre no puede tener más de 40 caracteres. Si el nombre supera este límite, no podrá crear direcciones IP públicas para utilizarlas con la nube privada. 

## <a name="identify-the-size-hosts"></a>Identificación de los hosts de tamaño

Identifique los hosts de tamaño que quiere usar al implementar Azure VMware Solution.  

[!INCLUDE [disk-capabilities-of-the-host](includes/disk-capabilities-of-the-host.md)]

## <a name="determine-the-number-of-clusters-and-hosts"></a>Determinación del número de clústeres y hosts

La primera implementación de Azure VMware Solution que realice consta de una nube privada que contendrá un único clúster. En su implementación, deberá definir el número de hosts que desea implementar en el primer clúster.

[!INCLUDE [hosts-minimum-initial-deployment-statement](includes/hosts-minimum-initial-deployment-statement.md)]

## <a name="define-the-ip-address-segment-for-private-cloud-management"></a>Definición del segmento de dirección IP para la administración de la nube privada

El primer paso para planificar la implementación es planificar la segmentación de IP. Azure VMware Solution requiere una red /22 CIDR. Este espacio de direcciones se divide en segmentos de red más pequeños (subredes) y se usa para los segmentos de administración de Azure VMware Solution, incluidas vCenter, VMware HCX, NSX-T y la funcionalidad vMotion. En la visualización siguiente se destaca dónde se va a utilizar este segmento.

Este bloque de direcciones de red CIDR /22 no debe superponerse con ningún segmento de red existente que ya tenga en el entorno local o en Azure.

**Ejemplo**: 10.0.0.0/22

Para obtener un desglose detallado del modo en que la red CIDR /22 se divide por [lista de comprobación de planeamiento de red](tutorial-network-checklist.md#routing-and-subnet-considerations)de nube privada.

:::image type="content" source="media/pre-deployment/management-vmotion-vsan-network-ip-diagram.png" alt-text="Identificación: segmento de dirección IP" border="false":::  

## <a name="define-the-ip-address-segment-for-vm-workloads"></a>Definición del segmento de dirección IP para cargas de trabajo de máquina virtual

Al igual que en cualquier entorno de VMware, las máquinas virtuales deben conectarse a un segmento de red. En Azure VMware Solution, hay dos tipos de segmentos, segmentos extendidos L2 (que se describen más adelante) y segmentos de red NSX-T. Cuando la implementación de producción de Azure VMware Solution se expande, a menudo hay una combinación de segmentos extendidos de nivel 2 de segmentos de red locales y de NSX-T. Para planear la implementación inicial, en Azure VMware Solution, identifique un solo segmento de red (red IP). Esta red no debe superponerse con segmentos de red locales ni dentro del resto de Azure y no debe estar dentro del segmento de red /22 definido anteriormente.

Este segmento de red se usa principalmente con para realizar pruebas durante la implementación inicial.

>[!NOTE]
>Esta red o redes no serán necesarias durante la implementación. Se crean como un paso posterior a la implementación.
  
**Ejemplo**: 10.0.4.0/24

:::image type="content" source="media/pre-deployment/nsx-segment-diagram.png" alt-text="Identificación: segmento de dirección IP para cargas de trabajo de máquina virtual" border="false":::     

## <a name="define-the-virtual-network-gateway"></a>Definición de la puerta de enlace de red virtual

>[!IMPORTANT]
>Puede conectarse a una puerta de enlace de red virtual en un Azure Virtual WAN, pero está fuera del ámbito de este inicio rápido.

Una nube privada de Azure VMware Solution requiere una instancia de Azure Virtual Network y un circuito ExpressRoute.  

Defina si quiere usar una puerta de enlace de red virtual de ExpressRoute *existente* o *nueva*.  Si decide usar una puerta de enlace de red virtual *nueva*, podrá crearla después de crear la nube privada. Es posible usar una puerta de enlace de red virtual de ExpressRoute que ya existe y, para poder tener una planificación, tome nota de la puerta de enlace de red virtual de ExpressRoute que va a usar. 

:::image type="content" source="media/pre-deployment/azure-vmware-solution-expressroute-diagram.png" alt-text="Diagrama en el que se muestra el Azure Virtual Network asociado a Azure VMware Solution" border="false":::



## <a name="define-vmware-hcx-network-segments"></a>Definición de segmentos de red de VMware HCX

VMware HCX es una tecnología que se incluye en el conjunto de Azure VMware Solution. Los casos de uso principales de VMware HCX son las migraciones de cargas de trabajo y la recuperación ante desastres. Si tiene previsto realizar alguna de ellas, es mejor planificar las redes ahora. En otro caso, puede omitirla y continuar en el paso siguiente.

[!INCLUDE [hcx-network-segments](includes/hcx-network-segments.md)]

## <a name="determine-whether-to-extend-your-networks"></a>Determine si hay que extender las redes

De forma opcional, los segmentos de red se pueden extender del entorno local a Azure VMware Solution. 

Si extiende segmentos de red, identifique las redes ahora siguiendo estas pautas:

- Las redes deben conectarse a [vSphere Distributed Switch (vDS)](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-B15C6A13-797E-4BCB-B9D9-5CBC5A60C3A6.html) en el entorno local de VMware.  
- Las redes que se encuentran [en vSphere Standard Switch](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-350344DE-483A-42ED-B0E2-C811EE927D59.html) no se pueden extender.

>[!IMPORTANT]
>Estas redes se extienden como paso final de la configuración, no durante la implementación.


## <a name="next-steps"></a>Pasos siguientes
Ahora que ha recopilado y documentado la información necesaria, continúe con la siguiente sección para crear la nube privada de Azure VMware Solution.

> [!div class="nextstepaction"]
> [Implementación de Azure VMware Solution](deploy-azure-vmware-solution.md)
> 
