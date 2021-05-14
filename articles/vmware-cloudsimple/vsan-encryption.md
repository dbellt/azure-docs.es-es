---
title: 'Azure VMware Solution by CloudSimple: configuración del cifrado de vSAN para una nube privada'
description: Describe cómo configurar la característica de cifrado de software de vSAN para que la nube privada de CloudSimple pueda trabajar con un servidor de administración de claves que se ejecuta en la red virtual de Azure.
author: shortpatti
ms.author: v-patsho
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: de2c3d889dd7f8bae4c50dee1c68d039e26e52d4
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108181083"
---
# <a name="configure-vsan-encryption-for-cloudsimple-private-cloud"></a>Configuración del cifrado de vSAN para la nube privada de CloudSimple

Puede configurar la característica de cifrado de software de vSAN para que la nube privada de CloudSimple pueda trabajar con un servidor de administración de claves que se ejecuta en la red virtual de Azure.

VMware requiere el uso de una herramienta externa de servidor de administración de claves (KMS) de terceros que cumple con el protocolo KMIP 1.1 cuando se usa el cifrado de vSAN. Puede aprovechar cualquier servidor de administración de claves compatible que esté certificado por VMware y que esté disponible para Azure.

En esta guía se describe cómo usar el servidor de administración de claves de HyTrust KeyControl que se ejecuta en una red virtual de Azure. Se puede usar un enfoque similar con cualquier otra solución de KMS de terceros certificada para vSAN.

Esta solución de KMS necesita que:

* Instale, configure y administre una herramienta de KMS de terceros con certificación de VMware en la red virtual de Azure.
* Proporcione sus propias licencias para la herramienta de KMS.
* Configure y administre el cifrado de vSAN en la nube privada mediante la herramienta de KMS de terceros que se ejecuta en la red virtual de Azure.

## <a name="kms-deployment-scenario"></a>Escenario de implementación de KMS

El clúster de servidores de KMS se ejecuta en la red virtual de Azure y es accesible mediante IP desde el vCenter de la nube privada a través de la conexión de Azure ExpressRoute configurada.

![Clúster de /media/KMS en una red virtual de Azure](media/vsan-kms-cluster.png)

## <a name="how-to-deploy-the-solution"></a>Implementación de la solución

El proceso de implementación consta de los siguientes pasos:

1. [Comprobación de que se cumplen los requisitos previos](#verify-prerequisites-are-met)
2. [Portal de CloudSimple: Obtención de información de emparejamiento de ExpressRoute](#cloudsimple-portal-obtain-expressroute-peering-information)
3. [Azure Portal: Conexión de la red virtual a la nube privada](#azure-portal-connect-your-virtual-network-to-your-private-cloud)
4. [Azure Portal: Implementación de un clúster de HyTrust KeyControl en la red virtual](#azure-portal-deploy-a-hytrust-keycontrol-cluster-in-the-azure-resource-manager-in-your-virtual-network)
5. [HyTrust WebUI: Configuración de servidores de KMIP](#hytrust-webui-configure-the-kmip-server)
6. [IU de vCenter: Configuración del cifrado de vSAN para que use el clúster de KMS en la red virtual de Azure](#vcenter-ui-configure-vsan-encryption-to-use-kms-cluster-in-your-azure-virtual-network)

### <a name="verify-prerequisites-are-met"></a>Comprobación de que se cumplen los requisitos previos

Compruebe lo siguiente antes de la implementación:

* El proveedor, la herramienta y la versión de KMS seleccionados se encuentran en la lista de compatibilidad de vSAN.
* El proveedor seleccionado admite que se ejecute una versión de la herramienta en Azure.
* La versión de Azure de la herramienta de KMS es compatible con KMIP 1.1.
* Ya se ha creado una instancia de Azure Resource Manager y una red virtual.
* Ya se ha creado una nube privada de CloudSimple.

### <a name="cloudsimple-portal-obtain-expressroute-peering-information"></a>Portal de CloudSimple: Obtención de información de emparejamiento de ExpressRoute

Para continuar con la instalación, necesita la clave de autorización y el identificador URI del circuito del mismo nivel para ExpressRoute además de acceso a su suscripción de Azure. Esta información está disponible en la página Virtual Network Connection en el portal de CloudSimple. Para obtener instrucciones, consulte [Configuración de una conexión de red virtual a la nube privada](virtual-network-connection.md). Si tiene algún problema para obtener la información, [abra una solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

### <a name="azure-portal-connect-your-virtual-network-to-your-private-cloud"></a>Azure Portal: Conexión de la red virtual a la nube privada

1. Cree una puerta de enlace de red virtual para la red virtual siguiendo las instrucciones que se encuentran en [Configuración de una puerta de enlace de red virtual para ExpressRoute con Azure Portal](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md).
2. Vincule la red virtual al circuito de ExpressRoute de CloudSimple siguiendo las instrucciones de [Conexión de una red virtual con un circuito de ExpressRoute mediante el portal](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md).
3. Use la información del circuito de ExpressRoute que recibió en el correo electrónico de bienvenida de CloudSimple para vincular la red virtual al circuito de ExpressRoute para CloudSimple en Azure.
4. Escriba la clave de autorización y el identificador URI de circuito del mismo nivel, asigne un nombre a la conexión y haga clic en **Aceptar**.

![Proporcionar el identificador URI del circuito del mismo nivel de CloudSimple al crear la red virtual](media/vsan-azureportal01.png) 

### <a name="azure-portal-deploy-a-hytrust-keycontrol-cluster-in-the-azure-resource-manager-in-your-virtual-network"></a>Azure Portal: Implementación de un clúster de HyTrust KeyControl en la instancia de Azure Resource Manager de la red virtual

Para implementar un clúster de HyTrust KeyControl en la instancia de Azure Resource Manager de la red virtual, realice las tareas siguientes. Consulte la [documentación de HyTrust](https://docs.hytrust.com/DataControl/Admin_Guide-4.0/Default.htm#OLH-Files/Azure.htm%3FTocPath%3DHyTrust%2520DataControl%2520and%2520Microsoft%2520Azure%7C_____0) para más información.

1. Cree un grupo de seguridad de red de Azure (nsg-hytrust) con las reglas de entrada especificadas siguiendo las instrucciones de la documentación de HyTrust.
2. Genere un par de claves SSH en Azure.
3. Implemente el nodo de KeyControl inicial de la imagen en Azure Marketplace.  Use la clave pública del par de claves que se generó y seleccione **nsg-hytrust** como el grupo de seguridad de red para el nodo de KeyControl.
4. Convierta la dirección IP privada de KeyControl en una dirección IP estática.
5. Conéctese mediante SSH a la máquina virtual de KeyControl mediante su dirección IP pública y la clave privada del par de claves mencionado anteriormente.
6. Cuando se le solicite en el shell de SSH, seleccione `No` para establecer el nodo como el nodo inicial de KeyControl.
7. Agregue nodos adicionales de KeyControl repitiendo los pasos 3-5 de este procedimiento y seleccionando `Yes` cuando se le pida que lo agregue a un clúster existente.

### <a name="hytrust-webui-configure-the-kmip-server"></a>HyTrust WebUI: Configuración de servidores de KMIP

Vaya a https://*public-ip*, donde *public-ip* es la dirección IP pública de la máquina virtual del nodo de KeyControl. Siga estos pasos de la [documentación de HyTrust](https://docs.hytrust.com/DataControl/Admin_Guide-4.0/Default.htm#OLH-Files/Azure.htm%3FTocPath%3DHyTrust%2520DataControl%2520and%2520Microsoft%2520Azure%7C_____0).

1. [Configuración de un servidor de KMIP](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/configuring-kmip-server.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____2)
2. [Creación de una agrupación de certificados para el cifrado de VMware](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-user-for-vmcrypt.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____3)

### <a name="vcenter-ui-configure-vsan-encryption-to-use-kms-cluster-in-your-azure-virtual-network"></a>IU de vCenter: Configuración del cifrado de vSAN para que use el clúster de KMS en la red virtual de Azure

Siga las instrucciones de HyTrust para [crear un clúster de KMS en vCenter](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-KMS-Cluster.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____4).

![Agregar detalles del clúster de KMS en vCenter](media/vsan-config01.png)

En vCenter, vaya a **Clúster > Configurar** y seleccione la opción **General** para vSAN. Habilite el cifrado y seleccione el clúster de KMS que se agregó previamente a vCenter.

![Habilitación del cifrado de vSAN y configuración del clúster de KMS en vCenter](media/vsan-config02.png)

## <a name="references"></a>Referencias

### <a name="azure"></a>Azure

[Configuración de una puerta de enlace de red virtual para ExpressRoute con Azure Portal](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md)

[Conexión de una red virtual con un circuito de ExpressRoute mediante el portal](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md)

### <a name="hytrust"></a>HyTrust

[HyTrust DataControl y Microsoft Azure](https://docs.hytrust.com/DataControl/Admin_Guide-4.0/Default.htm#OLH-Files/Azure.htm%3FTocPath%3DHyTrust%2520DataControl%2520and%2520Microsoft%2520Azure%7C_____0)

[Configuración de un servidor de KMIP](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/configuring-kmip-server.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____2)

[Creación de una agrupación de certificados para el cifrado de VMware](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-user-for-vmcrypt.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____3)

[Creación del clúster de KMS en vSphere](https://docs.hytrust.com/DataControl/4.2/Admin_Guide-4.2/index.htm#Books/VMware-vSphere-VSAN-Encryption/creating-KMS-Cluster.htm%3FTocPath%3DHyTrust%2520KeyControl%2520with%2520VSAN%25C2%25A0and%2520VMware%2520vSphere%2520VM%2520Encryption%7C_____4)
