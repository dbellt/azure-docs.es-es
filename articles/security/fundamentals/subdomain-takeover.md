---
title: Evitar adquisiciones de subdominios con registros de alias de Azure DNS y comprobación de dominios personalizados de Azure App Service
description: Obtenga información sobre cómo evitar la amenaza común de alta gravedad que supone la adquisición de subdominios.
services: security
author: memildin
manager: rkarlin
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/04/2021
ms.author: memildin
ms.openlocfilehash: 5b815914d4a34d50e2bb566097620ee29e15f957
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110677532"
---
# <a name="prevent-dangling-dns-entries-and-avoid-subdomain-takeover"></a>Evitar las entradas DNS pendientes y la adquisición de subdominios

En este artículo se describe la amenaza de seguridad común de la adquisición de subdominios y los pasos que puede seguir para mitigarla.


## <a name="what-is-a-subdomain-takeover"></a>¿Qué es una adquisición de subdominios?

Las adquisiciones de subdominios son una amenaza común muy grave para las organizaciones que crean y eliminan muchos recursos con regularidad. Una adquisición de subdominios puede producirse cuando tiene un [registro DNS](../../dns/dns-zones-records.md#dns-records) que apunta a un recurso de Azure desaprovisionado. Estos registros DNS también se conocen como entradas "DNS pendientes". Los registros CNAME son especialmente vulnerables a esta amenaza. Las adquisición de subdominios permiten a los actores malintencionados redirigir el tráfico destinado al dominio de una organización a un sitio que realiza una actividad malintencionada.

El siguiente es un escenario común de adquisición de subdominio:

1. **CREACIÓN:**

    1. Permite aprovisionar un recurso de Azure con un nombre de dominio completo (FQDN) de `app-contogreat-dev-001.azurewebsites.net`.

    1. Puede asignar un registro CNAME en la zona DNS con el subdominio `greatapp.contoso.com` que enruta el tráfico a su recurso de Azure.

1. **DESAPROVISIONAMIENTO**

    1. El recurso de Azure se desaprovisiona o se elimina cuando ya no se necesita. 
    
        En este momento, el registro CNAME `greatapp.contoso.com` *debe* quitarse de la zona DNS. Si no se quita el registro CNAME, se anuncia como un dominio activo pero no enruta el tráfico a un recurso activo de Azure. Esta es la definición de un registro de DNS "pendiente".

    1. El subdominio pendiente, `greatapp.contoso.com`, ahora es vulnerable y se puede adquirir mediante su asignación a otro recurso de suscripción de Azure.

1. **ADQUISICIÓN:**

    1. Con las herramientas y los métodos de disponibilidad general, un actor de amenaza detecta el subdominio pendiente.  

    1. Este actor aprovisiona un recurso de Azure con el mismo FQDN del recurso que se ha controlado previamente. En este ejemplo, `app-contogreat-dev-001.azurewebsites.net`.

    1. El tráfico que se envía al subdominio `greatapp.contoso.com` ahora se enruta al recurso del actor malintencionado en el que se controla el contenido.



![Adquisición de subdominios desde un sitio web desaprovisionado](./media/subdomain-takeover/subdomain-takeover.png)



## <a name="the-risks-of-subdomain-takeover"></a>Riesgos de la adquisición de subdominios

Cuando un registro DNS apunta a un recurso que no está disponible, el propio registro debería quitarse de la zona DNS. Si no se ha eliminado, es un registro "DNS pendiente" y genera la posibilidad de que se produzca la adquisición de subdominios.

Las entradas DNS pendientes permiten a los actores de amenazas tomar el control del nombre DNS asociado para hospedar un servicio o un sitio web malintencionado. Las páginas y los servicios malintencionados del subdominio de una organización pueden dar lugar a:

- **Pérdida de control sobre el contenido del subdominio**: prensa negativa sobre la incapacidad de la organización para proteger su contenido, así como perjuicios para la marca y pérdida de confianza.

- **Recopilación de cookies de visitantes confiados**: es habitual que las aplicaciones web expongan cookies de sesión a subdominios (*.contoso.com), de modo que cualquier subdominio puede acceder a estas. Los actores de amenazas pueden usar la adquisición de subdominios para crear una página de aspecto auténtico, engañar a los usuarios confiados para que la visiten y recopilar sus cookies (incluso las cookies seguras). Una idea equivocada habitual es que el uso de certificados SSL protege su sitio y las cookies de los usuarios de una adquisición. Sin embargo, un actor de amenaza puede usar el subdominio secuestrado para solicitar y recibir un certificado SSL válido. Los certificados SSL válidos les conceden acceso a las cookies seguras y pueden aumentar aún más la legitimidad aparente del sitio malintencionado.

- **Campañas de suplantación de identidad (phishing)** : los subdominios de aspecto auténtico se pueden usar en campañas de suplantación de identidad. Esto se aplica a los sitios malintencionados y a los registros MX, que permitirían que el actor de amenaza recibiera correos electrónicos dirigidos a un subdominio legítimo de una marca segura.

- **Más riesgos**: los sitios malintencionados se pueden usar para escalar a otros ataques clásicos, como XSS, CSRF, omisión de CORS, etc.



## <a name="identify-dangling-dns-entries"></a>Identificación de las entradas DNS pendientes

Para identificar las entradas DNS de la organización que podrían estar pendientes, utilice las herramientas de PowerShell ["Get-DanglingDnsRecords"](https://aka.ms/DanglingDNSDomains) hospedadas en GitHub de Microsoft.

Esta herramienta ayuda a los clientes de Azure a mostrar todos los dominios con un CNAME asociado a un recurso de Azure existente creado en sus suscripciones o inquilinos.

Si los CNAME están en otros servicios DNS y apuntan a recursos de Azure, proporcione el CNAME en un archivo de entrada a la herramienta.

La herramienta admite los recursos de Azure que se muestran en la tabla siguiente. La herramienta extrae, o toma como entradas, todo el CNAME del inquilino.


| Servicio                   | Tipo                                        | FQDNproperty                               | Ejemplo                         |
|---------------------------|---------------------------------------------|--------------------------------------------|---------------------------------|
| Azure Front Door          | microsoft.network/frontdoors                | properties.cName                           | `abc.azurefd.net`               |
| Azure Blob Storage        | microsoft.storage/storageaccounts           | properties.primaryEndpoints.blob           | `abc. blob.core.windows.net`    |
| Azure CDN                 | microsoft.cdn/profiles/endpoints            | properties.hostName                        | `abc.azureedge.net`             |
| Direcciones IP públicas       | microsoft.network/publicipaddresses         | properties.dnsSettings.fqdn                | `abc.EastUs.cloudapp.azure.com` |
| Administrador de tráfico de Azure     | microsoft.network/trafficmanagerprofiles    | properties.dnsConfig.fqdn                  | `abc.trafficmanager.net`        |
| Azure Container Instances  | microsoft.containerinstance/containergroups | properties.ipAddress.fqdn                  | `abc.EastUs.azurecontainer.io`  |
| Azure API Management      | microsoft.apimanagement/service             | properties.hostnameConfigurations.hostName | `abc.azure-api.net`             |
| Azure App Service         | microsoft.web/sites                         | properties.defaultHostName                 | `abc.azurewebsites.net`         |
| Azure App Service - Slots | microsoft.web/sites/slots                   | properties.defaultHostName                 | `abc-def.azurewebsites.net`     |



### <a name="prerequisites"></a>Requisitos previos

Ejecute la consulta como un usuario que tenga:

- al menos acceso de nivel de lector a las suscripciones de Azure
- acceso de lectura al gráfico de recursos de Azure

Si es un administrador global del inquilino de su organización, eleve los privilegios de su cuenta para que tenga acceso a toda la suscripción de su organización mediante las instrucciones del artículo [Elevación de los privilegios de acceso para administrar todas las suscripciones y los grupos de administración de Azure](../../role-based-access-control/elevate-access-global-admin.md).


> [!TIP]
> Azure Resource Graph tiene límites paginación y límites de ancho de banda que debe tener en cuenta si tiene un entorno de Azure de gran tamaño. 
> 
> [Más información sobre el trabajo con grandes conjuntos de datos de recursos de Azure](../../governance/resource-graph/concepts/work-with-data.md).
> 
> La herramienta usa el procesamiento por lotes de suscripciones para evitar estas limitaciones.

### <a name="run-the-script"></a>Ejecute el script.

Obtenga más información sobre el script de PowerShell, **Get-DanglingDnsRecords.ps1** y descárguelo desde GitHub: https://aka.ms/Get-DanglingDnsRecords.

## <a name="remediate-dangling-dns-entries"></a>Corrección de las entradas DNS pendientes 

Revise las zonas DNS e identifique los registros CNAME que están pendientes o que se han adquirido. Si se detecta que los subdominios están pendientes o se han adquirido, quite los subdominios vulnerables y mitigue los riesgos con los siguientes pasos:

1. En la zona DNS, quite todos los registros CNAME que señalen a los FQDN de los recursos que ya no se aprovisionan.

1. Para permitir que el tráfico se enrute a los recursos del control, aprovisione recursos adicionales con los FQDN especificados en los registros CNAME de los subdominios pendientes.

1. Revise el código de aplicación para ver las referencias a subdominios específicos y actualice las referencias de subdominios incorrectas o no actualizadas.

1. Investigue si se ha producido algún riesgo y tome medidas según los procedimientos de respuesta a incidentes de la organización. A continuación encontrará sugerencias y procedimientos recomendados para investigar este problema.

    Si la lógica de la aplicación es tal que los secretos como las credenciales de OAuth se enviaron al subdominio pendiente, o la información confidencial de la privacidad se envió a los subdominios pendientes, esos datos podrían haberse expuesto a terceros.

1. Comprenda por qué el registro CNAME no se quitó de la zona DNS cuando se canceló el aprovisionamiento del recurso y realice los pasos necesarios para asegurarse de que los registros DNS se actualicen correctamente cuando se desaprovisionan los recursos de Azure en el futuro.


## <a name="prevent-dangling-dns-entries"></a>Evitación de los registros DNS pendientes

Asegurarse de que su organización ha implementado procesos para evitar las entradas DNS pendientes y las adquisiciones de subdominios resultantes es una parte fundamental del programa de seguridad.

Algunos servicios de Azure ofrecen características que ayudan a crear medidas preventivas y se detallan a continuación. Otros métodos para evitar este problema se deben establecer a través de las prácticas recomendadas de la organización o de los procedimientos operativos estándar.

### <a name="enable-azure-defender-for-app-service"></a>Habilitación de Azure Defender para App Service

La plataforma integrada de protección de cargas de trabajo en la nube (CWP) de Azure Security Center, Azure Defender, ofrece la protección una amplia variedad de planes para proteger sus recursos y cargas de trabajo híbridas de varias nubes de Azure.

El plan de **Azure defender para App Service** incluye la detección de DNS pendiente. Con este plan habilitado, recibirá alertas de seguridad si retira un sitio web de App Service pero no quita su dominio personalizado del registrador de DNS.

La protección contra DNS pendiente de Azure Defender está disponible tanto si los dominios se administran con Azure DNS como con un registrador de dominios externo y se aplica a App Service en Windows y en Linux.

Obtenga más información sobre esta y otras ventajas de este plan de Azure Defender en [Introducción a Azure Defender para App Service](../../security-center/defender-for-app-service-introduction.md).

### <a name="use-azure-dns-alias-records"></a>Uso de registros de alias de Azure DNS

Los [registros de alias](../../dns/dns-alias.md#scenarios) de Azure DNS pueden evitar referencias pendientes mediante un acoplamiento del ciclo de vida de un registro DNS con un recurso de Azure. Por ejemplo, considere un registro DNS que se califica como un registro de alias que apunte a una dirección IP pública o a un perfil de Traffic Manager. Si elimina los recursos subyacentes, el registro de alias de DNS se convierte en un conjunto de registros vacío. Ya no hace referencia al recurso eliminado. Es importante tener en cuenta que se aplican límites a lo que se puede proteger con los registros de alias. Actualmente, esta lista se limita a lo siguiente:

- Azure Front Door
- Perfiles de Traffic Manager
- Puntos de conexión de Azure Content Delivery Network (CDN)
- Direcciones IP públicas

A pesar de las ofertas de servicio limitadas actualmente, se recomienda usar registros de alias para defenderse de la adquisición de subdominios siempre que sea posible.

[Obtenga más información sobre las funcionalidades de los registros de alias de Azure DNS](../../dns/dns-alias.md#capabilities).



### <a name="use-azure-app-services-custom-domain-verification"></a>Uso de la comprobación de dominio personalizado de Azure App Service

Al crear entradas DNS para Azure App Service, cree un registro TXT asuid.{subdominio} con el identificador de comprobación de dominio. Cuando existe un registro TXT de este tipo, ninguna otra suscripción a Azure puede validar el dominio personalizado es decir, adquirirlo. 

Estos registros no impiden que alguien cree la instancia de Azure App Service con el mismo nombre que en la entrada CNAME. Sin la capacidad de demostrar la propiedad del nombre de dominio, los actores de amenazas no pueden recibir tráfico ni controlar el contenido.

[Obtenga más información sobre cómo asignar un nombre DNS personalizado existente a Azure App Service](../../app-service/app-service-web-tutorial-custom-domain.md).



### <a name="build-and-automate-processes-to-mitigate-the-threat"></a>Compilación y automatización de procesos para mitigar la amenaza

A menudo, los desarrolladores y los equipos de operaciones deben ejecutar procesos de limpieza para evitar las amenazas de DNS pendientes. Los procedimientos siguientes le ayudarán a asegurarse de que su organización evita sufrir esta amenaza. 

- **Creación de procedimientos para la prevención:**

    - Instruya a los desarrolladores de aplicaciones para que redirijan las direcciones siempre que eliminen recursos.

    - Incluya "Quitar entrada DNS" en la lista de comprobaciones necesarias al retirar un servicio.

    - Incluya [bloqueos de eliminación](../../azure-resource-manager/management/lock-resources.md) en los recursos que tengan una entrada DNS personalizada. Un bloqueo de eliminación sirve como indicador para quitar la asignación antes de desaprovisionar el recurso. Medidas como esta solo pueden funcionar si se combinan con programas educativos internos.

- **Creación de procedimientos para la detección:**

    - Revise los registros DNS con regularidad para asegurarse de que todos los subdominios están asignados a recursos de Azure:

        - Existentes: consulte las zonas DNS para ver los recursos que apuntan a subdominios de Azure, como *.azurewebsites.net o *.cloudapp.azure.com (consulte la [lista de referencia de los dominios de Azure](azure-domains.md)).
        - De su propiedad: confirme que posee todos los recursos a los que se dirigen sus subdominios DNS.

    - Mantenga un catálogo de servicios de los puntos de conexión de nombre de dominio completo (FQDN) de Azure y los propietarios de la aplicación. Para compilar el catálogo de servicios, ejecute el siguiente script de consulta de Azure Resource Graph. Este script proyecta la información del punto de conexión de FQDN de los recursos a los que tiene acceso y los envía a un archivo CSV. Si tiene acceso a todas las suscripciones de su inquilino, el script tiene en cuenta todas estas suscripciones, tal y como se muestra en el siguiente script de ejemplo. Para limitar los resultados a un conjunto específico de suscripciones, edite el script como se muestra.


- **Creación de procedimientos para la corrección:**
    - Cuando se encuentran entradas DNS pendientes, su equipo debe investigar si se ha producido algún riesgo.
    - Investigue por qué la dirección no se reenrutó al retirar el recurso.
    - Elimine el registro DNS si ya no está en uso o apunte al recurso de Azure (FQDN) correcto que pertenece a su organización.
 

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los servicios relacionados y las características de Azure que puede usar para defenderse de la adquisición de subdominios, consulte las páginas siguientes.

- [Habilitación de Azure Defender para App Service](../../security-center/defender-for-app-service-introduction.md): para recibir alertas cuando se detectan entradas de DNS pendientes.

- [Impedir que los registros DNS queden pendientes con Azure DNS](../../dns/dns-alias.md#prevent-dangling-dns-records)

- [Uso de un identificador de comprobación de dominio al agregar dominios personalizados en Azure App Service](../../app-service/app-service-web-tutorial-custom-domain.md#3-get-a-domain-verification-id)

- [Inicio rápido: Ejecutar la primera consulta de Resource Graph con Azure PowerShell](../../governance/resource-graph/first-query-powershell.md)
