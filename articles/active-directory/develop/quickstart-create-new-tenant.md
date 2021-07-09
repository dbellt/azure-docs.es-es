---
title: 'Inicio rápido: Creación de un inquilino de Azure Active Directory'
titleSuffix: Microsoft identity platform
description: En esta guía de inicio rápido, aprenderá a crear un inquilino de Azure Active Directory para usarlo en el desarrollo de aplicaciones que empleen la plataforma de identidad de Microsoft para la autenticación y la autorización.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: quickstart
ms.date: 02/15/2021
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev, identityplatformtop40, fasttrack-edit
ms.openlocfilehash: 9ef84b1ac1a1d7f1b1c9c629c4e6eb2606f2441c
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111854853"
---
# <a name="quickstart-set-up-a-tenant"></a>Inicio rápido: Configuración de un inquilino

Para crear aplicaciones que usen la plataforma de identidad de Microsoft para la administración de identidades y acceso, necesita acceder a un *inquilino* de Azure Active Directory (Azure AD). Está en el inquilino de Azure AD que registra y administra sus aplicaciones, configura su acceso a los datos en Microsoft 365 y otras API web y habilita características como el acceso condicional.

Un inquilino representa una organización. Se trata de una instancia dedicada de Azure AD que una organización o un desarrollador de aplicaciones recibe al principio de su relación con Microsoft. Esa relación podría empezar, por ejemplo, por el registro en Azure, Microsoft Intune o Microsoft 365.

Cada inquilino de Azure AD es distinto e independiente de los demás inquilinos de Azure AD. Tiene su propia representación de identidades profesionales y educativas, identidades de consumidor (si es un inquilino de Azure AD B2C) y registros de aplicaciones. Si la aplicación se registra en el inquilino, se puede permitir la autenticación solo de cuentas del inquilino o de todos los inquilinos.

## <a name="prerequisites"></a>Requisitos previos

Una cuenta de Azure que tenga una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="determining-the-environment-type"></a>Determinación del tipo de entorno

Puede crear dos tipos de entorno. El entorno depende únicamente de los tipos de usuarios que vaya a autenticar la aplicación. 

Este inicio rápido trata dos escenarios para el tipo de aplicación que quiere crear:

* Cuentas profesionales o educativas (de Azure AD) o cuentas de Microsoft (como outlook.com y live.com)
* Cuentas locales y sociales (Azure AD B2C)

## <a name="work-and-school-accounts-or-personal-microsoft-accounts"></a>Cuentas profesionales o educativas o cuentas personales de Microsoft

Para crear un entorno para cuentas profesionales y educativas o cuentas personales de Microsoft, puede usar un inquilino de Azure AD existente o crear uno.
### <a name="use-an-existing-azure-ad-tenant"></a>Uso de un inquilino de Azure AD existente

Muchos desarrolladores ya tienen inquilinos mediante servicios o suscripciones vinculados a inquilinos de Azure AD, como suscripciones de Microsoft 365 o Azure.

Para comprobar el inquilino:

1. Inicie sesión en <a href="https://portal.azure.com/" target="_blank">Azure Portal</a>. Use la cuenta que usará para administrar la aplicación.
1. Compruebe la esquina superior derecha. Si tiene un inquilino, iniciará sesión automáticamente. Verá el nombre del inquilino directamente debajo del nombre de la cuenta.
   * Mantenga el puntero sobre el nombre de la cuenta para ver su nombre, dirección de correo electrónico, directorio o identificador de inquilino (GUID) y dominio.
   * Si la cuenta está asociada a varios inquilinos, puede seleccionar el nombre de la cuenta para abrir un menú donde puede cambiar entre los inquilinos. Cada inquilino tiene su propio identificador de inquilino.

> [!TIP]
> Para encontrar el identificador del inquilino, puede:
> * Mantener el puntero sobre el nombre de la cuenta para obtener el directorio o el identificador de inquilino.
> * Buscar y seleccionar **Azure Active Directory** > **Propiedades** > **Id. de inquilino** en Azure Portal.

Si no tiene un inquilino asociado a la cuenta, verá un GUID debajo del nombre de la cuenta. No podrá realizar acciones, como registrar aplicaciones, hasta que haya creado un inquilino de Azure AD.

### <a name="create-a-new-azure-ad-tenant"></a>Creación de un nuevo inquilino de Azure AD

Si aún no tiene un inquilino de Azure AD o si desea crear uno para el desarrollo, consulte [Creación de un inquilino en Azure AD](../fundamentals/active-directory-access-create-new-tenant.md). O bien, use la [experiencia de creación de directorios](https://portal.azure.com/#create/Microsoft.AzureActiveDirectory) de Azure Portal. 

Tendrá que proporcionar la información siguiente para crear el inquilino:

- **Nombre de la organización**
- **Dominio inicial**: el dominio inicial `<domainname>.onmicrosoft.com` no se puede editar ni eliminar. Puede agregar un nombre de dominio personalizado más adelante.
- **País o región**

> [!NOTE]
> Al asignar un nombre a su inquilino, utilice caracteres alfanuméricos. No se admiten caracteres especiales. El nombre no puede tener más de 256 caracteres.

## <a name="social-and-local-accounts"></a>Cuentas locales y sociales

Para empezar a crear aplicaciones que inician sesión en cuentas sociales y locales, cree un inquilino de Azure AD B2C. Para empezar, consulte [Creación de un inquilino de Azure AD B2C](../../active-directory-b2c/tutorial-create-tenant.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Registre una aplicación](quickstart-register-app.md) para integrar con la Plataforma de identidad de Microsoft.
