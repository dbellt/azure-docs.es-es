---
title: 'Tutorial: Escalado y protección de una aplicación web mediante Azure Front Door y Azure Web Application Firewall (WAF)'
description: En este tutorial se le mostrará cómo usar Azure Web Application Firewall con Azure Front Door Service.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/01/2020
ms.author: duau
ms.openlocfilehash: f02a8fff9ed3e75f5add287a2d5205988f87949f
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107896790"
---
# <a name="tutorial-quickly-scale-and-protect-a-web-application-by-using-azure-front-door-and-azure-web-application-firewall-waf"></a>Tutorial: Escalado y protección rápidos de una aplicación web mediante Azure Front Door y Azure Web Application Firewall (WAF)

En muchas aplicaciones web se ha producido un rápido aumento del tráfico en las últimas semanas con motivo de la COVID-19. También están experimentando un aumento en el tráfico malintencionado, incluidos los ataques por denegación de servicio. Existe una manera eficaz de realizar el escalado horizontal de la aplicación para las sobrecargas de tráfico y, al mismo tiempo, protegerse de ataques: configurar Azure Front Door con Azure WAF como una capa de aceleración, almacenamiento en caché y seguridad frente a la aplicación web. En este artículo se proporcionan instrucciones sobre cómo obtener esta configuración de Azure Front Door con Azure WAF para cualquier aplicación web que se ejecute dentro o fuera de Azure. 

Usaremos la CLI de Azure para configurar WAF en este tutorial. Puede realizar lo mismo mediante Azure Portal, Azure PowerShell, Azure Resource Manager o las API de REST de Azure. 

En este tutorial, aprenderá a:
> [!div class="checklist"]
> - Crear una instancia de Front Door
> - Crear una directiva de Azure WAF
> - Configurar conjuntos de reglas para una directiva WAF.
> - Asociar una directiva WAF con Front Door.
> - Configurar un dominio personalizado.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

- Las instrucciones de este tutorial utilizan la CLI de Azure. [Consulte esta guía](/cli/azure/get-started-with-azure-cli) para empezar a utilizar la CLI de Azure.

  > [!TIP] 
  > Una forma sencilla y rápida de empezar a trabajar en la CLI de Azure es con [Bash en Azure Cloud Shell](../cloud-shell/quickstart.md).

- Asegúrese de que la extensión `front-door` esté agregada a la CLI de Azure.

   ```azurecli-interactive 
   az extension add --name front-door
   ```

> [!NOTE] 
> Para más información sobre los comandos utilizados en este tutorial, consulte la [referencia de la CLI de Azure para Front Door](/cli/azure/).

## <a name="create-an-azure-front-door-resource"></a>Creación de un recurso de Azure Front Door

```azurecli-interactive 
az network front-door create --backend-address <>  --accepted-protocols <> --name <> --resource-group <>
```

`--backend-address`: nombre de dominio completo (FQDN) de la aplicación que quiere proteger. Por ejemplo, `myapplication.contoso.com`.

`--accepted-protocols`: especifica los protocolos que quiere que Azure Front Door admita para la aplicación web. Por ejemplo, `--accepted-protocols Http Https`.

`--name`: nombre del recurso de Azure Front Door.

`--resource-group`: grupo de recursos en el que desea colocar este recurso de Azure Front Door. Para más información sobre los grupos de recursos, consulte [Administración de grupos de recursos en Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).

En la respuesta que obtiene al ejecutar este comando, busque la clave `hostName`. Necesitará este valor en un paso posterior. `hostName` es el nombre DNS del recurso de Azure Front Door que creó.

## <a name="create-an-azure-waf-profile-to-use-with-azure-front-door-resources"></a>Creación de un perfil de Azure WAF para usarlo con recursos de Azure Front-Door

```azurecli-interactive 
az network front-door waf-policy create --name <>  --resource-group <>  --disabled false --mode Prevention
```

`--name`: nombre de la nueva directiva de Azure WAF.

`--resource-group`: grupo de recursos en el que quiere colocar este recurso de WAF. 

El código anterior de la CLI creará una directiva WAF habilitada y en modo de prevención. 

> [!NOTE] 
> Es posible que quiera crear la directiva WAF en modo de detección y observar cómo detecta y registra solicitudes malintencionadas (sin bloquearlas) antes de decidir usar el modo de protección.

En la respuesta que obtiene al ejecutar este comando, busque la clave `ID`. Necesitará este valor en un paso posterior. 

El campo `ID` debe tener este formato:

/subscriptions/**Id. de la suscripción**/resourcegroups/**nombre del grupo de recursos**/providers/Microsoft.Network/frontdoorwebapplicationfirewallpolicies/**nombre de la directiva de WAF**

## <a name="add-managed-rule-sets-to-the-waf-policy"></a>Incorporación de conjuntos de directivas administradas a la directiva WAF

Puede agregar conjuntos de reglas administradas a una directiva WAF. Un conjunto de reglas administradas es un conjunto de reglas que Microsoft ha creado y administra para ayudarle a protegerse frente a una clase de amenazas. En este ejemplo, vamos a agregar dos conjuntos de reglas:
- El conjunto de reglas predeterminado, que le ayuda a protegerse frente a amenazas web comunes. 
- El conjunto de reglas de protección contra bots, que le ayuda a protegerse frente a bots malintencionados.

Agregue el conjunto de reglas predeterminado:

   ```azurecli-interactive 
   az network front-door waf-policy managed-rules add --policy-name <> --resource-group <> --type DefaultRuleSet --version 1.0
   ```

Agregue el conjunto de reglas de protección contra bots:

   ```azurecli-interactive 
   az network front-door waf-policy managed-rules add --policy-name <> --resource-group <> --type Microsoft_BotManagerRuleSet --version 1.0
   ```

`--policy-name`: nombre que especificó para el recurso de Azure WAF.

`--resource-group`: grupo de recursos en el que colocó el recurso de WAF.

## <a name="associate-the-waf-policy-with-the-azure-front-door-resource"></a>Asociación de la directiva WAF con el recurso de Azure Front Door

En este paso, asociaremos la directiva WAF que hemos creado con el recurso de Azure Front Door que está delante de la aplicación web:

```azurecli-interactive 
az network front-door update --name <> --resource-group <> --set frontendEndpoints[0].webApplicationFirewallPolicyLink='{"id":"<>"}'
```

`--name`: nombre que especificó para el recurso de Azure Front Door.

`--resource-group`: grupo de recursos en el que colocó el recurso de Azure Front Door.

`--set`: Aquí es donde se actualiza el atributo `WebApplicationFirewallPolicyLink` del elemento `frontendEndpoint` asociado con el recurso de Azure Front Door con la nueva directiva WAF. Debe tener el identificador de la directiva WAF de la respuesta que obtuvo al crear el perfil de WAF anteriormente en este tutorial.

 > [!NOTE] 
> El ejemplo anterior es aplicable cuando no se usa un dominio personalizado. Si no usa dominios personalizados para acceder a las aplicaciones web, puede omitir la sección siguiente. En ese caso, proporcionará a los clientes el elemento `hostName` obtenido al crear el recurso de Azure Front Door. Usarán este elemento `hostName` para ir a su aplicación web.

## <a name="configure-the-custom-domain-for-your-web-application"></a>Configuración del dominio personalizado para la aplicación web

El nombre de dominio personalizado de la aplicación web es el que los clientes usan para hacer referencia a la aplicación. Por ejemplo, www.contoso.com. Inicialmente, este nombre de dominio personalizado apuntaba a la ubicación en la que se estaba ejecutando antes de introducir Azure Front Door. Después de agregar Azure Front Door y WAF frente a la aplicación, la entrada de DNS que corresponde a ese dominio personalizado debe apuntar al recurso de Azure Front Door. Para realizar este cambio, reasigne la entrada del servidor DNS al `hostName` de Azure Front Door que anotó al crear el recurso de Azure Front Door.

Los pasos específicos para actualizar los registros de DNS dependerán de su proveedor de servicios DNS. Si usa Azure DNS para hospedar su nombre DNS, puede consultar la documentación para conocer los [pasos para actualizar un registro de DNS](../dns/dns-operations-recordsets-cli.md) y apuntar al elemento `hostName` de Azure Front Door. 

Hay una cuestión importante que tener en cuenta si necesita que los clientes accedan a su sitio web mediante el vértice de zona (por ejemplo, contoso.com). En este caso, debe usar Azure DNS y su [tipo de registro de alias](../dns/dns-alias.md) para hospedar su nombre DNS. 

Además, también debe actualizar la configuración de Azure Front Door para [agregar el dominio personalizado](./front-door-custom-domain.md), de modo que tenga en cuenta esta asignación.

Por último, si está usando un dominio personalizado para llegar a la aplicación web y quiere habilitar el protocolo HTTPS, debe [configurar los certificados para el dominio personalizado en Azure Front Door](./front-door-custom-domain-https.md). 

## <a name="lock-down-your-web-application"></a>Bloqueo de la aplicación web

Se recomienda asegurarse de que solo los perímetros de Azure Front Door pueden comunicarse con la aplicación web. De este modo, se asegurará de que nadie pueda omitir la protección de Azure Front Door y acceder a la aplicación directamente. Para lograr este bloqueo, consulte [¿Cómo puedo hacer que Azure Front Door sea el único que tenga acceso a mi back-end?](./front-door-faq.yml#how-do-i-lock-down-the-access-to-my-backend-to-only-azure-front-door-)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos de este tutorial, use el comando [az group delete](/cli/azure/group#az_group_delete) para eliminar el grupo de recursos, Front Door y la directiva WAF.

```azurecli-interactive
  az group delete \
    --name <>
```
`--name`: nombre de grupo de recursos para todos los recursos que se usan en este tutorial.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo solucionar problemas de Front Door, consulte las guías paso a paso:

> [!div class="nextstepaction"]
> [Solución de problemas comunes de enrutamiento](front-door-troubleshoot-routing.md)
