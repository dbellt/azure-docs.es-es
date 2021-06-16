---
title: Comparación entre Azure Files y Azure NetApp Files | Microsoft Docs
description: Comparación entre Azure NetApp Files y Azure Files.
author: jeffpatt24
services: storage
ms.service: storage
ms.subservice: files
ms.topic: conceptual
ms.date: 5/25/2021
ms.author: jeffpatt
ms.openlocfilehash: f6143c6aa35cf8cefddef0dab33b6b96a8edbce4
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110493570"
---
# <a name="azure-files-and-azure-netapp-files-comparison"></a>Comparación entre Azure NetApp Files y Azure Files

En este artículo se proporciona una comparación entre Azure Files y Azure NetApp Files. 

La mayoría de las cargas de trabajo que requieren almacenamiento de archivos en la nube funcionan bien tanto en Azure Files como en Azure NetApp Files. Para ayudar a determinar la mejor opción para la carga de trabajo, revise la información proporcionada en este artículo. Para más información, consulte la documentación de [Azure Files](./index.yml) y [Azure NetApp Files](../../azure-netapp-files/index.yml).

## <a name="features"></a>Características

| Category | Azure Files | Azure NetApp Files |
|---------|-------------------------|---------|
| Descripción | [Azure Files](https://azure.microsoft.com/services/storage/files/) es un servicio de clase empresarial totalmente administrado y de alta disponibilidad que está optimizado para cargas de trabajo de acceso aleatorio con actualizaciones de datos locales.<br><br> Azure Files se basa en la misma plataforma de Azure Storage que otros servicios como Azure Blobs. | [Azure NetApp Files](https://azure.microsoft.com/services/netapp/) es un servicio NAS (almacenamiento conectado a la red) de nivel empresarial, de alta disponibilidad y totalmente administrado, que puede administrar las cargas de trabajo más exigentes, de alto rendimiento y baja latencia, que requieren funcionalidades avanzadas de administración de datos. Permite la migración de cargas de trabajo, que se consideran "no migrables" sin él.<br><br>  Azure NetApp Files se basa en un equipo sin sistema operativo de NetApp con sistema operativo de almacenamiento ONTAP que se ejecuta dentro del centro de datos de Azure para proporcionar una experiencia coherente de Azure y un rendimiento parecido al de un entorno local. |
| Protocolos | Premium<br><ul><li>SMB 2.1, 3.0, 3.1.1</li><li>NFS 4.1 (versión preliminar)</li><li>REST</li></ul><br>Estándar<br><ul><li>SMB 2.1, 3.0, 3.1.1</li><li>REST</li></ul><br> Para más información, consulte los [protocolos de recursos compartidos de archivos disponibles](./storage-files-compare-protocols.md). | Todos los niveles<br><ul><li>SMB 1, 2.x, 3.x</li><li>NFS 3.0, 4.1</li><li>Acceso de protocolo doble (NFSv3/SMB)</li></ul><br> Para más información, consulte cómo crear volúmenes [NFS](../../azure-netapp-files/azure-netapp-files-create-volumes.md), [SMB](../../azure-netapp-files/azure-netapp-files-create-volumes-smb.md) o de [protocolo doble](../../azure-netapp-files/create-volumes-dual-protocol.md). |
| Disponibilidad regional | Premium<br><ul><li>Más de 30 regiones</li></ul><br>Estándar<br><ul><li>Todas las regiones</li></ul><br> Para obtener más información, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=storage). | Todos los niveles<br><ul><li>Más de 25 regiones</li></ul><br> Para obtener más información, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=storage). |
| Redundancia | Premium<br><ul><li>LRS</li><li>ZRS</li></ul><br>Estándar<br><ul><li>LRS</li><li>ZRS</li><li>GRS</li><li>GZRS</li></ul><br> Para más información, consulte la sección sobre [redundancia](./storage-files-planning.md#redundancy). | Todos los niveles<br><ul><li>Alta disponibilidad local integrada</li><li>[Replicación entre regiones](../../azure-netapp-files/cross-region-replication-introduction.md)</li></ul> |
| Acuerdo de Nivel de Servicio (SLA)<br><br> Tenga en cuenta que los Acuerdos de Nivel de Servicio para Azure Files y Azure NetApp Files se calculan de forma diferente. | [Acuerdo de Nivel de Servicio para Azure Files](https://azure.microsoft.com/support/legal/sla/storage/) | [Acuerdo de Nivel de Servicio para Azure NetApp Files](https://azure.microsoft.com/support/legal/sla/netapp) |  
| Autenticación y autorización basadas en identidad | SMB<br><ul><li>Active Directory Domain Services (AD DS)</li><li>Azure Active Directory Domain Services (Azure AD DS)</li></ul><br> Tenga en cuenta que la autenticación basada en identidad solo se admite cuando se utiliza el protocolo SMB. Para más información, consulte [Preguntas frecuentes](./storage-files-faq.md#security-authentication-and-access-control). | SMB<br><ul><li>Active Directory Domain Services (AD DS)</li><li>Azure Active Directory Domain Services (Azure AD DS)</li></ul><br> Protocolo doble NFS/SMB<ul><li>Integración de ADDS/LDAP</li></ul><br>NFSv3/NFSv4.1<ul><li>Integración de ADDS/LDAP (próximamente)</li><li>Grupos extendidos de NFS (próximamente)</li></ul><br> Para más información, consulte [Preguntas frecuentes](../../azure-netapp-files/azure-netapp-files-faqs.md). |
| Cifrado | Todos los protocolos<br><ul><li>Cifrado en reposo (AES 256) con claves administradas por el cliente o por Microsoft</li></ul><br>SMB<br><ul><li>Cifrado de Kerberos con AES 256 o RC4-HMAC</li><li>Cifrado en tránsito</li></ul><br>REST<br><ul><li>Cifrado en tránsito</li></ul><br> Para más información, consulte la sección sobre [seguridad](./storage-files-compare-protocols.md#security). | Todos los protocolos<br><ul><li>Cifrado en reposo (AES 256) con claves administradas por Microsoft </li></ul><br>NFS 4.1<ul><li>Cifrado en tránsito mediante Kerberos con AES 256</li></ul><br> Para más información, consulte la sección sobre [preguntas frecuentes sobre seguridad](../../azure-netapp-files/azure-netapp-files-faqs.md#security-faqs). |
| Opciones de acceso | <ul><li>Internet</li><li>Acceso seguro a redes virtuales</li><li>VPN Gateway</li><li>ExpressRoute</li><li>Azure File Sync</li></ul><br> Para más información, consulte la sección acerca de las [consideraciones sobre la red](./storage-files-networking-overview.md). | <ul><li>Acceso seguro a redes virtuales</li><li>VPN Gateway</li><li>ExpressRoute</li><li>[Caché de archivos globales](https://cloud.netapp.com/global-file-cache/azure)</li><li>[HPC Cache](../../hpc-cache/hpc-cache-overview.md)</li></ul><br> Para más información, consulte la sección acerca de las [consideraciones sobre la red](../../azure-netapp-files/azure-netapp-files-network-topologies.md). |
| Protección de datos  | <ul><li>Instantáneas incrementales</li><li>Restauración automática de usuarios de archivos y directorios</li><li>Restauración en una nueva ubicación</li><li>Reversión local</li><li>Eliminación temporal del nivel de recurso compartido</li><li>Integración de Azure Backup</li></ul><br> Para más información, consulte [Azure Files mejora las funcionalidades de protección de datos](https://azure.microsoft.com/blog/azure-files-enhances-data-protection-capabilities/). | <ul><li>Instantáneas (255/volumen)</li><li>Restauración automática de usuarios de archivos o directorios</li><li>Restauración en el nuevo volumen</li><li>Reversión local</li><li>[Replicación entre regiones](../../azure-netapp-files/cross-region-replication-introduction.md) (versión preliminar)</li></ul><br> Para más información, consulte [Funcionamiento de las instantáneas de Azure NetApp Files](../../azure-netapp-files/snapshots-introduction.md). |
| Herramientas de migración  | <ul><li>Azure Data Box</li><li>Azure File Sync</li><li>Servicio de migración de almacenamiento</li><li>AzCopy</li><li>Robocopy</li></ul><br> Para más información, consulte [Migración a recursos compartidos de archivos de Azure](./storage-files-migration-overview.md). | <ul><li>[Caché de archivos globales](https://cloud.netapp.com/global-file-cache/azure)</li><li>[CloudSync](https://cloud.netapp.com/cloud-sync-service), [XCP](https://xcp.netapp.com/)</li><li>Servicio de migración de almacenamiento</li><li>AzCopy</li><li>Robocopy</li><li>Basado en la aplicación (por ejemplo, HSR, Data Guard, AOAG)</li></ul> |
| Niveles | <ul><li>Premium</li><li>Optimizado para transacciones</li><li>Acceso frecuente</li><li>Acceso esporádico</li></ul><br> Para más información, consulte la sección sobre [capas de almacenamiento](./storage-files-planning.md#storage-tiers). | <ul><li>Ultra</li><li>Premium</li><li>Estándar</li></ul><br> Todos los niveles proporcionan una latencia mínima en submilisegundos.<br><br> Para más información, consulte [Niveles de servicio](../../azure-netapp-files/azure-netapp-files-service-levels.md) y [Consideraciones sobre el rendimiento](../../azure-netapp-files/azure-netapp-files-performance-considerations.md). |
| Precios | [Precios de Azure Files](https://azure.microsoft.com/pricing/details/storage/files/) | [Precios de Azure NetApp Files](https://azure.microsoft.com/pricing/details/netapp/) |

## <a name="scalability-and-performance"></a>Escalabilidad y rendimiento

| Category | Azure Files | Azure NetApp Files |
|---------|---------|---------|
| Tamaño mínimo del volumen o recurso compartido | Premium<br><ul><li>100 GiB</li></ul><br>Estándar<br><ul><li>Sin mínimo.</li></ul> | Todos los niveles<br><ul><li>100 GiB (tamaño mínimo del grupo de capacidad: 4 TiB)</li></ul> |
| Tamaño máximo del volumen o recurso compartido | 100 TiB | Todos los niveles<br><ul><li>100 TiB (límite de grupo de capacidad de 500 TiB)</li></ul><br>Hasta 12,5 PiB por cuenta de Azure NetApp. |
| Número máximo de IOPS de volumen o de recurso compartido | Premium<br><ul><li>Hasta 100 000</li></ul><br>Estándar<br><ul><li>Hasta 10 000</li></ul> | Ultra y Premium<br><ul><li>Hasta 450 000 </li></ul><br>Estándar<br><ul><li>Hasta 320 000</li></ul> |
| Rendimiento máximo de volumen o recurso compartido | Premium<br><ul><li>Hasta 10 GiB/s</li></ul><br>Estándar<br><ul><li>Hasta 300 MiB/s</li></ul> | Ultra y Premium<br><ul><li>Hasta 4,5 GiB/s</li></ul><br>Estándar<br><ul><li>Hasta 3,2 GiB/s</li></ul> |
| Tamaño de archivo máximo | 4 TiB | 16 TiB |
| Número máximo de IOPS por archivo | Premium<br><ul><li>Hasta 8 000</li></ul><br>Estándar<br><ul><li>1,000</li></ul> | Todos los niveles<br><ul><li>Hasta el límite de volumen</li></ul> |
| Rendimiento máximo por archivo | Premium<br><ul><li>300 MiB/s (hasta 1 GiB/s con SMB multicanal)</li></ul><br>Estándar<br><ul><li>60 MiB/s</li></ul> | Todos los niveles<br><ul><li>Hasta el límite de volumen</li></ul> |
| SMB multicanal | Sí ([versión preliminar](./storage-files-smb-multichannel-performance.md)) | Sí |
| Latencia | Latencia mínima de un solo milisegundo (entre 2 ms y 3 ms para E/S pequeñas) | Latencia mínima en submilisegundos (<1 ms para E/S aleatorias)<br><br>Para más información, consulte la sección sobre [puntos de referencia de rendimiento](../../azure-netapp-files/performance-benchmarks-linux.md). |

Para más información sobre los objetivos de escalabilidad y rendimiento, consulte [Azure Files](./storage-files-scale-targets.md#azure-files-scale-targets) y [Azure NetApp Files](../../azure-netapp-files/azure-netapp-files-resource-limits.md).

## <a name="next-steps"></a>Pasos siguientes
* [Documentación de Azure Files](./index.yml)
* [Documentación sobre Azure NetApp Files](../../azure-netapp-files/index.yml)
