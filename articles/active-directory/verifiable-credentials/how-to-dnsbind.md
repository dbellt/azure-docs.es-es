---
title: 'Vinculación del dominio a su identificador descentralizado (DID) (versión preliminar): credenciales verificables de Azure Active Directory'
description: Información sobre el enlace DNS
documentationCenter: ''
author: barclayn
manager: daveba
ms.service: identity
ms.topic: how-to
ms.subservice: verifiable-credentials
ms.date: 04/01/2021
ms.author: barclayn
ms.openlocfilehash: 90ea52b0ed5ee2d8e36caab18491eecd6e1295fd
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106222822"
---
# <a name="link-your-domain-to-your-decentralized-identifier-did"></a>Vinculación del dominio a su identificador descentralizado (DID)

> [!IMPORTANT]
> Las credenciales verificables de Azure Active Directory se encuentran actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo:
> [!div class="checklist"]
> * ¿Por qué es necesario vincular nuestro DID al dominio?
> * ¿Cómo se vinculan los DID y los dominios?
> * ¿Cómo funciona el proceso de vinculación del dominio?
> * ¿Cómo funciona la lógica de dominio comprobado/no comprobado?

## <a name="prerequisites"></a>Requisitos previos

Para vincular su DID al dominio, debe haber completado lo siguiente.

- La [Introducción](get-started-verifiable-credentials.md) y el [conjunto de tutoriales](enable-your-tenant-verifiable-credentials.md) posteriores.

## <a name="why-do-we-need-to-link-our-did-to-our-domain"></a>¿Por qué es necesario vincular nuestro DID al dominio?

Un DID se inicia como un identificador que no está anclado a los sistemas existentes. Un DID es útil porque un usuario u organización puede ser su propietario y controlarlo. Si una entidad que interactúa con la organización no sabe a "quién" pertenece el DID, este no es útil.

Al vincular un DID a un dominio se resuelve el problema de confianza inicial, ya que permite que cualquier entidad compruebe criptográficamente la relación entre un DID y un dominio.

## <a name="how-do-we-link-dids-and-domains"></a>¿Cómo se vinculan los DID y los dominios?

Un vínculo entre un dominio y un DID se crea mediante la implementación de un estándar abierto escrito por Decentralized Identity Foundation, denominado [Well-Known DID configuration](https://identity.foundation/.well-known/resources/did-configuration/) (configuración de DID conocida). El servicio de credenciales verificables de Azure Active Directory (Azure AD) ayuda a la organización a crear el vínculo entre el DID y el dominio mediante la información de dominio incluida que proporcionó en el DID y la generación del archivo de configuración conocida:

1. Azure AD usa la información de dominio que se proporciona durante la configuración de la organización para escribir un punto de conexión de servicio en el documento DID. Todas las partes que interactúan con el DID pueden ver el dominio con el que el DID declara estar asociado.  

    ```json
        "service": [
          {
            "id": "#linkeddomains",
            "type": "LinkedDomains",
            "serviceEndpoint": {
              "origins": [
                "https://www.contoso.com/"
              ]
            }
          }
    ```

2. El servicio de credenciales verificables de Azure AD genera un recurso de configuración conocida compatible que se puede hospedar en el dominio. El archivo de configuración incluye una credencial verificable que se emite automáticamente con credentialType "DomainLinkageCredential" firmada con el DID que tiene el origen de su dominio. Este es un ejemplo del documento de configuración que se almacena en la dirección URL del dominio raíz.


    ```json
    {
      "@context": "https://identity.foundation/.well-known/contexts/did-configuration-v0.0.jsonld",
      "linked_dids": [
        "jwt..."
      ]
    }
    ```

Una vez que tenga el archivo de configuración conocida, deberá hacer que el archivo esté disponible con el nombre de dominio que especificó al habilitar AAD para las credenciales verificables.

* Hospede el archivo de configuración de DID conocida en la raíz del dominio.
* No use redirecciones.
* Use HTTPS para distribuir el archivo de configuración.

>[!IMPORTANT]
>Microsoft Authenticator no respeta las redirecciones, la dirección URL especificada debe ser la dirección URL de destino final.

## <a name="user-experience"></a>Experiencia del usuario 

Cuando un usuario pasa por un flujo de emisión o presenta una credencial verificable, debe tener alguna información sobre la organización y su DID. Si nuestra cartera de credenciales verificables del dominio, Microsoft Authenticator, valida una relación del DID con el dominio del documento DID, presenta a los usuarios dos experiencias diferentes en función del resultado.

## <a name="verified-domain"></a>Dominio comprobado

Antes de que Microsoft Authenticator muestre el icono **Comprobado**, deben cumplirse algunas condiciones:

* La firma del DID de la solicitud de Open ID emitido automáticamente (SIOP) debe tener un punto de conexión de servicio para el dominio vinculado.
* El dominio raíz no usa redireccionamiento y usa HTTPS.
* El dominio que aparece en el documento DID tiene un recurso conocido que se puede resolver.
* La credencial verificable del recurso conocido está firmada con el mismo DID que se usó para firmar el SIOP que Microsoft Authenticator usó para iniciar el flujo.

Si se cumplen todas las condiciones mencionadas anteriormente, Microsoft Authenticator muestra una página de comprobación e incluye el dominio que se validó.

![Nueva solicitud de permiso](media/how-to-dnsbind/new-permission-request.png) 

## <a name="unverified-domain"></a>Dominio sin comprobar

Si no se cumple alguno de los puntos anteriores, Microsoft Authenticator mostrará una advertencia de una página completa al usuario de que el dominio no está comprobado, que está realizando una transacción de riesgo y que debe proseguir con precaución. Hemos elegido esta ruta porque:

* El DID no está anclado a un dominio.
* La configuración no se configuró correctamente.
* El DID con el que interactúa el usuario es malintencionado y, en realidad, no puede probar la propiedad de un dominio (ya que realmente no la tiene). Debido a este último punto, es imprescindible que vincule su DID al dominio con el que está familiarizado el usuario, para evitar que se propague el mensaje de advertencia.

![Advertencia de dominio no comprobado en la pantalla de adición de credenciales](media/how-to-dnsbind/add-credential-not-verified-authenticated.png)

## <a name="distribute-well-known-config"></a>Distribución de configuración conocida

1. Vaya a la página de configuración en credenciales verificables y elija **Verify this domain** (Comprobar este dominio).

   ![Comprobación de este dominio en la configuración](media/how-to-dnsbind/settings-verify.png) 

2. Descargue el archivo did-configuration.json que se muestra en la imagen siguiente.

   ![Descarga de la configuración conocida](media/how-to-dnsbind/verify-download.png) 

3. Copie el JWT, abra [jwt.ms](https://www.jwt.ms) y valide que el dominio es correcto.

4. Copie el DID y abra el [explorador de red ION](https://identity.foundation/ion/explorer) para comprobar que se incluye el mismo dominio en el documento DID. 

5. Hospede el recurso de configuración conocida en la ubicación especificada. Ejemplo: https://www.example.com/.well-known/did-configuration.json

6. Pruebe la emisión o la presentación a Microsoft Authenticator para la validación. Asegúrese de que la configuración de Authenticator "Warn about unsafe apps" (Advertir sobre aplicaciones no seguras) está activada.

>[!NOTE]
>De forma predeterminada, "Warn about unsafe apps" (Advertir sobre aplicaciones no seguras) está activada.

Enhorabuena, ahora ha arrancado la web de confianza con su DID.

## <a name="next-steps"></a>Pasos siguientes

Si durante la incorporación se especifica información de dominio incorrecta que decide cambiar, deberá [no participar](how-to-opt-out.md). En este momento, no se admite la actualización del documento DID. Si abandona y regresa, se creará otro DID completamente nuevo.