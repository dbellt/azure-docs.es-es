---
title: Compatibilidad de Network File System 3.0 en Azure Blob Storage (versión preliminar) | Microsoft Docs
description: Blob Storage ya es compatible con el protocolo Network File System (NFS) 3.0. Esta compatibilidad permite a los clientes de Linux montar un contenedor en Blob Storage desde una máquina virtual (VM) de Azure o un equipo que se ejecute localmente.
author: normesta
ms.subservice: blobs
ms.service: storage
ms.topic: conceptual
ms.date: 04/28/2021
ms.author: normesta
ms.reviewer: yzheng
ms.custom: references_regions
ms.openlocfilehash: f50255510d5f4716f181a8e2b51cb5fa3998062d
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109715089"
---
# <a name="network-file-system-nfs-30-protocol-support-in-azure-blob-storage-preview"></a>Compatibilidad del protocolo Network File System (NFS) 3.0 en Azure Blob Storage (versión preliminar)

Blob Storage ya es compatible con el protocolo Network File System (NFS) 3.0. De este modo, proporciona compatibilidad con el sistema de archivos de Linux en cuanto a los precios y la escala de almacenamiento de objetos, y permite a los clientes de Linux montar un contenedor en el Blob Storage desde una máquina virtual (VM) de Azure o un equipo local. 

> [!NOTE]
> La compatibilidad con el protocolo NFS 3.0 en Azure Blob Storage se encuentra en versión preliminar pública. Este protocolo admite cuentas de almacenamiento GPV2 con un rendimiento de nivel estándar en las siguientes regiones: Este de Australia, Centro de Corea del Sur, Este de EE. UU. y Centro-sur de EE. UU. La versión preliminar también admite blobs en bloques con el nivel de rendimiento Premium en todas las regiones públicas.

Siempre es un reto ejecutar cargas de trabajo heredadas a gran escala, como la informática de alto rendimiento (HPC) en la nube. Una razón es que las aplicaciones suelen usar protocolos de archivo tradicionales, como NFS o Bloque de mensajes del servidor (SMB), para acceder a los datos. Además, los servicios nativos de almacenamiento en la nube se centran en el almacenamiento de objetos que tienen un espacio de nombres plano y metadatos extensivos, en lugar de sistemas de archivos que proporcionan un espacio de nombres jerárquico y operaciones de metadatos eficientes. 

Blob Storage ahora admite un espacio de nombres jerárquico y, al combinarse con la compatibilidad con el protocolo NFS 3.0, Azure facilita la ejecución de aplicaciones heredadas además del almacenamiento de objetos en la nube a gran escala. 

## <a name="applications-and-workloads-suited-for-this-feature"></a>Aplicaciones y cargas de trabajo adecuadas para esta característica

La característica del protocolo NFS 3.0 es más adecuada para el procesamiento de cargas de trabajo de alto rendimiento, gran escala y lectura intensiva, como el procesamiento de medios, las simulaciones de riesgo y la secuenciación genómica. Debe considerar la posibilidad de usar esta característica para cualquier otro tipo de carga de trabajo que use varios lectores y muchos subprocesos, que requieren un ancho de banda alto. 

## <a name="nfs-30-and-the-hierarchical-namespace"></a>NFS 3.0 y el espacio de nombres jerárquico

La compatibilidad con el protocolo NFS 3.0 requiere que los blobs se organicen en un espacio de nombres jerárquico. Puede habilitar un espacio de nombres jerárquico al crear una cuenta de almacenamiento. La capacidad de usar un espacio de nombres jerárquico se introdujo con Azure Data Lake Storage Gen2. Organiza objetos (archivos) en una jerarquía de directorios y subdirectorios de la misma manera en que se organiza el sistema de archivos en el equipo.  El espacio de nombres jerárquico se escala linealmente y no degrada el rendimiento ni la capacidad de datos. Los diferentes protocolos se extienden desde el espacio de nombres jerárquico. El protocolo NFS 3.0 es uno de estos protocolos disponibles.   

> [!div class="mx-imgBorder"]
> ![Espacio de nombres jerárquico](./media/network-protocol-support/hierarchical-namespace-and-nfs-support.png)
  
## <a name="data-stored-as-block-blobs"></a>Datos almacenados como blobs en bloques

Si habilita la compatibilidad con el protocolo NFS 3.0, todos los datos de la cuenta de almacenamiento se almacenarán como blobs en bloques. Los blobs en bloques están optimizados para procesar de manera eficaz grandes cantidades de datos de lectura intensiva. Los blobs en bloques están formados por bloques. Cada bloque se identifica mediante un identificador de bloque. Un blob en bloques puede incluir hasta 50 000 bloques. Cada bloque de un blob en bloques puede tener un tamaño diferente, hasta el tamaño máximo permitido para la versión del servicio que usa la cuenta.

Cuando la aplicación realiza una solicitud mediante el protocolo NFS 3.0, esa solicitud se traduce en una combinación de operaciones de blobs en bloques. Por ejemplo, las solicitudes de llamada a procedimiento remoto (RPC) de lectura NFS 3.0 se traducen en la operación [Obtener blob](/rest/api/storageservices/get-blob). Las solicitudes RPC de escritura de NFS 3.0 se convierten en una combinación de [Get Block List](/rest/api/storageservices/get-block-list), [Put Block](/rest/api/storageservices/put-block) y [Put Block List](/rest/api/storageservices/put-block-list).

## <a name="general-workflow-mounting-a-storage-account-container"></a>Flujo de trabajo general: montaje de un contenedor de cuenta de almacenamiento

Los clientes de Linux pueden montar un contenedor en Blob Storage desde una máquina virtual (VM) de Azure o un equipo local. Para montar un contenedor de cuenta de almacenamiento, tendrá que realizar estas acciones.

1. Registe la característica de protocolo NFS 3.0 con la suscripción.

2. Compruebe si la característica está registrada.

3. Cree una instancia de Azure Virtual Network (VNet).

4. Configure la seguridad de red.

5. Cree y configure una cuenta de almacenamiento que acepte tráfico solo desde la red virtual.

6. Cree un contenedor en la cuenta de almacenamiento.

7. Monte el contenedor.

Para obtener instrucciones paso a paso, consulte [Montaje de Blob Storage con el protocolo Network File System (NFS) 3.0 (versión preliminar)](network-file-system-protocol-support-how-to.md).

> [!IMPORTANT]
> Es importante completar estas tareas en orden. No se pueden montar contenedores creados antes de habilitar el protocolo NFS 3.0 en la cuenta. Además, después de haber habilitado el protocolo NFS 3.0 en la cuenta, no puede deshabilitarlo.

## <a name="network-security"></a>Seguridad de las redes

La cuenta de almacenamiento debe incluirse en una red virtual. Una red virtual permite a los clientes conectarse de forma segura a su cuenta de almacenamiento. La única manera de proteger los datos en su cuenta es mediante una red virtual y otra configuración de seguridad de red. Las demás herramientas que se usan para proteger los datos, incluidas la autorización de claves de cuenta, la seguridad de Azure Active Directory (AD) y las listas de control de acceso (ACL), no se admiten todavía en cuentas que tienen habilitada la compatibilidad con el protocolo NFS 3.0. 

Para más información, consulte [Recomendaciones de seguridad de red para Blob Storage](security-recommendations.md#networking).

## <a name="supported-network-connections"></a>Conexiones de red compatibles

Un cliente puede conectarse a través de un [punto de conexión privado](../common/storage-private-endpoints.md) o público y desde cualquiera de las siguientes ubicaciones de red:

- La red virtual que configure para la cuenta de almacenamiento. 

  En este artículo, nos referiremos a esa red virtual como *red virtual principal*. Para más información, consulte [Concesión de acceso desde una red virtual](../common/storage-network-security.md#grant-access-from-a-virtual-network).

- Una red virtual emparejada que se encuentre en la misma región que la red virtual principal.

  Tendrá que configurar la cuenta de almacenamiento para permitir el acceso a esta red virtual emparejada. Para más información, consulte [Concesión de acceso desde una red virtual](../common/storage-network-security.md#grant-access-from-a-virtual-network).

- Una red local que esté conectada a la red virtual principal mediante [VPN Gateway](../../vpn-gateway/vpn-gateway-about-vpngateways.md) o una [puerta de enlace de ExpressRoute](../../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md). 

  Para más información, consulte [Configuración del acceso desde redes locales](../common/storage-network-security.md#configuring-access-from-on-premises-networks).

- Una red local que esté conectada a una red emparejada.

  Esta acción puede realizarse mediante [VPN Gateway](../../vpn-gateway/vpn-gateway-about-vpngateways.md) o una [puerta de enlace de ExpressRoute](../../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md) junto con [tránsito de puerta de enlace](/azure/architecture/reference-architectures/hybrid-networking/vnet-peering#gateway-transit). 

> [!IMPORTANT]
> Si se va a conectar desde una red local, asegúrese de que el cliente permite la comunicación saliente a través de los puertos 111 y 2048. El protocolo NFS 3.0 usa estos puertos.

<a id="azure-storage-features-not-yet-supported"></a>

## <a name="support-for-azure-storage-features"></a>Compatibilidad con las características de Azure Storage

En la tabla siguiente se muestra el nivel actual de compatibilidad con características de Azure Storage en las cuentas que tienen habilitada la característica NFS 3.0. 

El estado de los elementos que aparecen en esta tabla va a cambiar con el tiempo a medida que se siga ampliando la compatibilidad.

| Característica de Azure Storage | Premium | Estándar |Característica de Azure Storage | Premium | Estándar |
|-----------------|---------|----------|----------------|---------|----------|
| [API REST de Blob service](/rest/api/storageservices/blob-service-rest-api)  | ✔️ |  ⛔ | [API REST de Azure Data Lake Storage](/rest/api/storageservices/data-lake-storage-gen2) | ⛔ |    ⛔ |
| [Niveles de acceso para Azure Blob Storage](storage-blob-storage-tiers.md) |    ✔️ |    ⛔ | [Etiquetas de índice de blobs](storage-blob-index-how-to.md) |   ⛔ | ⛔ |
| [Administración del ciclo de vida de Azure Blob Storage](storage-lifecycle-management-concepts.md) | ✔️  |   ⛔ | [Registro de Azure Storage Analytics](../common/storage-analytics-logging.md?toc=/azure/storage/blobs/toc.json) | ⛔ |   ⛔ |
|  [Inventario de blobs de Azure Storage](blob-inventory.md) |  ✔️  |   ⛔ | [Fuente de cambios](storage-blob-change-feed.md) |    ⛔ | ⛔ |
| [Azure Monitor](monitor-blob-storage.md) |    ✔️ |    ⛔ | [Control de versiones de blobs](versioning-enable.md) | ⛔ |   ⛔ |
| [Instantáneas de blob](snapshots-overview.md) | ✔️  |   ⛔ | [Restauración a un momento dado para blobs en bloques](point-in-time-restore-overview.md) | ⛔ |    ⛔ |
| [Puntos de conexión privados](../common/storage-private-endpoints.md?toc=/azure/storage/blobs/toc.json) | ✔️  | ⛔ | [Integración de Azure Backup](/azure/backup/blob-backup-overview) | ⛔ |    ⛔ |
| [Puntos de conexión de servicio](../../virtual-network/virtual-network-service-endpoints-overview.md) | ✔️  |  ⛔ | [Eliminación temporal de contenedores](soft-delete-container-overview.md) |   ⛔ | ⛔ |
| [Reglas de firewall](../common/storage-network-security.md?toc=/azure/storage/blobs/toc.json) | ✔️  | ⛔ | [Eliminación temporal para blobs](soft-delete-blob-overview.md) | ⛔ | ⛔ |
| [Impedir la autorización con clave compartida](../common/shared-key-authorization-prevent.md)  | ✔️ |    ⛔ | [Seguimiento de la hora del último acceso para la administración del ciclo de vida](storage-lifecycle-management-concepts.md#move-data-based-on-last-accessed-date-preview) |  ⛔|  ⛔ |
| [Claves administradas por el cliente para el cifrado de Azure Storage](../common/customer-managed-keys-overview.md) |   ✔️ |    ⛔ | [Claves proporcionadas por el cliente para el cifrado de Azure Storage](encryption-customer-provided-keys.md)  |  ⛔ | ⛔ |
| [Almacenamiento de blobs inmutable](storage-blob-immutable-storage.md) | ✔️    | ⛔ | [Hospedaje de sitios web estáticos](storage-blob-static-website.md) | ⛔  |    ⛔ |
| [Blobs en anexos](storage-blobs-introduction.md#blobs) | ✔️   |  ⛔ | [Blobs en páginas](storage-blobs-introduction.md#blobs) | ⛔ | ⛔ |
| [Seguridad de Azure Active Directory (AD).](../common/storage-auth-aad.md?toc=/azure/storage/blobs/toc.json) | ⛔ | ⛔ | [Ámbitos de cifrado](encryption-scope-overview.md)  |    ⛔ | ⛔ |
| [Replicación de objetos para blobs en bloques](object-replication-overview.md) | ⛔  |   ⛔ | [Conmutación por error de cuenta administrada por el cliente](../common/storage-disaster-recovery-guidance.md?toc=/azure/storage/blobs/toc.json) | ⛔ |    ⛔ |
| [Eventos de Blob Storage](storage-blob-event-overview.md)| ⛔ |    ⛔ 

  
## <a name="known-issues"></a>Problemas conocidos

- La compatibilidad con NFS 3.0 no se puede habilitar en las cuentas de almacenamiento existentes.

- La compatibilidad con NFS 3.0 no se puede deshabilitar en una cuenta de almacenamiento después de haberla habilitado.

-  Los archivos no se pueden ver en Azure Portal o en el Explorador de Azure Storage. Para enumerar archivos y directorios, [monte un contenedor de Blob Storage mediante el protoloco NFS 3.0](network-file-system-protocol-support-how-to.md) o use la [API REST de Blob service](/rest/api/storageservices/blob-service-rest-api).

### <a name="nfs-30-features-not-yet-supported"></a>Características de NFS 3.0 que aún no son compatibles

Todavía no se admiten las siguientes características de NFS 3.0.

- NFS 3.0 a través de UDP. Solo se admite NFS 3.0 a través de TCP.

- Bloqueo de archivos con Network Lock Manager (NLM). El montaje de comandos debe incluir el parámetro `-o nolock`.

- Montaje de subdirectorios. Solo puede montar el directorio raíz (contenedor).

- Enumeración de los montajes (por ejemplo: mediante el comando `showmount -a`).

- Enumeración de las exportaciones (por ejemplo: mediante el comando `showmount -e`).

- Vínculo físico

- Exportación de un contenedor como de solo lectura.

### <a name="nfs-30-clients-not-yet-supported"></a>Clientes de NFS 3.0 que aún no son compatibles

Todavía no se admiten los siguientes clientes de NFS 3.0.

- Cliente de Windows para NFS

## <a name="pricing"></a>Precios

Durante la versión preliminar, los datos almacenados en la cuenta de almacenamiento se facturan según la misma tarifa de capacidad que los cargos de Blob Storage por GB al mes. 

No se cobra ninguna transacción durante la versión preliminar. Los precios de las transacciones están sujetos a cambios y se determinarán cuando esté disponible con carácter general.

## <a name="next-steps"></a>Pasos siguientes

- Para comenzar, consulte [Montaje de Blob Storage con el protocolo Network File System (NFS) 3.0 (versión preliminar)](network-file-system-protocol-support-how-to.md).

- Para optimizar el rendimiento, consulte [Consideraciones de rendimiento de Network File System (NFS) 3.0 en Azure Blob Storage (versión preliminar)](network-file-system-protocol-support-performance.md).
