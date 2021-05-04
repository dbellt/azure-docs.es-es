---
title: Solución de problemas de acceso al portal de Azure EA
description: En este artículo se describen algunos problemas comunes que pueden producirse en una Contrato Enterprise de Azure (EA) en el portal de Azure EA.
author: bandersmsft
ms.author: banders
ms.date: 03/26/2021
ms.topic: troubleshooting
ms.service: cost-management-billing
ms.subservice: enterprise
ms.reviewer: boalcsva
ms.openlocfilehash: 852fb85607318772870a8b6826c934997b84ff6d
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107726473"
---
# <a name="troubleshoot-azure-ea-portal-access"></a>Solución de problemas de acceso al portal de Azure EA

En este artículo se describen algunos problemas comunes que pueden producirse en una Contrato Enterprise de Azure (EA). El portal del Contrato Enterprise de Azure se utiliza para administrar los costos y usuarios del Contrato Enterprise. Al configurar o actualizar el acceso al portal del Contrato Enterprise de Azure es posible que surjan estos problemas.

## <a name="issues-adding-an-admin-to-an-enrollment"></a>Problemas al agregar un administrador a una inscripción

Hay diferentes tipos de niveles de autenticación para las inscripciones empresariales. Si los niveles de autenticación se aplican de forma incorrecta, es posible que tenga problemas al intentar iniciar sesión en el portal del Contrato Enterprise de Azure.

El portal del Contrato Enterprise de Azure se usa para conceder acceso a los usuarios con distintos niveles de autenticación. Un administrador de empresa puede actualizar el nivel de autenticación para cumplir los requisitos de seguridad de su organización.

### <a name="authentication-level-types"></a>Tipos de nivel de autenticación

- Solo cuenta de Microsoft: para organizaciones que deseen usar, crear y administrar usuarios mediante cuentas de Microsoft.
- Cuenta profesional o educativa: para organizaciones que han configurado Active Directory con Federación en la nube y todas las cuentas están en un solo inquilino.
- Cuenta profesional o educativa entre inquilinos: para organizaciones que han configurado Active Directory con Federación en la nube y van a tener cuentas en varios inquilinos.
- Cuenta mixta: permite agregar usuarios con una cuenta Microsoft o con una cuenta profesional o educativa.

La primera cuenta profesional o educativa agregada a la inscripción determina el dominio _predeterminado_. Para agregar una cuenta profesional o educativa con otro inquilino, es preciso cambiar el nivel de autenticación de la inscripción a autenticación entre inquilinos.

Para actualizar el nivel de autenticación:

1. Inicie sesión en el portal del Contrato Enterprise de Azure como administrador de empresa.
2. Haga clic en **Administrar** en el panel de navegación izquierdo.
3. Haga clic en la pestaña **Inscripción**.
4. En **Detalles de la inscripción**, seleccione **Auth Level** (Nivel de autenticación).
5. Haga clic en el símbolo del lápiz.
6. Haga clic en **Save**(Guardar).

![Ejemplo que muestra los niveles de autenticación ](./media/ea-portal-troubleshoot/create-ea-authentication-level-types.png)

Las cuentas de Microsoft deben tener un identificador asociado creado en [https://signup.live.com](https://signup.live.com/).

Las cuentas profesionales o educativas están disponibles para las organizaciones que hayan configurado Active Directory con federación y donde todas las cuentas estén en un solo inquilino. Los usuarios se pueden agregar con autenticación de usuario federado profesional o educativa si la instancia de Active Directory interna de la empresa está federado.

Si la organización no usa la federación de Active Directory, no puede usar su dirección de correo electrónico profesional o educativa. En su lugar, registre o cree una nueva dirección de correo electrónico y regístrela como cuenta de Microsoft.

## <a name="unable-to-access-the-azure-ea-portal"></a>No se puede acceder al portal del Contrato Enterprise de Azure

Si recibe un mensaje de error al intentar iniciar sesión en el portal del Contrato Enterprise de Azure, use los siguientes pasos para solucionarlo:

- Asegúrese de que usa la dirección URL correcta del portal del Contrato Enterprise de Azure, que es https://ea.azure.com.
- Determine si el acceso al portal del Contrato Enterprise de Azure se ha agregado como una cuenta profesional o educativa, o bien como cuenta de Microsoft.
  - Si usa una cuenta profesional, escriba su correo electrónico y contraseña profesionales. La contraseña se la proporciona su organización. Si tiene algún problema, puede consultar al departamento de TI cómo restablecer la contraseña.
  - Si utiliza una cuenta de Microsoft, escriba su dirección de correo electrónico y contraseña de la cuenta de Microsoft. Si ha olvidado la contraseña de cuenta de Microsoft, puede restablecerla en [https://account.live.com/password/reset](https://account.live.com/password/reset).
- Use una sesión InPrivate o de incógnito en el explorador para iniciar sesión, con el fin de que no se guarden cookies ni la información almacenada en la caché de las sesiones anteriores o existentes. Borre la memoria caché del explorador y use una ventana InPrivate o de incógnito para abrir https://ea.azure.com.
- Si recibe el error _Usuario no válido_  al usar un cuenta de Microsoft, podría deberse a que tiene varias cuentas de Microsoft. La dirección con la que intenta iniciar sesión no es la principal.
También aparecería el error _Usuario no válido_ si se ha usado un tipo de cuenta incorrecto al agregar el usuario a la inscripción. Por ejemplo, una cuenta profesional o educativa, en lugar de una cuenta de Microsoft. En este ejemplo, pida a administrador de EA que agregue la cuenta correcta o póngase en contacto con el [servicio de soporte técnico](https://support.microsoft.com/supportforbusiness/productselection?sapId=cf791efa-485b-95a3-6fad-3daf9cd4027c).
  - Si necesita comprobar el alias principal, vaya a [https://account.live.com](https://account.live.com). Luego, haga clic en **Your Info** (Tu información) y, después, en **Manage how to sign in to Microsoft** (Administrar el modo de iniciar sesión en Microsoft). Siga las indicaciones para comprobar otra dirección de correo electrónico y obtener un código para acceder a la información confidencial. Escriba el código de seguridad. Seleccione **Configurarlo más tarde**  si no desea configurar la autenticación en dos fases.
  - Verá la página **Manage how to sign in to Microsoft** (Administrar el modo de iniciar sesión en Microsoft), donde podrá ver los alias de cuenta que tiene. Compruebe que el alias principal es el que usa para iniciar sesión en el portal de Azure EA. Si no lo es, puede convertirlo en el alias principal. O bien, puede usar el alias principal del portal del Contrato Enterprise de Azure.

## <a name="next-steps"></a>Pasos siguientes

- Los administradores del portal del Contrato Enterprise de Azure deben leer el artículo acerca de la [administración del portal del Contrato Enterprise de Azure](ea-portal-administration.md) para obtener información sobre las tareas administrativas comunes.
- Lea las [preguntas frecuentes sobre Azure Cost Management + Billing](../cost-management-billing-faq.yml), donde encontrará respuestas a los problemas comunes que surgen al activar Azure EA.
