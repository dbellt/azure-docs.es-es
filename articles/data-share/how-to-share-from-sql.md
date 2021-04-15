---
title: Uso compartido y recepción de datos de Azure SQL Database y Azure Synapse Analytics
description: Aprenda a compartir y recibir datos de Azure SQL Database y Azure Synapse Analytics
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 02/24/2021
ms.openlocfilehash: ef8c1a50cd3568c6cec9bdb053b02e6e14741eb0
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105644671"
---
# <a name="share-and-receive-data-from-azure-sql-database-and-azure-synapse-analytics"></a>Uso compartido y recepción de datos de Azure SQL Database y Azure Synapse Analytics

[!INCLUDE[appliesto-sql](includes/appliesto-sql.md)]

Azure Data Share admite el uso compartido basado en instantáneas de Azure SQL Database y Azure Synapse Analytics. En este artículo se explica cómo compartir y recibir datos desde estos orígenes.

Azure Data Share admite el uso compartido de tablas y vistas desde Azure SQL Database y Azure Synapse Analytics (anteriormente Azure SQL DW) y el uso compartido de tablas desde el grupo de SQL dedicado de Azure Synapse Analytics (área de trabajo). Actualmente, no se admite el uso compartido desde el grupo de SQL sin servidor de Azure Synapse Analytics (área de trabajo). Los consumidores de datos pueden elegir aceptar los datos en Azure Data Lake Storage Gen2 o Azure Blob Storage como archivo CSV o Parquet, así como en Azure SQL Database y Azure Synapse Analytics como tablas.

Al aceptar los datos en Azure Data Lake Store Gen2 o Azure Blob Storage, las instantáneas completas sobrescriben el contenido del archivo de destino si este ya existe.
Cuando se reciben datos en la tabla SQL y, si la tabla de destino no existe aún, Azure Data Share crea la tabla SQL con el esquema de origen. Si ya existe una tabla de destino con el mismo nombre, se quitará y se sobrescribirá con la instantánea completa más reciente. Actualmente no se admiten instantáneas incrementales.

## <a name="share-data"></a>Compartir datos

### <a name="prerequisites-to-share-data"></a>Requisitos previos para compartir datos

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* La dirección de correo electrónico de inicio de sesión de Azure del destinatario (con el alias de correo electrónico no funcionará).
* Si el almacén de datos de Azure de origen está en una suscripción de Azure diferente a la que va a usar para crear el recurso de Data Share, registre el [proveedor de recursos Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) en la suscripción en la que se encuentra el almacén de datos de Azure. 

### <a name="prerequisites-for-sql-source"></a>Requisitos previos para un origen de SQL
A continuación se muestra la lista de requisitos previos para compartir datos desde el origen de SQL. 

#### <a name="prerequisites-for-sharing-from-azure-sql-database-or-azure-synapse-analytics-formerly-azure-sql-dw"></a>Requisitos previos para el uso compartido desde Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW)


Para compartir datos con la autenticación de Azure Active Directory, aquí se muestra una lista de requisitos previos:

* Una instancia de Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW) con las tablas y vistas que quiera compartir.
* Permisos para escribir en las bases de datos de SQL Server, que se encuentra en *Microsoft.Sql/servers/databases/write*. Este permiso existe en el rol de **colaborador**.
* **Administración de Azure Active Directory** de SQL Server
* Acceso al firewall de SQL Server. Esto se puede hacer mediante los siguientes pasos: 
    1. En Azure Portal, vaya a SQL Server. Seleccione *Firewalls y redes virtuales* en el panel de navegación izquierdo.
    1. Haga clic en **Sí** en *Permitir que los servicios y recursos de Azure accedan a este servidor*.
    1. Haga clic en **+Agregar IP de cliente**. La dirección IP del cliente está sujeta a cambios. Es posible que se tenga repetir este proceso la próxima vez que comparta datos de SQL desde Azure Portal. También puede agregar un intervalo de direcciones IP.
    1. Haga clic en **Save**(Guardar). 

Para compartir datos mediante la autenticación de SQL, a continuación, se muestra una lista de requisitos previos. Puede seguir la [demostración paso a paso](https://youtu.be/hIE-TjJD8Dc) para configurar los requisitos previos.

* Una instancia de Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW) con las tablas y vistas que quiera compartir.
* Permisos para escribir en las bases de datos de SQL Server, que se encuentra en *Microsoft.Sql/servers/databases/write*. Este permiso existe en el rol de **colaborador**.
* Permiso para que la identidad administrada del recurso de Data Share tenga acceso a la base de datos. Esto se puede hacer mediante los siguientes pasos: 
    1. En Azure Portal, vaya al servidor SQL Server y establézcase como **administrador de Azure Active Directory**.
    1. Conéctese a Azure SQL Database/Data Warehouse con el [Editor de consultas](../azure-sql/database/connect-query-portal.md#connect-using-azure-active-directory) o SQL Server Management Studio mediante la autenticación de Azure Active Directory. 
    1. Ejecute el siguiente script para agregar la identidad administrada del recurso de Data Share como db_datareader. Debe conectarse mediante Active Directory y no con la autenticación de SQL Server. 
    
        ```sql
        create user "<share_acct_name>" from external provider;     
        exec sp_addrolemember db_datareader, "<share_acct_name>"; 
        ```                   
       Tenga en cuenta que *<share_acc_name>* es el nombre del recurso de Data Share. Si aún no ha creado un recurso de Data Share, puede volver a este requisito previo más adelante.  

* Un usuario de Azure SQL Database con acceso **"db_datareader"** para explorar y seleccionar las tablas o vistas que quiera compartir. 

* Acceso al firewall de SQL Server. Esto se puede hacer mediante los siguientes pasos: 
    1. En Azure Portal, vaya a SQL Server. Seleccione *Firewalls y redes virtuales* en el panel de navegación izquierdo.
    1. Haga clic en **Sí** en *Permitir que los servicios y recursos de Azure accedan a este servidor*.
    1. Haga clic en **+Agregar IP de cliente**. La dirección IP del cliente está sujeta a cambios. Es posible que se tenga repetir este proceso la próxima vez que comparta datos de SQL desde Azure Portal. También puede agregar un intervalo de direcciones IP.
    1. Haga clic en **Save**(Guardar). 

#### <a name="prerequisites-for-sharing-from-azure-synapse-analytics-workspace-sql-pool"></a>Requisitos previos para el uso compartido desde el grupo de SQL de Azure Synapse Analytics (área de trabajo)

* Un grupo de SQL dedicado de Azure Synapse Analytics (área de trabajo) con tablas que quiera compartir. Actualmente, no se admite el uso compartido de vistas. Actualmente, no se admite el uso compartido desde el grupo de SQL sin servidor.
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

### <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

### <a name="create-a-data-share-account"></a>Creación de una cuenta de Azure Data Share

Cree un recurso de Azure Data Share en un grupo de recursos de Azure.

1. Seleccione el botón de menú de la esquina superior izquierda del portal y, a continuación, seleccione **Crear un recurso**.

1. Busque *Data Share*.

1. Seleccione Data Share y, después, **Crear**.

1. Rellene los detalles básicos del recurso de Azure Data Share con la siguiente información. 

     **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | Subscription | Su suscripción | Seleccione la suscripción de Azure que desea usar para la cuenta del recurso compartido de datos.|
    | Grupo de recursos | *test-resource-group* | Use un grupo de recursos existente o cree uno. |
    | Location | *Este de EE. UU. 2* | Seleccione una región para la cuenta de Azure Data Share.
    | Nombre | *datashareaccount* | Especifique un nombre para la cuenta del recurso compartido de datos. |
    | | |

1. Seleccione **Revisar + Crear** y, después, **Crear** para aprovisionar la cuenta del recurso compartido de datos. El aprovisionamiento de una nueva cuenta de Azure Data Share normalmente tarda aproximadamente 2 minutos, o menos. 

1. Cuando la implementación se complete, seleccione **Ir al grupo de recursos**.

### <a name="create-a-share"></a>Creación de un recurso compartido

1. Vaya a la página de información general de Azure Data Share.

    ![Uso compartido de los datos](./media/share-receive-data.png "Uso compartido de los datos") 

1. Seleccione **Start sharing your data** (Empezar a compartir datos).

1. Seleccione **Crear**.   

1. Rellene los detalles del recurso compartido. Especifique un nombre, un tipo de recurso compartido, una descripción del contenido de los recursos compartidos y las condiciones de uso (opcional). 

    ![EnterShareDetails](./media/enter-share-details.png "Introducción de detalles del recurso compartido") 

1. Seleccione **Continuar**.

1. Para agregar conjuntos de datos al recurso compartido, seleccione **Agregar conjuntos de datos**. 

    ![Adición de conjuntos de datos al recurso compartido](./media/datasets.png "Conjuntos de datos")

1. Seleccione el tipo de conjunto de datos que desea agregar. Verá una lista diferente de tipos de conjuntos de datos en función del tipo de recurso compartido (instantánea o en contexto) que haya seleccionado en el paso anterior. 

    ![AddDatasets](./media/add-datasets.png "Incorporación de conjuntos de datos")    

1. Seleccione el área de trabajo de SQL Server o Synapse. Si usa la autenticación de AAD y aparece la casilla **Allow Data Share to run the above 'create user' SQL script on my behalf** (Permitir que Data Share ejecute el script de SQL "create user" anterior en mi nombre), actívela. Si usa la autenticación de SQL, proporcione credenciales y siga los pasos descritos en los requisitos previos para ejecutar el script que aparece en la pantalla. Esta acción concede al recurso de Data Share permiso para leer la base de datos SQL. 

   Seleccione **Siguiente** para ir al objeto que quiere compartir y elija "Agregar conjuntos de datos". Puede seleccionar tablas y vistas de Azure SQL Database y Azure Synapse Analytics (anteriormente Azure SQL DW) o tablas del grupo de SQL dedicado de Azure Synapse Analytics (área de trabajo). 

    ![SelectDatasets](./media/select-datasets-sql.png "Selección de conjuntos de datos")    

1. En la pestaña Destinatarios, escriba las direcciones de correo electrónico del consumidor de datos, para lo que debe seleccionar "+ Add Recipient" (+ Agregar a destinatario). La dirección de correo electrónico debe ser el correo electrónico de inicio de sesión de Azure del destinatario.

    ![AddRecipients](./media/add-recipient.png "Adición de destinatarios") 

1. Seleccione **Continuar**.

1. Si ha seleccionado el tipo de recurso compartido de instantáneas, puede configurar la programación de instantáneas para proporcionar actualizaciones de los datos al consumidor de datos. 

    ![EnableSnapshots](./media/enable-snapshots.png "Habilitación de instantáneas") 

1. Seleccione una hora de inicio y un intervalo de periodicidad. 

1. Seleccione **Continuar**.

1. En la pestaña Revisar y crear, examine el contenido del paquete, la configuración, los destinatarios y la configuración de la sincronización. Seleccione **Crear**.

Ya se ha creado Azure Data Share y el destinatario de Data Share ya está listo para aceptar la invitación. 

## <a name="receive-data"></a>Recibir datos

### <a name="prerequisites-to-receive-data"></a>Requisitos previos para recibir datos
Para poder aceptar una invitación a compartir datos, previamente debe aprovisionar un número de recursos de Azure, que se enumeran a continuación. 

Asegúrese de que se cumplen todos los requisitos previos antes de aceptar una invitación para compartir datos. 

* Suscripción de Azure: Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una invitación de Azure Data Share: Una invitación de Microsoft Azure con el asunto "Azure Data Share invitation from **<yourdataprovider@domain.com>** " (Invitación para Azure Data Share de yourdataprovider@domain.com).
* Registre el [proveedor de recursos Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) en la suscripción de Azure en la que va a crear un recurso de Data Share y la suscripción de Azure en la que se encuentran los almacenes de datos de Azure de destino.

### <a name="prerequisites-for-target-storage-account"></a>Requisitos previos de la cuenta de almacenamiento de destino
Si elige recibir datos en Azure Storage, a continuación se muestra la lista de requisitos previos.

* Una cuenta de Azure Storage: si no la tiene, puede crear una cuenta de [Azure Storage](../storage/common/storage-account-create.md). 
* Permisos para escribir en la cuenta de almacenamiento, que se encuentra en *Microsoft.Storage/storageAccounts/write*. Este permiso existe en el rol de **colaborador**. 
* Permiso para agregar la asignación de roles de la identidad administrada del recurso de Data Share a la cuenta de almacenamiento, que se encuentra en *Microsoft.Authorization/role assignments/write*. Este permiso existe en el rol de **propietario**.  

### <a name="prerequisites-for-sql-target"></a>Requisitos previos para un destino de SQL
Si opta por recibir datos en Azure SQL Database, Azure Synapse Analytics, esta es la lista de requisitos previos. 

#### <a name="prerequisites-for-receiving-data-into-azure-sql-database-or-azure-synapse-analytics-formerly-azure-sql-dw"></a>Requisitos previos para recibir datos en Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW)

Para recibir datos en un servidor SQL Server donde sea el **administrador de Azure Active Directory** de dicho servidor, se necesitan los siguientes requisitos previos:

* Una instancia de Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW).
* Permisos para escribir en las bases de datos de SQL Server, que se encuentra en *Microsoft.Sql/servers/databases/write*. Este permiso existe en el rol de **colaborador**.
* Acceso al firewall de SQL Server. Esto se puede hacer mediante los siguientes pasos: 
    1. En Azure Portal, vaya a SQL Server. Seleccione *Firewalls y redes virtuales* en el panel de navegación izquierdo.
    1. Haga clic en **Sí** en *Permitir que los servicios y recursos de Azure accedan a este servidor*.
    1. Haga clic en **+Agregar IP de cliente**. La dirección IP del cliente está sujeta a cambios. Es posible que se tenga repetir este proceso la próxima vez que comparta datos de SQL desde Azure Portal. También puede agregar un intervalo de direcciones IP.
    1. Haga clic en **Save**(Guardar). 
    
Para recibir datos en un servidor SQL Server donde no sea el **administrador de Azure Active Directory**, se necesitan los siguientes requisitos previos. Puede seguir la [demostración paso a paso](https://youtu.be/aeGISgK1xro) para configurar los requisitos previos.

* Una instancia de Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW).
* Permisos para escribir en las bases de datos de SQL Server, que se encuentra en *Microsoft.Sql/servers/databases/write*. Este permiso existe en el rol de **colaborador**. 
* Permiso para que la identidad administrada del recurso compartido de Data Share tenga acceso a Azure SQL Database o Azure Synapse Analytics. Esto se puede hacer mediante los siguientes pasos: 
    1. En Azure Portal, vaya al servidor SQL Server y establézcase como **administrador de Azure Active Directory**.
    1. Conéctese a Azure SQL Database/Data Warehouse con el [Editor de consultas](../azure-sql/database/connect-query-portal.md#connect-using-azure-active-directory) o SQL Server Management Studio mediante la autenticación de Azure Active Directory. 
    1. Ejecute el siguiente script para agregar la identidad administrada de Data Share como "db_datareader, db_datawriter o db_ddladmin". Debe conectarse mediante Active Directory y no con la autenticación de SQL Server. 

        ```sql
        create user "<share_acc_name>" from external provider; 
        exec sp_addrolemember db_datareader, "<share_acc_name>"; 
        exec sp_addrolemember db_datawriter, "<share_acc_name>"; 
        exec sp_addrolemember db_ddladmin, "<share_acc_name>";
        ```      
        Tenga en cuenta que *<share_acc_name>* es el nombre del recurso de Data Share. Si aún no ha creado un recurso de Data Share, puede volver a este requisito previo más adelante.         

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

### <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

### <a name="open-invitation"></a>Apertura de una invitación

1. Puede abrir la invitación desde el correo electrónico o directamente en Azure Portal. 

   Para abrir la invitación desde el correo electrónico, compruebe la bandeja de entrada para ver si hay una invitación de su proveedor de datos. La invitación es de Microsoft Azure y su título es **Azure Data Share invitation from <yourdataprovider@domain.com>** (Invitación para Azure Data Share de yourdataprovider@domain.com). Haga clic en **Ver invitación** para ver su invitación en Azure. 

   Para abrir la invitación desde Azure Portal directamente, busque **Invitaciones de recurso compartido de datos** en Azure Portal. Este paso le lleva a la lista de invitaciones de Data Share.

   ![Lista de invitaciones](./media/invitations.png "Lista de invitaciones") 

1. Seleccione el recurso compartido que desea ver. 

### <a name="accept-invitation"></a>Aceptación de la invitación
1. Asegúrese de que se han revisado todos los campos, incluido el de **condiciones de uso**. Si acepta las condiciones de uso, se le solicitará que seleccione la casilla para indicarlo. 

   ![Condiciones de uso](./media/terms-of-use.png "Términos de uso") 

1. En *Target Data Share Account* (Cuenta de Azure Data Share de destino), seleccione la suscripción y el grupo de recursos en los que va a implementar su instancia de Azure Data Share. 

   En el campo **Data Share Account** (Cuenta de Azure Data Share), seleccione **Crear nuevo** si no tiene una cuenta de Azure Data Share existente. En caso contrario, seleccione la cuenta de Azure Data Share existente en la que desea aceptar los datos compartidos. 

   En el campo **Received Share Name** (Nombre del recurso compartido recibido), puede dejar el valor predeterminado que ha especificado el proveedor de datos o especificar un nombre nuevo para el recurso compartido recibido. 

   Cuando haya aceptado los términos de uso y especificado una cuenta de Data Share para administrar el recurso compartido recibido, seleccione **Accept and configure** (Aceptar y configurar). Se creará una suscripción al recurso compartido. 

   ![Opciones de aceptación](./media/accept-options.png "Opciones de aceptación") 

   Esta opción le lleva al recurso compartido recibido en la cuenta de Data Share. 

   Si no desea aceptar la invitación, seleccione *Reject* (Rechazar). 

### <a name="configure-received-share"></a>Configuración del recurso compartido recibido
Siga los pasos que se indican a continuación para configurar dónde quiere recibir los datos.

1. Seleccione la pestaña **Conjuntos de datos**. Active la casilla situada junto al conjunto de datos al que desearía asignar un destino. Seleccione **+ Asignar a destino** para elegir un almacén de datos de destino. 

   ![Asignación a destino](./media/dataset-map-target.png "Asignar a destino") 

1. Seleccione un almacén de datos de destino en el que quiera colocar los datos. Los archivos de datos o las tablas del almacén de datos de destino se sobrescribirán con la misma ruta de acceso y nombre. Si va a recibir datos en el destino de SQL y aparece la casilla **Allow Data Share to run the above 'create user' SQL script on my behalf** (Permitir que Data Share ejecute el script de SQL "create user" anterior en mi nombre), actívela. De lo contrario, siga las instrucciones de los requisitos previos para ejecutar el script que aparece en la pantalla. De esta forma, el recurso de Data Share tendrá permiso de escritura para la base de datos SQL de destino.

   ![Cuenta de almacenamiento de destino](./media/dataset-map-target-sql.png "Almacén de datos de destino") 

1. En el caso del uso compartido basado en instantáneas, si el proveedor de datos ha creado una programación de instantáneas para proporcionar una actualización periódica de los datos, también puede habilitarla seleccionando la pestaña **Programación de instantáneas**. Active la casilla situada junto a la programación de instantáneas y seleccione **+ Habilitar**. Tenga en cuenta que la primera instantánea programada se iniciará en un minuto desde el tiempo de programación y las instantáneas posteriores comenzarán en cuestión de segundos desde la hora programada.

   ![Habilitación de la programación de instantáneas](./media/enable-snapshot-schedule.png "Habilitación de la programación de instantáneas")

### <a name="trigger-a-snapshot"></a>Desencadenamiento de una instantánea
Estos pasos solo se aplican al uso compartido basado en instantáneas.

1. Para desencadenar una instantánea, seleccione la pestaña **Detalles** y, luego, **Desencadenar instantánea**. En este caso, puede desencadenar una instantánea completa o incremental de los datos. Si es la primera vez que recibe datos del proveedor de datos, seleccione una copia completa. En el caso de los orígenes de SQL, solo se admite la instantánea completa. Cuando se ejecuta una instantánea, las posteriores no se iniciarán hasta que esta se complete.

   ![Desencadenamiento de instantánea](./media/trigger-snapshot.png "Desencadenamiento de instantánea") 

1. Cuando el estado de la última ejecución sea *correcto*, vaya al almacén de datos de destino para ver los datos recibidos. Seleccione **Conjuntos de datos** y haga clic en el vínculo de la ruta de acceso de destino. 

   ![Conjuntos de datos de consumidor](./media/consumer-datasets.png "Asignación de conjunto de datos de consumidor") 

### <a name="view-history"></a>Visualización del historial
Este paso solo se aplica al uso compartido basado en instantáneas. Para ver el historial de las instantáneas, seleccione la pestaña **Historial**. Aquí encontrará un historial de todas las instantáneas que se generaron en los últimos 30 días. 

## <a name="supported-data-types"></a>Tipos de datos admitidos
Al compartir datos desde el origen de SQL, se usa la siguiente asignación de los tipos de datos de SQL Server a los tipos de datos provisionales de Azure Data Share durante el proceso de instantánea. 

| Tipos de datos de SQL Server | Tipo de datos provisional de Azure Data Share |
|:--- |:--- |
| bigint |Int64 |
| binary |Byte[] |
| bit |Boolean |
| char |String, Char[] |
| date |DateTime |
| Datetime |DateTime |
| datetime2 |DateTime |
| Datetimeoffset |DateTimeOffset |
| Decimal |Decimal |
| FILESTREAM attribute (varbinary(max)) |Byte[] |
| Float |Double |
| imagen |Byte[] |
| int |Int32 |
| money |Decimal |
| NCHAR |String, Char[] |
| ntext |String, Char[] |
| NUMERIC |Decimal |
| NVARCHAR |String, Char[] |
| real |Single |
| rowversion |Byte[] |
| smalldatetime |DateTime |
| SMALLINT |Int16 |
| SMALLMONEY |Decimal |
| sql_variant |Object |
| text |String, Char[] |
| time |TimeSpan |
| timestamp |Byte[] |
| TINYINT |Int16 |
| UNIQUEIDENTIFIER |Guid |
| varbinary |Byte[] |
| varchar |String, Char[] |
| Xml |String |

>[!NOTE]
> 1. En el caso de los tipos de datos que se asignan al tipo decimal provisional, la instantánea actualmente admite una precisión de hasta 28. Si tiene datos que requieren una precisión mayor que 28, considere la posibilidad de convertir a una cadena. 
> 1.  Si va a compartir datos de Azure SQL Database en Azure Synapse Analytics, no se admiten todos los tipos de datos. Vea [Tipos de datos de tabla en el grupo de SQL dedicado](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-data-types.md) para obtener información. 

## <a name="sql-always-encrypted-or-dynamic-data-masking"></a>Always Encrypted o enmascaramiento dinámico de datos de SQL
Actualmente, Azure Data Share no es compatible con las bases de datos de Azure SQL con Always Encrypted configurado. 

En el caso de las tablas SQL de origen con enmascaramiento dinámico de datos, los datos aparecerán enmascarados en el lado del destinatario.

## <a name="sql-snapshot-performance"></a>Rendimiento de la instantánea de SQL
El rendimiento de la instantánea de SQL se ve afectado por una serie de factores. Siempre se recomienda llevar a cabo sus propias pruebas de rendimiento. A continuación, se muestran algunos factores de ejemplo que afectan al rendimiento.

* Configuración del hardware (por ejemplo, núcleos virtuales, memoria, DWU) del almacén de datos de SQL de origen y de destino. 
* Acceso simultáneo a los almacenes de datos de origen y de destino. Si comparte varias tablas y vistas desde el mismo almacén de datos de SQL, o recibe varias tablas y vistas en el mismo almacén de datos de SQL, el rendimiento se verá afectado.   
* Ubicación de los almacenes de datos de origen y de destino. 

## <a name="troubleshoot-sql-snapshot-failure"></a>Solución de problemas de error de la instantánea de SQL
La causa más común de error de la instantánea es que Data Share no tiene permiso para el almacén de datos de origen o de destino. Para conceder a Data Share permiso para la instancia de Azure SQL Database o Azure Synapse Analytics (anteriormente Azure SQL DW) de origen o de destino, debe ejecutar el script de SQL proporcionado al conectarse a la base de datos de SQL mediante la autenticación de Azure Active Directory. Para solucionar otro error de instantánea de SQL, consulte [Solucionar problemas de error de instantánea](data-share-troubleshoot.md#snapshots).

## <a name="next-steps"></a>Pasos siguientes
Ha obtenido información sobre cómo compartir y recibir datos de orígenes de almacenamiento mediante el servicio Azure Data Share. Para más información sobre el uso compartido desde otros orígenes de datos, vaya a los [almacenes de datos compatibles](supported-data-stores.md).