---
title: Información general sobre BareMetal Infrastructure en Azure
description: Ofrece información general sobre BareMetal Infrastructure en Azure.
ms.custom: references_regions
ms.topic: conceptual
ms.subservice: workloads
ms.date: 04/08/2021
ms.openlocfilehash: 7a4998a096a5c5d9e793c34d5046dce59262a2ae
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107257575"
---
#  <a name="what-is-baremetal-infrastructure-on-azure"></a>¿Qué es BareMetal Infrastructure en Azure?

Microsoft Azure ofrece una infraestructura en la nube con una amplia gama de servicios en la nube integrados para satisfacer sus necesidades empresariales. Sin embargo, en algunos casos, es posible que tenga que ejecutar servicios en servidores sin sistema operativo sin una capa de virtualización. Es posible que necesite acceso raíz y control sobre el sistema operativo (SO). Para satisfacer esta necesidad, Azure ofrece BareMetal Infrastructure para varias aplicaciones de alto valor y críticas.

BareMetal Infrastructure consta de instancias de BareMetal dedicadas (instancias de proceso), de almacenamiento adecuado para aplicaciones y alto rendimiento (NFS, ISCSI y canal de fibra), así como de un conjunto de LAN virtuales (VLAN) específicas de funciones en un entorno aislado. El almacenamiento se puede compartir entre instancias de BareMetal para habilitar características, como clústeres de escalabilidad horizontal, o para crear pares de alta disponibilidad con STONITH.
 
Este entorno también tiene VLAN especiales a las que puede acceder si ejecuta máquinas virtuales (VM) en una o varias redes virtuales (VNet) de Azure en su suscripción de Azure. Todo el entorno se representa como un grupo de recursos en la suscripción de Azure.

BareMetal Infrastructure se ofrece en más de 30 SKU de servidores de 2 sockets a 24 sockets y en una memoria que oscila entre 1,5 TB y 24 TB. También hay disponible un gran conjunto de SKU con la memoria Octane. Azure ofrece la mayor variedad de instancias sin sistema operativo en una nube de hiperescala.

## <a name="why-baremetal-infrastructure"></a>Motivo para usar BareMetal Infrastructure  

Algunas cargas de trabajo centrales de la empresa se componen de tecnologías que simplemente no están diseñadas para ejecutarse en una configuración de nube virtualizada típica. Requieren una arquitectura especial, un hardware certificado o tamaños extremadamente grandes. Aunque esas tecnologías tienen las características de continuidad empresarial y protección de datos más sofisticadas, esas características no se han creado para la nube virtualizada. Son más sensibles a las latencias y a los equipos cercanos ruidosos y, además, requieren mucho más control sobre la administración de cambios y la actividad de mantenimiento.

BareMetal Infrastructure se ha creado, certificado y probado para un conjunto seleccionado de estas aplicaciones. Azure fue el primero en ofrecer este tipo de soluciones y, desde entonces, destaca como líder con la cartera más grande y los sistemas más sofisticados.

BareMetal Infrastructure ofrece estas ventajas: 

- Instancias dedicadas
- Hardware certificado para cargas de trabajo especializadas
    - SAP (consulte la [nota de SAP n.º 1928533](https://launchpad.support.sap.com/#/notes/1928533))
    - Oracle (consulte el [id. de documento de Oracle n.º 948372.1](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=52088246571495&id=948372.1&_adf.ctrl-state=kwnkj1hzm_52))
- Sin sistema operativo (sin virtualización de proceso)
- Baja latencia entre las máquinas virtuales de aplicaciones hospedadas de Azure en las instancias de BareMetal (0,35 ms)
- Todas las unidades NVMe y SSD Flash
    - Hasta 1 PB/inquilino 
    - IOPS hasta 1,2 millones/inquilino 
    - Ancho de banda de red de 40/100 GB
    - Accesible a través de NFS, ISCSI y FC
- Alimentación redundante, fuentes de alimentación, NIC, TOR, puertos, WAN, almacenamiento y administración
- Reservas activas de sustitución en caso de error (sin necesidad de volver a configurar)
- Ventanas de mantenimiento coordinadas por el cliente
- Instantáneas compatibles con aplicación, archivo, creación de reflejo y clonación


## <a name="sku-availability-in-azure-regions"></a>Disponibilidad de SKU en regiones de Azure

BareMetal Infrastructure ofrece varias SKU certificadas para cargas de trabajo especializadas. Use las SKU específicas de la carga de trabajo para satisfacer sus necesidades.

- Instancias grandes: abarcan sistemas de 2 a 4 sockets.  
- Instancias muy grandes: abarcan sistemas de 4 a 20 sockets. 

BareMetal Infrastructure para cargas de trabajo especializadas está disponible en las siguientes regiones de Azure:
- Oeste de Europa
- Norte de Europa
- Compatibilidad con *zonas del Centro-oeste de Alemania
- Compatibilidad con *zonas del Este de EE. UU. 2
- Compatibilidad con *zonas del Este de EE. UU.
- Compatibilidad con *zonas del Oeste de EE. UU.
- Compatibilidad con *zonas del Oeste de EE. UU. 2
- Centro-sur de EE. UU.

>[!NOTE]
>La **compatibilidad con zonas** hace referencia a las zonas de disponibilidad dentro de una región donde las instancias de BareMetal se pueden implementar entre zonas para lograr alta resistencia y disponibilidad. Esta funcionalidad permite la compatibilidad con el escalado activo-activo multisitio.

## <a name="managing-baremetal-instances-in-azure"></a>Administración de instancias de BareMetal en Azure 

En función de sus necesidades, las topologías de aplicaciones de BareMetal Infrastructure pueden ser complejas. Puede implementar varias instancias, en una o varias ubicaciones, con almacenamiento compartido o dedicado, y conexiones LAN y WAN especializadas. Por lo tanto, para BareMetal Infrastructure, Azure ofrece una captura consultiva de esa información por un CSA o GBB en el campo de un portal de aprovisionamiento. 

Cuando se aprovisiona BareMetal Infrastructure, el sistema operativo, las redes, los volúmenes de almacenamiento, las ubicaciones en zonas y regiones y las conexiones WAN entre ubicaciones ya están preconfigurados. Puede registrar las licencias del sistema operativo (BYOL), configurar el sistema operativo e instalar el nivel de aplicación.

Podrá ver todos los recursos de BareMetal Infrastructure, así como su estado y sus atributos, en Azure Portal. Desde ahí, también puede operar las instancias y abrir solicitudes de servicio e incidencias de soporte técnico. 

## <a name="operational-model"></a>Modelo de funcionamiento
BareMetal Infrastructure es compatible con ISO 27001, ISO 27017, SOC 1 y SOC 2. También usa un modelo Traiga su propia licencia (BYOL): SO, carga de trabajo especializada y aplicaciones de terceros.  

En cuanto recibe acceso a la raíz y control total, asume la responsabilidad de los siguientes aspectos:
- Diseñar e implementar soluciones de copia de seguridad y recuperación, alta disponibilidad y recuperación ante desastres
- Proporcionar licencias, seguridad y soporte técnico para el sistema operativo y el software de terceros

Microsoft es responsable de lo siguiente:
- Proporcionar el hardware para cargas de trabajo especializadas 
- Aprovisionar el SO

:::image type="content" source="media/concepts-baremetal-infrastructure-overview/baremetal-support-model.png" alt-text="Diagrama del modelo de soporte técnico de BareMetal Infrastructure" border="false":::

## <a name="baremetal-instance-stamp"></a>Sello de instancias de BareMetal

El sello de instancias de BareMetal en sí combina los componentes siguientes:

- **Computación**: servidores basados en la generación de procesadores Intel Xeon que proporcionan la funcionalidad de computación necesaria y poseen certificación para la carga de trabajo especializada.

- **Red:** tejido de red unificada de alta velocidad que interconecta los componentes de computación, almacenamiento y LAN.

- **Storage:** infraestructura a la que se accede por medio de un tejido de red unificada.

Dentro de la infraestructura multiinquilino del sello de BareMetal, los clientes se implementan en inquilinos aislados. Al implementar un inquilino, asigne un nombre a una suscripción de Azure dentro de la inscripción de Azure. Esta suscripción de Azure es la que se factura para las instancias de BareMetal.

>[!NOTE]
>Un cliente que implementa una instancia de BareMetal está aislado en un inquilino. Un inquilino se aísla en el nivel de redes, almacenamiento y proceso de otros inquilinos. Las unidades de almacenamiento y proceso asignadas a distintos inquilinos no pueden verse ni comunicarse entre sí en sus instancias de BareMetal.

## <a name="operating-system"></a>Sistema operativo
Durante el aprovisionamiento de la instancia de BareMetal, puede seleccionar el sistema operativo que desea instalar en los equipos. 

>[!NOTE]
>Recuerde que BareMetal Infrastructure es un modelo de BYOL.

Las versiones del sistema operativo Linux disponibles son:
- Red Hat Enterprise Linux (RHEL)
- SUSE Linux Enterprise Server (SLES)

## <a name="storage"></a>Storage
BareMetal Infrastructure proporciona almacenamiento de NFS con redundancia alta y almacenamiento con canal de fibra. La infraestructura ofrece una integración profunda con cargas de trabajo empresariales como SAP, SQL, etc. También proporciona funcionalidades de administración de datos y protección de datos coherentes con la aplicación. Las herramientas de administración de autoservicio ofrecen funcionalidades de replicación pormenorizada, clonación e instantáneas que requieren menos espacio, junto con la supervisión desde un único panel. La infraestructura habilita las funcionalidades de RTO y RPO cero para satisfacer las necesidades de continuidad empresarial y disponibilidad de datos. 

La infraestructura de almacenamiento ofrece lo siguiente:
- Hasta 4 vínculos superiores de 100 GB
- Vínculos superiores de canal de fibra de hasta 32 GB
- Todas las unidades NVMe y SSD Flash
- Latencia ultra baja y alto rendimiento
- Escalado vertical hasta 4 PB de almacenamiento sin procesar 
- Hasta 11 millones de IOPS

Se admiten estos protocolos de acceso a los datos: 
- iSCSI 
- NFS (v3 o v4) 
- Canal de fibra 
- NVMe a través de FC  

## <a name="networking"></a>Redes
La arquitectura del servicio de red de Azure es un componente clave para implementar las carga de trabajo especializadas correctamente en instancias de BareMetal. Es probable que no todos los sistemas de TI ya se encuentren en Azure. Azure ofrece tecnología de red, de forma que se asemeja a un centro de datos virtual para las implementaciones de software locales. La funcionalidad de red de Azure necesaria para las instancias de BareMetal incluye lo siguiente:

- Redes virtuales de Azure conectadas al circuito Azure ExpressRoute que se conecta con los recursos de red locales.
- El circuito ExpressRoute que conecta el entorno local a Azure debería tener un ancho de banda mínimo de 1 Gbps o más.
- Active Directory y DNS ampliados en Azure o completamente en ejecución en Azure.

ExpressRoute permite ampliar la red local a la nube de Microsoft mediante una conexión privada con la ayuda de un proveedor de conectividad. Puede usar **ExpressRoute Local** para una transferencia de datos rentable entre la ubicación local y la región de Azure que desee. Para ampliar la conectividad a través de límites geopolíticos, puede habilitar **ExpressRoute Prémium**. 

Las instancias de BareMetal se aprovisionan en el intervalo de direcciones IP del servidor de red virtual de Azure.

:::image type="content" source="media/concepts-baremetal-infrastructure-overview/baremetal-infrastructure-diagram.png" alt-text="Diagrama de la arquitectura del diagrama de Azure BareMetal Infrastructure" lightbox="media/concepts-baremetal-infrastructure-overview/baremetal-infrastructure-diagram.png" border="false":::

La arquitectura que se muestra se divide en tres secciones:
- **Izquierda**: muestra la infraestructura local del cliente que ejecuta aplicaciones diferentes, que se conecta a través del enrutador perimetral de asociados o local, como Equinix. Para más información, vea [Ubicaciones y proveedores de conectividad: Azure ExpressRoute](../expressroute/expressroute-locations.md).
- **Centro**: muestra [ExpressRoute](../expressroute/expressroute-introduction.md) aprovisionado con la suscripción de Azure que ofrece conectividad con la red perimetral de Azure.
- **Derecha**: muestra IaaS de Azure y, en este caso, el uso de máquinas virtuales para hospedar las aplicaciones, que se aprovisionan en la red virtual de Azure.
- **Abajo**: muestra cómo usar la puerta de enlace de ExpressRoute habilitada con [FastPath de ExpressRoute](../expressroute/about-fastpath.md) para la conectividad de BareMetal con una latencia baja.   
   >[!TIP]
   >Para admitir esto, la puerta de enlace de ExpressRoute debe ser UltraPerformance. Para más información, vea [Acerca de las puertas de enlace de red virtual de ExpressRoute](../expressroute/expressroute-about-virtual-network-gateways.md).

## <a name="next-steps"></a>Pasos siguientes

El paso siguiente consiste en identificar las instancias de BareMetal y en interactuar con ellas en Azure Portal.

> [!div class="nextstepaction"]
> [Administración de instancias de BareMetal mediante Azure Portal](connect-baremetal-infrastructure.md)
