---
title: Webhooks para responder a las acciones del registro
description: Aprenda a usar webhooks para desencadenar eventos cuando tengan lugar acciones de inserción o extracción en uno de los repositorios del registro.
ms.topic: article
ms.date: 05/24/2019
ms.openlocfilehash: 4f6fb719f8d9d51429a19616aa5548b32a2687e0
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107773407"
---
# <a name="using-azure-container-registry-webhooks"></a>Webhooks de Azure Container Registry

Un registro de contenedor de Azure almacena y administra imágenes privadas de contenedor de Docker, de una forma similar a la que Docker Hub almacena imágenes públicas. También puede hospedar repositorios para [gráficos de Helm](container-registry-helm-repos.md) (versión preliminar), un formato de empaquetado para implementar aplicaciones en Kubernetes. Puede usar webhooks para desencadenar eventos cuando determinadas acciones tienen lugar en uno de los repositorios de registro. Los webhooks pueden responder a eventos en el nivel de registro o pueden limitarse a una etiqueta de repositorio específica. Con un registro [con replicación geográfica](container-registry-geo-replication.md), puede configurar cada webhook para responder a eventos en una réplica regional específica.

El punto de conexión de un webhook debe ser accesible públicamente desde el registro. Puede configurar solicitudes de webhook del registro para autenticarse en un punto de conexión protegido.

Para obtener información detallada sobre las solicitudes de webhook, consulte [Referencia de esquema de webhook de Azure Container Registry](container-registry-webhook-reference.md).

## <a name="prerequisites"></a>Prerrequisitos

* Registro de contenedor de Azure: cree un registro de contenedor en la suscripción de Azure. Por ejemplo, use [Azure Portal](container-registry-get-started-portal.md) o la [CLI de Azure](container-registry-get-started-azure-cli.md). Los [niveles de servicio de Azure Container Registry](container-registry-skus.md) tienen distintas cuotas de webhooks.
* CLI de docker: para configurar el equipo local como un host de Docker y tener acceso a los comandos de la CLI de Docker, instale [Docker Engine](https://docs.docker.com/engine/installation/).

## <a name="create-webhook---azure-portal"></a>Creación de webhooks: Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya al registro de contenedor en el que desea crear un webhook.
1. En **Servicios**, seleccione **Webhooks**.
1. Seleccione **Agregar** en la barra de herramientas del webhook.
1. Complete el formulario *Crear webhook* con la siguiente información:

| Value | Descripción |
|---|---|
| Nombre de webhook | El nombre que desea dar al webhook. Solo puede contener letras y números, y su longitud debe oscilar entre 5 y 50 caracteres. |
| Location | Para un registro [con replicación geográfica](container-registry-geo-replication.md), especifique la región de Azure de la réplica del registro. 
| URI de servicio | El identificador URI donde el webhook debe enviar notificaciones POST. |
| Encabezados personalizados | Los encabezados que van a pasar junto con la solicitud POST. Deben tener el formato "clave: valor". |
| Acciones de desencadenador | Acciones que desencadenan el webhook. Las acciones incluyen insertar y eliminar imágenes, insertar y eliminar gráficos de Helm, y poner imágenes en cuarentena. Puede elegir una o varias acciones para desencadenar el webhook. |
| Status | El estado del webhook después de su creación. Esto está habilitada de manera predeterminada. |
| Ámbito | El ámbito en el que trabaja el webhook. Si no se especifica, el ámbito sirve para todos los eventos del registro. Se puede especificar para un repositorio o una etiqueta con el formato "repositorio:etiqueta"o "repositorio:*" para todas las etiquetas de un repositorio. |

Formulario de webhook de ejemplo:

![Captura de pantalla que muestra la interfaz de usuario de creación del webhook de ACR en Azure Portal.](./media/container-registry-webhook/webhook.png)

## <a name="create-webhook---azure-cli"></a>Creación de webhooks: CLI de Azure

Para crear un webhook mediante la CLI de Azure, use el comando [az acr webhook create](/cli/azure/acr/webhook#az_acr_webhook_create). El siguiente comando crea un webhook para los eventos de eliminación de imágenes en el registro *mycontainerregistry*:

```azurecli-interactive
az acr webhook create --registry mycontainerregistry --name myacrwebhook01 --actions delete --uri http://webhookuri.com
```

## <a name="test-webhook"></a>Webhook de prueba

### <a name="azure-portal"></a>Azure Portal

Antes de usar el webhook, puede probarlo con el botón **Ping**. El comando Ping envía una solicitud POST genérica al punto de conexión especificado y registra la respuesta. Mediante la característica de ping puede comprobar que ha configurado correctamente el webhook.

1. Seleccione el webhook que desea probar.
2. En la barra de herramientas superior, seleccione **Ping**.
3. Compruebe la respuesta del punto de conexión en la columna **ESTADO HTTP**.

![Interfaz de usuario de creación de webhook ACR en Azure Portal](./media/container-registry-webhook/webhook-02.png)

### <a name="azure-cli"></a>Azure CLI

Para probar un webhook de ACR con la CLI de Azure, use el comando [az acr webhook ping](/cli/azure/acr/webhook#az_acr_webhook_ping).

```azurecli-interactive
az acr webhook ping --registry mycontainerregistry --name myacrwebhook01
```

Para ver los resultados, use el comando [az acr webhook list-events](/cli/azure/acr/webhook).

```azurecli-interactive
az acr webhook list-events --registry mycontainerregistry08 --name myacrwebhook01
```

## <a name="delete-webhook"></a>Eliminación del webhook

### <a name="azure-portal"></a>Azure Portal

Se puede eliminar cada webhook seleccionando uno y, después, con el botón **Eliminar** en Azure Portal.

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az acr webhook delete --registry mycontainerregistry --name myacrwebhook01
```

## <a name="next-steps"></a>Pasos siguientes

### <a name="webhook-schema-reference"></a>Referencia del esquema de webhook

Para más información sobre el formato y las propiedades de las cargas del evento JSON emitidas por Azure Container Registry, consulte la referencia del esquema de webhook:

[Referencia de esquema de webhook de Azure Container Registry](container-registry-webhook-reference.md)

### <a name="event-grid-events"></a>Eventos de Event Grid

Además de los eventos de webhook de registro nativos descritos en este artículo, Azure Container Registry puede emitir eventos a Event Grid:

[Inicio rápido: Envío de eventos del registro de contenedor a Event Grid](container-registry-event-grid-quickstart.md)
