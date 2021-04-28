---
title: 'Tutorial: Accesos de aplicaciones web a Storage mediante identidades administradas | Azure'
description: En este tutorial aprenderá a establecer el acceso a Azure Storage para una aplicación mediante identidades administradas.
services: storage, app-service-web
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 11/30/2020
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: a37b189ae98332b2d6c557b6bdfad98266002e9e
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107833911"
---
# <a name="tutorial-access-azure-storage-from-a-web-app"></a>Tutorial: Acceso a Azure Storage desde una aplicación web

Aprenda a establecer el acceso a Azure Storage para una aplicación web (no para un usuario con la sesión iniciada) que se ejecuta en Azure App Service mediante identidades administradas.

:::image type="content" alt-text="Diagrama que muestra cómo obtener acceso a Storage." source="./media/scenario-secure-app-access-storage/web-app-access-storage.svg" border="false":::

Puede que desee agregar acceso al plano de datos de Azure (Azure Storage, Azure SQL Database, Azure Key Vault u otros servicios) desde la aplicación web. Podría usar una clave compartida, pero tendría que preocuparse de la seguridad operativa de quién puede crear, implementar y administrar el secreto. También es posible que la clave pudiera insertarse en GitHub, donde los hackers saben cómo buscar. Una manera más segura de dar acceso a los datos a la aplicación web es usar [identidades administradas](../active-directory/managed-identities-azure-resources/overview.md).

Una identidad administrada de Azure Active Directory (Azure AD) permite a App Service acceder a los recursos a través del control de acceso basado en roles, sin necesidad de credenciales de aplicación. Después de asignar una identidad administrada a la aplicación web, Azure se encarga de la creación y distribución de un certificado. Los usuarios no tienen que preocuparse de administrar secretos ni credenciales de aplicaciones.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear una identidad administrada asignada por el sistema en una aplicación web.
> * Crear una cuenta de almacenamiento y un contenedor de Azure Blob Storage.
> * Acceder a Storage desde una aplicación web mediante identidades administradas.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

* Una aplicación web que se ejecuta en Azure App Service que tiene el [módulo de autenticación y autorización de App Service habilitado](scenario-secure-app-authentication-app-service.md).

## <a name="enable-managed-identity-on-an-app"></a>Habilitación de la identidad administrada en una aplicación

Si crea y publica la aplicación web mediante Visual Studio, habilitará la identidad administrada en la aplicación. En la instancia de App Service, seleccione **Identidad** en el panel izquierdo y, a continuación, **Asignado por el sistema**. Compruebe que el **Estado** está establecido en **Activo**. Si no es así, seleccione **Guardar** y, a continuación, **Sí** para habilitar la identidad administrada asignada por el sistema. Una vez habilitada la identidad administrada, el estado se establece en **Activo** y el identificador del objeto está disponible.

:::image type="content" alt-text="Captura de pantalla que muestra la opción de identidad asignada por el sistema." source="./media/scenario-secure-app-access-storage/create-system-assigned-identity.png":::

Este paso crea un nuevo identificador de objeto, diferente del identificador de la aplicación que se creó en el panel **Autenticación o autorización**. Copie el identificador de objeto de la identidad administrada asignada por el sistema. Lo necesitará más adelante.

## <a name="create-a-storage-account-and-blob-storage-container"></a>Creación de una cuenta de almacenamiento y un contenedor de Blob Storage

Ahora ya está listo para crear una cuenta de almacenamiento y un contenedor de Blob Storage.

Cada cuenta de almacenamiento debe pertenecer a un grupo de recursos de Azure. Un grupo de recursos es un contenedor lógico para agrupar servicios de Azure. Al crear una cuenta de almacenamiento, puede elegir entre crear un grupo de recursos o usar uno existente. En este artículo se muestra cómo crear un nuevo grupo de recursos.

Una cuenta de almacenamiento de uso general v2 proporciona acceso a todos los servicios de Azure Storage: Blob, File, Queue, Table y Disk. Los pasos indicados aquí crean una cuenta de almacenamiento de uso general v2, pero los pasos para crear cualquier otro tipo de cuenta de almacenamiento son similares.

Los blobs de Azure Storage se organizan en contenedores. Antes de poder cargar un blob más adelante en este tutorial, primero debe crear un contenedor.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para crear una cuenta de almacenamiento de uso general v2 en Azure Portal, siga estos pasos.

1. En el menú de Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba **Cuentas de almacenamiento**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Cuentas de almacenamiento**.

1. En la ventana **Cuentas de almacenamiento** que aparece, seleccione **Agregar**.

1. Seleccione la suscripción en la que se va a crear la cuenta de almacenamiento.

1. En el campo **Grupo de recursos**, seleccione en el menú desplegable el grupo de recursos que contiene la aplicación web.

1. Después, escriba un nombre para la cuenta de almacenamiento. El nombre que elija debe ser único en Azure. El nombre debe tener también una longitud de entre 3 y 24 caracteres y solo puede contener números y letras minúsculas.

1. Seleccione una ubicación para la cuenta de almacenamiento o utilice la ubicación predeterminada.

1. Deje estos campos con sus valores predeterminados:

    |Campo|Value|
    |--|--|
    |Modelo de implementación|Resource Manager|
    |Rendimiento|Estándar|
    |Tipo de cuenta|StorageV2 (uso general v2)|
    |Replicación|Almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS)|
    |Nivel de acceso|Acceso frecuente|

1. Seleccione **Revisar y crear** para revisar la configuración de la cuenta de almacenamiento y crear la cuenta.

1. Seleccione **Crear**.

Para crear un contenedor de Blob Storage en Azure Storage, siga estos pasos.

1. Vaya a la nueva cuenta de almacenamiento en Azure Portal.

1. En el menú izquierdo de la cuenta de almacenamiento, desplácese a la sección **Blob service** y seleccione **Contenedores**.

1. Seleccione el botón **+ Contenedor**.

1. Escriba un nombre para el nuevo contenedor. El nombre del contenedor debe escribirse en minúsculas, comenzar por una letra o un número, y solo puede incluir letras, números y el carácter de guión (-).

1. Establezca el nivel de acceso público al contenedor. El nivel predeterminado es **Private (no anonymous access)** [Privado (sin acceso anónimo)].

1. Seleccione **Aceptar** para crear el contenedor.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para crear una cuenta de almacenamiento de uso general v2 y un contenedor de Blob Storage, ejecute el siguiente script. Especifique el nombre del grupo de recursos que contiene la aplicación web. Escriba un nombre para la cuenta de almacenamiento. El nombre que elija debe ser único en Azure. El nombre debe tener también una longitud de entre 3 y 24 caracteres y solo puede contener números y letras minúsculas.

Especifique la ubicación de la cuenta de almacenamiento. Para ver una lista de ubicaciones válidas para la suscripción, ejecute ```Get-AzLocation | select Location```. El nombre del contenedor debe escribirse en minúsculas, comenzar por una letra o un número, y solo puede incluir letras, números y el carácter de guión (-).

No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores.

```powershell
Connect-AzAccount

$resourceGroup = "securewebappresourcegroup"
$location = "<location>"
$storageName="securewebappstorage"
$containerName = "securewebappblobcontainer"

$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageName `
  -Location $location `
  -SkuName Standard_RAGRS `
  -Kind StorageV2

$ctx = $storageAccount.Context

New-AzStorageContainer -Name $containerName -Context $ctx -Permission blob
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para crear una cuenta de almacenamiento de uso general v2 y un contenedor de Blob Storage, ejecute el siguiente script. Especifique el nombre del grupo de recursos que contiene la aplicación web. Escriba un nombre para la cuenta de almacenamiento. El nombre que elija debe ser único en Azure. El nombre debe tener también una longitud de entre 3 y 24 caracteres y solo puede contener números y letras minúsculas. 

Especifique la ubicación de la cuenta de almacenamiento. El nombre del contenedor debe escribirse en minúsculas, comenzar por una letra o un número, y solo puede incluir letras, números y el carácter de guión (-).

En el ejemplo siguiente se usa la cuenta de Azure AD para autorizar la operación de creación del contenedor. Antes de crear el contenedor, asígnese a sí mismo el rol Colaborador de datos de Storage Blob. Aunque sea el propietario de la cuenta, necesita permisos explícitos para realizar operaciones de datos en la cuenta de almacenamiento.

No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores.

```azurecli-interactive
az login

az storage account create \
    --name securewebappstorage \
    --resource-group securewebappresourcegroup \
    --location <location> \
    --sku Standard_ZRS \
    --encryption-services blob

storageId=$(az storage account show -n securewebappstorage -g securewebappresourcegroup --query id --out tsv)

az ad signed-in-user show --query objectId -o tsv | az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee @- \
    --scope $storageId

az storage container create \
    --account-name securewebappstorage \
    --name securewebappblobcontainer \
    --auth-mode login
```

---

## <a name="grant-access-to-the-storage-account"></a>Concesión de acceso a la cuenta de almacenamiento

Debe conceder a la aplicación web acceso a la cuenta de almacenamiento antes de poder crear, leer o eliminar blobs. En un paso anterior, configuró la aplicación web que se ejecuta en App Service con una identidad administrada. Con el control de acceso basado en roles de Azure puede conceder a la identidad administrada acceso a otro recurso, igual que cualquier entidad de seguridad. El rol Colaborador de datos de Storage Blob proporciona a la aplicación web (representada por la identidad administrada asignada por el sistema) acceso de lectura, escritura y eliminación al contenedor de blobs y a los datos.

# <a name="portal"></a>[Portal](#tab/azure-portal)

En [Azure Portal](https://portal.azure.com), vaya a la cuenta de almacenamiento para conceder acceso a la aplicación web. Seleccione **Control de acceso (IAM)** en el panel izquierdo y, después, **Asignaciones de roles**. Verá una lista de quién tiene acceso a la cuenta de almacenamiento. Ahora quiere agregar una asignación de roles a un robot, la instancia de App Service que necesita acceso a la cuenta de almacenamiento. Seleccione **Agregar** > **Agregar asignación de roles**.

En **Rol**, seleccione **Colaborador de datos de Storage Blob** para dar a la aplicación web acceso para leer blobs de almacenamiento. En **Asignar acceso a**, seleccione **App Service**. En **Suscripción**, seleccione su suscripción. Seleccione la instancia de App Service a la que desea proporcionar acceso. Seleccione **Guardar**.

:::image type="content" alt-text="Captura de pantalla que muestra la pantalla Agregar asignación de roles." source="./media/scenario-secure-app-access-storage/add-role-assignment.png":::

Ahora, la aplicación web ya tiene acceso a la cuenta de almacenamiento.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Ejecute el siguiente script para asignar a la aplicación web (representada por una identidad administrada asignada por el sistema) el rol Colaborador de datos de Storage Blob en su cuenta de almacenamiento.

```powershell
$resourceGroup = "securewebappresourcegroup"
$webAppName="SecureWebApp20201102125811"
$storageName="securewebappstorage"

$spID = (Get-AzWebApp -ResourceGroupName $resourceGroup -Name $webAppName).identity.principalid
$storageId= (Get-AzStorageAccount -ResourceGroupName $resourceGroup -Name $storageName).Id
New-AzRoleAssignment -ObjectId $spID -RoleDefinitionName "Storage Blob Data Contributor" -Scope $storageId
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Ejecute el siguiente script para asignar a la aplicación web (representada por una identidad administrada asignada por el sistema) el rol Colaborador de datos de Storage Blob en su cuenta de almacenamiento.

```azurecli-interactive
spID=$(az resource list -n SecureWebApp20201102125811 --query [*].identity.principalId --out tsv)

storageId=$(az storage account show -n securewebappstorage -g securewebappresourcegroup --query id --out tsv)

az role assignment create --assignee $spID --role 'Storage Blob Data Contributor' --scope $storageId
```

---

## <a name="access-blob-storage-net"></a>Acceso a Blob Storage (.NET)

Para obtener una credencial de token que el código pueda usar para autorizar solicitudes para Azure Storage, se utiliza la clase [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential). Cree una instancia de la clase [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential), que usa la identidad administrada para capturar los tokens y asociarlos al cliente del servicio. En el ejemplo de código siguiente se obtiene la credencial de token autenticada y se usa para crear un objeto de cliente del servicio que, luego, carga un nuevo blob.

Para ver este código como parte de una aplicación de ejemplo, consulte el [ejemplo en GitHub](https://github.com/Azure-Samples/ms-identity-easyauth-dotnet-storage-graphapi/tree/main/1-WebApp-storage-managed-identity).

### <a name="install-client-library-packages"></a>Instalación de los paquetes de biblioteca cliente

Instale el [paquete NuGet de Blob Storage](https://www.nuget.org/packages/Azure.Storage.Blobs/) para que funcione con este servicio y la [biblioteca cliente de Azure Identity del paquete NuGet de .NET](https://www.nuget.org/packages/Azure.Identity/) para autenticarse con las credenciales de Azure AD. Instale las bibliotecas cliente mediante la interfaz de la línea de comandos de .NET Core o la Consola del Administrador de paquetes en Visual Studio.

# <a name="command-line"></a>[Línea de comandos](#tab/command-line)

Abra una línea de comandos y cambie al directorio que contiene el archivo del proyecto.

Ejecute los comandos de instalación.

```dotnetcli
dotnet add package Azure.Storage.Blobs

dotnet add package Azure.Identity
```

# <a name="package-manager"></a>[Administrador de paquetes](#tab/package-manager)

Abra el proyecto o la solución en Visual Studio y abra la consola mediante **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes**.

Ejecute los comandos de instalación.
```powershell
Install-Package Azure.Storage.Blobs

Install-Package Azure.Identity
```

---

### <a name="example"></a>Ejemplo

```csharp
using System;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Text;
using System.IO;
using Azure.Identity;

// Some code omitted for brevity.

static public async Task UploadBlob(string accountName, string containerName, string blobName, string blobContents)
{
    // Construct the blob container endpoint from the arguments.
    string containerEndpoint = string.Format("https://{0}.blob.core.windows.net/{1}",
                                                accountName,
                                                containerName);

    // Get a credential and create a client object for the blob container.
    BlobContainerClient containerClient = new BlobContainerClient(new Uri(containerEndpoint),
                                                                    new DefaultAzureCredential());

    try
    {
        // Create the container if it does not exist.
        await containerClient.CreateIfNotExistsAsync();

        // Upload text to a new block blob.
        byte[] byteArray = Encoding.ASCII.GetBytes(blobContents);

        using (MemoryStream stream = new MemoryStream(byteArray))
        {
            await containerClient.UploadBlobAsync(blobName, stream);
        }
    }
    catch (Exception e)
    {
        throw e;
    }
}
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya ha terminado con este tutorial y no necesita la aplicación web ni los recursos asociados, [elimine los recursos que creó](scenario-secure-app-clean-up-resources.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a:

> [!div class="checklist"]
>
> * Crear una identidad administrada asignada por el sistema.
> * Crear una cuenta de almacenamiento y un contenedor de Blob Storage.
> * Acceder a Storage desde una aplicación web mediante identidades administradas.

> [!div class="nextstepaction"]
> [Accesos de App Service a Microsoft Graph en nombre del usuario](scenario-secure-app-access-microsoft-graph-as-user.md)
