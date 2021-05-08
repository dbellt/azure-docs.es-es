---
title: 'Azure VMware Solution by CloudSimple: creación de una nube privada de CloudSimple'
description: Describe cómo crear una nube privada de CloudSimple para ampliar las cargas de trabajo de VMware a la nube con continuidad y flexibilidad de funcionamiento.
author: shortpatti
ms.author: v-patsho
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 02cab9ee72dcd13b2ed8cd255ec4bb555b3894cc
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108177807"
---
# <a name="create-a-cloudsimple-private-cloud"></a>Crear una nube privada de CloudSimple

Una nube privada es una pila de VMware aislada que admite hosts ESXi, vCenter, vSAN y NSX. Las nubes privadas se administran desde el portal de CloudSimple. Tienen su propio servidor vCenter en su propio dominio de administración. La pila se ejecuta en nodos dedicados y nodos de hardware sin sistema operativo aislados.

Crear una nube privada le ayuda a satisfacer una variedad de necesidades comunes de infraestructura de red:

* **Crecimiento**. Si ha alcanzado un punto de actualización de hardware para la infraestructura existente, una nube privada le permite expandirse sin necesidad de ninguna inversión en hardware nuevo.

* **Rápida expansión**. Si surgen necesidades de capacidad temporales o no planeadas, una nube privada le permite crear la capacidad adicional sin demora.

* **Mayor protección**. Con una nube privada de tres nodos o más, obtiene protección automática de redundancia y alta disponibilidad.

* **Necesidades de infraestructura a largo plazo**. Si sus centros de datos trabajan a capacidad o quiere reestructurar para reducir los costos, una nube privada le permite retirar los centros de datos y migrar a una solución basada en la nube, a la vez que sigue siendo compatible con las operaciones empresariales.

Al crear una nube privada, obtendrá un clúster de vSphere único y todas las VM de administración que se crean en ese clúster.

## <a name="before-you-begin"></a>Antes de empezar

Los nodos deben aprovisionarse antes de poder crear su propia nube privada. Para más información sobre el aprovisionamiento de nodos, consulte el artículo [Aprovisionamiento de nodos para Azure VMware Solution by CloudSimple](create-nodes.md).

Asigne un intervalo de CIDR para las subredes vSphere/vSAN de la nube privada. Una nube privada se crea como un entorno de pila VMware aislada (con hosts ESXi, vCenter, vSAN y NSX) administrado por un servidor vCenter. Los componentes de administración se implementan en la red que se ha seleccionado para CIDR de subredes vSphere/vSAN. El intervalo de CIDR de red se divide en subredes diferentes durante la implementación. El espacio de direcciones de la subred vSphere/vSAN debe ser único. No se debe superponer con otras redes que se comuniquen con el entorno de CloudSimple. Las redes que se comunican con CloudSimple incluyen redes locales y redes virtuales de Azure. Para más información sobre las subredes vSphere/vSAN, consulte la introducción a las VLAN y subredes.

* Prefijo de intervalo de CIDR de subredes vSphere/vSAN mínimo: /24
* Prefijo de intervalo de CIDR de subredes vSphere/vSAN máximo: /21


## <a name="access-the-cloudsimple-portal"></a>Acceso al portal de CloudSimple

Acceda al [portal de CloudSimple](access-cloudsimple-portal.md).

## <a name="create-a-new-private-cloud"></a>Crear una nube privada

1. Seleccione **Todos los servicios**.
2. Busque **Servicios de CloudSimple**.
3. Seleccione el servicio de CloudSimple en el que quiere crear la nube privada.
4. En **Información general**, haga clic en **Crear nube privada** para abrir una nueva pestaña del explorador del portal de CloudSimple. Si se le solicita, inicie sesión con las credenciales de inicio de sesión en Azure.

    ![Creación de una nube privada desde Azure](media/create-private-cloud-from-azure.png)

5. En el portal de CloudSimple, proporcione un nombre para su nube privada.
6. Seleccione la **ubicación** para la nube privada.
7. Seleccione **Tipo de nodo** de acuerdo con lo que haya aprovisionado en Azure.
8. Especifique el **número de nodos**.  Para crear una nube privada, se requiere un mínimo de tres nodos.

    ![Creación de una nube privada: información básica](media/create-private-cloud-basic-info.png)

9. Haga clic en **Siguiente: Opciones avanzadas**.
10. Escriba el intervalo CIDR de subredes de vSphere/vSAN. Asegúrese de que el intervalo CIDR no se superponga con ninguna de las subredes locales u otras subredes de Azure (redes virtuales) ni con la subred de la puerta de enlace.

    **Opciones de intervalo de CIDR:** /24, /23, /22 o /21. Un intervalo de CIDR /24 admite hasta nueve nodos, un intervalo de CIDR /23 admite hasta 41 nodos y un intervalo de CIDR /22 y /21 admite hasta 64 nodos (el número máximo de nodos en una nube privada).

    > [!IMPORTANT]
    > Las direcciones IP en el intervalo CIDR de vSphere/vSAN están reservadas para que las use la infraestructura de nube privada.  No use la dirección IP de este intervalo en ninguna máquina virtual.

11. Haga clic en **Siguiente: Revisar y crear**.
12. Revise la configuración. Si necesita cambiar la configuración, haga clic en **Anterior**.
13. Haga clic en **Crear**.

Se inicia el proceso de aprovisionamiento de la nube privada. La nube privada puede tardar hasta dos horas en aprovisionarse.

Para obtener instrucciones sobre cómo expandir una nube privada existente, consulte [Expansión de una nube privada](expand-private-cloud.md).
