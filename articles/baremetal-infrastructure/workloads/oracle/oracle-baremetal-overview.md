---
title: ¿Qué es BareMetal Infrastructure para Oracle?
description: Obtenga información acerca de las características que ofrece BareMetal Infrastructure para las cargas de trabajo de Oracle.
ms.topic: conceptual
ms.subservice: baremetal-oracle
ms.date: 04/14/2021
ms.openlocfilehash: f3ecb1fbcad04455515408e4aadadfb2cef3cf9a
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110578591"
---
# <a name="what-is-baremetal-infrastructure-for-oracle"></a>¿Qué es BareMetal Infrastructure para Oracle?

En este artículo se proporciona información general acerca de las características que ofrece BareMetal Infrastructure para las cargas de trabajo de Oracle.

BareMetal Infrastructure para Oracle se basa en Unified Computing System (UCS) certificado por Oracle y FlexPod. La plataforma de FlexPod ofrece tecnologías de servidor, redes y almacenamiento validados previamente. Ofrece almacenamiento NFS, lo que proporciona integración con el protocolo DirectNFS. Los servidores BareMetal están dedicados al usuario, sin ningún hipervisor en las instancias de BareMetal. 

Estas instancias ejecutan aplicaciones críticas que requieren una carga de trabajo de Oracle. Las instancias de BareMetal proporcionan baja latencia (0,35 ms) a las aplicaciones que se ejecutan en máquinas virtuales (VM) de Azure. BareMetal proporciona un disco de almacenamiento compartido y es compatible con la multidifusión que es necesaria para la comunicación de nodo a nodo con una red de interconexión privada dedicada. 

Las siguientes son otras características de BareMetal Infrastructure para Oracle:

- Servidores UCS certificados por Oracle: UCSB200-M5, UCSB460-M4, UCSB480-M5
- Comunicación de nodo a nodo (multicanal) de Oracle Real Application Clusters (RAC) mediante la LAN virtual privada (VLAN) de 40 Gb.
- Hardware administrado por Microsoft
  - Almacenamiento, redes, energía y administración redundantes
  - Supervisión de infraestructura, reparaciones y reemplazos
  - Incluye Azure ExpressRoute al controlador de dominio del cliente
  - Seguridad física y de red mejorada, puede acceder a todos los servicios en la nube de Azure

### <a name="supported-protocols"></a>Protocolos admitidos

Los siguientes protocolos se usan para distintos puntos de montaje en los servidores BareMetal para la carga de trabajo de Oracle.

- Montaje del sistema operativo: iSCSI
- Datos/registro: NFSv3
- Copia de seguridad/archivo: NFSv4

### <a name="licensing"></a>Licencias

- Traiga su propio sistema operativo local y licencias de Oracle.

### <a name="operating-system"></a>Sistema operativo

Los servidores se cargan previamente con el sistema operativo RHEL 7.6.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de las SKU para cargas de trabajo de Oracle con BareMetal.

> [!div class="nextstepaction"]
> [SKU de BareMetal para cargas de trabajo de Oracle](oracle-baremetal-skus.md)
