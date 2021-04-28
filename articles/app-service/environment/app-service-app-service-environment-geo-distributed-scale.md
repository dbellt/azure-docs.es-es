---
title: Escalado distribuido geográficamente
description: Aprenda a escalar aplicaciones horizontalmente con distribución geográfica con el Administrador de tráfico y los entornos de App Service.
author: stefsch
ms.assetid: c1b05ca8-3703-4d87-a9ae-819d741787fb
ms.topic: article
ms.date: 09/07/2016
ms.author: stefsch
ms.custom: seodec18, references_regions, devx-track-azurepowershell
ms.openlocfilehash: 215132888749a54996b3341e43ef8d91c101a460
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107834307"
---
# <a name="geo-distributed-scale-with-app-service-environments"></a>Escala distribuida geográficamente con entornos de App Service
## <a name="overview"></a>Información general

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Aquellos escenarios de aplicaciones que requieren una escala muy elevada pueden superar la capacidad de recursos de proceso disponible para la implementación de una única aplicación.  Algunos ejemplos de los escenarios que requieren una escala extremadamente alta son las aplicaciones de votación, los acontecimientos deportivos y los espectáculos televisados. Se pueden cumplir los requisitos de gran escala mediante el escalado horizontal de las aplicaciones. Para controlar los requisitos de carga extrema, muchas implementaciones de aplicaciones se pueden realizar en una sola región y entre regiones.

Las instancias de App Service Environment son una plataforma ideal para el escalado horizontal. Una vez seleccionada una configuración de App Service Environment que admita una tasa de solicitudes conocida, los desarrolladores pueden implementar más instancias de App Service Environment a modo de "plantilla" para lograr una capacidad de carga máxima deseada.

Por ejemplo, imagine una aplicación que se ejecuta en una configuración de App Service Environment y en la que se probó el procesamiento de 20 000 solicitudes por segundo (RPS).  Si la capacidad de carga máxima deseada es 100 000 RPS, se pueden crear y configurar cinco (5) instancias de App Service Environment para asegurarse de que la aplicación pueda afrontar la carga máxima prevista.

Puesto que los clientes suelen acceder a las aplicaciones mediante un dominio personalizado (o mnemónico), los desarrolladores necesitan una forma de distribuir las solicitudes de aplicaciones a todas las instancias del entorno de App Service.  Una excelente manera de lograr este objetivo es resolver el dominio personalizado con un [perfil de Azure Traffic Manager][AzureTrafficManagerProfile].  El perfil del Administrador de tráfico puede configurarse para que apunte a todos los entornos de App Service individuales.  Traffic Manager controlará automáticamente la distribución de los clientes por todas las instancias de App Service Environment en función de la configuración de equilibrio de carga del perfil de Traffic Manager.  Este enfoque funciona independientemente de si todos los entornos de App Service están implementados en una sola región de Azure o en varias regiones de Azure del mundo.

Además, como los clientes acceden a las aplicaciones a través del dominio mnemónico, desconocen el número de entornos de App Service que ejecutan una aplicación.  Como resultado, los desarrolladores pueden agregar y quitar rápida y fácilmente entornos de App Service según la carga de tráfico observada.

En el siguiente diagrama conceptual, se muestra una aplicación escalada horizontalmente en tres entornos de App Service en una única región.

![Arquitectura conceptual][ConceptualArchitecture] 

El resto de este tema lo guía por los pasos necesarios para configurar una topología distribuida para la aplicación de ejemplo usando varios entornos de App Service.

## <a name="planning-the-topology"></a>Planeación de la topología
Antes de crear la superficie de una aplicación distribuida, resulta útil tener algunos datos con antelación.

* **Dominio personalizado para la aplicación:**  ¿cuál es el nombre de dominio personalizado que los clientes usarán para acceder a la aplicación?  Para la aplicación de ejemplo, el nombre de dominio personalizado es `www.scalableasedemo.com`.
* **Dominio de Traffic Manager:** elija un nombre de dominio al crear un [perfil de Azure Traffic Manager][AzureTrafficManagerProfile].  Este nombre se combinará con el sufijo *trafficmanager.net* para registrar una entrada de dominio que es administrada por el Administrador de tráfico.  Para la aplicación de ejemplo, el nombre elegido es *scalable-ase-demo*.  Como resultado, el nombre de dominio completo administrado por el Administrador de tráfico es *scalable-ase-demo.trafficmanager.net*.
* **Estrategia para escalar la superficie de la aplicación:**  ¿se va a distribuir el entorno de la aplicación entre varias instancias de App Service Environment de una sola región?  ¿Varias regiones?  ¿Una combinación de ambos enfoques?  Base la decisión en las expectativas de dónde se vaya a originar el tráfico del cliente y de la escalabilidad del resto de la infraestructura de back-end complementaria de una aplicación.  Por ejemplo, con una aplicación totalmente sin estado, se puede escalar una aplicación de forma masiva mediante una combinación de muchas instancias de App Service Environment en cada región de Azure, multiplicadas por las instancias de App Service Environment implementadas en muchas regiones de Azure.  Con más de 15 regiones de Azure globales entre las que elegir, los clientes pueden realmente crear una superficie de aplicación de gran escala en todo el mundo.  Para la aplicación de ejemplo usada en este artículo, se crearon tres instancias de App Service Environment en una sola región de Azure (Centro-sur de EE. UU.).
* **Convención de nomenclatura para las instancias de App Service Environment:**  cada instancia de App Service Environment requiere un nombre único.  Si existen más de uno o dos entornos de App Service Environment, resulta útil disponer de una convención de nomenclatura que ayude a identificarlos.  Para la aplicación de ejemplo, se usó una convención de nomenclatura sencilla.  Los nombres de las tres instancias de App Service Environment son *fe1ase*, *fe2ase* y *fe3ase*.
* **Convención de nomenclatura para las aplicaciones:**  dado que se van a implementar varias instancias de la aplicación, se necesita un nombre para cada instancia de la aplicación implementada.  Una característica poco conocida pero cómoda de App Service Environment es que se puede usar el mismo nombre de aplicación en varias instancias de App Service Environment.  Dado que cada uno tiene un sufijo de dominio único, los desarrolladores pueden reutilizar el mismo nombre de aplicación en cada entorno.  Por ejemplo, un desarrollador podría asignar los siguientes nombres a las aplicaciones: *miapp.foo1.p.azurewebsites.net*, *miapp.foo2.p.azurewebsites.net*, *miapp.foo3.p.azurewebsites.net*, etc.  No obstante, para la aplicación de ejemplo, cada instancia de aplicación tiene un nombre único.  Los nombres de las instancias de aplicación usados son *webfrontend1*, *webfrontend2* y *webfrontend3*.

## <a name="setting-up-the-traffic-manager-profile"></a>Configuración de un perfil del Administrador de tráfico
Una vez que se implementan varias instancias de una aplicación en varias instancias de App Service Environment, las instancias de aplicación individuales se pueden registrar en Traffic Manager.  Para la aplicación de ejemplo, se necesita un perfil de Traffic Manager para *scalable-ase-demo.trafficmanager.net* que pueda enrutar a los clientes a cualquiera de las siguientes instancias de aplicación implementadas:

* **webfrontend1.fe1ase.p.azurewebsites.net:**  una instancia de la aplicación de ejemplo implementada en la primera instancia de App Service Environment.
* **webfrontend2.fe2ase.p.azurewebsites.net:**  una instancia de la aplicación de ejemplo implementada en la segunda instancia de App Service Environment.
* **webfrontend3.fe3ase.p.azurewebsites.net:**  una instancia de la aplicación de ejemplo implementada en la tercera instancia de App Service Environment.

La forma más sencilla de registrar varios puntos de conexión de Azure App Service, todos en ejecución en la **misma** región de Azure, es usar la [compatibilidad de Azure Resource Manager con Traffic Manager][ARMTrafficManager] para PowerShell.  

El primer paso consiste en crear un perfil del Administrador de tráfico de Azure.  El código siguiente muestra cómo se creó el perfil para la aplicación de ejemplo:

```azurepowershell-interactive
$profile = New-AzureTrafficManagerProfile –Name scalableasedemo -ResourceGroupName yourRGNameHere -TrafficRoutingMethod Weighted -RelativeDnsName scalable-ase-demo -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
```

Observe la forma en que el parámetro *RelativeDnsName* se ha establecido en *scalable-ase-demo*.  Este parámetro permite crear el nombre de dominio *scalable-ase-demo.trafficmanager.net* y asociarlo con un perfil de Traffic Manager.

El parámetro *TrafficRoutingMethod* define la directiva de equilibrio de carga que Traffic Manager usará para determinar cómo distribuir la carga de los clientes entre todos los puntos de conexión disponibles.  En este ejemplo, se eligió el método *Weighted*.  Al elegir esta opción, las solicitudes de clientes se distribuirán entre todos los puntos de conexión de la aplicación registrados en función de las ponderaciones relativas asociadas con cada punto de conexión. 

Con el perfil creado, se agrega cada instancia de la aplicación al perfil como un punto de conexión nativo de Azure.  El código siguiente captura una referencia a cada aplicación web de front-end. Después, agrega cada aplicación como un punto de conexión de Traffic Manager a través del parámetro *TargetResourceId*.

```azurepowershell-interactive
$webapp1 = Get-AzWebApp -Name webfrontend1
Add-AzureTrafficManagerEndpointConfig –EndpointName webfrontend1 –TrafficManagerProfile $profile –Type AzureEndpoints -TargetResourceId $webapp1.Id –EndpointStatus Enabled –Weight 10

$webapp2 = Get-AzWebApp -Name webfrontend2
Add-AzureTrafficManagerEndpointConfig –EndpointName webfrontend2 –TrafficManagerProfile $profile –Type AzureEndpoints -TargetResourceId $webapp2.Id –EndpointStatus Enabled –Weight 10

$webapp3 = Get-AzWebApp -Name webfrontend3
Add-AzureTrafficManagerEndpointConfig –EndpointName webfrontend3 –TrafficManagerProfile $profile –Type AzureEndpoints -TargetResourceId $webapp3.Id –EndpointStatus Enabled –Weight 10

Set-AzureTrafficManagerProfile –TrafficManagerProfile $profile
```

Observe que existe una llamada a *Add-AzureTrafficManagerEndpointConfig* para cada instancia de aplicación individual.  El parámetro *TargetResourceId* de cada comando de PowerShell hace referencia a una de las tres instancias de la aplicación implementada.  El perfil de Traffic Manager repartirá la carga entre los tres puntos de conexión registrados en el perfil.

Los tres extremos usan el mismo valor (10) para el parámetro *Weight* .  Como resultado, Traffic Manager distribuye las solicitudes de los clientes entre las tres instancias de la aplicación de forma relativamente uniforme. 

## <a name="pointing-the-apps-custom-domain-at-the-traffic-manager-domain"></a>Hacer que el dominio personalizado de la aplicación apunte al dominio del Administrador de tráfico
El último paso necesario es hacer que el dominio personalizado de la aplicación apunte al dominio del Administrador de tráfico.  En la aplicación de ejemplo, apunte `www.scalableasedemo.com` en `scalable-ase-demo.trafficmanager.net`.  Complete este paso con el registrador de dominios que administra el dominio personalizado.  

Usando las herramientas de administración de dominios del registrador, se debe crear un registro CNAME que apunte al dominio personalizado en el dominio del Administrador de tráfico.  En la figura siguiente se muestra un ejemplo de esta configuración de CNAME:

![CNAME para el dominio personalizado][CNAMEforCustomDomain] 

Aunque no se trata en este tema, recuerde que cada instancia de aplicación individual debe tener el dominio personalizado registrado con ella también.  De lo contrario, si una solicitud llega a una instancia de la aplicación, y la aplicación no tiene el dominio personalizado registrado con la aplicación, se producirá un error con la solicitud.

En este ejemplo, el dominio personalizado es `www.scalableasedemo.com`, y cada instancia de la aplicación tiene asociado el dominio personalizado.

![Dominio personalizado][CustomDomain] 

Para acceder a un resumen del registro de un dominio personalizado con las aplicaciones de Azure App Service, vea el [registro de dominios personalizados][RegisterCustomDomain].

## <a name="trying-out-the-distributed-topology"></a>Prueba de la topología distribuida
El resultado final de la configuración de Traffic Manager y de DNS es que las solicitudes de `www.scalableasedemo.com` fluirán en la siguiente secuencia:

1. Un dispositivo o explorador realizará una búsqueda DNS de `www.scalableasedemo.com`.
2. La entrada CNAME en el registrador de dominios hace que la búsqueda DNS se redirija al Administrador de tráfico de Azure.
3. Se realiza una búsqueda DNS de *scalable-ase-demo.trafficmanager.net* en uno de los servidores DNS del Administrador de tráfico de Azure.
4. En función de la directiva de equilibrio de carga especificada anteriormente en el parámetro *TrafficRoutingMethod*, Traffic Manager selecciona uno de los puntos de conexión configurados. Después, devuelve el FQDN de ese punto de conexión al explorador o dispositivo.
5. Dado que el FQDN del punto de conexión es la dirección URL de una instancia de la aplicación que se ejecuta en una instancia de App Service Environment, el explorador o el dispositivo pedirá a un servidor DNS de Microsoft Azure que resuelva el FQDN en una dirección IP. 
6. El explorador o el dispositivo enviará la solicitud HTTP/S a la dirección IP.  
7. La solicitud llegará a una de las instancias de la aplicación que se ejecutan en uno de los entornos de App Service.

La imagen de la consola siguiente muestra una búsqueda de DNS para el dominio personalizado de la aplicación de ejemplo. Se resuelve correctamente en una instancia de la aplicación que se ejecuta en una de las tres instancias de App Service Environment de ejemplo (en este caso, la segunda de las tres instancias de App Service Environment):

![Búsqueda DNS][DNSLookup] 

## <a name="additional-links-and-information"></a>Información y vínculos adicionales
Documentación sobre la compatibilidad de [Azure Resource Manager con Azure Traffic Manager][ARMTrafficManager] para PowerShell.  

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[AzureTrafficManagerProfile]: ../../traffic-manager/traffic-manager-manage-profiles.md
[ARMTrafficManager]: ../../traffic-manager/traffic-manager-powershell-arm.md
[RegisterCustomDomain]: ../app-service-web-tutorial-custom-domain.md


<!-- IMAGES -->
[ConceptualArchitecture]: ./media/app-service-app-service-environment-geo-distributed-scale/ConceptualArchitecture-1.png
[CNAMEforCustomDomain]:  ./media/app-service-app-service-environment-geo-distributed-scale/CNAMECustomDomain-1.png
[DNSLookup]:  ./media/app-service-app-service-environment-geo-distributed-scale/DNSLookup-1.png
[CustomDomain]:  ./media/app-service-app-service-environment-geo-distributed-scale/CustomDomain-1.png 
