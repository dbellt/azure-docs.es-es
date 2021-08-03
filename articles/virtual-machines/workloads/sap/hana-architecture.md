---
title: Arquitectura de SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Conozca la arquitectura para implementar SAP HANA en Azure (instancias grandes).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: juergent
editor: ''
ms.service: virtual-machines-sap
ms.subservice: baremetal-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/19/2021
ms.author: madhukan
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 547e48e9cecb672c5274bc001178b2ea2aaf47af
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110577657"
---
# <a name="sap-hana-large-instances-architecture-on-azure"></a>Arquitectura de SAP HANA en Azure (instancias grandes)

En este artículo, se describe la arquitectura para implementar SAP HANA en Azure (instancias grandes) (también conocida como BareMetal Infrastructure). 

En un nivel genérico, la solución SAP HANA en Azure (instancias grandes) tiene la capa de aplicación de SAP en las máquinas virtuales (VM). La capa de base de datos se encuentra en la instancia grande de HANA certificada por SAP, que se encuentra en la misma región de Azure que las máquinas virtuales IaaS de Azure.

> [!NOTE]
> Implemente la capa de aplicación de SAP en la misma región de Azure que la capa del sistema de administración de bases de datos (DBMS) de SAP. Esta regla está bien documentada en la información publicada sobre cargas de trabajo de SAP en Azure. 

## <a name="architectural-overview"></a>Introducción a la arquitectura

La arquitectura global de SAP HANA en Azure (instancias grandes) proporciona una configuración de hardware con certificación SAP TDI. El hardware es un servidor de alto rendimiento sin sistema operativo y sin virtualización para la base de datos de SAP HANA. También proporciona la flexibilidad de Azure para escalar los recursos de la capa de aplicación de SAP según sus necesidades.

![Introducción a la arquitectura de SAP HANA en Azure (Instancias grandes)](./media/hana-overview-architecture/image1-architecture.png)

La arquitectura que se muestra se divide en tres secciones:

- **Derecha**: muestra una infraestructura local que ejecuta diferentes aplicaciones en los centros de datos de modo que los usuarios finales puedan acceder a aplicaciones de línea de negocio, como SAP. Idealmente, esta infraestructura local se conecta a Azure mediante [ExpressRoute](https://azure.microsoft.com/services/expressroute/).

- **Centro**: muestra IaaS de Azure y, en este caso, el uso de máquinas virtuales para hospedar SAP u otras aplicaciones que usan SAP HANA como DBMS. Las instancias de HANA menores que funcionan con la memoria proporcionada por las máquinas virtuales se implementan en máquinas virtuales junto con el nivel de aplicación. Para más información sobre máquinas virtuales, consulte [Virtual Machines](https://azure.microsoft.com/services/virtual-machines/).

   Se usan los servicios de red de Azure para agrupar los sistemas SAP junto con otras aplicaciones en redes virtuales. Estas redes virtuales se conectan a los sistemas locales, así como a SAP HANA en Azure (instancias grandes).

   Para más información sobre las aplicaciones y bases de datos de SAP NetWeaver que admiten la ejecución en Azure, consulte [SAP Support Note #1928533 – SAP Applications on Azure: Supported products and Azure VM types](https://launchpad.support.sap.com/#/notes/1928533). (Nota de SAP 1928533: Aplicaciones de SAP en Azure: productos y tipos de máquina virtual de Azure admitidos) Para obtener documentación sobre cómo implementar soluciones de SAP en Azure, consulte:

  -  [Uso de SAP en máquinas virtuales Windows](./get-started.md?toc=/azure/virtual-machines/linux/toc.json)
  -  [Uso de soluciones de SAP en Azure Virtual Machines](get-started.md)

- **Izquierda**: muestra el hardware con certificación SAP HANA TDI en la demarcación de instancias grandes de Azure. Las unidades de HANA (instancias grandes) se conectan a las redes virtuales de la suscripción de Azure con la misma tecnología que la empleada para la conectividad entre la infraestructura local y la de Azure. A partir de mayo de 2019, se introdujo una optimización que permite la comunicación entre las unidades de HANA (instancias grandes) y las máquinas virtuales de Azure sin la puerta de enlace de ExpressRoute. Esta optimización, denominada FastPath de ExpressRoute, se muestra en el diagrama anterior mediante las líneas rojas.

## <a name="components-of-the-azure-large-instance-stamp"></a>Componentes del stamp de Azure (instancias grandes)

El sello de instancias grandes de Azure en sí combina los componentes siguientes:

- **Computación**: servidores basados en distintas generaciones de procesadores Intel Xeon que ofrecen la funcionalidad de computación necesaria y poseen certificación para SAP HANA.
- **Network** (Red): tejido de red unificada de alta velocidad que interconecta los componentes de computación, almacenamiento y LAN.
- **Almacenamiento**: infraestructura de almacenamiento a la que se accede por medio de un tejido de red unificada. La capacidad de almacenamiento proporcionada depende de la configuración de SAP HANA en Azure (instancias grandes) implementada. Hay más capacidad de almacenamiento disponible a un costo mensual adicional.

## <a name="tenants"></a>Inquilinos

Dentro de la infraestructura multiinquilino del sello de instancias grandes, los clientes se implementan como inquilinos aislados. En la implementación del inquilino, asigne un nombre a una suscripción de Azure dentro de la inscripción de Azure. HANA (instancias grandes) se factura a esta suscripción de Azure. Estos inquilinos mantienen una relación de 1:1 con la suscripción de Azure. Para una red, es posible acceder a una unidad de HANA (instancias grandes) implementada en un inquilino de una región de Azure desde redes virtuales diferentes que pertenecen a diferentes suscripciones de Azure. Esas suscripciones de Azure deben pertenecer a la misma inscripción de Azure.

## <a name="availability-across-regions"></a>Disponibilidad entre regiones

Al igual que las máquinas virtuales, SAP HANA en Azure (instancias grandes) se ofrece en varias regiones de Azure. Para ofrecer funcionalidades de recuperación ante desastres, puede elegir participar. Los diferentes sellos de instancia grande en una región geopolítica están conectados entre sí. Por ejemplo, las demarcaciones de HANA (instancias grandes) en Oeste de EE. UU. y Este de EE. UU. se conectan a través de una conexión de red dedicada para la replicación de recuperación ante desastres.

## <a name="available-skus"></a>SKU disponibles

Al igual que Azure le permite elegir entre diferentes tipos de máquinas virtuales, puede elegir entre diferentes SKU de HANA (instancias grandes). Puede seleccionar la SKU adecuada para el tipo de carga de trabajo específico de SAP HANA. SAP aplica una relación de memoria a sockets de procesador para diferentes cargas de trabajo en función de la generación de los procesadores Intel. Para más información sobre las SKU disponibles, consulte [SKU disponibles para HLI](hana-available-skus.md).

## <a name="next-steps"></a>Pasos siguientes

Conozca la arquitectura de red de SAP HANA (instancias grandes).

> [!div class="nextstepaction"]
> [Arquitectura de red de SAP HANA (instancias grandes)](hana-network-architecture.md)
