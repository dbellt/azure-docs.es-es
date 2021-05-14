---
title: Creación de planes para una oferta de servicio administrado en Azure Marketplace
description: Aprenda a configurar planes para la oferta de servicio administrado en Azure Marketplace mediante el Centro de partners de Microsoft.
author: Microsoft-BradleyWright
ms.author: brwrigh
ms.reviewer: anbene
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 12/23/2020
ms.openlocfilehash: a20571e411b2849c3487582a9c316f0f0b35c91d
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109790944"
---
# <a name="how-to-create-plans-for-your-managed-service-offer"></a>Creación de planes para la oferta de servicio administrado

Las ofertas de servicio administrado vendidas mediante el marketplace comercial de Microsoft deben tener al menos un plan. Puede crear una variedad de planes con diferentes opciones dentro de la misma oferta. Estos planes (a veces denominados SKU) pueden diferir en lo tocante a versión, monetización o niveles de servicio. Para obtener instrucciones detalladas sobre los planes, consulte [Planes y precios de las ofertas del marketplace comercial](./plans-pricing.md).

## <a name="create-a-plan"></a>Creación de un plan

1. En la pestaña **Información general del plan** de la oferta del Centro de partners, seleccione **+ Crear nuevo plan**.
2. En el cuadro de diálogo que aparece, en **Id. de plan**, escriba un identificador de plan único. Use solo hasta 50 caracteres alfanuméricos en minúscula, guiones o caracteres de subrayado. No se puede modificar el identificador del plan después de seleccionar **Crear**. Este identificador será visible para los clientes.
3. En el cuadro **Nombre del plan**, escriba un nombre para este plan. Use un máximo de 50 caracteres. Este nombre será visible para los clientes.
4. Seleccione **Crear**.

## <a name="define-the-plan-listing"></a>Definición de la lista de planes

En la pestaña **Lista del plan**, defina el nombre y la descripción del plan como desee que aparezcan en el marketplace comercial.

1. En el cuadro **Nombre del plan**, se muestra el nombre que proporcionó anteriormente para este plan. Puede cambiarlo en cualquier momento. Este nombre aparecerá en el marketplace comercial como el título del plan de la oferta.
2. En el cuadro **Resumen del plan**, proporcione una descripción breve del plan, que se puede usar en los resultados de búsqueda de marketplace.
3. En el cuadro **Descripción del plan**, explique lo que hace que este plan sea único y las diferencias con respecto a otros planes de su oferta.
4. Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña.

## <a name="define-pricing-and-availability"></a>Definición del precio y la disponibilidad

El único modelo de precios disponible para las ofertas de servicio administrado es **Traiga su propia licencia (BYOL)** . Esto significa que facturará a sus clientes directamente los costos relacionados con esta oferta y Microsoft no le aplicará ningún cargo.

Puede configurar cada plan para que sea visible para todo el mundo o solo para una audiencia concreta (privada).

> [!NOTE]
> Los planes privados no son compatibles con las suscripciones que se establecen a través de un revendedor del programa Proveedor de soluciones en la nube (CSP).

> [!IMPORTANT]
> Una vez publicado un plan como público, no puede cambiarlo a privado. Para controlar qué clientes pueden aceptar su oferta y delegar recursos, use un plan privado. Con un plan público, no puede restringir la disponibilidad a determinados clientes ni a un determinado número de clientes, aunque puede dejar de vender el plan por completo si decide hacerlo. Puede quitar el acceso a una delegación después de que un cliente acepte una oferta solo si incluyó una autorización con la definición de rol establecida en Rol para eliminar la asignación de registros de servicios administrados al publicar la oferta. También puede ponerse en contacto con el cliente y solicitarle que retire el acceso.

## <a name="make-your-plan-public"></a>Conversión del plan en público

1. En **Plan visibility** (Visibilidad del plan), seleccione **Público**.
2. Seleccione **Guardar borrador**. Para volver a la pestaña Información general del plan, seleccione **Información general del plan** en la parte superior izquierda.
3. Para crear otro plan para esta oferta, seleccione **+ Crear nuevo plan**, en la pestaña **Información general del plan**.

## <a name="make-your-plan-private"></a>Conversión del plan en privado

Puede conceder acceso a un plan privado con los identificadores de suscripción de Azure. Puede agregar hasta 10 identificadores de suscripción manualmente o hasta 10 000 mediante un archivo .csv.

Para agregar hasta 10 identificadores de suscripción manualmente, haga lo siguiente:

1. En **Plan visibility** (Visibilidad del plan), seleccione **Privado**.
2. Escriba el identificador de suscripción de Azure del público al que quiere conceder acceso.
3. También tiene la opción de escribir una descripción de este público en el cuadro **Descripción**.
4. Para agregar otro identificador, seleccione **Add ID (Max 10)** (Agregar id. [10 máx.]).
5. Cuando haya terminado de agregar identificadores, seleccione **Guardar borrador**.

Para agregar hasta 10 000 identificadores de suscripción con un archivo CSV, haga lo siguiente:

1. En **Plan visibility** (Visibilidad del plan), seleccione **Privado**.
2. Seleccione el vínculo **Exportar público (CSV)** . Se descargará un archivo CSV.
3. Abra el archivo CSV. En la columna **Id.** , escriba los identificadores de suscripción de Azure a los que quiere conceder acceso.
4. En la columna  **Descripción** , tiene la opción de agregar una descripción para cada entrada.
5. En la columna  **Tipo** , agregue  **SubscriptionId**  a cada fila que tenga un identificador.
6. Guarde el archivo como un archivo CSV.
7. En el Centro de partners, seleccione el vínculo **Import Audience (csv)** (Importar audiencia [csv]).
8. En el cuadro de diálogo  **Confirmar** , seleccione  **Sí** y, luego, cargue el archivo CSV.
9. Seleccione  **Guardar borrador**.

## <a name="technical-configuration"></a>Configuración técnica

En esta sección se crea un manifiesto con información de autorización para administrar los recursos de los clientes. Esta información es necesaria para habilitar la [administración de recursos delegados de Azure](../lighthouse/concepts/architecture.md).

Revise [Inquilinos, roles y usuarios de los escenarios de Azure Lighthouse](../lighthouse/concepts/tenants-users-roles.md#best-practices-for-defining-users-and-roles) para comprender qué roles se admiten y cuáles son los procedimientos recomendados para definir las autorizaciones.

> [!NOTE]
> Los usuarios y roles de las entradas de autorización se aplicarán a todos los clientes que activen el plan. Si quiere limitar el acceso a un cliente específico, deberá publicar un plan privado para uso exclusivo.

### <a name="manifest"></a>Manifest

1. En **Manifiesto**, proporcione una **versión** para el manifiesto. Use el formato n.n.n (por ejemplo, 1.2.5).
2. Escriba su **identificador de inquilino**. Se trata de un GUID asociado con el identificador del inquilino de Azure Active Directory (Azure AD) de la organización; es decir, el inquilino de administración desde el que obtendrá acceso a los recursos de sus clientes. Si no lo tiene a mano, para buscarlo, mantenga el mouse sobre el nombre de la cuenta en la parte superior derecha de Azure Portal o seleccione **Cambiar directorio**.

Si publica una nueva versión de la oferta y necesita crear un manifiesto actualizado, seleccione **+ Nuevo manifiesto**. Asegúrese de aumentar el número de versión de la versión anterior del manifiesto.

### <a name="authorizations"></a>Autorizaciones

Las autorizaciones definen las entidades del inquilino de administración que pueden tener acceso a los recursos y las suscripciones para los clientes que compran el plan. A cada una de estas entidades se le asigna un rol integrado que concede niveles de acceso específicos.

Puede crear hasta veinte autorizaciones para cada plan.

> [!TIP]
> En la mayoría de los casos, querrá asignar roles a una entidad de servicio o un grupo de usuarios de Azure AD, en lugar de a una serie de cuentas de usuario individuales. Esto le permite agregar o quitar el acceso de usuarios individuales sin tener que actualizar y volver a publicar el plan cuando cambien los requisitos de acceso. Al asignar roles a grupos de Azure AD, [el tipo de grupo debe ser Seguridad y no Office 365](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md). Para recomendaciones adicionales, consulte [Inquilinos, roles y usuarios en escenarios de Azure Lighthouse](../lighthouse/concepts/tenants-users-roles.md).

Para cada autorización, debe proporcionar la siguiente información. Puede seleccionar **+ Agregar autorización** tantas veces como sea necesario para agregar más definiciones de roles y usuarios.

* **Id. de objeto de AAD**: el identificador de Azure AD de un usuario, un grupo de usuarios o una aplicación al que se concederán determinados permisos (según se indica en la definición de roles) para los recursos de los clientes.
* **AAD Object Display Name** (Nombre para mostrar de objeto de AAD): un nombre descriptivo que ayuda al cliente a comprender el propósito de esta autorización. El cliente verá este nombre al delegar recursos.
* **Definición de roles**: seleccione uno de los roles integrados de Azure AD disponibles de la lista. Este rol determinará los permisos que el usuario del campo **Id. de la entidad de seguridad** tendrá en los recursos de los clientes. Para obtener descripciones de estos roles, consulte [Roles integrados](../role-based-access-control/built-in-roles.md) y [Soporte de roles para Azure Lighthouse](../lighthouse/concepts/tenants-users-roles.md#role-support-for-azure-lighthouse).

> [!NOTE]
> A medida que se agreguen nuevos roles integrados aplicables a Azure, estarán disponibles aquí. Puede que haya algún retraso antes de que aparezcan.

* **Assignable Roles** (Roles asignables): esta opción solo aparece si ha seleccionado Administrador de acceso de usuario en **Definición de roles** para esta autorización. En ese caso, debe agregar uno o varios roles asignables aquí. El usuario del campo **Id. de objeto de Azure AD** podrá asignar estos roles a identidades administradas, algo necesario para [implementar directivas que pueden corregirse](../lighthouse/how-to/deploy-policy-remediation.md). No se aplicará a este usuario ningún otro permiso asociado normalmente al rol Administrador de acceso de usuario.

> [!TIP]
> Para asegurarse de que puede [quitar el acceso a una delegación](../lighthouse/how-to/remove-delegation.md), si es necesario, incluya una **autorización** con la **definición de rol** establecida en [Rol para eliminar la asignación de registros de servicios administrados](../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role). Si este rol no está asignado, solo un usuario puede quitar los recursos delegados del inquilino del cliente.

Cuando haya completado todas las secciones del plan, puede seleccionar **+ Crear nuevo plan** para crear planes adicionales. Cuando haya terminado, seleccione **Guardar borrador** antes de continuar.

## <a name="next-steps"></a>Pasos siguientes

* [Revisión y publicación](review-publish-offer.md)
