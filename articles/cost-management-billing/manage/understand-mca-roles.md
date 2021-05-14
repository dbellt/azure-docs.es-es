---
title: 'Roles de facturación del Contrato de cliente de Microsoft: Azure'
description: Aprenda sobre los roles de facturación para las cuentas de facturación de Azure para los contratos de cliente de Microsoft.
author: amberbhargava
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 03/10/2021
ms.author: banders
ms.openlocfilehash: e334a423fd11aa3a357d52099a792dcc905aedeb
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/11/2021
ms.locfileid: "103011670"
---
# <a name="understand-microsoft-customer-agreement-administrative-roles-in-azure"></a>Descripción de los roles administrativos del contrato de cliente de Microsoft en Azure

Para administrar la cuenta de facturación para un contrato de cliente de Microsoft, use los roles descritos en las secciones siguientes. Estos roles son adicionales a los roles integrados que tiene Azure para controlar el acceso a los recursos. Para más información, consulte [Roles integrados en Azure](../../role-based-access-control/built-in-roles.md).

Este artículo se aplica a una cuenta de facturación para un contrato de cliente de Microsoft. [Compruebe si tiene acceso a un contrato de cliente de Microsoft](#check-access-to-a-microsoft-customer-agreement).

Vea el vídeo [Administración del acceso a la cuenta de facturación de MCA](https://www.youtube.com/watch?v=9sqglBlKkho) para aprender a controlar el acceso a su cuenta de facturación del Contrato de cliente de Microsoft (MCA).

>[!VIDEO https://www.youtube.com/embed/9sqglBlKkho]

## <a name="billing-role-definitions"></a>Definiciones de roles de facturación

En la tabla siguiente se describen los roles de facturación que se utilizan para administrar la cuenta de facturación, los perfiles de facturación y las secciones de factura.

|Role|Descripción|
|---|---|
|Propietario de la cuenta de facturación|Administrar todo para la cuenta de facturación|
|Colaborador de la cuenta de facturación|Administrar todo, excepto los permisos, en la cuenta de facturación|
|Lector de la cuenta de facturación|Vista de solo lectura de todo en la cuenta de facturación|
|Propietario del perfil de facturación|Administrar todo para el perfil de facturación|
|Colaborador del perfil de facturación|Administrar todo, excepto los permisos, en el perfil de facturación|
|Lector del perfil de facturación|Vista de solo lectura de todo en el perfil de facturación|
|Administrador de facturación|Ver y pagar las facturas para el perfil de facturación|
|Propietario de la sección de facturas|Administrar todo en la sección de factura|
|Colaborador de la sección de factura|Administrar todo, excepto los permisos, en la sección de factura|
|Lector de la sección de facturas|Vista de solo lectura de todo en la sección de factura|
|Creador de la suscripción a Azure|Creación de suscripciones a Azure|

## <a name="billing-account-roles-and-tasks"></a>Tareas y roles de la cuenta de facturación

Cuando se registra para usar Azure, se crea una cuenta de facturación. Use su cuenta de facturación para administrar las facturas, los pagos y hacer seguimiento de los costos. Los roles de la cuenta de facturación tienen el máximo nivel de permisos y los usuarios de estos roles pueden ver el costo y la información de facturación de toda la cuenta. Asigne estos roles solo a los usuarios que necesiten ver las facturas y realizar el seguimiento de los costos de toda la cuenta, como los miembros de los equipos de finanzas y contabilidad. Para más información, consulte [Descripción de la cuenta de facturación](../understand/mca-overview.md#your-billing-account).

En las tablas siguientes se muestra qué rol necesita para completar las tareas en el contexto de la cuenta de facturación.

### <a name="manage-billing-account-permissions-and-properties"></a>Administración de las propiedades y permisos de la cuenta de facturación

|Tarea|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación|
|---|---|---|---|
|Ver las asignaciones de roles para la cuenta de facturación|✔|✔|✔|
|Conceder a otros usuarios permisos para consultar y administrar la cuenta de facturación|✔|✘|✘|
|Consultar las propiedades de la cuenta de facturación, como la dirección, los contratos, etc.|✔|✔|✔|
|Actualizar las propiedades de la cuenta de facturación, como el comprador, el nombre para mostrar, etc.|✔|✔|✘|

### <a name="manage-billing-profiles-for-billing-account"></a>Administrar perfiles de facturación para la cuenta de facturación

|Tarea|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación|
|---|---|---|---|
|Ver todos los perfiles de facturación de la cuenta|✔|✔|✔|
|Crear perfiles de facturación nuevos|✔|✔|✘|

### <a name="manage-invoices-for-billing-account"></a>Administrar las facturas de la cuenta de facturación

|Tarea|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación|
|---|---|---|---|
|Consultar todas las facturas de la cuenta|✔|✔|✔|
|Pagar las facturas con tarjeta de crédito|✔|✔|✘|
|Descargar las facturas, los archivos de uso de Azure, las hojas de precios y los documentos fiscales|✔|✔|✔|

### <a name="manage-products-for-billing-account"></a>Administrar los productos de la cuenta de facturación

|Tarea|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación|
|---|---|---|---|
|Ver todos los productos comprados para la cuenta|✔|✔|✔|
|Administrar la facturación para los productos, como cancelar, desactivar la renovación automática, etc.|✔|✔|✘|

### <a name="manage-subscriptions-for-billing-account"></a>Administración de suscripciones para la cuenta de facturación

|Tarea|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación|
|---|---|---|---|
|Ver todas las suscripciones de Azure creadas para la cuenta de facturación|✔|✔|✔|
|Crear suscripciones de Azure nuevas|✔|✔|✘|
|Cancelar suscripciones de Azure|✘|✘|✘|

## <a name="billing-profile-roles-and-tasks"></a>Tareas y roles del perfil de facturación

Cada cuenta de facturación tiene al menos un perfil de facturación. El primer perfil de facturación se configura al registrarse para usar Azure. Para el perfil de facturación se genera una factura mensual, con todos los cargos asociados del mes anterior. Puede configurar más perfiles de facturación en función de sus necesidades. Los usuarios con roles en un perfil de facturación pueden ver el costo, establecer el presupuesto, y administrar y pagar las facturas. Asigne estos roles a los usuarios responsables de administrar el presupuesto y pagar las facturas del perfil de facturación, como los miembros de los equipos de administración de la organización. Para más información, consulte [Descripción de los perfiles de facturación](../understand/mca-overview.md#billing-profiles).

En las tablas siguientes se muestra qué rol necesita para completar las tareas en el contexto del perfil de facturación.

### <a name="manage-billing-profile-permissions-properties-and-policies"></a>Administración de las directivas, propiedades y permisos de perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver las asignaciones de roles para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|
|Conceder a otros usuarios permisos para consultar y administrar el perfil de facturación|✔|✘|✘|✘|✔|✘|✘|
|Ver las propiedades del perfil de facturación, como el número del pedido de compra, la facturación, etc.|✔|✔|✔|✔|✔|✔|✔|
|Actualizar las propiedades del perfil de facturación |✔|✔|✘|✘|✔|✔|✘|
|Ver las directivas aplicadas en el perfil de facturación, como las compras de reservas de Azure, las compras de Azure Marketplace, etc.|✔|✔|✔|✔|✔|✔|✔|
|Aplicar las directivas en el perfil de facturación |✔|✔|✘|✘|✔|✔|✘|

### <a name="manage-invoices-for-billing-profile"></a>Administrar las facturas para el perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver todas las facturas para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|
|Pagar las facturas con tarjeta de crédito|✔|✔|✘|✔|✔|✘|✘|
|Descargar facturas, los archivos de uso y de cargos de Azure, las hojas de precios y los documentos fiscales para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|

### <a name="manage-invoice-sections-for-billing-profile"></a>Administración de las secciones de factura para el perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver todas las secciones de factura para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|
|Crear una nueva sección de factura para el perfil de facturación|✔|✔|✘|✘|✔|✔|✘|

### <a name="manage-products-for-billing-profile"></a>Administrar los productos para el perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver todos los productos comprads para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|
|Administrar la facturación para los productos, como cancelar, desactivar la renovación automática, etc.|✔|✔|✘|✘|✔|✔|✘|
|Cambiar perfil de facturación de los productos|✔|✔|✘|✘|✔|✔|✘|

### <a name="manage-payment-methods-for-billing-profile"></a>Administrar los métodos de pago para el perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver los métodos de pago para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|
|Administrar los métodos de pago, como reemplazar la tarjeta de crédito, desvincular la tarjeta de crédito, etc.|✔|✔|✘|✘|✔|✔|✘|
|Realizar el seguimiento del saldo del crédito de Azure para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|

### <a name="manage-subscriptions-for-billing-profile"></a>Administrar las suscripciones para el perfil de facturación

|Tarea|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación|Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación
|---|---|---|---|---|---|---|---|
|Ver todas las suscripciones de Azure para el perfil de facturación|✔|✔|✔|✔|✔|✔|✔|
|Crear suscripciones de Azure nuevas|✔|✔|✘|✘|✔|✔|✘|
|Cancelar suscripciones de Azure|✘|✘|✘|✘|✘|✘|✘|
|Cambiar el perfil de facturación de las suscripciones de Azure|✔|✔|✘|✘|✔|✔|✘|

## <a name="invoice-section-roles-and-tasks"></a>Tareas y roles de la sección de factura

Cada perfil de facturación contiene una sección de factura de forma predeterminada. Puede crear más secciones de factura para agrupar los costos en la factura del perfil de facturación.  Los usuarios con roles en la sección de factura controlan quién crea suscripciones de Azure y realizan otras compras. Asigne estos roles a los usuarios que configuren el entorno de Azure para los equipos de nuestra organización, como los jefes de ingeniería y los arquitectos técnicos. Para más información, consulte [Descripción de las secciones de factura](../understand/mca-overview.md#invoice-sections).

En las tablas siguientes se muestra qué rol necesita para completar las tareas en el contexto de las secciones de factura.

### <a name="manage-invoice-section-permissions-and-properties"></a>Administración de los permisos y propiedades de la sección de factura

|Tareas|Propietario de la sección de facturas|Colaborador de la sección de factura|Lector de la sección de facturas|Creador de la suscripción a Azure|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación |Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación 
|---|---|---|---|---|---|---|---|---|---|---|---|
|Ver las asignaciones de roles para la sección de factura|✔|✔|✔|✘|✔|✔|✔|✔|✔|✔|✔|
|Conceder a otros usuarios permisos para consultar y administrar la sección de factura|✔|✘|✘|✘|✔|✘|✘|✘|✔|✘|✘|
|Ver las propiedades de la sección de factura|✔|✔|✔|✘|✔|✔|✔|✔|✔|✔|✔|
|Actualizar las propiedades de la sección de factura|✔|✔|✘|✘|✔|✔|✘|✘|✔|✔|✘|

### <a name="manage-products-for-invoice-section"></a>Administrar los productos para la sección de factura

|Tareas|Propietario de la sección de facturas|Colaborador de la sección de factura|Lector de la sección de facturas|Creador de la suscripción a Azure|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación |Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación 
|---|---|---|---|---|---|---|---|---|---|---|---|
|Ver todos los productos comprados en la sección de factura|✔|✔|✔|✘|✔|✔|✔|✔|✔|✔|✔|
|Administrar la facturación para los productos, como cancelar, desactivar la renovación automática, etc.|✔|✔|✘|✘|✔|✔|✘|✘|✔|✔|✘|
|Cambiar la sección de factura para los productos|✔|✔|✘|✘|✔|✔|✘|✘|✔|✔|✘|

### <a name="manage-subscriptions-for-invoice-section"></a>Administrar las suscripciones para la sección de factura

|Tareas|Propietario de la sección de facturas|Colaborador de la sección de factura|Lector de la sección de facturas|Creador de la suscripción a Azure|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación |Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación 
|---|---|---|---|---|---|---|---|---|---|---|---|
|Visualización de todas las suscripciones a Azure para la sección de factura|✔|✔|✔|✘|✔|✔|✔|✔|✔|✔|✔|
|Creación de suscripciones a Azure|✔|✔|✘|✔|✔|✔|✘|✘|✔|✔|✘|
|Cancelar suscripciones de Azure|✘|✘|✘|✘|✘|✘|✘|✘|✘|✘|✘|
|Cambiar la sección de factura para la suscripción de Azure|✔|✔|✘|✘|✔|✔|✘|✘|✔|✔|✘|
|Solicitud de la propiedad de la facturación de las suscripciones de usuarios de otras cuentas de facturación|✔|✔|✘|✘|✔|✔|✘|✘|✔|✔|✘|

## <a name="subscription-billing-roles-and-tasks"></a>Tareas y roles de la facturación de suscripción

En la tabla siguiente se muestra qué rol necesita para completar las tareas en el contexto de una suscripción.

|Tareas|Propietario de la sección de facturas|Colaborador de la sección de factura|Lector de la sección de facturas|Creador de la suscripción a Azure|Propietario del perfil de facturación|Colaborador del perfil de facturación|Lector del perfil de facturación |Administrador de facturación|Propietario de la cuenta de facturación|Colaborador de la cuenta de facturación|Lector de la cuenta de facturación 
|---|---|---|---|---|---|---|---|---|---|---|---|
|Creación de suscripciones|✔|✔|✘|✔|✔|✔|✘|✘|✔|✔|✘|
|Actualizar el centro de coste para la suscripción|✔|✔|✘|✘|✔|✔|✘|✘|✔|✔|✘|
|Cambiar la sección de factura para la suscripción|✔|✔|✘|✘|✔|✔|✘|✘|✔|✔|✘|
|Cambiar el perfil de facturación para la suscripción|✘|✘|✘|✘|✔|✔|✘|✘|✔|✔|✘|
|Cancelar suscripciones de Azure|✘|✘|✘|✘|✘|✘|✘|✘|✘|✘|✘|

## <a name="manage-billing-roles-in-the-azure-portal"></a>Administración de roles integrados en Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Busque **Administración de costos + facturación**.

   ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/understand-mca-roles/billing-search-cost-management-billing.png)

3. Seleccione **Control de acceso (IAM)** en un ámbito como cuenta de facturación, perfil de facturación o sección de factura, donde desee dar acceso.

4. La página de Control de acceso (IAM) muestra los usuarios y grupos asignados a cada rol para ese ámbito.

   ![Captura de pantalla que muestra una lista de administradores para la cuenta de facturación](./media/understand-mca-roles/billing-list-admins.png)

5. Para dar acceso a un usuario, seleccione **Agregar** en la parte superior de la página. En la lista desplegable Rol, seleccione un rol. Escriba la dirección de correo electrónico del usuario al que desea dar acceso. Seleccione **Guardar** para asignar el rol.

   ![Captura de pantalla que muestra cómo agregar un administrador a una cuenta de facturación](./media/understand-mca-roles/billing-add-admin.png)

6. Para quitar el acceso a un usuario, seleccione dicho usuario con la asignación de roles que desea quitar. Seleccione Quitar.

   ![Captura de pantalla que se muestra cómo eliminar un administrador de una cuenta de facturación](./media/understand-mca-roles/billing-remove-admin.png)

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Comprobación del acceso a un contrato de cliente de Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Ponerse en contacto con soporte técnico
Si necesita ayuda, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para obtener información sobre la cuenta de facturación:

- [Empezar a trabajar con la cuenta de facturación para un contrato de cliente de Microsoft](../understand/mca-overview.md)
- [Creación de una suscripción a Azure para la cuenta de facturación del contrato de cliente de Microsoft](create-subscription.md)
