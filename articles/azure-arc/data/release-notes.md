---
title: 'Servicios de datos habilitados para Azure Arc: notas de la versión'
description: Notas de la versión más recientes
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.date: 04/09/2021
ms.topic: conceptual
ms.openlocfilehash: 1fe5974bafddcb4e474ef59a062836e071ab9461
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107304926"
---
# <a name="release-notes---azure-arc-enabled-data-services-preview"></a>Notas de la versión: servicios de datos habilitados para Azure Arc (versión preliminar)

En este artículo se resaltan las funcionalidades, las características y las mejoras que se han publicado u optimizado recientemente para los servicios de datos habilitados para Azure Arc. 

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="march-2021"></a>Marzo de 2021

La versión de marzo de 2021 se introdujo inicialmente el 5 de abril de 2021 y las fases finales de la versión se completaron el 9 de abril de 2021.

Revise las limitaciones de esta versión en [Problemas conocidos: Servicios de datos habilitados para Azure Arc (versión preliminar)](known-issues.md).

Número de versión de la CLI de datos de Azure (`azdata`): 20.3.2. Puede instalar `azdata` desde [Instalación de Azure Data CLI (`azdata`)](/sql/azdata/install/deploy-install-azdata).

### <a name="data-controller"></a>Controlador de datos

- Implemente el controlador de datos de Servicios de datos habilitados para Azure Arc en modo de conexión directa desde el portal. Comience desde [Implementación del controlador de datos: requisitos previos para el modo de conexión directa](deploy-data-controller-direct-mode-prerequisites.md).

### <a name="azure-arc-enabled-postgresql-hyperscale"></a>Hiperescala de PostgreSQL habilitada para Azure Arc

Las dos definiciones de recurso personalizado (CRD) para PostgreSQL se han consolidado en una sola. Consulte la tabla siguiente.

|Release |CRD |
|-----|-----|
|Febrero de 2021 y anteriores| postgresql-11s.arcdata.microsoft.com<br/>postgresql-12s.arcdata.microsoft.com |
|A partir de marzo de 2021 | postgresqls.arcdata.microsoft.com

Eliminará las definiciones de recurso personalizado anteriores a medida que limpie las instalaciones anteriores. Consulte [Limpieza de instalaciones anteriores](create-data-controller-using-kubernetes-native-tools.md#cleanup-from-past-installations).

### <a name="azure-arc-enabled-sql-managed-instance"></a>SQL Managed Instance para Azure Arc

- Ahora puede crear una instancia administrada de SQL desde Azure Portal en el modo de conexión directa.

- Ahora puede restaurar una base de datos a SQL Managed Instance con tres réplicas y se agregará automáticamente al grupo de disponibilidad. 

- Ahora puede conectarse a un punto de conexión secundario de solo lectura en las instancias de SQL Managed Instance implementadas con tres réplicas. Use `azdata arc sql endpoint list` para ver el punto de conexión secundario de solo lectura.

### <a name="known-issues"></a>Problemas conocidos

- En el modo de conexión directa, la carga de uso, las métricas y los registros mediante `azdata arc dc upload` están bloqueados actualmente. El uso se carga automáticamente. La carga del controlador de datos creado en modo de conexión indirecta debe seguir funcionando.
- La implementación del controlador de datos en modo directo solo se puede realizar desde Azure Portal y no está disponible en herramientas cliente como azdata, Azure Data Studio o kubectl.
- La implementación de SQL Managed Instance habilitado para Azure Arc en modo directo solo se puede realizar desde Azure Portal y no está disponible desde herramientas como azdata, Azure Data Studio o kubectl.
- La implementación de Hiperescala de PostgeSQL habilitada para Azure Arc en modo directo no está disponible actualmente.
- La carga automática de datos de uso en modo de conexión directa no se realizará correctamente si se usa el proxy mediante `–proxy-cert <path-t-cert-file>`.

## <a name="february-2021"></a>Febrero de 2021

### <a name="new-capabilities-and-features"></a>Nuevas características y funcionalidades

Número de versión de la CLI de datos de Azure (`azdata`): 20.3.1. Puede instalar `azdata` desde [Instalación de Azure Data CLI (`azdata`)](/sql/azdata/install/deploy-install-azdata).

Otras actualizaciones incluyen:

- SQL Managed Instance para Azure Arc
   - Alta disponibilidad con grupos de disponibilidad AlwaysOn

- Hiperescala de PostgreSQL habilitada para Azure Arc (Azure Data Studio): 
   - En la página de información general se muestra el estado del grupo de servidores detallado por nodo.
   - Ahora hay disponible una nueva página de propiedades para mostrar más detalles sobre el grupo de servidores.
   - Configuración de parámetros del motor de Postgres desde la página **Node Parameters** (Parámetros del nodo).

Para conocer los problemas asociados con esta versión, consulte [Problemas conocidos: servicios de datos habilitados para Azure Arc (versión preliminar)](known-issues.md).

## <a name="january-2021"></a>Enero de 2021

### <a name="new-capabilities-and-features"></a>Nuevas características y funcionalidades

Número de versión de la CLI de datos de Azure (`azdata`): 20.3.0. Puede instalar `azdata` desde [Instalación de Azure Data CLI (`azdata`)](/sql/azdata/install/deploy-install-azdata).

Otras actualizaciones incluyen:
- Portal localizado disponible en 17 idiomas nuevos.
- Cambios menores en los archivos .yaml nativos de Kube.
- Nuevas versiones de Grafana y Kibana.
- Problemas con entornos de Python cuando se usa azdata en cuadernos resueltos en Azure Data Studio.
- La extensión pg_audit ahora está disponible para Hiperescala de PostgreSQL.
- Ya no es necesario un identificador de copia de seguridad cuando se realiza una restauración completa de una base de datos de Hiperescala de PostgreSQL.
- El estado (estado de mantenimiento) se indica para cada una de las instancias de PostgreSQL que forman un grupo de servidores

   En versiones anteriores, el estado se agregaba en el nivel de grupo de servidores y no en el nivel de nodo de PostgreSQL.

- Las implementaciones de PostgreSQL ahora respetan los parámetros de tamaño del volumen indicados en los comandos de creación.
- Ahora se respetan los parámetros de versión del motor al editar un grupo de servidores
- Ha cambiado la convención de nomenclatura de los pods para Hiperescala de PostgreSQL habilitada para Azure Arc.

    Ahora tiene el formato `ServergroupName{c, w}-n`. Por ejemplo, un grupo de servidores con tres nodos, un nodo de coordinación y dos nodos de trabajo, se representa como:
   - `Postgres01c-0` (nodo de coordinación)
   - `Postgres01w-0` (nodo de trabajo)
   - `Postgres01w-1` (nodo de trabajo)

## <a name="december-2020"></a>Diciembre de 2020

### <a name="new-capabilities--features"></a>Nuevas características y funcionalidades

Número de versión de la CLI de datos de Azure (`azdata`): 20.2.5. Puede instalar `azdata` desde [Instalación de Azure Data CLI (`azdata`)](/sql/azdata/install/deploy-install-azdata).

Vea los puntos de conexión para SQL Managed Instance e Hiperescala de PostgreSQL mediante la CLI de Azure Data (`azdata`) con los comandos `azdata arc sql endpoint list` y `azdata arc postgres endpoint list`.

Edite solicitudes y límites de recursos de SQL Managed Instance (CPU y núcleos de CPU) mediante Azure Data Studio.

Hiperescala de PostgreSQL habilitada para Azure Arc ahora admite la restauración a un momento dado, además de la restauración de copia de seguridad completa para las versiones 11 y 12 de PostgreSQL. La funcionalidad de restauración a un momento dado le permite indicar una fecha y hora específicas en las que realizar la restauración.

Ha cambiado la convención de nomenclatura de los pods para Hiperescala de PostgreSQL habilitada para Azure Arc. Ahora tiene el formato: NombreDeGrupoDeServidores{r, s}-_n_. Por ejemplo, un grupo de servidores con tres nodos, un nodo de coordinación y dos nodos de trabajo, se representa como:
- `postgres02r-0` (nodo de coordinación)
- `postgres02s-0` (nodo de trabajo)
- `postgres02s-1` (nodo de trabajo)

### <a name="breaking-change"></a>Cambio importante

#### <a name="new-resource-provider"></a>Nuevo proveedor de recursos

En esta versión se incluye un [proveedor de recursos](../../azure-resource-manager/management/azure-services-resource-providers.md) actualizado denominado `Microsoft.AzureArcData`. Antes de poder usar esta característica, debe registrar este proveedor de recursos. 

Para registrar este proveedor de recursos: 

1. En Azure Portal, seleccione **Suscripciones**. 
2. Elija una suscripción
3. En **Configuración**, seleccione **Proveedores de recursos**. 
4. Busque `Microsoft.AzureArcData` y seleccione **Registrar**. 

Puede revisar los pasos detallados en [Proveedores y tipos de recursos de Azure](../../azure-resource-manager/management/resource-providers-and-types.md). Este cambio también quita todos los recursos de Azure existentes que haya cargado en Azure Portal. Para usar el proveedor de recursos, debe actualizar el controlador de datos y usar la CLI de `azdata` más reciente.  

### <a name="platform-release-notes"></a>Notas de la versión de la plataforma

#### <a name="direct-connectivity-mode"></a>Modo de conectividad directa

En esta versión se presenta el modo de conectividad directa. El modo de conectividad directa permite al controlador de datos cargar automáticamente la información de uso en Azure. Como parte de la carga de uso, el recurso de controlador de datos de Arc se crea de forma automática en el portal, si todavía no se ha creado mediante la carga de `azdata`.  

Puede especificar la conectividad directa al crear el controlador de datos. En el ejemplo siguiente se crea un controlador de datos con `azdata arc dc create` denominado `arc` mediante el modo de conectividad directa (`connectivity-mode direct`). Antes de ejecutar el ejemplo, reemplace `<subscription id>` por el identificador de la suscripción.

```console
azdata arc dc create --profile-name azure-arc-aks-hci --namespace arc --name arc --subscription <subscription id> --resource-group my-resource-group --location eastus --connectivity-mode direct
```

### <a name="known-issues"></a>Problemas conocidos

- No se admite en la versión 1.19.x de Kubernetes en Azure Kubernetes Service (AKS).
- No se admite `containerd` en Kubernetes 1.19.
- El recurso de controlador de datos en Azure es actualmente un recurso de Azure. Las actualizaciones como la eliminación no se propagan de vuelta al clúster de Kubernetes.
- Los nombres de instancia no pueden tener más de 13 caracteres
- No hay actualización local para el controlador de datos de Azure Arc ni las instancias de base de datos.
- Las imágenes de contenedor de los servicios de datos habilitados para Arc no están firmadas.  Es posible que deba configurar los nodos de Kubernetes para permitir la extracción de imágenes de contenedor sin firmar.  Por ejemplo, si usa Docker como entorno de ejecución del contenedor, puede establecer la variable de entorno DOCKER_CONTENT_TRUST=0 y reiniciar.  Otros entornos de ejecución de contenedores tienen opciones similares, como ocurre con [OpenShift](https://docs.openshift.com/container-platform/4.5/openshift_images/image-configuration.html#images-configuration-file_image-configuration).
- No se pueden crear instancias de SQL Managed Instance ni grupos de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc desde Azure Portal.
- Por ahora, si usa NFS, debe establecer `allowRunAsRoot` en `true` en el archivo del perfil de implementación antes de crear el controlador de datos de Azure Arc.
- Solo autenticación de inicio de sesión de SQL y PostgreSQL.  No se admiten Azure Active Directory ni Active Directory.
- La creación de un controlador de datos en OpenShift requiere restricciones de seguridad relajadas.  Para obtener información detallada, consulte la documentación.
- No se admite la actualización a una versión de Kubernetes más reciente si usa el motor de Azure Kubernetes Service (AKS) en Azure Stack Hub con el controlador de datos de Azure Arc e instancias de base de datos. Desinstale el controlador de datos de Azure Arc y todas las instancias de base de datos antes de actualizar el clúster de Kubernetes.
- Los clústeres de AKS que abarcan [varias zonas de disponibilidad](../../aks/availability-zones.md) no se admiten actualmente para los servicios de datos habilitados para Azure Arc. Para evitar este problema, al crear el clúster de AKS en Azure Portal, si selecciona una región en la que hay zonas disponibles, desactive todas las zonas del control de selección. Consulte la siguiente imagen:

   :::image type="content" source="media/release-notes/aks-zone-selector.png" alt-text="Desactive las casillas de cada zona para no especificar ninguna.":::

## <a name="october-2020"></a>Octubre de 2020 

Número de versión de la CLI de datos de Azure (`azdata`): 20.2.3. Puede instalar `azdata` desde [Instalación de Azure Data CLI (`azdata`)](/sql/azdata/install/deploy-install-azdata).

### <a name="breaking-changes"></a>Últimos cambios

Esta versión presenta los siguientes cambios importantes: 

* En la definición de recursos personalizada (CRD) de PostgreSQL, el término `shards` cambia a `workers`. Este término (`workers`) coincide con el nombre del parámetro de la línea de comandos.

* `azdata arc postgres server delete` solicita confirmación antes de eliminar una instancia de PostgreSQL.  Use `--force` para omitir la solicitud.

### <a name="additional-changes"></a>Cambios adicionales

* Se ha agregado un nuevo parámetro opcional a `azdata arc postgres server create` denominado `--volume-claim mounts`. El valor es una lista separada por comas de montajes de notificación de volumen. Un montaje de notificación de volumen es un par de tipo de volumen y nombre de PVC. El único tipo de volumen admitido actualmente es `backup`.  En PostgreSQL, cuando el tipo de volumen es `backup`, PVC se monta en `/mnt/db-backups`.  Esto permite compartir copias de seguridad entre instancias de PostgresSQL para que la copia de seguridad de una instancia de PostgresSQL pueda restaurarse en otra.

* Nuevos nombres cortos para las definiciones de recursos personalizados de PostgresSQL: 
  * `pg11` 
  * `pg12`
* La carga de telemetría proporciona al usuario:
   * El número de puntos cargados en Azure, o bien 
   * Si no se ha cargado ningún dato en Azure, una solicitud para volver a intentarlo.
* `azdata arc dc debug copy-logs` ahora también hace lecturas en la carpeta `/var/opt/controller/log` y recopila los registros del motor de PostgreSQL en Linux.
*   Visualización de un indicador de funcionamiento durante la creación y restauración de copias de seguridad con Hiperescala de PostgreSQL.
* `azdata arc postrgres backup list` ahora incluye información de tamaño de copia de seguridad.
* La propiedad de nombre de administrador de SQL Managed Instance se ha agregado a la columna derecha de la hoja de información general en Azure Portal.
* Azure Data Studio admite la configuración del número de nodos de trabajo, el núcleo virtual y la configuración de memoria para Hiperescala de PostgreSQL. 
* La versión preliminar admite la copia de seguridad y restauración de las versiones 11 y 12 de PostgreSQL.

## <a name="september-2020"></a>Septiembre de 2020

Los servicios de datos habilitados para Azure Arc se publican como versión preliminar pública. Los servicios de datos habilitados para Arc permiten administrar los servicios de datos en cualquier lugar.

- Instancia administrada de SQL
- Hiperescala de PostgreSQL

Para obtener instrucciones, consulte [¿Qué son los servicios de datos habilitados para Azure Arc?](overview.md)

## <a name="next-steps"></a>Pasos siguientes

> **¿Solo desea realizar algunas pruebas?**  
> Empiece a trabajar rápidamente con el [Inicio rápido de Azure Arc](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_data/) en AKS, AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) o en una máquina virtual de Azure.

- [Instalación de las herramientas de cliente](install-client-tools.md)
- [Creación del controlador de datos de Azure Arc](create-data-controller.md) (requiere primero la instalación de las herramientas de cliente).
- [Creación de una instancia administrada de Azure SQL en Azure Arc](create-sql-managed-instance.md) (requiere primero la creación de un controlador de datos de Azure Arc).
- [Creación de un grupo de servidores de Hiperescala de Azure Database for PostgreSQL en Azure Arc](create-postgresql-hyperscale-server-group.md) (requiere primero la creación de un controlador de datos de Azure Arc).
- [Proveedores de recursos para servicios de Azure](../../azure-resource-manager/management/azure-services-resource-providers.md)
