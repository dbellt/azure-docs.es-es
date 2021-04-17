---
title: Solución de problemas de autenticación mutua en Azure Application Gateway
description: Obtenga información acerca de cómo solucionar problemas de autenticación mutua en Azure Application Gateway.
services: application-gateway
author: mscatyao
ms.service: application-gateway
ms.topic: troubleshooting
ms.date: 04/02/2021
ms.author: caya
ms.openlocfilehash: 623addd253b3eb28bdf70db02ddfbe4b320cbae7
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106230936"
---
# <a name="troubleshooting-mutual-authentication-errors-in-application-gateway-preview"></a>Solución de problemas de autenticación mutua en Application Gateway (versión preliminar)

Obtenga información acerca de cómo solucionar problemas de autenticación mutua al usar Azure Application Gateway. 

## <a name="overview"></a>Información general 

Después de configurar la autenticación mutua en una instancia de Application Gateway, pueden aparecer varios errores al intentar usar la autenticación mutua. Entre las causas comunes de los errores se incluyen las siguientes:

* Carga de un certificado o una cadena de certificados sin un certificado de entidad de certificación raíz.
* Carga de una cadena de certificados con varios certificados de entidad de certificación raíz.
* Carga de una cadena de certificados que solo contenía un certificado de hoja sin un certificado de entidad de certificación. 
* Errores de validación debidos a una discrepancia del nombre distintivo del emisor.  

Veremos diferentes escenarios con los que podría encontrarse y cómo solucionar problemas en estos escenarios. A continuación, abordaremos los códigos de error y explicaremos las causas probables de ciertos códigos de error que podrían aparecer con la autenticación mutua. 

## <a name="scenario-troubleshooting---configuration-problems"></a>Solución de problemas de los escenarios: problemas de configuración
Existen diversos escenarios con los que podría encontrarse al intentar configurar la autenticación mutua. Veremos cómo solucionar algunos de los problemas más comunes. 

### <a name="self-signed-certificate"></a>Certificado autofirmado

#### <a name="problem"></a>Problema 

El certificado de cliente que ha cargado es un certificado autofirmado y genera el código de error ApplicationGatewayTrustedClientCertificateDoesNotContainAnyCACertificate.

#### <a name="solution"></a>Solución 

Compruebe que el certificado autofirmado que está usando tiene la extensión *BasicConstraintsOid* = "2.5.29.19", que indica que el sujeto puede actuar como CA. Esto garantizará que el certificado utilizado es un certificado de CA. Para más información sobre cómo generar certificados de cliente autofirmados, consulte [Certificados de cliente de confianza](./mutual-authentication-certificate-management.md).

## <a name="scenario-troubleshooting---connectivity-problems"></a>Solución de problemas de los escenarios: problemas de conectividad

Es posible que haya podido configurar la autenticación mutua sin problemas, pero que se encuentre con algún problema al enviar solicitudes a su instancia de Application Gateway. En la siguiente sección se analizan algunos problemas y soluciones comunes. Puede encontrar la propiedad *sslClientVerify* en los registros de acceso de su instancia de Application Gateway. 

### <a name="sslclientverify-is-none"></a>SslClientVerify es NONE

#### <a name="problem"></a>Problema 

La propiedad *sslClientVerify* aparece como "NONE" en los registros de acceso. 

#### <a name="solution"></a>Solución 

Esto se produce cuando el cliente no envía un certificado de cliente al enviar una solicitud a la instancia de Application Gateway. Esto puede ocurrir si el cliente que envía la solicitud a la instancia de Application Gateway no está configurado correctamente para usar certificados de cliente. Una manera de comprobar que la configuración de autenticación del cliente en Application Gateway funciona según lo previsto es mediante el siguiente comando de OpenSSL:

```
openssl s_client -connect <hostname:port> -cert <path-to-certificate> -key <client-private-key-file> 
```

La marca `-cert` es el certificado de hoja y la marca `-key` es el archivo de clave privada del cliente. 

Para más información sobre cómo usar el comando `s_client` de OpenSSL, consulte la [página del manual](https://www.openssl.org/docs/man1.0.2/man1/openssl-s_client.html).

### <a name="sslclientverify-is-failed"></a>SslClientVerify es FAILED

#### <a name="problem"></a>Problema

La propiedad *sslClientVerify* aparece como "FAILED" en los registros de acceso. 

#### <a name="solution"></a>Solución

Hay varias causas posibles de errores en los registros de acceso. A continuación se muestra una lista de las causas comunes de error:
* **No se puede obtener el certificado del emisor:** no se ha podido encontrar el certificado del emisor del certificado de cliente. Normalmente, esto significa que la cadena de certificados de la CA de cliente de confianza no está completa en Application Gateway. Compruebe que la cadena de certificados de entidad de certificación de cliente de confianza cargada en Application Gateway está completa.  
* **No se puede obtener el certificado del emisor local:** como en el anterior error de obtención del certificado del emisor, no se ha podido encontrar el certificado del emisor del certificado de cliente. Normalmente, esto significa que la cadena de certificados de la CA de cliente de confianza no está completa en Application Gateway. Compruebe que la cadena de certificados de entidad de certificación de cliente de confianza cargada en Application Gateway está completa.
* **No se puede comprobar el primer certificado:** no se puede comprobar el certificado de cliente. Este error se produce específicamente cuando el cliente presenta solo el certificado de hoja, cuyo emisor no es de confianza. Compruebe que la cadena de certificados de entidad de certificación de cliente de confianza cargada en Application Gateway está completa.
* **No se puede comprobar el emisor del certificado de cliente:** este error se produce cuando el valor de configuración *VerifyClientCertIssuerDN* está establecido en true. Esto suele ocurrir cuando el nombre distintivo del emisor del certificado de cliente no coincide con el valor de *ClientCertificateIssuerDN* extraído de la cadena de certificados de entidad de certificación de cliente de confianza cargada por el cliente. Para más información acerca de cómo Application Gateway extrae el valor de *ClientCertificateIssuerDN*, consulte [Extracción del nombre distintivo del emisor con Application Gateway](./mutual-authentication-overview.md#verify-client-certificate-dn). Como procedimiento recomendado, asegúrese de cargar una cadena de certificados por archivo en Application Gateway. 

Para más información sobre cómo extraer toda la cadena de certificados de entidad de certificación de cliente de confianza para cargarla Application Gateway, consulte [Extracción de cadenas de certificados de entidad de certificación de cliente de confianza](./mutual-authentication-certificate-management.md).

## <a name="error-code-troubleshooting"></a>Solución de códigos de error
Si ve alguno de los siguientes códigos de error, existen algunas soluciones recomendadas que le ayudarán a resolver el posible problema. 

### <a name="error-code-applicationgatewaytrustedclientcertificatemustspecifydata"></a>Código de error: ApplicationGatewayTrustedClientCertificateMustSpecifyData

#### <a name="cause"></a>Causa

Faltan datos del certificado. El certificado cargado podría ser un archivo vacío sin datos de certificado. 

#### <a name="solution"></a>Solución

Compruebe que no faltan datos en el archivo de certificado cargado. 

### <a name="error-code-applicationgatewaytrustedclientcertificatemustnothaveprivatekey"></a>Código de error: ApplicationGatewayTrustedClientCertificateMustNotHavePrivateKey

#### <a name="cause"></a>Causa

Hay una clave privada en la cadena de certificados. No debería haber una clave privada en la cadena de certificados. 

#### <a name="solution"></a>Solución

Compruebe la cadena de certificados cargada y quite la clave privada que formaba parte de la cadena. Vuelva a cargar la cadena sin la clave privada. 

### <a name="error-code-applicationgatewaytrustedclientcertificateinvaliddata"></a>Código de error: ApplicationGatewayTrustedClientCertificateInvalidData

#### <a name="cause"></a>Causa

Hay dos causas posibles de este código de error.
1. Se ha producido un error en el análisis debido a que la cadena no se presentó en el formato correcto. Application Gateway espera que una cadena de certificados esté en formato PEM y también espera que los datos de certificado individuales estén delimitados. 
2. El analizador no ha encontrado nada que analizar. El archivo cargado podría tener únicamente los delimitadores, sin datos de certificado. 

#### <a name="solution"></a>Solución

Dependiendo de la causa de este error, existen dos posibles soluciones. 
* Compruebe que la cadena de certificados cargada tenía el formato correcto (PEM) y que los datos de certificado estaban delimitados correctamente. 
* Compruebe que el archivo de certificado cargado contenía los datos de certificado además de los delimitadores. 

### <a name="error-code-applicationgatewaytrustedclientcertificatedoesnotcontainanycacertificate"></a>Código de error: ApplicationGatewayTrustedClientCertificateDoesNotContainAnyCACertificate

#### <a name="cause"></a>Causa

El certificado cargado solo contenía un certificado de hoja sin un certificado de entidad de certificación. La carga de una cadena de certificados con certificados de entidad de certificación y un certificado de hoja es aceptable, ya que el certificado de hoja simplemente se omitirá, pero el certificado debe tener una entidad de certificación. 

#### <a name="solution"></a>Solución 

Compruebe que la cadena de certificados cargada contenía algo más que el certificado de hoja. La extensión *BasicConstraintsOid* = "2.5.29.19" debe estar presente e indicar que el sujeto puede actuar como entidad de certificación.

### <a name="error-code-applicationgatewayonlyonerootcaallowedintrustedclientcertificate"></a>Código de error: ApplicationGatewayOnlyOneRootCAAllowedInTrustedClientCertificate

#### <a name="cause"></a>Causa

La cadena de certificados no contenía ningún certificado de entidad de certificación raíz *o* contenía más de uno. 

#### <a name="solution"></a>Solución

Los certificados cargados deben contener exactamente un certificado de entidad de certificación raíz (sin embargo, se necesitan muchos certificados de entidad de certificación intermedios).


