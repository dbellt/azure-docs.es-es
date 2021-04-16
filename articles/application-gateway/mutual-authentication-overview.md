---
title: Introducción a la autenticación mutua en Azure Application Gateway
description: Este artículo es una introducción a la autenticación mutua en Application Gateway.
services: application-gateway
author: mscatyao
ms.service: application-gateway
ms.date: 03/30/2021
ms.topic: conceptual
ms.author: caya
ms.openlocfilehash: a63697c2c9c32dfb48e77248a5e7a601677b8b3e
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106230953"
---
# <a name="overview-of-mutual-authentication-with-application-gateway-preview"></a>Introducción a la autenticación mutua con Application Gateway (versión preliminar)

La autenticación mutua, o autenticación de cliente, permite que Application Gateway autentique las solicitudes de envío de clientes. Normalmente, es solo el cliente el que se autentica en Application Gateway; la autenticación mutua permite que tanto el cliente como Application Gateway se autentiquen entre sí. 

> [!NOTE]
> Se recomienda usar TLS 1.2 con la autenticación mutua, ya que esta versión será obligatoria en el futuro. 

## <a name="mutual-authentication"></a>Autenticación mutua

Application Gateway admite la autenticación mutua basada en certificados, en la que puede cargar certificados de entidades de certificación de clientes de confianza en Application Gateway y la puerta de enlace usará esos certificados para autenticar al cliente que envía una solicitud a la puerta de enlace. Con el aumento de casos de uso de IoT y unos mayores requisitos de seguridad en los diversos sectores, la autenticación mutua proporciona una manera de administrar y controlar qué clientes pueden comunicarse con su instancia de Application Gateway. 

Para configurar la autenticación mutua, es necesario cargar un certificado de entidad de certificación de cliente de confianza como parte de la autenticación del cliente de un perfil de SSL. El perfil de SSL deberá asociarse a un cliente de escucha para completar la configuración de la autenticación mutua. Siempre debe haber un certificado de entidad de certificación raíz en el certificado de cliente que cargue. También puede cargar una cadena de certificados, pero la cadena debe incluir un certificado de una entidad de certificación (CA) raíz, además de todos los certificados de CA intermedios que desee. 

Por ejemplo, si el certificado de cliente contiene un certificado de CA raíz, varios certificados de CA intermedios y un certificado de hoja, asegúrese de que el certificado de CA raíz y todos los certificados de CA intermedios se cargan en Application Gateway en un archivo. Para más información sobre cómo extraer un certificado de entidad de certificación de cliente de confianza, consulte [Extracción de certificados de entidad de certificación de cliente de confianza](./mutual-authentication-certificate-management.md).

Si va a cargar una cadena de certificados con CA raíz y certificados de CA intermedios, la cadena de certificados se debe cargar como un archivo PEM o CER en la puerta de enlace. 

> [!NOTE] 
> La autenticación mutua solo está disponible en las SKU Standard_v2 y WAF_v2. 

### <a name="certificates-supported-for-mutual-authentication"></a>Certificados admitidos para la autenticación mutua

Application Gateway admite los siguientes tipos de certificados:

- Certificado CA (entidad de certificación): un certificado CA es un certificado digital emitido por una entidad de certificación (CA)
- Certificados autofirmados: los exploradores de cliente no confían en estos certificados y avisarán al usuario de que el certificado del servicio virtual no forma parte de una cadena de confianza. Los certificados de CA autofirmados son buenos para las pruebas o los entornos donde los administradores controlan a los clientes y pueden omitir de forma segura las alertas de seguridad del explorador. Las cargas de trabajo de producción nunca deben usar certificados de CA autofirmados.

Para más información sobre cómo configurar la autenticación mutua, consulte [Configuración de la autenticación mutua con Application Gateway](./mutual-authentication-portal.md).

> [!IMPORTANT]
> Asegúrese de cargar toda la cadena de certificados de la CA de cliente de confianza en Application Gateway al utilizar la autenticación mutua. 

## <a name="additional-client-authentication-validation"></a>Validación de la autenticación de cliente adicional

### <a name="verify-client-certificate-dn"></a>Comprobar el DN de certificado de cliente

Tiene la opción de comprobar el emisor inmediato del certificado de cliente y permitir que Application Gateway solo confíe en ese emisor. Esta opción está desactivada de forma predeterminada, pero puede habilitarla mediante Azure Portal, PowerShell o la CLI de Azure. 

Si decide habilitar Application Gateway para comprobar el emisor inmediato del certificado de cliente, aquí se indica cómo determinar el DN del emisor del certificado de cliente que se extraerá de los certificados cargados. 
* **Escenario 1:** la cadena de certificados incluye certificado raíz, certificado intermedio y certificado de hoja. 
    * El nombre del firmante del *certificado intermedio* es lo que Application Gateway extraerá como DN del emisor del certificado de cliente y lo que se comprobará. 
* **Escenario 2:** la cadena de certificados incluye certificado raíz, certificado intermedio 1, certificado intermedio 2 y certificado de hoja.
    * El nombre del firmante del *certificado intermedio 2* es lo que se extraerá como el DN del emisor del certificado de cliente y lo que se comprobará. 
* **Escenario 3:** la cadena de certificados incluye certificado raíz y certificado de hoja. 
    * El nombre del firmante del *certificado raíz* se extraerá y usará como DN del emisor del certificado de cliente.
* **Escenario 4:** varias cadenas de certificados de la misma longitud en el mismo archivo. La cadena 1 incluye: certificado raíz, certificado intermedio 1, certificado de hoja. La cadena 2 incluye: certificado raíz, certificado intermedio 2, certificado de hoja. 
    * El nombre del firmante del *certificado intermedio 1* se extraerá y usará como DN del emisor del certificado de cliente.  
* **Escenario 5:** varias cadenas de certificados de diferentes longitudes en el mismo archivo. La cadena 1 incluye: certificado raíz, certificado intermedio 1, certificado de hoja. La cadena 2 incluye: certificado raíz, certificado intermedio 2, certificado intermedio 3 y certificado de hoja. 
    * El nombre del firmante del *certificado intermedio 3* se extraerá y usará como DN del emisor del certificado de cliente. 

> [!IMPORTANT]
> Se recomienda cargar solo una cadena de certificados por cada archivo. Esto es especialmente importante si habilita la comprobación de DN del certificado de cliente. Si carga varias cadenas de certificados en un archivo, terminará en el escenario cuatro o cinco y es posible que surjan problemas más adelante cuando el certificado de cliente presentado no coincida con el DN del emisor del certificado de cliente que Application Gateway extrajo de las cadenas. 

Para más información sobre cómo extraer cadenas de certificados de entidad de certificación de cliente de confianza, consulte [Extracción de cadenas de certificados de entidad de certificación de cliente de confianza](./mutual-authentication-certificate-management.md).

## <a name="server-variables"></a>Variables de servidor 

Con la autenticación mutua, existen variables de servidor adicionales que se pueden usar para pasar información sobre el certificado de cliente a los servidores back-end detrás de Application Gateway. Para más información sobre las variables de servidor disponibles y cómo usarlas, consulte [Variables de servidor](./rewrite-http-headers-url.md#mutual-authentication-server-variables-preview).

## <a name="next-steps"></a>Pasos siguientes

Después de obtener información sobre la autenticación mutua, vaya a [Configuración de Application Gateway con autenticación mutua en PowerShell](./mutual-authentication-powershell.md) para crear una instancia de Application Gateway mediante la autenticación mutua. 

