---
title: 'Conexión de la red virtual de Azure a CloudSimple mediante ExpressRoute: Azure VMware Solution by CloudSimple'
description: Describe cómo obtener información de emparejamiento para una conexión entre la red virtual de Azure y el entorno de CloudSimple
author: shortpatti
ms.author: v-patsho
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 5bd0d7a02a82998b170ebb4fbfe207a634db2b93
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108183711"
---
# <a name="connect-azure-virtual-network-to-cloudsimple-using-expressroute"></a>Conexión de la red virtual de Azure a CloudSimple mediante ExpressRoute

Puede extender la red de la nube privada a la red virtual de Azure y a los recursos de Azure. Una conexión de ExpressRoute permite acceder a los recursos que se ejecutan en la suscripción de Azure desde la nube privada.

## <a name="request-authorization-key"></a>Solicitud de la clave de autorización

Se requiere una clave de autorización para la conexión de ExpressRoute entre la nube privada y la red virtual de Azure. Para obtenerla, abra una incidencia de <a href="https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest" target="_blank">soporte técnico</a>.  Use la siguiente información para la solicitud:

* Tipo de problema: **Técnico**
* Suscripción: **Seleccione la suscripción donde está implementado el servicio CloudSimple**.
* Servicio: **VMware Solution by CloudSimple**
* Tipo de problema: **Solicitud de servicio**
* Subtipo de problema: **Clave de autorización para la conexión de VNET de Azure**
* Asunto: **Solicitud de clave de autorización para la conexión de VNET de Azure**

## <a name="get-peering-information-from-cloudsimple-portal"></a>Obtención de información de emparejamiento desde el portal de CloudSimple

Para configurar la conexión, debe establecer una conexión entre la red virtual de Azure y el entorno de CloudSimple.  Como parte del procedimiento, debe proporcionar el URI del circuito de emparejamiento y la clave de autorización. Obtenga el URI y la clave de autorización del [portal de CloudSimple](access-cloudsimple-portal.md).  Seleccione **Red** en el menú lateral y **Azure Network Connection** (Conexión de red de Azure). O bien, seleccione **Cuenta** en el menú lateral y **Azure network connection** (Conexión de red de Azure).

Copie el URI del circuito de emparejamiento y de la clave de autorización para cada una de las regiones con el icono de *copia*. Para cada región de CloudSimple que quiera conectar:

1. Haga clic en **Copiar** para copiar el URI. Péguelo en un archivo donde pueda estar disponible para agregarlo a Azure Portal.  
2. Haga clic en **Copiar** para copiar la clave de autorización y péguela también en el archivo.

Copie la clave de autorización y el URI del circuito de emparejamiento que está en estado **Disponible**.  El estado **Usado** indica que la clave ya se ha usado para crear una conexión de red virtual.

![Página de conexión de la red virtual](media/virtual-network-connection.png)

Para más información para configurar el vínculo de la red virtual de Azure a CloudSimple, consulte [Connect your CloudSimple Private Cloud environment to the Azure virtual network using ExpressRoute](azure-expressroute-connection.md) (Conexión del entorno de nube privada de CloudSimple a la red virtual de Azure mediante ExpressRoute).

## <a name="next-steps"></a>Pasos siguientes

* [Conexión de red virtual de Azure a la nube privada](azure-expressroute-connection.md)
* [Conexión a la red local mediante Azure ExpressRoute](on-premises-connection.md)
