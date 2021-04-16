---
title: Uso compartido y recepción de datos de Azure Blob Storage y Azure Data Lake Storage
description: Aprenda a compartir y recibir datos de Azure Blob Storage y Azure Data Lake Storage.
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: how-to
ms.date: 02/23/2021
ms.openlocfilehash: c94d6f7d4d06d7dbaa6a2ad54f94575a6cc85d33
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105644664"
---
# <a name="share-and-receive-data-from-azure-blob-storage-and-azure-data-lake-storage"></a>Uso compartido y recepción de datos de Azure Blob Storage y Azure Data Lake Storage

[!INCLUDE[appliesto-storage](includes/appliesto-storage.md)]

Azure Data Share admite el uso compartido basado en instantáneas desde una cuenta de almacenamiento. En este artículo se explica cómo compartir y recibir datos de Azure Blob Storage, Azure Data Lake Storage Gen1 y Azure Data Lake Storage Gen2.

Azure Data Share admite el uso compartido de archivos, carpetas y sistemas de archivos desde Azure Data Lake Gen1 y Azure Data Lake Gen2. También admite el uso compartido de blobs, carpetas y contenedores desde Azure Blob Storage. Actualmente solo se admiten blobs en bloques. Los datos compartidos de estos orígenes pueden recibirse en Azure Data Lake Gen2 o Azure Blob Storage.

Cuando el uso compartido de sistemas de archivos, contenedores o carpetas se basa en instantáneas, los consumidores de datos pueden optar por hacer una copia completa de los datos de recursos compartidos. O bien, pueden usar la funcionalidad de instantáneas incrementales para copiar solo los archivos nuevos o actualizados. La funcionalidad de instantánea incremental se basa en la hora de la última modificación de los archivos. 

Los archivos existentes que tengan el mismo nombre se sobrescribirán durante una instantánea. Un archivo que se elimina del origen no se elimina en el destino. Las subcarpetas vacías del origen no se copian en el destino. 

## <a name="share-data"></a>Compartir datos

Use la información de las secciones siguientes para compartir datos con Azure Data Share. 
### <a name="prerequisites-to-share-data"></a>Requisitos previos para compartir datos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Busque la dirección de correo electrónico de inicio de sesión de Azure del destinatario. El alias de correo electrónico del destinatario no servirá para sus fines.
* Si el almacén de datos de Azure de origen está en una suscripción de Azure diferente a la que va a usar para crear el recurso de Data Share, registre el [proveedor de recursos Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) en la suscripción en la que se encuentra el almacén de datos de Azure. 

### <a name="prerequisites-for-the-source-storage-account"></a>Requisitos previos de la cuenta de almacenamiento de origen

* Una cuenta de Azure Storage. Si aún no tiene una cuenta, [cree una](../storage/common/storage-account-create.md).
* Permiso para escribir en la cuenta de almacenamiento. El permiso de escritura está en *Microsoft.Storage/storageAccounts/write*. Forma parte del rol de colaborador.
* Permiso para agregar asignación de roles a la cuenta de almacenamiento. Este permiso se encuentra en *de Microsoft.Authorization/role assignments/write*. Forma parte del rol de propietario. 

### <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

### <a name="create-a-data-share-account"></a>Creación de una cuenta de Data Share

Cree un recurso de Azure Data Share en un grupo de recursos de Azure.

1. En la esquina superior izquierda del portal, abra el menú y seleccione **Crear un recurso** (+).

1. Busque *Data Share*.

1. Seleccione **Data Share** y **Crear**.

1. Proporcione los detalles básicos del recurso de Azure Data Share: 

     **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | Subscription | Su suscripción | Seleccione una suscripción de Azure para la cuenta de recursos compartidos de datos.|
    | Grupo de recursos | *test-resource-group* | Use un grupo de recursos existente o cree uno. |
    | Location | *Este de EE. UU. 2* | Seleccione una región para la cuenta de Azure Data Share.
    | Nombre | *datashareaccount* | Asigne un nombre a la cuenta del recurso compartido de datos. |
    | | |

1. Seleccione **Revisar + crear** >  y, después, **Crear** para aprovisionar la cuenta del recurso compartido de datos. El aprovisionamiento de una nueva cuenta de recurso compartido de datos tarda aproximadamente 2 minutos. 

1. Cuando finalice la implementación, seleccione **Go to resource** (Ir al recurso).

### <a name="create-a-share"></a>Creación de un recurso compartido

1. Vaya a la página **Información general** del recurso compartido de datos.

   :::image type="content" source="./media/share-receive-data.png" alt-text="Captura de pantalla que muestra la información general del recurso compartido de datos.":::

1. Seleccione **Start sharing your data** (Empezar a compartir datos).

1. Seleccione **Crear**.   

1. Proporcione los detalles del recurso compartido. Especifique un nombre, un tipo de recurso compartido, una descripción del contenido de los recursos compartidos y las condiciones de uso (opcional). 

    ![Captura de pantalla que muestra los detalles del recurso compartido de datos.](./media/enter-share-details.png "Escriba los detalles del recurso compartido de datos.") 

1. Seleccione **Continuar**.

1. Para agregar conjuntos de datos al recurso compartido, seleccione **Agregar conjuntos de datos**. 

    ![Captura de pantalla que muestra cómo agregar conjuntos de datos al recurso compartido.](./media/datasets.png "Conjuntos de datos.")

1. Seleccione un tipo de conjunto de datos para agregar. La lista de tipos de conjuntos de datos depende de si seleccionó el uso compartido basado en instantáneas o el uso compartido en contexto del paso anterior. 

    ![Captura de pantalla que muestra dónde seleccionar un tipo de conjunto de datos.](./media/add-datasets.png "Adición de conjuntos de datos.")    

1. Vaya al objeto que quiere compartir. Luego, seleccione **Agregar conjuntos de datos**. 

    ![Captura de pantalla que muestra cómo seleccionar un objeto para compartir.](./media/select-datasets.png "Seleccionar conjuntos de datos.")    

1. En la pestaña **Destinatarios**, escriba las direcciones de correo electrónico del consumidor de datos, para lo que debe seleccionar **Agregar destinatario**. 

    ![Captura de pantalla que muestra cómo agregar direcciones de correo electrónico de destinatarios.](./media/add-recipient.png "Adición de destinatarios.") 

1. Seleccione **Continuar**.

1. Si seleccionó un tipo de recurso compartido de instantánea, puede configurar la programación de instantáneas para actualizar los datos del consumidor de datos. 

    ![Captura de pantalla que muestra la configuración de la programación de instantáneas.](./media/enable-snapshots.png "Habilitación de instantáneas.") 

1. Seleccione una hora de inicio y un intervalo de periodicidad. 

1. Seleccione **Continuar**.

1. En la pestaña **Revisar y crear**, examine el contenido del paquete, la configuración, los destinatarios y los valores de sincronización. Después, seleccione **Crear**.

Ahora ha creado el recurso compartido de datos de Azure. El destinatario del recurso compartido de datos puede aceptar su invitación. 

## <a name="receive-data"></a>Recibir datos

En las secciones siguientes se describe cómo recibir datos compartidos.
### <a name="prerequisites-to-receive-data"></a>Requisitos previos para recibir datos
Antes de aceptar una invitación a un recurso compartido de datos, asegúrese de que cumple los siguientes requisitos previos: 

* Suscripción a Azure. Si no tiene una suscripción, cree una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una invitación de Azure. El asunto del correo electrónico debe ser "Invitación a Azure Data Share de *\<yourdataprovider\@domain.com>* ".
* Un [proveedor Microsoft.DataShare](concepts-roles-permissions.md#resource-provider-registration) registrado en:
    * La suscripción a Azure en la que creará un recurso de Data Share.
    * La suscripción a Azure donde se encuentran los almacenes de datos de Azure de destino.

### <a name="prerequisites-for-a-target-storage-account"></a>Requisitos previos de una cuenta de almacenamiento de destino

* Una cuenta de Azure Storage. Si aún no la tiene, [cree una cuenta](../storage/common/storage-account-create.md). 
* Permiso para escribir en la cuenta de almacenamiento. Este permiso se encuentra en *Microsoft.Storage/storageAccounts/write*. Forma parte del rol de colaborador. 
* Permiso para agregar asignación de roles a la cuenta de almacenamiento. Esta asignación está en *Microsoft.Authorization/role assignments/write*. Forma parte del rol de propietario.  

### <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

### <a name="open-an-invitation"></a>Apertura de una invitación

Puede abrir una invitación desde el correo electrónico o directamente en Azure Portal.

1. Para abrir una invitación desde el correo electrónico, compruebe la bandeja de entrada para ver si hay una invitación de su proveedor de datos. La invitación de Microsoft Azure se titula *Azure Data Share invitation from \<yourdataprovider\@domain.com>* (Invitación para Azure Data Share de yourdataprovider@domain.com). Seleccione **View invitation** (Ver invitación) para ver su invitación en Azure. 

   Para abrir una invitación desde Azure Portal, busque *Data Share invitations* (Invitaciones de Data Share). Verá una lista de invitaciones de Data Share.

   ![Captura de pantalla que muestra la lista de invitaciones en Azure Portal.](./media/invitations.png "Lista de invitaciones.") 

1. Seleccione el recurso compartido que quiera ver. 

### <a name="accept-an-invitation"></a>Aceptación de una invitación
1. Revise todos los campos, incluidos los **Términos de uso**. Si acepta los términos, active la casilla. 

   ![Captura de pantalla que muestra el área Términos de uso.](./media/terms-of-use.png "Condiciones de uso.") 

1. En **Target Data Share Account** (Cuenta de Azure Data Share de destino), seleccione la suscripción y el grupo de recursos en los que va a implementar su instancia de Data Share. Luego, rellene los campos siguientes:

   * En el campo **Data Share Account** (Cuenta de Azure Data Share), seleccione **Crear nuevo** si no tiene una cuenta de Data Share. En caso contrario, seleccione la cuenta de Data Share existente que aceptará los recursos compartido de datos. 

   * En el campo **Received Share Name** (Nombre de recurso compartido recibido), deje el valor predeterminado que ha especificado el proveedor de datos o especifique un nombre nuevo. 

1. Seleccione **Accept and configure later** (Aceptar y configurar más adelante). Se crea una suscripción al recurso compartido. 

   ![Captura de pantalla que muestra dónde aceptar las opciones de configuración.](./media/accept-options.png "Opciones de aceptación") 

    El recurso compartido recibido aparece en la cuenta de Data Share. 

    Si no quiere aceptar la invitación, seleccione **Rechazar**. 

### <a name="configure-a-received-share"></a>Configuración de un recurso compartido recibido
Para configurar la ubicación para recibir datos, siga los pasos de esta sección.

1. En la pestaña **Conjuntos de datos**, active la casilla situada junto al conjunto de datos al que quiere asignar un destino. Seleccione **Asignar a destino** para elegir un almacén de datos de destino. 

   ![Captura de pantalla que muestra cómo realizar la asignación a un destino.](./media/dataset-map-target.png "Asignación a un destino.") 

1. Seleccione un almacén de datos de destino para los datos. Se sobrescribirán los archivos del almacén de datos de destino que tengan la misma ruta de acceso y el mismo nombre que los archivos de los datos recibidos. 

   ![Captura de pantalla que muestra dónde seleccionar una cuenta de almacenamiento de destino.](./media/map-target.png "Almacenamiento de destino.") 

1. En el caso del uso compartido basado en instantáneas, si el proveedor de datos usa una programación de instantáneas para actualizar periódicamente los datos, puede habilitar la programación en la pestaña **Programación de instantáneas**. Active la casilla situada junto a la programación de instantáneas. Seleccione **Habilitar**. Tenga en cuenta que la primera instantánea programada se iniciará en un minuto desde el tiempo de programación y las instantáneas posteriores comenzarán en cuestión de segundos desde la hora programada.

   ![Captura de pantalla que muestra cómo habilitar una programación de instantáneas.](./media/enable-snapshot-schedule.png "Habilitación de la programación de instantáneas.")

### <a name="trigger-a-snapshot"></a>Desencadenamiento de una instantánea
Los pasos de esta sección solo se aplican al uso compartido basado en instantáneas.

1. Puede desencadenar una instantánea desde la pestaña **Detalles**. En la pestaña, seleccione **Desencadenar instantánea**. Puede optar por desencadenar una instantánea completa o una instantánea incremental de los datos. Si recibe datos del proveedor de datos por primera vez, seleccione **Copia completa**. Cuando se ejecuta una instantánea, las posteriores no se iniciarán hasta que esta se complete.

   ![Captura de pantalla que muestra la selección de Desencadenar instantánea.](./media/trigger-snapshot.png "Desencadenamiento de instantáneas.") 

1. Cuando el estado de la última ejecución sea *correcto*, vaya al almacén de datos de destino para ver los datos recibidos. Seleccione **Conjuntos de datos** y, luego, seleccione el vínculo de la ruta de acceso de destino. 

   ![Captura de pantalla que muestra una asignación de conjunto de datos de consumidor.](./media/consumer-datasets.png "Asignación del conjunto de datos de consumidor.") 

### <a name="view-history"></a>Visualización del historial
Puede ver el historial de las instantáneas solo del uso compartido basado en instantáneas. Para ello, abra la pestaña **Historial**. Aquí verá el historial de todas las instantáneas que se generaron en los últimos 30 días. 

## <a name="storage-snapshot-performance"></a>Rendimiento de la instantánea de almacenamiento
El rendimiento de la instantánea de almacenamiento se ve afectado por una serie de factores, además de por el número de archivos y el tamaño de los datos compartidos. Siempre se recomienda llevar a cabo sus propias pruebas de rendimiento. A continuación, se muestran algunos factores de ejemplo que afectan al rendimiento.

* Acceso simultáneo a los almacenes de datos de origen y de destino.  
* Ubicación de los almacenes de datos de origen y de destino. 
* En el caso de una instantánea incremental, el número de archivos del conjunto de datos compartido puede afectar al tiempo que se tarda en buscar la lista de archivos con la última hora de modificación después de la última instantánea correcta. 


## <a name="next-steps"></a>Pasos siguientes
Ha aprendido a compartir y recibir datos de una cuenta de almacenamiento mediante el servicio Azure Data Share. Para aprender a compartir desde otros orígenes de datos, consulte [Almacenes de datos admitidos](supported-data-stores.md).