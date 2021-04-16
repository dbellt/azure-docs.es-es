---
title: Creación de una oferta de SaaS en el marketplace comercial de Microsoft
description: Aprenda a crear una oferta de software como servicio (SaaS) para publicarla o venderla en Microsoft AppSource, Azure Marketplace o mediante el programa del Proveedor de soluciones en la nube (CSP) mediante el portal del marketplace comercial del Centro de partners de Microsoft.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 03/19/2021
ms.openlocfilehash: 74d30b7c42002c8f134520e0198774eba1519bcd
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106553845"
---
# <a name="how-to-create-a-saas-offer-in-the-commercial-marketplace"></a>Creación de una oferta de SaaS en el marketplace comercial

Como editor del marketplace comercial, puede crear una oferta de software como servicio (SaaS) para que los clientes potenciales puedan comprar su solución técnica basada en SaaS. En este artículo se explica el proceso para crear una oferta de SaaS en el marketplace comercial de Microsoft.

## <a name="before-you-begin"></a>Antes de empezar

Si aún no lo ha hecho, lea la sección [Planeamiento de una oferta de SaaS para el marketplace comercial](plan-saas-offer.md). Ahí se explican los requisitos técnicos de la aplicación SaaS y la información y los recursos que necesitará al crear la oferta. A menos que tenga previsto publicar una oferta sencilla (opción de publicación **Ponerse en contacto conmigo**) en el marketplace comercial, la aplicación SaaS debe cumplir los requisitos técnicos que giran en torno a la autenticación.

> [!IMPORTANT]
> Le recomendamos que cree una oferta de desarrollo y pruebas (DEV) y una oferta de producción (PROD) independientes. En este artículo se describe cómo crear una oferta de desarrollo. Para obtener más información sobre cómo crear una oferta de desarrollo, consulte [Creación de una oferta de desarrollo y pruebas](create-saas-dev-test-offer.md).

## <a name="create-a-new-saas-offer"></a>Creación de una oferta de SaaS

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).
1. En el menú de navegación izquierdo, seleccione **Marketplace comercial** > **Información general**.
1. En la pestaña **Información general**, seleccione **+ Nueva oferta** > **Software como servicio**.

   :::image type="content" source="media/new-offer-saas.png" alt-text="Se muestra el menú de navegación izquierdo y la lista de nuevas ofertas.":::

1. En el cuadro de diálogo **Nueva oferta**, escriba un valor para **Id. de oferta**. Este identificador es visible en la dirección URL de la oferta del marketplace comercial y de las plantillas de Azure Resource Manager, si procede. Por ejemplo, si escribe **test-offer-1** en este cuadro, la dirección web de la oferta será `https://azuremarketplace.microsoft.com/marketplace/../test-offer-1`.
   + Cada oferta de su cuenta debe tener un identificador único.
   + Use solo letras minúsculas y números. Puede incluir guiones y caracteres de subrayado, pero no espacios, y está limitado a 50 caracteres.
   + El identificador de oferta no se puede cambiar después de seleccionar **Crear**.

1. Escriba un **Alias de la oferta**. Este es el nombre que se usa para la oferta en el Centro de partners.

   + No está visible en el marketplace comercial y es diferente del nombre de la oferta y otros valores que se muestran a los clientes.
   + El alias de la oferta no se puede cambiar después de seleccionar **Crear**.
1. Para generar la oferta y continuar, seleccione **Crear**.

## <a name="configure-your-saas-offer-setup-details"></a>Configuración de los detalles de configuración de la oferta de SaaS

En la pestaña **Configuración de la oferta**, en **Detalles de configuración**, elegirá si quiere vender la oferta a través de Microsoft o administrar las transacciones de forma independiente. Las ofertas vendidas a través de Microsoft se conocen como _ofertas procesables_, lo que significa que Microsoft facilita el intercambio de dinero por una licencia de software en nombre del editor. Para más información sobre estas opciones, consulte [Opciones de publicación](plan-saas-offer.md#listing-options) y [Determinación de la opción de publicación](determine-your-listing-type.md).

1. Para vender a través de Microsoft y que nosotros le facilitemos las transacciones, seleccione **Sí**. Continúe con [Habilitar una versión de prueba](#enable-a-test-drive-optional).

1. Para publicar la oferta a través del marketplace comercial y procesar las transacciones de forma independiente, seleccione **No** y, luego, realice una de las acciones siguientes:
   + Para proporcionar una suscripción gratuita para su oferta, seleccione **Obtener ahora (gratis)** . Luego, en el cuadro **Dirección URL de la oferta** que aparece, escriba la dirección URL (comenzando con *http* o *https*) donde los clientes pueden obtener una evaluación gratuita mediante la [autenticación con un clic con Azure Active Directory (Azure AD)](azure-ad-saas.md). Por ejemplo, `https://contoso.com/saas-app`.
   + Para proporcionar una evaluación gratuita de 30 días, seleccione **Evaluación gratuita** y, luego, en el cuadro **Dirección URL de la prueba** que aparece, escriba la dirección URL (comenzando con *http* o *https*) donde los clientes pueden acceder a la evaluación gratuita mediante la [autenticación con un clic con Azure Active Directory (Azure AD)](azure-ad-saas.md). Por ejemplo, `https://contoso.com/trial/saas-app`.
   + Para que los clientes potenciales se pongan en contacto con usted para adquirir su oferta, seleccione **Ponerse en contacto conmigo**.

## <a name="enable-a-test-drive-optional"></a>Habilitación de una versión de prueba (opcional)

Una versión de prueba es una excelente manera de exhibir la oferta a posibles clientes al concederles acceso a un entorno preconfigurado durante un número fijo de horas. La oferta de una versión de prueba tiene como resultado una mayor tasa de conversión y genera clientes potenciales muy cualificados. Para más información sobre las versiones de prueba, consulte [¿Qué es una versión de prueba?](./what-is-test-drive.md).

> [!TIP]
> Una versión de prueba es diferente de una evaluación gratuita. Puede ofrecer una versión de prueba, una evaluación gratuita o ambas. Las dos proporcionan a los clientes la solución durante un período fijo. Sin embargo, una versión de prueba también incluye un práctico recorrido autoguiado por las principales características y ventajas del producto demostradas en un escenario de implementación real.

**Habilitación de una versión de prueba**
1.  En **Versión de prueba**, active la casilla **Habilitar una versión de prueba**.
1.  En la lista que aparece, seleccione el tipo de versión de prueba.

## <a name="configure-lead-management"></a>Configuración de la administración de clientes potenciales

Conecte el sistema de administración de relaciones con clientes (CRM) con la oferta del marketplace comercial para que pueda recibir la información de contacto de un cliente cuando este muestre interés o implemente el producto. Esta conexión se puede modificar en cualquier momento durante la creación de la oferta o con posterioridad a esta.

> [!NOTE]
> Si va a vender la oferta a través de Microsoft o si ha seleccionado la opción de publicación **Ponerse en contacto conmigo**, debe configurar la administración de clientes potenciales. Para obtener instrucciones detalladas, vea [Clientes potenciales a partir de la oferta en el marketplace comercial](partner-center-portal/commercial-marketplace-get-customer-leads.md).

### <a name="configure-the-connection-details-in-partner-center"></a>Configuración de los detalles de la conexión en el Centro de partners

1.  En **Clientes potenciales**, seleccione el vínculo **Conectar**.
1. En el cuadro de diálogo **Detalles de conexión**, seleccione un destino de cliente potencial de la lista.
1. Rellene los campos que aparecen. Para más información, consulte los siguientes artículos:

   - [Configuración de la oferta para enviar clientes potenciales a la tabla de Azure](./partner-center-portal/commercial-marketplace-lead-management-instructions-azure-table.md#configure-your-offer-to-send-leads-to-the-azure-table)
   - [Configuración de la oferta para enviar clientes potenciales a Dynamics 365 Customer Engagement](./partner-center-portal/commercial-marketplace-lead-management-instructions-dynamics.md#configure-your-offer-to-send-leads-to-dynamics-365-customer-engagement) (antes Dynamics CRM Online)
   - [Configuración de la oferta para enviar clientes potenciales al punto de conexión HTTPS](./partner-center-portal/commercial-marketplace-lead-management-instructions-https.md#configure-your-offer-to-send-leads-to-the-https-endpoint)
   - [Configuración de la oferta para enviar clientes potenciales al Marketo](./partner-center-portal/commercial-marketplace-lead-management-instructions-marketo.md#configure-your-offer-to-send-leads-to-marketo)
   - [Configurar la oferta para enviar clientes potenciales a Salesforce](./partner-center-portal/commercial-marketplace-lead-management-instructions-salesforce.md#configure-your-offer-to-send-leads-to-salesforce)

1. Para validar la configuración proporcionada, seleccione el vínculo **Validar**.
1. Para cerrar el cuadro de diálogo, seleccione **Aceptar**.

## <a name="configure-microsoft-365-app-integration"></a>Configuración de la integración de aplicaciones de Microsoft 365

Puede aclarar la [detección y entrega unificadas](./plan-SaaS-offer.md) de la oferta de SaaS y cualquier consumo de aplicaciones de Microsoft 365 relacionado mediante su vinculación.

### <a name="integrate-with-microsoft-api"></a>Integración con Microsoft API

1. Si la oferta de SaaS no se integra con Microsoft Graph API, seleccione **No**. Continúe para vincular los clientes de consumo de aplicaciones de Microsoft 365 publicados.  

1. Si la oferta de SaaS se integra con Microsoft Graph API, seleccione **Sí** y, a continuación, proporcione el identificador de aplicación de Azure Active Directory que ha creado y registrado para integrarlo con Microsoft Graph API. 

### <a name="link-published-microsoft-365-app-consumption-clients"></a>Vinculación de los clientes de consumo de aplicaciones de Microsoft 365 publicados

1. Si no ha publicado el complemento de Office, la aplicación de Teams o las soluciones de SharePoint Framework que funcionan con la oferta de SaaS, seleccione **No**.

1. Si ha publicado el complemento de Office, la aplicación de Teams o las soluciones de SharePoint Framework que funcionan con la oferta de SaaS, seleccione **Sí** y, luego, seleccione **+Agregar otro vínculo de AppSource** para agregar nuevos vínculos.  

1. Proporcione un vínculo de AppSource válido.

1. Continúe con la adición de todos los vínculos mediante la selección de **+Agregar otro vínculo de AppSource** y proporcione vínculos de AppSource válidos.  

1. El orden en que se muestran los productos vinculados en la página de lista de la oferta de SaaS se indica mediante el valor de clasificación. Para cambiarlo, seleccione, mantenga presionado y mueva el icono de = hacia arriba y hacia abajo en la lista. 

1. Para eliminar un producto vinculado, seleccione **Eliminar** en la fila del producto.  


> [!IMPORTANT]
> Si deja de vender un producto vinculado, no se desvinculará automáticamente en la oferta de SaaS, debe eliminarlo de la lista de productos vinculados y volver a enviar la oferta de SaaS.  

 

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de las propiedades de la oferta de SaaS](create-new-saas-offer-properties.md)