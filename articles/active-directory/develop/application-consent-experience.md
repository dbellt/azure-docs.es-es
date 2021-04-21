---
title: Experiencias de consentimiento de la aplicación en Azure AD
titleSuffix: Microsoft identity platform
description: Más información sobre las experiencias de consentimiento de Azure AD para ver cómo se pueden usar al administrar y desarrollar aplicaciones en Azure AD
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: conceptual
ms.date: 04/06/2021
ms.author: ryanwi
ms.reviewer: jesakowi, asteen
ms.openlocfilehash: c570fc9f30d69f13546353cf6edab4122ae35142
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105465"
---
# <a name="understanding-azure-ad-application-consent-experiences"></a>Descripción de las experiencias de consentimiento de la aplicación de Azure AD

Aprenda más acerca de la experiencia del usuario para el consentimiento de la aplicación de Azure Active Directory (Azure AD). Por lo que puede administrar las aplicaciones para su organización o desarrollar aplicaciones de manera inteligente con una experiencia de consentimiento más sencilla.

## <a name="consent-and-permissions"></a>Consentimiento y permisos

El consentimiento es el proceso por el cual un usuario concede autorización para que una aplicación acceda a recursos protegidos en su nombre. Se puede solicitar el consentimiento a un administrador o usuario para que permita el acceso a los datos de la organización o de personas.

La experiencia real del usuario para otorgar el consentimiento variará en función de las directivas establecidas en el inquilino del usuario, el ámbito de autoridad del usuario (o su rol) y el tipo de [permisos](v2-permissions-and-consent.md) que solicita la aplicación cliente. Esto significa que los desarrolladores de aplicaciones y los administradores de inquilinos tienen cierto control sobre la experiencia de consentimiento. Los administradores tienen la flexibilidad de configurar y deshabilitar las directivas en una aplicación o inquilino para controlar la experiencia de consentimiento en su inquilino. Los desarrolladores de aplicaciones pueden dictar qué tipos de permiso se solicitan y si quieren guiar a los usuarios a través del flujo de consentimiento del usuario o el flujo de consentimiento del administrador.

- El **flujo de consentimiento del usuario** es cuando un desarrollador de aplicaciones dirige a los usuarios al punto de conexión de autorización con la intención de registrar el consentimiento solo para el usuario actual.
- El **flujo de consentimiento del administrador** es cuando un desarrollador de aplicaciones dirige a los usuarios al punto de conexión de consentimiento del administrador con la intención de registrar el consentimiento para todo el inquilino. Para asegurarse de que el flujo de consentimiento del administrador funcione correctamente, los desarrolladores de aplicaciones deben enumerar todos los permisos en la propiedad `RequiredResourceAccess` en el manifiesto de aplicación. Para más información, consulte [Manifiesto de aplicación](./reference-app-manifest.md).

## <a name="building-blocks-of-the-consent-prompt"></a>Bloques de creación de la petición de consentimiento

La petición de consentimiento está diseñada para garantizar que los usuarios tengan suficiente información para determinar si confían en que la aplicación cliente acceda a recursos protegidos en su nombre. Comprender los bloques de creación ayudará a los usuarios que otorgan consentimientos a tomar decisiones más informadas y ayudará a los desarrolladores a crear mejores experiencias de usuario.

En el diagrama y la tabla siguientes se proporciona información sobre los bloques de creación de la petición de consentimiento.

:::image type="content" source="./media/application-consent-experience/consent_prompt.png" alt-text="Bloques de creación de la petición de consentimiento":::

| # | Componente | Propósito |
| ----- | ----- | ----- |
| 1 | Identificador de usuario | Este identificador representa al usuario en cuyo nombre la aplicación cliente solicita acceso a recursos protegidos. |
| 2 | Título | El título cambia en función de si los usuarios siguen el flujo de consentimiento del usuario o del administrador. En el flujo de consentimiento del usuario, el título será "Permisos solicitados", mientras que en el flujo de consentimiento del administrador, el título tendrá una línea adicional "Aceptar para su organización". |
| 3 | Logotipo de la aplicación | Esta imagen debe ayudar a los usuarios a tener una indicación visual de si esta aplicación es la aplicación a la que pretenden acceder. Los desarrolladores de aplicaciones proporcionan esta imagen, y no se valida la propiedad de esta imagen. |
| 4 | Nombre de la aplicación | Este valor debe informar a los usuarios qué aplicación solicita acceso a sus datos. Tenga en cuenta que los desarrolladores proporcionan este nombre, y no se valida la propiedad del nombre de esta aplicación. |
| 5 | Dominio de editor | Este valor debe proporcionar a los usuarios un dominio cuya confiabilidad puedan evaluar. Los desarrolladores proporcionan este dominio, y no se valida la propiedad de este dominio de editor. |
| 6 | Publicador comprobado | El distintivo azul de "comprobado" significa que el publicador de la aplicación ha comprobado su identidad mediante una cuenta de Microsoft Partner Network y ha completado el proceso de comprobación.|
| 7 | Información del anunciante  | Muestra si Microsoft o su organización han publicado la aplicación. |
| 8 | Permisos | Esta lista contiene los permisos solicitados por la aplicación cliente. Los usuarios deben evaluar siempre los tipos de permisos que se solicitan a fin de comprender a qué datos estará autorizada a acceder en su nombre la aplicación cliente si aceptan. Como desarrollador de aplicaciones es mejor solicitar acceso a los permisos con los privilegios mínimos. |
| 9 | Descripción del permiso | El servicio que expone los permisos proporciona este valor. Para ver las descripciones de los permisos, debe activar o desactivar el botón de contenido adicional junto al permiso. |
| 10| Términos de la aplicación | Estos términos contienen vínculos a los términos de servicio y la declaración de privacidad de la aplicación. El editor es responsable de detallar sus reglas en las condiciones de servicio. Además, el editor es responsable de divulgar la forma en que usará y compartirá los datos de usuario en su declaración de privacidad. Si el editor no proporciona vínculos a estos valores para las aplicaciones multiinquilino, habrá una advertencia en negrita en la petición de consentimiento. |
| 11 | https://myapps.microsoft.com | Este es el vínculo donde los usuarios pueden revisar y quitar las aplicaciones que no son de Microsoft que actualmente tienen acceso a sus datos. |
| 12 | Notifíquelo aquí | Este vínculo se usa para notificar de una aplicación sospechosa si no confía en ella, si cree que la aplicación está suplantando a otra, si cree que la aplicación hará un uso incorrecto de sus datos o por algún otro motivo. |

## <a name="app-requires-a-permission-within-the-users-scope-of-authority"></a>La aplicación requiere un permiso dentro del ámbito de autoridad del usuario

Un escenario de consentimiento común es cuando el usuario accede a una aplicación que requiere un conjunto de permisos dentro del ámbito de autoridad del usuario. Se dirige al usuario al flujo de consentimiento del usuario.

Los administradores verán un control adicional en la petición de consentimiento tradicional que les permitirá dar su consentimiento en nombre de todo el inquilino. El control estará desactivado de manera predeterminada, por lo que solo cuando los administradores marquen la casilla explícitamente se otorgará el consentimiento en nombre de todo el inquilino. En este momento, esta casilla solo se mostrará para el rol de administrador global, por lo que el administrador en la nube y el administrador de aplicaciones no verán esta casilla.

:::image type="content" source="./media/application-consent-experience/consent_prompt_1a.png" alt-text="Petición de consentimiento para el escenario 1a":::

Los usuarios verán la petición de consentimiento tradicional.

:::image type="content" source="./media/application-consent-experience/consent_prompt_1b.png" alt-text="Captura de pantalla que muestra la petición de consentimiento tradicional.":::

## <a name="app-requires-a-permission-outside-of-the-users-scope-of-authority"></a>La aplicación requiere un permiso fuera del ámbito de autoridad del usuario

Otro escenario de consentimiento común es cuando el usuario accede a una aplicación que requiere como mínimo un permiso fuera del ámbito de autoridad del usuario.

Los administradores verán un control adicional en la petición de consentimiento tradicional que les permitirá dar su consentimiento en nombre de todo el inquilino.

:::image type="content" source="./media/application-consent-experience/consent_prompt_1a.png" alt-text="Petición de consentimiento para el escenario 1a":::

Los usuarios que no son administradores no podrán otorgar consentimiento a la aplicación y se les indicará que pidan al administrador acceso a la aplicación.

:::image type="content" source="./media/application-consent-experience/consent_prompt_2b.png" alt-text="Captura de pantalla de la petición de consentimiento que indica al usuario que solicite a un administrador el acceso a la aplicación.":::

## <a name="user-is-directed-to-the-admin-consent-flow"></a>Se dirige al usuario al flujo de consentimiento del administrador

Otro escenario común es cuando el usuario navega o se dirige al flujo de consentimiento del administrador.

Los usuarios administradores verán la petición de consentimiento del administrador. El título y las descripciones de permisos cambiaron en esta petición, los cambios destacan el hecho de que, al aceptar esta petición, se otorgará a la aplicación acceso a los datos solicitados en el nombre de todo el inquilino.

:::image type="content" source="./media/application-consent-experience/consent_prompt_3a.png" alt-text="Petición de consentimiento para el escenario 3a":::

Los usuarios que no son administradores no podrán otorgar consentimiento a la aplicación y se les indicará que pidan al administrador acceso a la aplicación.

:::image type="content" source="./media/application-consent-experience/consent_prompt_2b.png" alt-text="Captura de pantalla de la petición de consentimiento que indica al usuario que solicite a un administrador el acceso a la aplicación.":::

## <a name="next-steps"></a>Pasos siguientes

- Introducción paso a paso de [cómo el marco de consentimiento de Azure AD implementa el consentimiento](./quickstart-register-app.md).
- Para más detalles, aprenda [cómo una aplicación multiinquilino puede utilizar el marco de consentimiento](./howto-convert-app-to-be-multi-tenant.md) para implementar el consentimiento de "usuario" y "administrador", compatible con patrones de aplicación de niveles múltiples más avanzados.
- Obtenga información sobre [cómo configurar el dominio de editor de la aplicación](howto-configure-publisher-domain.md).