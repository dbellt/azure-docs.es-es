---
title: 'Preguntas de seguridad del método de autenticación: Azure Active Directory'
description: Obtenga información sobre el uso de preguntas de seguridad en Azure Active Directory para ayudar a mejorar y proteger los eventos de inicio de sesión.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 09/02/2020
ms.author: justinha
author: justinha
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: 841391778e0fb8c00f503aa0cc79b5562661e309
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530353"
---
# <a name="authentication-methods-in-azure-active-directory---security-questions"></a>Métodos de autenticación en Azure Active Directory: preguntas de seguridad

Las preguntas de seguridad no se usan como método de autenticación durante un evento de inicio de sesión. En su lugar, las preguntas de seguridad se pueden usar durante el proceso de autoservicio de restablecimiento de contraseña (SSPR) para confirmar quién es. Las cuentas de administrador no pueden usar preguntas de seguridad como método de verificación con SSPR.

Cuando los usuarios se registren en SSPR, se les pedirá que elijan los métodos de autenticación que usarán. Si optan por usar preguntas de seguridad, pueden elegir entre un conjunto de preguntas para que se muestren y, a continuación, proporcionar sus propias respuestas.

![Captura de pantalla de Azure Portal, que muestra los métodos de autenticación y las opciones para las preguntas de seguridad](media/concept-authentication-methods/security-questions-authentication-method.png)

> [!NOTE]
> Las preguntas de seguridad se almacenan de manera privada y segura en un objeto de usuario del directorio y solo las pueden responder los usuarios durante el registro. No hay forma de que un administrador lea o modifique las preguntas o respuestas de un usuario.

Las preguntas de seguridad pueden ser menos seguras que otros métodos porque es posible que algunas personas sepan las respuestas de las preguntas de otro usuario. Si usa preguntas de seguridad con SSPR, se recomienda usarlas junto con otro método. Se puede solicitar al usuario usar la aplicación Microsoft Authenticator o la autenticación por teléfono para verificar su identidad durante el proceso de autoservicio de restablecimiento de contraseña, y elegir preguntas de seguridad solo si no tiene su teléfono o dispositivo registrado consigo.

## <a name="predefined-questions"></a>Preguntas predefinidas

Las siguientes preguntas de seguridad predefinidas están disponibles para su uso como método de verificación con SSPR. Todas estas preguntas de seguridad están traducidas y localizadas al conjunto completo de idiomas de Microsoft 365 en función de la configuración regional del explorador del usuario:

* ¿En qué ciudad conoció a su cónyuge o pareja?
* ¿En qué ciudad se conocieron sus padres?
* ¿En qué ciudad vive su hermano más próximo?
* ¿En qué ciudad nació su padre?
* ¿En qué ciudad tuvo su primer trabajo?
* ¿En qué ciudad nació su madre?
* ¿En qué ciudad estaba en la Nochevieja del año 2000?
* ¿Cuál es el apellido de su profesor favorito del instituto?
* ¿En qué universidad solicitó plaza pero no asistió?
* ¿Cómo se llama el lugar en el que celebró su primer matrimonio?
* ¿Cuál es el segundo apellido de su padre?
* ¿Cuál es su comida favorita?
* ¿Cuál es el nombre y apellido de su abuela materna?
* ¿Cuál es el segundo apellido de su madre?
* ¿Cuáles son el año y mes de nacimiento del mayor de sus hermanos? (por ejemplo, noviembre de 1985)
* ¿Cuál es el segundo apellido del mayor de sus hermanos?
* ¿Cuál es el nombre y apellido de su abuelo paterno?
* ¿Cuántos años se lleva con el menor de sus hermanos?
* ¿En qué escuela cursó el sexto curso?
* ¿Cuál era el nombre y apellido de su mejor amigo de la infancia?
* ¿Cuál era el nombre y apellido de su primera pareja?
* ¿Cuál era el nombre de su maestro de primaria favorito?
* ¿Cuál era la marca y modelo de su primer coche o moto?
* ¿Cómo se llamaba la primera escuela a la que asistió?
* ¿Cómo se llamaba el hospital en el que nació?
* ¿Cuál era la calle de su primera casa de la infancia?
* ¿Cuál era el nombre de su héroe de la infancia?
* ¿Cuál era el nombre de su peluche favorito?
* ¿Cuál era el nombre de su primera mascota?
* ¿Cuál era su apodo en la infancia?
* ¿Cuál era su deporte favorito en el instituto?
* ¿Cuál fue su primer trabajo?
* ¿Cuáles eran los cuatro últimos números de su teléfono de la infancia?
* Cuando era joven, ¿qué quería ser de mayor?
* ¿Cuál es la persona más famosa que ha conocido?

## <a name="custom-security-questions"></a>Preguntas de seguridad personalizadas

Para mayor flexibilidad, puede definir sus propias preguntas de seguridad personalizadas. La longitud máxima de una pregunta de seguridad personalizada es 200 caracteres.

Las preguntas de seguridad personalizadas no se localizan automáticamente como sí lo están las preguntas de seguridad predeterminadas. Todas las preguntas personalizadas se muestran en el mismo idioma en el que se han escrito en la interfaz de usuario administrativa, aunque la configuración regional del explorador del usuario sea otra. Si necesita preguntas localizadas, debería usar las preguntas predefinidas.

## <a name="security-question-requirements"></a>Requisitos de las preguntas de seguridad

En el caso tanto de las preguntas de seguridad predeterminadas como de las personalizadas, se aplican los siguientes requisitos y limitaciones:

* El límite mínimo de caracteres de las respuestas es de tres.
* El límite máximo de caracteres de las respuestas es de 40.
* Los usuarios no pueden responder a la misma pregunta más de una vez.
* Los usuarios no pueden proporcionar la misma respuesta a más de una pregunta.
* Se puede usar cualquier juego de caracteres para definir las preguntas y respuestas, incluidos los caracteres Unicode.
* El número de preguntas que se definen debe ser mayor o igual que el número de preguntas que fueron necesarias para registrarse.

## <a name="next-steps"></a>Pasos siguientes

Para comenzar, consulte [Tutorial del autoservicio de restablecimiento de contraseña (SSPR)][tutorial-sspr].

Para obtener más información sobre los conceptos del SSPR, consulte [Funcionamiento: Autoservicio de restablecimiento de contraseña de Azure AD][concept-sspr].

Más información sobre la configuración de métodos de autenticación con la [API REST de Microsoft Graph](/graph/api/resources/authenticationmethods-overview).

<!-- INTERNAL LINKS -->
[tutorial-sspr]: tutorial-enable-sspr.md
[concept-sspr]: concept-sspr-howitworks.md
