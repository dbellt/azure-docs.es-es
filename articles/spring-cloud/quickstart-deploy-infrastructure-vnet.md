---
title: 'Inicio rápido: Aprovisionamiento de Azure Spring Cloud con una plantilla de Azure Resource Manager (plantilla de ARM)'
description: En este inicio rápido se muestra cómo implementar un clúster de Spring Cloud en una red virtual existente.
services: azure-resource-manager
author: ryhud
ms.service: spring-cloud
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-java
ms.author: rhudson
ms.date: 05/27/2021
ms.openlocfilehash: f57e3d6d1f3f562e35d38e45a3e0ca1703730d04
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111949218"
---
# <a name="quickstart-provision-azure-spring-cloud-using-an-arm-template"></a>Inicio rápido: Aprovisionamiento de Azure Spring Cloud con una plantilla de ARM

En este inicio rápido se describe cómo usar una plantilla de Azure Resource Manager (plantilla de ARM) para implementar un clúster de Azure Spring Cloud en una red virtual existente.

Azure Spring Cloud facilita la implementación de aplicaciones de microservicio de Spring Boot en Azure sin necesidad de realizar cambios en el código. El servicio administra la infraestructura de las aplicaciones de Spring Cloud, con el fin de que los desarrolladores puedan centrarse en el código. Azure Spring Cloud proporciona administración del ciclo de vida mediante el uso de una supervisión y un diagnóstico completos, administración de la configuración, detección de servicios, integración de CI/CD e implementaciones blue-green, entre otros.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-spring-cloud-reference-architecture%2Fmain%2FARM%2Fbrownfield-deployment%2fazuredeploy.json)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción, cree [una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
* Dos subredes dedicadas para el clúster de Azure Spring Cloud, una para el entorno de ejecución del servicio y otra para las aplicaciones de microservicios de Spring Boot. Puede encontrar los requisitos de la subred y la red virtual en la sección [Requisitos de red virtual](how-to-deploy-in-azure-virtual-network.md#virtual-network-requirements) de [Implementación de Azure Spring Cloud en una red virtual](how-to-deploy-in-azure-virtual-network.md).
* Un área de trabajo de Log Analytics existente para la configuración de diagnóstico de Azure Spring Cloud y un recurso de Application Insights basado en el área de trabajo. Para más información, consulte [Análisis de registros y métricas con la configuración de diagnóstico](diagnostic-services.md) y [Agente In-Process de Java de Application Insights en Azure Spring Cloud (versión preliminar)](how-to-application-insights.md).
* Tres intervalos internos de enrutamiento de interdominios sin clases (CIDR) (al menos */16* cada uno) que ha identificado para su uso por parte del clúster de Azure Spring Cloud. Estos intervalos CIDR no se podrán enrutar directamente y solo los usará internamente el clúster de Azure Spring Cloud. Los clústeres no pueden usar *169.254.0.0/16*, *172.30.0.0/16,* *172.31.0.0/16* o  *192.0.2.0/24* en los intervalos internos CIDR de Spring Cloud ni los intervalos IP incluidos dentro del intervalo de direcciones de red virtual del clúster.
* Permiso de servicio concedido a la red virtual. El proveedor de recursos de Azure Spring Cloud requiere permisos de propietario en la red virtual para conceder permisos a una entidad de servicio dinámica y dedicada en la red virtual para realizar tareas de implementación y mantenimiento adicionales. Puede encontrar instrucciones y más información en la sección [Concesión de permisos de servicio a la red virtual](how-to-deploy-in-azure-virtual-network.md#grant-service-permission-to-the-virtual-network) de [Implementación de Azure Spring Cloud en una red virtual](how-to-deploy-in-azure-virtual-network.md).
* Si usa Azure Firewall o una aplicación virtual de red (NVA), también deberá cumplir los siguientes requisitos previos:
   * Reglas de nombre de dominio completo (FQDN) y de red. Para más información, consulte [Requisitos de red virtual](how-to-deploy-in-azure-virtual-network.md#virtual-network-requirements).
   * Aplicación de una ruta definida por el usuario (UDR) única a cada uno de los entornos de ejecución del servicio y subredes de aplicación de microservicios de Spring Boot. Para más información sobre las UDR, consulte [Enrutamiento del tráfico de redes virtuales](../virtual-network/virtual-networks-udr-overview.md). La UDR debe configurarse con una ruta de *0.0.0.0/0* con un destino de la NVA antes de implementar el clúster de Spring Cloud. Para más información, consulte la sección [Traer su propia tabla de rutas](how-to-deploy-in-azure-virtual-network.md#bring-your-own-route-table) de [Implementación de Azure Spring Cloud en una red virtual](how-to-deploy-in-azure-virtual-network.md).

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido es de la [arquitectura de referencia de Azure Spring Cloud](reference-architecture.md).

:::code language="json" source="~/azure-spring-cloud-reference-architecture/ARM/brownfield-deployment/azuredeploy.json":::

En la plantilla se definen dos recursos de Azure:

* [Microsoft.AppPlatform/Spring](/azure/templates/microsoft.appplatform/spring): para crear una instancia de Azure Spring Cloud.
* [Microsoft.Insights/components](/azure/templates/microsoft.insights/components): para crear un área de trabajo de Application Insights.

En el caso de las implementaciones con la CLI de Azure y Terraform, consulte el repositorio de [arquitectura de referencia de Azure Spring Cloud](https://github.com/Azure/azure-spring-cloud-reference-architecture) en GitHub.

## <a name="deploy-the-template"></a>Implementación de la plantilla

Para implementar la plantilla, siga estos pasos:

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla. La plantilla crea una instancia de Azure Spring Cloud en una instancia de Virtual Network existente y una instancia de Application Insights basada en área de trabajo en un área de trabajo existente de Log Analytics de Azure Monitor.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-spring-cloud-reference-architecture%2Fmain%2FARM%2Fbrownfield-deployment%2fazuredeploy.json)

2. Escriba valores para los campos siguientes:

- **Grupo de recursos**: seleccione **Crear nuevo**, escriba un nombre único para el **grupo de recursos** y, después, seleccione **Aceptar**.
- **springCloudInstanceName:** escriba el nombre del recurso de Azure Spring Cloud.
- **appInsightsName:** escriba el nombre de la instancia de Application Insights para Azure Spring Cloud.
- **laWorkspaceResourceId:** escriba el identificador de recurso del área de trabajo de Log Analytics existente (por ejemplo, */subscriptions/<your subscription> /resourcegroups/<your log analytics resource group>/providers/Microsoft.OperationalInsights/workspaces/<your log analytics workspace name>* ).
- **springCloudAppSubnetID:** escriba el identificador de recurso de la subred de aplicación de Azure Spring Cloud.
- **springCloudRuntimeSubnetID:** escriba el identificador de recurso de la subred del entorno de ejecución de Azure Spring Cloud.
- **springCloudServiceCidrs:** escriba una lista separada por comas de intervalos de direcciones IP (3 en total) en formato CIDR. Los intervalos IP se reservan para hospedar la infraestructura de Azure Spring Cloud subyacente. Estos tres intervalos deben ser al menos */16* intervalos IP sin usar y no deben superponerse con los intervalos IP de subred enrutables usados dentro de la red.
- **tags:** escriba cualquier etiqueta personalizada.

3. Seleccione **Revisar y crear** y, a continuación, **Crear**.

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Puede usar Azure Portal para comprobar los recursos implementados, o bien usar un script de la CLI de Azure o Azure PowerShell script para enumerar los recursos implementados.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si planea seguir trabajando en otros inicios rápidos y tutoriales, considere la posibilidad de dejar estos recursos activos. Cuando ya no lo necesite, elimine el grupo de recursos, que elimina los recursos que contiene. Para eliminar el grupo de recursos mediante la CLI de Azure o Azure PowerShell, use estos comandos:

# <a name="cli"></a>[CLI](#tab/azure-cli)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```
---

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado una instancia de Azure Spring Cloud en una red virtual existente mediante una plantilla de ARM y, luego, ha validado la implementación. Para más información sobre Azure Spring Cloud y Azure Resource Manager, continúe con los artículos siguientes.

- Implemente una de las siguientes aplicaciones de ejemplo desde las ubicaciones siguientes:
   * [Aplicación Pet Clinic con integración de MySQL](https://github.com/azure-samples/spring-petclinic-microservices) (microservicios con back-end de MySQL).
   * [Simple Hello World](./quickstart.md?pivots=programming-language-java&tabs=Azure-CLI).
- Use [dominios personalizados](tutorial-custom-domain.md) con Azure Spring Cloud.
- Exponga aplicaciones de Azure Spring Cloud en Internet mediante [Azure Application Gateway](expose-apps-gateway-azure-firewall.md).
- Vea la [arquitectura de referencia de Azure Spring Cloud](reference-architecture.md) completa, que se basa en [Microsoft Azure Well-Architected Framework](/azure/architecture/framework/).
- Obtenga más información sobre [Administrador de recursos de Azure](../azure-resource-manager/management/overview.md).