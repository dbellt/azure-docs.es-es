---
title: Acerca de la transferencia de la propiedad de facturación para una suscripción de Azure
description: En este artículo se explican las cosas que debe saber antes de transferir la propiedad de facturación de una suscripción de Azure a otra cuenta.
keywords: transferencia de la suscripción de Azure, suscripción de transferencia de Azure, traslado de la suscripción de Azure a otra cuenta, cambio de propietario de la suscripción de Azure, transferencia de la suscripción de Azure a otra cuenta, facturación de transferencia de Azure
author: bandersmsft
ms.reviewer: amberb
tags: billing,top-support-issue
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: banders
ms.custom: contperf-fy21q1
ms.openlocfilehash: 746845004a7a1baf1cafb872aae9322177879b6b
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/11/2021
ms.locfileid: "103011109"
---
# <a name="about-transferring-billing-ownership-for-an-azure-subscription"></a>Acerca de la transferencia de la propiedad de facturación para una suscripción de Azure

Este artículo le ayudará a comprender las cosas que debe saber antes de transferir la propiedad de facturación de una suscripción de Azure a otra cuenta. 

Es posible que quiera transferir la propiedad de facturación de la suscripción de Azure si deja la organización o que quiera que la suscripción se facture en otra cuenta. La transferencia de la propiedad de facturación a otra cuenta proporciona a los administradores el permiso de nueva cuenta para las tareas de facturación. Pueden cambiar el método de pago, ver los cargos y cancelar la suscripción.

Si quiere mantener la propiedad de facturación, pero cambiar el tipo de suscripción, consulte [Cambio de la suscripción de Azure a otra oferta](../manage/switch-azure-offer.md). Para controlar quién puede acceder a los recursos de la suscripción, consulte [Roles integrados en Azure](../../role-based-access-control/built-in-roles.md).

Si es un cliente de Contrato Enterprise (EA), los administradores de la empresa pueden transferir la propiedad de facturación de las suscripciones entre cuentas.

Solo el administrador de facturación de una cuenta puede transferir la propiedad de una suscripción.

## <a name="determine-if-you-are-a-billing-administrator"></a>Determinación de si es usted un administrador de facturación

<a name="whoisaa"></a>

Para realizar la transferencia, busque la persona que tiene acceso para administrar la facturación de la cuenta. Está autorizado para tener acceso a la facturación en [Azure Portal](https://portal.azure.com) y realizar diversas tareas de facturación, como crear suscripciones, ver y pagar facturas o actualizar los métodos de pago.

### <a name="check-if-you-have-billing-access"></a>Comprobación de si tiene acceso a la facturación

1. Para identificar las cuentas para las que tiene acceso de facturación, visite la página [Administración de costos + facturación en Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Billing/ModernBillingMenuBlade/Overview).

2. Seleccione **Cuentas de facturación** en el menú de la izquierda.

3. La página **Ámbito de facturación** muestra todas las suscripciones en las que tiene acceso a los detalles de facturación.

### <a name="check-by-subscription"></a>Comprobación por suscripción

1. Si no está seguro de quién es el administrador de la cuenta de una suscripción, visite la [página Suscripciones en Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade). 

2. Seleccione la suscripción que desea comprobar.

3. En el encabezado **Configuración**, seleccione **Propiedades**. Consulte el cuadro **Administrador de cuenta** para saber quién es el administrador de cuenta de la suscripción.

   > [!NOTE]
   > No todos los tipos de suscripción muestran las propiedades.

## <a name="supported-subscription-types"></a>Tipos de suscripciones admitidos

La transferencia de suscripciones en Azure Portal está disponible para los tipos de suscripción que se enumeran a continuación. Actualmente no se admite transferencia de suscripciones de [Evaluación gratuita](https://azure.microsoft.com/offers/ms-azr-0044p/) o [Azure bajo licencia Open](https://azure.microsoft.com/offers/ms-azr-0111p/). Para ver una solución alternativa, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../azure-resource-manager/management/move-resource-group-and-subscription.md). Para transferir otras suscripciones, como planes de soporte técnico, [póngase en contacto con el soporte técnico de Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

- [Enterprise Agreement (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/)<sup>1</sup>
- [Microsoft Partner Network](https://azure.microsoft.com/offers/ms-azr-0025p/)  
- [Suscriptores de Visual Studio Enterprise (MPN)](https://azure.microsoft.com/offers/ms-azr-0029p/)
- [Plataformas de MSDN](https://azure.microsoft.com/offers/ms-azr-0062p/)  
- [Pay-As-You-Go](https://azure.microsoft.com/offers/ms-azr-0003p/)
- [Desarrollo/pruebas - Pago por uso](https://azure.microsoft.com/offers/ms-azr-0023p/)
- [Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/)
- [Visual Studio Enterprise: BizSpark](https://azure.microsoft.com/offers/ms-azr-0064p/)
- [Visual Studio Professional](https://azure.microsoft.com/offers/ms-azr-0059p/)
- [Visual Studio Test Professional](https://azure.microsoft.com/offers/ms-azr-0060p/)
- [Microsoft Azure Plan](https://azure.microsoft.com/offers/ms-azr-0017g/)<sup>2</sup>

<sup>1</sup> Mediante EA Portal.

<sup>2</sup> Solo se admite para las cuentas que se crean durante el registro en el sitio web de Azure.

## <a name="resources-transferred-with-subscriptions"></a>Recursos transferidos con suscripciones

Todos los recursos, como las VM, los discos y los sitios web, se transfieren a la nueva cuenta. Sin embargo, si transfiere una suscripción a una cuenta de otro inquilino de Azure AD, los [roles de administrador](../manage/add-change-subscription-administrator.md) y las [asignaciones de roles de Azure](../../role-based-access-control/role-assignments-portal.md) de la suscripción no se transfieren. Tampoco se transferirán los [registros de aplicaciones](../../active-directory/develop/quickstart-register-app.md) ni otros servicios específicos del inquilino junto con la suscripción.

## <a name="transfer-account-ownership-to-another-countryregion"></a>Transferencia de la propiedad de cuenta a otro país o región

Desafortunadamente, no puede transferir suscripciones entre países o regiones mediante Azure Portal. Sin embargo, se pueden transferir si abre una solicitud de soporte técnico de Azure. Para crear una solicitud de soporte técnico, [póngase en contacto con el soporte técnico](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="transfer-a-subscription-from-one-account-to-another"></a>Transferencia de una suscripción de una cuenta a otra

Si es administrador de dos cuentas, puede transferir una suscripción entre sus cuentas. En teoría, las cuentas se consideran cuentas de dos usuarios distintos, por lo que puede transferir las suscripciones entre las cuentas.
Para ver los pasos necesarios para transferir su suscripción, consulte [Transferencia de la propiedad de facturación de una suscripción de Azure](../manage/billing-subscription-transfer.md).

## <a name="transferring-a-subscription-shouldnt-create-downtime"></a>La transferencia de una suscripción no debe crear tiempo de inactividad

Si transfiere una suscripción a una cuenta en el mismo inquilino de Azure AD, los recursos que se ejecutan en la suscripción no se ven afectados. Sin embargo, la información de contexto guardada en PowerShell no se actualiza, por lo que es posible que tenga que borrarla o cambiar la configuración. Si transfiere la suscripción a una cuenta de otro inquilino y decide mover la suscripción al inquilino, todos los usuarios, grupos y entidades de servicio que tenían [asignaciones de roles de Azure](../../role-based-access-control/role-assignments-portal.md) para acceder a los recursos de la suscripción perderán el acceso. Podría producirse un tiempo de inactividad del servicio.

## <a name="new-account-usage-and-billing-history"></a>Historial de uso y facturación de cuenta nueva

La única información disponible para los usuarios de la nueva cuenta es el costo del último mes de la suscripción. El resto del historial de facturación y uso no se transfiere con la suscripción.

## <a name="manually-migrate-data-and-services"></a>Migración manual de datos y servicios

Cuando se transfiere una suscripción, sus recursos permanecen con ella. Si no puede transferir la propiedad de suscripción, puede migrar manualmente los recursos. Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../azure-resource-manager/management/move-resource-group-and-subscription.md).

## <a name="remaining-subscription-credits"></a>Créditos restantes de la suscripción 

Si tiene una suscripción de Visual Studio o Microsoft Partner Network, obtendrá créditos mensuales. Los créditos no se transfieren con la suscripción a la nueva cuenta. El usuario que acepta la solicitud de transferencia debe tener una licencia de Visual Studio para aceptar la solicitud de transferencia. La suscripción usa el crédito de Visual Studio que está disponible en la cuenta del usuario. Para más información, consulte [Transferencia de la propiedad de facturación de una suscripción de Azure a otra cuenta](../manage/billing-subscription-transfer.md#transfer-visual-studio-and-partner-network-subscriptions).

## <a name="users-keep-access-to-transferred-resources"></a>Los usuarios mantienen el acceso a los recursos transferidos

Tenga en cuenta que los usuarios con acceso a los recursos de una suscripción mantienen su acceso cuando se transfiere la propiedad. Sin embargo, es posible que se eliminen tanto los [roles de administrador](../manage/add-change-subscription-administrator.md) como las [asignaciones de roles de Azure](../../role-based-access-control/role-assignments-portal.md). La pérdida de acceso se produce cuando su cuenta se encuentra en un inquilino de Azure AD que no sea el inquilino de la suscripción y el usuario que envió la solicitud de transferencia mueve la suscripción al inquilino de su cuenta. 

Puede ver los usuarios que tienen asignaciones de roles de Azure para acceder a los recursos de la suscripción en Azure Portal. Visite la [página Suscripciones de Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade). A continuación, seleccione la suscripción que quiere comprobar y seleccione **Control de acceso (IAM)** en el panel izquierdo. Luego, en la parte superior de la página, seleccione **Asignaciones de roles**. La página de asignaciones de roles enumera todos los usuarios que tienen acceso en la suscripción.

Aunque las [asignaciones de roles de Azure](../../role-based-access-control/role-assignments-portal.md) se eliminen durante la transferencia, es posible que los usuarios de la cuenta del propietario original sigan teniendo acceso a la suscripción mediante otros mecanismos de seguridad, como:

* Certificados de administración que conceden al usuario derechos administrativos a los recursos de la suscripción. Para obtener más información, consulte [Crear y cargar un certificado de administración para Azure](../../cloud-services/cloud-services-certs-create.md).
* Claves de acceso para servicios como Almacenamiento. Para más información, consulte [Acerca de las cuentas de almacenamiento de Azure](../../storage/common/storage-account-create.md).
* Credenciales de acceso remoto para servicios como Azure Virtual Machines.

Si el destinatario debe restringir el acceso a sus recursos, debe considerar la actualización de todos los secretos asociados al servicio. La mayoría de los recursos se pueden actualizar. Inicie sesión en [Azure Portal](https://portal.azure.com) y seleccione **Todos los recursos** en el menú Concentrador. Después, seleccione el tamaño del recurso. A continuación, en la página de recursos, seleccione **Configuración**. Aquí puede ver y actualizar los secretos existentes.

## <a name="you-pay-for-usage-when-you-receive-ownership"></a>Se paga por el uso cuando se recibe la propiedad

La cuenta es responsable del pago de cualquier uso que se notifica desde el momento de la transferencia en adelante. Puede haber una parte del uso que se realizara antes de la transferencia, pero que se notificara con posterioridad. El uso se incluye en la factura de la cuenta.

## <a name="use-a-different-payment-method"></a>Uso de otro método de pago

Al aceptar la solicitud de transferencia, puede seleccionar un método de pago existente que esté vinculado a la cuenta o agregar uno nuevo.

## <a name="transfer-enterprise-agreement-subscription-ownership"></a>Transferencia de la propiedad de suscripción con un contrato Enterprise

El administrador de Enterprise puede actualizar la propiedad de cualquier cuenta, incluso después de que el propietario de la cuenta original ya no forme parte de la organización. Para obtener más información acerca de la transferencia de cuentas de Contrato Enterprise de Azure, consulte [Transferencias de Azure Enterprise](../manage/ea-transfers.md).

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- [Transferencia de la propiedad de facturación de una suscripción de Azure](../manage/billing-subscription-transfer.md)
