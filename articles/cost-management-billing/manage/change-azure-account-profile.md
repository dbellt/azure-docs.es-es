---
title: Cambio de la información de contacto de una cuenta de facturación de Azure
description: Describe cómo cambiar la información de contacto de su cuenta de facturación de Azure
author: bandersmsft
ms.reviewer: judupont
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 04/08/2021
ms.author: banders
ms.custom: contperf-fy21q2
ms.openlocfilehash: 43b3332629922bb60eefb68782b84e996b692848
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110461489"
---
# <a name="change-contact-information-for-an-azure-billing-account"></a>Cambio de la información de contacto de una cuenta de facturación de Azure

Este artículo le ayuda a actualizar la información de contacto de su *cuenta de facturación* en Azure Portal. Las instrucciones para actualizar la información de contacto varían en función del tipo de cuenta de facturación. Para más información sobre las cuentas de facturación y la identificación del tipo de cuenta de facturación, consulte [Visualización de las cuentas de facturación en Azure Portal](view-all-accounts.md). Una cuenta de facturación de Azure es independiente de la cuenta de usuario de Azure y de la [cuenta de Microsoft](https://account.microsoft.com/).

Si desea actualizar la información del perfil de usuario de Azure Active Directory, solo un administrador de usuarios puede realizar los cambios. Si no tiene asignado el rol de administrador de usuarios, póngase en contacto con el administrador de usuarios. Para más información sobre el cambio de perfil de un usuario, consulte [Incorporación o actualización de la información de perfil de un usuario mediante Azure Active Directory](../../active-directory/fundamentals/active-directory-users-profile-azure-portal.md).

*Dirección del comprador*: la dirección del comprador es la dirección e información de contacto de la organización o individuo responsables de una cuenta de facturación. Se muestra en todas las facturas generadas para la cuenta de facturación.

*Dirección de facturación*: la dirección del comprador es la dirección e información de contacto de la organización o individuo responsables de las facturas generadas para una cuenta de facturación. En el caso de una cuenta de facturación de un programa Microsoft Online Service Program (MOSP), hay una dirección de facturación, que se muestra en todas las facturas generadas para la cuenta. En el caso de una cuenta de facturación para un Contrato de cliente de Microsoft (MCA), hay una dirección de facturación para cada perfil de facturación y se muestra en la factura generada para el perfil de facturación.

*Dirección de correo electrónico de contacto para los correos electrónicos de servicio y marketing*: puede especificar una dirección de correo electrónico que sea distinta de la que usa para iniciar sesión para recibir notificaciones importantes relacionadas con la facturación y el servicio, así como recomendaciones sobre su cuenta de Azure. Las notificaciones del servicio por correo electrónico (como los problemas de seguridad urgentes, los cambios de precio o los cambios importantes en los servicios que usa su cuenta) siempre se envían a la dirección de inicio de sesión.

## <a name="update-an-mosp-billing-account-address"></a>Actualización de la dirección de la cuenta de facturación de MOSP

1. Inicie sesión en Azure Portal mediante la dirección de correo electrónico que tenga el permiso de administrador de cuenta en la cuenta.
1. Busque **Administración de costos + facturación**.  
    ![Captura de pantalla que muestra la búsqueda en Azure Portal de Cost Management + Billing](./media/change-azure-account-profile/search-cmb.png)
1. Seleccione **Propiedades** en el lado izquierdo.  
    ![Captura de pantalla que muestra las propiedades de la cuenta de facturación de MOSP](./media/change-azure-account-profile/update-contact-information-select-properties.png)
1. Seleccione **Actualizar dirección de facturación** para actualizar las direcciones del comprador y de facturación. Escriba la dirección nueva y seleccione **Guardar**.  
    ![Captura de pantalla que muestra la dirección de actualización de la cuenta de facturación de MOSP](./media/change-azure-account-profile/update-contact-information-mosp.png)

## <a name="update-an-mca-billing-account-sold-to-address"></a>Actualización de la dirección del comprador de una cuenta de facturación de MCA

1. Inicie sesión en Azure Portal mediante la dirección de correo electrónico, que tiene un rol de colaborador o propietario en la cuenta de facturación de un contrato de cliente de Microsoft.
1. Busque **Administración de costos + facturación**.  
    ![Captura de pantalla que muestra dónde buscar en Azure Portal](./media/change-azure-account-profile/search-cmb.png)
1. Seleccione **Propiedades** en el lado izquierdo y, después, seleccione **Update sold-to** (Actualizar comprador).  
    ![Captura de pantalla que muestra las propiedades de una cuenta de facturación de MCA en la que puede modificar la dirección de comprador](./media/change-azure-account-profile/update-sold-to-list-properties-mca.png)
1. Escriba la dirección nueva y seleccione **Guardar**.  
    ![Captura de pantalla que muestra la actualización de la dirección del comprador para una cuenta de MCA](./media/change-azure-account-profile/update-sold-to-save-mca.png)

    > [!IMPORTANT]
    > Algunas cuentas requieren una comprobación adicional para poder actualizar sus datos de comprador. Si su cuenta requiere aprobación manual, se le pedirá que se ponga en contacto con el soporte técnico de Azure.

## <a name="update-an-mca-billing-account-address"></a>Actualización de la dirección de la cuenta de facturación de MCA

1. Inicie sesión en Azure Portal mediante la dirección de correo electrónico, que tiene un rol de colaborador o propietario en una cuenta de facturación o un perfil de facturación de un MCA.
1. Busque **Administración de costos + facturación**.  
1. Seleccione **Perfiles de facturación** en el lado izquierdo.
1. Seleccione un perfil de facturación para actualizar la dirección de facturación.  
    ![Captura de pantalla que muestra la página Perfiles de facturación donde se selecciona un perfil de facturación](./media/change-azure-account-profile/update-bill-to-list-profiles-mca.png)
1. Seleccione **Propiedades** en el lado izquierdo.
1. Seleccione **Actualizar dirección**.  
    ![Captura de pantalla que muestra dónde actualizar la dirección](./media/change-azure-account-profile/update-bill-to-list-properties-mca.png)
1. Escriba la dirección nueva y seleccione **Guardar**.  
    ![Captura de pantalla que muestra la actualización de la dirección](./media/change-azure-account-profile/update-bill-to-save-mca.png)

## <a name="update-a-po-number"></a>Actualización de un número de pedido

De forma predeterminada, la factura del perfil de facturación no tiene un número de pedido de compra asociado. Después de agregar un número de pedido para un perfil de facturación, este aparece en las facturas de dicho perfil.

Para agregar o cambiar el número de pedido de un perfil de facturación, siga estos pasos.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque **Cost Management + Billing** y, a continuación, seleccione **Ámbitos de facturación**.
1. Seleccione el ámbito de facturación.
1. En el menú de la izquierda, en **Facturación**, seleccione **Perfiles de facturación**.
1. Seleccione el perfil de facturación adecuado.
1. En el menú de la izquierda, en **Configuración**, seleccione **Propiedades**.
1. Seleccione **Actualizar el número de pedido de compra**.
1. Escriba un número de pedido de compra y, a continuación, seleccione **Actualizar**.

## <a name="update-your-tax-id"></a>Actualización del identificador fiscal

Asegúrese de actualizar el identificador fiscal después de mover las suscripciones. Este número de identificación fiscal se utiliza para los cálculos de exención fiscal y aparece en la factura.

**Para actualizar la información de las cuentas de facturación**

1. Inicie sesión en [Microsoft Store para Empresas](https://businessstore.microsoft.com/) o [Microsoft Store para Educación](https://educationstore.microsoft.com/).
1. Seleccione **Administrar** y, a continuación, seleccione **Cuentas de facturación**.
1. En **Información general**, seleccione **Edit billing account information** (Editar información de la cuenta de facturación).
1. Realice las actualizaciones y, después, seleccione **Guardar**.

[Obtenga más información sobre cómo actualizar la configuración de la cuenta de facturación](/microsoft-store/update-microsoft-store-for-business-account-settings).


## <a name="service-and-marketing-emails"></a>Correos electrónicos de servicio y marketing

Se le solicita en Azure Portal que compruebe o actualice la dirección de correo electrónico cada 90 días. Microsoft envía correos electrónicos a esta dirección con información relacionada con la cuenta de Azure para:

- Notificaciones de servicio
- Alertas de seguridad
- Propósitos de facturación
- Soporte técnico
- Comunicaciones de marketing
- Procedimientos recomendados, según el uso de Azure

Escriba la dirección de correo electrónico donde quiera recibir comunicaciones sobre su cuenta. Al escribir una dirección de correo electrónico, está dando su consentimiento para recibir comunicaciones de Microsoft.

![Ejemplo de la solicitud para actualizar la información de contacto](./media/change-azure-account-profile/update-contact-information.png)

### <a name="change-your-contact-email-address"></a>Cambiar la dirección de correo electrónico de contacto

Para cambiar la dirección de correo electrónico de contacto, use uno de estos métodos. La actualización de la dirección de correo electrónico de contacto no actualiza la dirección de correo electrónico con la que se inicia sesión.

1. Si es administrador de cuenta de una cuenta de MOSP, siga las instrucciones que encontrará en el apartado [Actualización de la dirección de la cuenta de facturación de MOSP](#update-an-mosp-billing-account-address) y seleccione **Actualizar información de contacto** en el último paso. A continuación, escriba la nueva dirección de correo electrónico.
1. Vaya al área [Información de contacto](https://portal.azure.com/#blade/HubsExtension/ContactInfoBlade) en Azure Portal y escriba la nueva dirección de correo electrónico. 
1. En Azure Portal, seleccione el icono con sus iniciales o imagen. A continuación, seleccione el menú de contexto ( **...** ). Seleccione **Mi información de contacto** en el menú y escriba la nueva dirección de correo electrónico.

![Ejemplo de actualización de una dirección de correo electrónico en Azure](./media/change-azure-account-profile/azure-contact-information.png)

### <a name="opt-out-of-marketing-emails"></a>Optar por no recibir correos electrónicos de marketing

Para optar por no recibir correos electrónicos de marketing:

1. Vaya al [formulario de solicitud](https://account.microsoft.com/profile/permissions-link-request) para enviar una solicitud utilizando su dirección de correo electrónico de perfil. Recibirá un vínculo por correo electrónico para actualizar sus preferencias.
1. Seleccione el vínculo para abrir la página **Administrar permisos de comunicación**. Esta página le muestra los tipos de comunicaciones de marketing en las que está habilitada la dirección de correo electrónico. Borre todas las opciones seleccionadas que quiera excluir y seleccione **Guardar**.  
    ![Ejemplo de la página para administrar los permisos de comunicación](./media/change-azure-account-profile/manage-communication-permissions.png)

Cuando opta por no recibir comunicaciones de marketing, seguirá recibiendo notificaciones del servicio, en función de su cuenta.

## <a name="update-the-email-address-that-you-sign-in-with"></a>Actualización de la dirección de correo electrónico con la que se inicia sesión

La dirección de correo electrónico que se usa para acceder a la cuenta propia no se puede actualizar. Sin embargo, si tiene una cuenta de facturación para un MOSP, puede iniciar registrarse en otra cuenta mediante la nueva dirección de correo electrónico y transferir la propiedad de las suscripciones a la siguiente cuenta. En el caso de una cuenta de facturación de MCA, puede dar a la nueva dirección de correo electrónico permisos en su cuenta.

## <a name="update-your-credit-card"></a>Actualización de la tarjeta de crédito

Para aprender a actualizar una tarjeta de crédito, consulte el artículo en que se explica cómo [cambiar la tarjeta de crédito usada para pagar una suscripción de Azure](change-credit-card.md).

## <a name="update-your-country-or-region"></a>Actualización de su país o región

No se admite el cambio de país o región de las cuentas existentes. Sin embargo, se puede crear una cuenta en otro país o región y, luego, ponerse en contacto con el soporte técnico de Azure para transferir una suscripción a la cuenta nueva.

## <a name="change-the-subscription-name"></a>Cambio del nombre de la suscripción

1. Inicie sesión en Azure Portal, seleccione **Suscripción** en el panel izquierdo y después seleccione la suscripción cuyo nombre quiera cambiar.
1. Seleccione **Introducción** y después **Cambiar nombre** en la barra de comandos.  
    ![Ejemplo de cambio de nombre de una suscripción de Azure](./media/change-azure-account-profile/rename-sub.png)
1. Después de cambiar el nombre, seleccione **Guardar**.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- [Ver las cuentas de facturación](view-all-accounts.md)
