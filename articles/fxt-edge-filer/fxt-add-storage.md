---
title: 'Tutorial: Incorporación de almacenamiento a un clúster de Azure FXT Edge Filer'
description: Cómo configurar el almacenamiento de back-end y el pseudoespacio de nombres orientado al cliente para Azure FXT Edge Filer
author: ekpgh
ms.author: v-erkel
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 06/20/2019
ms.openlocfilehash: 87d6d24f1cc325909249c7ad65c2b63ffe228a2a
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110456285"
---
# <a name="tutorial-add-back-end-storage-and-configure-the-virtual-namespace"></a>Tutorial: Adición de almacenamiento de back-end y configuración del espacio de nombres virtual

En este tutorial se explica cómo agregar un almacenamiento de back-end para la caché y cómo configurar el sistema de archivos virtual orientado al cliente.

El clúster se conecta a los sistemas de almacenamiento de back-end para obtener acceso a la solicitud de los clientes de datos y para almacenar los cambios de forma más permanente que en la memoria caché.

El espacio de nombres es un pseudosistema de archivos orientado al cliente que le permite intercambiar el almacenamiento de back-end sin tener que cambiar los flujos de trabajo del lado del cliente.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Cómo agregar el almacenamiento de back-end al clúster de Azure FXT Edge Filer
> * Cómo definir la ruta de acceso de orientada al cliente para el almacenamiento

## <a name="about-back-end-storage"></a>Acerca del almacenamiento de back-end

El clúster de Azure FXT Edge Filer usa una definición del *archivador básico* para vincular un sistema de almacenamiento de back-end al clúster de FXT.

Azure FXT Edge Filer es compatible con varios sistemas de hardware NAS populares y puede usar contenedores vacíos de Azure Blob u otro almacenamiento en la nube.

Los contenedores de almacenamiento en la nube deben estar vacíos cuando se agregan para que el sistema operativo FXT pueda administrar completamente todos los datos en el volumen de almacenamiento en la nube. Puede mover sus datos al contenedor en la nube después de agregar ese contenedor al clúster como un archivador básico.

Use el Panel de control para agregar un archivador básico a su sistema.

> [!NOTE]
>
> Si quiere usar Amazon AWS o el almacenamiento en la nube de Google, debe instalar una licencia de característica de FlashCloud<sup>TM</sup>. Póngase en contacto con su representante de Microsoft para obtener una clave de licencia y siga las instrucciones de la guía de configuración heredada [Adding or removing feature licenses](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/install_licenses.html#install-licenses) (Agregar o eliminar licencias de características).
>
> La compatibilidad con Azure Blob Storage se incluye en la licencia del software Azure FXT Edge Filer.

Para obtener información detallada sobre cómo agregar archivadores básicos, lea estas secciones de la guía de configuración de clústeres:

* Para obtener más información sobre cómo elegir y prepararse para agregar un archivador básico, lea [Working With Core Filers](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/core_filer_overview.html#core-filer-overview) (Trabajar con archivadores básicos).
* Para obtener los requisitos previos detallados y las instrucciones paso a paso, lea estos artículos:

  * [Adding a New NAS Core Filer](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_nas.html#create-core-filer-nas) (Agregar un nuevo archivador básico de NAS)
  * [Adding a New Cloud Core Filer](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_cloud.html#create-core-filer-cloud) (Agregar un nuevo archivador básico en la nube)

Después de agregar un archivador básico, puede actualizar su configuración en la página de configuración de los detalles del archivador básico.

## <a name="add-a-core-filer"></a>Añadir un archivador básico

Defina un archivador de núcleo haciendo clic en el botón **Create** (Crear) en la página de configuración de **Core Filer** > **Manage Core Filers** (Archivador básico > Administrar archivadores básicos).

![Haga clic en el botón Crear en la lista de archivadores básicos de la página para administrar archivadores básicos](media/fxt-cluster-config/create-core-filer-button.png)

El asistente para **agregar un archivador básico nuevo** le guiará en el proceso de creación de un archivador básico que se vincula a su almacenamiento de back-end. La guía de configuración de clústeres tiene descripciones paso a paso del proceso, que es diferente para el almacenamiento NFS/NAS y para el almacenamiento en la nube (los enlaces están arriba).

Las subtareas incluyen:

* Especificar el tipo de archivador básico (NAS o en la nube)

  ![Primera página del asistente de archivador básico nuevo del hardware NAS. La opción del "archivador básico en la nube" está deshabilitada y muestra un mensaje de error sobre la licencia que falta.](media/fxt-cluster-config/new-nas-1.png)

* Establecer el nombre del archivador básico. Elija un nombre que ayude a los administradores de clústeres a comprender qué sistema de almacenamiento representa.

* Para los archivadores básicos de NAS, proporcione el nombre de dominio completo (FQDN) o la dirección IP. Se recomienda usar el FQDN en todos los archivadores básicos y se requiere para el acceso de SMB.

* Seleccione una directiva de caché: la segunda página del asistente enumera las directivas de caché disponibles para el nuevo archivador básico. Para obtener más información, lea la [sección de directivas de caché de la guía de configuración de clústeres](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_manage_cache_policies.html).

  ![Segunda página de un asistente de archivador básico nuevo de hardware NAS; el menú desplegable "Directivas de caché" está abierto y muestra varias opciones deshabilitadas y tres opciones de directivas de caché válidas (omisión, lectura del almacenamiento en caché y lectura o escritura del almacenamiento en caché).](media/fxt-cluster-config/new-nas-choose-cache-policy.png)

* Para el almacenamiento en la nube, debe especificar el servicio en la nube y las credenciales de acceso, entre otros parámetros. Para obtener más información, lea [Cloud service and protocol](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/new_core_filer_cloud.html#cloud-service-and-protocol) (Servicio y protocolo en la nube) en la guía de configuración del clúster.

  ![Información del archivador básico en la nube en el asistente de archivador básico nuevo](media/fxt-cluster-config/new-core-filer-cloud3.png)
  
  Si ya ha agregado credenciales de acceso a la nube para este clúster, aparecerán en la lista. Actualice y agregue las credenciales en la página de configuración **Cluster** > **Cloud Credentials** (Clúster > Credenciales de la nube).

Después de completar todas las configuraciones requeridas en el asistente, haga clic en el botón **Add Filer** (Agregar archivador) para enviar el cambio.

Después de unos momentos, el sistema de almacenamiento aparece en la lista de archivadores básicos de la **panel** y se puede acceder a él a través de las páginas de configuración del archivador básico.

![El archivador básico "Flurry-NAS" en la página de configuración Administrar archivadores básicos, con la vista de detalles del archivador expandida](media/fxt-cluster-config/core-filer-in-manage-page.png)

Al archivador básico de esta captura de pantalla le falta un elemento vserver. Debe vincular el archivador básico a un elemento vserver y crear una unión para que los clientes puedan obtener acceso al almacenamiento. Estos pasos se describen a continuación en [Configurar el espacio de nombres](#configure-the-namespace).

## <a name="configure-the-namespace"></a>Configurar el espacio de nombres

El clúster de Azure FXT Edge Filer crea un sistema de archivos virtual denominado *espacio de nombres de clúster* que simplifica el acceso del cliente a los datos almacenados en diversos sistemas de back-end. Como los clientes solicitan archivos mediante una ruta de acceso virtual, los sistemas de almacenamiento pueden agregarse o reemplazarse sin tener que cambiar el flujo de trabajo del cliente.

El espacio de nombres del clúster también le permite presentar los sistemas de almacenamiento en la nube y NAS en una estructura de archivos similar.

Los elementos vserver del clúster mantienen el espacio de nombres y el contenido que se entrega a los clientes. Existen dos pasos para crear el espacio de nombres del clúster:

1. Crear un vserver
1. Configure uniones entre los sistemas de almacenamiento de back-end y las rutas del sistema de archivos orientados al cliente.

### <a name="create-a-vserver"></a>Crear un vserver

Los vserver son servidores de archivos virtuales que controlan cómo fluyen los datos entre el cliente y los archivadores básicos del clúster:

* Los vserver hospedan direcciones IP orientadas al cliente
* Los vserver crean el espacio de nombres y definen las uniones que asignan la estructura de directorio virtual orientada al cliente a las exportaciones en el almacenamiento de back-end
* Los vserver aplican controles de acceso a archivos, incluidas las directivas de exportación del archivador básico y los sistemas de autenticación de usuarios
* Los vserver proporcionan infraestructura SMB

Antes de comenzar a configurar un vserver de clúster, lea la documentación adicional que tiene vinculada y consulte a su representante de Microsoft para que le ayude a entender el espacio de nombres y los vserver. Si usa redes VLAN, [créelas](fxt-configure-network.md#adjust-network-settings) antes de crear el vserver.

Estas secciones de la guía de configuración del clúster le ayudarán a familiarizarse con las características del vserver FXT y del espacio de nombres global:

* [Creating and Working with VServers](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/settings_overview.html#creating-and-working-with-vservers) (Crear vserver y trabajar con ellos)
* [Using a Global Namespace](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gns_overview.html) (Usar un espacio de nombres global)
* [Creating a VServer](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_vserver_manage.html#creating-a-vserver) (crear un vserver)

Necesita al menos un vserver para su clúster.

Para crear un vserver, necesita la información siguiente:

* El nombre para establecer el vserver

* El rango de direcciones IP orientadas al cliente que manejará el vserver

  Recuerde que debe proporcionar un único rango de direcciones IP contiguas cuando cree el vserver. Puede agregar más direcciones más adelante si usa la página de configuración de la **red orientada al cliente**.

* Si su red tiene VLAN, decida qué red VLAN usará en este vserver

Use la página de configuración **VServer** > **Administrar VServers** para crear un nuevo vserver. Para obtener más información, lea [Creating a VServer](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_vserver_manage.html#creating-a-vserver) (Crear un VServer) en la guía de configuración del clúster.

![ventana emergente para crear un nuevo vserver](media/fxt-cluster-config/new-vserver.png)

### <a name="create-a-junction"></a>Crear una unión

Una *unión* asigna una ruta de almacenamiento de back-end al espacio de nombres visible para el cliente.

Puede usar este sistema para simplificar la ruta que se usa en los puntos de montaje del cliente y para escalar la capacidad sin problemas, ya que una ruta de acceso virtual puede acomodar el almacenamiento de varios archivadores básicos.

![Página del asistente para agregar una nueva unión con la configuración completada](media/fxt-cluster-config/add-junction-full.png)

Consulte [**VServer** > **Espacio de nombres**](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/gui_namespace.html) en la guía de configuración de clústeres para obtener detalles completos sobre cómo crear una unión de espacio de nombres.

![Página de configuración VServer > Espacio de nombres que muestra los detalles de una unión](media/fxt-cluster-config/namespace-populated.png)

## <a name="configure-export-rules"></a>Configurar las reglas de exportación

Después de obtener un vserver y un archivador básico, debe personalizar las reglas y directivas de exportación que controlan cómo los clientes pueden obtener acceso a los archivos de las exportaciones del archivador básico.

Primero, use la página **VServer** > **Reglas de exportación** para agregar nuevas reglas, modificar la directiva predeterminada o crear su propia directiva de exportación personalizada.

En segundo lugar, use la página **VServer** > **Exportar directivas** para aplicar la directiva personalizada a las exportaciones del archivador básico cuando se acceda a través de ese vserver.

Lea el artículo de la guía de configuración del clúster [Controlling Access to Core Filer Exports](https://azure.github.io/Avere/legacy/ops_guide/4_7/html/export_rules_overview.html) (Controlar el acceso a las exportaciones del archivador básico) para obtener más información.

## <a name="next-steps"></a>Pasos siguientes

Después de agregar almacenamiento y configurar el espacio de nombres orientado al cliente, complete la configuración inicial de su clúster: 

> [!div class="nextstepaction"]
> [Configuración de la red en clúster](fxt-configure-network.md)
