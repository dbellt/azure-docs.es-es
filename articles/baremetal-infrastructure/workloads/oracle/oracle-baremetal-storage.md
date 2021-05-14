---
title: Almacenamiento en la infraestructura sin sistema operativo para cargas de trabajo de Oracle
description: Información sobre el almacenamiento que ofrece la infraestructura sin sistema operativo para cargas de trabajo de Oracle.
ms.topic: reference
ms.subservice: workloads
ms.date: 04/14/2021
ms.openlocfilehash: 685f4d401d05d897b91ad12dbd79f7d175db05c6
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107558787"
---
# <a name="storage-on-baremetal-for-oracle-workloads"></a>Almacenamiento en la infraestructura sin sistema operativo para cargas de trabajo de Oracle

En este artículo, se ofrece información general sobre el almacenamiento que ofrece la infraestructura sin sistema operativo para cargas de trabajo de Oracle.

La infraestructura sin sistema operativo para Oracle ofrece almacenamiento de Network File System (NFS) de NetApp. El almacenamiento de NFS no requiere la certificación Real Application Clusters (RAC) de Oracle. Para más información, consulte [Oracle RAC Technologies Matrix for Linux Clusters](https://www.oracle.com/database/technologies/tech-generic-linux-new.html) (Matriz de tecnologías de RAC de Oracle para clústeres de Linux).

Esta oferta de almacenamiento incluye compatibilidad de nivel 3 de un partner de OEM mediante controladores de almacenamiento A700 o A800s.

El almacenamiento en la infraestructura sin sistema operativo ofrece estas funcionalidades de almacenamiento prémium:

- Volúmenes de almacenamiento para datos/registro/cuórum/FSA ofrecidos mediante el protocolo dNFS.
- Redundancia de disco (*protección de hasta dos errores de disco*).
- Escale horizontalmente los datos a varios volúmenes, con un límite de 100 TB por volumen.
- Escale horizontalmente a varios controladores de almacenamiento, hasta un máximo de 12 controladores.
- No hay administración de nivel de disco (*agregar o quitar discos*), ya que Infra se encarga automáticamente.
- No hay tiempo de inactividad para redistribuir el contenido de archivos a volúmenes diferentes.
- Capacidad de aumentar o reducir volúmenes.
- Integración de SnapCenter para copias de seguridad mediante clonación y SnapVault.
- Cifrado de datos en reposo, compatible con FIPS (140-2).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre consideraciones de aplicación de revisiones para la infraestructura sin sistema operativo.

> [!div class="nextstepaction"]
> [Consideraciones de aplicación de revisiones para la infraestructura sin sistema operativo para Oracle](oracle-baremetal-patching.md)

