---
title: Inicios de sesión de AD FS en Azure AD con Connect Health | Microsoft Docs
description: En este documento se describe cómo integrar los inicios de sesión de AD FS con el informe de inicios de sesión de Azure AD Connect Health.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 03/16/2021
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 42326cd4080fe6737fd14f8dc2c5a2028c20077f
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111854547"
---
# <a name="ad-fs-sign-ins-in-azure-ad-with-connect-health---preview"></a>Inicios de sesión de AD FS en Azure AD con Connect Health (versión preliminar)

Los inicios de sesión de AD FS ahora se pueden integrar en el informe de inicios de sesión de Azure Active Directory mediante Connect Health. El [informe de inicios de sesión de Azure AD](../reports-monitoring/concept-all-sign-ins.md) incluye información sobre cuándo los usuarios, las aplicaciones y los recursos administrados inician sesión en Azure AD y acceden a los recursos. 

El agente de Connect Health para AD FS correlaciona varios identificadores de evento de AD FS según la versión del servidor con el fin de proporcionar información sobre la solicitud y los detalles del error si se produce un error en la solicitud. Esta información se correlaciona con el esquema del informe de inicios de sesión Azure AD y se muestra en la experiencia de usuario del informe de inicios de sesión de Azure AD. Junto con el informe, se proporciona una nueva secuencia de Log Analytics con los datos de AD FS y una nueva plantilla de libro de Azure Monitor. La plantilla se puede usar y modificar para hacer un análisis exhaustivo de escenarios como bloqueos de cuenta de AD FS, intentos con contraseñas incorrectas o picos de intentos inesperados de inicio de sesión.

## <a name="prerequisites"></a>Prerrequisitos
* Tener Azure AD Connect Health para AD FS instalado y actualizado con la versión más reciente (3.1.95.0 o posterior).
* Rol de administrador global o lector de informes para ver los inicios de sesión Azure AD.

## <a name="what-data-is-displayed-in-the-report"></a>¿Qué datos se muestran en el informe?
Los datos disponibles reflejan los mismos datos que se muestran para los inicios de sesión de Azure AD. Hay cinco pestañas con información disponible según el tipo de inicio de sesión, de Azure AD o AD FS. Connect Health correlaciona eventos de AD FS en función de la versión del servidor y los asocia con el esquema de AD FS. 



#### <a name="user-sign-ins"></a>Inicios de sesión de usuario 
Cada pestaña de la hoja de inicios de sesión muestra los siguientes valores predeterminados:
* Fecha de inicio de sesión
* Id. de solicitud
* Nombre de usuario o identificador de usuario
* Estado del inicio de sesión
* Dirección IP del dispositivo usado para el inicio de sesión
* Identificador de inicio de sesión

#### <a name="authentication-method-information"></a>Información sobre el método de autenticación
Los siguientes valores pueden mostrarse en la pestaña de autenticación. El método de autenticación se toma de los registros de auditoría de AD FS.

|Método de autenticación|Descripción|
|-----|-----|
|Formularios|Autenticación con nombre de usuario y contraseña|
|Windows|Autenticación integrada de Windows|
|Certificado|Autenticación mediante certificados SmartCard/VirtualSmart|
|WindowsHelloForBusiness|Este campo es para la autenticación con Windows Hello para empresas (Autenticación de Microsoft Passport).|
|Dispositivo | Se muestra si se ha seleccionado la autenticación de dispositivos como método "principal"de autenticación de la intranet o extranet y se lleva a cabo.  En este escenario no hay un método de autenticación de usuarios por separado.| 
|Federado|AD FS no efectuó la autenticación pero la transfirió a un proveedor de identidades de terceros.|
|SSO |Si se usó un token de inicio de sesión único, se mostrará este campo. Si el inicio de sesión único usa un proveedor MFA, se mostrará Multifactor.|
|Multifactor|Si un token de inicio de sesión único usa un proveedor MFA y se usó previamente para la autenticación, este campo mostrará Multifactor.|
|Azure MFA|Azure MFA se selecciona como proveedor de autenticación adicional en AD FS y se usó con fines de autenticación.|
|ADFSExternalAuthenticationProvider|Este campo se usa si un proveedor de autenticación de terceros se registró y usó para la autenticación.|


#### <a name="ad-fs-additional-details"></a>Otros detalles sobre AD FS
Los siguientes detalles está disponible para inicios de sesión de AD FS:
* Nombre del servidor
* Cadena de IP
* Protocolo

### <a name="enabling-log-analytics-and-azure-monitor"></a>Habilitación de Log Analytics y Azure Monitor
Log Analytics se pueden habilitar para los inicios de sesión AD FS y usar con otros componentes integrados de Log Analytics, como Sentinel.

> [!NOTE] 
> Los inicios de sesión de AD FS pueden aumentar significativamente el costo de Log Analytics en función del número de inicios de sesión en AD FS en la organización. Para habilitar y deshabilitar Log Analytics, seleccione la casilla correspondiente para la secuencia.

Para habilitar Log Analytics para la característica, vaya a la hoja Log Analytics y seleccione la secuencia "ADFSSignIns". Esta selección permitirá que los inicios de sesión de AD FS se transfieran a Log Analytics.

Para acceder a la plantilla de libro actualizada de Azure Monitor, vaya a "Azure Monitor Templates" (Plantillas de Azure Monitor) y seleccione el libro de "inicios de sesión".
Para más información sobre los libros, consulte [Libros de Azure Monitor](https://aka.ms/adfssigninspreview).




### <a name="frequently-asked-questions"></a>Preguntas más frecuentes
***¿Qué tipos de inicio de sesión puedo ver?***
El informe de inicios de sesión admite inicios de sesión mediante los protocolos O-Auth, WS-Fed, SAML y WS-Trust. 

***¿En qué se diferencian los tipos de inicio de sesión que se muestran en el informe de inicios de sesión?***
Si se efectúa un inicio de sesión de conexión directa, se mostrará una fila para el inicio de sesión con un identificador de correlación.
Si se efectúa la autenticación de un solo factor, se mostrarán dos filas con el mismo identificador de correlación, pero con dos métodos de autenticación diferentes (por ejemplo, Formularios y SSO).
En el caso de la autenticación multifactor, habrá tres filas con un identificador de correlación compartido y tres métodos de autenticación correspondientes (por ejemplo, Formularios, AzureMFA y Multifactor). En este ejemplo concreto, la autenticación multifactor muestra que el inicio de sesión único usa un proveedor MFA.

***¿Qué errores pueden verse en el informe?***
Para obtener una lista completa de los errores relacionados con AD FS que aparecen en el informe de inicios de sesión y sus descripciones, consulte la [información sobre códigos de error de la ayuda de AD FS](https://adfshelp.microsoft.com/References/ConnectHealthErrorCodeReference).

***Aparece “00000000-0000-0000-0000-000000000000” en la sección "User" (Usuario) de un inicio de sesión. ¿Qué significa esto?***
Si el inicio de sesión generó un error y el nombre principal de usuario que se probó no coincide con alguno existente, los campos "Usuario", "Nombre de usuario" e "Id. de usuario" serán "00000000-0000-0000-0000-000000000000" y el "identificador de inicio de sesión" se rellenará con el valor que el usuario probó. En estos casos, el usuario que intenta iniciar sesión no existe.

***¿Cómo puedo correlacionar mis eventos locales con el informe de inicios de sesión de Azure AD?***
El agente de Azure AD Connect Health para AD FS correlaciona los identificadores de evento de AD FS en función de la versión del servidor. Los eventos estarán disponibles en el registro de seguridad de los servidores de AD FS. 

***¿Por qué veo NotSet o NotApplicable en el nombre o el identificador de la aplicación para algunos inicios de sesión AD FS?***
El informe de inicios de sesión de AD FS mostrará identificadores de OAuth en el campo Id. de aplicación para inicios de sesión de OAuth. En los escenarios de inicio de sesión de WS-Fed y WS-Trust, el identificador de aplicación será NotSet o NotApplicable y los identificadores de recurso y de persona de confianza estarán presentes en el campo Id. de recurso.

***¿Hay más problemas conocidos con el informe de la versión preliminar?***
El informe tiene un problema conocido que consiste en que el campo "Requisito de autenticación" de la pestaña "Información básica" se rellena como un valor de autenticación de un solo factor para los inicios de sesión de AD FS con independencia del inicio de sesión. Además, la pestaña detalles de autenticación muestra "Principal o secundaria" en el campo Requisito, y hay una corrección en curso para diferenciar los tipos de autenticación principal o secundaria.


## <a name="related-links"></a>Vínculos relacionados
* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Instalación del agente de Azure AD Connect Health](how-to-connect-health-agent-install.md)
* [Informe de direcciones IP de riesgo](how-to-connect-health-adfs-risky-ip.md)