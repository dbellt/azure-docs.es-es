---
title: 'Registro de autoservicio para usuarios comprobados por correo electrónico: Azure AD | Microsoft Docs'
description: Uso del registro de autoservicio en una organización de Azure Active Directory (Azure AD)
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: enterprise-users
ms.topic: overview
ms.workload: identity
ms.date: 05/19/2021
ms.author: curtand
ms.reviewer: elkuzmen
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: cb1869c48e38bdaf56fe4bd428a7bcbd6d316b85
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110535708"
---
# <a name="what-is-self-service-sign-up-for-azure-active-directory"></a>¿Qué es el registro de autoservicio de Azure Active Directory?

En este artículo se explica cómo usar el registro de autoservicio para rellenar una organización en Azure Active Directory (Azure AD). Si desea asumir un nombre de dominio de una organización de Azure AD no administrada, consulte [Adquisición de un inquilino no administrado como administrador](domains-admin-takeover.md).

## <a name="why-use-self-service-sign-up"></a>¿Por qué usar el registro de autoservicio?

* Permita que los usuarios tengan acceso a los servicios que desean más rápido.
* Cree ofertas basadas en correo electrónico para un servicio.
* Cree flujos de registro basados en correo electrónico que permitan a los usuarios crear identidades rápidamente con sus alias de correo electrónico del trabajo fáciles de recordar.
* Un inquilino de Azure AD creado mediante autoservicio puede convertirse en un inquilino administrado que se puede utilizar para otros servicios.

## <a name="terms-and-definitions"></a>Términos y definiciones

* **Registro de autoservicio**: método por el que un usuario se registra en un servicio en la nube y por el que se le crea automáticamente una identidad en Azure AD basada en su dominio de correo electrónico.
* **Inquilino de Azure AD no administrado**: es el inquilino donde se crea esa identidad. Un inquilino no administrado es un inquilino que no tiene ningún administrador global.
* **Usuario comprobado por correo electrónico**: tipo de cuenta de usuario en Azure AD. Un usuario que tiene una identidad que se crean automáticamente después de suscribirse a una oferta de autoservicio se conoce como usuario comprobado por correo electrónico. Un usuario comprobado por correo electrónico es un miembro regular de un inquilino etiquetado con creationmethod = EmailVerified.

## <a name="how-do-i-control-self-service-settings"></a>¿Cómo controlo la configuración de autoservicio?

Actualmente, los administradores tienen dos controles de autoservicio . Pueden controlar:

* Si los usuarios pueden unirse o no al inquilino mediante el correo electrónico.
* Si los usuarios pueden concederse o no licencias para aplicaciones y servicios.

### <a name="how-can-i-control-these-capabilities"></a>¿Cómo puedo controlar estas capacidades?

Un administrador puede configurar estas capacidades con estos parámetros Set-MsolCompanySettings de cmdlet de Azure AD:

* **AllowEmailVerifiedUsers** controla si los usuarios pueden unirse al inquilino mediante validación por correo electrónico. Para unirse, el usuario debe tener una dirección de correo electrónico en un dominio que coincida con uno de los dominios comprobados del inquilino. Esta configuración se aplica a toda la empresa para todos los dominios del inquilino. Si establece este parámetro en $false, ningún usuario verificado por correo electrónico podrá unirse al inquilino.
* **AllowAdHocSubscriptions** controla la capacidad de los usuarios de realizar registros de autoservicio. Si establece el parámetro en $false, ningún usuario podrá realizar registros de autoservicio.
  
AllowEmailVerifiedUsers y AllowAdHocSubscriptions son configuraciones de todo el inquilino que se pueden aplicar a un inquilino administrado o no administrado. A continuación, se muestra un ejemplo en el que:

* Administra un inquilino con un dominio verificado, como contoso.com.
* Usa colaboración B2B de un inquilino diferente para invitar a un usuario que aún no existe (userdoesnotexist@contoso.com) en el inquilino principal de contoso.com.
* El inquilino principal tiene la opción AllowEmailVerifiedUsers activada.

Si se cumplen las condiciones anteriores, a continuación, se crea un usuario miembro en el inquilino principal y un usuario invitado B2B en el inquilino que invita.

Para más información sobre las suscripciones de Flow y PowerApps, consulte los siguientes artículos:

* [¿Cómo puedo impedir que mis usuarios existentes comiencen a utilizar Power BI?](https://support.office.com/article/Power-BI-in-your-Organization-d7941332-8aec-4e5e-87e8-92073ce73dc5#bkmk_preventjoining)
* [Preguntas y respuestas sobre Flow en su organización](/flow/organization-q-and-a)

### <a name="how-do-the-controls-work-together"></a>¿Cómo funciona los controles conjuntamente?
Estos dos parámetros se pueden usar juntos para definir un control más preciso del registro de autoservicio. Por ejemplo, el comando siguiente permitirá a los usuarios realizar registros de autoservicio, pero solo si estos usuarios ya tienen una cuenta en Azure AD (es decir, los usuarios que necesitan crear primero una cuenta comprobada por correo electrónico no pueden realizar registros de autoservicio):

```powershell
    Set-MsolCompanySettings -AllowEmailVerifiedUsers $false -AllowAdHocSubscriptions $true
```

En el siguiente diagrama de flujo se explican las distintas combinaciones de estos parámetros y las condiciones resultantes para el inquilino y el registro de autoservicio.

![diagrama de flujo de los controles del registro de autoservicio](./media/directory-self-service-signup/SelfServiceSignUpControls.png)

Los detalles de esta configuración se pueden recuperar mediante el cmdlet de PowerShell Get-MsolCompanyInformation siguiente. Para más información, consulte [Get-MsolCompanyInformation](/powershell/module/msonline/get-msolcompanyinformation).

```powershell
    Get-MsolCompanyInformation | Select AllowEmailVerifiedUsers, AllowAdHocSubscriptions
```

Para obtener más información y ejemplos de cómo usar estos parámetros, consulte [Set-MsolCompanySettings](/powershell/module/msonline/set-msolcompanysettings).

## <a name="next-steps"></a>Pasos siguientes

* [Incorporación de su nombre de dominio personalizado a Azure Active Directory](../fundamentals/add-custom-domain.md)
* [Instalación y configuración de Azure PowerShell](/powershell/azure/)
* [Azure PowerShell](/powershell/azure/)
* [Referencia de cmdlets de Azure](/powershell/azure/get-started-azureps)
* [Set-MsolCompanySettings](/powershell/module/msonline/set-msolcompanysettings)
* [Cierre de la cuenta profesional o educativa en un inquilino no administrado](users-close-account.md)
