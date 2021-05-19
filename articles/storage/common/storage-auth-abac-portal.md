---
title: 'Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante Azure Portal (versión preliminar): Azure ABAC'
titleSuffix: Azure Storage
description: Agregue una condición de asignación de roles para restringir el acceso a los blobs mediante Azure Portal y el control de acceso basado en atributos de Azure (Azure ABAC).
services: storage
author: rolyon
ms.service: storage
ms.topic: tutorial
ms.author: rolyon
ms.reviewer: ''
ms.subservice: common
ms.date: 05/06/2021
ms.openlocfilehash: 2099d190ca896a5c8124cdd6b605037256a5ab48
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109489442"
---
# <a name="tutorial-add-a-role-assignment-condition-to-restrict-access-to-blobs-using-the-azure-portal-preview"></a>Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante Azure Portal (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y Azure ABAC se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En la mayoría de los casos, una asignación de roles concederá los permisos necesarios para los recursos de Azure. Sin embargo, en algunos casos es posible que desee proporcionar un control de acceso más preciso, para lo que puede agregar una condición de asignación de roles.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar una condición a una asignación de roles
> * Restringir el acceso a blobs en función de una etiqueta de índice de blobs

## <a name="prerequisites"></a>Requisitos previos

Para obtener información sobre los requisitos previos para agregar o editar condiciones de asignación de roles, consulte [Requisitos previos de las condiciones](../../role-based-access-control/conditions-prerequisites.md).

## <a name="condition"></a>Condición

En este tutorial, restringirá el acceso a blobs con una etiqueta específica. Por ejemplo, agregue una condición a una asignación de roles para que Chandra solo pueda leer archivos con la etiqueta Project=Cascade.

![Diagrama de asignación de roles con una condición.](./media/shared/condition-role-assignment-rg.png)

Si Chandra intenta leer un blob sin la etiqueta Project=Cascade, no se permite el acceso.

![Diagrama que muestra el acceso de lectura a blobs con la etiqueta Project=Cascade.](./media/shared/condition-access.png)

Este es el aspecto de la condición en el código:

```
(
    (
        !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'}
        AND
        @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})
    )
    OR
    (
        @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$key_case_sensitive$>] StringEqualsIgnoreCase 'Cascade'
    )
)
```

## <a name="step-1-create-a-user"></a>Paso 1: Creación de un usuario

1. Inicie sesión en Azure Portal como propietario de una suscripción.

1. Haga clic en **Azure Active Directory**.
    
1. Cree un usuario o busque uno existente. En este tutorial se usa Chandra como ejemplo.

## <a name="step-2-set-up-storage"></a>Paso 2: Configuración del almacenamiento

1. Si aún no lo ha hecho, registre la suscripción para usar etiquetas de índice de blobs. Para más información, consulte [Registro de la suscripción (versión preliminar)](../blobs/storage-manage-find-blobs.md#register-your-subscription-preview).

1. Cree una cuenta de almacenamiento que sea compatible con la característica de etiquetas de índice de blobs, que actualmente se encuentra en versión preliminar pública. Para más información, consulte [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](../blobs/storage-manage-find-blobs.md#regional-availability-and-storage-account-support).

1. Cree un contenedor de blobs dentro de la cuenta de almacenamiento y establezca el nivel de acceso Público en **Privado (sin acceso anónimo)** .

1. En el contenedor, haga clic **Cargar** para abrir el panel Cargar blob.

1. Busque el archivo de texto que desee cargar.

1. Haga clic en **Opciones avanzadas** para expandir el panel.

1. En la sección **Blob index tags** (Etiquetas de índice de blob), agregue la siguiente etiqueta de índice de blob al archivo de texto.

    Si no ve la sección y acaba de registrar la suscripción, es posible que tenga que esperar unos minutos hasta que los cambios se propaguen. Para más información, consulte [Uso de etiquetas de índice de blobs (versión preliminar) para administrar y buscar datos en Azure Blob Storage](../blobs/storage-blob-index-how-to.md).

    > [!NOTE]
    > Los blobs también admiten la capacidad de almacenar metadatos arbitrarios de clave-valor definidos por el usuario. Aunque los metadatos son similares a las etiquetas de índice de blobs, debe usar etiquetas de índice de blobs con condiciones. 

    | Clave | Value |
    | --- | --- |
    | Project  | Cascade |

   ![Captura de pantalla que muestra el panel Cargar blob con la sección Blog index tags (Etiquetas de índice de blog).](./media/storage-auth-abac-portal/container-upload-blob.png)

1. Haga clic en el botón **Cargar** para cargar el archivo.

1. Cargue un segundo archivo de texto.

1. Agregue la siguiente etiqueta de índice de blobs al segundo archivo de texto.

    | Clave | Value |
    | --- | --- |
    | Project  | Baker |

## <a name="step-3-assign-a-storage-blob-data-role"></a>Paso 3: Asignación de un rol de datos de blobs de almacenamiento

1. Abra el grupo de recursos.

1. Haga clic en **Control de acceso (IAM).**

1. Haga clic en la pestaña **Asignaciones de roles** para ver todas las asignaciones de roles en este ámbito.

1. Haga clic en **Agregar** > **Agregar asignación de roles (versión preliminar)** .

   ![Captura de pantalla de la versión preliminar del menú Agregar > Agregar asignación de roles.](./media/storage-auth-abac-portal/add-role-assignment-menu-preview.png)

    Se abre la página Agregar asignación de roles.

1. En la pestaña **Roles,** seleccione el rol [Lector de datos de Storage Blob](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader).

    ![Captura de pantalla de la página Agregar asignación de roles con la pestaña Roles.](./media/storage-auth-abac-portal/roles.png)

1. En la pestaña **Miembros,** seleccione el usuario que creó anteriormente.

   ![Captura de pantalla de la página Agregar asignación de roles con la pestaña Miembros.](./media/storage-auth-abac-portal/members.png)

1. (Opcional) En el cuadro **Descripción**, escriba **Acceso de lectura a blobs con la etiqueta Project=Cascade**.

1. Haga clic en **Next**.

## <a name="step-4-add-a-condition"></a>Paso 4: Incorporación de una condición

1. En la pestaña **Condición**, haga clic en **Agregar condición**.

    ![Captura de pantalla de la página de condiciones de Agregar asignación de roles para una nueva condición.](./media/storage-auth-abac-portal/condition-add-new.png)

    Aparece la página de condiciones de Agregar asignación de roles.

1. En la sección Agregar acción, haga clic en **Seleccionar acciones**.

    Aparece el panel Seleccionar una acción. Este panel es una lista filtrada de acciones de datos basada en la asignación de roles que será el destino de la condición. 

    ![Captura de pantalla del panel Seleccionar una acción con una acción seleccionada.](./media/storage-auth-abac-portal/condition-actions-select.png)

1. En Leer un blog, haga clic en **Read content from a blob with tag conditions** (Leer contenido de un blob con condiciones de etiqueta) y, después, haga clic en **Select** (Seleccionar).

1. En la sección Build expression (Generar expresión), haga clic en **Add expression** (Agregar expresión).

    Se expande la sección Expresión.

1. En la lista Attribute source (Origen del atributo), seleccione **Resource** (Recurso).

1. En la lista Atributo, seleccione **Blob index tags [Values in key]** (Etiquetas de índice de blobs [Valores en clave]).

    Al seleccionar este atributo se agrega un cuadro Clave, donde se puede especificar la clave de etiqueta que la condición debe comprobar.

1. En el cuadro Clave, escriba **Project**.

1. En la lista Operador, seleccione **StringEqualsIgnoreCase**.

1. En el cuadro Valor, escriba **Cascade**.

    ![Captura de pantalla de la sección Build expression (Generar expresión) para etiquetas de índice de blob.](./media/storage-auth-abac-portal/condition-expressions.png)

1. Desplácese hacia arriba hasta **Editor type** (Tipo de editor) y haga clic en **Code** (Código).

    La condición se muestra en forma de código. En este editor de código se pueden realizar cambios en la condición. Para volver al editor de objetos visuales, haga clic en **Objeto visual**.

    ![Captura de pantalla de la condición que se muestra en el editor de código.](./media/storage-auth-abac-portal/condition-code.png)

1. Haga clic en **Guardar** para agregar la condición y volver a la página Agregar asignación de roles.

1. Haga clic en **Next**.

1. En la pestaña **Revisar y asignar**, haga clic en **Revisar y asignar** para asignar el rol con una condición.

    Transcurridos unos instantes, se asigna el rol a la entidad de seguridad en el ámbito seleccionado.

    ![Captura de pantalla de la lista de asignación de roles después de asignar el rol.](./media/storage-auth-abac-portal/rg-role-assignments-condition.png)

## <a name="step-5-test-the-condition"></a>Paso 5: Prueba de la condición

Para probar la condición, deberá usar Azure PowerShell.

1. Abra una ventana de PowerShell.

1. Use [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) para iniciar sesión como Chandra.

    ```azurepowershell
    Connect-AzAccount
    ```

1. Inicialice las siguientes variables con los nombres que ha usado.

    ```azurepowershell
    $storageAccountName = "<storageAccountName>"
    $containerName = "<containerName>"
    $blobNameBaker = "<blobNameBaker>"
    $blobNameCascade = "<blobNameCascade>"
    ```

1. Use [New-AzStorageContext](/powershell/module/az.storage/new-azstoragecontext) para crear un contexto específico para acceder a la cuenta de almacenamiento más fácilmente.

    ```azurepowershell
    $bearerCtx = New-AzStorageContext -StorageAccountName $storageAccountName
    ```

1. Use [Get-AzStorageBlob](/powershell/module/az.storage/get-azstorageblob) para intentar leer el archivo del proyecto Baker.

    ```azurepowershell
    Get-AzStorageBlob -Container $containerName -Blob $blobNameBaker -Context $bearerCtx 
    ```

    Este es un ejemplo de la salida. Tenga en cuenta que **no puede** leer el archivo debido a la condición que ha agregado.
    
    ```azurepowershell
    Get-AzStorageBlob: This request is not authorized to perform this operation using this permission. HTTP Status Code: 403 - HTTP Error Message: This request is not authorized to perform this operation using this permission.
    ErrorCode: AuthorizationPermissionMismatch
    ErrorMessage: This request is not authorized to perform this operation using this permission.
    RequestId: <requestId>
    Time: Sun, 13 Sep 2020 12:33:42 GMT
    ```
    
1. Lea el archivo del proyecto Cascade.

    ```azurepowershell
    Get-AzStorageBlob -Container $containerName -Blob $blobNameCascade -Context $bearerCtx 
    ```

    Este es un ejemplo de la salida. Observe que sí puede leer el archivo porque tiene la etiqueta Project=Cascade.
    
    ```azurepowershell
       AccountName: <storageAccountName>, ContainerName: <containerName>
    
    Name                 BlobType  Length          ContentType                    LastModified         AccessTier SnapshotT
                                                                                                                  ime
    ----                 --------  ------          -----------                    ------------         ---------- ---------
    CascadeFile.txt      BlockBlob 7               text/plain                     2021-04-24 05:35:24Z Hot
    ```

## <a name="step-6-clean-up-resources"></a>Paso 6: Limpieza de recursos

1. Quite la asignación de roles que agregó.

1. Elimine la cuenta de almacenamiento de prueba que ha creado.

1. Elimine el usuario que ha creado.

## <a name="next-steps"></a>Pasos siguientes

- [Condiciones de asignación de roles de Azure de ejemplo](storage-auth-abac-examples.md)
- [Acciones y atributos para las condiciones de asignación de roles de Azure en Azure Storage (versión preliminar)](storage-auth-abac-attributes.md)
- [Sintaxis y formato de las condiciones de asignación de roles de Azure](../../role-based-access-control/conditions-format.md)
