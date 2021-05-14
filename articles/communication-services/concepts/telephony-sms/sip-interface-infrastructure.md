---
title: 'Requisitos de infraestructura del enrutamiento directo de Azure: Azure Communication Services'
description: Familiarícese con los requisitos de infraestructura para la configuración del enrutamiento directo de Azure Communication Services.
author: boris-bazilevskiy
manager: nmurav
services: azure-communication-services
ms.author: bobazile
ms.date: 03/10/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: f6e5aceaa6824d4bb6fd1bf938973c79cb5847e1
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108324594"
---
# <a name="azure-direct-routing-infrastructure-requirements"></a>Requisitos de infraestructura del enrutamiento directo de Azure 

[!INCLUDE [Private Preview Notice](../../includes/private-preview-include.md)]

 
En este artículo se proporcionan detalles sobre la infraestructura, las licencias y la conectividad del controlador de borde de sesión (SBC) que es conveniente tener en cuenta al planear la implementación del enrutamiento directo de Azure.


## <a name="infrastructure-requirements"></a>Requisitos de infraestructura
En la tabla siguiente se enumeran los requisitos de infraestructura de los SBC admitidos, los dominios y otros requisitos de conectividad de red para implementar el enrutamiento directo de Azure:  

|Requisito de infraestructura|Necesita lo siguiente|
|:--- |:--- |
|Controlador de borde de sesión (SBC)|Un SBC compatible. Para más información, consulte [SBC compatibles](#supported-session-border-controllers-sbcs).|
|Troncos de telefonía conectados al SBC|Uno o varios troncos de telefonía conectados al SBC. En un extremo, el SBC se conecta a Azure Communication Services mediante enrutamiento directo. El SBC también puede conectarse a entidades de telefonía de terceros, como PBX, adaptadores de telefonía analógicos, etc. Funcionará cualquier opción de conectividad de RTC conectada al SBC (Para conocer la configuración de los troncos de RTC en el SBC, consulte a los proveedores de SBC o los proveedores de troncos).|
|Suscripción de Azure|Una suscripción de Azure que se usa para crear un recurso de ACS, así como la configuración y la conexión al SBC.|
|Token de acceso de Communication Services|Para realizar llamadas, se necesita un token de acceso válido con el ámbito `voip`. Consulte [Tokens de acceso](../identity-model.md#access-tokens)|
|Dirección IP pública del SBC|Una dirección IP pública que se puede usar para conectarse al SBC. En función de su tipo, el SBC puede usar NAT.|
|Nombre de dominio completo (FQDN) del SBC|El nombre de dominio completo del SBC, donde la parte de dominio no coincide con los dominios registrados de una organización de Microsoft 365 u Office 365. Para más información, consulte [Nombres de dominio del SBC](#sbc-domain-names).|
|Entrada DNS pública para el SBC |Una entrada DNS pública que asigna el nombre de dominio completo del SBC a la dirección IP pública. |
|Certificado de confianza público para el SBC |Un certificado para el SBC que se va a usar en todas las comunicaciones con enrutamiento directo de Azure. Para más información, consulte el apartado sobre el [certificado de confianza público para el SBC](#public-trusted-certificate-for-the-sbc).|
|Direcciones IP y puertos del firewall para los elementos multimedia y la señalización de SIP |El SBC se comunica con los siguientes servicios en la nube:<br/><br/>Proxy SIP, que controla la señalización<br/>Procesador de multimedia, que controla los elementos multimedia<br/><br/>Estos dos servicios tienen direcciones IP independientes en Microsoft Cloud, que se describen más adelante en este documento.


## <a name="sbc-domain-names"></a>Nombres de dominio del SBC

Los clientes que no tengan Office 365 pueden utilizar cualquier nombre de dominio para el que puedan obtener un certificado público.

En la tabla siguiente se muestran ejemplos de nombres DNS registrados para el inquilino y de nombres de dominio completo válidos, y se indica si el nombre se puede usar como nombre de dominio completo (FQDN) para el SBC:

|Nombre DNS|Se puede usar para el nombre de dominio completo del SBC|Ejemplos de nombres de dominio completo|
|:--- |:--- |:--- |
contoso.com|Sí|**Nombres válidos:**<br/>sbc1.contoso.com<br/>ssbcs15.contoso.com<br/>europe.contoso.com|
|contoso.onmicrosoft.com|No|No se admiten los dominios *. onmicrosoft.com como nombres del SBC

Si es cliente de Office 365, el nombre de dominio del SBC no debe coincidir con el registrado en los dominios del inquilino de Office 365. A continuación se muestra un ejemplo de coexistencia de Office 365 y Azure Communication Service:

|Dominio registrado en Office 365|Ejemplos de nombre de dominio completo del SBC en Teams|Ejemplos de nombre de dominio completo del SBC en ACS|
|:--- |:--- |:--- |
**contoso.com** (dominio de segundo nivel)|**sbc.contoso.com** (nombre en el dominio de segundo nivel)|**sbc.acs.contoso.com** (nombre en el dominio de tercer nivel)<br/>**sbc.fabrikam.com** (cualquier nombre de otro dominio)|
|**o365.contoso.com** (dominio de tercer nivel)|**sbc.o365.contoso.com** (nombre en el dominio de tercer nivel)|**sbc.contoso.com** (nombre en el dominio de segundo nivel)<br/>**sbc.acs.o365.contoso.com** (nombre en el dominio de cuarto nivel)<br/>**sbc.fabrikam.com** (cualquier nombre de otro dominio)

El emparejamiento de SBC funciona en el nivel de recursos de Communication Services, lo que significa que puede emparejar muchos SBC con un solo recurso de Communication Services, pero no puede emparejar un solo SBC a más de un recurso de Communication Services. Los nombres de dominio completo únicos del SBC son necesarios para realizar el emparejamiento en recursos diferentes.

## <a name="public-trusted-certificate-for-the-sbc"></a>Certificado de confianza público para el SBC

Microsoft recomienda que para solicitar el certificado para el SBC se genere una solicitud de firma de certificación (CSR). Para obtener instrucciones concretas sobre la generación de una CSR para un SBC, consulte las instrucciones de interconexión o la documentación proporcionada por los proveedores del SBC. 

  > [!NOTE]
  > La mayoría de las entidades de certificación (CA) requieren que el tamaño de la clave privada sea al menos 2048. Téngalo en cuenta al generar la solicitud de firma de certificación.

El certificado debe tener el nombre de dominio completo del SBC como nombre común (CN) o en el campo de nombre alternativo del firmante (SAN). El certificado debe emitirse directamente desde una entidad de certificación, no desde un proveedor intermedio.

Como alternativa, el enrutamiento directo de Communication Services admite un carácter comodín en el CN o el SAN, y el carácter comodín debe ajustarse al estándar [HTTP de RFC sobre TLS](https://tools.ietf.org/html/rfc2818#section-3.1). 

Un ejemplo sería usar `\*.contoso.com`, que coincidiría con el nombre de dominio completo del SBC `sbc.contoso.com`, pero no con `sbc.test.contoso.com`.

El certificado debe generarlo una de las siguientes entidades de certificación raíz:

- AffirmTrust
- AddTrust External CA Root
- Baltimore CyberTrust Root*
- Buypass
- Cybertrust
- Class 3 Public Primary Certification Authority
- Comodo Secure Root CA
- Deutsche Telekom 
- DigiCert Global Root CA
- DigiCert High Assurance EV Root CA
- Entrust
- GlobalSign
- Go Daddy
- GeoTrust
- Verisign, Inc. 
- SSL.com
- Starfield
- Symantec Enterprise Mobile Root for Microsoft 
- SwissSign
- Thawte Timestamping CA
- Trustwave
- TeliaSonera 
- T-Systems International GmbH (Deutsche Telekom)
- QuoVadis

Microsoft está trabajando en agregar las entidades de certificación que solicitan los clientes. 

## <a name="sip-signaling-fqdns"></a>Señalización de SIP: nombres de dominio completo 

Los puntos de conexión para el enrutamiento directo de Communication Services son los tres nombres de dominio completo siguientes:

- **sip.pstnhub.microsoft.com**: nombre de dominio completo global (es el primero que se debe probar). Cuando el SBC envía una solicitud para resolver este nombre, los servidores DNS de Microsoft Azure devuelven una dirección IP que apunta al centro de recursos de Azure principal asignado al SBC. La asignación se basa en las métricas de rendimiento de los centros de trabajo y en la proximidad geográfica al SBC. La dirección IP devuelta corresponde al nombre de dominio completo principal.
- **sip2.pstnhub.microsoft.com**: nombre de dominio completo secundario (se asigna geográficamente a la segunda región de prioridad).
- **sip3.pstnhub.microsoft.com**: nombre de dominio completo terciario (se asigna geográficamente a la tercera región de prioridad).

Estos tres nombres de dominio completo deben colocarse en orden para:

- Proporcionar una experiencia óptima (menos cargada y más cercana al centro de datos del SBC asignado mediante la consulta del primer nombre de dominio completo).
- Proporcionar conmutación por error cuando se establece la conexión desde un SBC a un centro de datos en el que surge un problema temporal. Para más información, consulte el apartado [Mecanismo de conmutación por error](#failover-mechanism-for-sip-signaling) a continuación.  

Los nombres de dominio completos (sip.pstnhub.microsoft.com, sip2.pstnhub.microsoft.com y sip3.pstnhub.microsoft.com) se resolverán en una de las siguientes direcciones IP:

- `52.114.148.0`
- `52.114.132.46`
- `52.114.75.24` 
- `52.114.76.76` 
- `52.114.7.24` 
- `52.114.14.70`
- `52.114.16.74`
- `52.114.20.29`

Abra los puertos del firewall para estas direcciones IP, con el fin de permitir tanto que entre tráfico a estas direcciones, como que salga de ellas para la señalización. Si el firewall admite nombres DNS, el nombre de dominio completo`sip-all.pstnhub.microsoft.com` se resuelve en todas estas direcciones IP. 

## <a name="sip-signaling-ports"></a>Señalización de SIP: Puertos

Use los siguientes puertos para el enrutamiento directo de Azure Communication Services:

|Tráfico|From|En|Puerto de origen|Puerto de destino|
|:--- |:--- |:--- |:--- |:--- |
|SIP/TLS|Proxy SIP|SBC|1024 – 65535|Definido en el SBC (para GCC High y DoD de Office 365 solo se debe usar el puerto 5061)|
SIP/TLS|SBC|Proxy SIP|Definido en el SBC|5061|

### <a name="failover-mechanism-for-sip-signaling"></a>Mecanismo de conmutación por error para la señalización de SIP

El SBC realiza una consulta de DNS para resolver sip.pstnhub.microsoft.com. Para seleccionar el centro de datos principal se tienen en cuenta la ubicación del SBC y de las métricas de rendimiento del centro de recursos. Si surge algún problema en el centro de datos principal, el SBC probará sip2.pstnhub.microsoft.com, que se resuelve en el segundo centro de datos asignado y, en el caso excepcional de que no estén disponibles los centros de datos de ambas, el SBC reintenta el último nombre de dominio completo (sip3.pstnhub.microsoft.com), que proporciona la IP del centro de datos terciario.

## <a name="media-traffic-ip-and-port-ranges"></a>Tráfico de elementos multimedia: Intervalos de puertos y de IP

El tráfico de elementos multimedia no solo fluye hacia un servicio independiente denominado procesador de multimedia, sino también desde este. En el momento de publicar el procesador de multimedia para ACS, puede usar cualquier dirección IP de Azure. Descargue [la lista completa de direcciones](https://www.microsoft.com/download/details.aspx?id=56519).

### <a name="port-range"></a>Intervalo de puertos
En la tabla siguiente se muestra el intervalo de puertos de los procesadores de multimedia: 

|Tráfico|From|En|Puerto de origen|Puerto de destino|
|:--- |:--- |:--- |:--- |:--- |
|UDP/SRTP|Procesador de multimedia|SBC|3478-3481 y 49152 – 53247|Definido en el SBC|
|UDP/SRTP|SBC|Procesador de multimedia|Definido en el SBC|3478-3481 y 49152 – 53247|

  > [!NOTE]
  > Microsoft recomienda al menos dos puertos por llamada simultánea en el SBC.


## <a name="media-traffic-media-processors-geography"></a>Tráfico de elementos multimedia: Geografía de procesadores de multimedia

El tráfico del elementos multimedia fluye a través de componentes denominados procesadores de multimedia. Los procesadores de multimedia se colocan en los mismos centros de datos que los servidores proxy SIP. Además, hay más procesadores de multimedia para optimizar el flujo de medios. Por ejemplo, ahora no tenemos un componente de proxy de SIP en Australia (SIP fluye mediante el SAR de Hong Kong o Singapur), pero tenemos el procesador de multimedia local en Australia. La necesidad de los procesadores multimedia de forma local viene determinada por la latencia que experimentamos al enviar tráfico a larga distancia, por ejemplo, desde Australia hasta el SAR de Hong Kong o Singapur. Aunque la latencia en el ejemplo del tráfico que fluye desde Australia hasta el SAR de Hong Kong o Singapur es aceptable para conservar una buena calidad de llamada para el tráfico SIP, no lo es para el tráfico de elementos multimedia en tiempo real.

Ubicaciones en que se implementaron los componentes del proxy SIP y del procesador de multimedia:
- EE. UU. (dos en los centros de datos de Oeste de EE. UU. y Este de EE. UU.)
- Europa (centros de datos de Amsterdam y Dublín)
- Asia (centros de datos de Singapur y el SAR de Hong Kong)
- Australia (centros de datos de Este y Sudeste de Australia)

Ubicaciones en las que solo se implementan procesadores de multimedia (SIP fluye a través del centro de datos más cercano de los que se han enumerado anteriormente):
- Japón (centros de recursos de Este y Oeste de Japón)


## <a name="media-traffic-codecs"></a>Tráfico de elementos multimedia: Códecs

### <a name="leg-between-sbc-and-cloud-media-processor-or-microsoft-teams-client"></a>Segmento entre el procesador de multimedia en la nube y del SBC o el cliente de Microsoft Teams.
Se aplica al caso de omisión de elementos multimedia y a los de no omisión.

La interfaz de enrutamiento directo de Azure en el segmento entre el controlador de borde de sesión y el procesador multimedia en la nube puede usar los siguientes códecs:

- SILK, G.711, G.722, G.729

Puede forzar que se use el códec específico en el controlador de límites de sesión excluyendo los códecs no deseados de la oferta.

### <a name="leg-between-acs-sdk-app-and-cloud-media-processor"></a>Segmento entre la aplicación de SDK de ACS y el procesador de multimedia en la nube

En el segmento entre el procesador de multimedia en la nube y la aplicación de SDK de ACS, se usan SLIK o G.722. La elección del códec en este segmento se basa en los algoritmos de Microsoft, que tienen en cuenta varios parámetros. 

## <a name="supported-session-border-controllers-sbcs"></a>Controlador de límites de sesión admitidos (SBC)

La certificación está en curso. Entretanto, los clientes pueden usar los [controladores de límites de sesión certificados de Teams](/MicrosoftTeams/direct-routing-border-controllers). 

## <a name="next-steps"></a>Pasos siguientes

### <a name="conceptual-documentation"></a>Documentación conceptual

- [Concepto de telefonía](./telephony-concept.md)
- [Tipos de número de teléfono en Azure Communication Services](./plan-solution.md)
- [Precios](../pricing.md)

### <a name="quickstarts"></a>Guías de inicio rápido

- [Llamada a teléfono](../../quickstarts/voice-video-calling/pstn-call.md)