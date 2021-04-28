---
title: Restricciones de acceso de Azure App Service
description: Aprenda a proteger la aplicación en Azure App Service configurando restricciones de acceso.
author: ccompy
ms.assetid: 3be1f4bd-8a81-4565-8a56-528c037b24bd
ms.topic: article
ms.date: 12/17/2020
ms.author: ccompy
ms.custom: seodec18, devx-track-azurepowershell
ms.openlocfilehash: 541af6d0051d06de5721b22616fbf1e2867b71d6
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107833371"
---
# <a name="set-up-azure-app-service-access-restrictions"></a>Configuración de las restricciones de acceso de Azure App Service

Al configurar las restricciones de acceso, puede definir una lista ordenada por prioridad de elementos permitidos o denegados con la que se controla el acceso de red a la aplicación. La lista puede incluir direcciones IP o subredes de Azure Virtual Network. Si hay una o varias entradas, existe un valor *denegar todo* implícito al final de la lista.

La capacidad de restricción de acceso funciona con todas las cargas de trabajo hospedadas por Azure App Service. Las cargas de trabajo pueden incluir aplicaciones web, aplicaciones de API, aplicaciones Linux, aplicaciones de contenedor de Linux e instancias de Functions.

Cuando se realiza una solicitud a la aplicación, la dirección del remitente se evalúa con respecto a las reglas de la lista de restricción de acceso. Si la dirección del remitente está en una subred configurada con puntos de conexión de servicio para Microsoft.Web, la subred de origen se compara con las reglas de red virtual de la lista de restricción de acceso. Si la dirección no tiene permitido el acceso según las reglas de la lista, el servicio responde con un código de estado [HTTP 403](https://en.wikipedia.org/wiki/HTTP_403).

La capacidad de restricción de acceso se implementa en los roles de front-end de App Service, que están en un nivel superior de los hosts de trabajo donde se ejecuta el código. Por lo tanto, las restricciones de acceso son listas de control de acceso de red de facto.

La capacidad de restringir el acceso a la aplicación web desde una instancia de Azure Virtual Network (red virtual) se obtiene mediante [puntos de conexión de servicio][serviceendpoints]. Con los puntos de conexión de servicio, puede restringir el acceso a un servicio multiinquilino desde subredes seleccionadas. No funciona para restringir el tráfico a aplicaciones hospedadas en un entorno App Service Environment. Si está en un entorno App Service Environment, puede controlar el acceso a la aplicación con reglas de direcciones IP.

> [!NOTE]
> Los puntos de conexión de servicio se deben habilitar tanto en el lado de la red como en el servicio de Azure con el que se están habilitando. Para obtener una lista de los servicios de Azure que admiten puntos de conexión de servicio, consulte [Puntos de conexión de servicio de red virtual](../virtual-network/virtual-network-service-endpoints-overview.md).
>

:::image type="content" source="media/app-service-ip-restrictions/access-restrictions-flow.png" alt-text="Diagrama del flujo de restricciones de acceso.":::

## <a name="manage-access-restriction-rules-in-the-portal"></a>Administración de reglas de restricción de acceso en el portal

Para agregar una regla de restricción de acceso a la aplicación, haga lo siguiente:

1. Inicie sesión en Azure Portal.

1. En el panel izquierdo, seleccione **Redes**.

1. En el panel **Redes**, bajo **Restricciones de acceso**, seleccione **Configurar restricciones de acceso**.

    :::image type="content" source="media/app-service-ip-restrictions/access-restrictions.png" alt-text="Captura de pantalla del panel de opciones de redes de App Service en Azure Portal.":::

1. En la página **Restricciones de acceso**, revise la lista de reglas de restricción de acceso definidas para la aplicación.

   :::image type="content" source="media/app-service-ip-restrictions/access-restrictions-browse.png" alt-text="Captura de pantalla de la página Restricciones de acceso de Azure Portal que muestra la lista de reglas de restricción de acceso definidas para la aplicación seleccionada.":::

   La lista muestra todas las restricciones actuales referentes a la aplicación. Si tiene una restricción de red virtual en la aplicación, la tabla muestra si los puntos de conexión de servicio están habilitados para Microsoft.Web. Si no hay restricciones definidas en la aplicación, se puede acceder a la aplicación desde cualquier lugar.

### <a name="add-an-access-restriction-rule"></a>Incorporación de una regla de restricción de acceso

Para agregar una regla de restricción de acceso a la aplicación, en el panel **Restricciones de acceso**, seleccione **Agregar regla**. Una vez que agregue una regla, entrará en vigor de inmediato. 

Las reglas se aplican en orden de prioridad, empezando por el número más bajo en la columna **Prioridad**. Una *denegación de todo* implícita entra en vigor una vez que agregue incluso una sola regla.

En el panel **Agregar restricción de acceso**, cuando cree una regla, haga lo siguiente:

1. En **Acción**, seleccione **Permitir** o **Denegar**.  

   :::image type="content" source="media/app-service-ip-restrictions/access-restrictions-ip-add.png?v2" alt-text="Captura de pantalla del panel &quot;Agregar restricción de acceso&quot;.":::

1. Opcionalmente, escriba un nombre y una descripción para la regla.
1. En el cuadro **Prioridad**, escriba un valor de prioridad.
1. En la lista desplegable **Tipo**, seleccione el tipo de regla.

En las siguientes secciones se describen los distintos tipos de reglas.

> [!NOTE]
> - Hay un límite de 512 reglas de restricción de acceso. Si necesita más de 512 reglas de restricción de acceso, se sugiere instalar un producto de seguridad independiente, como Azure Front Door, Azure Application Gateway o un WAF alternativo.
>
#### <a name="set-an-ip-address-based-rule"></a>Establecimiento de una regla basada en direcciones IP

Siga el procedimiento descrito en la sección anterior, pero con la siguiente adición:
* En el paso 4, en la lista desplegable **Tipo**, seleccione **IPv4** o **IPv6**. 

Especifique el **Bloque de direcciones IP** en la notación Enrutamiento de interdominios sin clases (CIDR) para las direcciones IPv4 e iPv6. Para especificar una dirección, puede usar un formato como *1.2.3.4/32*, donde los cuatro primeros octetos representan la dirección IP y */32* es la máscara. La notación CIDR IPv4 para todas las direcciones es 0.0.0.0/0. Para más información acerca de la notación CIDR, consulte [Enrutamiento de interdominios sin clases](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing). 

#### <a name="set-a-service-endpoint-based-rule"></a>Establecimiento de una regla basada en un punto de conexión de servicio

* En el paso 4, en la lista desplegable **Tipo**, seleccione **Red virtual**.

   :::image type="content" source="media/app-service-ip-restrictions/access-restrictions-vnet-add.png?v2" alt-text="Captura de pantalla del panel &quot;Agregar restricción&quot; con el tipo Red virtual seleccionado.":::

En las listas desplegables **Suscripción**, **Red virtual** y **Subred**, especifique a qué quiere restringir el acceso.

Con los puntos de conexión de servicio, puede restringir el acceso a las subredes de Azure Virtual Network. Si los puntos de conexión de servicio ya no están habilitados con Microsoft.Web para la subred seleccionada, se habilitarán automáticamente a menos que active la casilla **Omitir los puntos de conexión de servicio de Microsoft.Web que faltan**. La situación en la que convendría habilitar puntos de conexión de servicio en la aplicación pero no en la subred depende fundamentalmente de si se tienen los permisos para habilitarlos en la subred. 

Si necesita que otra persona habilite los puntos de conexión de servicio en la subred, active la casilla **Omitir los puntos de conexión de servicio de Microsoft.Web que faltan**. La aplicación se configurará para los puntos de conexión de servicio en previsión de que se habiliten posteriormente en la subred. 

No puede usar puntos de conexión de servicio para restringir el acceso a las aplicaciones que se ejecutan en un entorno App Service Environment. Si la aplicación está en un entorno App Service Environment, puede controlar el acceso a ella con reglas de acceso de IP. 

Con puntos de conexión de servicio, la aplicación se puede configurar con puertas de enlace de aplicación u otros dispositivos de firewall de aplicaciones web (WAF). También se pueden configurar aplicaciones de varios niveles con back-ends seguros. Para más información, lea [Características de redes de App Service](networking-features.md) e [Integración de Application Gateway con puntos de conexión de servicio](networking/app-gateway-with-service-endpoints.md).

> [!NOTE]
> - Actualmente no se admiten puntos de conexión de servicio para las aplicaciones web que usan IP virtual (VIP) con Capa de sockets seguros (SSL) de IP.
>
#### <a name="set-a-service-tag-based-rule"></a>Establecimiento de una regla basada en una etiqueta de servicio

* En el paso 4, en la lista desplegable **Tipo**, seleccione **Etiqueta de servicio**.

   :::image type="content" source="media/app-service-ip-restrictions/access-restrictions-service-tag-add.png?v2" alt-text="Captura de pantalla del panel &quot;Agregar restricción&quot; con el tipo Etiqueta de servicio seleccionado.":::

Cada etiqueta de servicio representa una lista de intervalos IP de servicios de Azure. Puede encontrar una lista de estos servicios y vínculos a los intervalos específicos en la [documentación de la etiqueta de servicio][servicetags].

Todas las etiquetas de servicio disponibles se admiten en las reglas de restricción de acceso. Para simplificar, solo hay disponible una lista de las etiquetas más comunes a través de Azure Portal. Use plantillas de Azure Resource Manager o scripting para configurar reglas más avanzadas, como las reglas de ámbito regional. Estas son las etiquetas disponibles en Azure Portal:

* ActionGroup
* ApplicationInsightsAvailability
* AzureCloud
* AzureCognitiveSearch
* AzureEventGrid
* AzureFrontDoor.Backend
* AzureMachineLearning
* AzureTrafficManager
* LogicApps

### <a name="edit-a-rule"></a>Edición de una regla

1. Para empezar a editar una regla de restricción de acceso existente, en la página **Restricciones de acceso**, seleccione la regla que quiere editar.

1. En el panel **Editar restricción de acceso**, realice los cambios y seleccione **Actualizar regla**. Las modificaciones son efectivas inmediatamente, incluidos los cambios en el orden de prioridad.

   :::image type="content" source="media/app-service-ip-restrictions/access-restrictions-ip-edit.png?v2" alt-text="Captura de pantalla del panel &quot;Editar restricción de acceso&quot; de Azure Portal que muestra los campos de una regla de restricción de acceso existente.":::

   > [!NOTE]
   > Cuando se edita una regla, no se puede cambiar entre tipos de regla. 

### <a name="delete-a-rule"></a>Eliminar una regla

Para eliminar una regla, en la página **Restricciones de acceso**, seleccione los puntos suspensivos ( **...** ) junto a la regla que desea eliminar y luego seleccione **Quitar**.

:::image type="content" source="media/app-service-ip-restrictions/access-restrictions-delete.png" alt-text="Captura de pantalla de la página &quot;Restricciones de acceso&quot; que muestra los puntos suspensivos &quot;Quitar&quot; junto a la regla de restricción de acceso que se va a eliminar.":::

## <a name="access-restriction-advanced-scenarios"></a>Escenarios avanzados de restricción de acceso
En las secciones siguientes se describen algunos escenarios avanzados que usan restricciones de acceso.

### <a name="filter-by-http-header"></a>Filtrar por encabezado http

Como parte de cualquier regla, puede agregar filtros de encabezados HTTP adicionales. Se admiten los siguientes nombres de encabezados http:
* X-Forwarded-For
* X-Forwarded-Host
* X-Azure-FDID
* X-FD-HealthProbe

Para cada nombre de encabezado, puede agregar hasta 8 valores separados por comas. Los filtros de encabezados http se evalúan después de la propia regla y deben cumplirse las dos condiciones para que se aplique la regla.

### <a name="multi-source-rules"></a>Reglas de varios orígenes

Las reglas de varios orígenes permiten combinar hasta 8 intervalos IP u 8 etiquetas de servicio en una sola regla. Puede utilizarlo si tiene más de 512 intervalos IP o si desea crear reglas lógicas en las que varios intervalos IP se combinen con un solo filtro de encabezado http.

Las reglas de varios orígenes se definen de la misma manera que se definen las reglas de un solo origen, pero con cada intervalo separado por una coma.

Ejemplo de PowerShell:

  ```azurepowershell-interactive
  Add-AzWebAppAccessRestrictionRule -ResourceGroupName "ResourceGroup" -WebAppName "AppName" `
    -Name "Multi-source rule" -IpAddress "192.168.1.0/24,192.168.10.0/24,192.168.100.0/24" `
    -Priority 100 -Action Allow
  ```

### <a name="block-a-single-ip-address"></a>Bloqueo de una dirección IP única

Al agregar la primera regla de restricción de acceso, el servicio agrega una regla *Denegar todo* explícita con una prioridad de 2147483647. En la práctica, la regla explícita *Denegar todo* es la última regla que se ejecuta, y bloquea el acceso a cualquier dirección IP que no esté expresamente permitida mediante una regla *Permitir*.

En una situación en la que quiera bloquear expresamente una única dirección IP o un bloque de direcciones IP, pero permitir el acceso a todo lo demás, agregue una regla *Permitir todo* explícita.

:::image type="content" source="media/app-service-ip-restrictions/block-single-address.png" alt-text="Captura de pantalla de la página &quot;Restricciones de acceso&quot; de Azure Portal que muestra una única dirección IP bloqueada.":::

### <a name="restrict-access-to-an-scm-site"></a>Restricción del acceso a un sitio de SCM 

Aparte de controlar el acceso a la aplicación, también puede restringir el acceso al sitio de SCM que utiliza la aplicación. El sitio de SCM es el punto de conexión de Web Deploy y la consola de Kudu. Puede asignar restricciones de acceso al sitio de SCM desde la aplicación de manera independiente o usar el mismo conjunto de restricciones para la aplicación y el sitio de SCM. Al activar la casilla **Mismas restricciones que \<app name>** , todo está en blanco. Si desactiva la casilla, se vuelve a aplicar la configuración del sitio de SCM. 

:::image type="content" source="media/app-service-ip-restrictions/access-restrictions-scm-browse.png" alt-text="Captura de pantalla de la página &quot;Restricciones de acceso&quot; de Azure Portal que muestra que no hay restricciones de acceso establecidas para la aplicación o el sitio de SCM.":::

### <a name="restrict-access-to-a-specific-azure-front-door-instance"></a>Restricción del acceso a una instancia específica de Azure Front Door
El tráfico de Azure Front Door a la aplicación se origina en un conjunto conocido de intervalos IP definidos en la etiqueta de servicio AzureFrontDoor.Backend. Con una regla de restricción de etiquetas de servicio, puede restringir el tráfico al que se origina en Azure Front Door. Para asegurarse de que el tráfico se origina únicamente en la instancia específica, tiene que filtrar aún más las solicitudes entrantes en función del encabezado HTTP único que envía Azure Front-Door.

:::image type="content" source="media/app-service-ip-restrictions/access-restrictions-frontdoor.png?v2" alt-text="Captura de pantalla de la página &quot;Restricciones de acceso&quot; de Azure Portal, en la que se muestra cómo agregar una restricción de Azure Front Door.":::

Ejemplo de PowerShell:

  ```azurepowershell-interactive
  $afd = Get-AzFrontDoor -Name "MyFrontDoorInstanceName"
  Add-AzWebAppAccessRestrictionRule -ResourceGroupName "ResourceGroup" -WebAppName "AppName" `
    -Name "Front Door example rule" -Priority 100 -Action Allow -ServiceTag AzureFrontDoor.Backend `
    -HttpHeader @{'x-azure-fdid' = $afd.FrontDoorId}
  ```

## <a name="manage-access-restriction-rules-programmatically"></a>Administración de reglas de restricción de acceso mediante programación

Puede agregar restricciones de acceso mediante programación si realiza alguna de las siguientes acciones: 

* Use la [CLI de Azure](/cli/azure/webapp/config/access-restriction). Por ejemplo:
   
  ```azurecli-interactive
  az webapp config access-restriction add --resource-group ResourceGroup --name AppName \
    --rule-name 'IP example rule' --action Allow --ip-address 122.133.144.0/24 --priority 100
  ```

* Use [Azure PowerShell](/powershell/module/Az.Websites/Add-AzWebAppAccessRestrictionRule). Por ejemplo:


  ```azurepowershell-interactive
  Add-AzWebAppAccessRestrictionRule -ResourceGroupName "ResourceGroup" -WebAppName "AppName"
      -Name "Ip example rule" -Priority 100 -Action Allow -IpAddress 122.133.144.0/24
  ```
   > [!NOTE]
   > Para trabajar con etiquetas de servicio, encabezados HTTP o reglas de varios orígenes se requiere al menos la versión 5.7.0. Para comprobar la versión del módulo instalado, escriba: **Get-InstalledModule -Name Az**

También puede establecer los valores manualmente mediante una de las siguientes acciones:

* Use una operación PUT de la [API REST de Azure](/rest/api/azure/) en la configuración de la aplicación en Azure Resource Manager. La ubicación de esta información en Azure Resource Manager es la siguiente:

  management.azure.com/subscriptions/**subscription ID**/resourceGroups/**resource groups**/providers/Microsoft.Web/sites/**web app name**/config/web?api-version=2020-06-01

* Use una plantilla de Resource Manager. Por ejemplo, puede usar resources.azure.com y editar el bloque ipSecurityRestrictions para agregar el código JSON requerido.

  La sintaxis JSON para el ejemplo anterior es:

  ```json
  {
    "properties": {
      "ipSecurityRestrictions": [
        {
          "ipAddress": "122.133.144.0/24",
          "action": "Allow",
          "priority": 100,
          "name": "IP example rule"
        }
      ]
    }
  }
  ```
  La sintaxis JSON de un ejemplo avanzado con etiqueta de servicio y restricción de encabezado HTTP es:
  ```json
  {
    "properties": {
      "ipSecurityRestrictions": [
        {
          "ipAddress": "AzureFrontDoor.Backend",
          "tag": "ServiceTag",
          "action": "Allow",
          "priority": 100,
          "name": "Azure Front Door example",
          "headers": {
            "x-azure-fdid": [
              "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
            ]
          }
        }
      ]
    }
  }
  ```
## <a name="set-up-azure-functions-access-restrictions"></a>Configuración de las restricciones de acceso de Azure Functions

También hay restricciones de acceso disponibles para las aplicaciones de funciones con la misma funcionalidad que los planes de App Service. Al habilitar las restricciones de acceso, también deshabilita el editor de código de Azure Portal en las direcciones IP no permitidas.

## <a name="next-steps"></a>Pasos siguientes
[Restricciones de acceso para Azure Functions](../azure-functions/functions-networking-options.md#inbound-access-restrictions)  
[Integración de Application Gateway con puntos de conexión de servicio](networking/app-gateway-with-service-endpoints.md)

<!--Links-->
[serviceendpoints]: ../virtual-network/virtual-network-service-endpoints-overview.md
[servicetags]: ../virtual-network/service-tags-overview.md
