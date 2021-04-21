---
title: Uso de Azure AD Identity Governance para revisar y eliminar usuarios externos que ya no tienen acceso a los recursos
description: Uso de revisiones de acceso para extender o eliminar el acceso de miembros de organizaciones asociadas
services: active-directory
documentationcenter: ''
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: compliance
ms.date: 09/06/2020
ms.author: ajburnle
ms.openlocfilehash: c976562224d4a0caca8921e46d8f8566800027ee
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107532235"
---
# <a name="use-azure-active-directory-azure-ad-identity-governance-to-review-and-remove-external-users-who-no-longer-have-resource-access"></a>Uso de Azure Active Directory (Azure AD) Identity Governance para revisar y eliminar usuarios externos que ya no tienen acceso a los recursos

En este artículo se describen las características y los métodos que le permiten localizar y seleccionar identidades externas para que pueda revisarlas y eliminarlas de Azure AD si ya no son necesarias. La nube hace que sea más fácil que nunca colaborar con usuarios internos o externos. Al adoptar Office 365, las organizaciones empiezan a ver la proliferación de identidades externas (incluidos los invitados), ya que los usuarios trabajan juntos en datos, documentos o áreas de trabajo digitales como equipos. Las organizaciones deben equilibrar, habilitar la colaboración y cumplir los requisitos de seguridad y gobernanza. Una parte de estos esfuerzos debe incluir la evaluación y limpieza de los usuarios externos, a los que se ha invitado para colaborar en el inquilino y que provienen de organizaciones asociadas, y su eliminación de Azure AD cuando ya no son necesarios.

>[!NOTE]
>Se requiere una licencia válida de Azure AD Premium P2, Enterprise Mobility + Security E5 de pago, o una licencia de prueba para usar las revisiones de acceso de Azure AD. Para obtener más información, consulte [Ediciones de Azure Active Directory](../fundamentals/active-directory-whatis.md).

## <a name="why-review-users-from-external-organizations-in-your-tenant"></a>¿Por qué revisar los usuarios de organizaciones externas del inquilino?

En la mayoría de las organizaciones, los usuarios finales inician el proceso de invitar a socios comerciales y proveedores para colaborar. La necesidad de colaborar lleva a las organizaciones a proporcionar a los propietarios de los recursos y a los usuarios finales una manera de evaluar y atestar los usuarios externos con regularidad. A menudo, el proceso de incorporación de nuevos asociados de colaboración está planeado y se ha tenido en cuenta, pero hay muchas colaboraciones que no tienen una fecha de finalización clara y no siempre es obvio cuándo un usuario ya no necesita acceso. Además, la administración del ciclo de vida de las identidades impulsa a las empresas a mantener Azure AD limpio y eliminar a los usuarios que ya no necesitan tener acceso a los recursos de la organización. Mantener solo las referencias de identidad pertinentes para asociados y proveedores en el directorio ayuda a reducir el riesgo de los empleados de seleccionar y conceder acceso inadvertidamente a usuarios externos que se deberían haber eliminado. Este documento le guía con varias opciones que van desde sugerencias proactivas recomendadas hasta actividades reactivas y de limpieza para la gobernanza de las identidades externas.

## <a name="use-entitlement-management-to-grant-and-revoke-access"></a>Uso de la administración de derechos para conceder y revocar el acceso

Las características de administración de derechos permiten el [ciclo de vida automatizado de las identidades externas](entitlement-management-external-users.md#manage-the-lifecycle-of-external-users) con acceso a los recursos. Al establecer procesos y procedimientos para administrar el acceso mediante la administración de derechos, y publicar los recursos mediante paquetes de acceso, realizar un seguimiento del acceso de los usuarios externos a los recursos se convierte en un problema mucho menos complicado de resolver. Al administrar el acceso mediante [paquetes de acceso de administración de derechos](entitlement-management-overview.md) en Azure AD, la organización puede definir y administrar de forma centralizada el acceso de los usuarios, así como el de los usuarios de organizaciones asociadas. La administración de derechos utiliza aprobaciones y asignaciones de paquetes de acceso para realizar un seguimiento de dónde han solicitado acceso los usuarios externos y dónde se les ha asignado acceso. Si un usuario externo pierde todas sus asignaciones, la administración de derechos puede eliminar estos usuarios externos automáticamente del inquilino. 

## <a name="find-guests-not-invited-through-entitlement-management"></a>Búsqueda de invitados que no han sido invitados mediante la administración de derechos

Cuando los empleados están autorizados a colaborar con usuarios externos, pueden invitar a cualquier número de usuarios de fuera de la organización. La búsqueda y agrupación de los asociados externos en grupos dinámicos alineados con la empresa y su revisión puede no ser factible, ya que puede haber demasiadas empresas individuales que revisar, o bien no hay ningún propietario o patrocinador para la organización. Microsoft proporciona un script de PowerShell de ejemplo que puede ayudarle a analizar el uso de identidades externas en un inquilino. El script enumera las identidades externas y las categoriza. El script puede ayudarle a identificar y limpiar las identidades externas que ya no sean necesarias. Como parte de su salida, el script de ejemplo admite la creación automatizada de grupos de seguridad que contienen a los asociados externos sin un grupo identificado, para un análisis más exhaustivo y su uso con las revisiones de acceso de Azure AD.
El script está disponible en [GitHub](https://github.com/microsoft/access-reviews-samples/tree/master/ExternalIdentityUse). Una vez finalizada la ejecución del script, se genera un archivo HTML de salida que resalta las identidades externas que:

- Ya no tienen ninguna pertenencia a grupos en el inquilino
- Tienen una asignación para un rol con privilegios en el inquilino
- Tienen una asignación para una aplicación en el inquilino

La salida también incluye los dominios individuales de cada una de estas identidades externas. 

>[!NOTE]
>El script al que se hace referencia anteriormente es un script de ejemplo que comprueba la pertenencia a grupos, las asignaciones de roles y las asignaciones de aplicaciones en Azure AD. Puede haber otras asignaciones de aplicaciones que los usuarios externos hayan recibido fuera de Azure AD, como SharePoint (asignación de pertenencia directa), RBAC de Azure o Azure DevOps.

## <a name="review-resources-used-by-external-identities"></a>Revisión de los recursos utilizados por identidades externas

Si tiene identidades externas que usan recursos, como Teams u otras aplicaciones, que aún no se rigen por la administración de derechos, puede que también desee revisar el acceso a estos recursos con regularidad. Las [revisiones de acceso](create-access-review.md) de Azure AD le ofrecen la posibilidad de revisar el acceso de las identidades externas, al permitir que el propietario del recurso, las propias identidades externas u otra persona delegada de confianza, atesten si es necesario continuar con el acceso. Las revisiones de acceso se dirigen a un recurso y crean una actividad de revisión cuyo ámbito puede ser todo el que tiene acceso al recurso o bien solo los usuarios invitados. A continuación, el revisor verá la lista resultante de los usuarios que necesitan revisión: todos los usuarios, incluidos los empleados de la organización o solo las identidades externas.

![Uso de un grupo para revisar el acceso](media/access-reviews-external-users/group-members.png)

El establecimiento de una cultura de revisión controlada por el propietario del recurso ayuda a controlar el acceso de las identidades externas. Los propietarios de recursos, quienes son responsables del acceso, la disponibilidad y la seguridad de la información que poseen, son, en la mayoría de los casos, la mejor audiencia para impulsar las decisiones sobre el acceso a sus recursos y están más cerca de los usuarios que tienen acceso a ellos que el departamento de TI central o un patrocinador que administra muchos usuarios externos.

## <a name="create-access-reviews-for-external-identities"></a>Creación de revisiones de acceso para identidades externas

Los usuarios que ya no tienen acceso a los recursos del inquilino se pueden eliminar si ya no colaboran con la organización. Antes de bloquear y eliminar estas identidades externas, es posible que desee ponerse en contacto con estos usuarios externos y asegurarse de que no se ha pasado por alto un proyecto o acceso permanente que todavía puedan necesitar. Al crear un grupo que contiene todas las identidades externas que se ha detectado que no tienen acceso a ningún recurso del inquilino, puede usar las revisiones de acceso para que todos los usuarios externos se otorguen su propia atestación sobre si todavía necesitan o tienen acceso, o si seguirán necesitando tener acceso en el futuro. Como parte de la revisión, el creador de la revisión en las revisiones de acceso puede usar la función **Requerir motivo de la aprobación** para requerir a los usuarios externos que proporcionen una justificación para el acceso continuado, mediante la cual puede obtener información sobre dónde y cómo todavía necesitan acceso en el inquilino. Además, puede habilitar la característica de configuración **Correo electrónico de contenido adicional para el revisor** para que los usuarios sepan que van a perder el acceso si no responden y, en caso de que sigan necesitando acceso, se requiere una justificación. Si desea continuar y permitir que las revisiones de acceso **deshabiliten y eliminen** las identidades externas en caso de que no respondan o proporcionen un motivo válido para el acceso continuado, puede usar la opción Deshabilitar y eliminar, como se describe en la sección siguiente.

![Limitar el ámbito de la revisión a solo los usuarios invitados](media/access-reviews-external-users/guest-users-only.png)

Cuando finaliza la revisión, la página **Resultados** muestra una visión general de la respuesta proporcionada por cada identidad externa. Puede optar por aplicar los resultados automáticamente y permitir que las revisiones de acceso los deshabiliten y eliminen. También puede examinar las respuestas dadas y decidir si desea eliminar el acceso de un usuario o hacer un seguimiento de ellos y obtener información adicional antes de tomar una decisión. Si algunos usuarios siguen con acceso a los recursos que aún no ha revisado, puede usar la revisión como parte de la detección y enriquecer el siguiente ciclo de revisión y atestación.

## <a name="disable-and-delete-external-identities-with-azure-ad-access-reviews"></a>Deshabilitación y eliminación de identidades externas con las revisiones de acceso de Azure AD

Además de la opción de eliminar las identidades externas no deseadas de recursos como grupos o aplicaciones, las revisiones de acceso de Azure AD pueden impedir que las identidades externas inicien sesión en el inquilino y eliminar las identidades externas del inquilino después de 30 días. Una vez que seleccione **Bloquear el inicio de sesión del usuario durante 30 días y después quitar el usuario del inquilino**, la revisión permanecerá en el estado "Aplicando" durante 30 días. Durante este período, la configuración, los resultados, los revisores o los registros de auditoría de la revisión actual no se podrán ver ni configurar. 

![Configuración de finalización](media/access-reviews-external-users/upon-completion-settings.png)

Al crear una nueva revisión de acceso, en la sección "Configuración de finalización", para la opción **Acción que se aplicará a los usuarios denegados** puede definir **Impedir que los usuarios inicien sesión durante 30 días y, a continuación, eliminar el usuario del inquilino**.

Esta configuración le permite identificar, bloquear y eliminar identidades externas del inquilino de Azure AD. Las identidades externas que se han revisado y a las que el revisor ha denegado continuar con el acceso se bloquearán y eliminarán, independientemente del acceso a los recursos o de la pertenencia a grupos que tengan. Esta configuración es más útil como último paso después de haber validado que los usuarios externos en revisión ya no conllevan el acceso a los recursos y se pueden eliminar de forma segura del inquilino, o si desea asegurarse de que se eliminan, con independencia de su acceso actual. La característica "Deshabilitar y eliminar" bloquea primero al usuario externo, retirando su capacidad de iniciar sesión en el inquilino y acceder a los recursos. En esta fase no se revoca el acceso a los recursos y, en caso de que quisiera volver a crear instancias del usuario externo, se puede volver a configurar la capacidad de iniciar sesión. Si no se realiza ninguna otra acción, una identidad externa bloqueada se eliminará del directorio después de 30 días y se eliminará la cuenta, así como su acceso.

## <a name="next-steps"></a>Pasos siguientes

- [Revisiones de acceso: Graph API](/graph/api/resources/accessreviewsv2-root?view=graph-rest-beta&preserve-view=true)
- [Administración de derechos: Graph API](/graph/api/resources/entitlementmanagement-root)
