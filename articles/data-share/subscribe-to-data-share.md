---
title: 'Tutorial: Aceptación y recepción de datos mediante Azure Data Share'
description: 'Tutorial: Aceptación y recepción de datos mediante Azure Data Share'
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: tutorial
ms.date: 03/24/2021
ms.openlocfilehash: d063a0870616b5b977df18c56d9d66515b03d0a5
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107870866"
---
# <a name="tutorial-accept-and-receive-data-using-azure-data-share"></a>Tutorial: Aceptación y recepción de datos con Azure Data Share  

En este tutorial aprenderá a aceptar una invitación a un recurso compartido de datos mediante Azure Data Share. En él aprenderá a recibir los datos que se hayan compartido contigo, así como a habilitar un intervalo de actualización periódico para asegurarse de que siempre tiene la instantánea más reciente de los datos que se comparten con usted. 

> [!div class="checklist"]
> * Aceptación de una invitación de Azure Data Share
> * Creación de una cuenta de Azure Data Share
> * Especificación de un destino para los datos
> * Creación de una suscripción en su recurso compartido de datos para la actualización programada

## <a name="prerequisites"></a>Prerrequisitos
Para poder aceptar una invitación a compartir datos, previamente debe aprovisionar un número de recursos de Azure, que se enumeran a continuación. 

Asegúrese de que se cumplen todos los requisitos previos antes de aceptar una invitación para compartir datos. 

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una invitación de Azure Data Share: Una invitación de Microsoft Azure con el asunto "Azure Data Share invitation from **<yourdataprovider@domain.com>** " (Invitación para Azure Data Share de yourdataprovider@domain.com).
* Registre el [proveedor de recursos Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) en la suscripción de Azure en la que va a crear un recurso de Data Share y la suscripción de Azure en la que se encuentran los almacenes de datos de Azure de destino.

### <a name="receive-data-into-a-storage-account"></a>Recepción de los datos en una cuenta de almacenamiento

* Una cuenta de Azure Storage: si no la tiene, puede crear una cuenta de [Azure Storage](../storage/common/storage-account-create.md). 
* Permisos para escribir en la cuenta de almacenamiento, que se encuentra en *Microsoft.Storage/storageAccounts/write*. Este permiso existe en el rol de colaborador. 
* Permisos para agregar la asignación de roles a la cuenta de almacenamiento, que se encuentra en *Microsoft.Authorization/role assignments/write*. Este permiso existe en el rol de propietario.  

### <a name="receive-data-into-a-sql-based-target"></a>Recepción de datos en un destino basado en SQL
Si opta por recibir datos en Azure SQL Database, Azure Synapse Analytics, esta es la lista de requisitos previos. 

#### <a name="prerequisites-for-receiving-data-into-azure-sql-database-or-azure-synapse-analytics-formerly-azure-sql-dw"></a>Requisitos previos para recibir datos en Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW)

* Una instancia de Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW).
* Permisos para escribir en las bases de datos de SQL Server, que se encuentra en *Microsoft.Sql/servers/databases/write*. Este permiso existe en el rol de **colaborador**. 
* **Administrador de Azure Active Directory** del servidor SQL Server
* Acceso al firewall de SQL Server. Esto se puede hacer mediante los siguientes pasos: 
    1. En SQL Server en Azure Portal, vaya a *Firewalls y redes virtuales*.
    1. Haga clic en **Sí** en *Permitir que los servicios y recursos de Azure accedan a este servidor*.
    1. Haga clic en **+Agregar IP de cliente**. La dirección IP del cliente está sujeta a cambios. Es posible que se tenga repetir este proceso la próxima vez que comparta datos de SQL desde Azure Portal. También puede agregar un intervalo de direcciones IP.
    1. Haga clic en **Save**(Guardar). 
 
#### <a name="prerequisites-for-receiving-data-into-azure-synapse-analytics-workspace-sql-pool"></a>Requisitos previos para recibir datos en el grupo de SQL de Azure Synapse Analytics (área de trabajo)

* Un grupo de SQL dedicado de Azure Synapse Analytics (área de trabajo). Actualmente, no se admite la recepción de datos en un grupo de SQL sin servidor.
* Permiso para escribir en el grupo de SQL en el área de trabajo de Synapse, que se encuentra en *Microsoft.Synapse/workspaces/sqlPools/write*. Este permiso existe en el rol de **colaborador**.
* Permiso para que la identidad administrada del recurso de Data Share tenga acceso al grupo de SQL del área de trabajo de Synapse. Esto se puede hacer mediante los siguientes pasos: 
    1. En Azure Portal, vaya al área de trabajo de Synapse. Seleccione el administrador de Active Directory de SQL desde el panel de navegación izquierdo y establézcase como **administración de Azure Active Directory**.
    1. Abra Synapse Studio y seleccione *Administrar* en el panel de navegación izquierdo. En Seguridad, seleccione *Control de acceso*. Debe asignarse el rol **administrador de SQL** o **administrador del área de trabajo**.
    1. En Synapse Studio, seleccione *Desarrollar* en el panel de navegación izquierdo. Ejecute el siguiente script en el grupo de SQL para agregar la identidad administrada del recurso de Data Share como "db_datareader, db_datawriter o db_ddladmin". 
    
        ```sql
        create user "<share_acc_name>" from external provider; 
        exec sp_addrolemember db_datareader, "<share_acc_name>"; 
        exec sp_addrolemember db_datawriter, "<share_acc_name>"; 
        exec sp_addrolemember db_ddladmin, "<share_acc_name>";
        ```                   
       Tenga en cuenta que *<share_acc_name>* es el nombre del recurso de Data Share. Si aún no ha creado un recurso de Data Share, puede volver a este requisito previo más adelante.  

* Acceso al firewall del área de trabajo de Synapse. Esto se puede hacer mediante los siguientes pasos: 
    1. En Azure Portal, vaya al área de trabajo de Synapse. Seleccione *Firewalls* en el panel de navegación izquierdo.
    1. En **Permitir que los servicios y recursos de Azure accedan a esta área de trabajo**, haga clic en *Activado*.
    1. Haga clic en **+Agregar IP de cliente**. La dirección IP del cliente está sujeta a cambios. Es posible que se tenga repetir este proceso la próxima vez que comparta datos de SQL desde Azure Portal. También puede agregar un intervalo de direcciones IP.
    1. Haga clic en **Save**(Guardar). 

### <a name="receive-data-into-an-azure-data-explorer-cluster"></a>Recibir datos en un clúster de Azure Data Explorer: 

* Un clúster de Azure Data Explorer en el mismo centro de datos de Azure que el clúster de Data Explorer del proveedor de datos: Si aún no tiene uno, puede crear un [clúster de Azure Data Explorer](/azure/data-explorer/create-cluster-database-portal). Si no conoce el centro de datos de Azure del clúster del proveedor de datos, puede crear el clúster más adelante en el proceso.
* Permisos para escribir en el clúster de Azure Data Explorer, que se encuentra en *Microsoft.Kusto/clusters/write*. Este permiso existe en el rol de colaborador. 

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="open-invitation"></a>Apertura de una invitación

### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Puede abrir la invitación desde el correo electrónico o directamente en Azure Portal. 

   Para abrir la invitación desde el correo electrónico, compruebe la bandeja de entrada para ver si hay una invitación de su proveedor de datos. La invitación es de Microsoft Azure y su título es **Azure Data Share invitation from <yourdataprovider@domain.com>** (Invitación para Azure Data Share de yourdataprovider@domain.com). Haga clic en **Ver invitación** para ver su invitación en Azure. 

   Para abrir la invitación desde Azure Portal directamente, busque **Invitaciones de recurso compartido de datos** en Azure Portal. Esta acción le lleva a la lista de invitaciones de Data Share.

   ![Lista de invitaciones](./media/invitations.png "Lista de invitaciones") 

1. Seleccione el recurso compartido que desea ver. 

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Prepare el entorno de la CLI de Azure y, a continuación, vea las invitaciones.

Empiece por preparar el entorno para la CLI de Azure:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Ejecute el comando [az datashare consumer invitation list](/cli/azure/datashare/consumer/invitation#az_datashare_consumer_invitation_list) para ver las invitaciones actuales:

```azurecli
az datashare consumer invitation list --subscription 11111111-1111-1111-1111-111111111111
```

Copie el identificador de invitación para usarlo en la sección siguiente.

---

## <a name="accept-invitation"></a>Aceptación de la invitación

### <a name="portal"></a>[Portal](#tab/azure-portal)

1. Asegúrese de que se han revisado todos los campos, incluido el de **condiciones de uso**. Si acepta las condiciones de uso, se le solicitará que seleccione la casilla para indicarlo. 

   ![Condiciones de uso](./media/terms-of-use.png "Términos de uso") 

1. En *Target Data Share Account* (Cuenta de Azure Data Share de destino), seleccione la suscripción y el grupo de recursos en los que va a implementar su instancia de Azure Data Share. 

   En el campo **Data Share Account** (Cuenta de Azure Data Share), seleccione **Crear nuevo** si no tiene una cuenta de Azure Data Share existente. En caso contrario, seleccione la cuenta de Azure Data Share existente en la que desea aceptar los datos compartidos. 

   En el campo **Received Share Name** (Nombre del recurso compartido recibido), puede dejar el valor predeterminado que ha especificado el proveedor de datos o especificar un nombre nuevo para el recurso compartido recibido. 

   Cuando haya aceptado los términos de uso y especificado una cuenta de Data Share para administrar el recurso compartido recibido, seleccione **Accept and configure** (Aceptar y configurar). Se creará una suscripción al recurso compartido. 

   ![Opciones de aceptación](./media/accept-options.png "Opciones de aceptación") 

   Esta acción le lleva al recurso compartido recibido en la cuenta de Data Share. 

   Si no desea aceptar la invitación, seleccione *Reject* (Rechazar). 

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el comando [az datashare consumer share-subscription create](/cli/azure/datashare/consumer/share-subscription#az_datashare_consumer_share_subscription_create) para crear la instancia de Data Share.

```azurecli
az datashare consumer share-subscription create --resource-group share-rg \
  --name "Fabrikam Solutions" --account-name FabrikamDataShareAccount \
  --invitation-id 89abcdef-0123-4567-89ab-cdef01234567 \
  --source-share-location "East US 2" --subscription 11111111-1111-1111-1111-111111111111
```

---

## <a name="configure-received-share"></a>Configuración del recurso compartido recibido

### <a name="portal"></a>[Portal](#tab/azure-portal)

Siga los pasos que se indican a continuación para configurar dónde quiere recibir los datos.

1. Seleccione la pestaña **Conjuntos de datos**. Active la casilla situada junto al conjunto de datos al que desearía asignar un destino. Seleccione **+ Asignar a destino** para elegir un almacén de datos de destino. 

   ![Asignación a destino](./media/dataset-map-target.png "Asignar a destino") 

1. Seleccione un tipo de datos de destino en el que quiera colocar los datos. Los archivos de datos o las tablas del almacén de datos de destino se sobrescribirán con la misma ruta de acceso y nombre. Si recibe datos en Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW), active la casilla **Allow Data Share to run the above 'create user' script on my behalf** (Permitir que Data Share ejecute el script "create user" anterior en mi nombre).

   Para el uso compartido en contexto, seleccione un almacén de datos en la ubicación especificada. La ubicación es el centro de datos de Azure donde se encuentra el almacén de datos de origen del proveedor de datos. Una vez que se asigna un conjunto de datos, puede seguir el vínculo de la ruta de acceso de destino para acceder a los datos.

   ![Cuenta de almacenamiento de destino](./media/dataset-map-target-sql.png "Almacenamiento de destino") 

1. En el caso del uso compartido basado en instantáneas, si el proveedor de datos ha creado una programación de instantáneas para proporcionar una actualización periódica de los datos, también puede habilitarla seleccionando la pestaña **Programación de instantáneas**. Active la casilla situada junto a la programación de instantáneas y seleccione **+ Habilitar**. Tenga en cuenta que la primera instantánea programada se iniciará en un minuto desde el tiempo de programación y las instantáneas posteriores comenzarán en cuestión de segundos desde la hora programada.

   ![Habilitación de la programación de instantáneas](./media/enable-snapshot-schedule.png "Habilitación de la programación de instantáneas")

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use estos comandos para configurar dónde quiere recibir los datos.

1. Ejecute el comando [az datashare consumer share-subscription list-source-dataset](/cli/azure/datashare/consumer/share-subscription#az_datashare_consumer_share_subscription_list_source_dataset) para obtener el identificador del conjunto de datos:

   ```azurecli
   az datashare consumer share-subscription list-source-dataset \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" \
     --subscription 11111111-1111-1111-1111-111111111111 --query "[0].dataSetId"
   ```

1. Para crear una cuenta de almacenamiento para esta instancia de Data Share, ejecute el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create):

   ```azurecli
   az storage account create --resource-group "share-rg" --name "FabrikamDataShareAccount" \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

1. Use el comando [az storage account show](/cli/azure/storage/account#az_storage_account_show) para obtener el identificador de la cuenta de almacenamiento:

   ```azurecli
   az storage account show --resource-group "share-rg" --name "FabrikamDataShareAccount" \
     --subscription 11111111-1111-1111-1111-111111111111 --query "id"
   ```

1. Use el siguiente comando para obtener el identificador de la entidad de seguridad de la cuenta:

   ```azurecli
   az datashare account show --resource-group "share-rg" --name "cli_test_consumer_account" \
     --subscription 11111111-1111-1111-1111-111111111111 --query "identity.principalId"
   ```

1. Use el comando [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) para crear una asignación de roles para la entidad de seguridad de la cuenta:

   ```azurecli
   az role assignment create --role "01234567-89ab-cdef-0123-456789abcdef" \
     --assignee-object-id 6789abcd-ef01-2345-6789-abcdef012345 
     --assignee-principal-type ServicePrincipal --scope 456789ab-cdef-0123-4567-89abcdef0123 \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

1. Cree una variable para la asignación basada en el identificador del conjunto de datos:

   ```azurecli
   $mapping='{\"data_set_id\":\"' + $dataset_id + '\",\"container_name\":\"newcontainer\",
     \"storage_account_name\":\"datashareconsumersa\",\"kind\":\"BlobFolder\",\"prefix\":\"consumer\"}'
   ```

1. Use el comando [az datashare consumer dataset-mapping create](/cli/azure/datashare/consumer/dataset-mapping#az_datashare_consumer_dataset_mapping_create) para crear la asignación del conjunto de datos:

   ```azurecli
   az datashare consumer dataset-mapping create --resource-group "share-rg" \
     --name "consumer-data-set-mapping" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" --mapping $mapping \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

1. Ejecute el comando [az datashare consumer share-subscription synchronization start](/cli/azure/datashare/consumer/share-subscription/synchronization#az_datashare_consumer_share_subscription_synchronization_start) para iniciar la sincronización del conjunto de datos.

   ```azurecli
   az datashare consumer share-subscription synchronization start \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount"  \
     --share-subscription-name "Fabrikam Solutions" --synchronization-mode "Incremental" \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

   Ejecute el comando [az datashare consumer share-subscription synchronization list](/cli/azure/datashare/consumer/share-subscription/synchronization#az_datashare_consumer_share_subscription_synchronization_list) para ver una lista de las sincronizaciones:

   ```azurecli
   az datashare consumer share-subscription synchronization list \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" \
     --subscription 11111111-1111-1111-1111-111111111111
   ```

   Use el comando [az datashare consumer share-subscription list-source-share-synchronization-setting](/cli/azure/datashare/consumer/share-subscription#az_datashare_consumer_share_subscription_list_source_share_synchronization_setting) para ver la configuración de sincronización establecida en el recurso compartido.

   ```azurecli
   az datashare consumer share-subscription list-source-share-synchronization-setting \
     --resource-group "share-rg" --account-name "FabrikamDataShareAccount" \
     --share-subscription-name "Fabrikam Solutions" --subscription 11111111-1111-1111-1111-111111111111
   ```

---

## <a name="trigger-a-snapshot"></a>Desencadenamiento de una instantánea

### <a name="portal"></a>[Portal](#tab/azure-portal)

Estos pasos solo se aplican al uso compartido basado en instantáneas.

1. Para desencadenar una instantánea, seleccione la pestaña **Detalles** y, luego, **Desencadenar instantánea**. En este caso, puede desencadenar una instantánea completa o incremental de los datos. Si es la primera vez que recibe datos del proveedor de datos, seleccione una copia completa. 

   ![Desencadenamiento de instantánea](./media/trigger-snapshot.png "Desencadenamiento de instantánea") 

1. Cuando el estado de la última ejecución sea *correcto*, vaya al almacén de datos de destino para ver los datos recibidos. Seleccione **Conjuntos de datos** y haga clic en el vínculo de la ruta de acceso de destino. 

   ![Conjuntos de datos de consumidor](./media/consumer-datasets.png "Asignación de conjunto de datos de consumidor") 

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Ejecute el comando [az datashare consumer trigger create](/cli/azure/datashare/consumer/trigger#az_datashare_consumer_trigger_create) para desencadenar una instantánea:

```azurecli
az datashare consumer trigger create --resource-group "share-rg" \
  --name "share_test_trigger" --account-name "FabrikamDataShareAccount" \
  --share-subscription-name "Fabrikam Solutions" --recurrence-interval "Day" \
  --synchronization-time "2020-04-23 18:00:00 +00:00" --kind ScheduleBased \
  --subscription 11111111-1111-1111-1111-111111111111
```

> [!NOTE]
> Use este comando solo para el uso compartido basado en instantáneas.

---

## <a name="view-history"></a>Visualización del historial
Este paso solo se aplica al uso compartido basado en instantáneas. Para ver el historial de las instantáneas, seleccione la pestaña **Historial**. Aquí encontrará un historial de todas las instantáneas que se generaron en los últimos 30 días.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando el recurso ya no sea necesario, vaya a la página **Información general de Azure Data Share** y seleccione **Eliminar** para quitarlo.

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha aprendido a aceptar y recibir un recurso de Azure Data Share. Para más información sobre los conceptos de Azure Data Share, pase a examinar su terminología.

> [!div class="nextstepaction"]
> [Conceptos de Azure Data Share](terminology.md)