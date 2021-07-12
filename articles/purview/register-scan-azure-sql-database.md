---
title: Registro y examen de Azure SQL Database
description: En este tutorial se describe cómo examinar Azure SQL Database
author: hophanms
ms.author: hophan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: tutorial
ms.date: 06/08/2021
ms.openlocfilehash: da265e1be47a7ee1a98f6e8169f2531110b5c772
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111756606"
---
# <a name="register-and-scan-an-azure-sql-database"></a>Registro y análisis de una instancia de Azure SQL Database

En este artículo se describe cómo registrar un origen de datos de Azure SQL Database en Purview y cómo configurar un examen en él.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

El origen de datos de Azure SQL Database admite la funcionalidad siguiente:

- **Exámenes completos e incrementales** para capturar metadatos y su clasificación en Azure SQL Database.

- **Linaje** entre los recursos de datos de las actividades de copia y flujo de datos de ADF.

### <a name="known-limitations"></a>Restricciones conocidas

> * Azure Purview no admite más de 300 columnas en la pestaña Esquema y mostrará el mensaje "Additional-Columns-Truncated" (Columnas adicionales truncadas). 

## <a name="prerequisites"></a>Prerrequisitos

1. Cree una cuenta de Purview si aún no tiene una.

1. Acceso de red entre la cuenta de Purview y Azure SQL Database.


### <a name="set-up-authentication-for-a-scan"></a>Configuración de la autenticación para un examen

Autenticación para examinar Azure SQL Database. Si necesita crear una autenticación, debe [autorizar el acceso a la base de datos a SQL Database](../azure-sql/database/logins-create-manage.md). A día de hoy, Purview admite tres métodos de autenticación:

- Autenticación SQL
- Entidad de servicio
- Identidad administrada

#### <a name="sql-authentication"></a>Autenticación SQL

> [!Note]
> Solo pueden crear inicios de sesión el inicio de sesión de entidad de seguridad a nivel de servidor (creado por el proceso de aprovisionamiento) o los miembros del rol de base de datos `loginmanager` en la base de datos maestra. Unos **15 minutos** después de conceder el permiso, la cuenta de Purview debería tener los permisos adecuados para poder examinar los recursos.

Puede seguir las instrucciones de [CREAR INICIO DE SESIÓN](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-current&preserve-view=true#examples-1) para crear un inicio de sesión de Azure SQL Database si aún no tiene uno. En los pasos siguientes necesitará un **nombre de usuario** y una **contraseña**.

1. Vaya al almacén de claves en Azure Portal.
1. Seleccione **Configuración > Secretos**.
1. Seleccione **+ Generar/Importar** y escriba el **nombre** y el **valor** como la *contraseña* del inicio de sesión de Azure SQL Database.
1. Seleccione **Crear** para completar la acción.
1. Si el almacén de claves no está conectado todavía a Purview, necesitará [crear una conexión del almacén de claves](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Por último, [cree una nueva credencial](manage-credentials.md#create-a-new-credential) mediante el **nombre de usuario** y la **contraseña** para configurar el examen.

#### <a name="service-principal-and-managed-identity"></a>Entidad de servicio e identidad administrada

Para permitir que Purview use la entidad de servicio o la **identidad administrada** de Purview para examinar la instancia de Azure SQL Database se necesitan varios pasos.

   > [!Note]
   > Purview necesitará el valor de **Id. de la aplicación (cliente)** y el **secreto de cliente** para el examen.

##### <a name="create-or-use-an-existing-service-principal"></a>Creación de una entidad de servicio o uso de una existente

> [!Note]
> Omita este paso si usa la **identidad administrada de Purview**.

Puede usar una entidad de servicio existente o crear una. 

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

##### <a name="configure-azure-ad-authentication-in-the-database-account"></a>Configuración de la autenticación de Azure AD en la cuenta de la base de datos

La entidad de servicio o la identidad administrada debe tener permiso para obtener los metadatos de la base de datos, los esquemas y las tablas. También debe ser capaz de consultar las tablas para realizar el muestreo para la clasificación.

- [Configuración y administración de la autenticación de Azure AD con Azure SQL](../azure-sql/database/authentication-aad-configure.md)
- Si usa la identidad administrada, la cuenta de Purview tiene la suya propia, que es básicamente el nombre que le diera a la cuenta de Purview al crearla. Debe crear un usuario de Azure AD en Azure SQL Database con la identidad administrada de Purview exacta o su propia entidad de servicio siguiendo el tutorial [Cree el usuario de la entidad de servicio en Azure SQL Database](../azure-sql/database/authentication-aad-service-principal-tutorial.md#create-the-service-principal-user-in-azure-sql-database). Tiene que asignar el permiso adecuado (por ejemplo, `db_datareader`) a la identidad. Ejemplo de la sintaxis de SQL para crear el usuario y conceder el permiso:

    ```sql
    CREATE USER [Username] FROM EXTERNAL PROVIDER
    GO
    
    EXEC sp_addrolemember 'db_datareader', [Username]
    GO
    ```

    > [!Note]
    > `Username` es la entidad de servicio propia o la identidad administrada de Purview. Puede leer más información sobre [roles fijos de base de datos y sus funcionalidades](/sql/relational-databases/security/authentication-access/database-level-roles#fixed-database-roles).
    
##### <a name="add-service-principal-to-key-vault-and-purviews-credential"></a>Incorporación de la entidad de servicio al almacén de claves y la credencial de Purview

> [!Note]
> Si tiene previsto usar la **identidad administrada** de Purview, puede omitir este paso, puesto que esta ya está en la credencial de **Purview-MSI**.

Es necesario obtener el id. de aplicación y el secreto de la entidad de servicio:

1. Vaya a la entidad de servicio en [Azure Portal](https://portal.azure.com).
1. Copie los valores de **Id. de aplicación (cliente)** de **Información general** y **Secreto de cliente** que están en **Certificados y secretos**.
1. Vaya a almacén de claves.
1. Seleccione **Configuración > Secretos**.
1. Seleccione **+ Generar/Importar** y escriba el **nombre** que quiera y el **valor** como **Secreto de cliente** de la entidad de servicio.
1. Seleccione **Crear** para completar la acción.
1. Si el almacén de claves no está conectado todavía a Purview, necesitará [crear una conexión del almacén de claves](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Por último, [cree una credencial](manage-credentials.md#create-a-new-credential) mediante la entidad de servicio para configurar el examen.

### <a name="firewall-settings"></a>Configuración de firewall

El servidor de bases de datos debe permitir que se habiliten las conexiones de Azure. De este modo, Azure Purview podrá llegar y conectarse al servidor. Puede seguir la guía de procedimientos para las [Conexiones desde dentro de Azure](../azure-sql/database/firewall-configure.md#connections-from-inside-azure).

1. Vaya a la cuenta de la base de datos.
1. En la página **Información general**, seleccione el nombre del servidor.
1. Seleccione **Security > Firewalls and virtual networks** (Seguridad > Firewalls y redes virtuales).
1. Seleccione **Sí** en **Permitir que los servicios y recursos de Azure accedan a este servidor**.

    :::image type="content" source="media/register-scan-azure-sql-database/sql-firewall.png" alt-text="Permitir que los servicios y recursos de Azure accedan a este servidor." border="true":::
    
> [!Note]
> Actualmente, Azure Purview no admite la configuración de redes virtuales. Por lo tanto, no puede configurar el firewall basado en IP.

## <a name="register-an-azure-sql-database-data-source"></a>Registro de un origen de datos de Azure SQL Database

Para registrar una nueva instancia de Azure SQL Database en el catálogo de datos, haga lo siguiente:

1. Vaya a la cuenta de Purview.

1. Seleccione **Sources** (Orígenes) en el panel de navegación izquierdo.

1. Seleccione **Registrar**.

1. En **Register sources** (Registrar orígenes), seleccione **Azure SQL Database**. Seleccione **Continuar**.

:::image type="content" source="media/register-scan-azure-sql-database/register-new-data-source.png" alt-text="Registro de un origen de datos nuevo" border="true":::

En la pantalla **Register sources (Azure SQL Database)** (Registrar orígenes [Azure SQL Database]), haga lo siguiente:

1. Escriba un **nombre** con el que se mostrará el origen de datos en el catálogo.
1. Seleccione **From Azure subscription** (Desde la suscripción de Azure) y la suscripción adecuada en el cuadro desplegable **Suscripción de Azure** y el servidor correspondiente en el cuadro desplegable **Nombre del servidor**.
1. Seleccione **Registrar** para registrar el origen de datos.

:::image type="content" source="media/register-scan-azure-sql-database/add-azure-sql-database.png" alt-text="Opciones de registro de orígenes." border="true":::

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

> [!NOTE]
> * Al eliminar el examen, no se eliminan los recursos de los exámenes de Azure SQL Database anteriores.
> * El recurso no se volverá a actualizar con los cambios de esquema si se cambia la tabla de origen y se vuelve a examinar la tabla de origen después de editar la descripción en la pestaña de esquema de Purview.

## <a name="next-steps"></a>Pasos siguientes

- [Examen del catálogo de datos de Azure Purview](how-to-browse-catalog.md)
- [Búsqueda en el catálogo de datos de Azure Purview](how-to-search-catalog.md)
