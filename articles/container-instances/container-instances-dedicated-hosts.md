---
title: Implementación en un host dedicado
description: Uso de un host dedicado para lograr un aislamiento a nivel de host real para las cargas de trabajo de Azure Container Instances
ms.topic: article
ms.date: 01/17/2020
author: macolso
ms.author: macolso
ms.openlocfilehash: 72ad05eea9232e7a0d6ac24d1d0d22a971a7f6a5
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107790840"
---
# <a name="deploy-on-dedicated-hosts"></a>Implementación en hosts dedicados

"Dedicado" es una SKU de Azure Container Instances (ACI) que proporciona un entorno de proceso aislado y dedicado para ejecutar contenedores de forma segura. El uso de la SKU dedicada da como resultado que cada grupo de contenedores tenga un servidor físico dedicado en un centro de recursos de Azure, lo que garantiza un aislamiento completo de la carga de trabajo para ayudar a satisfacer los requisitos de cumplimiento y seguridad de su organización. 

La SKU dedicada es adecuada para cargas de trabajo de contenedor que requieren aislamiento de la carga de trabajo desde una perspectiva de servidor físico.

## <a name="prerequisites"></a>Prerrequisitos

> [!NOTE]
> Debido a algunas limitaciones actuales, no se garantiza la aprobación de todas las solicitudes de aumento de límite.

* El límite predeterminado de cualquier suscripción para usar la SKU dedicada es 0. Si desea usar esta SKU para las implementaciones de contenedores de producción, cree una [solicitud de soporte técnico de Azure][azure-support] para aumentar el límite.

## <a name="use-the-dedicated-sku"></a>Uso de la SKU dedicada

> [!IMPORTANT]
> El uso de la SKU dedicada solo está disponible en la versión de API más reciente (2019-12-01) que se está implementando actualmente. Especifique esta versión de API en la plantilla de implementación.
>

A partir de la versión 2019-12-01 de la API, hay una propiedad `sku` en la sección Propiedades del grupo de contenedores de una plantilla de implementación, que es necesaria para una implementación de ACI. Actualmente, puede usar esta propiedad como parte de una plantilla de implementación de Azure Resource Manager para ACI. Obtenga más información sobre la implementación de recursos de ACI con una plantilla en el [Tutorial: Implementación de un grupo con varios contenedores con una plantilla de Resource Manager](./container-instances-multi-container-group.md). 

La propiedad `sku` admite cualquiera de los siguientes valores:
* `Standard`: la opción de implementación de ACI estándar, que sigue garantizando la seguridad de nivel de hipervisor 
* `Dedicated`: se usa para el aislamiento de nivel de carga de trabajo con hosts físicos dedicados para el grupo de contenedores

## <a name="modify-your-json-deployment-template"></a>Modificar la plantilla de implementación JSON

En la plantilla de implementación, modifique o agregue las siguientes propiedades:
* En `resources`, establezca `apiVersion` en `2019-12-01`.
* En las propiedades del grupo de contenedores, agregue una propiedad `sku` con el valor `Dedicated`.

Este es un fragmento de ejemplo para la sección de recursos de una plantilla de implementación de grupo de contenedores que usa la SKU dedicada:

```json
[...]
"resources": [
    {
        "name": "[parameters('containerGroupName')]",
        "type": "Microsoft.ContainerInstance/containerGroups",
        "apiVersion": "2019-12-01",
        "location": "[resourceGroup().location]",    
        "properties": {
            "sku": "Dedicated",
            "containers": {
                [...]
            }
        }
    }
]
```

A continuación, se muestra una plantilla completa que implementa un grupo de contenedores de ejemplo que ejecuta una única instancia de contenedor:

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
    "resources": [
        {
            "name": "[parameters('containerGroupName')]",
            "type": "Microsoft.ContainerInstance/containerGroups",
            "apiVersion": "2019-12-01",
            "location": "[resourceGroup().location]",
            "properties": {
                "sku": "Dedicated",
                "containers": [
                    {
                        "name": "container1",
                        "properties": {
                            "image": "nginx",
                            "command": [
                                "/bin/sh",
                                "-c",
                                "while true; do echo `date`; sleep 1000000; done"
                            ],
                            "ports": [
                                {
                                    "protocol": "TCP",
                                    "port": 80
                                }
                            ],
                            "environmentVariables": [],
                            "resources": {
                                "requests": {
                                    "memoryInGB": 1.0,
                                    "cpu": 1
                                }
                            }
                        }
                    }
                ],
                "restartPolicy": "Always",
                "ipAddress": {
                    "ports": [
                        {
                            "protocol": "TCP",
                            "port": 80
                        }
                    ],
                    "type": "Public"
                },
                "osType": "Linux"
            },
            "tags": {}
        }
    ]
}
```

## <a name="deploy-your-container-group"></a>Implementación del grupo de contenedores

Si ha creado y editado el archivo de plantilla de implementación en el escritorio, puede cargarlo en el directorio de Cloud Shell si lo arrastra a dicho directorio. 

Para crear un grupo de recursos, use el comando [az group create][az-group-create].

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Implemente la plantilla con el comando [az deployment group create][az-deployment-group-create].

```azurecli-interactive
az deployment group create --resource-group myResourceGroup --template-file deployment-template.json
```

Al cabo de unos segundos, debe recibir una respuesta inicial de Azure. Se realizará una implementación correcta en un host dedicado.

<!-- LINKS - Internal -->
[az-group-create]: /cli/azure/group#az_group_create
[az-deployment-group-create]: /cli/azure/deployment/group#az_deployment_group_create

<!-- LINKS - External -->
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest
