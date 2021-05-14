---
title: Transferencia de la propiedad de facturación de la suscripción de Azure en un Contrato de cliente de Microsoft
description: Aprenda a transferir la propiedad de facturación de las suscripciones de Azure.
author: amberbhargava
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 04/29/2021
ms.author: banders
ms.openlocfilehash: c3294c09012339cb19f04327f1933688b05ae077
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108321472"
---
# <a name="transfer-azure-subscription-billing-ownership-for-a-microsoft-customer-agreement"></a>Transferencia de la propiedad de facturación de la suscripción de Azure en un Contrato de cliente de Microsoft

Deberá transferir la propiedad de facturación de las suscripciones de Azure cuando:
- Quiera trasladar las responsabilidades de facturación de las suscripciones a un propietario de facturación diferente.
- Va a transferir las suscripciones de Azure de un contrato de licencia a otro. Por ejemplo, de un Contrato Enterprise o un Contrato Microsoft Online Subscription (MOSA) a un Contrato de cliente de Microsoft. [Compruebe si tiene acceso a un contrato de cliente de Microsoft](#check-for-access).

La persona a la que solicite la propiedad de facturación debe tener uno de los roles siguientes:
- Para un Contrato de cliente de Microsoft, la persona debe tener un rol de propietario o colaborador para la cuenta de facturación o para el perfil de facturación o la sección de factura correspondiente. Para más información, consulte [Tareas y roles de facturación](understand-mca-roles.md#invoice-section-roles-and-tasks).
- En el caso de un Contrato Enterprise, el usuario debe tener un rol Propietario de la cuenta.
- En el caso de un Contrato Microsoft Online Subscription, el usuario debe ser un administrador de cuenta.

## <a name="request-billing-ownership"></a>Solicitud de propiedad de la facturación

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como propietario o colaborador de la sección de factura de una cuenta de facturación para un contrato de cliente de Microsoft.
1. Busque **Administración de costos + facturación**.  
    ![Captura de pantalla que muestra la búsqueda en Azure Portal de Administración de costos + facturación](./media/mca-request-billing-ownership/billing-search-cost-management-billing.png)
1. En la página de ámbitos de facturación, seleccione **Ámbitos de facturación** y, después, la cuenta de facturación que se utilizará para pagar el uso de las suscripciones. La cuenta de facturación será del tipo **Contrato de cliente de Microsoft**.  
    [![Captura de pantalla que muestra la búsqueda en Azure Portal de Administración de costos + facturación](./media/mca-request-billing-ownership/list-of-scopes.png)](./media/mca-request-billing-ownership/list-of-scopes.png#lightbox)
    > [!NOTE]
    > Azure Portal recuerda el último ámbito de facturación al que tiene acceso y muestra el ámbito la próxima vez que llega a la página Administración de costos + facturación. No verá la página de ámbitos de facturación si ha visitado Administración de costos y facturación anteriormente. Si es así, compruebe que se encuentra en el [ámbito correcto](#check-for-access). En caso contrario, [cambie el ámbito](view-all-accounts.md#switch-billing-scope-in-the-azure-portal) para seleccionar la cuenta de facturación de un Contrato de cliente de Microsoft.
1. Seleccione **Perfiles de facturación** en el lado izquierdo.  
    [![Captura de pantalla que muestra la selección de perfiles de facturación](./media/mca-request-billing-ownership/mca-select-profiles.png)](./media/mca-request-billing-ownership/mca-select-profiles.png#lightbox)
    > [!Note]
    > Si no ve los perfiles de facturación, no se encuentra en el ámbito de facturación adecuado. Tiene que seleccionar una cuenta de facturación de un Contrato de cliente de Microsoft y después seleccione los perfiles de facturación. Para obtener información sobre cómo cambiar los ámbitos, consulte [Cambio de ámbitos de facturación en Azure Portal](view-all-accounts.md#switch-billing-scope-in-the-azure-portal).
1. Seleccione un **perfil de facturación** en la lista. Cuando se asume la propiedad de las suscripciones, se facturará por su uso en este perfil de facturación.
1. Seleccione **Secciones de factura** en el lado izquierdo.  
    [![Captura de pantalla que muestra la selección de las secciones de la factura](./media/mca-request-billing-ownership/mca-select-invoice-sections.png)](./media/mca-request-billing-ownership/mca-select-invoice-sections.png#lightbox)   
1. Seleccione una sección de la factura en la lista. Cuando se asume la propiedad de las suscripciones, se asignará su uso a esta sección de la factura del perfil de facturación.
1. Seleccione **Solicitudes de transferencia** en el lado inferior izquierdo y, luego, **Agregar una nueva solicitud**.  
    [![Captura de pantalla que muestra la selección de solicitudes de transferencia](./media/mca-request-billing-ownership/mca-select-transfer-requests.png)](./media/mca-request-billing-ownership/mca-select-transfer-requests.png#lightbox)
1. Escriba la dirección de correo electrónico del usuario al que solicita la propiedad de la facturación. Seleccione **Enviar solicitud de transferencia**.  
    [![Captura de pantalla que muestra el envío de una solicitud de transferencia](./media/mca-request-billing-ownership/mca-send-transfer-requests.png)](./media/mca-request-billing-ownership/mca-send-transfer-requests.png#lightbox)
1. El usuario recibe un correo electrónico con instrucciones para revisar la solicitud de transferencia.  
    ![Captura de pantalla que muestra un correo electrónico de solicitud de transferencia](./media/mca-request-billing-ownership/mca-review-transfer-request-email.png)
1. Para aprobar la solicitud de transferencia, el usuario selecciona el vínculo en el correo electrónico y sigue las instrucciones.
    [![Captura de pantalla que muestra la revisión de la solicitud de transferencia](./media/mca-request-billing-ownership/review-transfer-requests.png)](./media/mca-request-billing-ownership/review-transfer-requests.png#lightbox) El usuario puede seleccionar la cuenta de facturación desde la que desea transferir productos de Azure. Una vez seleccionada, se muestran los productos que se pueden transferir. **Nota:** las suscripciones deshabilitadas no se pueden transferir y se mostrarán en la lista de productos de Azure que no se pueden transferir, si procede. Una vez seleccionados los productos de Azure que se van a transferir, seleccione **Validar**.
1. El área **Transfer Validation Result** (Resultado de validación de transferencia) mostrará el impacto de los productos de Azure que se van a transferir. Estos son los posibles estados:
    * **Superado**: se ha superado la validación de este producto de Azure y se puede transferir.
    * **Advertencia**: hay una advertencia para el producto de Azure seleccionado. Aunque el producto se puede transferir, el usuario debe tener en cuenta que esto tendrá consecuencias, por si desea realizar acciones encaminadas a reducirlas. Por ejemplo, la suscripción de Azure que se va a transferir se beneficia de una instancia reservada de máquina virtual. Después de la transferencia, dicha suscripción dejará de recibir esa ventaja. Para maximizar el ahorro, asegúrese de que la instancia reservada de máquina virtual esté asociada a otra suscripción que puede usar sus ventajas. En su lugar, el usuario también puede elegir volver a la página de selección y anular la selección de esta suscripción de Azure.
    * **Error**: el producto de Azure seleccionado no se puede transferir debido a un error. El usuario tendrá que volver a la página de selección y anular la selección de este producto para transferir los restantes productos de Azure seleccionados.  
    ![Captura de pantalla que muestra la experiencia de validación](./media/mca-request-billing-ownership/validate-transfer-request.png)

## <a name="check-the-transfer-request-status"></a>Comprobación del estado de una solicitud de transferencia

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque **Administración de costos + facturación**.  
    ![Captura de pantalla que muestra la búsqueda en Azure Portal de Administración de costos + facturación](./media/mca-request-billing-ownership/billing-search-cost-management-billing.png)
1. En la página de ámbitos de facturación, seleccione la cuenta de facturación a la que se envió la solicitud de transferencia.
1. Seleccione **Perfiles de facturación** en el lado izquierdo.  
    [![Captura de pantalla que muestra la selección de perfiles de facturación](./media/mca-request-billing-ownership/mca-select-profiles.png)](./media/mca-request-billing-ownership/mca-select-profiles.png#lightbox)
1. Seleccione el **perfil de facturación** para el que se envió la solicitud de transferencia.
1. Seleccione **Secciones de factura** en el lado izquierdo.  
    [![Captura de pantalla que muestra la selección de las secciones de la factura](./media/mca-request-billing-ownership/mca-select-invoice-sections.png)](./media/mca-request-billing-ownership/mca-select-invoice-sections.png#lightbox)   
1. Seleccione en la lista la sección de la factura para la que envió la solicitud de transferencia.
1. Seleccione **Solicitudes de transferencia** en el lado inferior izquierdo. En la página Solicitudes de transferencia se muestra la siguiente información:  
    [![Captura de pantalla que muestra una lista de solicitudes de transferencia](./media/mca-request-billing-ownership/mca-select-transfer-requests-for-status.png)](./media/mca-request-billing-ownership/mca-select-transfer-requests-for-status.png#lightbox)

   |Columna|Definición|
   |---------|---------|
   |Fecha de solicitud|La fecha en la que se envió la solicitud de transferencia.|
   |Recipient|La dirección de correo electrónico del usuario que envió la solicitud para transferir la propiedad de la facturación|
   |Fecha de expiración|La fecha en que expira la solicitud|
   |Status|El estado de la solicitud de transferencia|

    La solicitud de transferencia puede tener uno de los siguientes estados:

   |Status|Definición|
   |---------|---------|
   |En curso|El usuario no ha aceptado la solicitud de transferencia.|
   |Processing|El usuario aprueba la solicitud de transferencia. La facturación de las suscripciones seleccionadas por el usuario está en proceso de transferencia a la sección de factura|
   |Completed| La facturación de las suscripciones seleccionadas por el usuario se transfiere a la sección de factura.|
   |Finalizado con errores|La solicitud se ha completado pero la facturación de algunas suscripciones que el usuario ha seleccionado no se pudo transferir.|
   |Expirada|El usuario no aceptó la solicitud a tiempo y ha expirado.|
   |Canceled|Un usuario con acceso a la solicitud de transferencia ha cancelado la solicitud.|
   |Rechazado|El usuario ha rechazado la solicitud de transferencia.|

1. Seleccione una solicitud de transferencia para ver los detalles. En la página de detalles de la transferencia se muestra la siguiente información:  
    [![Captura de pantalla que muestra la lista de las suscripciones transferidas](./media/mca-request-billing-ownership/mca-transfer-completed.png)](./media/mca-request-billing-ownership/mca-transfer-completed.png#lightbox)

   |Columna  |Definición|
   |---------|---------|
   |Identificador de la solicitud de transferencia|El identificador único de la solicitud de transferencia. Si envía una solicitud de soporte técnico, comparta el identificador con el Soporte técnico de Azure para agilizar la solicitud.|
   |Transferencia solicitada el|La fecha en la que se envió la solicitud de transferencia.|
   |Transferencia solicitada por|La dirección de correo electrónico del usuario que envió la solicitud de transferencia.|
   |La solicitud de transferencia caduca el| La fecha en la que expira la solicitud de transferencia.|
   |Dirección de correo electrónico del destinatario|La dirección de correo electrónico del usuario que envió la solicitud para transferir la propiedad de la facturación|
   |Vínculo de transferencia enviado al destinatario|La dirección URL que se envió al usuario para revisar la solicitud de transferencia.|

## <a name="supported-subscription-types"></a>Tipos de suscripciones admitidos

Puede solicitar la propiedad de facturación de los tipos de suscripción que se describen a continuación.

- [Action Pack](https://azure.microsoft.com/offers/ms-azr-0025p/)\*
- [Azure bajo licencia Open](https://azure.microsoft.com/offers/ms-azr-0111p/)\*
- [Patrocinio de Pase para Azure](https://azure.microsoft.com/offers/azure-pass/)\*
- [Desarrollo/pruebas - Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p/)
- [Evaluación gratuita](https://azure.microsoft.com/offers/ms-azr-0044p/)\*
- [Pay-As-You-Go](https://azure.microsoft.com/offers/ms-azr-0003p/)
- [Desarrollo/pruebas - Pago por uso](https://azure.microsoft.com/offers/ms-azr-0023p/)
- [Plan de Microsoft Azure](https://azure.microsoft.com/offers/ms-azr-0017g/)\*\*
- [Oferta de patrocinio de Microsoft Azure](https://azure.microsoft.com/offers/ms-azr-0036p/)\*
- [Contrato Enterprise (EA) de Microsoft](https://azure.microsoft.com/pricing/enterprise-agreement/)
- [Contrato de cliente de Microsoft](https://azure.microsoft.com/pricing/purchase-options/microsoft-customer-agreement/)
- [Microsoft Partner Network](https://azure.microsoft.com/offers/ms-azr-0025p/)\*
- [Plataformas de MSDN](https://azure.microsoft.com/offers/ms-azr-0062p/)\*
- [Suscriptores de Visual Studio Enterprise (BizSpark)](https://azure.microsoft.com/offers/ms-azr-0064p/)\*
- [Suscriptores de Visual Studio Enterprise (MPN)](https://azure.microsoft.com/offers/ms-azr-0029p/)\*
- [Suscriptores de Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/)\*
- [Visual Studio Professional](https://azure.microsoft.com/offers/ms-azr-0059p/)\*
- [Suscriptores de Visual Studio Test Professional](https://azure.microsoft.com/offers/ms-azr-0060p/)\*

\* Ninguno de los créditos disponibles en la suscripción estará disponible en la cuenta nueva después de la transferencia.

\*\* Solo se admite para las suscripciones en cuentas que se crean durante el inicio de sesión en el sitio web de Azure.

## <a name="additional-information"></a>Información adicional

En las siguientes secciones se proporciona información adicional acerca de la transferencia de suscripciones.

### <a name="cancel-a-prior-support-plan"></a>Cancelación de un plan de soporte técnico anterior

Si tiene un plan de soporte técnico de Azure y transfiere todas las suscripciones de Azure a un nuevo contrato, debe cancelar el plan de soporte técnico porque no se transfiere con las suscripciones. Por ejemplo, al transferir un Contrato Microsoft Online Subscription (una suscripción de Azure comprada en la web) al Contrato de cliente de Microsoft. Para cancelar el plan de soporte técnico:

Debe usar sus credenciales de administrador de cuenta de la cuenta anterior si las credenciales difieren de las usadas para acceder a la nueva cuenta del Contrato de cliente de Microsoft.

1.  Inicie sesión en Azure Portal en https://portal.azure.com.
1.  Vaya a **Administración de costos + facturación**.
1.  Seleccione **Ámbitos de facturación** en el panel izquierdo.
1.  Seleccione la cuenta de facturación asociada al plan de soporte técnico de Microsoft.
    - Para el Contrato de cliente de Microsoft:
        - Seleccione **Cargos periódicos** en el panel izquierdo.
        - En el panel derecho, a la derecha del elemento de la línea del plan de soporte técnico, seleccione los puntos suspensivos ( **...** ) y, después, seleccione **Desactivar renovación automática**.
    - Para un Contrato Microsoft Online Subscription:
        - Seleccione **Suscripciones** en el panel izquierdo.
        - Seleccione la suscripción del plan de soporte técnico en el panel derecho y, a continuación, seleccione **Cancelar**.

### <a name="access-your-historical-invoices"></a>Acceso a las facturas históricas

Es posible que quiera acceder a las facturas de su cuenta antigua del Contrato Microsoft Online Subscription (una suscripción de Azure comprada en la web) después de transferir la propiedad de facturación a la nueva cuenta del Contrato de cliente de Microsoft. Para ello, siga estos pasos:

Debe usar sus credenciales de administrador de cuenta de la cuenta anterior si las credenciales difieren de las usadas para acceder a la nueva cuenta del Contrato de cliente de Microsoft.

1.  Inicie sesión en Azure Portal en https://azure.portal.com.
1.  Vaya a **Administración de costos + facturación**.
1.  Seleccione **Ámbitos de facturación** en el panel izquierdo.
1.  Seleccione la cuenta de facturación asociada a la cuenta del Contrato Microsoft Online Subscription.
1.  Seleccione **Facturas** en el panel izquierdo para acceder a las facturas históricas.

### <a name="no-service-downtime"></a>No hay ningún tiempo de inactividad del servicio

Los servicios de Azure de la suscripción se siguen ejecutando sin interrupción. Solo cambiamos la relación de facturación de las suscripciones de Azure que el usuario selecciona para transferir.

### <a name="disabled-subscriptions"></a>Suscripciones deshabilitadas

Las suscripciones deshabilitadas no se pueden transferir. Las suscripciones deben estar en estado activo para transferir su propiedad de la facturación.

### <a name="azure-resources-transfer"></a>Transferencia de recursos de Azure

Todos los recursos de las suscripciones, como máquinas virtuales, discos y sitios web se transfieren.

### <a name="azure-marketplace-products-transfer"></a>Transferencia de productos de Azure Marketplace

Los productos de Azure Marketplace se transfieren junto con sus suscripciones correspondientes.

### <a name="azure-reservations-transfer"></a>Transferencia de reservas de Azure

Si se transfieren suscripciones a Contrato Enterprise (EA) o contratos de cliente de Microsoft, Azure Reservations no se mueve automáticamente con ellas. [Póngase en contacto con soporte técnico de Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para mover las reservas.

### <a name="access-to-azure-services"></a>Acceso a servicios de Azure

El acceso de los usuarios, grupos o entidades de servicio existentes que se asignó mediante el [control de acceso basado en rol (Azure RBAC)](../../role-based-access-control/overview.md) no se ve afectado durante la transición.

### <a name="charges-for-transferred-subscription"></a>Cargos por la suscripción transferida

El propietario de la facturación original de las suscripciones es responsable de cualquier cargo que se haya informado hasta el momento en que se complete la transferencia. La sección de factura es responsable de los cargos informados desde el momento de la transferencia en adelante. Puede haber algunos cargos que se realizaran antes de la transferencia, pero que se notificaron con posterioridad. Estos cargos se muestran en la sección de factura.

### <a name="cancel-a-transfer-request"></a>Cancelación de una solicitud de transferencia

Puede cancelar la solicitud de transferencia hasta que se apruebe o rechace la solicitud. Para cancelar la solicitud de transferencia, vaya a la [página de detalles de la transferencia](#check-the-transfer-request-status) y seleccione Cancelar en la parte inferior de la página.

### <a name="software-as-a-service-saas-transfer"></a>Transferencia del Software como servicio (SaaS)

No se transfieren los productos de SaaS con las suscripciones. Pida al usuario que [se ponga en contacto con el soporte técnico de Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para transferir la propiedad de la facturación de los productos de SaaS. Junto con la propiedad de la facturación, el usuario también puede transferir la propiedad de los recursos. La propiedad de los recursos permite realizar operaciones de administración como eliminar y consultar los detalles del producto. El usuario debe ser propietario de un recurso en el producto de SaaS para poder transferir su propiedad.

## <a name="check-for-access"></a>Comprobación del acceso
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-support"></a>¿Necesita ayuda? Ponerse en contacto con soporte técnico

Si necesita ayuda, [póngase en contacto con soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para resolver el problema rápidamente.

## <a name="next-steps"></a>Pasos siguientes

- La propiedad de la facturación de las suscripciones a Azure se transfiere a la sección de factura. Realice un seguimiento de los cargos de estas suscripciones en [Azure Portal](https://portal.azure.com).
- Asigne permisos para consultar y administrar la facturación de estas suscripciones a otros usuarios. Para más información, consulte [Tareas y roles de la sección de factura](understand-mca-roles.md#invoice-section-roles-and-tasks).
