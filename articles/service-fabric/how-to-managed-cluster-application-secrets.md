---
title: Uso de secretos de aplicación en clústeres administrados de Service Fabric
description: Obtenga información sobre los secretos de aplicación de Azure Service Fabric y sobre cómo recopilar la información necesaria para su uso en clústeres administrados
ms.topic: how-to
ms.date: 5/10/2021
ms.openlocfilehash: 820fb2a116ba5343a2f2126950a7f5d5896ddee3
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111950116"
---
# <a name="use-application-secrets-in-service-fabric-managed-clusters"></a>Uso de secretos de aplicación en clústeres administrados de Service Fabric

Los secretos pueden ser cualquier información confidencial, como cadenas de conexión de almacenamiento, contraseñas u otros valores que no se deben administrar en texto sin formato. En este artículo se usa Azure Key Vault para administrar claves y secretos, ya que se trata de clústeres administrados de Service Fabric. Sin embargo, el *uso* de secretos en una aplicación es independiente de la plataforma de nube para permitir que las aplicaciones se implementen en un clúster hospedado en cualquier parte.

La forma recomendada de administrar las opciones de configuración de servicio es mediante los [paquetes de configuración de servicio][config-package]. Los paquetes de configuración se actualizan mediante actualizaciones acumulativas administradas con validación de estado y reversión automática. Esto es preferible a la configuración global ya que reduce las posibilidades de una interrupción del servicio global. Los secretos cifrados no son ninguna excepción. Service Fabric presenta características integradas para cifrar y descifrar valores en un archivo Settings.xml de paquete de configuración mediante cifrado de certificados.

En el diagrama siguiente se ilustra el flujo básico para la administración de secretos en una aplicación de Service Fabric:

![información general de administración de secretos][overview]

Hay cuatro pasos principales en este flujo:

1. Obtener un certificado de cifrado de datos.
2. Instalar el certificado en el clúster.
3. Cifrar los valores de secreto al implementar una aplicación con el certificado e insértelos en un archivo de configuración Settings.xml del servicio.
4. Leer los valores cifrados de Settings.xml y usar el mismo certificado de cifrado para descifrarlos. 

[Azure Key Vault][key-vault-get-started] se usa aquí como ubicación de almacenamiento segura para los certificados y como forma de obtener los certificados instalados en los nodos de clústeres administrados de Service Fabric en Azure.

Para obtener un ejemplo de la implementación de secretos de aplicaciones, vea [Administración de secretos de aplicación](service-fabric-application-secret-management.md).

Como alternativa, también se admite [KeyVaultReference](service-fabric-keyvault-references.md). La compatibilidad de KeyVaultReference de Service Fabric facilita la implementación de secretos en las aplicaciones con solo hacer referencia a la dirección URL del secreto almacenado en Key Vault.

## <a name="create-a-data-encipherment-certificate"></a>Creación de un certificado de cifrado de datos
Para crear un almacén de claves propio y configurar certificados, siga las instrucciones de Azure Key Vault mediante la [CLI de Azure, PowerShell, el portal, etc.][key-vault-certs]

>[!NOTE]
> El almacén de claves debe estar [habilitado para la implementación de plantillas](../key-vault/general/manage-with-cli2.md#bkmk_KVperCLI) de forma que el proveedor de recursos de proceso pueda obtener certificados de él e instalarlos en nodos de clústeres.

## <a name="install-the-certificate-in-your-cluster"></a>Instalación del certificado en el clúster
Este certificado debe instalarse en cada nodo del clúster y los clústeres administrados de Service Fabric lo facilitan. El servicio de clúster administrado puede insertar secretos específicos de la versión en los nodos para ayudar a instalar secretos que no cambien con frecuencia, como la instalación de una CA raíz privada en los nodos. Para la mayoría de las cargas de trabajo de producción se recomienda usar la [extensión KeyVault][key-vault-windows]. La extensión de máquina virtual de Key Vault proporciona actualización automática de certificados almacenados en un almacén de claves de Azure frente a una versión estática.

En el caso de los clústeres administrados se necesitan tres valores, dos de Azure Key Vault y uno que decida para el nombre del almacén local en los nodos.

Parámetros: 
* Almacén de origen: es el 
    * por ejemplo: /subscriptions/{subscriptionid}/resourceGroups/myrg1/providers/Microsoft.KeyVault/vaults/mykeyvault1
* URL del certificado: es el identificador de objeto completo, no distingue mayúsculas de minúsculas y es inmutable
    * https://mykeyvault1.vault.azure.net/secrets/{secretname}/{secret-version}
* Almacén de certificados: es el almacén de certificados local en los nodos donde se va a colocar el certificado
    * nombre del almacén de certificados en los nodos, por ejemplo: "MY"

Los clústeres administrados de Service Fabric admiten dos métodos para agregar secretos específicos de la versión a los nodos.

1. Durante la creación inicial del clúster, inserte solo valores de arriba en esta área:

![entrada de secretos del portal][sfmc-secrets]

2. Azure Resource Manager durante la creación o en cualquier momento

```json
{
    "apiVersion": "2021-05-01",
    "type": "Microsoft.ServiceFabric/managedclusters/nodetypes",
    "properties": {
        "vmSecrets": [
          {
            "sourceVault": {
              "id": "/subscriptions/{subscriptionid}/resourceGroups/myrg1/providers/Microsoft.KeyVault/vaults/mykeyvault1"
            },
            "vaultCertificates": [
              {
                "certificateStore": "MY",
                "certificateUrl": "https://mykeyvault1.vault.azure.net/certificates/{certificatename}/{secret-version}"
              }
            ]
          }
        ]
    }    
}
```


<!-- Links -->
[key-vault-get-started]:../key-vault/general/overview.md
[key-vault-certs]: ../key-vault/certificates/quick-create-cli.md
[config-package]: service-fabric-application-and-service-manifests.md
[key-vault-windows]: ../virtual-machines/extensions/key-vault-windows.md

<!-- Images -->
[overview]:./media/service-fabric-application-and-service-security/overview.png
[sfmc-secrets]:./media/how-to-managed-cluster-application-secrets/sfmc-secrets.png