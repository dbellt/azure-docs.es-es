---
title: Integración de Microsoft Azure con Oracle Cloud Infrastructure | Microsoft Docs
description: Obtenga información acerca de las soluciones que integran las aplicaciones de Oracle que se ejecutan en Microsoft Azure con bases de datos de Oracle Cloud Infrastructure (OCI).
author: dbakevlar
ms.service: virtual-machines
ms.subservice: oracle
ms.collection: linux
ms.topic: article
ms.date: 06/01/2020
ms.author: kegorman
ms.openlocfilehash: 04a3fb9e4e7dd1d498714cd3b2ebd4c5f6b55bec
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107210353"
---
# <a name="oracle-application-solutions-integrating-microsoft-azure-and-oracle-cloud-infrastructure"></a>Soluciones de aplicaciones de Oracle que integran Microsoft Azure y Oracle Cloud Infrastructure

Microsoft y Oracle se han asociado para proporcionar baja latencia y conectividad de alto rendimiento entre nubes, lo que permite sacar provecho de lo mejor de ambas nubes. 

Mediante esta conectividad entre nubes, puede particionar una aplicación de niveles múltiples para que ejecute su nivel de base de datos en Oracle Cloud Infrastructure (OCI) y la aplicación y los restantes niveles en Microsoft Azure. La experiencia es similar a ejecutar la pila de toda la solución en una nube individual. 

Si está interesado en ejecutar el middleware, incluido WebLogic Server, en la infraestructura de Azure, pero tiene la base de datos de Oracle ejecutándose en OCI, consulte [Aplicaciones de Azure de WebLogic Server](oracle-weblogic.md).

Si está interesado en implementar soluciones de Oracle por completo en la infraestructura de Azure, consulte [Imágenes de máquina virtual de Oracle y su implementación en Microsoft Azure](oracle-vm-solutions.md).

## <a name="scenario-overview"></a>Información general de escenario

La conectividad entre nubes proporciona una solución para ejecutar no solo las aplicaciones líderes del sector de Oracle, sino también sus propias aplicaciones personalizadas, en máquinas virtuales de Azure y disfrutar de las ventajas de servicios de base de datos hospedada en OCI. 

A partir de mayo de 2020, las siguientes aplicaciones están certificadas en una configuración entre nubes:

* E-Business Suite
* JD Edwards EnterpriseOne
* PeopleSoft
* Aplicaciones comerciales de Oracle
* Oracle Hyperion Financial Management

El siguiente diagrama es una introducción de alto nivel de la solución conectada. Por motivos de simplicidad, el diagrama muestra solo una capa de aplicación y una capa de datos. En función de la arquitectura de la aplicación, la solución puede incluir niveles adicionales, como un clúster de WebLogic Server o un nivel web en Azure. Para más información, consulte las siguientes secciones:

![Introducción a la solución OCI de Azure](media/oracle-oci-overview/crosscloud.png)

## <a name="region-availability"></a>Disponibilidad regional 

La conectividad entre nubes está limitada a las siguientes regiones:
* Este de EE. UU. de Azure (eastus) y OCI Ashburn VA (Este de EE. UU.)
* Sur de Reino Unido de Azure (uksouth) y OCI London (Sur de Reino Unido)
* Centro de Canadá de Azure (canadacentral) y OCI Toronto (Sudeste de Canadá)
* Oeste de Europa de Azure (westeurope) y OCI Amsterdam (Noroeste de Países Bajos)
* Japón Oriental de Azure (japaneast) y OCI Tokio (Japón Oriental)
* Oeste de EE. UU. de Azure (WestUS) y OCI San Jose (US West)
* Centro-oeste de Alemania (Fráncfort) y OCI Centro de Alemania (Fráncfort)


## <a name="networking"></a>Redes

Los clientes empresariales a menudo deciden diversificar e implementar las cargas de trabajo a través de varias nubes por motivos empresariales y de funcionamiento. Para diversificar, los clientes interconectan las redes en la nube mediante Internet, IPSec VPN, Internet o mediante la solución de conectividad directa del proveedor de la nube a través de su red local. La interconexión de redes en la nube puede requerir considerables inversiones de tiempo, dinero, diseño, adquisición, instalación, pruebas y operaciones. 

Para abordar los desafíos de los clientes, Oracle y Microsoft han integrado varias nubes. Las redes entre nubes se establecen mediante la conexión de un circuito [ExpressRoute](../../../expressroute/expressroute-introduction.md) de Microsoft Azure con un circuito [FastConnect](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/fastconnectoverview.htm) de OCI. Esta conectividad es posible donde una ubicación de emparejamiento de Azure ExpressRoute está cerca o en la misma ubicación de emparejamiento que FastConnect de OCI. Esta configuración permite una conectividad rápida y segura entre las dos nubes sin necesidad de que haya ningún proveedor de servicios intermedio.

Con ExpressRoute y FastConnect, los clientes pueden emparejar una red virtual de Azure con una red virtual en la nube de OCI, siempre que el espacio de direcciones IP privado no se solape. El emparejamiento de las dos redes permite que un recurso de la red virtual se comunique con un recurso de la red virtual en la nube de OCI como si ambos estuvieran en la misma red.

## <a name="network-security"></a>Seguridad de las redes

La seguridad de la red es un componente fundamental de cualquier aplicación empresarial y es clave para esta solución de varias nubes. Todo el tráfico que atraviesa ExpressRoute y FastConnect pasa por una red privada. Esta configuración permite una comunicación segura entre una red virtual de Azure y una red virtual en la nube de Oracle. No es necesario proporcionar una dirección IP pública a las máquinas virtuales de Azure. Tampoco se necesita una puerta de enlace de Internet en OCI. Toda la comunicación se produce mediante la dirección IP privada de las máquinas.

Además, puede configurar [listas de seguridad](https://docs.cloud.oracle.com/iaas/Content/Network/Concepts/securitylists.htm) en la red virtual en la nube de OCI y en las reglas de seguridad (adjuntas a los[grupos de seguridad de red](../../../virtual-network/network-security-groups-overview.md) de Azure). Dichas reglas se usan para controlar el tráfico que fluye entre las máquinas en las redes virtuales. Las reglas de seguridad de red se pueden agregar a nivel de máquina, a nivel de subred y a nivel de red virtual.

Las [aplicaciones de Azure WebLogic Server](oracle-weblogic.md) cada una crea un grupo de seguridad de red preconfigurado para trabajar con las configuraciones de puerto de WebLogic Server.
 
## <a name="identity"></a>Identidad

La identidad es uno de los pilares principales de la asociación entre Microsoft y Oracle. Se ha realizado un trabajo considerable para integrar [Oracle Identity Cloud Service](https://docs.oracle.com/en/cloud/paas/identity-cloud/index.html) (IDCS) con [Azure Active Directory](../../../active-directory/index.yml) (Azure AD). Azure AD es un servicio de administración de acceso y de identidades basado en la nube de Microsoft. Los usuarios pueden iniciar sesión y acceder a varios recursos con la ayuda de Azure AD. Azure AD también permite administrar los usuarios y sus permisos.

Actualmente, esta integración permite realizar la adminsitración en ubicación central, que es Azure Active Directory. Azure AD sincroniza los cambios en el directorio con el directorio correspondiente de Oracle y se usa para el inicio de sesión único en las soluciones de Oracle entre nubes.

## <a name="next-steps"></a>Pasos siguientes

Empezar a trabajar con una [red entre nubes](configure-azure-oci-networking.md) de Azure y OCI. 

Para más información y las notas del producto acerca de OCI, consulte la documentación de [Oracle Cloud](https://docs.cloud.oracle.com/iaas/Content/home.htm).
