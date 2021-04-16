---
title: Introducción al modo de dispositivo compartido
titleSuffix: Microsoft identity platform | Azure
description: Obtenga información sobre el modo de dispositivo compartido para habilitar el uso compartido de dispositivos para los trabajadores de primera línea.
services: active-directory
author: brandwe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 03/31/2020
ms.author: brandwe
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: cf8869002fb3e0170331709af3da5b971a098740
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105612405"
---
# <a name="overview-of-shared-device-mode"></a>Introducción al modo de dispositivo compartido

El modo de dispositivo compartido es una característica de Azure Active Directory que permite crear aplicaciones que admiten trabajadores de primera línea y habilitar el modo de dispositivo compartido en los dispositivos implementados.

>[!IMPORTANT]
> Modo de dispositivo compartido para [!INCLUDE [PREVIEW BOILERPLATE](../../../includes/active-directory-develop-preview.md)] iOS

## <a name="what-are-frontline-workers"></a>¿Qué son los trabajadores de primera línea?

Los trabajadores de primera línea son los empleados de comercios, los agentes de mantenimiento y de campo, el personal médico y otros usuarios que no se encuentran delante de un equipo ni usan el correo electrónico corporativo para la colaboración. En las secciones siguientes se presentan los aspectos y los desafíos de habilitar los trabajadores de primera línea, seguido de una introducción a las características proporcionadas por Microsoft que habilitan una aplicación para su uso por parte de los trabajadores de primera línea de una organización.

### <a name="challenges-of-supporting-frontline-workers"></a>Desafíos de la compatibilidad para trabajadores de primera línea

La habilitación de flujos de trabajo de trabajadores de primera línea incluye desafíos que normalmente no se presentan con los trabajadores de la información típicos. Estos desafíos pueden incluir una alta tasa de rotación y menor familiaridad con las principales herramientas de productividad de la organización. Para dotar de recursos a sus trabajadores de primera línea, las organizaciones adoptan diferentes estrategias. Algunas adoptan la estrategia Traiga su propio dispositivo (BYOD), en la que los empleados usan aplicaciones empresariales en su teléfono personal, mientras que otras proporcionan a sus empleados dispositivos compartidos como iPads o tabletas Android.

### <a name="supporting-multiple-users-on-devices-designed-for-one-user"></a>Compatibilidad con varios usuarios en dispositivos diseñados para un usuario

Dado que los dispositivos móviles que ejecutan iOS o Android se diseñaron para usuarios individuales, la mayoría de las aplicaciones optimizan su experiencia para su uso por parte de un solo usuario. Parte de esta experiencia optimizada significa habilitar el inicio de sesión único entre aplicaciones y mantener a los usuarios con sesión iniciada en su dispositivo. Cuando un usuario elimina su cuenta de una aplicación, normalmente la aplicación no lo considera un evento relacionado con la seguridad. Muchas aplicaciones incluso conservan las credenciales de un usuario para el inicio de sesión rápido. Es posible que haya experimentado esto cuando ha eliminado una aplicación de su dispositivo móvil y, a continuación, la ha reinstalado, solo para descubrir que todavía está conectado.

### <a name="global-sign-in-and-sign-out-sso"></a>Inicio y cierre de sesión global (SSO)

Para permitir que los empleados de una organización usen sus aplicaciones en un grupo de dispositivos compartidos por dichos empleados, los desarrolladores deben habilitar la experiencia opuesta. Los empleados deben poder tomar un dispositivo del grupo y realizar un único gesto para "hacerlo suyo" mientras dure el turno. Al final de su turno, deberían poder realizar otro gesto para cerrar la sesión globalmente en el dispositivo, con la información personal y de la empresa eliminadas para que puedan devolverlo al grupo de dispositivos. Además, si un empleado olvida cerrar la sesión, el dispositivo debe cerrar sesión automáticamente al final de su turno y/o después de un período de inactividad.

Azure Active Directory habilita estos escenarios con una característica llamada **modo de dispositivo compartido**.

## <a name="introducing-shared-device-mode"></a>Presentación del modo de dispositivo compartido

Como se mencionó, el modo de dispositivo compartido es una característica de Azure Active Directory que le permite:

* Creación de aplicaciones que admiten trabajadores de primera línea
* Implementación de dispositivos para trabajadores de primera línea y activación del modo de dispositivo compartido

### <a name="build-applications-that-support-frontline-workers"></a>Creación de aplicaciones que admiten trabajadores de primera línea

Puede admitir trabajadores de primera línea en sus aplicaciones mediante el uso de la biblioteca de autenticación de Microsoft (MSAL) y la [aplicación Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) para habilitar un estado del dispositivo llamado *modo de dispositivo compartido*. Cuando un dispositivo está en modo de dispositivo compartido, Microsoft proporciona información a la aplicación para que pueda modificar el comportamiento en función del estado del usuario en el dispositivo, protegiendo los datos de usuario.

Las características admitidas son:

* **Inicio de sesión de un usuario en todo el dispositivo** en cualquier aplicación compatible.
* **Cierre de sesión de un usuario en todo el dispositivo** en cualquier aplicación compatible.
* **Consulta del estado del dispositivo** para determinar si la aplicación está en un dispositivo que a su vez está en modo de dispositivo compartido.
* **Consulta del estado del dispositivo del usuario** en el dispositivo para determinar si ha cambiado algo desde la última vez que se usó la aplicación.

La compatibilidad con el modo de dispositivo compartido debe considerarse como una mejora de seguridad y una actualización de características de la aplicación y puede ayudar a aumentar su adopción en entornos altamente regulados como la atención sanitaria y las finanzas.

Los usuarios dependen de usted para asegurarse de que sus datos no se filtran a otro usuario. El modo de dispositivo compartido proporciona señales útiles para indicar a la aplicación que se ha producido un cambio que debe administrar. La aplicación es responsable de comprobar el estado del usuario en el dispositivo cada vez que se usa la aplicación, eliminando los datos del usuario anterior. Esto incluye cuando se vuelve a cargar desde segundo plano en modo multitarea. En un cambio de usuario, debe asegurarse de que se eliminan los datos del usuario anterior y se eliminan los datos almacenados en caché que se muestran en la aplicación. Se recomienda realizar siempre un proceso de revisión de seguridad exhaustivo después de agregar la funcionalidad de modo de dispositivo compartido a la aplicación.

Para más información sobre cómo modificar las aplicaciones para admitir el modo de dispositivo compartido, consulte la sección [Pasos siguientes](#next-steps) al final de este artículo.

### <a name="deploy-devices-to-frontline-workers-and-turn-on-shared-device-mode"></a>Implementación de dispositivos para trabajadores de primera línea y activación del modo de dispositivo compartido

Una vez que las aplicaciones admitan el modo de dispositivo compartido e incluyan los datos necesarios y los cambios de seguridad, puede anunciarlas como aptas para su uso por trabajadores de primera línea.

Los administradores de dispositivos de una organización pueden implementar sus dispositivos y sus aplicaciones en sus tiendas y lugares de trabajo mediante una solución de administración de dispositivos móviles (MDM) como Microsoft Intune. Parte del proceso de aprovisionamiento consiste en marcar el dispositivo como *dispositivo compartido*. Los administradores configuran el modo de dispositivo compartido mediante la implementación de la aplicación [Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) y estableciendo el modo de dispositivo compartido en los parámetros de configuración. Después de realizar estos pasos, todas las aplicaciones que admiten el modo de dispositivo compartido usarán la aplicación Microsoft Authenticator para administrar su estado de usuario y proporcionar características de seguridad para el dispositivo y la organización.

## <a name="next-steps"></a>Pasos siguientes

Se admiten las plataformas iOS y Android para el modo de dispositivo compartido. Revise la documentación siguiente correspondiente a su plataforma para empezar a admitir trabajadores de primera línea en sus aplicaciones.

* [Compatibilidad con el modo de dispositivo compartido para iOS](msal-ios-shared-devices.md)
* [Compatibilidad con el modo de dispositivo compartido para Android](msal-android-shared-devices.md)
