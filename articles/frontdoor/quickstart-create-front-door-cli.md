---
title: 'Inicio rápido: Configuración de alta disponibilidad con Azure Front Door: CLI de Azure'
description: En este inicio rápido se mostrará cómo usar Azure Front Door para crear una aplicación web global de alto rendimiento y alta disponibilidad mediante la CLI de Azure.
services: front-door
author: duongau
manager: KumudD
ms.assetid: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 4/19/2021
ms.author: duau
ms.openlocfilehash: 3567d5af31b0c7bc2443e3d02426a5bb7aba06f7
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107862010"
---
# <a name="quickstart-create-a-front-door-for-a-highly-available-global-web-application-using-azure-cli"></a>Inicio rápido: Creación de una instancia de Front Door para una aplicación web global de alta disponibilidad mediante la CLI de Azure

El primer paso con Azure Front Door es utilizar la CLI de Azure para crear una aplicación web global de alto rendimiento y alta disponibilidad.

Front Door dirige el tráfico web a recursos concretos de un grupo de back-end. Ha definido el dominio del front-end, agregado recursos a un grupo de back-end y creado una regla de enrutamiento. En este artículo se usa una configuración simple de un grupo de back-end con dos recursos de aplicación web y una regla de enrutamiento única que usa la coincidencia de ruta de acceso predeterminada "/*".

:::image type="content" source="media/quickstart-create-front-door/environment-diagram.png" alt-text="Diagrama del entorno de implementación de Front Door mediante la CLI de Azure." border="false":::

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- CLI de Azure instalada localmente o Azure Cloud Shell
- Asegúrese de que se agrega la extensión front-door a la CLI de Azure.

```azurecli-interactive 
az extension add --name front-door
```

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para esta guía de inicio rápido se necesita la versión 2.0.28 de la CLI de Azure o una versión posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

En Azure, puede asignar recursos relacionados a un grupo de recursos. Puede usar un grupo de recursos existente o crear uno nuevo.

Para este inicio rápido, necesita dos grupos de recursos. Uno en *Centro de EE. UU.* y el segundo en *Centro-sur de EE. UU.* .

Cree un grupo de recursos con [az group create](/cli/azure/group#az_group_create):

```azurecli-interactive
az group create \
    --name myRGFDCentral \
    --location centralus

az group create \
    --name myRGFDEast \
    --location eastus
```

## <a name="create-two-instances-of-a-web-app"></a>Creación de dos instancias de una aplicación web

Para este inicio rápido se necesitan dos instancias de una aplicación web que se ejecutan en regiones diferentes de Azure. Las dos instancias de la aplicación web se ejecutan en modo Activo/Activo, por lo que cualquiera de ellas puede atender el tráfico.

Si aún no tiene una aplicación web, use el siguiente script para configurar dos aplicaciones web de ejemplo.

### <a name="create-app-service-plans"></a>Creación de planes de App Service

Para poder crear las aplicaciones web, necesitará dos planes de App Service, uno en *Centro de EE. UU.* y el segundo en *Este de EE. UU.* .

Cree planes de App Service con [az appservice plan create](/cli/azure/appservice/plan#az_appservice_plan_create&preserve-view=true):

```azurecli-interactive
az appservice plan create \
--name myAppServicePlanCentralUS \
--resource-group myRGFDCentral

az appservice plan create \
--name myAppServicePlanEastUS \
--resource-group myRGFDEast
```

### <a name="create-web-apps"></a>Creación de aplicaciones web

Al ejecutar los siguientes comandos, se creará una aplicación web en cada uno de los planes de App Service del paso anterior. Los nombres de las aplicaciones web tienen que ser únicos globalmente.

Cree una aplicación web con [az webapp create](/cli/azure/webapp#az_webapp_create&preserve-view=true):

```azurecli-interactive
az webapp create \
--name WebAppContoso-1 \
--resource-group myRGFDCentral \
--plan myAppServicePlanCentralUS 

az webapp create \
--name WebAppContoso-2 \
--resource-group myRGFDEast \
--plan myAppServicePlanEastUS
```

Anote el nombre de host predeterminado de cada aplicación web, con el fin de que pueda definir las direcciones de back-end al implementar la instancia de Front Door en el paso siguiente.

## <a name="create-the-front-door"></a>Creación de la instancia de Front Door

Cree una instancia de Front Door básica con la configuración predeterminada del equilibrio de carga, el sondeo de estado y las reglas de enrutamiento mediante la ejecución de lo siguiente:

Cree una instancia de Front Door con [az network front-door create](/cli/azure/network/front-door#az_network_front_door_create&preserve-view=true):

```azurecli-interactive
az network front-door create \
--resource-group myRGFDCentral \
--name contoso-frontend \
--accepted-protocols http https \
--backend-address webappcontoso-1.azurewebsites.net webappcontoso-2.azurewebsites.net 
```

**--resource-group:** especifique el grupo de recursos en el que desea implementar la instancia de Front Door.

**--name:** especifique un nombre globalmente único para la instancia de Azure Front Door. 

**--accepted-protocols:** los valores aceptados son: **http** y **https**. Si desea usar ambos, especifíquelos separados por un espacio.

**--backend-address:** defina aquí el nombre de host de las dos aplicaciones web separados por un espacio.

Una vez que la implementación se haya completado correctamente, anote el nombre de host en la sección *frontEndpoints*.

## <a name="test-the-front-door"></a>Prueba de Front Door

Abra un explorador web y escriba el nombre de host obtenido de los comandos. Front Door dirigirá su solicitud a uno de los recursos de back-end.

:::image type="content" source="./media/quickstart-create-front-door-cli/front-door-testing-page.png" alt-text="Página de pruebas de Front Door":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando deje de necesitar los recursos que ha creado con Front Door, elimine los dos grupos de recursos. Al eliminar el grupo de recursos, también elimina la instancia de Front Door y todos sus recursos relacionados. 

Para eliminar el grupo de recursos, use [az group delete](/cli/azure/group#az_group_delete&preserve-view=true):

```azurecli-interactive
az group delete \
--name myRGFDCentral 

az group delete \
--name myRGFDEast
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado lo siguiente:
* Front Door
* Dos aplicaciones web

Para obtener información sobre cómo agregar un dominio personalizado a Front Door, continúe con los tutoriales de Front Door.

> [!div class="nextstepaction"]
> [Agregar un dominio personalizado](front-door-custom-domain.md)
