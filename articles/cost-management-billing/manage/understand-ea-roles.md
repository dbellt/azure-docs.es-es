---
title: Información sobre los roles de administrador en contratos Enterprise (EA) de Azure
description: Conozca los roles de administrador de empresa en Azure. Puede asignar cinco roles administrativos distintos.
author: bandersmsft
ms.reviewer: adwise
ms.service: cost-management-billing
ms.subservice: enterprise
ms.topic: conceptual
ms.date: 04/05/2021
ms.author: banders
ms.custom: contperf-fy21q1
ms.openlocfilehash: 7331f9a894d36ee15702a8fe53804efd53049762
ms.sourcegitcommit: c6a2d9a44a5a2c13abddab932d16c295a7207d6a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107284126"
---
# <a name="managing-azure-enterprise-agreement-roles"></a>Administración de los roles del Contrato Enterprise de Azure

Para ayudar a administrar el uso y gasto de su organización, los clientes de Azure con un Contrato Enterprise pueden asignar seis roles de administrador diferentes:

- Administrador de empresa
- Administrador de empresa (solo lectura)<sup>1</sup>
- Comprador de EA
- Administrador de departamentos
- Administrador de departamento (solo lectura)
- Propietario de la cuenta <sup>2</sup>

<sup>1</sup> El contacto de facturación del Contrato Enterprise estará bajo este rol.

<sup>2</sup> No se puede agregar ni cambiar el contacto de destinatario de la factura en el portal de EA de Azure y se agregará a la inscripción de EA según el usuario que esté configurado como tal en el nivel de contrato. Para cambiar el contacto de destinatario de la factura, es necesario realizar una solicitud a través de un asesor de software o asociado al centro de operaciones regional (ROC).

El primer administrador de inscripciones que se configure durante el aprovisionamiento de la inscripción determina el tipo de autenticación de la cuenta de contacto de facturación. Cuando el contacto de facturación se agrega a Enterprise Portal como administrador de solo lectura, se le proporciona la autenticación de la cuenta Microsoft. 

Por ejemplo, si el tipo de autenticación inicial se establece en Mixto, EA se agregará como un cuenta Microsoft y el contacto de facturación tendrá privilegios de administrador de EA de solo lectura. Si el administrador de EA no aprueba la autorización de un cuenta Microsoft para un contacto de facturación existente, puede eliminar el usuario en cuestión y pedir al cliente que vuelva a agregarlo como administrador de solo lectura con una cuenta profesional o educativa establecida únicamente en el nivel de inscripción en el portal de EA.

Estos roles son específicos de la administración de Contratos Enterprise de Azure y son adicionales a los roles integrados que tiene Azure para controlar el acceso a los recursos. Para más información, consulte [Roles integrados en Azure](../../role-based-access-control/built-in-roles.md).

## <a name="azure-enterprise-portal-hierarchy"></a>Jerarquía de Azure Enterprise Portal

La jerarquía de Azure Enterprise Portal consta de:

- **Azure Enterprise Portal**: Portal de administración en línea que ayuda a administrar los costos de los servicios de Azure EA. Puede:

  - Crear una jerarquía de Azure EA con departamentos, cuentas y suscripciones.
  - Conciliar los costos de los servicios consumidos, descargar informes de uso y ver listas de precios.
  - Crear claves de API para la inscripción.

- Los **departamentos** ayudan a segmentar los costos en agrupaciones lógicas. Los departamentos le permiten establecer un presupuesto o una cuota a nivel de departamento.

- Las **cuentas** son unidades organizativas en Azure Enterprise Portal. Puede usar las cuentas para administrar las suscripciones y obtener acceso a los informes.

- Las **suscripciones** son la unidad más pequeña Azure Enterprise Portal. Son contenedores para los servicios de Azure que administra el administrador de servicios.

En el siguiente diagrama se ilustran las jerarquías simples de Azure EA.

![Diagrama de jerarquías simples de Azure EA](./media/understand-ea-roles/ea-hierarchies.png)

## <a name="enterprise-user-roles"></a>Roles de los usuarios de empresa

Los siguientes roles de usuario administrativo forman parte de la inscripción empresarial:

- Administrador de empresa
- Comprador de EA
- Administrador de departamentos
- Propietario de cuenta
- Administrador de servicios
- Contacto para notificaciones

Los roles funcionan en dos portales distintos para completar tareas. Use [Azure Enterprise Portal](https://ea.azure.com) para administrar la facturación y los costos, y [Azure Portal](https://portal.azure.com) para administrar los servicios de Azure.

Los roles de usuario están asociados a una cuenta de usuario. Para validar la autenticidad de los usuarios, todos ellos deben tener una cuenta profesional, educativa o de Microsoft válida. Asegúrese de que cada cuenta está asociada a una dirección de correo electrónico que se supervisa activamente. Las notificaciones de las cuentas se envían a la dirección de correo electrónico.

Al configurar usuarios, puede asignar varias cuentas al rol Administrador de empresa. Sin embargo, solo una cuenta puede contener el rol Propietario de la cuenta. Además, puede asignar tanto el rol Administrador de empresa como el rol Propietario de cuenta a una sola cuenta.

### <a name="enterprise-administrator"></a>Administrador de empresa

Los usuarios con este rol tienen el nivel de acceso más alto. Pueden realizar las acciones siguientes:

- Administrar cuentas y propietarios de cuentas.
- Administrar otros administradores de empresa.
- Administrar administradores de departamento.
- Administrar contactos de notificación.
- Comprar servicios de Azure, incluidas reservas.
- Ver el uso en todas las cuentas.
- Ver los cargos no facturados en todas las cuentas.
- Ver y administrar no solo todos los pedidos de reserva, sino también las propias reservas que se aplican al Contrato Enterprise.
  - El administrador de empresa (solo lectura) puede ver tanto los pedidos de reserva como las propias reservas, pero no pueden administrarlos.

Puede tener varios administradores de empresa en una inscripción empresarial. Puede conceder acceso de solo lectura a los administradores de empresa. Estos heredan el rol de administrador de departamento.

### <a name="ea-purchaser"></a>Comprador de EA

Los usuarios con este rol tienen permisos para comprar servicios de Azure, pero no se les permite administrar cuentas. Pueden realizar las acciones siguientes:

- Comprar servicios de Azure, incluidas reservas.
- Ver el uso en todas las cuentas.
- Ver los cargos no facturados en todas las cuentas.
- Ver y administrar no solo todos los pedidos de reserva, sino también las propias reservas que se aplican al Contrato Enterprise.

Actualmente, el rol de comprador de EA solo está habilitado para el acceso basado en SPN. Para aprender a asignar el rol a un nombre de entidad de seguridad de servicio, consulte [Asignación de roles a los nombres de entidad de seguridad de servicio del Contrato Enterprise de Azure](assign-roles-azure-service-principals.md).

### <a name="department-administrator"></a>Administrador de departamentos

Los usuarios con este rol pueden:

- Crear y administrar departamentos.
- Crear nuevos propietarios de cuentas.
- Ver detalles de uso de los departamentos que administran.
- Ver los costos, si tienen los permisos necesarios.

Puede tener varios administradores de departamento en cada inscripción empresarial.

Puede conceder a los administradores de departamento acceso de solo lectura al editar o crear un nuevo administrador de departamento. Establezca la opción de solo lectura en **Sí**.

### <a name="account-owner"></a>Propietario de cuenta

Los usuarios con este rol pueden:

- Crear y administrar suscripciones.
- Administrar administradores de servicios.
- Ver el uso de las suscripciones.

Todas las cuentas requieren una cuenta profesional, educativa o de Microsoft única. Para obtener más información sobre los roles administrativos de Azure Enterprise Portal, consulte [Roles administrativos del Contrato Enterprise de Azure en Azure](understand-ea-roles.md).

### <a name="service-administrator"></a>Administrador de servicios

El rol Administrador de servicios tiene permisos para administrar servicios en Azure Portal y asignar a los usuarios el rol de coadministrador.

### <a name="notification-contact"></a>Contacto para notificaciones

El contacto para notificaciones recibe las notificaciones de uso relacionadas con la inscripción.

En las secciones siguientes se describen las limitaciones y funcionalidades de cada rol.

## <a name="user-limit-for-admin-roles"></a>Límite de usuarios para roles de administrador

|Role| Límite de usuarios|
|---|---|
|Administrador de empresa|Sin límite|
|Administrador de empresa (solo lectura)|Sin límite|
| Comprador de EA asignado a un nombre de entidad de seguridad de servicio | Sin límite |
|Administrador de departamentos|Sin límite|
|Administrador de departamento (solo lectura)|Sin límite|
|Propietario de cuenta|1 por cuenta <sup>3</sup>|

<sup>3</sup> Cada cuenta requiere una única cuenta de Microsoft o una cuenta profesional o educativa.

## <a name="organization-structure-and-permissions-by-role"></a>Estructura de organización y permisos por rol

|Tareas| Administrador de empresa|Administrador de empresa (solo lectura)| Comprador de EA | Administrador de departamentos|Administrador de departamento (solo lectura)|Propietario de cuenta| Asociado|
|---|---|---|---|---|---|---|---|
|Ver administradores de empresa|✔|✔| ✔|✘|✘|✘|✔|
|Agregar o quitar administradores de empresa|✔|✘|✘|✘|✘|✘|✘|
|Ver contactos de notificación<sup>4</sup> |✔|✔|✔|✘|✘|✘|✔|
|Agregar o quitar contactos de notificación<sup>4</sup> |✔|✘|✘|✘|✘|✘|✘|
|Crear y administrar departamentos |✔|✘|✘|✘|✘|✘|✘|
|Ver administradores de departamento|✔|✔|✔|✔|✔|✘|✔|
|Agregar o quitar administradores de departamento|✔|✘|✘|✔|✘|✘|✘|
|Ver cuentas en la inscripción |✔|✔|✔|✔<sup>5</sup>|✔<sup>5</sup>|✘|✔|
|Agregar cuentas a la inscripción y cambiar el propietario de la cuenta|✔|✘|✘|✔<sup>5</sup>|✘|✘|✘|
|Compra de reservas|✔|✘|✔|✘|✘|✘|✘|
|Crear y administrar suscripciones y permisos de suscripción|✘|✘|✘|✘|✘|✔|✘|

- <sup>4</sup> A los contactos de notificación se les envían comunicaciones por correo electrónico sobre el Contrato Enterprise de Azure.
- <sup>5</sup> La tarea se limita a las cuentas de su departamento.

## <a name="add-a-new-enterprise-administrator"></a>Adición de un nuevo administrador de empresa

Los administradores de empresa tienen los máximos privilegios al administrar una inscripción del Contrato Enterprise de Azure. Se creó el administrador del Contrato Enterprise de Azure inicial cuando se configuró el acuerdo del Contrato Enterprise. Sin embargo, puede agregar o quitar nuevos administradores en cualquier momento. Solo los administradores existentes agregan nuevos administradores. Para más información sobre cómo agregar administradores de empresa adicionales, consulte [Creación de otro administrador de empresa](ea-portal-administration.md#create-another-enterprise-administrator). Para más información acerca de los roles del perfil de facturación, consulte [Tareas y roles del perfil de facturación](understand-mca-roles.md#billing-profile-roles-and-tasks).

## <a name="update-account-owner-state-from-pending-to-active"></a>Actualización del estado del propietario de la cuenta, de pendiente a activo

Cuando se agregan nuevos propietarios de la cuenta a una inscripción del Contrato Enterprise de Azure por primera vez, su estado aparece como _pendiente_. Cuando un nuevo propietario de la cuenta recibe el correo electrónico de bienvenida de activación, puede iniciar sesión para activar su cuenta. Al activar su cuenta, el estado de la cuenta se actualiza de _pendiente_ a _activa_. El propietario de la cuenta tiene que leer el mensaje de advertencia y seleccionar **Continuar**. Es posible que se pida a los nuevos usuarios que escriban su nombre y apellidos para crear una cuenta comercial. En ese caso, deben agregar la información necesaria para continuar y, acto seguido, se activará la cuenta.

## <a name="add-a-department-admin"></a>Adición de un administrador de departamento

Cuando un administrador de Contrato Enterprise de Azure crea un departamento, el administrador puede agregar administradores de departamento y asociar cada uno de ellos a un departamento. Un administrador de departamento puede crear nuevas cuentas. Se necesitan nuevas cuentas para que se creen las suscripciones del Contrato Enterprise de Azure.

Para más información sobre cómo agregar un administrador de departamento, consulte [Creación de un administrador de departamento del Contrato Enterprise de Azure](ea-portal-administration.md#add-a-department-administrator).

## <a name="usage-and-costs-access-by-role"></a>Acceso de uso y costos por rol

|Tareas| Administrador de empresa|Administrador de empresa (solo lectura)|Comprador de EA|Administrador de departamentos|Administrador de departamento (solo lectura) |Propietario de cuenta| Asociado|
|---|---|---|---|---|---|---|---|
|Ver saldo de crédito, incluido el prepago de Azure|✔|✔|✔|✘|✘|✘|✔|
|Ver cuotas de gastos de departamento|✔|✔|✔|✘|✘|✘|✔|
|Establecer cuotas de gasto de departamento|✔|✘|✘|✘|✘|✘|✘|
|Ver la hoja de precios de EA de la organización|✔|✔|✔|✘|✘|✘|✔|
|Ver los detalles de uso y costo|✔|✔|✔|✔<sup>6</sup>|✔<sup>6</sup>|✔<sup>7</sup>|✔|
|Administrar recursos en Azure Portal|✘|✘|✘|✘|✘|✔|✘|

- <sup>6</sup> Requiere que el administrador de empresa habilite la directiva de **visualización de cargos del administrador de departamento** en Enterprise Portal. El administrador de departamento puede entonces ver los detalles de costo del departamento.
- <sup>7</sup> Requiere que el administrador de empresa habilite la directiva de **visualización de cargos del propietario de la cuenta** en Enterprise Portal. El propietario de la cuenta puede ver entonces los detalles del costo de la cuenta.

## <a name="see-pricing-for-different-user-roles"></a>Consulte los precios de los diferentes roles de usuario

Es posible que vea un precio diferente en Azure Portal según su rol administrativo y cómo el administrador de empresa haya establecido las directivas de visualización de los cargos. Las dos directivas de Enterprise Portal que afectan a los precios que ve en Azure Portal son:

- Visualización de cargos del AD
- Visualización de cargos del PC

Para saber cómo establecer estas directivas, consulte [Administración del acceso a la información de facturación en Azure](manage-billing-access.md).

En la tabla siguiente se muestra la relación entre los roles de administrador del Contrato Enterprise, la directiva de visualización de cargos, el rol de Azure en Azure Portal y los precios que ve en Azure Portal. El administrador de empresa siempre ve los detalles de uso según los precios de EA de la organización. Sin embargo, el administrador de departamento y el propietario de la cuenta tienen vistas de precios diferentes según la directiva de visualización de cargos y su rol de Azure. El rol de administrador de departamento que se muestra en la tabla siguiente hace referencia a los roles de administrador de departamento y administrador de departamento (solo lectura).

|Rol de administrador de Contrato Enterprise|Directiva de visualización de cargos por rol|Rol de Azure|Vista de precios|
|---|---|---|---|
|Propietario de la cuenta o administrador de departamento|✔ Habilitado|Propietario|Precios de EA de la organización|
|Propietario de la cuenta o administrador de departamento|✘ Deshabilitado|Propietario|Precio de venta|
|Propietario de la cuenta o administrador de departamento|✔ Habilitado |None|No hay precios|
|Propietario de la cuenta o administrador de departamento|✘ Deshabilitado |None|No hay precios|
|None|No aplicable |Propietario|Precio de venta|

El rol de administrador de Enterprise y las directivas de visualización de cargos se establecen en Enterprise Portal. El rol de Azure puede actualizarse en Azure Portal. Para más información, consulte [Asignación de roles de Azure mediante Azure Portal](../../role-based-access-control/role-assignments-portal.md).

## <a name="next-steps"></a>Pasos siguientes

- [Administración del acceso a la información de facturación de Azure](manage-billing-access.md)
- [Asignación de roles de Azure mediante Azure Portal](../../role-based-access-control/role-assignments-portal.md)
- [Roles integrados en los recursos de Azure](../../role-based-access-control/built-in-roles.md)
