---
title: Visualización y descarga de la factura de Azure
description: Aprenda a consultar y descargar su factura de Azure. También puede descargar la factura desde Azure Portal o recibirla por correo electrónico.
keywords: factura, facturación, descarga de factura, factura de Azure, uso de Azure
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 37ce1a292b6ff2efe0abecdb2ab934f096689f87
ms.sourcegitcommit: 1f1d29378424057338b246af1975643c2875e64d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/05/2021
ms.locfileid: "105560804"
---
# <a name="view-and-download-your-microsoft-azure-invoice"></a>Visualización y descarga de la factura de Microsoft Azure

También puede descargar la factura en [Azure Portal](https://portal.azure.com/) o recibirla por correo electrónico. Si es cliente de Azure con un Contrato Enterprise (cliente de EA), no puede descargar la factura de su organización. En su lugar, las facturas se envían a la persona establecida para recibir las facturas de la inscripción.

## <a name="when-invoices-are-generated"></a>Cuándo se generan las facturas

Una factura se genera en función del tipo de cuenta de facturación. Las facturas se crean para las cuentas de facturación de Microsoft Online Service Program (MOSP), Contrato de cliente de Microsoft (MCA) y Microsoft Partner Agreement (MPA). También se generan facturas para cuentas de facturación de Contrato Enterprise (EA). Sin embargo, las facturas para cuentas de facturación de EA no se muestran en Azure Portal.

Para más información sobre las cuentas de facturación y la identificación del tipo de cuenta de facturación, consulte [Visualización de las cuentas de facturación en Azure Portal](../manage/view-all-accounts.md).

### <a name="invoice-status"></a>Estado de la factura

Al revisar el estado de la factura en Azure Portal, cada una tendrá uno de los siguientes símbolos.

|  Símbolo de estado | Description  |
|---|---|
| ![Símbolo de estado de vencimiento](./media/download-azure-invoice/due.svg) | *Vencimiento* se muestra cuando se genera una factura, pero aún no se ha pagado. |
| ![Símbolo de estado de vencido](./media/download-azure-invoice/past-due.svg)  | *Vencido* se muestra cuando Azure ha intentado realizar el cargo al medio de pago, pero este se ha rechazado. |
| ![Símbolo de estado pagado](./media/download-azure-invoice/paid.svg)  | El estado *Pagado* se muestra cuando Azure ha realizado el cargo correctamente al medio de pago. |

Cuando se crea una factura, aparece en Azure Portal con el estado *Vencimiento*. El estado Vencimiento es normal y esperado.  

Cuando no se ha pagado una factura, el estado se muestra como *Vencido*. Las suscripciones vencidas se deshabilitan si no se paga la factura.

## <a name="invoices-for-mosp-billing-accounts"></a>Facturas de cuentas de facturación de MOSP

Al registrarse en Azure mediante el sitio web de Azure, se crea una cuenta de facturación de MOSP. Por ejemplo, cuando se registra para obtener una [cuenta gratuita de Azure](https://azure.microsoft.com/offers/ms-azr-0044p/), como una [cuenta con tarifas de pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/) o como un [suscriptor de Visual Studio](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/).

Los clientes de algunas regiones seleccionadas, que se registran en el sitio web de Azure para una [cuenta con tarifas de pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/) o una [cuenta gratuita de Azure](https://azure.microsoft.com/offers/ms-azr-0044p/), pueden tener una cuenta de facturación para un Contrato de cliente de Microsoft.

Si no está seguro de su tipo de cuenta de facturación, consulte [Comprobación del tipo de cuenta de facturación](../manage/view-all-accounts.md#check-the-type-of-your-account) antes de seguir las instrucciones de este artículo. 

Una cuenta de facturación de MOSP puede tener las siguientes facturas:

**Cargos por servicios de Azure**: se genera una factura para cada suscripción de Azure que contenga recursos de Azure usados por la suscripción. La factura contiene los cargos de un período de facturación. El período de facturación viene determinado por el día del mes en que se creó la suscripción.

Por ejemplo, Juan crea *Azure sub 01* el 5 de marzo y *Azure sub 02* el 10 de marzo. La factura de *Azure sub 01* tendrá cargos desde el quinto día de un mes al cuarto día del mes siguiente. La factura de *Azure sub 02* tendrá cargos desde el décimo día de un mes al noveno día del mes siguiente. Las facturas para todas las suscripciones de Azure se generan normalmente el día del mes en que se creó la cuenta, pero puede ser hasta dos días más tarde. En este ejemplo, si Juan creó su cuenta el 2 de febrero, las facturas para *Azure sub 01* y *Azure sub 02* se generarán normalmente el segundo día de cada mes, pero también podrían crearse hasta dos días más tarde.

**Azure Marketplace, reservas y máquinas virtuales de Spot**: se genera una factura para las reservas, los productos de Marketplace y las máquinas virtuales de Spot adquiridos mediante una suscripción. La factura muestra los cargos correspondientes del mes anterior. Por ejemplo, Juan compró una reserva el 1 de marzo y otra reserva el 30 de marzo. Se generará una sola factura para ambas reservas en abril. Las facturas de Azure Marketplace, reservas y máquinas virtuales de Spot siempre se generan aproximadamente en el noveno día del mes.

Si paga por Azure con una tarjeta de crédito y compra una reserva, Azure genera una factura inmediata. Sin embargo, cuando se facture, se le cobrará la reserva en la siguiente factura mensual.

**Plan de soporte técnico de Azure**: se genera una factura cada mes para la suscripción del plan de soporte técnico. La primera factura se genera el día de la compra o hasta dos días más tarde. Las facturas del plan de soporte técnico posteriores se generan normalmente el mismo día del mes en que se creó la cuenta, pero puede ser hasta dos días más tarde.

## <a name="download-your-mosp-azure-subscription-invoice"></a>Descarga de la factura de suscripción de Azure MOSP

Solo se genera una factura para una suscripción que pertenece a una cuenta de facturación de MOSP. [Compruebe si tiene acceso a una cuenta de MOSP](../manage/view-all-accounts.md#check-the-type-of-your-account). 

Debe tener un rol de administrador de cuenta de una suscripción para descargar su factura. Los usuarios con roles de propietario, colaborador o lector pueden descargar su factura si el administrador de la cuenta les ha concedido permiso. Para más información, consulte [Permitir que los usuarios descarguen facturas](../manage/manage-billing-access.md#opt-in).

1. Seleccione su suscripción en la [página Suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) de Azure Portal.
1. Seleccione **Facturas** en la sección de facturación.  
    ![Captura de pantalla que muestra un usuario que selecciona la opción Facturas para una suscripción](./media/download-azure-invoice/select-subscription-invoice.png)
1. Seleccione la factura que quiera descargar y haga clic en **Descargar facturas**.  
    ![Captura de pantalla de la opción de descarga de una factura del Programa de suscripción en línea de Microsoft.](./media/download-azure-invoice/downloadinvoice-subscription.png)
1. También puede descargar un desglose diario de las cantidades consumidas y los cargos, haga clic en el icono de descarga y, a continuación, en el botón **Prepare Azure usage file** (Preparar archivo de uso de Azure) en la sección de detalles de uso. La preparación del archivo CSV puede tardar unos minutos.  
    ![Captura de pantalla que muestra la página Descarga de factura y uso](./media/download-azure-invoice/usage-and-invoice-subscription.png)

Para más información sobre la factura, consulte [Comprender la factura de Microsoft Azure](../understand/review-individual-bill.md). Para ayuda en la identificación de los costos atípicos, consulte [Análisis de cargos inesperados](analyze-unexpected-charges.md).

## <a name="download-your-mosp-support-plan-invoice"></a>Descarga de la factura del plan de soporte técnico de MOSP

Solo se genera una factura para una suscripción de plan de soporte técnico que pertenece a una cuenta de facturación de MOSP. [Compruebe si tiene acceso a una cuenta de MOSP](../manage/view-all-accounts.md#check-the-type-of-your-account).

Debe tener un rol de administrador de cuenta en la suscripción de plan de soporte técnico para descargar su factura.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque **Administración de costos + facturación**.  
    ![Captura de pantalla que muestra la búsqueda en el portal para la administración de costos y facturación](./media/download-azure-invoice/search-cmb.png)
1. Seleccione **Facturas** en el lado izquierdo.
1. Seleccione la suscripción al plan de soporte técnico.  
    [![Captura de pantalla que muestra una lista de perfiles de facturación del plan de soporte técnico del Programa de suscripción en línea de Microsoft.](./media/download-azure-invoice/cmb-invoices.png)](./media/download-azure-invoice/cmb-invoices-zoomed-in.png#lightbox)
1. Seleccione la factura que quiera descargar y haga clic en **Descargar facturas**.  
    ![Captura de pantalla que muestra la opción de descarga de una factura del plan de soporte técnico del Programa de suscripción en línea de Microsoft.](./media/download-azure-invoice/download-invoice-support-plan.png)

## <a name="allow-others-to-download-your-subscription-invoice"></a>Concesión de permiso a otros usuarios para descargar la factura de la suscripción

Para descargar una factura:

1.  Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de cuenta para la suscripción.

2.  Busque **Administración de costos + facturación**.

    ![Captura de pantalla que muestra la búsqueda en el portal para la administración de cosos y facturación](./media/download-azure-invoice/search-cmb.png)

3.  Seleccione **Facturas** en el lado izquierdo.

4.  Seleccione la suscripción de Azure y haga clic en **Permitir que otros usuarios descarguen la factura**.

    [![Captura de pantalla que muestra la selección del acceso a la factura](./media/download-azure-invoice/cmb-select-access-to-invoice.png)](./media/download-azure-invoice/cmb-select-access-to-invoice-zoomed-in.png#lightbox)

5.  En la parte superior de la página, seleccione **Activado** y, a continuación, **Guardar**.  
    ![Captura de pantalla que muestra la selección del acceso a la factura](./media/download-azure-invoice/cmb-access-to-invoice.png)
    
> [!NOTE]
> Microsoft no recomienda compartir información confidencial o de identificación personal con terceros. Esta recomendación se aplica al uso compartido de la factura de Azure con terceros para optimizar costos. Para obtener más información, vea https://azure.microsoft.com/support/legal/ y https://www.microsoft.com/trust-center.

## <a name="get-mosp-subscription-invoice-in-email"></a>Recepción de la factura de suscripción de MOSP en el correo electrónico

Debe tener un rol de administrador de cuenta en una suscripción o un plan de soporte técnico para poder recibir su factura por correo electrónico. Las facturas por correo electrónico solo están disponibles para las suscripciones y los planes de soporte técnico, no para reservas o compras de Azure Marketplace. Tras seleccionar esta opción, puede agregar destinatarios adicionales, que también recibirán la factura por correo electrónico.

1.  Inicie sesión en [Azure Portal](https://portal.azure.com).
2.  Busque **Administración de costos + facturación**.  
3.  Seleccione **Facturas** en el lado izquierdo.
4.  Seleccione la suscripción a Azure o del plan de soporte técnico y, a continuación, seleccione **Receive invoice by email** (Recibir factura por correo electrónico).  
    [![Captura de pantalla que muestra la opción Recibir factura por correo electrónico.](./media/download-azure-invoice/cmb-email-invoice.png)](./media/download-azure-invoice/cmb-email-invoice-zoomed-in.png#lightbox)
5. Haga clic en **Enviar factura por correo electrónico** y acepte los términos.  
    ![Captura de pantalla que muestra el paso 2 del flujo de participación](./media/download-azure-invoice/invoicearticlestep02.png)
6. La factura se enviará a su dirección de correo electrónico de comunicación preferida. Seleccione **Actualizar perfil** para actualizar el correo electrónico.  
    ![Captura de pantalla que muestra el paso 3 del flujo de participación](./media/download-azure-invoice/invoicearticlestep03-verifyemail.png)

## <a name="share-subscription-and-support-plan-invoice"></a>Uso compartido de facturas del plan de soporte técnico y suscripción

Es posible que desee compartir las facturas de su suscripción y su plan de soporte técnico cada mes con el equipo de contabilidad o enviarlas a una de sus otras direcciones de correo electrónico.

1. Siga los pasos de [Recepción de facturas de suscripción y plan de soporte técnico por correo electrónico](#get-mosp-subscription-invoice-in-email) y seleccione **Configurar destinatarios**.  
    [![Captura de pantalla que muestra un usuario seleccionando la configuración de destinatarios.](./media/download-azure-invoice/invoice-article-step03.png)](./media/download-azure-invoice/invoice-article-step03-zoomed.png#lightbox)
1. Escriba una dirección de correo electrónico y seleccione **Agregar destinatario**. Puede agregar varias direcciones de correo electrónico.  
    ![Captura de pantalla que muestra a un usuario agregando destinatarios adicionales](./media/download-azure-invoice/invoice-article-step04.png)
1. Una vez agregadas todas las direcciones de correo electrónico, seleccione **Listo** en la parte inferior de la pantalla.

## <a name="invoices-for-mca-and-mpa-billing-accounts"></a>Facturas de cuentas de facturación de MCA y MPA

Cuando su organización trabaja con un representante de Microsoft para firmar un Contrato de cliente de Microsoft (MCA), se crea una cuenta de facturación de MCA. Algunos clientes de regiones seleccionadas, que se registran en el sitio web de Azure para una [cuenta con tarifas de pago por uso](https://azure.microsoft.com/offers/ms-azr-0003p/) o una [cuenta gratuita de Azure](https://azure.microsoft.com/offers/ms-azr-0044p/), también pueden tener una cuenta de facturación para un Contrato de cliente de Microsoft. Para más información, consulte [Introducción a la cuenta de facturación de MCA](../understand/mca-overview.md).

Para los asociados del programa Proveedor de soluciones en la nube (CSP), se crea una cuenta de facturación de MCA con el fin de administrar sus clientes en la nueva experiencia comercial. Los asociados deben tener al menos un cliente con un [plan de Azure](/partner-center/purchase-azure-plan) para administrar su cuenta de facturación en Azure Portal. Para más información, consulte [Introducción a la cuenta de facturación de MPA](../understand/mpa-overview.md).

Una factura mensual se genera al comienzo del mes para cada perfil de facturación de la cuenta. La factura incluye los cargos respectivos de todas las suscripciones de Azure y otras compras realizadas el mes anterior. Por ejemplo, Juan creó *Azure sub 01* el 5 de marzo y *Azure sub 02* el 10 de marzo. Ha adquirido la suscripción de *Soporte técnico de Azure 01* el 28 de marzo mediante *Perfil de facturación 01*. Juan recibirá una sola factura al principio de abril que contendrá los cargos de las suscripciones de Azure y del plan de soporte técnico.

##  <a name="download-an-mca-or-mpa-billing-profile-invoice"></a>Descarga de una factura de perfil de facturación de MCA o MPA

Debe tener un rol de propietario, colaborador, lector o administrador de facturación del perfil de facturación para descargar su factura en Azure Portal. Los usuarios con un rol de propietario, colaborador o lector en una cuenta de facturación pueden descargar facturas para todos los perfiles de facturación de la cuenta.

1.  Inicie sesión en [Azure Portal](https://portal.azure.com).

2.  Busque **Administración de costos + facturación**.

    ![Captura de pantalla que muestra la búsqueda en el portal para la administración de cosos y facturación](./media/download-azure-invoice/search-cmb.png)

3. Seleccione **Facturas** en el lado izquierdo.

    [![Captura de pantalla que muestra la página Facturas de una cuenta de facturación de MCA](./media/download-azure-invoice/mca-billingprofile-invoices.png)](./media/download-azure-invoice/mca-billingprofile-invoices-zoomed-in.png#lightbox)

4. En la tabla de facturas, seleccione la factura que desea descargar.

5. Haga clic en el botón **Descargar PDF de factura** en la parte superior de la página.

    [![Captura de pantalla que muestra la descarga del archivo PDF de una factura](./media/download-azure-invoice/mca-billingprofile-download-invoice.png)](./media/download-azure-invoice/mca-billingprofile-download-invoice-zoomed-in.png#lightbox)

6. Asimismo, para descargar un desglose diario de las cantidades consumidas y los gastos estimados, haga clic en **Descargar uso de Azure**. La preparación del archivo csv puede tardar unos minutos.

## <a name="get-your-billing-profiles-invoice-in-email"></a>Recepción de la factura del perfil de facturación por correo electrónico

Debe tener un rol de propietario o colaborador en el perfil de facturación o en su cuenta de facturación para actualizar su preferencia de factura por correo electrónico. Tras habilitar el envío, todos los usuarios con un rol de propietario, colaborador, lector o administrador de facturas de un perfil de facturación recibirán su factura por correo electrónico. 

1.  Inicie sesión en [Azure Portal](https://portal.azure.com).
1.  Busque **Administración de costos + facturación**.  
1.  Seleccione **Facturas** en el panel izquierdo y, a continuación, **Preferencias sobre la recepción de facturas por correo electrónico** en la parte superior de la página.  
    [![Captura de pantalla que muestra la opción Factura por correo electrónico para las facturas.](./media/download-azure-invoice/mca-billing-profile-select-email-invoice.png)](./media/download-azure-invoice/mca-billing-profile-select-email-invoice-zoomed.png#lightbox)
1.  Si tiene varios perfiles de facturación, seleccione uno y **Sí**.  
    [![Captura de pantalla que muestra la opción de participar.](./media/download-azure-invoice/mca-billing-profile-email-invoice.png)](./media/download-azure-invoice/mca-billing-profile-select-email-invoice-zoomed.png#lightbox)
1.  Seleccione **Guardar**.

Puede conceder acceso a otros usuarios para ver, descargar y pagar facturas si les asigna el rol de administrador de facturas para un perfil de facturación de MCA o MPA. Si ha optado por recibir la factura por correo electrónico, los usuarios también recibirán las facturas en el correo electrónico.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque **Administración de costos + facturación**.  
1. Seleccione **Perfiles de facturación** en el lado izquierdo. En la lista de perfiles de facturación, seleccione uno para el que quiera asignar un rol de administrador de facturas.  
   ![Captura de pantalla que muestra la lista de perfiles de facturación donde se selecciona uno.](./media/download-azure-invoice/mca-select-profile-zoomed-in.png)
1. Seleccione **Access Control (IAM)** en el lado izquierdo y, a continuación, **Agregar** en la parte superior de la página.  
    ![Captura de pantalla que muestra la página de control de acceso](./media/download-azure-invoice/mca-select-access-control-zoomed-in.png)
1. En la lista desplegable Rol, seleccione **Administrador de facturación**. Escriba la dirección de correo electrónico del usuario al que desea dar acceso. Seleccione **Guardar** para asignar el rol.  
    [![Captura de pantalla que muestra cómo agregar un usuario como administrador de facturas](./media/download-azure-invoice/mca-added-invoice-manager.png)](./media/download-azure-invoice/mca-added-invoice-manager.png#lightbox)
   

## <a name="share-your-billing-profiles-invoice"></a>Uso compartido de la factura del perfil de facturación

Es posible que quiera compartir la factura cada mes con el equipo de contabilidad o enviarla a una de sus otras direcciones de correo electrónico sin conceder al equipo de contabilidad ni a otros permisos de correo electrónico a su perfil de facturación.

1.  Inicie sesión en [Azure Portal](https://portal.azure.com).
1.  Busque **Administración de costos + facturación**.  
1.  Seleccione **Facturas** en el panel izquierdo y, a continuación, **Preferencias sobre la recepción de facturas por correo electrónico** en la parte superior de la página.  
    [![Captura de pantalla que muestra la opción Factura por correo electrónico para las facturas.](./media/download-azure-invoice/mca-billing-profile-select-email-invoice.png)](./media/download-azure-invoice/mca-billing-profile-select-email-invoice-zoomed.png#lightbox)
1.  Si tiene varios perfiles de facturación, seleccione uno.
1.  En la sección de destinatarios adicionales, agregue las direcciones de correo electrónico para recibir facturas.
    [![Captura de pantalla que muestra destinatarios adicionales para la factura por correo electrónico.](./media/download-azure-invoice/mca-billing-profile-add-invoice-recipients.png)](./media/download-azure-invoice/mca-billing-profile-add-invoice-recipients-zoomed.png#lightbox)
1.  Seleccione **Guardar**.

##  <a name="why-you-might-not-see-an-invoice"></a>¿Por qué no puede ver una factura?

<a name="noinvoice"></a>

Pueden existir varias razones por las que no ve una factura:

- La factura todavía no está lista.
    
    - Han transcurrido menos de 30 días desde el día que se suscribió a Azure. 

    - Azure le factura unos días después del final del período de facturación. Por lo tanto, es posible que todavía no se haya generado una factura.

- No tiene permiso para ver facturas. 
    
    - Si tiene una cuenta de facturación de MCA o MPA, debe tener un rol de propietario, colaborador, lector o administrador de facturación en un perfil de facturación o el rol de propietario, colaborador o lector en la cuenta de facturación para ver facturas. 
    
    - Para otras cuentas de facturación, es posible que no vea las facturas si no es el administrador de cuenta.

- La cuenta no admite una factura.

    - Si tiene una cuenta de facturación del Programa de suscripción en línea de Microsoft (MOSP) y se registró para obtener una cuenta gratuita de Azure o una suscripción con un importe de crédito mensual, solo recibirá una factura cuando supere el importe de crédito mensual.

    - Si tiene una cuenta de facturación para un Contrato de cliente de Microsoft (MCA) o un contrato Microsoft Partner Agreement, siempre recibirá una factura.

- Tiene acceso a la factura mediante una de sus otras cuentas.

    - Normalmente, esta situación se produce cuando se hace clic en un vínculo en el correo electrónico y se le pide que vea la factura en el portal. Haga clic en el vínculo y verá un mensaje de error: `We can't display your invoices. Please try again`. Compruebe que ha iniciado sesión con la dirección de correo electrónico que tiene permisos para ver las facturas.

- Tiene acceso a la factura mediante una identidad diferente. 

    - Algunos clientes tienen dos identidades con la misma dirección de correo electrónico: una cuenta profesional y una cuenta de Microsoft. Normalmente, solo una de sus identidades tiene permisos para ver facturas. Si inicia sesión con la identidad que no tiene permiso, no verá las facturas. Compruebe que está usando la identidad correcta para iniciar sesión.

- Ha iniciado sesión en el inquilino de Azure Active Directory (Azure AD) incorrecto. 

    - Su cuenta de facturación está asociada a un inquilino de Azure AD. Si ha iniciado sesión en un inquilino incorrecto, no verá la factura de las suscripciones de la cuenta de facturación. Compruebe que ha iniciado sesión en el inquilino de Azure AD correcto. Si no ha iniciado sesión en el inquilino correcto, use lo siguiente para cambiar el inquilino en Azure Portal:

        1. Seleccione el correo electrónico en la parte superior derecha de la página.

        2. Seleccione **Cambiar directorio**.

           ![Captura de pantalla que muestra la selección de Cambiar directorio en el portal](./media/download-azure-invoice/select-switch-directory.png)

        3. Seleccione un directorio en la sección **Todos los directorios**.

           ![Captura de pantalla que muestra la selección de un directorio en el portal](./media/download-azure-invoice/select-directory.png)

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de la factura y los cargos, consulte:

- [Visualización y descarga de los datos de uso y los cargos de Microsoft Azure](download-azure-daily-usage.md)
- [Comprender la factura de Microsoft Azure](review-individual-bill.md)
- [Descripción de los términos en su factura de Microsoft Azure](understand-invoice.md)

Si tiene un MCA, consulte:

- [Comprender los cargos de facturación de su contrato de cliente de Microsoft](review-customer-agreement-bill.md)
- [Descripción de los términos en la factura del contrato de cliente de Microsoft](mca-understand-your-invoice.md)
- [Descripción de los términos en los CSV de cargos y el uso de Azure para un contrato de cliente de Microsoft](mca-understand-your-usage.md)