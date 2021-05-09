---
title: 'Preguntas frecuentes sobre la asociación de aplicaciones en formato MSIX de Windows Virtual Desktop: Azure'
description: Preguntas más frecuentes sobre la asociación de aplicaciones en formato MSIX para Windows Virtual Desktop.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: 1277e053e9a9eb6e54eb11a9ede42ca28d99d073
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108141314"
---
# <a name="msix-app-attach-faq"></a>Preguntas frecuentes sobre la asociación de aplicaciones en formato MSIX

En este artículo se responden las preguntas más frecuentes sobre la asociación de aplicaciones en formato MSIX para Windows Virtual Desktop.

## <a name="whats-the-difference-between-msix-and-msix-app-attach"></a>¿Cuál es la diferencia entre MSIX y la conexión de aplicaciones en formato MSIX ?

MSIX es un formato de empaquetado de aplicaciones, mientras que la conexión de aplicaciones en formato MSIX es la característica que proporciona paquetes MSIX a la implementación.

## <a name="does-msix-app-attach-use-fslogix"></a>¿La asociación de aplicaciones en formato MSIX usa FSLogix?

La asociación de aplicaciones en formato MSIX no utiliza FSLogix. Sin embargo, la asociación de aplicaciones en formato MSIX y FSLogix están diseñados para trabajar conjuntamente con el fin de proporcionar una experiencia de usuario sin problemas.

## <a name="can-i-use-the-msix-app-attach-outside-of-windows-virtual-desktop"></a>¿Puedo usar la asociación de aplicaciones con formato MSIX fuera de Windows Virtual Desktop?

Las API que permiten el funcionamiento de la conexión de aplicaciones en formato MSIX están disponibles para Windows 10 Enterprise. Estas se pueden usar fuera de Windows Virtual Desktop. Sin embargo, no hay ningún plano de administración para la asociación de aplicaciones en formato MSIX fuera de Windows Virtual Desktop.

## <a name="how-do-i-get-an-msix-package"></a>¿Cómo se obtiene un paquete MSIX?

Su proveedor de software le proporcionará un paquete MSIX. También puede convertir paquetes que no sean MSIX en MSIX. Obtenga más información en [Cómo trasladar los instaladores existentes a MSIX](/windows/msix/packaging-tool/create-an-msix-overview#how-to-move-your-existing-installers-to-msix).

## <a name="which-operating-systems-support-msix-app-attach"></a>¿Qué sistemas operativos admiten la asociación de aplicaciones en formato MSIX?

Windows 10 Enterprise y sesión múltiple de Windows 10 Enterprise, versión 2004 o posterior

## <a name="is-msix-app-attach-currently-generally-available"></a>¿La conexión de aplicaciones en formato MSIX está disponible con carácter general actualmente?

La conexión de aplicaciones en formato MSIX forma parte de Windows 10 Enterprise y la sesión múltiple de Windows 10 Enterprise, versión 2004 o una posterior. Ambos sistemas operativos están disponibles actualmente con carácter general. 

## <a name="can-i-use-msix-app-attach-outside-of-windows-virtual-desktop"></a>¿Puedo usar la asociación de aplicaciones con formato MSIX fuera de Windows Virtual Desktop?

Las API de la conexión de aplicaciones en formato MSIX y MSIX forman parte de Windows 10 Enterprise y la sesión múltiple de Windows 10 Enterprise, versión 2004 y las posteriores. Actualmente, no ofrecemos ningún software de administración para la conexión de aplicaciones en formato MSIX fuera de Windows Virtual Desktop.

## <a name="can-i-run-two-versions-of-the-same-application-at-the-same-time"></a>¿Puedo ejecutar dos versiones de la misma aplicación al mismo tiempo?

Para que dos versiones de las mismas aplicaciones MSIX se ejecuten simultáneamente, la familia de paquetes MSIX definida en el archivo appxmanifest.xml debe ser diferente para cada aplicación.

## <a name="should-i-disable-auto-update-when-using-msix-app-attach"></a>¿Debo deshabilitar la actualización automática cuando uso la conexión de aplicaciones en formato MSIX?

Sí. La conexión de aplicaciones en formato MSIX no admite la actualización automática para aplicaciones de MSIX.

## <a name="how-do-permissions-work-with-msix-app-attach"></a>¿Cómo funcionan los permisos con la conexión de aplicaciones en formato MSIX?

Todas las máquinas virtuales (VM) de un grupo de hosts que usa la conexión de aplicaciones en formato MSIX deben tener permisos de lectura en el recurso compartido de archivos en el que se almacenan las imágenes de MSIX. Si también usa Azure Files, deberán tener permisos de control de acceso basado en rol (RBAC) y de New Technology File System (NTFS).

## <a name="can-i-use-azure-active-directory-domain-services-azure-ad-ds-with-msix-app-attach"></a>¿Puedo usar Azure Active Directory Domain Services (Azure AD DS) con la asociación de aplicaciones en formato MSIX?

La asociación de aplicaciones en formato MSIX no admite actualmente Azure AD DS. Dado que los objetos de equipo de Azure AD DS no se sincronizan con Azure Active Directory (Azure AD), el administrador no puede proporcionar los permisos de control de acceso basado en roles (RBAC) necesarios para Azure Files.

## <a name="can-i-use-msix-app-attach-for-http-or-https"></a>¿Puedo usar la conexión de aplicaciones en formato MSIX para HTTP o HTTPS?

Actualmente, no se admite la conexión de aplicaciones en formato MSIX a través de HTTP o HTTPS.

## <a name="can-i-restage-the-same-msix-application"></a>¿Puedo realizar una copia intermedia de la misma aplicación MSIX?

Sí. Puede realizar una copia intermedia de las aplicaciones para las que ya la ha hecho y esto no debería producir ningún error.

## <a name="does-msix-app-attach-support-self-signed-certificates"></a>¿La conexión de aplicaciones en formato MSIX admite certificados autofirmados?

Sí. Debe instalar el certificado autofirmado en todas las máquinas virtuales de host de sesión donde se usa la asociación de aplicaciones en formato MSIX para hospedar la aplicación autofirmada.

## <a name="what-applications-can-i-repackage-to-msix"></a>¿Qué aplicaciones puedo volver a empaquetar en MSIX?

Cada aplicación usa características diferentes del sistema operativo, de los lenguajes de programación y de los marcos. Para volver a empaquetar la aplicación, siga las instrucciones de [Traslado de los instaladores existentes a MSIX](/windows/msix/packaging-tool/create-an-msix-overview#how-to-move-your-existing-installers-to-msix). Puede encontrar una lista de las cosas que necesita para volver a empaquetar una aplicación en [Preparación del empaquetado de una aplicación de escritorio](/windows/msix/desktop/desktop-to-uwp-prepare). 

Algunas aplicaciones no pueden dividirse en capas de aplicación, lo que significa que no se pueden volver a empaquetar en un archivo MSIX. Esta es una lista de las aplicaciones que no se pueden volver a empaquetar:

- Controladores 
- Active-X o Silverlight
- Clientes VPN
- Programas antivirus

## <a name="next-steps"></a>Pasos siguientes

Si quiere obtener más información sobre la conexión de aplicaciones en formato MSIX, consulte nuestra [introducción](what-is-app-attach.md) y [glosario](app-attach-glossary.md). De lo contrario, empiece a trabajar con [Configuración de la conexión de aplicaciones](app-attach.md).
