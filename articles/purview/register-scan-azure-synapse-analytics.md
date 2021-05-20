---
title: Examen de grupos de SQL dedicados
description: En esta guía de procedimientos se describen los detalles acerca de cómo examinar los grupos de SQL dedicados.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 05/08/2021
ms.openlocfilehash: f2797af01dad10c04c8a56cf52a584ea0f04af31
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109656762"
---
# <a name="register-and-scan-dedicated-sql-pools-formerly-sql-dw"></a>Registro y examen de grupos de SQL dedicados (antes SQL DW)

> [!NOTE]
> Si quiere registrar y examinar una base de datos SQL dedicada dentro de un área de trabajo de Synapse, debe seguir las instrucciones [que se indican aquí](register-scan-synapse-workspace.md).

En este artículo se describe cómo registrar y examinar una instancia de un grupo de SQL dedicado (antes SQL DW) en Purview.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Azure Synapse Analytics (anteriormente SQL DW) admite exámenes completos e incrementales para capturar los metadatos y el esquema. Los exámenes también clasifican los datos automáticamente en función de las reglas de clasificación personalizadas y del sistema.

### <a name="known-limitations"></a>Restricciones conocidas

> * Azure Purview no admite el examen de [vistas](/sql/relational-databases/views/views?view=azure-sqldw-latest&preserve-view=true) en Azure Synapse Analytics.
> * Azure Purview no admite más de 300 columnas en la pestaña Esquema y mostrará el mensaje "Additional-Columns-Truncated" (columnas adicionales truncadas). 

## <a name="prerequisites"></a>Requisitos previos

- Antes de registrar los orígenes de datos, cree una cuenta de Azure Purview. Para más información sobre cómo crear una cuenta de Purview, consulte [Inicio rápido: creación de una cuenta de Azure Purview](create-catalog-portal.md).
- Tenga en cuenta que debe ser administrador de los orígenes de datos de Azure Purview.
- Acceso de red entre la cuenta de Purview y Azure Synapse Analytics.
 
## <a name="setting-up-authentication-for-a-scan"></a>Configuración de la autenticación para un examen

Hay tres maneras de configurar la autenticación de Azure Synapse Analytics:

- Identidad administrada
- Autenticación de SQL
- Entidad de servicio

    > [!Note]
    > Solo pueden crear inicios de sesión el inicio de sesión de entidad de seguridad a nivel de servidor (creado por el proceso de aprovisionamiento) o los miembros del rol de base de datos `loginmanager` en la base de datos maestra. Unos 15 minutos después de conceder el permiso, la cuenta de Purview debería tener los permisos adecuados para poder examinar los recursos.

### <a name="managed-identity-recommended"></a>Identidad administrada (recomendada) 
   
La cuenta de Purview tiene su propia identidad administrada, que es básicamente el nombre que le di a la cuenta de Purview cuando la creó. Debe crear un usuario de Azure AD en Azure Synapse Analytics (anteriormente SQL DW) con el nombre exacto de la identidad administrada de Purview; para ello, siga los requisitos previos y el tutorial [Creación de usuarios de Azure AD con aplicaciones de Azure AD](../azure-sql/database/authentication-aad-service-principal-tutorial.md).

Ejemplo de la sintaxis de SQL para crear el usuario y conceder el permiso:

```sql
CREATE USER [PurviewManagedIdentity] FROM EXTERNAL PROVIDER
GO

EXEC sp_addrolemember 'db_owner', [PurviewManagedIdentity]
GO
```

La autenticación debe tener permiso para obtener los metadatos de la base de datos, los esquemas y las tablas. También debe ser capaz de consultar las tablas para realizar el muestreo para la clasificación. Es recomendable asignar el permiso `db_owner` a la identidad.

### <a name="service-principal"></a>Entidad de servicio

Para usar la autenticación de entidad de servicio para realizar exámenes, puede usar una existente o crear una nuevo. 

> [!Note]
> Si tiene que crear una entidad de servicio, siga estos pasos:
> 1. Acceda a [Azure Portal](https://portal.azure.com).
> 1. Seleccione **Azure Active Directory** en el menú de la izquierda.
> 1. Seleccione **App registrations** (Registros de aplicaciones).
> 1. Seleccione **+ Nuevo registro de aplicaciones**.
> 1. Escriba un nombre para la **aplicación** (el nombre de la entidad de servicio).
> 1. Seleccione **Solo las cuentas de este directorio organizativo**.
> 1. En el URI de redirección, seleccione **Web** y escriba cualquier dirección URL; no tiene que ser real ni del trabajo.
> 1. Después, seleccione **Register** (Registrar).

Es necesario obtener el id. de aplicación y el secreto de la entidad de servicio:

1. Vaya a la entidad de servicio en [Azure Portal](https://portal.azure.com).
1. Copie los valores de **Id. de aplicación (cliente)** de **Información general** y **Secreto de cliente** que están en **Certificados y secretos**.
1. Vaya a almacén de claves.
1. Seleccione **Configuración > Secretos**.
1. Seleccione **+ Generar/Importar** y escriba el **nombre** que quiera y el **valor** como **Secreto de cliente** de la entidad de servicio.
1. Seleccione **Crear** para completar la acción.
1. Si el almacén de claves no está conectado todavía a Purview, necesitará [crear una conexión del almacén de claves](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Por último, [cree una credencial](manage-credentials.md#create-a-new-credential) mediante la entidad de servicio para configurar el examen. 

#### <a name="granting-the-service-principal-access-to-your-azure-synapse-analytics-formerly-sql-dw"></a>Concesión de acceso a la entidad de servicio a Azure Synapse Analytics (anteriormente SQL DW)

Asimismo, debe crear un usuario de Azure AD en Azure Synapse Analytics siguiendo los requisitos previos y el tutorial [Creación de usuarios de Azure AD mediante las aplicaciones de Azure AD](../azure-sql/database/authentication-aad-service-principal-tutorial.md). Ejemplo de la sintaxis de SQL para crear el usuario y conceder el permiso:

```sql
CREATE USER [ServicePrincipalName] FROM EXTERNAL PROVIDER
GO

ALTER ROLE db_owner ADD MEMBER [ServicePrincipalName]
GO
```

> [!Note]
> Purview necesitará el valor de **Id. de la aplicación (cliente)** y el **secreto de cliente** para el examen.

### <a name="sql-authentication"></a>Autenticación SQL

Puede seguir las instrucciones de [CREAR INICIO DE SESIÓN](/sql/t-sql/statements/create-login-transact-sql?view=azure-sqldw-latest&preserve-view=true#examples-1) para crear un inicio de sesión de Azure Synapse Analytics (anteriormente SQL DW) si aún no tiene uno.

Cuando el método de autenticación seleccionado sea **Autenticación de SQL**, debe obtener la contraseña y almacenarla en el almacén de claves:

1. Obtención de la contraseña para el inicio de sesión de SQL
1. Vaya a almacén de claves.
1. Seleccione **Configuración > Secretos**.
1. Seleccione **+ Generate/Import** (+ Generar/Importar) y escriba el **nombre** y el **valor** como la *contraseña* del inicio de sesión de SQL.
1. Seleccione **Crear** para completar la acción.
1. Si el almacén de claves no está conectado todavía a Purview, necesitará [crear una conexión del almacén de claves](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Por último, [cree una nueva credencial](manage-credentials.md#create-a-new-credential) mediante la clave para configurar el examen.

## <a name="register-a-sql-dedicated-pool-formerly-sql-dw"></a>Registro de un grupo de SQL dedicado (antes SQL DW)

Para registrar un nuevo servidor de Azure Synapse Analytics en el catálogo de datos, haga lo siguiente:

1. Vaya a la cuenta de Purview.
1. Seleccione **Sources** (Orígenes) en el panel de navegación izquierdo.
1. Seleccione **Registrar**.
1. En **Registro de orígenes**, seleccione **Grupo dedicado de SQL (antes SQL DW)** .
1. Seleccione **Continuar**.

En la pantalla **Registrar orígenes (Azure Synapse Analytics)** , haga lo siguiente:

1. Escriba un **nombre** con el que se mostrará el origen de datos en el catálogo.
2. Elija la suscripción a Azure para filtrar las áreas de trabajo de Azure Synapse.
3. Seleccione un área de trabajo de Azure Synapse.
4. Seleccione una colección o cree una nueva (opcional).
5. Seleccione **Registrar** para registrar el origen de datos.

:::image type="content" source="media/register-scan-azure-synapse-analytics/register-sources.png" alt-text="Opciones de registro de orígenes" border="true":::

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Examen del catálogo de datos de Azure Purview](how-to-browse-catalog.md)
- [Búsqueda en el catálogo de datos de Azure Purview](how-to-search-catalog.md)
