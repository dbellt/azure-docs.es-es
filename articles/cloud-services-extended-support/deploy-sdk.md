---
title: 'Implementación de Cloud Services (soporte extendido): SDK'
description: Implementación de Cloud Services (soporte extendido) con Azure SDK
ms.topic: tutorial
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: a91d26139fae80b38383e7f65f7ae2f58358e6f6
ms.sourcegitcommit: 89c4843ec85d1baea248e81724781d55bed86417
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108795319"
---
# <a name="deploy-cloud-services-extended-support-by-using-the-azure-sdk"></a>Implementación de Cloud Services (soporte extendido) con Azure SDK

En este artículo se muestra cómo usar [Azure SDK](https://azure.microsoft.com/downloads/) para implementar una instancia de Cloud Services (soporte extendido) con varios roles (rol web y rol de trabajo) y la extensión de escritorio remoto. Cloud Services (soporte extendido) es un modelo de implementación de Azure Cloud Services basado en Azure Resource Manager.

## <a name="before-you-begin"></a>Antes de comenzar

Consulte los [requisitos previos de implementación](deploy-prerequisite.md) de Cloud Services (soporte extendido) y cree recursos asociados.

## <a name="deploy-cloud-services-extended-support"></a>Implementación de Cloud Services (soporte extendido)
1. Instale el [paquete NuGet de Azure Compute SDK](https://www.nuget.org/packages/Microsoft.Azure.Management.Compute/43.0.0-preview) e inicialice el cliente con un mecanismo de autenticación estándar.

    ```csharp
        public class CustomLoginCredentials : ServiceClientCredentials
    {
        private string AuthenticationToken { get; set; }
        public override void InitializeServiceClient<T>(ServiceClient<T> client)
           {
               var authenticationContext = new AuthenticationContext("https://login.windows.net/{tenantID}");
               var credential = new ClientCredential(clientId: "{clientID}", clientSecret: "{clientSecret}");
               var result = authenticationContext.AcquireTokenAsync(resource: "https://management.core.windows.net/", clientCredential: credential);
               if (result == null) throw new InvalidOperationException("Failed to obtain the JWT token");
               AuthenticationToken = result.Result.AccessToken;
           }
        public override async Task ProcessHttpRequestAsync(HttpRequestMessage request, CancellationToken cancellationToken)
           {
                if (request == null) throw new ArgumentNullException("request");
                if (AuthenticationToken == null) throw new InvalidOperationException("Token Provider Cannot Be Null");
                request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", AuthenticationToken);
                request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
                //request.Version = new Version(apiVersion);
                await base.ProcessHttpRequestAsync(request, cancellationToken);
            }
    
        var creds = new CustomLoginCredentials();
        m_subId = Environment.GetEnvironmentVariable("AZURE_SUBSCRIPTION_ID");
        ResourceManagementClient m_ResourcesClient = new ResourceManagementClient(creds);
        NetworkManagementClient m_NrpClient = new NetworkManagementClient(creds);
        ComputeManagementClient m_CrpClient = new ComputeManagementClient(creds);
        StorageManagementClient m_SrpClient = new StorageManagementClient(creds);
        m_ResourcesClient.SubscriptionId = m_subId;
        m_NrpClient.SubscriptionId = m_subId;
        m_CrpClient.SubscriptionId = m_subId;
        m_SrpClient.SubscriptionId = m_subId;
    ```

2. Cree un nuevo grupo de recursos mediante la instalación del paquete NuGet de Azure Resource Manager.

    ```csharp 
    var resourceGroups = m_ResourcesClient.ResourceGroups;
    var m_location = “East US”;
    var resourceGroupName = "ContosoRG";//provide existing resource group name, if created already
    var resourceGroup = new ResourceGroup(m_location); 
    resourceGroup = await resourceGroups.CreateOrUpdateAsync(resourceGroupName, resourceGroup);
    ```

3. Cree una cuenta de almacenamiento y un contenedor en el que va a almacenar el paquete de servicio (.cspkg) y los archivos de configuración del servicio (.cscfg). Instale el [paquete NuGet de Azure Storage](https://www.nuget.org/packages/Azure.Storage.Common/). Este paso es opcional si se usa una cuenta de almacenamiento existente. El nombre de la cuenta de almacenamiento debe ser único.

    ```csharp
    string storageAccountName = “ContosoSAS”
    var stoInput = new StorageAccountCreateParameters
       {
            Location = m_location,
            Kind = Microsoft.Azure.Management.Storage.Models.Kind.StorageV2,
            Sku = new Microsoft.Azure.Management.Storage.Models.Sku(SkuName.StandardRAGRS),
        };
            StorageAccount storageAccountOutput = m_SrpClient.StorageAccounts.Create(rgName,
            storageAccountName, stoInput);
            bool created = false;
            while (!created)
               {
                    Thread.Sleep(600);
                    var stos = m_SrpClient.StorageAccounts.ListByResourceGroup(rgName);
                    created =
                    stos.Any(
                        t =>
                        StringComparer.OrdinalIgnoreCase.Equals(t.Name, storageAccountName));
                }
        
    StorageAccount storageAccountOutput = m_SrpClient.StorageAccounts.GetProperties(rgName, storageAccountName);.
    var accountKeyResult = m_SrpClient.StorageAccounts.ListKeysWithHttpMessagesAsync(rgName, storageAccountName).Result;
    CloudStorageAccount storageAccount = new CloudStorageAccount(new StorageCredentials(storageAccountName, accountKeyResult.Body.Keys.FirstOrDefault(). Value), useHttps: true);
        
    var blobClient = storageAccount.CreateCloudBlobClient();
    CloudBlobContainer container = blobClient.GetContainerReference("sascontainer");
    container.CreateIfNotExistsAsync().Wait();
    sharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy();
    sasConstraints.SharedAccessStartTime = DateTime.UtcNow.AddDays(-1);
    sasConstraints.SharedAccessExpiryTime = DateTime.UtcNow.AddDays(2);
    sasConstraints.Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Write;
    ```

4. Cargue el archivo del paquete de servicio (.cspkg) en la cuenta de almacenamiento. La dirección URL del paquete puede ser un identificador URI de firma de acceso compartido (SAS) de cualquier cuenta de almacenamiento.

    ```csharp
    CloudBlockBlob cspkgblockBlob = container.GetBlockBlobReference(“ContosoApp.cspkg”);
    cspkgblockBlob.UploadFromFileAsync(“./ContosoApp/ContosoApp.cspkg”). Wait();
    
    //Generate the shared access signature on the blob, setting the constraints directly on the signature.
    string cspkgsasContainerToken = cspkgblockBlob.GetSharedAccessSignature(sasConstraints);
    
    //Return the URI string for the container, including the SAS token.
    string cspkgSASUrl = cspkgblockBlob.Uri + cspkgsasContainerToken;
    ```

5. Cargue el archivo de configuración del servicio (.cscfg) en la cuenta de almacenamiento. Especifique la configuración del servicio como una cadena XML o con formato de dirección URL.

    ```csharp
    CloudBlockBlob cscfgblockBlob = container.GetBlockBlobReference(“ContosoApp.cscfg”);
    cscfgblockBlob.UploadFromFileAsync(“./ContosoApp/ContosoApp.cscfg”). Wait();
    
    //Generate the shared access signature on the blob, setting the constraints directly on the signature.
    string sasCscfgContainerToken = cscfgblockBlob.GetSharedAccessSignature(sasConstraints);
    
    //Return the URI string for the container, including the SAS token.
    string cscfgSASUrl = cscfgblockBlob.Uri + sasCscfgContainerToken;
    ```

6. Cree una red virtual y una subred. Instale el [paquete NuGet de Azure Network](https://www.nuget.org/packages/Azure.ResourceManager.Network/). Este paso es opcional si se usan una red y una subred existentes.

    ```csharp
    VirtualNetwork vnet = new VirtualNetwork(name: vnetName) 
       { 
            AddressSpace = new AddressSpace 
               { 
                    AddressPrefixes = new List<string> { "10.0.0.0/16" } 
               }, 
                    Subnets = new List<Subnet> 
                    { 
                        new Subnet(name: subnetName) 
                        { 
                            AddressPrefix = "10.0.0.0/24" 
                        } 
                    }, 
                    Location = m_location 
               }; 
    m_NrpClient.VirtualNetworks.CreateOrUpdate(resourceGroupName, “ContosoVNet”, vnet);
    ```

7. Cree una dirección IP pública y establezca la propiedad de la etiqueta DNS de esa dirección IP pública. Cloud Services (soporte extendido) solo admite direcciones IP públicas de la SKU [básica] (https://docs.microsoft.com/azure/virtual-network/public-ip-addresses#basic) ). Las direcciones IP públicas de la SKU estándar no funcionan con Cloud Services.
Si usa una dirección IP estática, debe hacer referencia a ella como IP reservada del archivo de configuración de servicio (.cscfg).

    ```csharp
    PublicIPAddress publicIPAddressParams = new PublicIPAddress(name: “ContosIp”) 
       { 
            Location = m_location, 
            PublicIPAllocationMethod = IPAllocationMethod.Dynamic, 
            DnsSettings = new PublicIPAddressDnsSettings() 
               { 
                    DomainNameLabel = “contosoappdns” 
               } 
       }; 
    PublicIPAddress publicIpAddress = m_NrpClient.PublicIPAddresses.CreateOrUpdate(resourceGroupName, publicIPAddressName, publicIPAddressParams);
    ```

8. Cree un objeto de perfil de red y asocie la dirección IP pública al front-end del equilibrador de carga. La plataforma Azure crea automáticamente un recurso de equilibrador de carga de SKU "clásica" en la misma suscripción que el recurso de servicio en la nube. El recurso de equilibrador de carga es un recurso de solo lectura en ARM. Las actualizaciones del recurso solo se admiten a través de los archivos de implementación de servicios en la nube (.cscfg y .csdef).

    ```csharp
    LoadBalancerFrontendIPConfiguration feipConfiguration = new LoadBalancerFrontendIPConfiguration() 
        { 
            Name = “ContosoFe”, 
            Properties = new LoadBalancerFrontendIPConfigurationProperties() 
                { 
                PublicIPAddress = new CM.SubResource() 
                    { 
                        Id = $"/subscriptions/{m_subId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/publicIPAddresses/{publicIPAddressName}", 
                    } 
                } 
        }; 
    
    CloudServiceNetworkProfile cloudServiceNetworkProfile = new CloudServiceNetworkProfile() 
        { 
            LoadBalancerConfigurations = new List<LoadBalancerConfiguration>() 
                { 
                    new LoadBalancerConfiguration() 
                       { 
                        Name  = 'ContosoLB', 
                        Properties = new LoadBalancerConfigurationProperties() 
                            { 
                            FrontendIPConfigurations = new List<LoadBalancerFrontendIPConfiguration>() 
                                { 
                                feipConfig 
                                } 
                            } 
                        } 
                } 
        }; 
    
    ```

9. Cree un almacén de claves. Este almacén de claves se usará para almacenar los certificados asociados a los roles de Cloud Services (soporte extendido). El almacén de claves debe estar ubicado en la misma región y suscripción que la instancia de Cloud Services (soporte extendido), y debe tener un nombre único. Para más información, consulte [Uso de certificados con Azure Cloud Services (soporte extendido)](certificates-and-key-vault.md).

    ```powershell
    New-AzKeyVault -Name "ContosKeyVault” -ResourceGroupName “ContosoOrg” -Location “East US”
    ```

10. Actualice la directiva de acceso del almacén de claves y conceda permisos de certificados a la cuenta de usuario.

    ```powershell
    Set-AzKeyVaultAccessPolicy -VaultName 'ContosKeyVault' -ResourceGroupName 'ContosoOrg'      -UserPrincipalName 'user@domain.com' -PermissionsToCertificates create,get,list,delete
    ```

    Como alternativa, establezca la directiva de acceso mediante el identificador de objeto (puede ejecutar `Get-AzADUser` para obtenerlo).

    ```powershell
    Set-AzKeyVaultAccessPolicy -VaultName 'ContosKeyVault' -ResourceGroupName 'ContosOrg' -     ObjectId 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' -PermissionsToCertificates          create,get,list,delete
    ```

11. En este ejemplo, agregaremos un certificado autofirmado a un almacén de claves. Es necesario agregar la huella digital del certificado en el archivo de configuración del servicio (.cscfg) para su implementación en los roles de Cloud Services (soporte extendido).

    ```powershell
    $Policy = New-AzKeyVaultCertificatePolicy -SecretContentType "application/x-pkcs12" -       SubjectName "CN=contoso.com" -IssuerName "Self" -ValidityInMonths 6 -ReuseKeyOnRenewal 
    Add-AzKeyVaultCertificate -VaultName "ContosKeyVault" -Name "ContosCert" -      CertificatePolicy $Policy
    ```

12. Cree un objeto de perfil de SO. El perfil de SO especifica los certificados asociados a los roles de Cloud Services (soporte extendido). Aquí, es el mismo certificado que hemos creado en el paso anterior.

    ```csharp
    CloudServiceOsProfile cloudServiceOsProfile = 
        new CloudServiceOsProfile 
           { 
                Secrets = new List<CloudServiceVaultSecretGroup> 
                   { 
                        New CloudServiceVaultSecretGroup { 
                        SourceVault = <sourceVault>, 
                        VaultCertificates = <vaultCertificates> 
                        } 
                   } 
           };
    ```

13. Cree un objeto de perfil de rol. El perfil de rol define las propiedades específicas del rol de una SKU, como el nombre, la capacidad y el nivel. 

    En este ejemplo se definen dos roles: ContosoFrontend y ContosoBackend. La información del perfil de rol debe coincidir con la configuración de rol definida en el archivo de configuración del servicio (cscfg) y el archivo de definición del servicio (.csdef).

    ```csharp
    CloudServiceRoleProfile cloudServiceRoleProfile = new CloudServiceRoleProfile()
       {
            Roles = new List<CloudServiceRoleProfileProperties>();
            
                // foreach role in cloudService
                roles.Add(new CloudServiceRoleProfileProperties()
                    {
                        Name = 'ContosoFrontend',
                        Sku = new CloudServiceRoleSku
                        {
                            Name = 'Standard_D1_v2',
                            Capacity = 2,
                            Tier = 'Standard'
                        }
                    );
    
                roles.Add(new CloudServiceRoleProfileProperties()
                    {
                        Name = 'ContosoBackend',
                        Sku = new CloudServiceRoleSku
                        {
                            Name = 'Standard_D1_v2',
                            Capacity = 2,
                            Tier = 'Standard'
                        }
                    );
    
                    }
                    }
    ```

14. (Opcional) Cree un objeto de perfil de extensión que desee agregar a la instancia de Cloud Services (soporte extendido). En este ejemplo se agrega la extensión RDP.

    ```csharp
    string rdpExtensionPublicConfig = "<PublicConfig>" +
        "<UserName>adminRdpTest</UserName>" +
        "<Expiration>2021-10-27T23:59:59</Expiration>" +
        "</PublicConfig>";
        string rdpExtensionPrivateConfig = "<PrivateConfig>" +
        "<Password>VsmrdpTest!</Password>" +
        "</PrivateConfig>";
    
        Extension rdpExtension = new Extension
                {
                    Name = name,
                    Properties = new CloudServiceExtensionProperties
                    {
                        Publisher = "Microsoft.Windows.Azure.Extensions",
                        Type = "RDP",
                        TypeHandlerVersion = "1.2.1",,
                        AutoUpgradeMinorVersion = true,
                        Settings = rdpExtensionPublicConfig,
                        ProtectedSettings = rdpExtensionPrivateConfig,
                        RolesAppliedTo = [“*”],
                    }
                };
                        
    CloudServiceExtensionProfile cloudServiceExtensionProfile = new CloudServiceExtensionProfile
        {
            Extensions = rdpExtension
        };
    ```

15. Cree la implementación de la instancia de Cloud Services (soporte extendido).

    ```csharp
    CloudService cloudService = new CloudService
        {
            Properties = new CloudServiceProperties
                {
                    RoleProfile = cloudServiceRoleProfile
                    Configuration = < Add Cscfg xml content here>,
                    // ConfigurationUrl = <Add your configuration URL here>,
                    PackageUrl = <Add cspkg SAS url here>,
                    ExtensionProfile = cloudServiceExtensionProfile,
                    OsProfile= cloudServiceOsProfile,
                    NetworkProfile = cloudServiceNetworkProfile,
                    UpgradeMode = 'Auto'
                },
                    Location = m_location
                };
    
    CloudService createOrUpdateResponse = m_CrpClient.CloudServices.CreateOrUpdate(“ContosOrg”, “ContosoCS”, cloudService);
    ```

## <a name="next-steps"></a>Pasos siguientes
- Vea las [preguntas más frecuentes](faq.md) sobre Cloud Services (soporte extendido).
- Implemente una instancia de Cloud Services (soporte extendido) mediante [Azure Portal](deploy-portal.md), [PowerShell](deploy-powershell.md), una [plantilla](deploy-template.md) o [Visual Studio](deploy-visual-studio.md).
- Consulte [Repositorio de ejemplos de Cloud Services (soporte extendido)](https://github.com/Azure-Samples/cloud-services-extended-support).
