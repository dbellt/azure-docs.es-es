---
title: 'Inicio rápido: Su primera consulta de API REST'
description: En este inicio rápido, dará los pasos necesarios para llamar al punto de conexión de Resource Graph para la API REST y ejecutará su primera consulta.
ms.date: 05/01/2021
ms.topic: quickstart
ms.openlocfilehash: ef157eebbdd562262bb33f32e726d0d567c421d5
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108324948"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-rest-api"></a>Inicio rápido: Ejecución de la primera consulta de Resource Graph mediante la API REST

El primer paso para usar Azure Resource Graph con la API REST es comprobar que tiene a su disposición alguna herramienta para llamar a las API REST. Este inicio rápido le guía por el proceso de ejecutar una consulta y recuperar los resultados mediante una llamada al punto de conexión de la API REST de Azure Resource Graph.

Al final de este proceso, tendrá las herramientas necesarias para llamar a los puntos de conexión de la API REST y ejecutar la primera consulta de Resource Graph.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="getting-started-with-rest-api"></a>Introducción a la API REST

Si no está familiarizado con la API REST, comience por revisar la [Referencia de la API REST de Azure](/rest/api/azure/) para obtener una descripción general de la API REST, específicamente el identificador URI de la solicitud y el cuerpo de la solicitud. En este artículo se utilizan estos conceptos para proporcionar instrucciones para trabajar con Azure Resource Graph y se da por supuesto un conocimiento práctico del mismo. Herramientas como [ARMClient](https://github.com/projectkudu/ARMClient) y otras pueden controlar la autorización automáticamente y se recomiendan para principiantes.

Para conocer las especificaciones de Azure Resource Graph, consulte [API REST de Azure Resource Graph](/rest/api/azure-resourcegraph/).

### <a name="rest-api-and-powershell"></a>API REST y PowerShell

Si aún no tiene una herramienta para realizar llamadas de API REST, considere el uso de PowerShell para estas instrucciones. El siguiente código de ejemplo obtiene un encabezado para realizar la autenticación con Azure. Genere un encabezado de autenticación, a veces llamado **Token de portador**, y proporcione el identificador URI de la API REST al que conectarse con cualquier parámetro o un **cuerpo de la solicitud**:

```azurepowershell-interactive
# Log in first with Connect-AzAccount if not using Cloud Shell

$azContext = Get-AzContext
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azProfile)
$token = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)
$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $token.AccessToken
}

# Invoke the REST API
$restUri = 'https://management.azure.com/subscriptions/{subscriptionId}?api-version=2020-01-01'
$response = Invoke-RestMethod -Uri $restUri -Method Get -Headers $authHeader
```

Reemplace `{subscriptionId}` en la variable `$restUri` para obtener información de la suscripción.
La variable `$response`contiene el resultado del cmdlet `Invoke-RestMethod`, que puede analizarse con cmdlets como [ConvertFrom-Json](/powershell/module/microsoft.powershell.utility/convertfrom-json). Si el punto de conexión de servicio de la API REST espera un **cuerpo de la solicitud**, proporcione una variable con formato JSON al parámetro `-Body` de `Invoke-RestMethod`.

## <a name="run-your-first-resource-graph-query"></a>Ejecutar la primera consulta de Resource Graph

Una vez agregadas las herramientas de API REST al entorno elegido, es el momento de probar una consulta simple de Resource Graph. La consulta devolverá los cinco primeros recursos de Azure con el **nombre** y el **tipo de recurso** de cada recurso.

En el cuerpo de la solicitud de cada llamada a la API REST, hay una variable que se usa que debe reemplazar por su propio valor:

- `{subscriptionID}`: reemplácelo por el identificador de suscripción

1. Ejecute la primera consulta de Azure Resource Graph mediante la API REST y el punto de conexión `resources`:

   - URI DE LA API REST

     ```http
     POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2019-04-01
     ```

   - Cuerpo de la solicitud

     ```json
     {
         "subscriptions": [
             "{subscriptionID}"
         ],
         "query": "Resources | project name, type | limit 5"
     }
     ```

   > [!NOTE]
   > Como esta consulta de ejemplo no proporciona un modificador de ordenación como `order by`, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud.

1. Actualice la llamada al punto de conexión `resouces` y cambie el valor de **query** a ordenanar por nombre de propiedad (`order by` **Name**):

   - URI DE LA API REST

     ```http
     POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2019-04-01
     ```

   - Cuerpo de la solicitud

     ```json
     {
         "subscriptions": [
             "{subscriptionID}"
         ],
         "query": "Resources | project name, type | limit 5 | order by name asc"
     }
     ```

   > [!NOTE]
   > Al igual que con la primera consulta, es probable que al ejecutar esta consulta varias veces se produzca un conjunto diferente de recursos por solicitud. El orden de los comandos de consulta es importante. En este ejemplo, el `order by` viene después del `limit`. Este orden de comandos limita primero los resultados de la consulta y, luego, los ordena.

1. Actualice la llamada al punto de conexión `resources` y, primero, cambie el valor de **query** a ordenar por nombre (`order by`**Name**) y, después, ordenar por `limit` los cinco primeros resultados:

   - URI DE LA API REST

     ```http
     POST https://management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2019-04-01
     ```

   - Cuerpo de la solicitud

     ```json
     {
         "subscriptions": [
             "{subscriptionID}"
         ],
         "query": "Resources | project name, type | order by name asc | limit 5"
     }
     ```

Cuando la consulta final se ejecuta varias veces, suponiendo que nada cambie en su entorno, los resultados devueltos serán coherentes y estarán ordenados por la propiedad **Nombre**, pero todavía limitados a los cinco primeros resultados.

Para más ejemplos de llamadas de la API REST para Azure Resource Graph, consulte la sección de [ejemplos de REST de Azure Resource Graph](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources#examples).

## <a name="clean-up-resources"></a>Limpieza de recursos

La API REST no tiene bibliotecas ni módulos que se desinstalen. Si instaló una herramienta como _ARMClient_ o _Postman_ para realizar las llamadas y ya no la necesita, puede desinstalarla ahora mismo si lo desea.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha llamado al punto de conexión de la API REST de Resource Graph y ha ejecutado la primera consulta. Para más información sobre el lenguaje de Resource Graph, vaya a la página de detalles del lenguaje de consulta.

> [!div class="nextstepaction"]
> [Obtener más información sobre el lenguaje de consulta](./concepts/query-language.md)
