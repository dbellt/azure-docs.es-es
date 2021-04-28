---
title: 'Método de autenticación mediante tokens OATH: Azure Active Directory'
description: Obtenga información sobre el uso de tokens OATH en Azure Active Directory para ayudar a mejorar y proteger los eventos de inicio de sesión.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 03/31/2021
ms.author: justinha
author: justinha
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: 99d0dd081e3e1a681ba55e3457b79a548d6b2bb7
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530388"
---
# <a name="authentication-methods-in-azure-active-directory---oath-tokens"></a>Métodos de autenticación en Azure Active Directory: tokens OATH 

TOTP (contraseñas de un solo uso y duración definida) de OATH es un estándar abierto que especifica cómo se general los códigos de contraseña de un solo uso (OTP). TOTP de OATH se puede implementar mediante software o hardware para generar los códigos. Azure AD no es compatible con HOTP de OATH, otro estándar de generación de códigos.

## <a name="oath-software-tokens"></a>Tokens de software OATH

Los tokens de software OATH suelen ser aplicaciones, como la aplicación Microsoft Authenticator y otras aplicaciones de autenticador. Azure AD genera la clave secreta, o valor de inicialización, que se introduce en la aplicación y se usa para generar cada OTP.

La aplicación Authenticator genera códigos automáticamente cuando se configura para realizar notificaciones de inserción, de modo que un usuario tenga una copia de seguridad incluso si el dispositivo no tiene conectividad. También se pueden usar aplicaciones de terceros que usen TOTP de OATH para generar códigos.

Algunos tokens de hardware TOTP de OATH son programables, es decir, no incluyen una clave secreta ni un valor de inicialización programados previamente. Estos tokens de hardware programables se pueden configurar con la clave secreta o el valor de inicialización obtenidos del flujo de configuración del token de software. Los clientes pueden adquirir estos tokens del proveedor de su elección y usar la clave secreta o el valor de inicialización en el proceso de configuración de su proveedor.

## <a name="oath-hardware-tokens-preview"></a>Tokens de hardware OATH (versión preliminar)

Azure AD admite el uso de tokens TOTP SHA-1 de OATH, que actualizan los códigos cada 30 o 60 segundos. Los clientes pueden adquirir estos tokens a través del proveedor de su elección.

Los tokens de hardware TOTP de OATH suelen incluir una clave secreta, o valor de inicialización, programada previamente en el token. Estas claves se deben introducir en Azure AD según se describe en los pasos siguientes. Las claves secretas se limitan a 128 caracteres lo que puede no ser compatible con todos los tokens. La clave secreta solo puede contener los caracteres *a-z* o *A-Z* y los dígitos *2-7*, y debe estar codificada en *base 32*.

Los tokens de hardware TOTP de OATH programables que se pueden reinicializar también se pueden configurar con Azure AD en el flujo de configuración de los tokens de software.

Los tokens de hardware OATH se admiten como parte de una versión preliminar pública. Para más información sobre las versiones preliminares, consulte [Términos de uso complementarios de las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

![Carga de tokens OATH en la hoja de tokens OATH de MFA](media/concept-authentication-methods/mfa-server-oath-tokens-azure-ad.png)

Una vez que se adquieren los tokens, se deben cargar en un formato de archivo de valores separados por comas (CSV), incluidos los valores de UPN, número de serie, clave secreta, intervalo de tiempo, fabricante y modelo, como se muestra en el ejemplo siguiente:

```csv
upn,serial number,secret key,time interval,manufacturer,model
Helga@contoso.com,1234567,2234567abcdef2234567abcdef,60,Contoso,HardwareKey
```  

> [!NOTE]
> Asegúrese de incluir la fila de encabezado en el archivo CSV. Si un UPN tiene una comilla simple, debe aplicar el escape con otra comilla simple. Por ejemplo, si el UPN es my’user@domain.com, cámbielo a my’’user@domain.com al cargar el archivo.

Una vez formateado correctamente como archivo CSV, un administrador global puede iniciar sesión en Azure Portal e ir a **Azure Active Directory > Seguridad > MFA > Tokens OATH** y cargar el archivo CSV resultante.

En función del tamaño del archivo CSV, puede tardar unos minutos en procesarse. Seleccione el botón **Actualizar** para ver el estado actual. Si hay algún error en el archivo, puede descargar un archivo CSV que enumere los errores para poder resolverlos. Los nombres de campo del archivo CSV descargado son diferentes de los de la versión cargada.  

Una vez solucionados los errores, para activar cada clave, el administrador puede seleccionar **Activar** para el token y escribir la OTP que se muestra en el token. Puede activar un máximo de 200 tokens OATH cada 5 minutos. 

Los usuarios pueden tener una combinación de hasta cinco tokens de hardware OATH o aplicaciones de autenticación, como la aplicación Microsoft Authenticator, configurada para utilizarse en cualquier momento. Los tokens OATH de hardware no se pueden asignar a los usuarios invitados en el inquilino de recursos.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la configuración de métodos de autenticación con la [API REST de Microsoft Graph](/graph/api/resources/authenticationmethods-overview).
