---
title: Procedimiento para crear un inquilino de desarrollador de Azure Active Directory gratuito
description: En este artículo aprenderá a crear una cuenta de desarrollador.
services: active-directory
author: barclayn
manager: davba
ms.service: identity
ms.subservice: verifiable-credentials
ms.topic: how-to
ms.date: 04/01/2021
ms.author: barclayn
ms.openlocfilehash: 1e8bb59c09622a39dad680940ff34e643ee0cc3b
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106222748"
---
# <a name="how-to-create-a-free-azure-active-directory-developer-tenant"></a>Procedimiento para crear un inquilino de desarrollador de Azure Active Directory gratuito

> [!IMPORTANT]
> Las credenciales verificables de Azure Active Directory se encuentran actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

> [!NOTE]
> En la versión preliminar, se requiere una licencia P2. 

Hay dos formas sencillas de crear una instancia de Azure Active Directory gratuita con una licencia de prueba P2 para poder instalar el servicio de emisor de credenciales verificables y probar la creación y validación de las credenciales verificables:

- [Únase](https://aka.ms/o365devprogram) al programa para desarrolladores de Microsoft 365 gratuito y obtenga un espacio aislado gratuito, herramientas y otros recursos, como una instancia de Azure Active Directory con licencias P2, usuarios, grupos, buzones de correo, etc. configurados.
- Cree un nuevo [inquilino](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) y active una [evaluación gratuita](https://azure.microsoft.com/trial/get-started-active-directory/) de Azure AD Premium P1 o P2 en el nuevo inquilino.

Si decide registrarse en el programa para desarrolladores de Microsoft 365 gratuito, debe seguir unos pocos pasos sencillos:

1. Haga clic en el botón **Unirse ahora** en la pantalla.

2. Inicie sesión con una nueva cuenta Microsoft o use una cuenta (profesional) existente que ya tenga.

3. En la página de registro, seleccione su región, escriba un nombre de empresa y acepte los términos y condiciones del programa antes de hacer clic en **Siguiente**.

4. Haga clic en **Set up subscription** (Configurar suscripción). Especifique la región en la que quiere crear el nuevo inquilino, cree un nombre de usuario y un dominio, y escriba una contraseña. Así creará un nuevo inquilino y el primer administrador del inquilino.

5. Escriba la información de seguridad necesaria para proteger la cuenta de administrador de su nuevo inquilino. Esto configurará la autenticación MFA para la cuenta.


En este punto, ha creado un inquilino con 25 licencias de usuario E5. Las licencias E5 incluyen licencias de Azure AD P2. Opcionalmente, puede agregar paquetes de datos de ejemplo con usuarios, grupos, correo y SharePoint para ayudarle a probar en su entorno de desarrollo. En el caso del servicio de emisión de credenciales verificables, no son necesarios.

Para su comodidad, puede agregar su propia cuenta profesional como [invitado](https://docs.microsoft.com/azure/active-directory/b2b/b2b-quickstart-add-guest-users-portal.md) en el inquilino recién creado y usar esa cuenta para administrar el inquilino. Si quiere que la cuenta de invitado pueda administrar el servicio de credenciales verificables, debe asignar el rol "Administrador global" a ese usuario.

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene una cuenta de desarrollador, puede probar nuestro [primer tutorial](get-started-verifiable-credentials.md) para obtener más información sobre las credenciales verificables.