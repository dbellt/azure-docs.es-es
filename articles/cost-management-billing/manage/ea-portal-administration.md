---
title: Administración del portal del Contrato Enterprise de Azure
description: En este artículo se explican las tareas comunes que un administrador lleva a cabo en el portal del Contrato Enterprise de Azure.
author: bandersmsft
ms.author: banders
ms.date: 03/19/2021
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: enterprise
ms.reviewer: boalcsva
ms.custom: contperf-fy21q1
ms.openlocfilehash: 09e719408926c071777a7022264c10f14531067f
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110075527"
---
# <a name="azure-ea-portal-administration"></a>Administración del portal del Contrato Enterprise de Azure

En este artículo se explican las tareas comunes que un administrador lleva a cabo en el portal del Contrato Enterprise de Azure (https://ea.azure.com). El portal del Contrato Enterprise de Azure es un portal de administración en línea que ayuda a los clientes a administrar el costo de los servicios del Contrato Enterprise de Azure. Para obtener información introductoria acerca del portal del Contrato Enterprise de Azure, consulte el artículo [Introducción al portal del Contrato Enterprise de Azure](ea-portal-get-started.md).

## <a name="activate-your-enrollment"></a>Activación de la inscripción

Para activar el servicio, el administrador de empresa inicial abre [Azure Enterprise Portal](https://ea.azure.com) e inicia sesión con la dirección del correo electrónico de invitación.

Si le han asignado el rol de administrador de empresa, no es preciso que reciba el correo electrónico de activación. Vaya a [Azure Enterprise Portal](https://ea.azure.com) e inicie sesión con la dirección de correo electrónico de su cuenta profesional, educativa o de Microsoft y su contraseña.

Si tiene más de una inscripción, elija la que desea activar. De forma predeterminada, solo se muestran las inscripciones activas. Para ver el historial de inscripciones, desactive la opción **Activa** en la parte superior derecha de Azure Enterprise Portal.

En **Inscripción**, aparece el estado **Activa**.

![Ejemplo en el que se muestra una inscripción activa](./media/ea-portal-administration/ea-enrollment-status.png)

Los administradores de empresa de Azure existentes son los únicos que pueden crear otros administradores de empresa.

### <a name="create-another-enterprise-administrator"></a>Creación de otro administrador de empresa

Use algunas de las siguientes opciones según su situación.

#### <a name="if-youre-already-an-enterprise-administrator"></a>Si ya es un administrador de empresa

1. Inicie sesión en [Azure Enterprise Portal](https://ea.azure.com).
1. Vaya a **Administrar** > **Enrollment Detail** (Detalles de inscripción).
1. En la parte superior derecha, seleccione **Agregar administrador**.

Asegúrese de tener la dirección de correo electrónico del usuario y el método de autenticación preferido, como cuenta profesional, educativa o de Microsoft.

#### <a name="if-youre-not-an-enterprise-administrator"></a>Si no es administrador de empresa

Si no es administrador de empresa, póngase en contacto con uno para pedirle que le agregue a una inscripción. El administrador de empresa usa los pasos anteriores para agregarle como administrador de empresa. Una vez que se haya agregado a una inscripción, recibirá un correo electrónico de activación.

#### <a name="if-your-enterprise-administrator-cant-help-you"></a>Si el administrador de empresa no puede ayudarle

Si el administrador de su empresa no puede ayudarle, cree una [solicitud de soporte técnico de Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). Proporcione la siguiente información:

- Número de inscripción
- Dirección de correo electrónico que se debe agregar y tipo de autenticación (cuenta profesional, educativa o de Microsoft)
- Aprobación por correo electrónico por parte de un administrador de empresa existente
  - Si el administrador de empresa existente no está disponible, póngase en contacto con su partner o asesor de software para solicitarle que cambie los detalles del contacto a través de la herramienta Centro de servicios de licencias por volumen (VLSC).

## <a name="create-an-azure-enterprise-department"></a>Creación de un departamento de Azure Enterprise

Los administradores de empresa y los administradores de departamento usan departamentos para organizar e informar no solo acerca de los servicios empresariales de Azure sino también de su uso por departamento y centro de costos. El administrador de empresa puede:

- Agregar o quitar departamentos.
- Asociar una cuenta a un departamento.
- Crear administradores de departamento.
- Permitir a los administradores de departamento ver el precio y los costos.

Un administrador de departamento puede agregar cuentas a sus departamentos. Puede quitar cuentas de sus departamentos, pero no de la inscripción.

Para agregar un departamento:

1. Inicie sesión en Azure Enterprise Portal.
1. Seleccione **Administrar** en el panel izquierdo.
1. Seleccione la pestaña **Departamento** y, a continuación, seleccione **+ Add Department** (+ Agregar departamento).
1. Escriba la información.
   El nombre del departamento es el único campo obligatorio. Debe tener tres caracteres como mínimo.
1. Cuando haya terminado, seleccione **Agregar**.

## <a name="add-a-department-administrator"></a>Adición de un administrador de departamento

Una vez creado un departamento, el administrador de empresa puede agregar administradores de departamento y asociar cada uno de ellos a un departamento. Los administradores de departamento pueden realizar las siguientes acciones para sus departamentos:

- Crear otros administradores de departamento
- Ver y editar propiedades de departamento, como el nombre o el centro de costos
- Adición de cuentas
- Quitar cuentas
- Descargar detalles de uso
- Ver el uso y los cargos mensuales <sup>1</sup>

> <sup>1</sup> Un administrador de empresa debe conceder estos permisos. Si tiene permiso para ver el uso y los cargos mensuales del departamento, pero no puede verlos, póngase en contacto con su partner.

### <a name="to-add-a-department-administrator"></a>Para agregar un administrador de departamento

Como administrador de empresa:

1. Inicie sesión en Azure Enterprise Portal.
1. Seleccione **Administrar** en el panel izquierdo.
1. Seleccione la pestaña **Departamento** y, a continuación, seleccione el departamento.
1. Haga clic en **+ Agregar administrador** y agregue la información necesaria.
1. Para que el acceso sea de solo lectura, establezca la opción **Solo lectura** en **Sí** y seleccione **Agregar**.

![Ejemplo que muestra el cuadro de diálogo Add Department Administrator (Agregar administrador de departamento)](./media/ea-portal-administration/ea-create-add-department-admin.png)

### <a name="to-set-read-only-access"></a>Para establecer acceso de solo lectura

Puede conceder acceso de solo lectura a los administradores de departamento.

- Cuando cree un administrador de departamento, establezca la opción de solo lectura en **Sí**.

- Para editar un administrador de departamento existente:
   1. Seleccione un departamento y, después, seleccione el símbolo del lápiz situado junto al **administrador de departamento** que desea editar.
   1. Establezca la opción de solo lectura en **Sí** y, a continuación, seleccione **Guardar**.

Los administradores de empresa obtienen automáticamente permisos de administrador de departamento.

## <a name="add-an-account"></a>Agregar una cuenta

La estructura de las cuentas y suscripciones afecta al modo en que se administran y aparecen en las facturas e informes. Entre los ejemplos de estructura típica de una organización se incluye las divisiones empresariales, los equipos funcionales y las ubicaciones geográficas.

Para agregar una cuenta:

1. En Azure Enterprise Portal, seleccione **Administrar** en el área de navegación de la izquierda.
1. Seleccione la pestaña **Cuenta**. En la página **Cuenta**, haga clic en **+Agregar cuenta**.
1. Seleccione un departamento o déjelo como sin asignar y, después, seleccione el tipo de autenticación que desea.
1. Escriba el nombre descriptivo para identificar la cuenta en los informes.
1. Escriba la dirección de **correo electrónico del propietario de la cuenta** que se va a asociar a la nueva cuenta.
1. Confirme la dirección de correo electrónico y seleccione **Agregar**.

![Ejemplo que muestra la lista de cuentas y la opción + Add Account (+ Agregar cuenta)](./media/ea-portal-administration/create-ea-add-an-account.png)

Para agregar otra cuenta, seleccione **Add Another Account** (Agregar otra cuenta), o bien puede seleccionar **Agregar** en la esquina inferior derecha de la barra de herramientas izquierda.

Para confirmar la propiedad de la cuenta:

1. Inicie sesión en Azure Enterprise Portal.
1. Vea el estado.

   El estado debe cambiar de **Pending** (Pendiente) a **Start/End date** (Fecha de inicio y de finalización). La fecha de inicio y de finalización es la fecha en que el usuario inició sesión por primera vez y la fecha de finalización del contrato.
1. Cuando aparezca el mensaje **Advertencia**, el propietario de la cuenta debe seleccionar **Continuar** para activar la cuenta la primera vez que inicie sesión en Azure Enterprise Portal.

## <a name="change-azure-subscription-or-account-ownership"></a>Cambio de la propiedad de la cuenta o de la suscripción de Azure

Los administradores de empresa pueden usar Azure Enterprise Portal para transferir la propiedad de la cuenta de todas las suscripciones de una inscripción, o solo de algunas de ellas.

Cuando se completa la transferencia de una suscripción o de la propiedad de una cuenta, Microsoft actualiza el propietario de la cuenta.

Antes de realizar la transferencia de la propiedad debe conocer estas directivas de control de acceso basado en rol de Azure (Azure RBAC):

- Cuando se realizan transferencias de suscripciones o de la propiedad de cuentas entre dos identificadores de organización que se encuentran en el mismo inquilino, se conservan tanto las directivas de Azure RBAC como los roles de administrador y coadministrador de servicios existentes.
- Las transferencias de suscripciones o de propiedad de cuentas entre inquilinos provocan la pérdida no solo de las directivas de Azure RBAC, sino también de las asignaciones de roles.
- Las directivas y los roles de administrador no se transfieren entre los distintos directorios. Los administradores de servicios se actualizan al propietario de la cuenta de destino.
- Para evitar la pérdida tanto de las directivas de Azure RBAC como de las asignaciones de roles al transferir la suscripción de un inquilino a otro, asegúrese de que la casilla **Move the subscriptions to the recipient’s Azure AD tenant** (Mover las suscripciones al inquilino de Azure AD del destinatario) no está **seleccionada**. De esta forma, se conservarán los servicios, los roles de Azure RBAC y las directivas del inquilino de Azure AD actual y solo se transferirán la propiedad de facturación de la cuenta.  
    :::image type="content" source="./media/ea-portal-administration/unselected-checkbox-move-subscriptions-to-recipients-tenant.png" alt-text="Imagen que muestra la casilla no seleccionada, con el fin de que se puedan mover las suscripciones al inquilino de Azure AD" lightbox="./media/ea-portal-administration/unselected-checkbox-move-subscriptions-to-recipients-tenant.png" :::


Antes de cambiar el propietario de una cuenta:

1. En Azure Enterprise Portal, consulte la pestaña **Cuenta** e identifique la cuenta de origen. La cuenta de origen debe estar activa.
1. Identifique la cuenta de destino y asegúrese de que esté activa.

Para transferir la propiedad de la cuenta para todas las suscripciones:

1. Inicie sesión en Azure Enterprise Portal.
1. En el área de navegación izquierda, seleccione **Administrar**.
1. Seleccione la pestaña **Cuenta** y mantenga el puntero sobre una cuenta.
1. Seleccione el icono de cambio de propietario de la cuenta a la derecha. El icono se parece a una persona.  
    ![Imagen que muestra el símbolo Cambiar propietario de cuenta.](./media/ea-portal-administration/create-ea-create-sub-transfer-account-ownership-of-sub.png)
1. Elija la cuenta de destino a la que desea realizar la transferencia y, después, seleccione **Siguiente**.
1. Si desea transferir la propiedad de la cuenta entre los inquilinos de Azure AD, seleccione la casilla **Move the subscriptions to the recipient's Azure AD tenant** (Mover las suscripciones al inquilino de Azure AD del destinatario).  
    :::image type="content" source="./media/ea-portal-administration/selected-checkbox-move-subscriptions-to-recipients-tenant.png" alt-text="Imagen que muestra la casilla seleccionada para mover suscripciones al inquilino de Azure AD" lightbox="./media/ea-portal-administration/selected-checkbox-move-subscriptions-to-recipients-tenant.png" :::
1. Confirme la transferencia y seleccione **Enviar**.

Para transferir la propiedad de la cuenta para una sola suscripción:

1. Inicie sesión en Azure Enterprise Portal.
1. En el área de navegación izquierda, seleccione **Administrar**.
1. Seleccione la pestaña **Cuenta** y mantenga el puntero sobre una cuenta.
1. Seleccione el icono de transferir suscripciones a la derecha. El icono se parece a una página.  
    ![Imagen que muestra el símbolo Transferir suscripciones](./media/ea-portal-administration/ea-transfer-subscriptions.png)
1. Elija la cuenta de destino a la que desea transferir la suscripción y, después, seleccione **Siguiente**.
1. Si desea transferir la propiedad de la suscripción entre los inquilinos de Azure AD, seleccione la casilla **Move the subscriptions to the recipient's Azure AD tenant** (Mover las suscripciones al inquilino de Azure AD del destinatario).  
    :::image type="content" source="./media/ea-portal-administration/selected-checkbox-move-subscriptions-to-recipients-tenant.png" alt-text="Imagen que muestra la casilla seleccionada, para que se puedan mover las suscripciones al inquilino de Azure AD" lightbox="./media/ea-portal-administration/selected-checkbox-move-subscriptions-to-recipients-tenant.png" :::
1. Confirme la transferencia y seleccione **Enviar**.


## <a name="associate-an-account-to-a-department"></a>Asociar una cuenta a un departamento

Los administradores de empresa pueden asociar cuentas existentes a los departamentos de la inscripción.

### <a name="to-associate-an-account-to-a-department"></a>Para asociar una cuenta a un departamento

1. Inicie sesión en Azure EA Portal como administrador de empresa.
1. Seleccione **Administrar** en el panel de navegación izquierdo.
1. Seleccione **Departamento**.
1. Mantenga el puntero sobre la fila en que se encuentra la cuenta y seleccione el icono de lápiz de la derecha.
1. Seleccione el departamento en el menú desplegable.
1. Seleccione **Guardar**.

## <a name="associate-an-existing-account-with-your-pay-as-you-go-subscription"></a>Asociación de una cuenta existente con la suscripción de pago por uso

Si ya tiene una cuenta de Microsoft Azure existente en Azure Portal, escriba la cuenta profesional, educativa o de Microsoft asociada con el fin de asociarla a la inscripción del Contrato Enterprise.

### <a name="associate-an-existing-account"></a>Asociación de una cuenta existente

1. En Azure Enterprise Portal, seleccione **Administrar**.
1. Seleccione la pestaña **Cuenta**.
1. Haga clic en **+Agregar cuenta**.
1. Escriba la cuenta profesional, educativa o de Microsoft asociada a la cuenta de Azure existente.
1. Confirme la cuenta asociada a la cuenta de Azure existente.
1. Proporcione el nombre que desea usar para identificar esta cuenta en los informes.
1. Seleccione **Agregar**.
1. Para agregar una cuenta adicional, puede seleccionar de nuevo la opción **+Agregar cuenta** o volver a la página principal; para ello, seleccione el botón **Administración**.
1. Si ve la página **Cuenta**, la cuenta recién agregada aparecerá con estado **Pendiente**.

### <a name="confirm-account-ownership"></a>Confirmación de la propiedad de la cuenta

1. Inicie sesión en la cuenta de correo electrónico asociada a la cuenta profesional, educativa o de Microsoft que ha proporcionado.
1. Abra la notificación de correo electrónico titulada _"Invitation to Activate your Account on the Microsoft Azure Service from Microsoft Volume Licensing"_ (Invitación para activar su cuenta en el servicio Microsoft Azure de licencias por volumen de Microsoft).
1. Seleccione el vínculo **Log into the Microsoft Azure Enterprise Portal** (Iniciar sesión en Microsoft Azure Enterprise Portal) de la invitación.
1. Seleccione **Iniciar sesión**.
1. Escriba la cuenta profesional, educativa o de Microsoft y la contraseña para iniciar sesión y confirmar la propiedad de la cuenta.

### <a name="azure-marketplace"></a>Azure Marketplace

Aunque la mayoría de las suscripciones pueden convertirse del entorno de pago por uso a Contrato Enterprise de Azure, los servicios de Azure Marketplace no. Para tener una vista única de todas las suscripciones y cargos, se recomienda agregar los servicios de Azure Marketplace a Azure Enterprise Portal.

1. Inicie sesión en Azure Enterprise Portal.
1. Seleccione **Administrar** en el panel de navegación izquierdo.
1. Seleccione la pestaña **Inscripción**.
1. Observe la sección **Enrollment Detail** (Detalles de la inscripción).
1. A la derecha del campo Azure Marketplace, seleccione el icono de lápiz para habilitarlo. Seleccione **Guardar**.

El propietario de la cuenta ahora puede comprar cualquier servicio de Azure Marketplace que haya adquirido anteriormente en la suscripción de pago por uso.

Después de que se activen las nuevas suscripciones de Azure Marketplace en su inscripción de Azure EA, cancele los servicios de Azure Marketplace que se crearon en el entorno de pago por uso. Este paso es fundamental para que las suscripciones de Azure Marketplace no entren en un estado incorrecto cuando expire el instrumento de pago por uso.

### <a name="msdn"></a>MSDN

Las suscripciones a MSDN se convierten automáticamente a Desarrollo/pruebas de MSDN, y la oferta de Azure EA perderá cualquier crédito monetario existente.

### <a name="azure-in-open"></a>Azure bajo licencia Open

Si asocia una suscripción de Azure bajo licencia Open con un Contrato Enterprise, perderá todos los créditos no consumidos de Azure bajo licencia Open. Por lo tanto, se recomienda usar todo el crédito en una suscripción de Azure bajo licencia Open antes de agregar la cuenta al Contrato Enterprise.  

### <a name="accounts-with-support-subscriptions"></a>Cuentas con suscripciones de soporte técnico

Si el Contrato Enterprise no tiene una suscripción de soporte técnico, y usted agrega una cuenta existente con una suscripción de soporte técnico a Azure Enterprise Portal, la suscripción de soporte técnico de MOSA no se transferirá automáticamente. Deberá volver a comprar una suscripción de soporte técnico en Azure EA durante el período de gracia, al final del mes siguiente.

## <a name="department-spending-quotas"></a>Cuota de gasto de departamento

Los clientes con Contrato Enterprise pueden establecer o cambiar las cuotas de gasto de cada departamento de una inscripción. El importe de la cuota de gasto se establece para el período de prepago actual. Al final de ese período, el sistema ampliará la cuota de gasto existente al siguiente período de prepago, a menos que se actualicen los valores.

El administrador del departamento puede ver la cuota de gasto, pero solo el administrador de la empresa puede actualizar el importe de la cuota. El administrador de la empresa y el administrador del departamento reciben notificaciones una vez que la cuota ha alcanzado el 50 %, 75 %, 90 % y 100 %.

### <a name="enterprise-administrator-to-set-the-quota"></a>Para que el administrador de la empresa establezca la cuota:

 1. Abra Azure EA Portal.
 1. Seleccione **Administrar** en el panel de navegación izquierdo.
 1. Seleccione la pestaña **Departamento**.
 1. Seleccione el departamento.
 1. Seleccione el icono del lápiz de la sección Department Details (Detalles del departamento) o seleccione el símbolo **+ Add Department** (Agregar departamento) para agregar una cuota de gasto junto con un nuevo departamento.
 1. En Department Details (Detalles del departamento), escriba el importe de la cuota de gasto en la moneda de la inscripción en el cuadro Spending Quota $ (Cuota de gasto) (debe ser mayor que 0).
    - El nombre del departamento y el centro de costos también se pueden editar en este momento.
 1. Seleccione **Guardar**.

La cuota de gasto del departamento ahora estará visible en la vista de lista de departamentos en la pestaña Department (Departamento). Al final del período de prepago actual, Azure EA Portal mantendrá las cuotas de gasto para el siguiente período de prepago.

El importe de la cuota de departamento es independiente del prepago actual de Azure y el importe de la cuota y las alertas solo se aplican al uso de la primera parte. La cuota de gasto del departamento solo tiene fines informativos, no supone una exigencia de los límites de gasto.

### <a name="department-administrator-to-view-the-quota"></a>Para que el administrador de departamento pueda ver la cuota:

1. Abra Azure EA Portal.
1. Seleccione **Administrar** en el panel de navegación izquierdo.
1. Seleccione la pestaña **Department** (Departamento) y accederá a la vista de lista de departamentos con sus cuotas de gasto.

Si es un cliente indirecto, el partner de canal debe habilitar las características de costos.

## <a name="enterprise-user-roles"></a>Roles de los usuarios de empresa

El portal del Contrato Enterprise de Azure le ayuda a administrar los costos y el uso del Contrato Enterprise de Azure. Hay tres roles principales en el portal del Contrato Enterprise de Azure:

- Administrador del Contrato Enterprise
- Administrador de departamentos
- Propietario de cuenta

Cada rol tiene un nivel de acceso y autoridad diferente.

Para más información sobre los roles de usuario, consulte [Roles de usuario de empresa](./understand-ea-roles.md#enterprise-user-roles).

## <a name="add-an-azure-ea-account"></a>Adición de una cuenta del Contrato Enterprise de Azure

La cuenta del Contrato Enterprise de Azure es una unidad organizativa del portal del Contrato Enterprise de Azure. Se utiliza para administrar las suscripciones y los informes. Para acceder a los servicios de Azure y usarlos, debe crear una cuenta o tener una ya creada.

Para más información sobre las cuentas de Azure, consulte [Agregar una cuenta](#add-an-account).

## <a name="enterprise-devtest-offer"></a>Oferta Desarrollo/pruebas - Enterprise

Como administrador de empresa de Azure, puede permitir a los propietarios de cuentas de su organización crear suscripciones basadas en la oferta Desarrollo/pruebas del Contrato Enterprise. Para ello, seleccione el cuadro Desarrollo/pruebas del propietario de la cuenta en Azure EA Portal.

Una vez que haya activado la casilla Dev/Test (Desarrollo/pruebas), comuníqueselo al propietario de la cuenta para que pueda configurar las suscripciones de Desarrollo/pruebas del Contrato Enterprise necesarias para sus equipos de suscriptores de desarrollo y pruebas.

La oferta permite a los suscriptores activos de Visual Studio ejecutar cargas de trabajo de desarrollo y pruebas en Azure a las tarifas especiales de desarrollo y pruebas. Proporciona acceso a toda la galería de imágenes de desarrollo y pruebas incluido Windows 8.1 y Windows 10.

### <a name="to-set-up-the-enterprise-devtest-offer"></a>Para configurar la oferta Desarrollo/pruebas - Enterprise:

1. Inicie sesión como administrador de empresa.
1. Seleccione **Administrar** en el panel de navegación izquierdo.
1. Seleccione la pestaña **Cuenta**.
1. Seleccione la fila de la cuenta en la que desea habilitar el acceso a Desarrollo/pruebas.
1. Seleccione el símbolo del lápiz situado a la derecha de la fila.
1. Seleccione la casilla Dev/Test (Desarrollo/pruebas).
1. Seleccione **Guardar**.

Cuando se agrega un usuario como propietario de la cuenta mediante Azure EA Portal, las suscripciones de Azure asociadas con el propietario de la cuenta que se basan en la oferta Desarrollo/pruebas de pago por uso, o en las ofertas de crédito mensuales para suscriptores de Visual Studio, se convertirán en la oferta Desarrollo/pruebas - Enterprise. Las suscripciones basadas en otros tipos de oferta, como Pago por uso, asociados al propietario de la cuenta se convertirán en ofertas de Microsoft Azure Enterprise.

Los clientes de Azure Gov no pueden disfrutar de la oferta Desarrollo/pruebas en este momento.

## <a name="create-a-subscription"></a>una suscripción

Los propietarios de cuentas pueden ver y administrar suscripciones. Las suscripciones se pueden usar para dar acceso a los equipos de la organización a proyectos y entornos de desarrollo. Por ejemplo: prueba, producción, desarrollo y almacenamiento provisional.

La creación de distintas suscripciones para cada entorno de aplicación ayuda a proteger cada entorno.

- También se puede asignar una cuenta de administrador de servicios diferente a cada suscripción.
- Se pueden asociar las suscripciones a cualquier número de servicios.
- El propietario de la cuenta crea suscripciones y asigna una cuenta de administrador de servicios a cada una de las suscripciones de su cuenta.

### <a name="add-a-subscription"></a>Agregar una suscripción

Use la siguiente información para agregar una suscripción.

La primera vez que agrega una suscripción a la cuenta, se le pide que acepte el Contrato Microsoft Online Subscription (MOSA) y un plan de tarifas. Aunque ni MOSA ni el plan de tarifas se aplican a los clientes con Contrato Enterprise, son necesarios para crear la suscripción. La inscripción del Contrato Enterprise de Microsoft Azure sustituye a los elementos anteriores y la relación contractual no cambia. Cuando se le solicite, seleccione la casilla que indica que acepta los términos.

_Microsoft Azure Enterprise_ es el nombre predeterminado cuando se crea una suscripción. Puede cambiar el nombre para diferenciarla de las demás suscripciones de la inscripción y para asegurarse de reconocerla en los informes de empresa.

Para agregar una suscripción:

1. En Azure Enterprise Portal, inicie sesión con la cuenta.
1. Seleccione la pestaña **Administración** y, después, seleccione **Suscripción** en la parte superior de la página.
1. Compruebe que ha iniciado sesión como propietario de la cuenta.
1. Seleccione **+Agregar suscripción** y luego seleccione **Compra**.

   La primera vez que se agrega una suscripción a una cuenta, es preciso especificar la información de contacto. Cuando agrega suscripciones adicionales, su información de contacto se agrega automáticamente.

1. Seleccione **Suscripciones** y, después, seleccione la suscripción que ha creado.
1. Seleccione **Editar detalles de suscripción**.
1. Edite **Nombre de la suscripción** y **Administrador de servicios** y, después, seleccione la marca de verificación.

   El nombre de la suscripción aparece en los informes. Se trata del nombre del proyecto asociado a la suscripción en el portal de desarrollo.

Las nuevas suscripciones pueden tardar hasta 24 horas en aparecer en la lista de suscripciones. Después de crear una suscripción, puede:

- [Editar los detalles de la suscripción](https://portal.azure.com)
- [Administrar los servicios de la suscripción](https://portal.azure.com/#home)

## <a name="delete-subscription"></a>Eliminar suscripción

Para eliminar una suscripción en la que es el propietario de la cuenta:

1. Inicie sesión en Azure Portal con las credenciales asociadas a su cuenta.
1. En el menú de concentrador, seleccione **Suscripción**.
1. En la pestaña de suscripciones de la esquina superior izquierda de la página, seleccione la suscripción que desea cancelar y, después, **Cancel Sub** (Cancelar suscripción) para iniciar la pestaña de cancelación.
1. Escriba el nombre de la suscripción, elija el motivo de la cancelación y seleccione el botón **Cancel Sub** (Cancelar suscripción).

Solo los administradores de la cuenta pueden cancelar las suscripciones.

Para más información, consulte [¿Qué ocurre después de cancelar la suscripción?](cancel-azure-subscription.md#what-happens-after-subscription-cancellation).

## <a name="delete-an-account"></a>Eliminación de una cuenta

Solo se puede completar la eliminación de cuentas de las cuentas activas sin suscripciones activas.

1. En Enterprise Portal, seleccione **Manage** (Administrar) en el panel de navegación izquierdo.
1. Seleccione la pestaña **Cuenta**.
1. En la tabla de cuentas, seleccione la cuenta que desea eliminar.
1. Seleccione el símbolo X de la derecha de la fila de la cuenta.
1. Una vez que no haya suscripciones activas en la cuenta, seleccione **Yes** (Sí) en la fila de la cuenta para confirmar la eliminación de la cuenta.

## <a name="update-notification-settings"></a>Actualización de la configuración de notificación

Los administradores de empresa se inscriben automáticamente para recibir notificaciones de uso asociadas a la inscripción. Cada administrador de empresa puede cambiar el intervalo de las notificaciones individuales o puede desactivarlas completamente.

Los contactos de notificación se muestran en el portal del Contrato Enterprise de Azure en la sección **Notification Contact** (Contacto para notificaciones). La administración de los contactos de notificaciones garantiza que las personas adecuadas de la organización obtienen notificaciones del Contrato Enterprise de Azure.

Para ver la configuración actual de notificaciones:

1. En el portal del Contrato Enterprise de Azure, vaya a **Manage** > **Notification Contact** (Administrar > Contacto para notificaciones).
2. Dirección de correo electrónico: la dirección de correo electrónico asociada a la cuenta de Microsoft del administrador de empresa, la cuenta profesional o educativa que recibe las notificaciones.
3. Unbilled Balance Notification Frequency (Frecuencia de notificación de saldo sin facturar): frecuencia con la que las notificaciones están configuradas para enviarse a cada administrador de empresa individual.

Para agregar un contacto:

1. Seleccione **+Add Contact** (Agregar contacto).
2. Especifique la dirección de correo electrónico y confírmela.
3. Seleccione **Guardar**.

El nuevo contacto de notificación se muestra en la sección **Notification Contact** (Contacto para notificaciones). Para cambiar la frecuencia de notificación, seleccione el contacto de la notificación y, después, el símbolo del lápiz situado a la derecha de la fila seleccionada. Establezca la frecuencia en **diaria**, **semanal**, **mensual** o **ninguna**.

Puede suprimir _que se aproxima a la fecha de finalización del período de cobertura_ y _que se aproxima a la fecha de deshabilitación y desaprovisionamiento_ de las notificaciones del ciclo de vida. Al deshabilitar las notificaciones del ciclo de vida, se suprimen las notificaciones sobre el período de cobertura y la fecha de finalización del contrato.

## <a name="azure-sponsorship-offer"></a>Oferta Patrocinio de Azure

La oferta Patrocinio de Azure es una cuenta de Microsoft Azure patrocinada limitada. Está disponible mediante invitación por correo electrónico solo para un número limitado de clientes seleccionados por Microsoft. Si es apto para la oferta Patrocinio de Microsoft Azure, recibirá una invitación por correo electrónico a su identificador de cuenta.

Para más información, cree una [solicitud de soporte técnico para la activación del patrocinio](https://aka.ms/azrsponsorship).

## <a name="conversion-to-work-or-school-account-authentication"></a>Conversión a la autenticación de la cuenta profesional o educativa

Los usuarios de Azure Enterprise pueden convertir desde una cuenta de Microsoft (MSA o Live ID) al tipo de autenticación de una cuenta profesional o educativa (que usa Azure Active Directory).

Para empezar:

1. Agregue la cuenta profesional o educativa a Azure EA Portal en los roles necesarios.
1. Si recibe errores, es posible que la cuenta no sea válida en Active Directory.  Azure usa el nombre principal de usuario (UPN), que no es siempre idéntico a la dirección de correo electrónico.
1. Inicie sesión en Azure EA Portal con la cuenta profesional o educativa.

### <a name="to-convert-subscriptions-from-microsoft-accounts-to-work-or-school-accounts"></a>Para convertir las suscripciones de las cuentas de Microsoft en cuentas profesionales o educativas:

1. Inicie sesión en el portal de administración con la cuenta de Microsoft que es propietaria de las suscripciones.
1. Use la transferencia de propiedad de la cuenta para pasar a la nueva cuenta.
1. Ahora la cuenta de Microsoft debe estar libre de suscripciones activas y se puede eliminar.
1. Cualquier cuenta eliminada permanecerá en la vista en el portal en un estado inactivo por razones del historial de facturación.  Puede filtrarla de la vista activando una casilla de verificación para mostrar solo las cuentas activas.

## <a name="azure-ea-term-glossary"></a>Glosario de términos de Azure EA

- **Cuenta**: Unidad organizativa en Azure Enterprise Portal. Se utiliza para administrar las suscripciones y para los informes.
- **Propietario de la cuenta**: Persona que administra las suscripciones y los administradores de servicios en Azure. Pueden ver los datos de uso de esta cuenta y sus suscripciones asociadas.
- **Suscripción de enmienda**: Suscripción de un año o coincidente de la enmienda de la inscripción.
- **Pago por adelantado**: pago por adelantado de una cantidad monetaria anual por los servicios de Azure con una tasa de prepago con descuento por el uso de este pago por adelantado.
- **Administrador de departamento**: Persona que administra departamentos, crea cuentas nuevas y propietarios de cuentas, ve los detalles de uso de los departamentos que administra y ve los costos cuando se le conceden permisos.
- **Número de inscripción**: Identificador único proporcionado por Microsoft para identificar la inscripción específica asociada a un Contrato Enterprise.
- **Administrador de empresa**: Persona que administra departamentos, propietarios de departamentos, cuentas y propietarios de cuentas en Azure. Tienen la capacidad de administrar los administradores de empresa, así como ver los datos de uso, las cantidades facturadas y los cargos no facturados en todas las cuentas y suscripciones asociadas a la inscripción de la empresa.
- **Contrato Enterprise**: un contrato de licencias de Microsoft para clientes con adquisición centralizada que desean estandarizar toda la organización en la tecnología de Microsoft y mantener una infraestructura de tecnología de la información en un estándar de software de Microsoft.
- **Inscripción de Contrato Enterprise**: Inscripción en el programa de Contrato Enterprise que proporciona productos de Microsoft en volumen a tarifas con descuento.
- **Cuenta de Microsoft**: Servicio basado en web que permite a los sitios participantes autenticar a un usuario con un único conjunto de credenciales.
- **Enmienda de inscripción de Microsoft Azure Enterprise (enmienda de inscripción)** : Enmienda firmada por una empresa, que proporciona acceso a Azure como parte de la inscripción empresarial.
- **Azure Enterprise Portal**: Portal que usan los clientes empresariales para administrar sus cuentas de Azure y las suscripciones relacionadas.
- **Cantidad de recursos consumidos**: Cantidad de un servicio individual de Azure que se usó en un mes.
- **Administrador del servicio**: Persona que accede y administra suscripciones y proyectos de desarrollo en Azure Enterprise Portal.
- **Suscripción**: Representa una suscripción de Azure Enterprise Portal y es un contenedor de servicios de Azure administrados por el mismo administrador de servicios.
- **Cuenta profesional o educativa**: Para organizaciones que han configurado Azure Active Directory con Federación en la nube y todas las cuentas están en un solo inquilino.

### <a name="enrollment-statuses"></a>Estados de inscripción

- **Nuevo**: Este estado se asigna a una inscripción que se creó hace menos de 24 horas y que se actualizará a un estado pendiente en un plazo máximo de 24 horas.
- **Pending**: El administrador de la inscripción debe iniciar sesión en Azure Enterprise Portal. Una vez que inicia sesión, la inscripción cambiará a estado Activa.
- **Activa**: La inscripción está activa, y se pueden crear cuentas y suscripciones en Azure Enterprise Portal. La inscripción permanecerá activa hasta la fecha de finalización del Contrato Enterprise.
- **Período extendido indefinido**: Un período extendido indefinido tiene lugar después de que se haya alcanzado la fecha de finalización del Contrato Enterprise. Permite a los clientes de Azure EA que participan en el período extendido seguir usando los servicios de Azure indefinidamente al final de su Contrato Enterprise.

   Antes de que la inscripción de Azure EA alcance la fecha de finalización del Contrato Enterprise, el administrador de inscripciones debe decidir entre las opciones siguientes:

  - Renovar la inscripción mediante la adición de un pago por adelantado adicional.
  - Transferirla a una nueva inscripción.
  - Migrar al Programa de suscripción en línea de Microsoft (MOSP).
  - Confirmar la deshabilitación de todos los servicios asociados con la inscripción.
- **Expirada**: El cliente de Azure EA no participa en el período extendido, y la inscripción de Azure EA ha alcanzado la fecha de finalización del Contrato Enterprise. La inscripción expirará, y se deshabilitarán todos los servicios asociados.
- **Transferida**: Las inscripciones en las que se hayan transferido a una nueva inscripción todas las cuentas y servicios asociados aparecen con estado Transferida.
  >[!NOTE]
  > Las inscripciones no se transfieren automáticamente si se genera un nuevo número de inscripción en el momento de la renovación. Debe incluir el número de inscripción anterior en la solicitud de renovación para facilitar la transferencia automática.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información acerca de cómo las [reservas de máquinas virtuales](ea-portal-vm-reservations.md) pueden ayudarle a ahorrar dinero.
- Si necesita ayuda para solucionar problemas con el portal del Contrato Enterprise de Azure, consulte [Solución de problemas de acceso al portal del Contrato Enterprise de Azure](ea-portal-troubleshoot.md).
- Lea las [preguntas frecuentes de Cost Management + Billing](../cost-management-billing-faq.yml) para más información sobre la propiedad de la suscripción de EA.