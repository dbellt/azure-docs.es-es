---
title: 'Servicios de datos habilitados para Azure Arc: problemas conocidos'
description: Problemas conocidos más recientes
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.date: 03/02/2021
ms.topic: conceptual
ms.openlocfilehash: ee652047a33d73ece2d7648905fa590d90b1fb2f
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107029512"
---
# <a name="known-issues---azure-arc-enabled-data-services-preview"></a>Problemas conocidos: Servicios de datos habilitados para Azure Arc (versión preliminar)

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="march-2021"></a>Marzo de 2021

### <a name="data-controller"></a>Controlador de datos

- Puede crear un controlador de datos en modo de conexión directa con Azure Portal. No se admite la implementación con otras herramientas de Servicios de datos habilitados para Azure Arc. En concreto, no se puede implementar un controlador de datos en modo de conexión directa con ninguna de las herramientas siguientes durante esta versión.
   - Azure Data Studio
   - CLI de datos de Azure (`azdata`)
   - Herramientas nativas de Kubernetes

   En [Controlador de datos de Azure Arc | Modo de conexión directa](deploy-data-controller-direct-mode.md) se explica cómo crear el controlador de datos en el portal. 

### <a name="azure-arc-enabled-postgresql-hyperscale"></a>Hiperescala de PostgreSQL habilitada para Azure Arc

- No se admite la implementación de un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc en un controlador de datos de Arc habilitado para el modo de conexión directa.
- Al pasar un valor no válido al parámetro `--extensions` al editar la configuración de un grupo de servidores para habilitar extensiones adicionales, se restablece incorrectamente la lista de extensiones habilitadas al momento de la creación del grupo de servidores e impide que el usuario cree extensiones adicionales. La única solución disponible cuando esto sucede es eliminar el grupo de servidores y volver a implementarlo.

## <a name="february-2021"></a>Febrero de 2021

### <a name="data-controller"></a>Controlador de datos

- El modo de clúster de conexión directa está deshabilitado.

### <a name="azure-arc-enabled-postgresql-hyperscale"></a>Hiperescala de PostgreSQL habilitada para Azure Arc

- La restauración a un momento dado no se admite por ahora en el almacenamiento NFS.
- No es posible habilitar y configurar la extensión pg_cron al mismo tiempo. Debe usar dos comandos para esto. Un comando para habilitarlo y otro para configurarlo. 

   Por ejemplo:
   ```console
   § azdata arc postgres server edit -n myservergroup --extensions pg_cron 
   § azdata arc postgres server edit -n myservergroup --engine-settings cron.database_name='postgres'
   ```

   El primer comando requiere reiniciar el grupo de servidores. Por lo tanto, antes de ejecutar el segundo comando, asegúrese de que el estado del grupo de servidores ha pasado de Actualizando a Listo. Si ejecuta el segundo comando antes de que se complete el reinicio, se producirá un error. Si ese es el caso, espere unos instantes más y vuelva a ejecutar el segundo comando.

## <a name="introduced-prior-to-february-2021"></a>Incorporado antes de febrero de 2021

### <a name="data-controller"></a>Controlador de datos

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


## <a name="next-steps"></a>Pasos siguientes

> **¿Solo desea realizar algunas pruebas?**  
> Empiece a trabajar rápidamente con el [Inicio rápido de Azure Arc](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_data/) en AKS, AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) o en una máquina virtual de Azure.

- [Instalación de las herramientas de cliente](install-client-tools.md)
- [Creación del controlador de datos de Azure Arc](create-data-controller.md) (requiere primero la instalación de las herramientas de cliente).
- [Creación de una instancia administrada de Azure SQL en Azure Arc](create-sql-managed-instance.md) (requiere primero la creación de un controlador de datos de Azure Arc).
- [Creación de un grupo de servidores de Hiperescala de Azure Database for PostgreSQL en Azure Arc](create-postgresql-hyperscale-server-group.md) (requiere primero la creación de un controlador de datos de Azure Arc).
- [Proveedores de recursos para servicios de Azure](../../azure-resource-manager/management/azure-services-resource-providers.md)
- [Notas de la versión: servicios de datos habilitados para Azure Arc (versión preliminar)](release-notes.md)
