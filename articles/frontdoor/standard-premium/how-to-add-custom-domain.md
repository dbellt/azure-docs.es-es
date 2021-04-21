---
title: Procedimiento para agregar un dominio personalizado a la configuración de un SKU de Azure Front Door Estándar/Premium
description: En este tutorial, aprenderá a incorporar un dominio personalizado a la SKU de Azure Front Door Estándar/Prémium.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 02/18/2021
ms.author: amsriva
ms.openlocfilehash: 69e216c8893f9361a18354e5d165ecc0499601aa
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107387928"
---
# <a name="create-a-custom-domain-on-azure-front-door-standardpremium-sku-preview-using-the-azure-portal"></a>Creación de un dominio personalizado en la SKU de Azure Front Door Estándar/Prémium (versión preliminar) mediante Azure Portal

> [!Note]
> Esta documentación trata sobre Azure Front Door Estándar/Prémium (versión preliminar). ¿Busca información sobre Azure Front Door? Consulte [aquí](../front-door-overview.md).

Si se usa Azure Front Door Standard/Premium para la entrega de aplicaciones, es necesario un dominio personalizado si desea que su nombre de dominio se vea en la solicitud del usuario final. El hecho de tener un nombre de dominio visible puede ser cómodo para sus clientes y útil con fines de personalización de marca.

Después de crear un perfil de Azure Front Door Estándar/Premium, el host de front-end predeterminado tendrá un subdominio de azurefd.net. Este subdominio se incluye en la dirección URL cuando Azure Front Door Estándar/Premium entrega contenido de su back-end de forma predeterminada. Por ejemplo, `https://contoso-frontend.azurefd.net/activeusers.htm`. Para su comodidad, Azure Front Door permite asociar un dominio personalizado al host predeterminado. Con esta opción, se entrega el contenido con un dominio personalizado en la dirección URL, en lugar de un nombre de dominio propiedad de Azure Front Door Estándar/Premium. Por ejemplo: 'https://www.contoso.com/photo.png'.

> [!IMPORTANT]
> Azure Front Door Estándar/Prémium (versión preliminar) está disponible actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [**Condiciones de uso complementarias de las versiones preliminares de Microsoft Azure**](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos
* Para poder completar los pasos de este tutorial, primero debe crear una instancia de Front Door. Para más información, consulte el inicio rápido de [creación de una instancia de Front Door Estándar/Premium](create-front-door-portal.md).

* Si todavía no tiene un dominio personalizado, primero debe comprar uno a un proveedor de dominios. Por ejemplo, vea [Compra de un nombre de dominio personalizado](../../app-service/manage-custom-dns-buy-domain.md).

* Si usa Azure para hospedar sus [dominios DNS](../../dns/dns-overview.md), debe delegar el sistema de nombres de dominio (DNS) del proveedor de dominios en una instancia de Azure DNS. Para más información, vea [Delegación de un dominio en DNS de Azure](../../dns/dns-delegate-domain-azure-dns.md). De lo contrario, si usa un proveedor de dominios para controlar el dominio DNS, debe validar manualmente el dominio. Para ello, debe escribir los registros TXT de DNS solicitados.

## <a name="add-a-new-custom-domain"></a>Adición de un nuevo dominio personalizado

> [!NOTE]
> En la versión preliminar pública, no se admite el uso de Azure DNS para crear dominios de Apex en Azure Front Door Estándar/Premium. Hay otros proveedores de DNS que admiten el acoplamiento de CNAME o el seguimiento de DNS que permitirán utilizar los dominios APEX para Azure Front Door Estándar/Premium.

Los dominios personalizados se administran en la sección Dominios del portal. Los dominios personalizados se pueden crear y validar antes de la asociación a un punto de conexión. Un dominio personalizado y sus subdominio no se pueden asociar con más de un punto de conexión simultáneamente. Sin embargo, puede utilizar diferentes dominios desde el mismo dominio personalizado para distintas instancias de Front Door. También puede asignar dominios personalizados con diferentes subdominios al mismo punto de conexión de Front Door.

1. En la configuración de su perfil de Azure Front Door, seleccione *Dominios* y, después, haga clic en el botón **Agregar un dominio**.

    :::image type="content" source="../media/how-to-add-custom-domain/add-domain-button.png" alt-text="Captura de pantalla del botón Agregar dominio en la página de aterrizaje del dominio.":::

1. Aparecerá la página **Agregar un dominio**, donde puede especificar información acerca del dominio personalizado. Puede elegir DNS administrado por Azure, que es lo que se recomienda, o bien puede usar su propio proveedor de DNS. Si elige DNS administrado por Azure, seleccione una zona DNS existente y, después, seleccione un subdominio personalizado o cree uno nuevo. Si usa otro proveedor de DNS, escriba manualmente el nombre del dominio personalizado. Seleccione **Agregar** para agregar un dominio personalizado.

    :::image type="content" source="../media/how-to-add-custom-domain/add-domain-page.png" alt-text="Captura de pantalla de la página Agregar un dominio.":::

    Se crea un dominio personalizado con el estado de validación **En envío**.

    :::image type="content" source="../media/how-to-add-custom-domain/validation-state-submitting.png" alt-text="Captura de pantalla del estado de validación de dominio En envío.":::

    Espere hasta que el estado de la validación cambie a **Pendiente**. Esta operación puede tardar unos minutos.

    :::image type="content" source="../media/how-to-add-custom-domain/validation-state-pending.png" alt-text="Captura de pantalla del estado de validación de dominio Pendiente.":::

1. Seleccione el estado de validación **Pendiente**. Aparecerá una nueva página con la información del registro TXT de DNS necesaria para validar el dominio personalizado. El registro TXT tiene la forma `_dnsauth.<your_subdomain>`. Si utiliza una zona basada en Azure DNS, seleccione el botón **Agregar** y se creará un registro TXT con el valor de registro mostrado en la zona Azure DNS. Si usa otro proveedor de DNS, cree manualmente un registro TXT de nombre `dnsauth.<your_subdomain>` con el valor de registro tal como se muestra en la página.

    :::image type="content" source="../media/how-to-add-custom-domain/validate-custom-domain.png" alt-text="Captura de pantalla de la página para validar un dominio personalizado.":::

1. Seleccione el estado de actualización. Una vez que el dominio se valida mediante el registro TXT de DNS, el estado de validación cambiará a **comprobado**. Esta operación puede tardar varios minutos en validarse.

    :::image type="content" source="../media/how-to-add-custom-domain/domain-status-verified.png" alt-text="Captura de pantalla de un dominio personalizado comprobado.":::

1. Cierre la página para volver a la página de aterrizaje de la lista de dominios personalizados. El estado de aprovisionamiento del dominio personalizado debe cambiar a **Aprovisionado** y el estado de validación debe cambiar a **Aprobado**.

    :::image type="content" source="../media/how-to-add-custom-domain/provisioned-approved-status.png" alt-text="Captura de pantalla de los estados Aprovisionado y Aprobado.":::

## <a name="associate-the-custom-domain-with-your-front-door-endpoint"></a>Asociación del dominio personalizado con un punto de conexión de Front Door

Una vez que haya validado un dominio personalizado, puede agregarlo al punto de conexión de Azure Front Door Estándar/Premium.

1. Una vez que se valida el dominio personalizado, se puede asociar a una ruta y un punto de conexión de Azure Front Door Estándar/Premium. Seleccione el vínculo **Endpoint association** (Asociación del punto de conexión) para abrir la página **Associate endpoint and routes** (Asociar punto de conexión y rutas). Seleccione un punto de conexión y las rutas con las que desea establecer una asociación. Luego, seleccione **Asociar**. Cierre la página una vez que se complete la operación de asociación.

    :::image type="content" source="../media/how-to-add-custom-domain/associate-endpoint-routes.png" alt-text="Captura de pantalla de la página Asociar punto de conexión y rutas.":::

    El estado Endpoint association (Asociación del punto de conexión) debe cambiar para reflejar el punto de conexión al que está asociado actualmente el dominio personalizado. 

    :::image type="content" source="../media/how-to-add-custom-domain/endpoint-association-status.png" alt-text="Captura de pantalla del vínculo Asociación del punto de conexión.":::

1. Seleccione el vínculo de estado de DNS.

    :::image type="content" source="../media/how-to-add-custom-domain/dns-state-link.png" alt-text="Captura de pantalla del vínculo de estado de DNS.":::

1. Aparecerá la página **Add or update the CNAME record** (Agregar o actualizar registro CNAME), en la que se mostrará la información del registro CNAME que se debe proporcionar para que el tráfico pueda empezar a fluir. Si usa zonas hospedadas de Azure DNS, podrá crear los registros CNAME seleccionando el botón **Agregar** de la página. Si usa otro proveedor de DNS, debe especificar manualmente el nombre y el valor del registro CNAME como se muestra en la página.

    :::image type="content" source="../media/how-to-add-custom-domain/add-update-cname-record.png" alt-text="Captura de pantalla de Agregar o actualizar registro CNAME.":::

1. Una vez que se cree el registro CNAME y que el dominio personalizado se asocie al punto de conexión de Azure Front Door, el tráfico comenzará a fluir.

    > [!NOTE]
    > Si HTTPS está habilitado, es posible que la propagación y el aprovisionamiento de certificados tarde unos minutos, ya que la propagación se realiza a todas las ubicaciones perimetrales. 

## <a name="verify-the-custom-domain"></a>Comprobación del dominio personalizado

Después de validar y asociar el dominio personalizado, compruebe que este está correctamente referenciado al punto de conexión.

:::image type="content" source="../media/how-to-add-custom-domain/verify-configuration.png" alt-text="Captura de pantalla del dominio personalizado validado y asociado.":::

Por último, compruebe que el contenido de la aplicación se atiende mediante un explorador.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo habilitar HTTPS para el dominio personalizado, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Habilitar HTTPS para un dominio personalizado]()
