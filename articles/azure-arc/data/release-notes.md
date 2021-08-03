---
title: 'Servicios de datos habilitados para Azure Arc: notas de la versión'
description: Notas de la versión más recientes
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.date: 06/02/2021
ms.topic: conceptual
ms.openlocfilehash: fc4dd4bcdb18b33fa9c6098c32e0aefecb7c46ee
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111439645"
---
# <a name="release-notes---azure-arc-enabled-data-services-preview"></a>Notas de la versión: servicios de datos habilitados para Azure Arc (versión preliminar)

En este artículo se resaltan las funcionalidades, las características y las mejoras que se han publicado u optimizado recientemente para los servicios de datos habilitados para Azure Arc. 

## <a name="may-2021"></a>Mayo de 2021

Esta versión preliminar se publicó el 2 de junio de 2021.

Como característica en versión preliminar, la tecnología que se presenta en este artículo está sujeta a los [términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="breaking-change"></a>Cambio importante

- Se han modificado las plantillas de implementación nativa de Kubernetes. Actualice las plantillas .yml.
    - Se han actualizado las plantillas para el controlador de datos, el programa previo y SQL Managed Instance: [GitHub microsoft/azure-arc pr 574](https://github.com/microsoft/azure_arc/pull/574).
    - Se han actualizado las plantillas para Hiperescala de PostgreSQL: [GitHub microsoft/azure-arc pr 574](https://github.com/microsoft/azure_arc/pull/574).

### <a name="whats-new"></a>Novedades

#### <a name="platform"></a>Plataforma

- Cree y elimine el controlador de datos, SQL Managed Instance y los grupos de servidores de Hiperescala de PostgreSQL desde Azure Portal. 
- Valide las acciones del portal al eliminar servicios de datos de Azure Arc. Por ejemplo, el portal alerta cuando intenta eliminar el controlador de datos cuando hay instancias de SQL Managed Instance implementadas mediante el controlador de datos.
- Cree perfiles de configuración personalizados para admitir la configuración personalizada al implementar el controlador de datos habilitado para Arc mediante Azure Portal.
- De manera opcional, cargue automáticamente los registros en el área de trabajo de Azure Log Analytics en el modo de conexión directa.

####    <a name="azure-arc-enabled-postgresql-hyperscale"></a>Hiperescala de PostgreSQL habilitada para Azure Arc

Esta versión presenta las siguientes características o funcionalidades:

- Elimine una instancia de Hiperescala de PostgreSQL habilitada para Azure Arc desde Azure Portal cuando su controlador de datos se haya configurado para el modo de conexión directa.
- Implemente Hiperescala de PostgreSQL habilitada para Azure Arc desde la página de implementación de Azure Database for Postgres en Azure Portal. Consulte [Selección de la opción de implementación de Azure Database for PostgreSQL: Microsoft Azure](https://ms.portal.azure.com/#create/Microsoft.PostgreSQLServer).
- Especifique las clases de almacenamiento y las extensiones de Postgres al implementar Hiperescala de PostgreSQL habilitada para Azure Arc desde Azure Portal.
- Reduzca el número de nodos de trabajo en su instancia de Hiperescala de PostgreSQL habilitada para Azure Arc. Puede realizar esta operación (conocida como reducción horizontal, en oposición al escalado horizontal, al aumentar el número de nodos de trabajo) desde la línea de comandos `azdata`.

#### <a name="azure-arc-enabled-sql-managed-instance"></a>SQL Managed Instance para Azure Arc

- La nueva [extensión de la CLI de Azure](/cli/azure/azure-cli-extensions-overview) para SQL Managed Instance habilitada para Arc tiene los mismos comandos que `azdata arc sql mi <command>`. Todos los comandos de SQL Managed Instance habilitada para Arc se encuentran en `az sql mi-arc`. Todos los comandos de `azdata` relacionados con Arc estarán en desuso y se moverán a la CLI de Azure en una versión futura.

   Para agregar la extensión:
  
   ```azurecli
   az extension add --source https://azurearcdatacli.blob.core.windows.net/cli-extensions/arcdata-0.0.1-py2.py3-none-any.whl -y
   az sql mi-arc --help
   ```

- Desencadene manualmente una conmutación por error de mediante Transact-SQL. Ejecute los comandos siguientes en orden:

   1. En la conexión del punto de conexión de la réplica principal:
   
      ```sql
       ALTER AVAILABILITY GROUP current SET (ROLE = SECONDARY);
      ```

   1. En la conexión del punto de conexión de la réplica secundaria:
   
      ```sql
      ALTER AVAILABILITY GROUP current SET (ROLE = PRIMARY);
      ```
    
- El comando `BACKUP` de Transact-SQL está bloqueado, a menos que use el valor `COPY_ONLY`. Esto admite la funcionalidad de restauración a un momento dado.

### <a name="known-issues"></a>Problemas conocidos

#### <a name="platform"></a>Plataforma

- Puede crear un controlador de datos, una instancia SQL Managed Instance o un grupo de servidores de Hiperescala de PostgreSQL en un clúster conectado con Azure Portal. No se admite la implementación con otras herramientas de Servicios de datos habilitados para Azure Arc. En concreto, no se puede implementar un controlador de datos en modo de conexión directa con ninguna de las herramientas siguientes durante esta versión.
   - Azure Data Studio
   - CLI de datos de Azure (`azdata`)
   - Herramientas nativas de Kubernetes (`kubectl`)

   En [Controlador de datos de Azure Arc | Modo de conexión directa](deploy-data-controller-direct-mode.md) se explica cómo crear el controlador de datos en el portal. 

- Todavía puede usar `kubectl` para crear recursos directamente en un clúster de Kubernetes, pero no se verán reflejados en Azure Portal.

- En el modo de conexión directa, la carga de uso, las métricas y los registros mediante `azdata arc dc upload` están bloqueados actualmente. El uso se carga automáticamente. La carga del controlador de datos creado en modo de conexión indirecta debe seguir funcionando.
- La carga automática de datos de uso en modo de conexión directa no se realizará correctamente si se usa el proxy mediante `–proxy-cert <path-t-cert-file>`.
- SQL Managed Instance habilitado para Azure Arc e Hiperescala de PostgreSQL habilitada para Azure Arc no tiene la certificación GB18030.
- Actualmente, solo se admite un controlador de datos de Azure Arc en modo de conexión directa por clúster de Kubernetes.

#### <a name="azure-arc-enabled-postgresql-hyperscale"></a>Hiperescala de PostgreSQL habilitada para Azure Arc

- La restauración a un momento dado no se admite por ahora en el almacenamiento NFS.
- No es posible habilitar ni configurar la extensión `pg_cron` al mismo tiempo. Debe usar dos comandos para esto. Un comando para habilitarlo y otro para configurarlo. Por ejemplo:

   1. Habilite la extensión:
   
      ```console
      azdata arc postgres server edit -n myservergroup --extensions pg_cron 
      ```

   1. Reinicie el grupo de servidores.

   1. Configure la extensión:
   
      ```console
      azdata arc postgres server edit -n myservergroup --engine-settings cron.database_name='postgres'
      ```

   Si ejecuta el segundo comando antes de que se complete el reinicio, se producirá un error. Si ese es el caso, espere unos instantes más y vuelva a ejecutar el segundo comando.

- Al pasar un valor no válido al parámetro `--extensions` al editar la configuración de un grupo de servidores para habilitar extensiones adicionales, se restablece incorrectamente la lista de extensiones habilitadas al momento de la creación del grupo de servidores e impide que el usuario cree extensiones adicionales. La única solución disponible cuando esto sucede es eliminar el grupo de servidores y volver a implementarlo.

## <a name="april-2021"></a>Abril de 2021

Esta versión preliminar se ha publicado el 29 de abril de 2021.

### <a name="whats-new"></a>Novedades

En esta sección se describen las nuevas características incorporadas o habilitadas en esta versión. 

#### <a name="platform"></a>Plataforma

- Los clústeres de conexión directa cargan automáticamente la información de telemetría de Azure. 

####    <a name="azure-arc-enabled-postgresql-hyperscale"></a>Hiperescala de PostgreSQL habilitada para Azure Arc

- Hiperescala de PostgreSQL habilitada para Azure Arc ahora se admite en el modo de conexión directa. Ahora puede implementar Hiperescala de PostgreSQL habilitada para Azure Arc desde Azure Market Place en Azure Portal. 
- Hiperescala de PostgreSQL habilitada para Azure Arc incluye la extensión Citus 10.0, que incorpora almacenamiento de tablas en columnas
- Hiperescala de PostgreSQL habilitada para Azure Arc ahora admite la administración completa de usuarios y roles.
- Hiperescala de PostgreSQL habilitada para Azure Arc ahora admite extensiones adicionales con `Tdigest` y `pg_partman`.
- Hiperescala de PostgreSQL habilitada para Azure Arc ahora admite la configuración de núcleo virtual y memoria por rol de la instancia de PostgreSQL en el grupo de servidores.
- Hiperescala de PostgreSQL habilitada para Azure Arc ahora admite la configuración de motor de base de datos o servidor por rol de la instancia de PostgreSQL en el grupo de servidores.

#### <a name="azure-arc-enabled-sql-managed-instance"></a>SQL Managed Instance para Azure Arc

- Restaure una base de datos a SQL Managed Instance con tres réplicas para que se agregue automáticamente al grupo de disponibilidad. 
- Conéctese a un punto de conexión secundario de solo lectura en las instancias de SQL Managed Instance implementadas con tres réplicas. Use `azdata arc sql endpoint list` para ver el punto de conexión secundario de solo lectura.

## <a name="march-2021"></a>Marzo de 2021

La versión de marzo de 2021 se introdujo inicialmente el 5 de abril de 2021 y las fases finales de la versión se completaron el 9 de abril de 2021.

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

- Ahora puede restaurar una base de datos a SQL Managed Instance con tres réplicas para que se agregue automáticamente al grupo de disponibilidad. 

- Ahora puede conectarse a un punto de conexión secundario de solo lectura en las instancias de SQL Managed Instance implementadas con tres réplicas. Use `azdata arc sql endpoint list` para ver el punto de conexión secundario de solo lectura.

## <a name="february-2021"></a>Febrero de 2021

### <a name="new-capabilities-and-features"></a>Nuevas características y funcionalidades

Número de versión de la CLI de datos de Azure (`azdata`): 20.3.1. Puede instalar `azdata` desde [Instalación de Azure Data CLI (`azdata`)](/sql/azdata/install/deploy-install-azdata).

Otras actualizaciones incluyen:

- SQL Managed Instance para Azure Arc
   - Alta disponibilidad con grupos de disponibilidad AlwaysOn

- Hiperescala de PostgreSQL habilitada para Azure Arc (Azure Data Studio): 
   - En la página de información general se muestra el estado del grupo de servidores desglosado por nodo.
   - Una nueva página de propiedades muestra más detalles sobre el grupo de servidores.
   - Configuración de parámetros del motor de Postgres desde la página **Node Parameters** (Parámetros del nodo).

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

- Las implementaciones de PostgreSQL respetan los parámetros de tamaño de volumen indicados en los comandos de creación.
- Ahora se respetan los parámetros de versión de motor al editar un grupo de servidores.
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
