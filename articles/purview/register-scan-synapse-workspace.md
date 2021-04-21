---
title: Cómo examinar áreas de trabajo de Azure Synapse
description: Obtenga información sobre cómo examinar un área de trabajo de Synapse en su catálogo de datos de Azure Purview.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 3/31/2021
ms.openlocfilehash: 230894b8e474c8d230322fb1e240f0317512d036
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107031322"
---
# <a name="register-and-scan-azure-synapse-workspaces"></a>Registro y examen de áreas de trabajo de Azure Synapse

En este artículo se describe cómo registrar un área de trabajo de Azure Synapse en Purview y cómo configurar un examen de esta.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Los exámenes de áreas de trabajo de Azure Synapse permiten capturar metadatos y esquemas con relación a bases de datos SQL dedicadas y sin servidor contenidas dentro de ellos. También clasifica los datos automáticamente en función de las reglas de clasificación personalizadas y del sistema.

## <a name="prerequisites"></a>Requisitos previos

- Antes de registrar los orígenes de datos, cree una cuenta de Azure Purview. Para más información sobre cómo crear una cuenta de Purview, consulte [Inicio rápido: creación de una cuenta de Azure Purview](create-catalog-portal.md).
- Tenga en cuenta que debe ser administrador de los orígenes de datos de Azure Purview.
- Configure la autenticación como se explica en las secciones siguientes.

### <a name="setting-up-authentication-for-enumerating-dedicated-sql-database-resources-under-a-synapse-workspace"></a>Configuración de la autenticación para enumerar recursos de bases de datos SQL dedicadas en un área de trabajo de Synapse

1. En Azure Portal, vaya al **grupo de recursos** o a la **suscripción** donde se encuentre el área de trabajo de Synapse.  
1. Seleccione  **Control de acceso (IAM)**   en el menú de navegación izquierdo. 
1. Debe ser propietario o administrador de acceso de usuarios para agregar un rol al grupo de recursos o a la suscripción. Seleccione el botón *+Agregar*. 
1. Establezca el rol **Lector** y escriba el nombre de la cuenta de Azure Purview (que representa su MSI) en el cuadro de entrada Select (Seleccionar). Haga clic en *Save* (Guardar) para finalizar la asignación de roles.
1. Siga los pasos 2 a 4 anteriores para agregar también el rol **Lector de datos de blobs** para el MSI de Azure Purview en el grupo de recursos o la suscripción donde se encuentra el área de trabajo de Synapse.

### <a name="setting-up-authentication-for-enumerating-serverless-sql-database-resources-under-a-synapse-workspace"></a>Configuración de la autenticación para enumerar recursos de bases de datos SQL sin servidor en un área de trabajo de Synapse

> [!NOTE]
> Debe ser **administrador de Synapse** en el área de trabajo para ejecutar estos comandos. Obtenga más información sobre los permisos de Synapse [aquí](../synapse-analytics/security/how-to-set-up-access-control.md).

1. Vaya al área de trabajo de Synapse.
1. Vaya a la sección **Datos** y a una de las bases de datos SQL sin servidor.
1. Haga clic en el icono de puntos suspensivos e inicie un nuevo script de SQL.
1. Agregue el MSI de la cuenta de Azure Purview (representado mediante el nombre de la cuenta) como **sysadmin** en las bases de datos SQL sin servidor, ejecutando el siguiente comando en el script de SQL:
    ```sql
    CREATE LOGIN [PurviewAccountName] FROM EXTERNAL PROVIDER;
    ALTER SERVER ROLE sysadmin ADD MEMBER [PurviewAccountName];
    ```

### <a name="setting-up-authentication-to-scan-resources-under-a-synapse-workspace"></a>Configuración de la autenticación para examinar recursos en un área de trabajo de Synapse

Hay tres maneras de configurar la autenticación de un origen de Azure Synapse:

- Identidad administrada
- Entidad de servicio
 
#### <a name="using-managed-identity-for-dedicated-sql-databases"></a>Uso de una identidad administrada para bases de datos SQL dedicadas

1. Vaya al **área de trabajo de Synapse**.
1. Vaya a la sección **Datos** y a una de las bases de datos SQL sin servidor.
1. Haga clic en el icono de puntos suspensivos e inicie un nuevo script de SQL.
1. Agregue el MSI de la cuenta de Azure Purview (representado mediante el nombre de la cuenta) como **db_owner** en las bases de datos SQL dedicadas, ejecutando el siguiente comando en el script de SQL:

    ```sql
    CREATE USER [PurviewAccountName] FROM EXTERNAL PROVIDER
    GO
    
    EXEC sp_addrolemember 'db_owner', [PurviewAccountName]
    GO
    ```
#### <a name="using-managed-identity-for-serverless-sql-databases"></a>Uso de una identidad administrada para bases de datos SQL sin servidor

1. Vaya al **área de trabajo de Synapse**.
1. Vaya a la sección **Datos** y a una de las bases de datos SQL sin servidor.
1. Haga clic en el icono de puntos suspensivos e inicie un nuevo script de SQL.
1. Agregue el MSI de la cuenta de Azure Purview (representado mediante el nombre de la cuenta) como **sysadmin** en las bases de datos SQL sin servidor, ejecutando el siguiente comando en el script de SQL:
    ```sql
    CREATE LOGIN [PurviewAccountName] FROM EXTERNAL PROVIDER;
    ALTER SERVER ROLE sysadmin ADD MEMBER [PurviewAccountName];
    ```

#### <a name="using-service-principal-for-dedicated-sql-databases"></a>Uso de una entidad de servicio para bases de datos SQL dedicadas

> [!NOTE]
> Primero debe configurar una nueva **credencial** del tipo de entidad de servicio siguiendo las instrucciones que se indican [aquí](manage-credentials.md).

1. Vaya al **área de trabajo de Synapse**.
1. Vaya a la sección **Datos** y a una de las bases de datos SQL sin servidor.
1. Haga clic en el icono de puntos suspensivos e inicie un nuevo script de SQL.
1. Agregue el **identificador de la entidad de servicio** como **db_owner** en la base de datos SQL dedicada ejecutando el siguiente comando en el script de SQL:

    ```sql
    CREATE USER [ServicePrincipalID] FROM EXTERNAL PROVIDER
    GO
    
    EXEC sp_addrolemember 'db_owner', [ServicePrincipalID]
    GO
    ```

#### <a name="using-service-principal-for-serverless-sql-databases"></a>Uso de una entidad de servicio para bases de datos SQL sin servidor

1. Vaya al **área de trabajo de Synapse**.
1. Vaya a la sección **Datos** y a una de las bases de datos SQL sin servidor.
1. Haga clic en el icono de puntos suspensivos e inicie un nuevo script de SQL.
1. Agregue el MSI de la cuenta de Azure Purview (representado mediante el nombre de la cuenta) como **sysadmin** en las bases de datos SQL sin servidor, ejecutando el siguiente comando en el script de SQL:
    ```sql
    CREATE LOGIN [ServicePrincipalID] FROM EXTERNAL PROVIDER;
    ALTER SERVER ROLE sysadmin ADD MEMBER [ServicePrincipalID];
    ```

> [!NOTE]
> Debe configurar la autenticación en cada base de datos SQL dedicada que esté contenida en el área de trabajo de Synapse que tenga intención de registrar y examinar. Los permisos mencionados anteriormente con relación a la base de datos SQL sin servidor se aplican a todas las contenidas en el área de trabajo, es decir, solo tendrá que ejecutarlo una vez.
    
## <a name="register-an-azure-synapse-source"></a>Registro de un origen de Azure Synapse

Para registrar un nuevo origen de Azure Synapse en el catálogo de datos, haga lo siguiente:

1. Vaya a la cuenta de Purview.
1. Seleccione **Sources** (Orígenes) en el panel de navegación izquierdo.
1. Seleccione **Registrar**.
1. En **Registrar orígenes**, seleccione **Azure Synapse Analytics (varios).**
1. Seleccione **Continuar**

   :::image type="content" source="media/register-scan-synapse-workspace/register-synapse-source.png" alt-text="Configuración del origen de Azure Synapse":::

En la pantalla **Registrar orígenes (Azure Synapse Analytics)** , haga lo siguiente:

1. Escriba un **nombre** con el que se mostrará el origen de datos en el catálogo.
1. También puede seleccionar una **suscripción** por la que filtrar.
1. **Seleccione un nombre de área de trabajo de Synapse** en la lista desplegable. Los puntos de conexión de SQL se rellenan automáticamente en función del área de trabajo seleccionada. 
1. Seleccione una **Collection** (Colección) o cree una nueva (opcional).
1. Seleccione **Finish** (Finalizar) para registrar el origen de datos.

    :::image type="content" source="media/register-scan-synapse-workspace/register-synapse-source-details.png" alt-text="Rellenado de los detalles del origen de Azure Synapse":::

## <a name="creating-and-running-a-scan"></a>Creación y ejecución de un examen

Para crear y ejecutar un nuevo examen, siga estos pasos:

1. Vaya a la sección **Orígenes**.

1. Seleccione el origen de datos que ha registrado.

1. Haga clic en View details (Ver detalles) y seleccione **+ New scan** (+ Nuevo examen) o use el icono de acción rápida en el icono del origen.

1. Rellene *Nombre* y seleccione todos los tipos de recursos que quiera examinar en este origen. **SQL Database** es el único tipo que se admite actualmente en un área de trabajo de Synapse.
   
    :::image type="content" source="media/register-scan-synapse-workspace/synapse-scan-setup.png" alt-text="Examen del origen de Azure Synapse":::

1. Seleccione la **credencial** para conectarse a los recursos del origen de datos. 
  
1. En cada tipo puede optar por examinar todos los recursos o un subconjunto de ellos por nombre.
1.  Haga clic en **Continuar** para seguir. 

1.  Seleccione un **conjunto de reglas de examen** de tipo Azure Synapse SQL. También puede crear conjuntos de reglas de examen insertados.

1. Elija el desencadenador del examen. Puede programarlo para que se ejecute **semanalmente/mensualmente** o **una vez**.

1. Revise el examen y seleccione Save (Guardar) para completar la configuración.   

## <a name="viewing-your-scans-and-scan-runs"></a>Visualización de los exámenes y las ejecuciones de exámenes

1. Para ver los detalles del origen, haga clic en **View details** (Ver detalles) en el icono de la sección de orígenes. 

      :::image type="content" source="media/register-scan-synapse-workspace/synapse-source-details.png" alt-text="Detalles del origen de Azure Synapse"::: 

1. Para ver los detalles de la ejecución del examen, vaya a la página **Scan details** (Detalles del examen).
    1. La *barra de estado* es un breve resumen sobre el estado de ejecución de los recursos secundarios. Se mostrará en el examen de nivel de área de trabajo.
    1. El color verde significa que se ha realizado correctamente, mientras que el rojo significa error. El gris significa que el examen sigue su curso.
    1. Puede hacer clic en cada examen para ver detalles más precisos.

      :::image type="content" source="media/register-scan-synapse-workspace/synapse-scan-details.png" alt-text="Detalles del examen de Azure Synapse" lightbox="media/register-scan-synapse-workspace/synapse-scan-details.png"::: 

1. Vea un resumen de las ejecuciones de exámenes recientes con errores en la parte inferior de la página de detalles del origen. También puede hacer clic para ver detalles más precisos relacionados con estas ejecuciones.

## <a name="manage-your-scans---edit-delete-or-cancel"></a>Administración de exámenes: editar, eliminar o cancelar
Para administrar o eliminar un examen, haga lo siguiente:

- Vaya al centro de administración. Seleccione Data sources (Orígenes de datos) en la sección Sources and scanning (Orígenes y examen) y, a continuación, seleccione el origen de datos deseado.

- Seleccione el examen que desea administrar. Seleccione Edit (Editar) para editar el examen.

- Seleccione Delete (Eliminar) para eliminar el examen.
- Si se está ejecutando un examen, también puede cancelarlo.

## <a name="next-steps"></a>Pasos siguientes

- [Examen del catálogo de datos de Azure Purview](how-to-browse-catalog.md)
- [Búsqueda en el catálogo de datos de Azure Purview](how-to-search-catalog.md)   