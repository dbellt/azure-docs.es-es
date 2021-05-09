---
title: Permisos del catálogo (versión preliminar)
description: En este artículo se proporciona información general sobre cómo configurar el control de acceso basado en rol (RBAC) en Azure Purview.
author: yarong
ms.author: yarong
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 10/20/2020
ms.openlocfilehash: 9c27287f661c133641f7b3da45784478c8fc597e
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108016018"
---
# <a name="role-based-access-control-in-azure-purviews-data-plane"></a>Control de acceso basado en rol en el plano de datos de Azure Purview

En este artículo se describe cómo se implementa el control de acceso basado en rol (RBAC) en el [plano de datos](../azure-resource-manager/management/control-plane-and-data-plane.md#data-plane) de Azure Purview.

> [!IMPORTANT]
> A la entidad de seguridad que ha creado una cuenta de Purview se le conceden automáticamente todos los permisos del plano de datos, independientemente de los roles del plano de datos en los que pueda estar. Para que cualquier otro usuario realice cualquier acción en Azure Purview, debe encontrarse en al menos uno de los roles predefinidos del plano de datos.

## <a name="azure-purviews-pre-defined-data-plane-roles"></a>Roles predefinidos del plano de datos de Azure Purview

Azure Purview define un conjunto de roles predefinidos del plano de datos que se pueden usar para controlar quién puede acceder a qué en Azure Purview. Estos roles son:

* **Rol Lector de datos de Purview**: tiene acceso al portal de Purview y puede leer todo el contenido de Azure Purview, excepto los enlaces de examen.
* **Rol Conservador de datos de Purview**: tiene acceso al portal de Purview y puede leer todo el contenido de Azure Purview, excepto los enlaces de examen, puede editar información sobre los recursos, editar definiciones de clasificación y términos del glosario, y puede aplicar clasificaciones y términos del glosario a los recursos.
* **Rol Administrador de orígenes de datos de Purview**: no tiene acceso al portal de Purview (el usuario también debe estar en los roles Lector de datos o Conservador de datos) y puede administrar todos los aspectos del examen de datos en Azure Purview, pero no tiene acceso de lectura o escritura al contenido de Azure Purview más allá de lo relacionado con el examen.

## <a name="understanding-how-to-use-azure-purviews-data-plane-roles"></a>Descripción del uso de los roles del plano de datos de Azure Purview

Cuando se crea una cuenta de Azure Purview, el creador se trata como si estuviera en los roles Conservador de datos de Purview y Administrador de orígenes de datos de Purview. Pero el creador de la cuenta no se asigna a estos roles en el almacén de roles. Azure Purview reconoce que la entidad de seguridad es el creador de la cuenta y extiende estas funcionalidades en función de su identidad.

Todos los demás usuarios solo pueden usar la cuenta de Azure Purview si se encuentran en al menos uno de estos roles. Esto significa que, cuando se crea una cuenta de Azure Purview, nadie salvo el creador puede acceder a la cuenta o usar sus API hasta que se asignan a uno o varios de los roles definidos anteriormente.

Tenga en cuenta que el rol Administrador de orígenes de datos de Purview tiene dos escenarios admitidos. El primer escenario es para los usuarios que ya están en el rol Lector de datos de Purview o Conservador de datos de Purview, y que también necesitan poder crear exámenes. Esos usuarios deben tener dos roles, al menos uno entre Lector de datos de Purview o Conservador de datos de Purview, así como estar asignados al rol Administrador de orígenes de datos de Purview.

El otro escenario para el Administrador de orígenes de datos de Purview es para los procesos de programación, como las entidades de servicio, que necesitan poder configurar y supervisar exámenes, pero no deben tener acceso a los datos del catálogo.

Para implementar este escenario, asigne a la entidad de servicio el rol Administrador de orígenes de datos de Purview sin asignar ninguno de los otros dos roles. La entidad de seguridad no tendrá acceso al portal de Purview, pero eso es correcto porque es una entidad de seguridad de programación y solo se comunica mediante las API.

## <a name="putting-users-into-roles"></a>Asignación de usuarios a roles

Por lo tanto, la primera tarea en la empresa después de crear una cuenta de Azure Purview es asignar personas a estos roles.

La asignación de roles se administra mediante [RBAC de Azure](../role-based-access-control/overview.md).

Solo dos roles integrados del plano de control en Azure pueden asignar roles de usuario, son los Propietarios o Administradores de acceso de usuarios. Por lo tanto, para asignar roles a los usuarios en Azure Purview, debe encontrar a alguien que sea Propietario o Administrador de acceso de usuarios o que se convierta en uno de ellos.

### <a name="an-example-of-assigning-someone-to-a-role"></a>Ejemplo de asignación de un usuario a un rol

1. Vaya a https://portal.azure.com y vaya a la cuenta de Azure Purview.
1. Haga clic en "Control de acceso (IAM)" en el lado izquierdo.
1. A continuación, siga las instrucciones generales que se indican [aquí](../role-based-access-control/quickstart-assign-role-user-portal.md#create-a-resource-group).

## <a name="role-definitions-and-actions"></a>Definiciones de roles y acciones

Un rol se define como una colección de acciones. Consulte [aquí](../role-based-access-control/role-definitions.md) para más información sobre cómo se definen los roles. Y consulte [aquí](../role-based-access-control/built-in-roles.md) para conocer las definiciones de roles de los roles de Azure Purview.

## <a name="getting-added-to-a-data-plane-role-in-an-azure-purview-account"></a>Incorporación a un rol del plano de datos en una cuenta de Azure Purview

Si desea tener acceso a una cuenta de Azure Purview para poder usar Studio o llamar a sus API, necesita que le agreguen a un rol del plano de datos de Azure Purview. Las únicas personas que pueden hacer esto son aquellas que son Propietarios o Administradores de acceso de usuarios en la cuenta de Azure Purview. Para la mayoría de los usuarios, el siguiente paso es encontrar un administrador local que pueda ayudar a encontrar a las personas adecuadas que les puedan dar acceso.

En el caso de los usuarios que tienen acceso a [Azure Portal](https://portal.azure.com) de la empresa, pueden buscar la cuenta de Azure Purview en particular a la que desean unirse, hacer clic en la pestaña "Control de acceso (IAM)" y ver quiénes son los propietarios o los administradores de acceso de usuarios (UAA). Pero tenga en cuenta que, en algunos casos, los grupos de Azure Active Directory o las entidades de servicio se pueden usar como propietarios o UAA, en cuyo caso puede que no sea posible ponerse en contacto con ellos directamente. En su lugar, debe buscar un administrador para obtener ayuda.

## <a name="who-should-be-assigned-to-what-role"></a>¿A quién se le debe asignar cada rol?

|Escenario de usuario|Roles adecuados|
|-------------|-----------------|
|Solo necesito buscar recursos, no deseo editar nada.|Rol Lector de datos de Purview|
|Necesito editar información sobre los recursos, poner clasificaciones en ellos, asociarlos a entradas del glosario, etc.|Rol Conservador de datos de Purview|
|Necesito editar el glosario o configurar nuevas definiciones de clasificación.|Rol Conservador de datos de Purview|
|La entidad de servicio de la aplicación debe insertar datos en Azure Purview.|Rol Conservador de datos de Purview|
|Necesito configurar exámenes mediante Purview Studio.|Rol Administrador de orígenes de datos de Purview más al menos uno de los roles Lector de datos de Purview o Conservador de datos de Purview|
|Necesito habilitar una entidad de servicio u otra identidad de programación para configurar y supervisar exámenes en Azure Purview sin permitir que la identidad de programación tenga acceso a la información del catálogo. |Rol Administrador de orígenes de datos de Purview|
|Necesito asignar usuarios a roles en Azure Purview. | Propietario o Administrador de acceso de usuarios |

Para obtener más información sobre cómo agregar una entidad de seguridad a un rol, consulte [Inicio rápido: Creación de una cuenta de Azure Purview](create-catalog-portal.md).

## <a name="next-steps"></a>Pasos siguientes

* [Información de datos](concept-insights.md)
