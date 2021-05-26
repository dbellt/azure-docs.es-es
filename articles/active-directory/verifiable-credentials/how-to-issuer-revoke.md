---
title: 'Procedimiento para revocar una credencial verificable como emisor: credenciales verificables de Azure Active Directory'
description: Procedimiento para revocar una credencial verificable que ha emitido
documentationCenter: ''
author: barclayn
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.subservice: verifiable-credentials
ms.date: 04/01/2021
ms.author: barclayn
ms.openlocfilehash: 86a1683751b76b0b6e08b71fdeccaae47d88ffc3
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110466135"
---
# <a name="revoke-a-previously-issued-verifiable-credential-preview"></a>Revocación de una credencial verificable emitida previamente (versión preliminar)

Como parte del proceso de trabajo con credenciales verificables (VC), no solo tendrá que emitirlas. En ocasiones, también tendrá que revocarlas. En este artículo se analiza la parte de la propiedad **Status** de la especificación VC y se examina con más detalle el proceso de revocación, por qué puede que deseemos revocar las credenciales, y algunas implicaciones en relación con los datos y la privacidad.

> [!IMPORTANT]
> Las credenciales verificables de Azure Active Directory se encuentran actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="status-property-in-verifiable-credentials-specification"></a>Propiedad Status en la especificación de credenciales verificables

Antes de comprender las implicaciones de revocar una credencial verificable, puede que le ayude saber qué es la **comprobación de estado** y cómo funciona actualmente.

La [especificación de credenciales verificables W3C](https://www.w3.org/TR/vc-data-model/) hace referencia a la propiedad **status** de la sección [4.9:](https://www.w3.org/TR/vc-data-model/#status)

"Esta especificación define la siguiente propiedad **credentialStatus** para la detección de información sobre el estado actual de una credencial verificable. Por ejemplo, si se ha suspendido o revocado".

Sin embargo, la especificación W3C no define la forma de implementar la **comprobación de estado**.

"Definir el modelo de datos, los formatos y los protocolos para los esquemas de estado quedan fuera del ámbito de esta especificación. Existe un registro de extensión de credenciales verificables [VC-EXTENSION-REGISTRY] que contiene los esquemas de estado disponibles para los implementadores que deseen habilitar la comprobación de estado de credenciales verificables".

>[!NOTE]
>Por ahora, la implementación de la comprobación de estado de Microsoft es una funcionalidad propietaria, pero estamos trabajando de forma activa con la comunidad para seguir un estándar.

## <a name="how-does-the-status-property-work"></a>¿Cómo funciona la propiedad **status**?

En cada credencial verificable emitidas por Microsoft, hay un atributo denominado credentialStatus. Este atributo se rellena con una API de estado que Microsoft administra en su nombre. A continuación se muestra un ejemplo del aspecto que tiene.

```json
    "credentialStatus": {
      "id": "https://portableidentitycards.azure-api.net/v1.0/7952032d-d1f3-4c65-993f-1112dab7e191/portableIdentities/card/status",
      "type": "PortableIdentityCardServiceCredentialStatus2020"
    }
```

El SDK de credenciales verificables de código abierto se encarga de llamar a la API de estado y proporcionar los datos necesarios.

Una vez que se llama a la API y se proporciona la información correcta, la API devolverá "True" (Verdadero) o "False" (Falso). "True" (Verdadero) es la credencial verificable que está todavía activa con el emisor; y "False" (Falso) significa que el emisor ha revocado activamente la credencial verificable.

## <a name="why-you-may-want-to-revoke-a-vc"></a>¿Por qué se podría querer revocar una credencial verificable?

Cada cliente tendrá su propia razón para revocar una credencial verificable, pero estos son algunos de los motivos comunes que hemos oído hasta ahora. 

- Identificador de alumno: el alumno ya no está en activo en la universidad.
- Identificador de empleado: el empleado ya no está en activo en la empresa.
- Permiso de conducir estatal: el conductor ya no reside en ese estado.

## <a name="how-to-set-up-a-verifiable-credential-with-the-ability-to-revoke"></a>Cómo configurar una credencial verificable con la posibilidad de revocación

No todos los datos de credenciales verificables se almacenan con Microsoft de forma predeterminada. Por lo tanto, no tenemos ningún dato al que hacer referencia para revocar un identificador de credencial verificable específico. El emisor debe especificar un campo concreto del atributo de credencial verificable para que Microsoft lo indexe y, posteriormente, lo cifre con sal y use un algoritmo hash.

>[!NOTE]
>Un algoritmo hash es una operación criptográfica unidireccional que convierte una entrada, llamada ```preimage```, y produce una salida llamada "hash" que tiene una longitud fija. En este momento, no es factible desde una perspectiva computacional invertir una operación hash.

Puede indicar a Microsoft qué atributo de la credencial verificable le gustaría indexar. La implicación de la indexación es que los valores indizados se pueden usar para buscar en las credenciales verificables aquellas que desea revocar.

**Ejemplo:** Alicia era empleada de Woodgrove. Dejó Woodgrove para trabajar en Contoso. Julia, la administradora de TI de Woodgrove, busca el correo electrónico de Alicia en la consulta de búsqueda de revocación de credenciales verificables. En este ejemplo, Julia indexa el campo de correo electrónico de la credencial de empleado verificado de Woodgrove. 

A continuación se muestra un ejemplo de cómo se modifica el archivo de reglas para incluir el índice.

```json
{
  "attestations": {
    "idTokens": [
      { 
        "mapping": {
          "Name": { "claim": "name" },
          "email": { "claim": "email", "indexed": true}
        },
        "configuration": "https://login.microsoftonline.com/tenant-id-here7/v2.0/.well-known/openid-configuration",
        "client_id": "c0d6b785-7a08-494e-8f63-c30744c3be2f",
        "redirect_uri": "vcclient://openid"
      }
    ]
  },
  "validityInterval": 25920000,
  "vc": {
    "type": ["WoodgroveEmployee"]
  }
}
```

>[!NOTE]
>Solo se puede indexar un atributo del archivo de reglas.  

## <a name="how-do-i-revoke-a-verifiable-credential"></a>¿Cómo se revoca una credencial verificable?

Una vez que se ha establecido una notificación de índice y se han emitido credenciales verificables a sus usuarios, es el momento de ver cómo revocar una credencial verificable en la hoja de credenciales verificables.

1. Vaya a la hoja credenciales verificables de Azure Active Directory.
1. Elija las credenciales verificables en las que ha configurado previamente la notificación de índice y ha emitido una credencial verificable para un usuario. =
1. En el menú de la izquierda, seleccione **Revoke a credential** (Revocar una credencial). 
   ![Revocar una credencial](media/how-to-issuer-revoke/settings-revoke.png) 
1. Busque el atributo de índice del usuario que desea revocar. 

   ![Buscar la credencial que se va a revocar](media/how-to-issuer-revoke/revoke-search.png)

    >[!NOTE]
    >Dado que únicamente estamos almacenando un hash de la notificación indizada de la credencial verificable, solo una coincidencia exacta se incluirá en los resultados de la búsqueda. Tomamos la entrada tal y como la buscó el administrador de TI y utilizamos el mismo algoritmo hash para ver si obtenemos una coincidencia de hash en nuestra base de datos.
    
1. Una vez que identifique una coincidencia, seleccione la opción **Revoke** (Revocar), situada a la derecha de la credencial que desea revocar.

   ![Una advertencia que informa de que el usuario todavía tiene la credencial después de la revocación](media/how-to-issuer-revoke/warning.png) 

1. Una vez que la revocación se efectúe correctamente, verá la actualización de estado y aparecerá un banner verde en la parte superior de la página. 
   ![Comprobación de este dominio en la configuración](media/how-to-issuer-revoke/revoke-successful.png) 

Ahora, cada vez que un usuario de confianza llame a para comprobar el estado de esta credencial verificable específica, la API de estado de Microsoft devolverá una respuesta "False" (Falso) en nombre del inquilino.

## <a name="next-steps"></a>Pasos siguientes

Pruebe la funcionalidad por su cuenta con una credencial de prueba para familiarizarse con el flujo. Puede ver información sobre cómo configurar el inquilino para emitir credenciales verificables en [nuestros tutoriales](get-started-verifiable-credentials.md).