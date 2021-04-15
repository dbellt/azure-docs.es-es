---
title: 'Configuración del teléfono de la oficina como método de verificación en dos fases: Azure Active Directory | Microsoft Docs'
description: Aprenda a configurar el teléfono de la oficina como método de verificación en dos fases.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 08/12/2019
ms.author: curtand
ms.openlocfilehash: c77672ce895cc63a04973c4cb3e752ca319e269d
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106553420"
---
# <a name="set-up-an-office-phone-as-your-two-factor-verification-method"></a>Configuración del teléfono de la oficina como método de verificación en dos fases

Es posible configurar el teléfono de la oficina para que actúe como método de verificación en dos fases.

> [!Note]
> Si la opción de **teléfono de la oficina** no está disponible para su selección, es posible que su organización no le permita usar un número de teléfono de la oficina para la verificación. En este caso, tendrá que seleccionar otro método o ponerse en contacto con su administrador para obtener más ayuda. 
> 
> Los usuarios de registro combinado no verán una opción para usar una extensión con la opción de **teléfono de la oficina**.

## <a name="set-up-your-office-phone-number-as-your-verification-method"></a>Configuración del número de teléfono de la oficina como método de verificación

1. En la página **Comprobación de seguridad adicional**, seleccione **Teléfono de Office** en el área **Paso 1: ¿De qué manera deberíamos ponernos en contacto con usted?** , seleccione su país o región en la lista desplegable y, a continuación, introduzca su extensión, si la tiene.

    ![Página Comprobación de seguridad adicional, con teléfono de autenticación y llamada de teléfono](media/multi-factor-authentication-verification-methods/multi-factor-authentication-office-phone.png)

2. Recibirá una llamada de teléfono de Microsoft en la que se le pedirá que presione el signo de almohadilla (#) en el teléfono de la oficina para verificar su identidad.

    ![Prueba del número de teléfono especificado](media/multi-factor-authentication-verification-methods/multi-factor-authentication-office-phone-test.png)

3. En el área **Paso 3: Siga usando sus aplicaciones existentes** se indicará una contraseña de aplicación; cópiela y péguela en un lugar seguro.

    ![Área de contraseñas de aplicación de la página Comprobación de seguridad adicional](media/multi-factor-authentication-verification-methods/multi-factor-authentication-app-passwords.png)

    >[!Note]
    >Para obtener información sobre el uso de la contraseña de aplicación con las aplicaciones antiguas, consulte [Administración de las contraseñas de aplicaciones](multi-factor-authentication-end-user-app-passwords.md). Solo tiene que usar contraseñas de aplicación si va a seguir usando aplicaciones antiguas que no admiten la verificación en dos fases.

4. Seleccione **Listo**.

## <a name="next-steps"></a>Pasos siguientes

Después de configurar el método de verificación en dos fases, puede agregar otros métodos, administrar la configuración y las contraseñas de aplicación, iniciar sesión u obtener ayuda con algunos problemas comunes relacionados con la verificación en dos fases.

- [Administración de la configuración del método de verificación en dos fases](multi-factor-authentication-end-user-manage-settings.md)

- [Administración de las contraseñas de aplicación](multi-factor-authentication-end-user-app-passwords.md)

- [Inicio de sesión con la verificación en dos pasos](multi-factor-authentication-end-user-signin.md)

- [Obtención de ayuda con la verificación en dos fases](multi-factor-authentication-end-user-troubleshoot.md)
