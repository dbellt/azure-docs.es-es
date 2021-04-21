---
title: Cifrado de datos de implementación
description: Obtenga información sobre el cifrado de datos persistentes para los recursos de instancia de contenedor y cómo cifrar los datos con una clave administrada por el cliente.
ms.topic: article
ms.date: 01/17/2020
author: macolso
ms.author: macolso
ms.openlocfilehash: 23c81aeab3bf6e9ee7f2d89fbdf8def20dab4aa7
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107790876"
---
# <a name="encrypt-deployment-data"></a>Cifrado de datos de implementación

Al ejecutar los recursos de Azure Container Instances (ACI) en la nube, el servicio ACI recopila y conserva los datos relacionados con los contenedores. ACI cifra automáticamente los datos al guardarlos en la nube. Este cifrado protege los datos para cumplir los compromisos de cumplimiento y seguridad de la organización. ACI también ofrece la opción de cifrar estos datos con una clave propia, lo que proporciona un mayor control sobre los datos relacionados con las implementaciones de ACI.

## <a name="about-aci-data-encryption"></a>Acerca del cifrado de datos de ACI 

Los datos en ACI se cifran y descifran con el cifrado AES de 256 bits. Este está habilitado para todas las implementaciones de ACI y no es necesario modificar la implementación ni los contenedores para usarlo. Esto incluye los metadatos sobre la implementación, las variables de entorno, las claves que se pasan a los contenedores y los registros que se conservan después de detener los contenedores para que pueda verlos. El cifrado no afecta al rendimiento del grupo de contenedores y el cifrado no genera ningún costo adicional.

## <a name="encryption-key-management"></a>Administración de claves de cifrado

Puede confiar en las claves administradas por Microsoft para el cifrado de los datos de los contenedores, o puede administrar el cifrado con sus propias claves. En la siguiente tabla se comparan estas opciones: 

|    |    Claves administradas por Microsoft     |     Claves administradas por el cliente     |
|----|----|----|
|    **Operaciones de cifrado y descifrado**    |    Azure    |    Azure    |
|    **Almacenamiento de claves**    |    Almacén de claves de Microsoft    |    Azure Key Vault    |
|    **Responsabilidad de la rotación de claves**    |    Microsoft    |    Customer    |
|    **Acceso a la clave**    |    Solo Microsoft    |    Microsoft, cliente    |

En el resto del documento se describen los pasos necesarios para cifrar los datos de implementación de ACI con su clave (clave administrada por el cliente). 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="encrypt-data-with-a-customer-managed-key"></a>Cifrado de datos con una clave administrada por el cliente

### <a name="create-service-principal-for-aci"></a>Crear una entidad de servicio para ACI

El primer paso es asegurarse de que el [inquilino de Azure](../active-directory/develop/quickstart-create-new-tenant.md) tiene asignada una entidad de servicio para conceder permisos al servicio de Azure Container Instances. 

> [!IMPORTANT]
> Para ejecutar el siguiente comando y crear una entidad de servicio correctamente, confirme que tiene permisos para crear entidades de servicio en el inquilino.
>

El siguiente comando de la CLI configurará la entidad de servicio de ACI en el entorno de Azure:

```azurecli-interactive
az ad sp create --id 6bb8e274-af5d-4df2-98a3-4fd78b4cafd9
```

El resultado de la ejecución de este comando debe mostrar una entidad de servicio configurada con el valor "displayName": "Azure Container Instance Service".

En caso de que no pueda crear correctamente la entidad de servicio:
* confirme que tiene permisos para hacerlo en el inquilino
* compruebe si ya existe una entidad de servicio en el inquilino para la implementación en ACI. Para ello, ejecute `az ad sp show --id 6bb8e274-af5d-4df2-98a3-4fd78b4cafd9` y use esa entidad de servicio en su lugar.

### <a name="create-a-key-vault-resource"></a>Creación de un recurso de Key Vault

Cree una instancia de Azure Key Vault mediante [Azure Portal](../key-vault/general/quick-create-portal.md), la [CLI de Azure](../key-vault/general/quick-create-cli.md) o [Azure PowerShell](../key-vault/general/quick-create-powershell.md).

En el caso de las propiedades del almacén de claves, siga estas instrucciones: 
* Nombre: se requiere un nombre único. 
* Suscripción: Elija una suscripción.
* En Grupo de recursos, seleccione un grupo existente o cree uno y escriba un nombre para el grupo de recursos.
* En el menú desplegable Ubicación elija una ubicación.
* Puede dejar los valores predeterminados de las demás opciones o elegir otros en función de los requisitos adicionales.

> [!IMPORTANT]
> Al usar claves administradas por el cliente para cifrar una plantilla de implementación de ACI, se recomienda establecer las dos propiedades siguientes en el almacén de claves, Eliminación temporal y No purgar. Estas propiedades no están habilitadas de forma predeterminada, pero se pueden habilitar mediante PowerShell o la CLI de Azure tanto en un almacén de claves nuevo como en uno existente.

### <a name="generate-a-new-key"></a>Generar una nueva clave 

Una vez creado el almacén de claves, navegue hasta el recurso en Azure Portal. En el menú de navegación izquierdo de la hoja de recursos, en Configuración, haga clic en **Claves**. En la vista "Claves", haga clic en "Generar/importar" para generar una nueva clave. Use cualquier nombre único para esta clave y establezca cualquier otra preferencia en función de sus necesidades. 

![Generar una nueva clave](./media/container-instances-encrypt-data/generate-key.png)

### <a name="set-access-policy"></a>Establecer la directiva de acceso

Cree una nueva directiva de acceso para permitir que el servicio ACI tenga acceso a la clave.

* Una vez generada la clave, en la hoja de recursos del almacén de claves, en Configuración, haga clic en **Directivas de acceso**.
* En la página "Directivas de acceso" del almacén de claves, haga clic en **Agregar directiva de acceso**.
* Establezca los *Permisos de las claves* para incluir las opciones **Obtener** y **Desencapsular clave**. ![Establecer los permisos de las claves](./media/container-instances-encrypt-data/set-key-permissions.png)
* En *Seleccionar la entidad de seguridad*, seleccione **Servicio Azure Container Instance**.
* Haga clic en **Agregar** en la parte inferior. 

La directiva de acceso debe mostrarse ahora en las directivas de acceso del almacén de claves.

![Nueva directiva de acceso](./media/container-instances-encrypt-data/access-policy.png)

### <a name="modify-your-json-deployment-template"></a>Modificar la plantilla de implementación JSON

> [!IMPORTANT]
> El cifrado de datos de implementación con una clave administrada por el cliente está disponible en la versión más reciente de la API (2019-12-01) que se está implementando actualmente. Especifique esta versión de API en la plantilla de implementación. Si tiene algún problema con este valor, póngase en contacto con el soporte técnico de Azure.

Una vez configuradas la clave del almacén de claves y la directiva de acceso, agregue las siguientes propiedades a la plantilla de implementación de ACI. Obtenga más información sobre la implementación de recursos de ACI con una plantilla en el [Tutorial: Implementación de un grupo con varios contenedores con una plantilla de Resource Manager](./container-instances-multi-container-group.md). 
* En `resources`, establezca `apiVersion` en `2019-12-01`.
* En la sección Propiedades del grupo de contenedores de la plantilla de implementación, agregue un elemento `encryptionProperties` con los siguientes valores:
  * `vaultBaseUrl`: nombre DNS del almacén de claves, que se puede encontrar en la hoja de Información general del recurso de almacén de claves en el portal.
  * `keyName`: nombre de la clave generada anteriormente.
  * `keyVersion`: versión actual de la clave. Para encontrar este valor, haga clic en la clave (debajo de "Claves" en la sección Configuración del recurso de almacén de claves).
* En las propiedades del grupo de contenedores, agregue una propiedad `sku` con el valor `Standard`. La propiedad `sku` es obligatoria en la versión 2019-12-01 de la API.

El siguiente fragmento de plantilla muestra estas propiedades adicionales para cifrar los datos de implementación:

```json
[...]
"resources": [
    {
        "name": "[parameters('containerGroupName')]",
        "type": "Microsoft.ContainerInstance/containerGroups",
        "apiVersion": "2019-12-01",
        "location": "[resourceGroup().location]",    
        "properties": {
            "encryptionProperties": {
                "vaultBaseUrl": "https://example.vault.azure.net",
                "keyName": "acikey",
                "keyVersion": "xxxxxxxxxxxxxxxx"
            },
            "sku": "Standard",
            "containers": {
                [...]
            }
        }
    }
]
```

A continuación se muestra una plantilla completa, adaptada a partir de la plantilla de [Tutorial: Implementación de un grupo con varios contenedores con una plantilla de Resource Manager](./container-instances-multi-container-group.md). 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "containerGroupName": {
      "type": "string",
      "defaultValue": "myContainerGroup",
      "metadata": {
        "description": "Container Group name."
      }
    }
  },
  "variables": {
    "container1name": "aci-tutorial-app",
    "container1image": "mcr.microsoft.com/azuredocs/aci-helloworld:latest",
    "container2name": "aci-tutorial-sidecar",
    "container2image": "mcr.microsoft.com/azuredocs/aci-tutorial-sidecar"
  },
  "resources": [
    {
      "name": "[parameters('containerGroupName')]",
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2019-12-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "encryptionProperties": {
            "vaultBaseUrl": "https://example.vault.azure.net",
            "keyName": "acikey",
            "keyVersion": "xxxxxxxxxxxxxxxx"
        },
        "sku": "Standard",  
        "containers": [
          {
            "name": "[variables('container1name')]",
            "properties": {
              "image": "[variables('container1image')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              },
              "ports": [
                {
                  "port": 80
                },
                {
                  "port": 8080
                }
              ]
            }
          },
          {
            "name": "[variables('container2name')]",
            "properties": {
              "image": "[variables('container2image')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              }
            }
          }
        ],
        "osType": "Linux",
        "ipAddress": {
          "type": "Public",
          "ports": [
            {
              "protocol": "tcp",
              "port": "80"
            },
            {
                "protocol": "tcp",
                "port": "8080"
            }
          ]
        }
      }
    }
  ],
  "outputs": {
    "containerIPv4Address": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.ContainerInstance/containerGroups/', parameters('containerGroupName'))).ipAddress.ip]"
    }
  }
}
```

### <a name="deploy-your-resources"></a>Implementar los recursos

Si ha creado y editado el archivo de plantilla en el escritorio, puede cargarlo en el directorio de Cloud Shell si lo arrastra a dicho directorio. 

Para crear un grupo de recursos, use el comando [az group create][az-group-create].

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Implemente la plantilla con el comando [az deployment group create][az-deployment-group-create].

```azurecli-interactive
az deployment group create --resource-group myResourceGroup --template-file deployment-template.json
```

Al cabo de unos segundos, debe recibir una respuesta inicial de Azure. Una vez completada la implementación, todos los datos relacionados con Azure que conserva el servicio ACI se cifrarán con la clave proporcionada.

<!-- LINKS - Internal -->
[az-group-create]: /cli/azure/group#az_group_create
[az-deployment-group-create]: /cli/azure/deployment/group/#az_deployment_group_create
