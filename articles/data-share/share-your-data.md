---
title: 'Tutorial: Uso compartido fuera de la organización mediante Azure Data Share'
description: 'Tutorial: Uso compartido de datos con clientes y asociados mediante Azure Data Share'
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: tutorial
ms.date: 03/24/2021
ms.openlocfilehash: b8d49e3b3c6f6407fe241e00ada5039bd94fd706
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107870884"
---
# <a name="tutorial-share-data-using-azure-data-share"></a>Tutorial: Uso compartido de datos mediante Azure Data Share  

En este tutorial aprenderá a configurar una nueva instancia de Azure Data Share y a empezar a compartir sus datos con clientes y asociados externos a la organización de Azure. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una instancia de Azure Data Share.
> * Agregar conjuntos de datos a la instancia de Azure Data Share.
> * Habilitar una programación de instantáneas para la instancia de Data Share. 
> * Agregar destinatarios a una instancia de Azure Data Share. 

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* La dirección de correo electrónico de inicio de sesión de Azure del destinatario (con el alias de correo electrónico no funcionará).
* Si el almacén de datos de Azure de origen está en una suscripción de Azure diferente a la que va a usar para crear el recurso de Data Share, registre el [proveedor de recursos Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) en la suscripción en la que se encuentra el almacén de datos de Azure. 

### <a name="share-from-a-storage-account"></a>Uso compartido desde una cuenta de almacenamiento

* Una cuenta de Azure Storage: si no la tiene, puede crear una cuenta de [Azure Storage](../storage/common/storage-account-create.md)
* Permisos para escribir en la cuenta de almacenamiento, que se encuentra en *Microsoft.Storage/storageAccounts/write*. Este permiso existe en el rol de **colaborador**.
* Permisos para agregar la asignación de roles a la cuenta de almacenamiento, que se encuentra en *Microsoft.Authorization/role assignments/write*. Este permiso existe en el rol de **propietario**. 


### <a name="share-from-a-sql-based-source"></a>Uso compartido desde un origen basado en SQL
A continuación se muestra la lista de requisitos previos para compartir datos desde el origen de SQL. 

#### <a name="prerequisites-for-sharing-from-azure-sql-database-or-azure-synapse-analytics-formerly-azure-sql-dw"></a>Requisitos previos para el uso compartido desde Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW)

* Una instancia de Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW) con las tablas y vistas que quiera compartir.
* Permisos para escribir en las bases de datos de SQL Server, que se encuentra en *Microsoft.Sql/servers/databases/write*. Este permiso existe en el rol de **colaborador**.
* **Administrador de Azure Active Directory** del servidor SQL Server
* Acceso al firewall de SQL Server. Esto se puede hacer mediante los siguientes pasos: 
    1. En Azure Portal, vaya a SQL Server. Seleccione *Firewalls y redes virtuales* en el panel de navegación izquierdo.
    1. Haga clic en **Sí** en *Permitir que los servicios y recursos de Azure accedan a este servidor*.
    1. Haga clic en **+Agregar IP de cliente**. La dirección IP del cliente está sujeta a cambios. Es posible que se tenga repetir este proceso la próxima vez que comparta datos de SQL desde Azure Portal. También puede agregar un intervalo de direcciones IP.
    1. Haga clic en **Save**(Guardar). 

#### <a name="prerequisites-for-sharing-from-azure-synapse-analytics-workspace-sql-pool"></a>Requisitos previos para el uso compartido desde el grupo de SQL de Azure Synapse Analytics (área de trabajo)

* * Un grupo de SQL dedicado de Azure Synapse Analytics (área de trabajo) con tablas que quiera compartir. Actualmente, no se admite el uso compartido de vistas. Actualmente, no se admite el uso compartido desde el grupo de SQL sin servidor.
* Permiso para escribir en el grupo de SQL en el área de trabajo de Synapse, que se encuentra en *Microsoft.Synapse/workspaces/sqlPools/write*. Este permiso existe en el rol de **colaborador**.
* Permiso para que la identidad administrada del recurso de Data Share tenga acceso al grupo de SQL del área de trabajo de Synapse. Esto se puede hacer mediante los siguientes pasos: 
    1. En Azure Portal, vaya al área de trabajo de Synapse. Seleccione el administrador de Active Directory de SQL desde el panel de navegación izquierdo y establézcase como **administración de Azure Active Directory**.
    1. Abra Synapse Studio y seleccione *Administrar* en el panel de navegación izquierdo. En Seguridad, seleccione *Control de acceso*. Debe asignarse el rol **administrador de SQL** o **administrador del área de trabajo**.
    1. En Synapse Studio, seleccione *Desarrollar* en el panel de navegación izquierdo. Ejecute el siguiente script en el grupo de SQL para agregar la identidad administrada del recurso de Data Share como db_datareader. 
    
        ```sql
        create user "<share_acct_name>" from external provider;     
        exec sp_addrolemember db_datareader, "<share_acct_name>"; 
        ```                   
       Tenga en cuenta que *<share_acc_name>* es el nombre del recurso de Data Share. Si aún no ha creado un recurso de Data Share, puede volver a este requisito previo más adelante.  

* Acceso al firewall del área de trabajo de Synapse. Esto se puede hacer mediante los siguientes pasos: 
    1. En Azure Portal, vaya al área de trabajo de Synapse. Seleccione *Firewalls* en el panel de navegación izquierdo.
    1. En **Permitir que los servicios y recursos de Azure accedan a esta área de trabajo**, haga clic en *Activado*.
    1. Haga clic en **+Agregar IP de cliente**. La dirección IP del cliente está sujeta a cambios. Es posible que se tenga repetir este proceso la próxima vez que comparta datos de SQL desde Azure Portal. También puede agregar un intervalo de direcciones IP.
    1. Haga clic en **Save**(Guardar). 


### <a name="share-from-azure-data-explorer"></a>Uso compartido desde Azure Data Explorer
* Un clúster de Azure Data Explorer con las bases de datos que desea compartir.
* Permisos para escribir en el clúster de Azure Data Explorer, que se encuentra en *Microsoft.Kusto/clusters/write*. Este permiso existe en el rol de **colaborador**.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="create-a-data-share-account"></a>Creación de una cuenta de Azure Data Share

### <a name="portal"></a>[Portal](#tab/azure-portal)

Cree un recurso de Azure Data Share en un grupo de recursos de Azure.

1. Seleccione el botón de menú de la esquina superior izquierda del portal y, a continuación, seleccione **Crear un recurso**.

1. Busque *Data Share*.

1. Seleccione Data Share y, después, **Crear**.

1. Rellene los detalles básicos del recurso de Azure Data Share con la siguiente información. 

     **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | Subscription | Su suscripción | Seleccione la suscripción de Azure que desea usar para la cuenta del recurso compartido de datos.|
    | Grupo de recursos | *testresourcegroup* | Use un grupo de recursos existente o cree uno. |
    | Location | *Este de EE. UU. 2* | Seleccione una región para la cuenta de Azure Data Share.
    | Nombre | *datashareaccount* | Especifique un nombre para la cuenta del recurso compartido de datos. |
    | | |

1. Seleccione **Revisar + Crear** y, después, **Crear** para aprovisionar la cuenta del recurso compartido de datos. El aprovisionamiento de una nueva cuenta de Azure Data Share normalmente tarda aproximadamente 2 minutos, o menos. 

1. Cuando la implementación se complete, seleccione **Ir al grupo de recursos**.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Cree un recurso de Azure Data Share en un grupo de recursos de Azure.

Empiece por preparar el entorno para la CLI de Azure:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Use estos comandos para crear el recurso:

1. Use el comando [az account set](/cli/azure/account#az_account_set) para establecer que la suscripción sea la suscripción predeterminada actual:

   ```azurecli
   az account set --subscription 00000000-0000-0000-0000-000000000000
   ```

1. Para registrar el proveedor de recursos, ejecute el comando [az provider register](/cli/azure/provider#az_provider_register).

   ```azurecli
   az provider register --name "Microsoft.DataShare"
   ```

1. Ejecute el comando [az group create](/cli/azure/group#az_group_create) para crear un grupo de recursos o use un grupo de recursos existente:

   ```azurecli
   az group create --name testresourcegroup --location "East US 2"
   ```

1. Ejecute el comando [az datashare account create](/cli/azure/datashare/account#az_datashare_account_create) para crear una cuenta de Data Share:

   ```azurecli
   az datashare account create --resource-group testresourcegroup --name datashareaccount --location "East US 2" 
   ```

   Ejecute el comando [az datashare account list](/cli/azure/datashare/account#az_datashare_account_list) para ver las cuentas de Data Share:

   ```azurecli
   az datashare account list --resource-group testresourcegroup
   ```

---

## <a name="create-a-share"></a>Creación de un recurso compartido

### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Vaya a la página de información general de Azure Data Share.

    ![Uso compartido de los datos](./media/share-receive-data.png "Uso compartido de los datos") 

1. Seleccione **Start sharing your data** (Empezar a compartir datos).

1. Seleccione **Crear**.   

1. Rellene los detalles del recurso compartido. Especifique un nombre, un tipo de recurso compartido, una descripción del contenido de los recursos compartidos y las condiciones de uso (opcional). 

    ![EnterShareDetails](./media/enter-share-details.png "Introducción de detalles del recurso compartido") 

1. Seleccione **Continuar**.

1. Para agregar conjuntos de datos al recurso compartido, seleccione **Agregar conjuntos de datos**. 

    ![Adición de conjuntos de datos al recurso compartido](./media/datasets.png "Conjuntos de datos")

1. Seleccione el tipo de conjunto de datos que desea agregar. Verá una lista diferente de tipos de conjuntos de datos en función del tipo de recurso compartido (instantánea o en contexto) que haya seleccionado en el paso anterior. Si comparte desde Azure SQL Database o Azure Synapse Analytics (anteriormente conocido como Azure SQL DW), se le pedirá el método de autenticación para mostrar las tablas. Seleccione la autenticación de AAD y active la casilla **Allow Data Share to run the above 'create user' script on my behalf** (Permitir que Data Share se ejecute en el script de "creación de usuario" anterior). 

    ![AddDatasets](./media/add-datasets.png "Incorporación de conjuntos de datos")    

1. Vaya hasta el objeto que desea compartir y seleccione "Add Datasets" (Agregar conjuntos de datos). 

    ![SelectDatasets](./media/select-datasets.png "Selección de conjuntos de datos")    

1. En la pestaña Destinatarios, escriba las direcciones de correo electrónico del consumidor de datos, para lo que debe seleccionar "+ Add Recipient" (+ Agregar a destinatario). 

    ![AddRecipients](./media/add-recipient.png "Adición de destinatarios") 

1. Seleccione **Continuar**.

1. Si ha seleccionado el tipo de recurso compartido de instantáneas, puede configurar la programación de instantáneas para proporcionar actualizaciones de los datos al consumidor de datos. 

    ![EnableSnapshots](./media/enable-snapshots.png "Habilitación de instantáneas") 

1. Seleccione una hora de inicio y un intervalo de periodicidad. 

1. Seleccione **Continuar**.

1. En la pestaña Revisar y crear, examine el contenido del paquete, la configuración, los destinatarios y la configuración de la sincronización. Seleccione **Crear**.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Ejecute el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create) para crear una instancia de Data Share:

   ```azurecli
   az storage account create --resource-group testresourcegroup --name ContosoMarketplaceAccount
   ```

1. Use el comando [az storage container create](/cli/azure/storage/container#az_storage_container_create) para crear un contenedor para el recurso compartido del comando anterior:

   ```azurecli
   az storage container create --name ContosoMarketplaceContainer --account-name ContosoMarketplaceAccount
   ```

1. Ejecute el comando [az datashare create](/cli/azure/datashare#az_datashare_create) para crear la instancia de Data Share:

   ```azurecli
   az datashare create --resource-group testresourcegroup \
     --name ContosoMarketplaceDataShare --account-name ContosoMarketplaceAccount \
     --description "Data Share" --share-kind "CopyBased" --terms "Confidential"
   ```

1. Use el comando [az datashare invitation create](/cli/azure/datashare/invitation#az_datashare_invitation_create) para crear la invitación para la dirección especificada:

   ```azurecli
   az datashare invitation create --resource-group testresourcegroup \
     --name DataShareInvite --share-name ContosoMarketplaceDataShare \
     --account-name ContosoMarketplaceAccount --target-email "jacob@fabrikam"
   ```

---

Ya se ha creado Azure Data Share y el destinatario de Data Share ya está listo para aceptar la invitación.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando el recurso ya no sea necesario, vaya a la página **Información general de Azure Data Share** y seleccione **Eliminar** para quitarlo.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a crear una instancia de Azure Data Share y a invitar a destinatarios. Para obtener información acerca de la forma en que in consumidor de datos puede aceptar y recibir un recurso compartido de datos, vaya al tutorial para aceptar y recibir datos.

> [!div class="nextstepaction"]
> [Tutorial: Aceptación y recepción de datos con Azure Data Share](subscribe-to-data-share.md)
