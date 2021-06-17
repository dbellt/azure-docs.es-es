---
title: Autenticación basada en certificados de Azure Active Directory con Azure Cosmos DB
description: Aprenda a configurar una identidad de Azure AD para la autenticación basada en certificados de las claves de acceso desde Azure Cosmos DB.
author: voellm
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/11/2019
ms.author: tvoellm
ms.reviewer: sngun
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: ccbc63534e5aa52996716152c22d50379ca2cffa
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110681801"
---
# <a name="certificate-based-authentication-for-an-azure-ad-identity-to-access-keys-from-an-azure-cosmos-db-account"></a>Autenticación basada en certificados para una identidad de Azure AD para las claves de acceso desde una cuenta de Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

La autenticación basada en certificados permite que una aplicación cliente se autentique mediante Azure Active Directory (Azure AD) con un certificado de cliente. Puede realizar la autenticación basada en certificados en un equipo en el que necesite una identidad, como un equipo local o una máquina virtual en Azure. A partir de ese momento, la aplicación puede leer las claves de Azure Cosmos DB sin tenerlas directamente en la aplicación. En este artículo se describe cómo crear una aplicación de Azure AD de ejemplo, configurarla para la autenticación basada en certificados, iniciar sesión en Azure mediante la nueva identidad de aplicación y, posteriormente, recuperar las claves de una cuenta de Azure Cosmos. En este artículo se usa Azure PowerShell para configurar las identidades y se proporciona una aplicación en C# de ejemplo que autentica las claves y accede a ellas desde su cuenta de Azure Cosmos.  

## <a name="prerequisites"></a>Requisitos previos

* Instale la [versión más reciente](/powershell/azure/install-az-ps) de Azure PowerShell.

* Si no tiene una [suscripción a Azure](../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.

## <a name="register-an-app-in-azure-ad"></a>Registro de una aplicación en Azure AD

En este paso, va a registrar una aplicación web de ejemplo en su cuenta de Azure AD. Esta aplicación se utilizará más adelante para leer las claves de su cuenta de Azure Cosmos DB. Para registrar una aplicación, siga estos pasos: 

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/).

1. Abra el panel **Azure Active Directory**, vaya al panel **Registros de aplicaciones** y seleccione **Nuevo registro**. 

   :::image type="content" source="./media/certificate-based-authentication/new-app-registration.png" alt-text="Nuevo registro de aplicación en Active Directory":::

1. Rellene el formulario **Registrar una aplicación** con los detalles siguientes:  

   * **Name** (Nombre): especifique el nombre de la aplicación; puede ser cualquiera, como "sampleApp".
   * **Supported account types** (Tipos de cuenta admitidos): elija **Solo las cuentas de este directorio organizativo (directorio predeterminado)** para permitir que los recursos del directorio actual accedan a esta aplicación. 
   * **URL de redireccionamiento**: elija una aplicación del tipo **Web** y proporcione la dirección URL en que se hospeda la aplicación, puede ser cualquier dirección URL. En este ejemplo, puede proporcionar una dirección URL de prueba como `https://sampleApp.com`. No importa si la aplicación no existe.

   :::image type="content" source="./media/certificate-based-authentication/register-sample-web-app.png" alt-text="Registrar una aplicación web de ejemplo":::

1. Seleccione **Registrar** después de rellenar el formulario.

1. Después de registrar la aplicación, anote los valores de **Application (client) ID** [Id. de aplicación (cliente)] y **Object ID** (Id. de objeto), usará estos detalles en los pasos siguientes. 

   :::image type="content" source="./media/certificate-based-authentication/get-app-object-ids.png" alt-text="Obtener los identificadores de aplicación y de objeto":::

## <a name="install-the-azuread-module"></a>Instalación del módulo de Azure AD

En este paso instalará el módulo PowerShell de Azure AD. Este módulo se requiere para obtener el identificador de la aplicación que registró en el paso anterior y asociar un certificado autofirmado a dicha aplicación. 

1. Abra Windows PowerShell ISE con derechos de administrador. Si aún no lo ha hecho, instale el módulo de PowerShell AZ y conéctese a su suscripción. Si tiene varias suscripciones, puede establecer el contexto de la suscripción actual, como se muestra en los siguientes comandos:

   ```powershell
   Install-Module -Name Az -AllowClobber
   Connect-AzAccount

   Get-AzSubscription
   $context = Get-AzSubscription -SubscriptionId <Your_Subscription_ID>
   Set-AzContext $context 
   ```

1. Instale e importar el módulo [AzureAD](/powershell/module/azuread/)

   ```powershell
   Install-Module AzureAD
   Import-Module AzureAD 
   ```

## <a name="sign-into-your-azure-ad"></a>Inicio de sesión en Azure AD

Inicie sesión en Azure AD en el que ha registrado la aplicación. Use el comando Connect-AzureAD para iniciar sesión en su cuenta y escriba las credenciales de su cuenta de Azure en la ventana emergente. 

```powershell
Connect-AzureAD 
```

## <a name="create-a-self-signed-certificate"></a>Creación de un certificado autofirmado

Abra otra instancia de Windows PowerShell ISE y ejecute los comandos siguientes para crear un certificado autofirmado y leer la clave asociada con el certificado:

```powershell
$cert = New-SelfSignedCertificate -CertStoreLocation "Cert:\CurrentUser\My" -Subject "CN=sampleAppCert" -KeySpec KeyExchange
$keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData()) 
```

## <a name="create-the-certificate-based-credential"></a>Creación de la credencial basada en certificados 

A continuación, ejecute los comandos siguientes para obtener el identificador de objeto de la aplicación y crear la credencial basada en certificados. En este ejemplo, establecemos que el certificado expire después de un año, pero se puede establecer cualquier fecha de finalización.

```powershell
$application = Get-AzureADApplication -ObjectId <Object_ID_of_Your_Application>

New-AzureADApplicationKeyCredential -ObjectId $application.ObjectId -CustomKeyIdentifier "Key1" -Type AsymmetricX509Cert -Usage Verify -Value $keyValue -EndDate "2020-01-01"
```

El comando anterior da como resultado una salida similar a la de la captura de pantalla siguiente:

:::image type="content" source="./media/certificate-based-authentication/certificate-based-credential-output.png" alt-text="Salida de creación de credenciales basada en certificados":::

## <a name="configure-your-azure-cosmos-account-to-use-the-new-identity"></a>Configuración de una cuenta de Azure Cosmos para usar la nueva identidad

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/).

1. Vaya a la cuenta de Azure Cosmos y abra la hoja **Control de acceso (IAM)** .

1. Seleccione **Agregar** y **Agregar asignación de roles**. Agregar la aplicación sampleApp que creó en el paso anterior con el rol **Colaborador**, como se muestra en la captura de pantalla siguiente:

   :::image type="content" source="./media/certificate-based-authentication/configure-cosmos-account-with-identify.png" alt-text="Configuración de una cuenta de Azure Cosmos para usar la nueva identidad":::

1. Seleccione **Guardar** después de rellenar el formulario

## <a name="register-your-certificate-with-azure-ad"></a>Registro del certificado con Azure AD

Puede asociar la credencial basada en certificado a la aplicación cliente en Azure AD desde Azure Portal. Para asociar la credencial, debe cargar el archivo de certificado con los pasos siguientes:

En el registro de aplicación de Azure para la aplicación cliente:

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/).

1. Abra el panel **Azure Active Directory**, vaya al panel **Registros de aplicaciones** y abra la aplicación de ejemplo que creó en el paso anterior. 

1. Seleccione **Certificados y secretos** y, a continuación, seleccione **Cargar certificado**. Busque el archivo de certificado que creó en el paso anterior para cargarlo.

1. Seleccione **Agregar**. Una vez cargado el certificado, se muestran los valores de huella digital, fecha de inicio y expiración.

## <a name="access-the-keys-from-powershell"></a>Acceso a las claves desde PowerShell

En este paso, iniciará sesión en Azure mediante la aplicación y el certificado que ha creado y accederá a las claves de su cuenta de Azure Cosmos. 

1. Lo primero que debe hacer es borrar las credenciales de cuenta de Azure que ha usado para iniciar sesión en su cuenta. Para ello, use el siguiente comando:

   ```powershell
   Disconnect-AzAccount -Username <Your_Azure_account_email_id> 
   ```

1. Después, asegúrese de que puede iniciar sesión en Azure Portal mediante las credenciales de la aplicación y acceda a las claves de Azure Cosmos DB:

   ```powershell
   Login-AzAccount -ApplicationId <Your_Application_ID> -CertificateThumbprint $cert.Thumbprint -ServicePrincipal -Tenant <Tenant_ID_of_your_application>

   Get-AzCosmosDBAccountKey `
      -ResourceGroupName "<Resource_Group_Name_of_your_Azure_Cosmos_account>" `
      -Name "<Your_Azure_Cosmos_Account_Name>" `
      -Type "Keys"
   ```

El comando anterior muestra las claves principales primaria y secundaria de la cuenta de Azure Cosmos. Puede ver el registro de actividad de la cuenta de Azure Cosmos para validar que la solicitud de claves get se ha realizado correctamente y que la aplicación "sampleApp" inicia el evento.

:::image type="content" source="./media/certificate-based-authentication/activity-log-validate-results.png" alt-text="Validar la llamada a las claves get en Azure AD":::

## <a name="access-the-keys-from-a-c-application"></a>Acceso a las claves desde una aplicación en C# 

Este escenario también se puede validar mediante el acceso a las claves desde una aplicación creada en C#. La siguiente aplicación de consola de C#, que puede acceder a las claves de Azure Cosmos DB mediante la aplicación registrada en Active Directory. Asegúrese de actualizar los valores de tenantId, clientID, certName, el nombre del grupo de recursos, el identificador de la suscripción y el nombre de la cuenta de Azure Cosmos antes de ejecutar el código. 

```csharp
using System;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System.Linq;
using System.Net.Http;
using System.Security.Cryptography.X509Certificates;
using System.Threading;
using System.Threading.Tasks;
 
namespace TodoListDaemonWithCert
{
    class Program
    {
        private static string aadInstance = "https://login.windows.net/";
        private static string tenantId = "<Your_Tenant_ID>";
        private static string clientId = "<Your_Client_ID>";
        private static string certName = "<Your_Certificate_Name>";
 
        private static int errorCode = 0;
        static int Main(string[] args)
        {
            MainAync().Wait();
            Console.ReadKey();
 
            return 0;
        } 
 
        static async Task MainAync()
        {
            string authContextURL = aadInstance + tenantId;
            AuthenticationContext authContext = new AuthenticationContext(authContextURL);
            X509Certificate2 cert = ReadCertificateFromStore(certName);
 
            ClientAssertionCertificate credential = new ClientAssertionCertificate(clientId, cert);
            AuthenticationResult result = await authContext.AcquireTokenAsync("https://management.azure.com/", credential);
            if (result == null)
            {
                throw new InvalidOperationException("Failed to obtain the JWT token");
            }
 
            string token = result.AccessToken;
            string subscriptionId = "<Your_Subscription_ID>";
            string rgName = "<ResourceGroup_of_your_Cosmos_account>";
            string accountName = "<Your_Cosmos_account_name>";
            string cosmosDBRestCall = $"https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.DocumentDB/databaseAccounts/{accountName}/listKeys?api-version=2015-04-08";
 
            Uri restCall = new Uri(cosmosDBRestCall);
            HttpClient httpClient = new HttpClient();
            httpClient.DefaultRequestHeaders.Remove("Authorization");
            httpClient.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);
            HttpResponseMessage response = await httpClient.PostAsync(restCall, null);
 
            Console.WriteLine("Got result {0} and keys {1}", response.StatusCode.ToString(), response.Content.ReadAsStringAsync().Result);
        }
 
        /// <summary>
        /// Reads the certificate
        /// </summary>
        private static X509Certificate2 ReadCertificateFromStore(string certName)
        {
            X509Certificate2 cert = null;
            X509Store store = new X509Store(StoreName.My, StoreLocation.CurrentUser);
            store.Open(OpenFlags.ReadOnly);
            X509Certificate2Collection certCollection = store.Certificates;
 
            // Find unexpired certificates.
            X509Certificate2Collection currentCerts = certCollection.Find(X509FindType.FindByTimeValid, DateTime.Now, false);
 
            // From the collection of unexpired certificates, find the ones with the correct name.
            X509Certificate2Collection signingCert = currentCerts.Find(X509FindType.FindBySubjectName, certName, false);
 
            // Return the first certificate in the collection, has the right name and is current.
            cert = signingCert.OfType<X509Certificate2>().OrderByDescending(c => c.NotBefore).FirstOrDefault();
            store.Close();
            return cert;
        }
    }
}
```

Este script genera las claves principales primaria y secundaria, como se muestra en la captura de pantalla siguiente:

:::image type="content" source="./media/certificate-based-authentication/csharp-application-output.png" alt-text="Salida de la aplicación de csharp":::

De forma similar a la sección anterior puede ver el registro de actividad de la cuenta de Azure Cosmos para validar que la aplicación "sampleApp" inicia el evento de solicitud de claves get. 


## <a name="next-steps"></a>Pasos siguientes

* [Protección de las claves de Azure Cosmos con Azure Key Vault](access-secrets-from-keyvault.md)

* [Línea de base de seguridad para Azure Cosmos DB](security-baseline.md)
