---
title: Conexión privada a una aplicación web de Azure mediante un punto de conexión privado
description: Conexión privada a una aplicación web mediante un punto de conexión privado de Azure
author: ericgre
ms.assetid: 2dceac28-1ba6-4904-a15d-9e91d5ee162c
ms.topic: article
ms.date: 04/27/2021
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.custom: fasttrack-edit, references_regions
ms.openlocfilehash: 582a3303d5e43fff2390025bc35281c8ac916d84
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110096371"
---
# <a name="using-private-endpoints-for-azure-web-app"></a>Uso de puntos de conexión privados para una aplicación web de Azure

> [!IMPORTANT]
> El punto de conexión privado está disponible para la aplicación web de Windows y Linux en un contenedor (o no), y que esté hospedada en estos planes de App Service: **Aislado**, **PremiumV2**, **PremiumV3**, **Functions Premium** (a veces se conoce como plan Elástico Premium). 

Puede usar el punto de conexión privado para la aplicación web de Azure a fin de permitir que los clientes ubicados en la red privada accedan de forma segura a la aplicación a través de un vínculo privado. El punto de conexión privado usa una dirección IP del espacio de direcciones de la red virtual de Azure. El tráfico de red entre un cliente en la red privada y la aplicación web atraviesa la red virtual y un servicio Private Link en la red troncal de Microsoft, lo que elimina la exposición desde la red pública de Internet.

El uso de un punto de conexión privado para la aplicación web le permite:

- Proteger la aplicación web mediante la configuración del punto de conexión privado, lo que elimina la exposición pública.
- Conectarse de forma segura a la aplicación web desde las redes locales que se conectan a la red virtual mediante VPN o el emparejamiento privado de ExpressRoute.
- Evite la filtración de datos desde la red virtual. 

Si solo necesita una conexión segura entre la red virtual y la aplicación web, los puntos de conexión de servicio son la solución más sencilla. Si también necesita acceso a la aplicación web desde el entorno local mediante una instancia de Azure Gateway, una red virtual emparejada de forma regional o globalmente, el punto de conexión privado es la solución.  

Para obtener más información, consulte [Puntos de conexión de servicio][serviceendpoint].

## <a name="conceptual-overview"></a>Información general conceptual

Un punto de conexión privado es una interfaz de red (NIC) especial para la aplicación web de Azure en una subred dentro de la red virtual (Vnet).
Cuando se crea un punto de conexión privado para la aplicación web, se proporciona una conectividad segura entre los clientes de la red privada y la aplicación web. Al punto de conexión privado se le asigna una dirección IP del intervalo de direcciones IP de la red virtual.
La conexión entre el punto de conexión privado y la aplicación web usa una instancia de [Private Link][privatelink] segura. El punto de conexión privado solo se usa para los flujos entrantes en la aplicación web. Los flujos salientes no usarán este punto de conexión privado, pero puede insertar flujos salientes en la red de una subred diferente mediante la [característica de integración de red virtual][vnetintegrationfeature].

La subred en la que se conecta el punto de conexión privado puede tener otros recursos, no necesita una subred vacía dedicada.
También puede implementar el punto de conexión privado en una región distinta a la de la aplicación web. 

> [!Note]
>La característica de integración de red virtual no puede usar la misma subred que el punto de conexión privado; esta es una limitación de la característica de integración de red virtual.

Desde la perspectiva de seguridad:

- Cuando se habilitan puntos de conexión privados a la aplicación web, se deshabilita todo el acceso público.
- Puede habilitar varios puntos de conexión privados en otras redes virtuales y subredes, incluidas las redes virtuales de otras regiones.
- La dirección IP de la NIC de punto de conexión privado debe ser dinámica, pero seguirá siendo la misma hasta que se elimine el punto de conexión privado.
- La NIC del punto de conexión privado no puede tener un grupo de seguridad de red asociado.
- La subred que hospeda el punto de conexión privado puede tener un grupo de seguridad de red asociado, pero debe deshabilitar la aplicación de directivas de red para el punto de conexión privado. Consulte [Deshabilitar las directivas de red de puntos de conexión privados][disablesecuritype]. Como resultado, no puede filtrar por ningún grupo de seguridad de red el acceso al punto de conexión privado.
- Al habilitar un punto de conexión privado en la aplicación web, no se evalúa la configuración de [restricciones de acceso ][accessrestrictions] de la aplicación web.
- Puede eliminar el riesgo de filtración de datos desde la red virtual mediante la eliminación de todas las reglas de grupo de seguridad de red en las que el destino es la etiqueta Internet o servicios de Azure. Al implementar un punto de conexión privado para una aplicación web, solo puede acceder a esta aplicación web específica a través del punto de conexión privado. Si tiene otra aplicación web, tendrá que implementar en ella otro punto de conexión privado dedicado.

En los registros HTTP web de la aplicación web, encontrará la dirección IP de origen del cliente. Esta característica se implementa mediante el protocolo proxy TCP, reenviando la propiedad de la dirección IP de cliente a la aplicación web. Para obtener más información, consulte [Cómo obtener información de conexión mediante el proxy TCP V2][tcpproxy].


  > [!div class="mx-imgBorder"]
  > ![Información general global del punto de conexión privado de aplicación web](media/private-endpoint/global-schema-web-app.png)

## <a name="dns"></a>DNS

Cuando se usa un punto de conexión privado para la aplicación web, la dirección URL solicitada debe coincidir con el nombre de la aplicación web. De forma predeterminada, mywebappname.azurewebsites.net.

De forma predeterminada, sin el punto de conexión privado, el nombre público de la aplicación web es un nombre canónico para el clúster.
Por ejemplo, la resolución de nombres será:

|Nombre |Tipo |Value |
|-----|-----|------|
|mywebapp.azurewebsites.net|CNAME|clustername.azurewebsites.windows.net|
|clustername.azurewebsites.windows.net|CNAME|cloudservicename.cloudapp.net|
|cloudservicename.cloudapp.net|A|40.122.110.154| 


Cuando se implementa un punto de conexión privado, se actualiza la entrada DNS para apuntar al nombre canónico mywebapp.privatelink.azurewebsites.net.
Por ejemplo, la resolución de nombres será:

|Nombre |Tipo |Value |Comentario |
|-----|-----|------|-------|
|mywebapp.azurewebsites.net|CNAME|mywebapp.privatelink.azurewebsites.net|
|mywebapp.privatelink.azurewebsites.net|CNAME|clustername.azurewebsites.windows.net|
|clustername.azurewebsites.windows.net|CNAME|cloudservicename.cloudapp.net|
|cloudservicename.cloudapp.net|A|40.122.110.154|<-- esta dirección IP pública no es su punto de conexión privado; recibirá un error 403.|

Debe configurar un servidor DNS privado o una zona privada de Azure DNS para las pruebas en las que puede modificar la entrada del host de la máquina de prueba.
La zona DNS que debe crear es: **privatelink.azurewebsites.net**. Realice el registro de la aplicación web con un registro A y la dirección IP del punto de conexión privado.
Por ejemplo, la resolución de nombres será:

|Nombre |Tipo |Value |Comentario |
|-----|-----|------|-------|
|mywebapp.azurewebsites.net|CNAME|mywebapp.privatelink.azurewebsites.net|<--Azure crea esta entrada en el DNS público de Azure para que App Service apunte al vínculo privado que administramos nosotros.|
|mywebapp.privatelink.azurewebsites.net|A|10.10.10.8|<--Esta entrada se administra en el sistema DNS para apuntar a la dirección IP del punto de conexión privado.|

Después de esta configuración de DNS, puede acceder a la aplicación web de forma privada con el nombre predeterminado mywebappname.azurewebsites.net. Debe usar este nombre, ya que el certificado predeterminado se emite para *.azurewebsites.net.


Si necesita usar un nombre DNS personalizado, debe agregarlo en la aplicación web. El nombre personalizado se debe validar como cualquier otro mediante la resolución DNS pública. Para obtener más información, vea [Validación de un DNS personalizado][dnsvalidation].

En el caso de la consola de Kudu o la API de REST de Kudu (implementación con agentes autohospedados de Azure DevOps, por ejemplo), debe crear dos registros en la zona privada de Azure DNS o en el servidor DNS personalizado. 

| Nombre | Tipo | Value |
|-----|-----|-----|
| mywebapp.privatelink.azurewebsites.net | A | PrivateEndpointIP | 
| mywebapp.scm.privatelink.azurewebsites.net | A | PrivateEndpointIP | 



## <a name="pricing"></a>Precios

Para más información sobre los precios, consulte [Precios de Azure Private Link][pricing].

## <a name="limitations"></a>Limitaciones

Al usar Azure Functions en el plan Elástico Premium con un punto de conexión privado, para ejecutar la función en el portal web de Azure, debe tener acceso directo a la red o se producirá un error HTTP 403. En otras palabras, el explorador debe ser capaz de contactar con el punto de conexión privado para ejecutar la función desde el portal web de Azure. 

Puede conectar hasta 100 puntos de conexión privados a una aplicación web en particular.

La funcionalidad de depuración remota no está disponible cuando se habilita el punto de conexión privado para la aplicación web. Le recomendamos implementar el código en una ranura y depurarlo de forma remota en la misma.

El acceso FTP se proporciona a través de la dirección IP pública de entrada. El punto de conexión privado no admite el acceso FTP a la aplicación web.

Hay una limitación conocida que afecta a los puntos de conexión privados y al enrutamiento del tráfico con ranuras (la [característica Probar en producción][TiP]). A partir de abril de 2021, el enrutamiento automático y manual de solicitudes entre ranuras provocará el mensaje "403 Acceso denegado". Esta limitación se eliminará en una próxima versión.

Estamos mejorando la característica Private Link y el punto de conexión privado periódicamente. Consulte [este artículo][pllimitations] para información actualizada sobre las limitaciones.

## <a name="next-steps"></a>Pasos siguientes

- Para implementar un punto de conexión privado para la aplicación web mediante el portal, consulte [Cómo conectarse de forma privada a una aplicación web con el Portal][howtoguide1].
- Para implementar un punto de conexión privado para la aplicación web con el CLI de Azure, consulte [Cómo conectarse de forma privada a una aplicación web con CLI de Azure][howtoguide2].
- Para implementar un punto de conexión privado para la aplicación web con PowerShell, consulte [Cómo conectarse de forma privada a una aplicación web con PowerShell][howtoguide3].
- Para implementar un punto de conexión privado para la aplicación web con la plantilla de Azure, consulte [Cómo conectarse de forma privada a una aplicación web con la plantilla de Azure][howtoguide4].
- Ejemplo de un extremo a otro, cómo conectar una aplicación web de front-end a una aplicación web de back-end segura con inserción de red virtual y un punto de conexión privado mediante la plantilla de ARM, consulte este [inicio rápido][howtoguide5]
- Ejemplo de un extremo a otro, cómo conectar una aplicación web de front-end a una aplicación web de back-end segura con inserción de red virtual y un punto de conexión privado mediante Terraform, consulte este [ejemplo][howtoguide6]


<!--Links-->
[serviceendpoint]: ../../virtual-network/virtual-network-service-endpoints-overview.md
[privatelink]: ../../private-link/private-link-overview.md
[vnetintegrationfeature]: ../web-sites-integrate-with-vnet.md
[disablesecuritype]: ../../private-link/disable-private-endpoint-network-policy.md
[accessrestrictions]: ../app-service-ip-restrictions.md
[tcpproxy]: ../../private-link/private-link-service-overview.md#getting-connection-information-using-tcp-proxy-v2
[dnsvalidation]: ../app-service-web-tutorial-custom-domain.md
[pllimitations]: ../../private-link/private-endpoint-overview.md#limitations
[pricing]: https://azure.microsoft.com/pricing/details/private-link/
[howtoguide1]: ../../private-link/tutorial-private-endpoint-webapp-portal.md
[howtoguide2]: ../scripts/cli-deploy-privateendpoint.md
[howtoguide3]: ../scripts/powershell-deploy-private-endpoint.md
[howtoguide4]: ../scripts/template-deploy-private-endpoint.md
[howtoguide5]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.web/webapp-privateendpoint-vnet-injection
[howtoguide6]: ../scripts/terraform-secure-backend-frontend.md
[TiP]: https://docs.microsoft.com/azure/app-service/deploy-staging-slots#route-traffic
