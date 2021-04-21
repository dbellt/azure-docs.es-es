---
title: Ejecución de contenedores de inicialización
description: Ejecute los contenedores de inicialización en Azure Container Instances para realizar tareas de instalación en un grupo de contenedores antes de que se ejecuten los contenedores de la aplicación.
ms.topic: article
ms.date: 06/01/2020
ms.openlocfilehash: 9ccaf1a67d6ca3bcff422acb591b528cc72a9608
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107763944"
---
# <a name="run-an-init-container-for-setup-tasks-in-a-container-group"></a>Ejecución de un contenedor de inicialización para tareas de configuración en un grupo de contenedores

Azure Container Instances admite *contenedores de inicialización* en un grupo de contenedores. Los contenedores de inicialización se ejecutan por completo antes de que se inicie el contenedor o contenedores de la aplicación. De modo similar a los [contenedores de inicialización de Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/), utilice uno o varios contenedores de inicialización para realizar la lógica de inicialización de los contenedores de la aplicación, como establecer cuentas, ejecutar scripts de instalación o configurar bases de datos.

En este artículo se muestra cómo usar una plantilla de Azure Resource Manager para configurar un grupo de contenedores con un contenedor de inicialización.

## <a name="things-to-know"></a>Cosas que debe saber

* **Versión de API**: para implementar contenedores de inicialización al menos se requiere la versión 2019-12-01 de la API de Azure Container Instances. Realice la implementación con una propiedad `initContainers` en un [archivo YAML](container-instances-multi-container-yaml.md) o una [plantilla de Resource Manager](container-instances-multi-container-group.md).
* **Orden de ejecución**: los contenedores de inicialización se ejecutan en el orden especificado en la plantilla y antes que otros contenedores. De forma predeterminada, puede especificar 59 contenedores de inicialización, como máximo, por cada grupo de contenedores. En el grupo debe haber al menos un contenedor que no sea de inicialización.
* **Entorno de host**: los contenedores de inicialización se ejecutan en el mismo hardware que el resto de los contenedores del grupo.
* **Recursos**: no se especifican recursos para los contenedores de inicialización. Se les conceden los recursos totales como la CPU y la memoria disponible para el grupo de contenedores. Mientras se ejecuta un contenedor de inicialización, no se ejecuta ningún otro contenedor en el grupo.
* **Propiedades admitidas**: los contenedores de inicialización pueden utilizar propiedades de grupo como volúmenes, secretos e identidades administradas. Sin embargo, no pueden usar puertos ni una dirección IP si están configurados para el grupo de contenedores. 
* **Directiva de reinicio**: cada contenedor de inicialización debe finalizar correctamente antes de que se inicie el siguiente contenedor del grupo. Si un contenedor de inicialización no finaliza correctamente, su acción de reinicio depende de la [directiva de reinicio](container-instances-restart-policy.md) configurada para el grupo:

    |Directiva en el grupo  |Directiva en la inicialización  |
    |---------|---------|
    |Siempre     |OnFailure         |
    |OnFailure     |OnFailure         |
    |Nunca     |Nunca         |
* **Cargos**: el grupo de contenedores incurre en cargos desde la primera implementación de un contenedor de inicialización.

## <a name="resource-manager-template-example"></a>Ejemplo de plantilla de Resource Manager

Para empezar, copie el siguiente código JSON en un nuevo archivo denominado `azuredeploy.json`. La plantilla configura un grupo de contenedores con un contenedor de inicialización y dos contenedores de aplicaciones:

* El contenedor *init1* ejecuta la imagen [busybox](https://hub.docker.com/_/busybox) desde Docker Hub. Se suspende durante 60 segundos y, a continuación, escribe una cadena de línea de comandos en un archivo de un [volumen emptyDir](container-instances-volume-emptydir.md).
* Ambos contenedores de aplicaciones ejecutan la imagen de contenedor `aci-wordcount` de Microsoft:
    * El contenedor *Hamlet* ejecuta la aplicación de recuento de palabras en su configuración predeterminada y cuenta la frecuencia de palabras en la obra de Shakespeare *Hamlet*.
    * El contenedor de la aplicación *Juliet* lee la cadena de línea de comandos del volumen emptyDir para ejecutar la aplicación de recuento de palabras en su lugar, en la obra *Romeo y Julieta* de Shakespeare.

Para obtener más información y ejemplos del uso de la imagen de `aci-wordcount`, consulte [Establecimiento de variables de entorno en instancias de contenedor](container-instances-environment-variables.md).

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
                "sku": "Standard",
                "initContainers": [
                {
                    "name": "init1",
                    "properties": {
                        "image": "busybox",
                        "environmentVariables": [],
                        "volumeMounts": [
                            {
                                "name": "emptydir1",
                                "mountPath": "/mnt/emptydir"
                            }
                        ],
                         "command": [
                            "/bin/sh",
                            "-c",
                            "sleep 60; echo python wordcount.py http://shakespeare.mit.edu/romeo_juliet/full.html > /mnt/emptydir/command_line.txt"
                        ]
                    }
                }
            ], 
                "containers": [
                    {
                        "name": "hamlet",
                        "properties": {
                            "image": "mcr.microsoft.com/azuredocs/aci-wordcount",
                            "volumeMounts": [
                            {
                                "name": "emptydir1",
                                "mountPath": "/mnt/emptydir"
                            }
                        ],
                            "environmentVariables": [
                             {
                                "name": "NumWords",
                                "value": "3"
                             },
                             {  "name": "MinLength",
                                "value": "5"
                             }
                            ],
                            "resources": {
                                "requests": {
                                    "memoryInGB": 1.0,
                                    "cpu": 1
                                }
                            }
                        }
                    },
                    {
                        "name": "juliet",
                        "properties": {
                            "image": "mcr.microsoft.com/azuredocs/aci-wordcount",
                            "volumeMounts": [
                            {
                                "name": "emptydir1",
                                "mountPath": "/mnt/emptydir"
                            }
                            ],
                            "command": [
                                "/bin/sh",
                                "-c",
                                "$(cat /mnt/emptydir/command_line.txt)"
                            ],
                            "environmentVariables": [
                             {
                                "name": "NumWords",
                                "value": "3"
                             },
                             {  "name": "MinLength",
                                "value": "5"
                             }
                            ],
                            "resources": {
                                "requests": {
                                    "memoryInGB": 1.0,
                                    "cpu": 1
                                }
                            }
                        }
                    }
                ],
                "restartPolicy": "OnFailure",
                "osType": "Linux",
                "volumes": [
                    {
                        "name": "emptydir1",
                        "emptyDir": {}
                    }
                ]           
            },
            "tags": {}
        }
    ]
}
```

## <a name="deploy-the-template"></a>Implementación de la plantilla

Para crear un grupo de recursos, use el comando [az group create][az-group-create].

```azurecli
az group create --name myResourceGroup --location eastus
```

Implemente la plantilla con el comando [az deployment group create][az-deployment-group-create].

```azurecli
az deployment group create \
  --resource-group myResourceGroup \
  --template-file azuredeploy.json
```

En un grupo con un contenedor de inicialización, el tiempo de implementación aumenta debido al tiempo que tardan en completarse el contenedor o contenedores de inicialización.


## <a name="view-container-logs"></a>Visualización de registros de contenedores

Para comprobar que el contenedor de inicialización se ejecutó correctamente, vea la salida del registro de los contenedores de la aplicación mediante el comando [az container logs][az-container-logs]. El argumento `--container-name` especifica el contenedor del que se van a extraer registros. En este ejemplo, extraiga los registros de los contenedores *hamlet* y *juliet*, que muestran una salida del comando diferente:

```azurecli
az container logs \
  --resource-group myResourceGroup \
  --name myContainerGroup \
  --container-name hamlet
```

Salida:

```console
[('HAMLET', 386), ('HORATIO', 127), ('CLAUDIUS', 120)]
```

```azurecli
az container logs \
  --resource-group myResourceGroup \
  --name myContainerGroup \
  --container-name juliet
```

Salida:

```console
[('ROMEO', 177), ('JULIET', 134), ('CAPULET', 119)]
```

## <a name="next-steps"></a>Pasos siguientes

Los contenedores de inicialización ayudan a realizar tareas de instalación e inicialización de los contenedores de la aplicación. Para más información sobre cómo ejecutar contenedores basados en tareas, consulte [Ejecución de tareas en contenedores con directivas de reinicio](container-instances-restart-policy.md).

Azure Container Instances proporciona otras opciones para modificar el comportamiento de los contenedores de la aplicación. Algunos ejemplos son:

* [Establecimiento de variables de entorno en instancias de contenedor](container-instances-environment-variables.md)
* [Establecimiento de la línea de comandos en una instancia de contenedor para invalidar la operación de línea de comandos predeterminada](container-instances-start-command.md)


[az-group-create]: /cli/azure/group#az_group_create
[az-deployment-group-create]: /cli/azure/deployment/group#az_deployment_group_create
[az-container-logs]: /cli/azure/container#az_container_logs
