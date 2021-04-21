---
title: Planificación de la implementación de Azure VMware Solution
description: En este artículo se describe el flujo de trabajo de implementación de Azure VMware Solution.  El resultado final es un entorno listo para la creación y migración de máquinas virtuales (VM).
ms.topic: tutorial
ms.date: 03/17/2021
ms.openlocfilehash: 60e0a4083c0253d322b2e10472d0df7496722c10
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107251"
---
# <a name="planning-the-azure-vmware-solution-deployment"></a>Planificación de la implementación de Azure VMware Solution

En este artículo, se proporciona el proceso de planeamiento para identificar y recopilar la información que se usará durante la implementación. Cuando planee la implementación, asegúrese de documentar la información que recopila para facilitar la referencia durante la implementación.

Los pasos destacados de este inicio rápido proporcionan un entorno listo para producción para la creación de máquinas virtuales (VM) y la migración. 

Para realizar un seguimiento de los datos que va a recopilar, obtenga la lista [de comprobación del planeamiento de HCX](https://www.virtualworkloads.com/2021/04/hcx-planning-checklist/).

> [!IMPORTANT]
> Es importante solicitar una cuota de host al prepararse para crear el recurso Azure VMware Solution. Puede solicitar una cuota de host ahora, de forma que cuando finalice el proceso de planeamiento, estará listo para implementar la nube privada de Azure VMware Solution. Cuando el equipo de soporte técnico reciba su solicitud, tardará un máximo de cinco días laborables en confirmarla y asignar los hosts. Si tiene una nube privada de Azure VMware Solution y desea asignar más hosts, debe realizar el mismo proceso. Para más información, consulte los vínculos siguientes, en función del tipo de suscripción que tenga:
> - [Clientes de EA](enable-azure-vmware-solution.md?tabs=azure-portal#request-host-quota-for-ea-customers)
> - [Clientes de CSP](enable-azure-vmware-solution.md?tabs=azure-portal#request-host-quota-for-csp-customers)

## <a name="subscription"></a>Suscripción

Identifique la suscripción que piensa usar para implementar Azure VMware Solution.  Puede crear una nueva suscripción o reutilizar una existente.

>[!NOTE]
>La suscripción debe estar asociada a un plan de Contrato Enterprise de Microsoft o a un plan de Proveedor de soluciones en la nube de Azure. Para más información, consulte [Habilitación del recurso de Azure VMware Solution](enable-azure-vmware-solution.md).

## <a name="resource-group"></a>Resource group

Identifique el grupo de recursos que quiere usar para Azure VMware Solution.  Por lo general, se crea un grupo de recursos específicamente para Azure VMware Solution, pero puede usar un grupo de recursos existente.

## <a name="region"></a>Region

Identifique la región en la que desea implementar Azure VMware Solution.  Para más información, consulte la [Guía de productos Azure disponibles por región](https://azure.microsoft.com/en-us/global-infrastructure/services/?products=azure-vmware).

## <a name="resource-name"></a>Nombre del recurso

Defina el nombre del recurso que va a usar durante la implementación.  El nombre del recurso es un nombre descriptivo en el que se le da un título a la nube privada de Azure VMware Solution.

>[!IMPORTANT]
>El nombre no puede tener más de 40 caracteres. Si el nombre supera este límite, no podrá crear direcciones IP públicas para utilizarlas con la nube privada. 

## <a name="size-hosts"></a>Hosts de tamaño

Identifique los hosts de tamaño que quiere usar al implementar Azure VMware Solution.  Para obtener una lista completa, consulte la documentación de [nubes privadas y clústeres de Azure VMware Solution](concepts-private-clouds-clusters.md#hosts).

## <a name="number-of-clusters-and-hosts"></a>Número de clústeres y hosts

La primera implementación de Azure VMware Solution que realice constará de una nube privada que contendrá un único clúster. En su implementación, deberá definir el número de hosts que desea implementar en el primer clúster.

>[!NOTE]
>El número mínimo de hosts por clúster es tres y el máximo es 16. El número máximo de clústeres por nube privada es cuatro. 

Para más información, consulte la documentación de [nubes privadas y clústeres de Azure VMware Solution](concepts-private-clouds-clusters.md#clusters).

>[!TIP]
>Si necesita aumentar el número de implementación inicial, siempre puede extender el clúster y agregar clústeres adicionales más adelante.

## <a name="ip-address-segment-for-private-cloud-management"></a>Segmento de dirección IP para la administración de la nube privada

El primer paso para planificar la implementación es planificar la segmentación de IP. Azure VMware Solution requiere una red /22 CIDR. Este espacio de direcciones se divide en segmentos de red más pequeños (subredes) y se usa para los segmentos de administración de Azure VMware Solution, incluidas vCenter, VMware HCX, NSX-T y la funcionalidad vMotion. En la visualización siguiente se destaca dónde se va a utilizar este segmento.

Este bloque de direcciones de red CIDR /22 no debe superponerse con ningún segmento de red existente que ya tenga en el entorno local o en Azure.

**Ejemplo**: 10.0.0.0/22

Para obtener un desglose detallado del modo en que la red CIDR /22 se divide por [lista de comprobación de planeamiento de red](tutorial-network-checklist.md#routing-and-subnet-considerations)de nube privada.

:::image type="content" source="media/pre-deployment/management-vmotion-vsan-network-ip-diagram.png" alt-text="Identificación: segmento de dirección IP" border="false":::  

## <a name="ip-address-segment-for-virtual-machine-workloads"></a>Segmento de dirección IP para cargas de trabajo de máquina virtual

Al igual que en cualquier entorno de VMware, las máquinas virtuales deben conectarse a un segmento de red. En Azure VMware Solution, hay dos tipos de segmentos, segmentos extendidos L2 (que se describen más adelante) y segmentos de red NSX-T. Cuando la implementación de producción de Azure VMware Solution se expande, a menudo hay una combinación de segmentos extendidos de nivel 2 de segmentos de red locales y de NSX-T. Para planear la implementación inicial, en Azure VMware Solution, identifique un solo segmento de red (red IP). Esta red no debe superponerse con segmentos de red locales ni dentro del resto de Azure y no debe estar dentro del segmento de red /22 definido anteriormente.

Este segmento de red se usa principalmente con para realizar pruebas durante la implementación inicial.

>[!NOTE]
>Esta red o redes no serán necesarias durante la implementación. Se crean como un paso posterior a la implementación.
  
**Ejemplo**: 10.0.4.0/24

:::image type="content" source="media/pre-deployment/nsx-segment-diagram.png" alt-text="Identificación: segmento de dirección IP para cargas de trabajo de máquina virtual" border="false":::     

## <a name="attach-azure-virtual-network-to-azure-vmware-solution"></a>Conexión de Azure Virtual Network a Azure VMware Solution

Para proporcionar conectividad con Azure VMware Solution, se crea una instancia de ExpressRoute desde la nube privada de Azure VMware Solution a una puerta de enlace de red virtual de ExpressRoute.

Puede usar una puerta de enlace de red virtual de ExpressRoute *existente* o *nueva*.

:::image type="content" source="media/pre-deployment/azure-vmware-solution-expressroute-diagram.png" alt-text="Identificación: Azure Virtual Network para conectar Azure VMware Solution" border="false":::

### <a name="use-an-existing-expressroute-virtual-network-gateway"></a>Uso de una puerta de enlace de red virtual de ExpressRoute existente

Si planea usar una puerta de enlace de red virtual de ExpressRoute *existente*, el circuito de ExpressRoute de Azure VMware Solution se establece como un paso posterior a la implementación. En este caso, deje en blanco el campo **Red virtual**.

Como recomendación general, se permite el uso de una puerta de enlace de red virtual de ExpressRoute existente. En los planeamientos, tome nota de la puerta de enlace de red virtual de ExpressRoute que va a usar y vaya al [siguiente paso](#vmware-hcx-network-segments).

### <a name="create-a-new-expressroute-virtual-network-gateway"></a>Creación de una nueva puerta de enlace de red virtual de ExpressRoute

Al crear una *nueva* puerta de enlace de red virtual de ExpressRoute, puede usar una red virtual de Azure existente o crear una nueva.  

- En el caso de una red virtual de Azure existente:
   1. Identifique una red virtual de Azure en la que no haya puertas de enlace de red virtual de ExpressRoute preexistentes.
   2. Antes de realizar la implementación, cree un elemento [GatewaySubnet](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md#create-the-gateway-subnet) en Azure Virtual Network.

- En el caso de red virtual de Azure y una puerta de enlace de red virtual nuevas, lo creará durante la implementación, para lo que debe seleccionar el vínculo **Crear nueva** en la lista **Red virtual**.  Es importante definir el espacio de direcciones y las subredes antes de la implementación, con el fin de que pueda escribir esa información al completar los pasos de implementación.

En la imagen siguiente se muestra la pantalla de implementación **Create a private cloud** (Crear una nube privada) con el campo **Virtual Network** (Red virtual) resaltado.

:::image type="content" source="media/pre-deployment/azure-vmware-solution-deployment-screen-vnet-circle.png" alt-text="Captura de pantalla de la pantalla de implementación de Azure VMware Solution con el campo Red virtual resaltado.":::

> [!NOTE]
> El entorno local y Azure VMware Solution pueden ver cualquier red virtual que se vaya a usar o crear, por lo que debe asegurarse de que el segmento IP que use en esa red virtual y las subredes no se superpongan.

## <a name="vmware-hcx-network-segments"></a>Segmentos de red de VMware HCX

VMware HCX es una tecnología que se incluye en el conjunto de Azure VMware Solution. Los casos de uso principales de VMware HCX son las migraciones de cargas de trabajo y la recuperación ante desastres. Si tiene previsto realizar alguna de ellas, es mejor planificar las redes ahora. En otro caso, puede omitirla y continuar en el paso siguiente.

[!INCLUDE [hcx-network-segments](includes/hcx-network-segments.md)]

## <a name="optional-extend-your-networks"></a>(Opcional) Extensión de las redes

Los segmentos de red se pueden extender del entorno local a Azure VMware Solution. Si extiende segmentos de red, identifique las redes ahora.  

Estos son algunos de factores que se deben tener en cuenta:

- Si tiene previsto extender las redes del entorno local, esas redes deben conectarse a [vSphere Distributed Switch (vDS)](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-B15C6A13-797E-4BCB-B9D9-5CBC5A60C3A6.html) en el entorno local de VMware.  
- Las redes que se encuentran [en vSphere Standard Switch](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-350344DE-483A-42ED-B0E2-C811EE927D59.html) no se pueden extender.

>[!NOTE]
>Estas redes se extienden como paso final de la configuración, no durante la implementación.
>
## <a name="next-steps"></a>Pasos siguientes
Ahora que ha recopilado y documentado la información necesaria, continúe con la siguiente sección para crear la nube privada de Azure VMware Solution.

> [!div class="nextstepaction"]
> [Implementación de Azure VMware Solution](deploy-azure-vmware-solution.md)
> 
