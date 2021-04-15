---
title: Compatibilidad de Network File System 3.0 en Azure Blob Storage (versión preliminar) | Microsoft Docs
description: Blob Storage ya es compatible con el protocolo Network File System (NFS) 3.0. Esta compatibilidad permite a los clientes de Linux montar un contenedor en Blob Storage desde una máquina virtual (VM) de Azure o un equipo que se ejecute localmente.
author: normesta
ms.subservice: blobs
ms.service: storage
ms.topic: conceptual
ms.date: 02/19/2021
ms.author: normesta
ms.reviewer: yzheng
ms.custom: references_regions
ms.openlocfilehash: 57dec4187ea42476ff1c0a993b751a7389da5639
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106490359"
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

## <a name="azure-storage-features-not-yet-supported"></a>Características de Azure Storage que aún no son compatibles

Las siguientes características de Azure Storage no se admiten cuando se habilita el protocolo NFS 3.0 en la cuenta. 

- Seguridad de Azure Active Directory (AD).

- Listas de control de acceso (ACL) tipo POSIX.

- Capacidad de habilitar la compatibilidad con NFS 3.0 en cuentas de almacenamiento existentes.

- Capacidad de deshabilitar la compatibilidad con NFS 3.0 en una cuenta de almacenamiento (después de haberla habilitado).

- Capacidad de escribir en blobs mediante API REST o SDK. 
  
## <a name="nfs-30-features-not-yet-supported"></a>Características de NFS 3.0 que aún no son compatibles

Todavía no se admiten las siguientes características de NFS 3.0.

- NFS 3.0 a través de UDP. Solo se admite NFS 3.0 a través de TCP.

- Bloqueo de archivos con Network Lock Manager (NLM). El montaje de comandos debe incluir el parámetro `-o nolock`.

- Montaje de subdirectorios. Solo puede montar el directorio raíz (contenedor).

- Enumeración de los montajes (por ejemplo: mediante el comando `showmount -a`).

- Enumeración de las exportaciones (por ejemplo: mediante el comando `showmount -e`).

- Vínculo físico

- Exportación de un contenedor como de solo lectura.

## <a name="nfs-30-clients-not-yet-supported"></a>Clientes de NFS 3.0 que aún no son compatibles

Todavía no se admiten los siguientes clientes de NFS 3.0.

- Cliente de Windows para NFS

## <a name="pricing"></a>Precios

Durante la versión preliminar, los datos almacenados en la cuenta de almacenamiento se facturan según la misma tarifa de capacidad que los cargos de Blob Storage por GB al mes. 

No se cobra ninguna transacción durante la versión preliminar. Los precios de las transacciones están sujetos a cambios y se determinarán cuando esté disponible con carácter general.

## <a name="next-steps"></a>Pasos siguientes

- Para comenzar, consulte [Montaje de Blob Storage con el protocolo Network File System (NFS) 3.0 (versión preliminar)](network-file-system-protocol-support-how-to.md).

- Para optimizar el rendimiento, consulte [Consideraciones de rendimiento de Network File System (NFS) 3.0 en Azure Blob Storage (versión preliminar)](network-file-system-protocol-support-performance.md).
