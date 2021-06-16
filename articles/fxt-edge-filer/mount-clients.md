---
title: Montaje de clientes en el clúster de Microsoft Azure FXT Edge Filer
description: Cómo las máquinas cliente NFS pueden montar la caché de almacenamiento híbrido de Azure FXT Edge Filer
author: ekpgh
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 06/20/2019
ms.author: v-erkel
ms.openlocfilehash: 1385e81b5e5a4047a23b47f39d206ea94ec4d79c
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111414571"
---
# <a name="tutorial-mount-the-cluster"></a>Tutorial: Montaje del clúster

En este tutorial le enseñamos a montar clientes NFS en el clúster de sistemas de Azure FXT Edge Filer. Los clientes montan las rutas de acceso del espacio de nombres virtual que asignó al agregar el almacenamiento de back-end.

Este tutorial le enseña lo siguiente:

> [!div class="checklist"]
>
> * Estrategias para clientes de equilibrio de carga en el intervalo de direcciones IP orientadas al cliente.
> * Cómo construir una ruta de acceso de montaje a partir de una unión de espacio de nombres y direcciones IP orientadas al cliente.
> * Los argumentos que se van a usar en un comando de montaje.

Este tutorial se completa en unos 45 minutos.

## <a name="steps-to-mount-the-cluster"></a>Pasos para montar el clúster

Siga estos pasos para conectar equipos cliente al clúster de Azure FXT Edge Filer.

1. Decida cómo equilibrar la carga del tráfico del cliente entre los nodos de su clúster. Lea la sección [Equilibrar la carga de cliente](#balance-client-load) que tiene a continuación para obtener más información.
1. Identifique la dirección IP y la ruta de acceso de unión del clúster que se va a montar.
1. Determine la ruta de acceso orientada al cliente para el montaje.
1. Ejecute el [comando de montaje](#use-recommended-mount-command-options) con los argumentos apropiados.

## <a name="balance-client-load"></a>Equilibrar la carga de cliente

Para equilibrar las solicitudes de cliente entre todos los nodos del clúster, debe montar a los clientes en el rango completo de direcciones IP orientadas al cliente. Existen varias formas de automatizar esta tarea.

Para obtener información acerca del equilibrio de carga DNS de round robin del clúster, consulte [Configure DNS for the Azure FXT Edge Filer cluster](configure-network.md#configure-dns-for-load-balancing) (Configuración de DNS para el clúster de Azure FXT Edge Filer). Para usar este método, debe mantener un servidor DNS, lo cual no se explica en estos artículos.

Un método más sencillo para instalaciones pequeñas consiste en usar un script para asignar direcciones IP durante el intervalo en el tiempo de montaje del cliente.

Otros métodos para equilibrar la carga pueden ser apropiados para sistemas grandes o complicados. Póngase en contacto con su representante de Microsoft o abra una [solicitud de soporte técnico](support-ticket.md) para obtener ayuda. (Actualmente Azure Load Balancer *no es compatible* con Azure FXT Edge Filer).

## <a name="create-the-mount-command"></a>Crear el comando de montaje

Desde el cliente, el comando ``mount`` asigna el servidor virtual (vserver) en el clúster de Azure FXT Edge Filer a una ruta de acceso en el sistema de archivos local.

El formato es ``mount <FXT cluster path> <local path> {options}``.

Hay tres elementos para el comando de montaje:

* Ruta de acceso el clúster: es una combinación de la dirección IP y la ruta de acceso de la unión del espacio de nombres descrita a continuación.
* Ruta de acceso local: es la ruta en el cliente.
* Opciones del comando de montaje: están enumeradas en [Uso de las opciones recomendadas del comando de montaje](#use-recommended-mount-command-options).

### <a name="create-the-cluster-path"></a>Creación de la ruta de acceso del clúster

La ruta de acceso del clúster es una combinación de la *dirección IP* del servidor virtual y la ruta de acceso a una *unión del espacio de nombres*. La unión del espacio de nombres es una ruta de acceso virtual que definió al [agregar el sistema de almacenamiento](add-storage.md#create-a-junction).

Por ejemplo, si usó ``/fxt/files`` como ruta de acceso del espacio de nombres, los clientes usarán *IP_address*:/fxt/files en su punto de montaje local.

![Cuadro de diálogo "Agregar nueva unión" con/avere/files en el campo de la ruta de acceso del espacio de nombres](media/fxt-mount/fxt-junction-example.png)

La dirección IP es una de las direcciones IP orientadas al cliente que se definen para el servidor virtual (vserver). Puede encontrar el rango de direcciones IP orientadas al cliente en dos lugares del panel de control del clúster:

* Tabla **VServers** (pestaña de panel) -

  ![Pestaña "Panel" del panel de control del clúster con la pestaña del servidor virtual seleccionada en la tabla de datos que está debajo del gráfico, y con la sección de dirección IP indicada](media/fxt-mount/fxt-ip-addresses-dashboard.png)

* Página de configuración de **red orientada al cliente** -

  ![Configuración> Servidor virtual > Página de configuración de red orientada al cliente en la que se indica la sección Rango de direcciones de la tabla de un servidor virtual en particular](media/fxt-mount/fxt-ip-addresses-settings.png)

Combine la dirección IP y la ruta de acceso del espacio de nombres para formar la ruta de acceso del clúster para el comando de montaje.

Ejemplo de comando de montaje de cliente: ``mount 10.0.0.12:/sd-access /mnt/fxt {options}``

### <a name="create-the-local-path"></a>Creación de la ruta de acceso local

La ruta de acceso local para el comando de montaje es decisión suya. Puede establecer cualquier estructura de ruta de acceso que quiera como parte del espacio de nombres virtual. Diseñe un espacio de nombres y la ruta de acceso local que más le convenga para el flujo de trabajo de cliente.

Para obtener más información sobre el espacio de nombres orientado al cliente, lea la [Introducción al espacio de nombres](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gns_overview.html) de la guía de configuración del clúster.

Además de las rutas de acceso, incluya las [opciones del comando de montaje](#use-recommended-mount-command-options) que se describen a continuación cuando monte cada cliente.

### <a name="use-recommended-mount-command-options"></a>Uso de las opciones recomendadas del comando de montaje

Para garantizar un montaje de cliente sin problemas, use estos valores y argumentos en el comando de montaje:

``mount -o hard,nointr,proto=tcp,mountproto=tcp,retry=30 ${VSERVER_IP_ADDRESS}:/${NAMESPACE_PATH} ${LOCAL_FILESYSTEM_MOUNT_POINT}``

| Configuración requerida | Descripción |
--- | ---
``hard`` | Los montajes leves en el clúster de Azure FXT Edge Filer están asociados con errores de la aplicación y la posible pérdida de datos.
``proto=netid`` | Esta opción es compatible con el control adecuado de los errores de la red NFS.
``mountproto=netid`` | Esta opción es compatible con el control adecuado de errores de red para las operaciones de montaje.
``retry=n`` | Configure ``retry=30`` para evitar errores de montaje transitorios. (Se recomienda usar un valor diferente en los montajes de primer plano).

| Configuración preferida  | Descripción |
--- | ---
``nointr``            | Si los clientes usan kernel de sistemas operativos más antiguos (anteriores a abril de 2008) que admitan esta opción, úselos. La opción "intr" es el valor predeterminado.

## <a name="next-steps"></a>Pasos siguientes

Después de montar los clientes, puede probar el flujo de trabajo y empezar a trabajar con el clúster.

Si necesita mover datos a un nuevo archivador principal en la nube, aproveche la estructura de la memoria caché mediante el uso de ingesta paralela de datos. En [Mover datos al clúster de vFXT](../avere-vfxt/avere-vfxt-data-ingest.md) se describen algunas estrategias. (Avere vFXT for Azure es un producto basado en la nube que usa una tecnología de almacenamiento en caché muy similar a Azure FXT Edge Filer).

Consulte [Monitor Azure FXT Edge Filer hardware status](monitor.md) (Supervisión del estado de hardware de Azure FXT Edge Filer) si necesita solucionar problemas de hardware.