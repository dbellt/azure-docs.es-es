---
title: 'Inicio rápido: Biblioteca cliente de certificados de Azure Key Vault para .NET (versión 4)'
description: Aprenda a crear, recuperar y eliminar certificados de una instancia de Azure Key Vault mediante la biblioteca cliente de .NET (versión 4)
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/23/2020
ms.service: key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: 9188079a93c9c2688a310413401e90014b278bf8
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107813423"
---
# <a name="quickstart-azure-key-vault-certificate-client-library-for-net-sdk-v4"></a>Inicio rápido: Biblioteca cliente de certificados de Azure Key Vault para .NET (SDK v4)

Empiece a trabajar con la biblioteca cliente de certificados de Azure Key Vault para .NET. [Azure Key Vault](../general/overview.md) es un servicio en la nube que proporciona un almacén seguro para los certificados. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Las instancias de Azure Key Vault se pueden crear y administrar a través de Azure Portal. En este inicio rápido, aprenderá a crear, recuperar y eliminar certificados de una instancia de Azure Key Vault mediante la biblioteca cliente de .NET.

Recursos de la biblioteca cliente de Key Vault:

[Documentación de referencia de la API](/dotnet/api/azure.security.keyvault.certificates) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault) | [Paquete (NuGet)](https://www.nuget.org/packages/Azure.Security.KeyVault.Certificates/)

Para más información sobre Key Vault y los certificados, consulte:
- [Introducción a Azure Key Vault](../general/overview.md)
- [Introducción a los certificados](about-certificates.md).

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/dotnet).
* [SDK de .NET Core 3.1 o una versión posterior](https://dotnet.microsoft.com/download/dotnet-core)
* [CLI de Azure](/cli/azure/install-azure-cli)
* Un almacén de claves: puede crear uno mediante [Azure Portal](../general/quick-create-portal.md), la [CLI de Azure](../general/quick-create-cli.md) o [Azure PowerShell](../general/quick-create-powershell.md).

En este inicio rápido se usa `dotnet` y la CLI de Azure.

## <a name="setup"></a>Configurar

En este inicio rápido se usa la biblioteca de identidades de Azure con la CLI de Azure para autenticar al usuario en los servicios de Azure. Los desarrolladores también pueden usar Visual Studio o Visual Studio Code para autenticar sus llamadas. Para más información, consulte [Autenticación del cliente mediante la biblioteca cliente Azure Identity](/dotnet/api/overview/azure/identity-readme?#authenticate-the-client&preserve-view=true).

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

1. Ejecute el comando `login`.

    ```azurecli-interactive
    az login
    ```

    Si la CLI puede abrir el explorador predeterminado, lo hará y cargará una página de inicio de sesión de Azure.

    En caso contrario, abra una página del explorador en [https://aka.ms/devicelogin](https://aka.ms/devicelogin) y escriba el código de autorización que se muestra en el terminal.

2. Inicie sesión con las credenciales de su cuenta en el explorador.

#### <a name="grant-access-to-your-key-vault"></a>Concesión de acceso al almacén de claves

Cree una directiva de acceso para el almacén de claves que conceda permisos de certificado a la cuenta de su usuario

```console
az keyvault set-policy --name <your-key-vault-name> --upn user@domain.com --certificate-permissions delete get list create purge
```

### <a name="create-new-net-console-app"></a>Creación de una aplicación de consola de .NET

1. En un shell de comandos, ejecute el siguiente comando para crear un proyecto llamado `key-vault-console-app`:

    ```dotnetcli
    dotnet new console --name key-vault-console-app
    ```

1. Cambie al directorio *key-vault-console-app* recién creado y ejecute el comando siguiente para compilar el proyecto:

    ```dotnetcli
    dotnet build
    ```

    La salida de la compilación no debe contener advertencias ni errores.
    
    ```console
    Build succeeded.
     0 Warning(s)
     0 Error(s)
    ```

### <a name="install-the-packages"></a>Instalación de los paquetes

En el shell de comandos, instale la biblioteca cliente de certificados de Azure Key Vault para .NET:

```dotnetcli
dotnet add package Azure.Security.KeyVault.Certificates
```

Para este inicio rápido también deberá instalar la biblioteca cliente del SDK de Azure SDK para Azure Identity:

```dotnetcli
dotnet add package Azure.Identity
```

#### <a name="set-environment-variables"></a>Establecimiento de variables de entorno

Esta aplicación también usa el nombre del almacén de claves como variable de entorno llamada `KEY_VAULT_NAME`.

Windows
```cmd
set KEY_VAULT_NAME=<your-key-vault-name>
````
Windows PowerShell
```powershell
$Env:KEY_VAULT_NAME="<your-key-vault-name>"
```

macOS o Linux
```bash
export KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="object-model"></a>Modelo de objetos

La biblioteca cliente de certificados de Azure Key Vault para .NET permite administrar los certificados. En la sección [Ejemplos de código](#code-examples) se muestra cómo crear un cliente y establecer, recuperar y eliminar un certificado.

## <a name="code-examples"></a>Ejemplos de código

### <a name="add-directives"></a>Adición de directivas

Agregue las directivas siguientes al principio de *Program.cs*:

```csharp
using System;
using Azure.Identity;
using Azure.Security.KeyVault.Certificates;
```

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente

En este inicio rápido se emplea el usuario que ha iniciado sesión para autenticarlo en el almacén de claves, que es el método preferido para el desarrollo local. Para las aplicaciones implementadas en Azure, la identidad administrada debe asignarse a App Service o la máquina virtual. Para más información, consulte [Introducción a la identidad administrada](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview).

En el ejemplo siguiente, el nombre del almacén de claves se expande al URI del almacén de claves, con el formato "https://\<your-key-vault-name\>.vault.azure.net". En este ejemplo se usa la clase ["DefaultAzureCredential()"](/dotnet/api/azure.identity.defaultazurecredential) de la [biblioteca de identidades de Azure](https://docs.microsoft.com/dotnet/api/overview/azure/identity-readme), que permite usar el mismo código en entornos diferentes con distintas opciones para proporcionar la identidad. Para más información sobre la autenticación en el almacén de claves, consulte la [Guía del desarrollador](https://docs.microsoft.com/azure/key-vault/general/developers-guide#authenticate-to-key-vault-in-code).

```csharp
string keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
var kvUri = "https://" + keyVaultName + ".vault.azure.net";

var client = new CertificateClient(new Uri(kvUri), new DefaultAzureCredential());
```

### <a name="save-a-certificate"></a>Guardar certificado

En este ejemplo, para simplificar, puede usar un certificado autofirmado con la directiva de emisión predeterminada. Para esta tarea, use el método [StartCreateCertificateAsync](/dotnet/api/azure.security.keyvault.certificates.certificateclient.startcreatecertificateasync). Los parámetros del método aceptan un nombre de certificado y la [directiva de certificados](https://docs.microsoft.com/dotnet/api/azure.security.keyvault.certificates.certificatepolicy).

```csharp
var operation = await client.StartCreateCertificateAsync("myCertificate", CertificatePolicy.Default);
var certificate = await operation.WaitForCompletionAsync();
```

> [!NOTE]
> Si el nombre del certificado existe, el código anterior creará una versión de ese certificado.

### <a name="retrieve-a-certificate"></a>Recuperación de un certificado

Ahora puede recuperar el certificado creado anteriormente con el método [GetCertificateAsync](/dotnet/api/azure.security.keyvault.certificates.certificateclient.getcertificateasync).

```csharp
var certificate = await client.GetCertificateAsync("myCertificate");
```

### <a name="delete-a-certificate"></a>Eliminación de un certificado

Por último, vamos a eliminar y purgar el certificado desde el almacén de claves con los métodos [StartDeleteCertificateAsync](/dotnet/api/azure.security.keyvault.certificates.certificateclient.startdeletecertificateasync) y [PurgeDeletedCertificateAsync](/dotnet/api/azure.security.keyvault.certificates.certificateclient.purgedeletedcertificateasync).

```csharp
var operation = await client.StartDeleteCertificateAsync("myCertificate");

// You only need to wait for completion if you want to purge or recover the certificate.
await operation.WaitForCompletionAsync();

var certificate = operation.Value;
await client.PurgeDeletedCertificateAsync("myCertificate");
```

## <a name="sample-code"></a>Código de ejemplo

Modifique la aplicación de consola de .NET Core para interactuar con el almacén de claves; para ello, complete los pasos siguientes:

- Reemplace el código de *Program.cs* por el código siguiente:

    ```csharp
    using System;
    using System.Threading.Tasks;
    using Azure.Identity;
    using Azure.Security.KeyVault.Certificates;
    
    namespace key_vault_console_app
    {
        class Program
        {
            static async Task Main(string[] args)
            {
                const string certificateName = "myCertificate";
                var keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
                var kvUri = $"https://{keyVaultName}.vault.azure.net";
    
                var client = new CertificateClient(new Uri(kvUri), new DefaultAzureCredential());
    
                Console.Write($"Creating a certificate in {keyVaultName} called '{certificateName}' ...");
                CertificateOperation operation = await client.StartCreateCertificateAsync(certificateName, CertificatePolicy.Default);
                await operation.WaitForCompletionAsync();
                Console.WriteLine(" done.");
    
                Console.WriteLine($"Retrieving your certificate from {keyVaultName}.");
                var certificate = await client.GetCertificateAsync(certificateName);
                Console.WriteLine($"Your certificate version is '{certificate.Value.Properties.Version}'.");
    
                Console.Write($"Deleting your certificate from {keyVaultName} ...");
                DeleteCertificateOperation deleteOperation = await client.StartDeleteCertificateAsync(certificateName);
                // You only need to wait for completion if you want to purge or recover the certificate.
                await deleteOperation.WaitForCompletionAsync();
                Console.WriteLine(" done.");

                Console.Write($"Purging your certificate from {keyVaultName} ...");
                await client.PurgeDeletedCertificateAsync(certificateName);
                Console.WriteLine(" done.");
            }
        }
    }
    ```
### <a name="test-and-verify"></a>Prueba y comprobación

Ejecute el siguiente comando para compilar el proyecto.

```dotnetcli
dotnet build
```

Aparece una variación del resultado siguiente:

```console
Creating a certificate in mykeyvault called 'myCertificate' ... done.
Retrieving your certificate from mykeyvault.
Your certificate version is '8532359bced24e4bb2525f2d2050738a'.
Deleting your certificate from mykeyvault ... done
Purging your certificate from mykeyvault ... done
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de claves, ha almacenado un certificado y, posteriormente, lo ha recuperado. 

Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, consulte los artículos siguientes:

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Lea una [introducción a los certificados](about-certificates.md).
- Vea un [Tutorial sobre el acceso a Key Vault desde una aplicación de App Service](../general/tutorial-net-create-vault-azure-web-app.md).
- Consulte un [Tutorial sobre el acceso a Key Vault desde una máquina virtual](../general/tutorial-net-virtual-machine.md).
- Consulte la [guía del desarrollador de Azure Key Vault](../general/developers-guide.md).
- Consulte [Introducción a la seguridad de Azure Key Vault](../general/security-features.md)
