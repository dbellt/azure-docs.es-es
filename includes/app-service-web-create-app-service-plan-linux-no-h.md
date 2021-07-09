---
title: "include file" description: "include file" services: app-service author: cephalin ms.service: app-service ms.topic: "include" ms.date: 05/02/2021 ms.author: cephalin ms.custom: "include file", devx-track-azurecli
---

En Cloud Shell, cree un plan de App Service con el comando [`az appservice plan create`](/cli/azure/appservice/plan).

<!-- [!INCLUDE [app-service-plan](app-service-plan.md)] -->

En el siguiente ejemplo se crea un plan de App Service denominado `myAppServicePlan` con el plan de tarifa **Gratis**:

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku FREE --is-linux
```

Cuando se crea el plan de App Service, la CLI de Azure muestra información similar al ejemplo siguiente:

<pre>
{ 
  "freeOfferExpirationTime": null,
  "geoRegion": "West Europe",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "linux",
  "location": "West Europe",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  &lt; JSON data removed for brevity. &gt;
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
} 
</pre>
