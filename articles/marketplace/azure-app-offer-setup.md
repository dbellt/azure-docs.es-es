---
title: Creación de una oferta de aplicación de Azure en Azure Marketplace
description: Cree una oferta de aplicación de Azure para mostrarla o venderla en Azure Marketplace o mediante el programa de proveedores de soluciones en la nube (CSP) con el portal del marketplace comercial.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 06/01/2021
ms.openlocfilehash: 8d59aa45343c90520b416e9bf151e06e0339038b
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111542914"
---
# <a name="create-an-azure-application-offer"></a>Creación de una oferta de aplicación de Azure

Como editor del marketplace comercial, puede crear una oferta de Aplicación de Azure para que los posibles clientes puedan comprar su solución. En este artículo se explica el proceso de crear una oferta de Aplicación de Azure en el marketplace comercial de Microsoft.

Si aún no lo ha hecho, lea la sección [Planeamiento de una oferta de Aplicación de Azure para el marketplace comercial](plan-azure-application-offer.md). Ahí encontrará recursos y ayuda para recopilar la información y los recursos que necesita al crear la oferta.

## <a name="create-a-new-offer"></a>Crear una nueva oferta

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).

1. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.

1. En la página Información general, seleccione **+ Nueva oferta** > **Aplicación de Azure**.

    ![Muestra el menú de navegación izquierdo.](./media/create-new-azure-app-offer/new-offer-azure-app.png)

1. En el cuadro de diálogo **Nueva oferta**, escriba un valor para **Id. de oferta**. Se trata de un identificador único para cada oferta de su cuenta. Este identificador es visible en la dirección URL de la oferta del marketplace comercial y de las plantillas de Azure Resource Manager, si procede. Por ejemplo, si escribe test-offer-1 en este cuadro, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.

     * Cada oferta de su cuenta debe tener un identificador único.
     * Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres.
     * El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

1. Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

     * Este nombre solo es visible en el Centro de partners y es diferente del nombre de la oferta y otros valores que se muestran a los clientes.
     * El alias de la oferta no se puede cambiar después de seleccionar **Crear**.

1. Para generar la oferta y continuar, seleccione **Crear**.

## <a name="configure-your-azure-application-offer-setup-details"></a>Configuración de los detalles de configuración de la oferta de Aplicación de Azure

En la pestaña **Offer setup** (Configuración de la oferta), en **Setup details** (Detalles de la configuración), podrá elegir si quiere configurar una versión de prueba. También conectará el sistema de administración de las relaciones con el cliente (CRM) con la oferta de marketplace comercial.

### <a name="enable-a-test-drive-optional"></a>Habilitación de una versión de prueba (opcional)

Una versión de prueba es una excelente manera de exhibir la oferta a posibles clientes al concederles acceso a un entorno preconfigurado durante un número fijo de horas. La oferta de una versión de prueba tiene como resultado una mayor tasa de conversión y genera clientes potenciales muy cualificados. Para más información sobre las versiones de prueba, consulte [Versión de prueba](plan-azure-application-offer.md#test-drive).

#### <a name="to-enable-a-test-drive"></a>Habilitación de una versión de prueba

- En **Versión de prueba**, active la casilla **Habilitar una versión de prueba**.

### <a name="customer-lead-management"></a>Administración de clientes potenciales

Conecte el sistema de administración de relaciones con clientes (CRM) con la oferta del marketplace comercial para que pueda recibir la información de contacto de un cliente cuando este muestre interés o implemente el producto.

#### <a name="to-configure-the-connection-details-in-partner-center"></a>Configuración de los detalles de la conexión en el Centro de partners

1. En **Clientes potenciales**, seleccione el vínculo **Conectar**.
1. En el cuadro de diálogo **Detalles de conexión**, seleccione un destino de cliente potencial de la lista.
1. Rellene los campos que aparecen. Para más información, consulte los siguientes artículos:

   - [Configuración de la oferta para enviar clientes potenciales a la tabla de Azure](partner-center-portal/commercial-marketplace-lead-management-instructions-azure-table.md#configure-your-offer-to-send-leads-to-the-azure-table)
   - [Configuración de la oferta para enviar clientes potenciales a Dynamics 365 Customer Engagement](partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md#configure-your-offer-to-send-leads-to-dynamics-365-customer-engagement) (antes Dynamics CRM Online)
   - [Configuración de la oferta para enviar clientes potenciales al punto de conexión HTTPS](partner-center-portal/commercial-marketplace-lead-management-instructions-https.md#configure-your-offer-to-send-leads-to-the-https-endpoint)
   - [Configuración de la oferta para enviar clientes potenciales al Marketo](partner-center-portal/commercial-marketplace-lead-management-instructions-marketo.md#configure-your-offer-to-send-leads-to-marketo)
   - [Configurar la oferta para enviar clientes potenciales a Salesforce](partner-center-portal/commercial-marketplace-lead-management-instructions-salesforce.md#configure-your-offer-to-send-leads-to-salesforce)

1. Para validar la configuración proporcionada, seleccione el vínculo **Validar**, si corresponde.
1. Para cerrar el cuadro de diálogo, seleccione **Conectar**.
1. Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña: Propiedades.

> [!NOTE]
> Asegúrese de que la conexión con el destino de clientes potenciales permanece actualizada para que no pierda ningún cliente potencial. Asegúrese de actualizar estas conexiones siempre que haya algún cambio.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de las propiedades de la aplicación de Azure](azure-app-properties.md)
