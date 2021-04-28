---
title: Control de acceso de HSM administrado de Azure
description: Administre permisos de acceso para HSM administrado de Azure y claves. Trata sobre el modelo de autenticación y autorización de HSM administrado y cómo proteger sus HSM.
services: key-vault
author: amitbapat
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: ambapat
ms.openlocfilehash: bea1ccf0777c6325bc86c15e0f88304c465d89c9
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107750291"
---
# <a name="managed-hsm-access-control"></a>Control de acceso de HSM administrado

> [!NOTE]
> El proveedor de recursos de Key Vault admite dos tipos de recursos: **almacenes** y **HSM administrados**. El control de acceso descrito en este artículo solo se aplica a los **HSM administrados**. Para obtener más información sobre el control de acceso para HSM administrado, consulte [Acceso a las claves, los certificados y los secretos de Key Vault con un control de acceso basado en rol de Azure](../general/rbac-guide.md).

HSM administrado de Azure Key Vault es un servicio en la nube que protege las claves de cifrado. Dado que estos datos son confidenciales y críticos para la empresa, debe proteger el acceso a los HSM administrados de modo que solo se admita el acceso de las aplicaciones y los usuarios autorizados. En este artículo se proporciona información general sobre el modelo de control de acceso del HSM administrado. Se explican la autenticación y la autorización, y se describe cómo proteger el acceso a los HSM administrados.

## <a name="access-control-model"></a>Modelo de control de acceso

El acceso a un HSM administrado se controla mediante dos interfaces: el **plano de administración** y el **plano de datos**. El plano de administración es donde puede administrar el propio HSM. Entre las operaciones de este plano se incluyen la creación y eliminación de HSM administrados, y la recuperación de propiedades de HSM administrado. El plano de datos es donde trabaja con los datos almacenados en un HSM administrado (es decir, claves de cifrado protegidas con HSM). Puede agregar, eliminar, modificar y usar claves para realizar operaciones criptográficas, administrar asignaciones de roles para controlar el acceso a las claves, crear una copia de seguridad de HSM completa, restaurar la copia de seguridad completa y administrar el dominio de seguridad desde la interfaz de plano de datos.

Para obtener acceso a un HSM administrado en cualquier plano, todos los llamadores deben tener una autorización y autenticación correctas. La autenticación establece la identidad del llamador. La autorización determina las operaciones que puede ejecutar el llamador. Un llamador puede ser cualquiera de las [entidades de seguridad](../../role-based-access-control/overview.md#security-principal) definidas en Azure Active Directory: usuario, grupo, entidad de servicio o identidad administrada.

Ambos planos usan Azure Active Directory para la autenticación. Para la autorización, usan sistemas diferentes como se indica a continuación.
- El plano de administración usa el control de acceso basado en rol de Azure (RBAC de Azure), un sistema de autorización generado en Azure Resource Manager 
- El plano de datos usa un RBAC de nivel de HSM administrado (RBAC local de HSM administrado), un sistema de autorización implementado y aplicado en el nivel de HSM administrado.

Cuando se crea un HSM administrado, el solicitante también proporciona una lista de administradores de planos de datos (se admiten todas las [entidades de seguridad](../../role-based-access-control/overview.md#security-principal)). Solo estos administradores pueden tener acceso al plano de datos de HSM administrado para realizar operaciones clave y administrar asignaciones de roles del plano de datos (RBAC local de HSM administrado).

El modelo de permisos para ambos planos usa la misma sintaxis, pero se aplican en diferentes niveles y las asignaciones de roles usan distintos ámbitos. Azure Resource Manager aplica Azure RBAC del plano de administración, mientras que el propio HSM administrado aplica el RBAC local del HSM administrado de plano de datos.

> [!IMPORTANT]
> La concesión de acceso del plano de administración de entidades de seguridad a un HSM administrado no les concede acceso al plano de datos para tener acceso a las claves o las asignaciones de roles del plano de datos (RBAC local de HSM administrado). Este aislamiento se hace por diseño para prevenir la expansión involuntaria de privilegios que afectan al acceso a las claves almacenadas en HSM administrado.

Por ejemplo, un administrador de suscripciones (ya que tienen permiso de "Colaborador" en todos los recursos de la suscripción) puede eliminar un HSM administrado en su suscripción, pero si no tienen acceso del plano de datos concedido específicamente a través del RBAC local de HSM administrado, no pueden obtener acceso a las claves ni administrar la asignación de roles en el HSM administrado para concederse a sí mismos o a otras personas acceso al plano de datos.

## <a name="azure-active-directory-authentication"></a>Autenticación con Azure Active Directory

Cuando se crea un HSM administrado en una suscripción de Azure, se asocia automáticamente al inquilino de Azure Active Directory de la suscripción. En ambos planos, todos los llamadores deben registrarse en este inquilino y autenticarse para acceder al HSM administrado.

La aplicación se autentica con Azure Active Directory antes de llamar a cualquier plano. La aplicación puede utilizar cualquier [método de autenticación compatible](../../active-directory/develop/authentication-vs-authorization.md) según el tipo de aplicación. La aplicación adquiere un token para un recurso del plano para conceder acceso. El recurso es un punto de conexión en el plano de administración o de datos, según el entorno de Azure. La aplicación usa el token y envía la solicitud de una API de REST al punto de conexión de HSM administrado. Para más información, revise [todo el flujo de autenticación](../../active-directory/develop/v2-oauth2-auth-code-flow.md).

El uso de un único mecanismo de autenticación para ambos planos tiene varias ventajas:

- Las organizaciones pueden controlar el acceso de forma centralizada a todos los HSM administrados de su organización.
- Si un usuario abandona la organización, al instante pierde el acceso a todos los HSM administrados de la organización.
- Las organizaciones pueden personalizar la autenticación mediante las opciones de Azure Active Directory, como la habilitación de Multi-Factor Authentication para aumentar la seguridad.

## <a name="resource-endpoints"></a>Puntos de conexión de recursos

Las entidades de seguridad acceden a los planos a través de puntos de conexión. Los controles de acceso para los dos planos funcionan de forma independiente. Para conceder acceso a una aplicación para usar las claves de un HSM administrado, debe conceder acceso al plano de datos mediante el RBAC local de HSM administrado. Para conceder acceso a un usuario a un recurso de HSM administrado para crear, leer, eliminar, trasladar los HSM administrados y editar otras propiedades y etiquetas, use RBAC de Azure.

En la siguiente tabla se muestran los puntos de conexión para los planos de administración y datos.

| Plano de&nbsp;acceso | Puntos de conexión de acceso | Operaciones | Mecanismo de control de acceso |
| --- | --- | --- | --- |
| Plano de administración | **Global:**<br> management.azure.com:443<br> | Creación, lectura, actualización, eliminación y traslado de HSM administrados<br>Establecimiento de etiquetas de HSM administrado | Azure RBAC |
| Plano de datos | **Global:**<br> &lt;hsm-name&gt;.managedhsm.azure.net:443<br> | **Claves**: decrypt, encrypt,<br> unwrap, wrap, verify, sign, get, list, update, create, import, delete, backup, restore, purge<br/><br/> **Administración de roles del plano de datos (RBAC local de HSM administrado)** _: enumerar definiciones de roles, asignar roles, eliminar asignaciones de roles, definir roles personalizados<br/><br/>_ *Copia de seguridad y restauración **: creación y restauración de copias de seguridad, comprobación de las operaciones de creación y restauración de copias de seguridad de estado <br/><br/>** Dominio de seguridad**: descargar y cargar el dominio de seguridad | RBAC local de HSM administrado |
|||||

## <a name="management-plane-and-azure-rbac"></a>Plano de administración y RBAC de Azure

En el plano de administración, utilice RBAC de Azure para autorizar las operaciones que un llamador puede ejecutar. En el modelo de Azure RBAC, cada suscripción a Azure tiene una instancia de Azure Active Directory. Puede conceder acceso a usuarios, grupos y aplicaciones desde este directorio. El acceso se concede para administrar recursos de la suscripción de Azure que usan el modelo de implementación de Azure Resource Manager. Para conceder acceso, use [Azure Portal](https://portal.azure.com/), la [CLI de Azure](/cli/azure/install-classic-cli), [Azure PowerShell](/powershell/azureps-cmdlets-docs) o las [API de REST de Azure Resource Manager](/rest/api/authorization/roleassignments).

Puede crear un almacén de claves en un grupo de recursos y administrar el acceso mediante Azure Active Directory. Puede conceder a usuarios o grupos la capacidad de administrar los almacenes de claves en un grupo de recursos. Puede conceder acceso a un nivel de ámbito específico mediante la asignación de roles de Azure apropiados. Para conceder acceso a un usuario para administrar almacén de claves, debe asignar un rol `key vault Contributor` predefinido al usuario en un ámbito específico. Los siguientes niveles de ámbitos se pueden asignar a un rol de Azure:

- **Grupo de administración**:  Un rol de Azure asignado al nivel de suscripción se aplica a todas las suscripciones de ese grupo de administración.
- **Suscripción**: Un rol de Azure asignado al nivel de suscripción se aplica a todos los recursos y grupos de recursos de esa suscripción.
- **Grupo de recursos**: Un rol de Azure asignado al nivel de grupo de recursos se aplica a todos los recursos de dicho grupo de recursos.
- **Recurso específico**: un rol de Azure asignado a un recurso concreto se aplica a dicho recurso. En este caso, el recurso es un almacén de claves específico.

Existen varios roles predefinidos. Si un rol predefinido no se ajusta a sus necesidades, puede definir uno propio. Para más información, consulte [Azure RBAC: roles integrados](../../role-based-access-control/built-in-roles.md).

## <a name="data-plane-and-managed-hsm-local-rbac"></a>RBAC local de HSM administrado y plano de datos

Conceda a una entidad de seguridad acceso para ejecutar operaciones con claves específicas asignando un rol. Para cada asignación de roles, debe especificar un rol y ámbito sobre los que se aplica esa asignación. Para RBAC local de HSM administrado, hay dos ámbitos disponibles.

- **"/" o "/keys"** : ámbito de nivel de HSM. Las entidades de seguridad asignadas a un rol en este ámbito pueden realizar las operaciones definidas en el rol para todos los objetos (claves) en el HSM administrado.
- **"/keys/&lt;key-name&gt;"** : ámbito de nivel de clave. Las entidades de seguridad asignadas a un rol en este ámbito pueden realizar las operaciones definidas en este rol para todas las versiones de la clave especificada únicamente.

## <a name="next-steps"></a>Pasos siguientes

- Para seguir un tutorial de introducción para un administrador, consulte el documento sobre [qué es HSM administrado](overview.md).
- Para ver un tutorial de administración de roles, consulte [RBAC local de HSM administrado](role-management.md)
- Para obtener más información sobre el registro de uso del registro de HSM administrado, consulte el documento sobre el [registro de HSM administrado](logging.md).