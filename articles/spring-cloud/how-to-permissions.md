---
title: Uso de permisos en Azure Spring Cloud
description: En este artículo se muestra cómo crear roles personalizados para permisos en Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 09/04/2020
ms.custom: devx-track-java
ms.openlocfilehash: 8bc17ef020ae61a18f5259a956c7b5d3b3998a4d
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110463586"
---
# <a name="how-to-use-permissions-in-azure-spring-cloud"></a>Uso de permisos en Azure Spring Cloud
En este artículo se muestra cómo crear roles personalizados que delegan permisos en recursos de Azure Spring Cloud. Los roles personalizados amplían los [roles integrados de Azure](../role-based-access-control/built-in-roles.md) con varios permisos estándar.

Implementaremos los siguientes roles personalizados:

* **Rol de desarrollador**: 
    * Implementar
    * Prueba
    * Reiniciar aplicaciones
    * Puede aplicar y realizar cambios en las configuraciones de aplicaciones del repositorio de Git
    * Puede obtener la secuencia de registros
* **Rol de operaciones/ingeniería de confiabilidad de sitios**: 
    * Reiniciar aplicaciones
    * Obtener secuencias de registros
    * No puede realizar cambios en las aplicaciones ni configuraciones
* **Rol de Azure Pipelines/Jenkins/Acciones de GitHub**:
    * Puede realizar operaciones de creación, lectura, actualización y eliminación
    * Puede crear y configurar todo en Azure Spring Cloud y las aplicaciones de la instancia de servicio: Acciones de Azure Pipelines, Jenkins o GitHub, con plantillas de Terraform o Resource Manager

## <a name="define-developer-role"></a>Definición del rol de desarrollador

El rol de desarrollador incluye permisos para reiniciar aplicaciones y ver sus secuencias de registros, pero no puede realizar cambios en las aplicaciones ni en la configuración.

### <a name="navigate-subscription-and-resource-group-access-control-iam"></a>Navegación al control de acceso (IAM) de la suscripción y el grupo de recursos

Siga estos pasos para empezar a definir un rol.

1. En Azure Portal, abra la suscripción y el grupo de recursos donde desee que se pueda asignar el rol personalizado.
2. Abra **Control de acceso (IAM)** .
3. Haga clic en **+ Agregar**.
4. Haga clic en **Agregar rol personalizado**.
5. Haga clic en **Next**.

   ![Creación de un rol personalizado](media/spring-cloud-permissions/create-custom-role.png)

6. Haga clic en **Agregar permisos**.

   ![Incorporación de permisos: iniciar](media/spring-cloud-permissions/add-permissions.png)

### <a name="search-for-azure-spring-cloud-permissions"></a>Búsqueda de permisos de Azure Spring Cloud:
7. En el cuadro de búsqueda, busque *Microsoft.app*.
Seleccione *Microsoft Azure Spring Cloud*.

   ![Seleccione Azure Spring Cloud](media/spring-cloud-permissions/spring-cloud-permissions.png)

8. Seleccione los permisos para el rol de desarrollador:

En **Microsoft.AppPlatform/Spring**, seleccione:
* Write : Create or Update Azure Spring Cloud service instance (Escritura: Crear o actualizar instancia del servicio Azure Spring Cloud)
* Read : Get Azure Spring Cloud service instance (Lectura: Obtener instancia del servicio Azure Spring Cloud)
* Other : List Azure Spring Cloud service instance test keys (Otro: Enumerar claves de prueba de la instancia del servicio Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps**, seleccione:
* Read : Read Microsoft Azure Spring Cloud application (Lectura: Leer aplicación de Microsoft Azure Spring Cloud)
* Other : Get Microsoft Azure Spring Cloud application resource upload URL (Otro: Obtener dirección URL de carga de recursos de aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps/bindings**, seleccione:
* Read : Read Microsoft Azure Spring Cloud application binding (Lectura: Leer enlace de aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps/deployments**, seleccione:
* Write : Write Microsoft Azure Spring Cloud application deployment (Escritura: Escribir implementación de aplicación de Microsoft Azure Spring Cloud)
* Read : Read Microsoft Azure Spring Cloud application deployment (Lectura: Leer implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Start Microsoft Azure Spring Cloud application deployment (Otro: Iniciar implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Stop Microsoft Azure Spring Cloud application deployment (Otro: Detener implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Restart Microsoft Azure Spring Cloud application deployment (Otro: Reiniciar implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Get Microsoft Azure Spring Cloud application deployment log file URL (Otro: Obtener dirección URL de archivo de registro de implementación de aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps/domains**, seleccione:
* Read : Read Microsoft Azure Spring Cloud application custom domain (Lectura: Leer dominio personalizado de aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/certificates**, seleccione:
* Read : Read Microsoft Azure Spring Cloud certificate (Lectura: Leer certificado de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/locations/operationResults/Spring**, seleccione:
* Read : Read operation result (Lectura: Leer resultado de la operación)

En **Microsoft.AppPlatform/locations/operationStatus/operationId**, seleccione:
* Read : Read operation status (Lectura: Leer estado de la operación)

    [ ![Crear permisos de desarrollador](media/spring-cloud-permissions/developer-permissions-box.png) ](media/spring-cloud-permissions/developer-permissions-box.png#lightbox)

9. Haga clic en **Agregar**.

10. Revise los permisos.

11. Haga clic en **Revisar y crear**.

## <a name="define-devops-engineer-role"></a>Definición del rol de ingeniero de DevOps
Este procedimiento define un rol con permisos para implementar, probar y reiniciar aplicaciones de Azure Spring Cloud.

1. Repita el procedimiento para navegar a la suscripción y el grupo de recursos, y vaya a Control de acceso (IAM).
2. Seleccione los permisos para el rol del ingeniero de DevOps:

En **Microsoft.AppPlatform/Spring**, seleccione:
* Write : Create or Update Azure Spring Cloud service instance (Escritura: Crear o actualizar instancia del servicio Azure Spring Cloud)
* Delete : Delete Azure Spring Cloud service instance (Eliminación: Eliminar instancia del servicio Azure Spring Cloud)
* Read : Get Azure Spring Cloud service instance (Lectura: Obtener instancia del servicio Azure Spring Cloud)
* Other : Enable Azure Spring Cloud service instance test endpoint (Otro: Habilitar punto de conexión de prueba de la instancia del servicio Azure Spring Cloud)
* Other : Disable Azure Spring Cloud service instance test endpoint (Otro: Deshabilitar punto de conexión de prueba de la instancia del servicio Azure Spring Cloud)
* Other : List Azure Spring Cloud service instance test keys (Otro: Enumerar claves de prueba de la instancia del servicio Azure Spring Cloud)
* Other : Regenerate Azure Spring Cloud service instance test keys (Otro: Regenerar clave de prueba de la instancia del servicio Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps**, seleccione:
* Write : Write Microsoft Azure Spring Cloud application (Escritura: Escribir aplicación de Microsoft Azure Spring Cloud)
* Delete : Delete Microsoft Azure Spring Cloud application (Eliminación: Eliminar aplicación de Microsoft Azure Spring Cloud)
* Read : Read Microsoft Azure Spring Cloud application (Lectura: Leer aplicación de Microsoft Azure Spring Cloud)
* Other : Get Microsoft Azure Spring Cloud application resource upload URL (Otro: Obtener dirección URL de carga de recursos de aplicación de Microsoft Azure Spring Cloud)
* Other : Validate Microsoft Azure Spring Cloud application custom domain (Otro: Validar dominio personalizado de aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps/bindings**, seleccione:
* Write : Write Microsoft Azure Spring Cloud application binding (Escritura: Escribir enlace de aplicación de Microsoft Azure Spring Cloud)
* Delete : Delete Microsoft Azure Spring Cloud application binding (Eliminación: Eliminar enlace de aplicación de Microsoft Azure Spring Cloud)
* Read : Read Microsoft Azure Spring Cloud application binding (Lectura: Leer enlace de aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps/deployments**, seleccione:
* Write : Write Microsoft Azure Spring Cloud application deployment (Escritura: Escribir implementación de aplicación de Microsoft Azure Spring Cloud)
* Delete : Delete Azure Spring Cloud application deployment (Eliminación: Eliminar implementación de aplicación de Azure Spring Cloud)
* Read : Read Microsoft Azure Spring Cloud application deployment (Lectura: Leer implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Start Microsoft Azure Spring Cloud application deployment (Otro: Iniciar implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Stop Microsoft Azure Spring Cloud application deployment (Otro: Detener implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Restart Microsoft Azure Spring Cloud application deployment (Otro: Reiniciar implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Get Microsoft Azure Spring Cloud application deployment log file URL (Otro: Obtener dirección URL de archivo de registro de implementación de aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps/deployments/skus**, seleccione:
* Read : List application deployment available skus (Lectura: Enumerar SKU disponibles de implementación de aplicación)

En **Microsoft.AppPlatform/locations**, seleccione:
* Other : Check name availability (Otro: Comprobar disponibilidad del nombre)

En Microsoft.AppPlatform/locations/operationResults/Spring seleccione: Read : Read operation result (Lectura: Leer resultado de la operación)

En **Microsoft.AppPlatform/locations/operationStatus/operationId**, seleccione:
* Read : Read operation status (Lectura: Leer estado de la operación)

En **Microsoft.AppPlatform/skus**, seleccione:
* Read : List available skus (Lectura: Enumerar SKU disponibles)

   [ ![Permisos de desarrollo y operaciones](media/spring-cloud-permissions/dev-ops-permissions.png) ](media/spring-cloud-permissions/dev-ops-permissions.png#lightbox)

3. Haga clic en **Agregar**.

4. Revise los permisos.

5. Haga clic en **Revisar y crear**.

## <a name="define-ops---site-reliability-engineering-role"></a>Definición del rol de operaciones/ingeniería de confiabilidad de sitios
Este procedimiento define un rol con permisos para implementar, probar y reiniciar aplicaciones de Azure Spring Cloud.

1. Repita el procedimiento para navegar a la suscripción y el grupo de recursos, y vaya a Control de acceso (IAM).

2. Seleccione los permisos para el rol de operaciones/ingeniería de confiabilidad de sitios:

En **Microsoft.AppPlatform/Spring**, seleccione:
* Read : Get Azure Spring Cloud service instance (Lectura: Obtener instancia del servicio Azure Spring Cloud)
* Other : List Azure Spring Cloud service instance test keys (Otro: Enumerar claves de prueba de la instancia del servicio Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps**, seleccione:
* Read : Read Microsoft Azure Spring Cloud application (Lectura: Leer aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/apps/deployments**, seleccione:
* Read : Read Microsoft Azure Spring Cloud application deployment (Lectura: Leer implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Start Microsoft Azure Spring Cloud application deployment (Otro: Iniciar implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Stop Microsoft Azure Spring Cloud application deployment (Otro: Detener implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Restart Microsoft Azure Spring Cloud application deployment (Otro: Reiniciar implementación de aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/locations/operationResults/Spring**, seleccione:
* Read : Read operation result (Lectura: Leer resultado de la operación)

En **Microsoft.AppPlatform/locations/operationStatus/operationId**, seleccione:
* Read : Read operation status (Lectura: Leer estado de la operación)

   [ ![Permisos de operaciones/ingeniería de confiabilidad de sitios](media/spring-cloud-permissions/ops-sre-permissions.png) ](media/spring-cloud-permissions/ops-sre-permissions.png#lightbox)

3. Haga clic en **Agregar**.

4. Revise los permisos.

5. Haga clic en **Revisar y crear**.

## <a name="define-azure-pipelinesprovisioning-role"></a>Definición del rol de Azure Pipelines/aprovisionamiento
Este rol de Jenkins/Acciones de GitHub lo puede crear y configurar todo en Azure Spring Cloud y las aplicaciones con una instancia de servicio. Este rol se usa para liberar o implementar código.

1. Repita el procedimiento para navegar a la suscripción y el grupo de recursos, y vaya a Control de acceso (IAM).

2. Abra las opciones de **Permisos**.

3. Seleccione los permisos para el rol de Azure Pipelines/aprovisionamiento:
  
En **Microsoft.AppPlatform/Spring**, seleccione:
* Write : Create or Update Azure Spring Cloud service instance (Escritura: Crear o actualizar instancia del servicio Azure Spring Cloud)
* Delete : Delete Azure Spring Cloud service instance (Eliminación: Eliminar instancia del servicio Azure Spring Cloud)
* Read : Get Azure Spring Cloud service instance (Lectura: Obtener instancia del servicio Azure Spring Cloud)
* Other : Enable Azure Spring Cloud service instance test endpoint (Otro: Habilitar punto de conexión de prueba de la instancia del servicio Azure Spring Cloud)
* Other : Disable Azure Spring Cloud service instance test endpoint (Otro: Deshabilitar punto de conexión de prueba de la instancia del servicio Azure Spring Cloud)
* Other : List Azure Spring Cloud service instance test keys (Otro: Enumerar claves de prueba de la instancia del servicio Azure Spring Cloud)
* Other : Regenerate Azure Spring Cloud service instance test keys (Otro: Regenerar clave de prueba de la instancia del servicio Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps**, seleccione:
* Write : Write Microsoft Azure Spring Cloud application (Escritura: Escribir aplicación de Microsoft Azure Spring Cloud)
* Delete : Delete Microsoft Azure Spring Cloud application (Eliminación: Eliminar aplicación de Microsoft Azure Spring Cloud)
* Read : Read Microsoft Azure Spring Cloud application (Lectura: Leer aplicación de Microsoft Azure Spring Cloud)
* Other : Get Microsoft Azure Spring Cloud application resource upload URL (Otro: Obtener dirección URL de carga de recursos de aplicación de Microsoft Azure Spring Cloud)
* Other : Validate Microsoft Azure Spring Cloud application custom domain (Otro: Validar dominio personalizado de aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps/bindings**, seleccione:
* Write : Write Microsoft Azure Spring Cloud application binding (Escritura: Escribir enlace de aplicación de Microsoft Azure Spring Cloud)
* Delete : Delete Microsoft Azure Spring Cloud application binding (Eliminación: Eliminar enlace de aplicación de Microsoft Azure Spring Cloud)
* Read : Read Microsoft Azure Spring Cloud application binding (Lectura: Leer enlace de aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/Spring/apps/deployments**, seleccione:
* Write : Write Microsoft Azure Spring Cloud application deployment (Escritura: Escribir implementación de aplicación de Microsoft Azure Spring Cloud)
* Delete : Delete Azure Spring Cloud application deployment (Eliminación: Eliminar implementación de aplicación de Azure Spring Cloud)
* Read : Read Microsoft Azure Spring Cloud application deployment (Lectura: Leer implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Start Microsoft Azure Spring Cloud application deployment (Otro: Iniciar implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Stop Microsoft Azure Spring Cloud application deployment (Otro: Detener implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Restart Microsoft Azure Spring Cloud application deployment (Otro: Reiniciar implementación de aplicación de Microsoft Azure Spring Cloud)
* Other : Get Microsoft Azure Spring Cloud application deployment log file URL (Otro: Obtener dirección URL de archivo de registro de implementación de aplicación de Microsoft Azure Spring Cloud)

En **Microsoft.AppPlatform/skus**, seleccione:
* Read : List available skus (Lectura: Enumerar SKU disponibles)

En **Microsoft.AppPlatform/locations**, seleccione:
* Other : Check name availability (Otro: Comprobar disponibilidad del nombre)

En **Microsoft.AppPlatform/locations/operationResults/Spring**, seleccione:
* Read : Read operation result (Lectura: Leer resultado de la operación)

En **Microsoft.AppPlatform/locations/operationStatus/operationId**, seleccione:
* Read : Read operation status (Lectura: Leer estado de la operación)

En **Microsoft.AppPlatform/skus**, seleccione:
* Read : List available skus (Lectura: Enumerar SKU disponibles)

   [ ![Permisos de Pipelines](media/spring-cloud-permissions/pipelines-permissions-box.png) ](media/spring-cloud-permissions/pipelines-permissions-box.png#lightbox)  

4. Haga clic en **Agregar**.

5. Revise los permisos.

6. Haga clic en **Revisar y crear**.


## <a name="see-also"></a>Consulte también
* [Creación o actualización de roles personalizados de Azure mediante Azure Portal](../role-based-access-control/custom-roles-portal.md)

Para obtener más información sobre los tres métodos que definen permisos personalizados, consulte:
* [Clonación de un rol](../role-based-access-control/custom-roles-portal.md#clone-a-role)
* [Comienzo desde cero](../role-based-access-control/custom-roles-portal.md#start-from-scratch)
* [Inicio desde un archivo JSON](../role-based-access-control/custom-roles-portal.md#start-from-json)
