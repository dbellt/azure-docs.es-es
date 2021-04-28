---
title: Configuración del nivel PremiumV3
description: Obtenga información sobre cómo mejorar el rendimiento para su aplicación web, móvil y de API en Azure App Service mediante el escalado al nuevo plan de tarifa Prémium v3.
keywords: servicio de aplicaciones, servicio de aplicaciones de azure, escala, escalable, plan del servicio de aplicaciones, costo del servicio de aplicaciones
ms.assetid: ff00902b-9858-4bee-ab95-d3406018c688
ms.topic: article
ms.date: 10/01/2020
ms.custom: seodec18, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 8fbd841626e2a074bc0a35cd1b4ac094e267b34a
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107833353"
---
# <a name="configure-premiumv3-tier-for-azure-app-service"></a>Configuración del nivel Prémium v3 para Azure App Service

Con el nuevo plan de tarifa **Prémium v3**, se obtienen procesadores más rápidos, almacenamiento SSD y el cuádruple de la relación memoria-núcleo que con los planes de tarifa existentes (el doble con respecto al plan **Prémium v2**). Con la ventaja de rendimiento, puede ahorrar dinero mediante la ejecución de las aplicaciones en menos instancias. En este artículo, aprenderá a crear una aplicación en el nivel **Prémium v3** o a escalar verticalmente una aplicación al nivel **Prémium v3**.

## <a name="prerequisites"></a>Requisitos previos

Para escalar verticalmente una aplicación a **Prémium v3**, debe tener una aplicación de Azure App Service que se ejecute en un plan de tarifa inferior a **Prémium v3**, y esta aplicación debe ejecutarse en una implementación de App Service que admita Prémium v3.

<a name="availability"></a>

## <a name="premiumv3-availability"></a>Disponibilidad de Prémium v3

El plan **Prémium V3** está disponible para aplicaciones nativas y de contenedor, incluidos los contenedores de Windows y Linux.

> [!NOTE]
> Todos los contenedores de Windows que se ejecutan en el plan de **contenedores prémium** durante el período de la versión preliminar seguirán funcionando tal cual, pero el plan de **contenedores prémium** continuará en versión preliminar. El plan **Prémium v3** es el sustituto oficial del plan de **contenedores prémium**. 

**Prémium v3** está disponible en algunas regiones de Azure, y la disponibilidad en regiones adicionales se agregará continuamente. Para ver si está disponible en su región, ejecute el siguiente comando de la CLI de Azure en [Azure Cloud Shell](../cloud-shell/overview.md):

```azurecli-interactive
az appservice list-locations --sku P1V3
```

<a name="create"></a>

## <a name="create-an-app-in-premiumv3-tier"></a>Creación de una aplicación en el plan Prémium v3

El plan de tarifa de una aplicación de App Service se define en el [plan de App Service](overview-hosting-plans.md) en el que se ejecuta. Puede crear un plan de App Service de forma independiente o durante el proceso de creación de la aplicación.

Cuando configure el plan de App Service en <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, seleccione **Plan de tarifa**. 

Seleccione **Producción**, después **P1V3**, **P2V3**, o **P3V3** y, luego, haga clic en **Aplicar**.

![Captura de pantalla que muestra los planes de tarifa recomendados para la aplicación.](media/app-service-configure-premium-tier/scale-up-tier-select.png)

> [!IMPORTANT] 
> Si no ve **P1V3**, **P2V3** y **P3V3** como opciones, o si las opciones están atenuadas, es probable que **Prémium v3** no esté disponible en la implementación subyacente de App Service que contiene el plan de App Service. Vea [Escalado vertical desde una combinación de región y grupos de recursos no admitidos](#unsupported) para obtener más información.

## <a name="scale-up-an-existing-app-to-premiumv3-tier"></a>Escalado vertical de una aplicación existente al plan Prémium v3

Antes de escalar una aplicación existente al plan **Prémium v3**, asegúrese de que **Prémium v3** está disponible. Para información, vea [Disponibilidad del nivel Prémium v3](#availability). Si no está disponible en su región, ve [Escalado vertical desde una combinación de región y grupos de recursos no admitidos](#unsupported).

Dependiendo del entorno de hospedaje, el escalado vertical puede requerir pasos adicionales. 

En <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, abra la página de la aplicación de App Service.

En el panel izquierdo de navegación de la página de la aplicación de App Service, seleccione **Escalar verticalmente (plan de App Service)** .

![Captura de pantalla que muestra cómo escalar verticalmente el plan de App Service.](media/app-service-configure-premium-tier/scale-up-tier-portal.png)

Seleccione **Producción**, después **P1V3**, **P2V3**, o **P3V3** y, luego, haga clic en **Aplicar**.

![Captura de pantalla que muestra los planes de tarifa recomendados para la aplicación.](media/app-service-configure-premium-tier/scale-up-tier-select.png)

Si la operación finaliza correctamente, la página de información general de la aplicación muestra que ahora está en un plan **Prémium v3**.

![Captura de pantalla que muestra el plan de tarifa Prémium v3 en la página de información general de la aplicación.](media/app-service-configure-premium-tier/finished.png)

### <a name="if-you-get-an-error"></a>Si obtiene un mensaje de error

Algunos planes de App Service no se pueden escalar verticalmente al nivel Prémium v3 si la implementación de App Service subyacente no admite Prémium v3. Vea [Escalado vertical desde una combinación de región y grupos de recursos no admitidos](#unsupported) para obtener más información.

<a name="unsupported"></a>

## <a name="scale-up-from-an-unsupported-resource-group-and-region-combination"></a>Escalado vertical desde una combinación de región y grupos de recursos no admitidos

Si la aplicación se ejecuta en una implementación de App Service en la que **Prémium v3** no está disponible o se ejecuta en una región que actualmente no admite **Prémium v3**, tendrá que volver a implementar la aplicación para beneficiarse de **Prémium v3**.  Tiene dos opciones:

- Cree una aplicación en un nuevo grupo de recursos y con un plan de App Service. Al crear el plan de App Service, seleccione un plan **Prémium v3**. Este paso garantiza que el plan de App Service se implemente en una unidad de implementación que admita **Prémium v3**. Luego, vuelva a implementar el código de aplicación en la aplicación recién creada. Aunque reduzca verticalmente el plan de App Service a un plan inferior para ahorrar costos, siempre podrá volver a escalarlo verticalmente a **Prémium v3** porque la unidad de implementación lo admite.
- Si la aplicación ya se ejecuta en un plan **Prémium** existente, puede clonar la aplicación con toda la configuración de la aplicación, las cadenas de conexión y la configuración de implementación en un nuevo grupo de recursos de un nuevo plan de App Service que utilice **Prémium v3**.

    ![Captura de pantalla en que se muestra cómo clonar la aplicación.](media/app-service-configure-premium-tier/clone-app.png)

    En la página **Clonar aplicación**, puede crear un plan de App Service con **Prémium v3** en la región que prefiera y especificar la configuración que quiera clonar.

## <a name="moving-from-premium-container-to-premium-v3-sku"></a>Traslado de SKU de contenedores prémium a Prémium v3

Si tiene una aplicación que usa la SKU de contenedores prémium en versión preliminar y desea cambiar a la nueva SKU Prémium v3, debe volver a implementar la aplicación para aprovechar las ventajas de **Prémium v3**. Para ello, vea la primera opción [Escalado vertical desde una combinación de región y grupos de recursos no admitidos](#scale-up-from-an-unsupported-resource-group-and-region-combination).

## <a name="automate-with-scripts"></a>Automatizar con scripts

Puede automatizar la creación de la aplicación en el plan **Prémium v3** con scripts mediante la [CLI de Azure](/cli/azure/install-azure-cli) o [Azure PowerShell](/powershell/azure/).

### <a name="azure-cli"></a>Azure CLI

El siguiente comando crea un plan de App Service en _P1V3_. También puede ejecutarlo en Cloud Shell. Las opciones para `--sku` son P1V3, _P2V3_ y _P3V3_.

```azurecli-interactive
az appservice plan create \
    --resource-group <resource_group_name> \
    --name <app_service_plan_name> \
    --sku P1V3
```

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

El siguiente comando crea un plan de App Service en _P1V3_. Las opciones para `-WorkerSize` son _Pequeño_, _Medio_ y _Grande_.

```powershell
New-AzAppServicePlan -ResourceGroupName <resource_group_name> `
    -Name <app_service_plan_name> `
    -Location <region_name> `
    -Tier "PremiumV3" `
    -WorkerSize "Small"
```

## <a name="more-resources"></a>Más recursos

[Escalado vertical de una aplicación en Azure](manage-scale-up.md)
[Escalado automático o manual del recuento de instancias](../azure-monitor/autoscale/autoscale-get-started.md)
