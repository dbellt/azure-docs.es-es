---
title: Conectar Azure ExpressRoute con Oracle Cloud Infrastructure | Microsoft Docs
description: Conectar Azure ExpressRoute con Oracle Cloud Infrastructure (OCI) FastConnect para habilitar soluciones de aplicaciones de Oracle en la nube
author: dbakevlar
ms.service: virtual-machines
ms.subservice: oracle
ms.collection: linux
ms.topic: article
ms.date: 03/16/2020
ms.author: rogardle
ms.openlocfilehash: 06f0eb1ef36e711c5297af936d704f442596fc1a
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107887583"
---
# <a name="set-up-a-direct-interconnection-between-azure-and-oracle-cloud-infrastructure"></a>Configurar una interconexión directa entre Azure y Oracle Cloud Infrastructure  

Para crear una [experiencia integrada de nube múltiple](oracle-oci-overview.md) , Microsoft y Oracle ofrecen interconexión directa entre Azure y Oracle Cloud Infrastructure (OCI) a través de [ExpressRoute](../../../expressroute/expressroute-introduction.md) y [FastConnect](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/fastconnectoverview.htm). A través de la interconexión de ExpressRoute y FastConnect, los clientes pueden experimentar una baja latencia, un alto rendimiento y una conectividad privada directa entre las dos nubes.

> [!IMPORTANT]
> Oracle certificará estas aplicaciones para que se ejecuten en Azure cuando se use la solución de interconexión en la nube de Azure/Oracle hasta mayo de 2020.
> * E-Business Suite
> * JD Edwards EnterpriseOne
> * PeopleSoft
> * Aplicaciones comerciales de Oracle
> * Oracle Hyperion Financial Management

En la siguiente imagen se muestra una descripción general de alto nivel de la interconexión:

![Conexión de red entre nubes](https://user-images.githubusercontent.com/37556655/115093592-bced0180-9ecf-11eb-976d-9d4c7a1be2a8.png)

## <a name="prerequisites"></a>Prerrequisitos

* Para establecer la conectividad entre Azure y OCI, debe tener una suscripción activa de Azure y una tenencia activa de OCI.

* Esta conectividad solo es posible cuando una ubicación de emparejamiento de Azure ExpressRoute está cerca o en la misma ubicación de emparejamiento que FastConnect de OCI. Consulte la [Disponibilidad regional](oracle-oci-overview.md#region-availability).

## <a name="configure-direct-connectivity-between-expressroute-and-fastconnect"></a>Configurar la conectividad directa entre ExpressRoute y FastConnect

1. Cree un circuito estándar de ExpressRoute en su suscripción de Azure en un grupo de recursos. 
    * Al crear ExpressRoute, elija **Oracle Cloud FastConnect** como proveedor de servicios. Para crear un circuito de ExpressRoute, consulte la [guía paso a paso](../../../expressroute/expressroute-howto-circuit-portal-resource-manager.md).
    * Un circuito de Azure ExpressRoute proporciona opciones de ancho de banda granular, mientras que FastConnect admite 1, 2, 5 o 10 Gbps. Por lo tanto, se recomienda elegir una de estas opciones de ancho de banda en ExpressRoute.

    ![Crear un circuito de ExpressRoute](media/configure-azure-oci-networking/exr-create-new.png)
1. Anote su **clave de servicio** de ExpressRoute. Debe proporcionar la clave mientras configura su circuito de FastConnect.

    ![Clave de servicio de ExpressRoute](media/configure-azure-oci-networking/exr-service-key.png)

    > [!IMPORTANT]
    > Se le facturarán los cargos de ExpressRoute tan pronto como se aprovisione el circuito de ExpressRoute (incluso si el **estado del proveedor** es **No aprovisionado**).

1. Establezca dos espacios de direcciones IP privadas de /30 cada una, para que no se superpongan con la red virtual de Azure o con el espacio de direcciones IP de la red de la nube virtual OCI. Esto es, debe usar el primer espacio de direcciones IP como espacio de direcciones principal y el segundo espacio de direcciones IP como espacio de direcciones secundario. Anote las direcciones que necesite al configurar su circuito de FastConnect.
1. Crear una puerta de enlace de enrutamiento dinámico (DRG). La necesitará cuando cree su circuito de FastConnect. Para obtener más información, consulte la documentación de la [puerta de enlace de enrutamiento dinámico](https://docs.cloud.oracle.com/iaas/Content/Network/Tasks/managingDRGs.htm).
1. Cree un circuito FastConnect en su inquilino de Oracle. Para obtener más información, consulte la [documentación de Oracle](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/azure.htm).
  
    * En la configuración de FastConnect, seleccione **Microsoft Azure: ExpressRoute** como el proveedor.
    * Seleccione la puerta de enlace de enrutamiento dinámico que aprovisionó en el paso anterior.
    * Seleccione el ancho de banda que será aprovisionado. Para conseguir un rendimiento óptimo, el ancho de banda debe coincidir con el ancho de banda seleccionado al crear el circuito de ExpressRoute.
    * En **Clave de servicio del proveedor**, pegue la clave de servicio de ExpressRoute.
    * Utilice el primer espacio privado de direcciones IP /30 que se detalló en un paso anterior para la **dirección IP del BGP principal** y el segundo espacio de direcciones IP /30 privadas para la **dirección del BGP secundario**.
        * Asigne la primera dirección que se pueda usar de los dos rangos para la dirección IP de Oracle BGP (principal y secundaria) y la segunda dirección a la dirección IP de BGP del cliente (desde una perspectiva de FastConnect). La primera dirección IP que se puede usar es la segunda dirección IP en el espacio de direcciones /30 (la primera dirección IP está reservada para Microsoft).
    * Haga clic en **Crear**.
1. Complete la vinculación de FastConnect a la red de la nube virtual en su inquilino de Oracle a través de la puerta de enlace de enrutamiento dinámico mediante la tabla de rutas.
1. Vaya a Azure y asegúrese de que el **estado del proveedor** del circuito de ExpressRoute haya cambiado a **Aprovisionado** y que se haya aprovisionado un emparejamiento **privado de Azure**. Este es un requisito previo para los siguientes pasos.

    ![Estado del proveedor de ExpressRoute](media/configure-azure-oci-networking/exr-provider-status.png)
1. Haga clic en el emparejamiento **privado de Azure**. Verá que los detalles del emparejamiento se configuraron automáticamente en función de la información que especificó al configurar su circuito de FastConnect.

    ![Configuración de emparejamiento privado](media/configure-azure-oci-networking/exr-private-peering.png)

## <a name="connect-virtual-network-to-expressroute"></a>Conectar la red virtual a ExpressRoute

1. Cree una red virtual y una puerta de enlace de red virtual, si aún no lo ha hecho. Para obtener más información, consulte la [guía paso a paso](../../../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md).
1. Configure la conexión entre la puerta de enlace de red virtual y su circuito de ExpressRoute ejecutando el [script de Terraform](https://github.com/microsoft/azure-oracle/tree/master/InterConnect-2) o ejecutando el comando de PowerShell para [configurar ExpressRoute FastPath ](../../../expressroute/expressroute-howto-linkvnet-arm.md#configure-expressroute-fastpath).

Una vez que haya completado la configuración de red, puede comprobar la validez de su configuración haciendo clic en **Obtener registros de ARP** y **Obtener tabla de rutas** en la hoja de emparejamiento privado de ExpressRoute en Azure Portal.

## <a name="automation"></a>Automation

Microsoft ha creado scripts de Terraform para permitir la implementación automatizada de la interconexión de la red. Los scripts de Terraform deben autenticarse con Azure antes de ejecutarse, ya que deben tener los permisos adecuados para la suscripción de Azure. La autenticación se puede realizar con una [entidad de servicio de Azure Active Directory ](../../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) o con la CLI de Azure. Para obtener más información, consulte la [documentación de Terraform](https://www.terraform.io/docs/providers/azurerm/auth/azure_cli.html).

Los scripts de Terraform y la documentación relacionada para implementar la interconexión se pueden encontrar en este [repositorio de GitHub](https://aka.ms/azureociinterconnecttf).

## <a name="monitoring"></a>Supervisión

Al instalar agentes en ambas nubes, puede aprovechar Azure [Network Performance Monitor (NPM)](../../../expressroute/how-to-npm.md) para supervisar el rendimiento de la red de un extremo a otro. NPM le ayudará a identificar y solucionar fácilmente los problemas de la red.

## <a name="delete-the-interconnect-link"></a>Eliminar el vínculo de interconexión

Para eliminar la interconexión, debe seguir los siguientes pasos en el orden especificado. De lo contrario, el circuito de ExpressRoute tendrá un "estado erróneo".

1. Elimine la conexión de ExpressRoute. Elimine la conexión haciendo clic en el icono de **eliminar** que está en la página de la conexión. Para obtener más información, consulte la [documentación de ExpressRoute](../../../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md#clean-up-resources).
1. Elimine Oracle FastConnect de la consola en la nube de Oracle.
1. Una vez que se haya eliminado el circuito de Oracle FastConnect, podrá eliminar el circuito de Azure ExpressRoute.

En este punto, el proceso de eliminación y desaprovisionamiento estará completo.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre la conexión entre las nubes de OCI y Azure, consulte la [documentación de Oracle](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/azure.htm).
* Use los [scripts de Terraform](https://aka.ms/azureociinterconnecttf) para implementar la infraestructura para aplicaciones de Oracle específicas en Azure, y configure la interconexión de la red. 
