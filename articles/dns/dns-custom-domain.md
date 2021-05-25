---
title: 'Integración de Azure DNS con los recursos de Azure: Azure DNS'
description: En este artículo, aprenderá a usar Azure DNS para proporcionar servicios de DNS para los recursos de Azure.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 05/06/2021
ms.author: rohink
ms.openlocfilehash: c09c1840c9976a2ac14cb5ef73f8c4e15000c54f
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109635107"
---
# <a name="use-azure-dns-to-provide-custom-domain-settings-for-an-azure-service"></a>Usar Azure DNS para proporcionar la configuración de un dominio personalizado para un servicio de Azure

Azure DNS proporciona resolución de nombres para cualquiera de los recursos de Azure que admiten dominios personalizados o que tienen un nombre de dominio completo (FQDN). Por ejemplo, tiene una aplicación web de Azure y quiere que los usuarios puedan acceder a ella mediante `contoso.com` o `www.contoso.com` como FQDN. Este artículo le guiará a través de la configuración del servicio de Azure con Azure DNS para usar dominios personalizados.

## <a name="prerequisites"></a>Prerrequisitos

Para poder usar Azure DNS con dominio personalizado, primero debe delegarlo a Azure DNS. Consulte [Delegación de un dominio en Azure DNS](./dns-delegate-domain-azure-dns.md) para obtener instrucciones sobre cómo configurar los servidores de nombres para su delegación. Una vez que el dominio se haya delegado a la zona de Azure DNS, es posible configurar los registros de DNS necesarios.

Puede configurar un dominio personal o personalizado para Azure Function Apps, direcciones IP públicas, App Service (Web Apps), Blob Storage y Azure CDN.

## <a name="azure-function-app"></a>Aplicación de función de Azure

Para configurar un dominio personalizado para las aplicaciones de Azure Functions, se crea un registro CNAME y se configura en la propia aplicación de funciones.
 
1. Vaya a **Function App** y seleccione la aplicación de función. Seleccione **Dominios personalizados** en *Configuración*. Observe la **dirección URL actual** en *Dominios personalizados asignados*. Esta dirección se usa como alias para el registro DNS que se creó.

    :::image type="content" source="./media/dns-custom-domain/function-app.png" alt-text="Captura de pantalla de dominios personalizados para la aplicación de funciones.":::

1. Vaya a la zona DNS y seleccione **+ Conjunto de registros**. Escriba la información siguiente en la página **Agregar conjunto de registros** y seleccione **Aceptar** para crearlo.

    :::image type="content" source="./media/dns-custom-domain/function-app-record.png" alt-text="Captura de pantalla de la página Agregar conjunto de registros de la aplicación de funciones.":::

    |Propiedad  |Value  |Descripción  |
    |---------|---------|---------|
    | Nombre | myfunctionapp | Este valor junto con la etiqueta de nombre de dominio es el FQDN del nombre de dominio personalizado. |
    | Tipo | CNAME | Use un registro CNAME que use un alias. |
    | TTL | 1 | 1 se usa para 1 hora  |
    | Unidad de TTL | Horas | Las horas se utilizan como medida de tiempo  |
    | Alias | contosofunction.azurewebsites.net | El nombre DNS para el que va a crear el alias, en este ejemplo es el nombre DNS contosofunction.azurewebsites.net proporcionado de manera predeterminada para la aplicación de funciones.        |
    
1. Vuelva a la aplicación de funciones y seleccione **Dominios personalizados** en *Configuración*. Luego seleccione **+ Agregar dominio personalizado**.

    :::image type="content" source="./media/dns-custom-domain/function-app-add-domain.png" alt-text="Captura de pantalla del botón Agregar dominio personalizado para la aplicación de funciones.":::    

1. En la página **Agregar dominio personalizado**, escriba el registro CNAME en el campo de texto **Dominio personalizado** y seleccione **Validar**. Si se encuentra el registro, aparece el botón **Agregar dominio personalizado**. Seleccione **Agregar dominio personalizado** para agregar el alias.

    :::image type="content" source="./media/dns-custom-domain/function-app-cname.png" alt-text="Captura de pantalla de la página Agregar dominio personalizado para la aplicación de funciones.":::

## <a name="public-ip-address"></a>Dirección IP pública

Para configurar un dominio personalizado para servicios que usan una dirección IP pública como, por ejemplo, Application Gateway, Load Balancer, Cloud Service, VM de Resource Manager y VM clásicas, se usa un registro A.

1. Vaya al recurso de dirección IP pública y seleccione **Configuración**. Anote la dirección IP que aparece.

    :::image type="content" source="./media/dns-custom-domain/public-ip.png" alt-text="Captura de pantalla de la página de configuración de IP pública.":::

1. Vaya a la zona DNS y seleccione **+ Conjunto de registros**. Escriba la información siguiente en la página **Agregar conjunto de registros** y seleccione **Aceptar** para crearlo.

    :::image type="content" source="./media/dns-custom-domain/public-ip-record.png" alt-text="Captura de pantalla de la página del conjunto de registros de IP pública.":::

    | Propiedad | Value | Descripción |
    | -------- | ----- | ------------|
    | Nombre | webserver1 | Este valor junto con la etiqueta de nombre de dominio es el FQDN del nombre de dominio personalizado. |
    | Tipo | Un | Use un registro A como recurso en una dirección IP. |
    | TTL | 1 | 1 se usa para 1 hora |
    | Unidad de TTL | Horas | Las horas se utilizan como medida de tiempo |
    | Dirección IP | `<your ip address>` | La dirección IP pública. |

1. Una vez creado el registro A, ejecute `nslookup` para comprobar que el registro se resuelve.

    :::image type="content" source="./media/dns-custom-domain/public-ip-nslookup.png" alt-text="Captura de pantalla de nslookup en cmd para la dirección IP pública.":::

## <a name="app-service-web-apps"></a>App Service (Web Apps)

Los pasos siguientes le guiarán en el proceso de configuración de un dominio personalizado para una instancia de App Service Web Apps.

1. Vaya a **App Service**, seleccione el recurso para el que está configurando un nombre de dominio personalizado, y seleccione **Dominios personalizados** en *Configuración*. Observe la **dirección URL actual** en *Dominios personalizados asignados*. Esta dirección se usa como alias para el registro DNS que se creó.

    :::image type="content" source="./media/dns-custom-domain/web-app.png" alt-text="Captura de pantalla de dominios personalizados para la aplicación web.":::

1. Vaya a la zona DNS y seleccione **+ Conjunto de registros**. Escriba la información siguiente en la página **Agregar conjunto de registros** y seleccione **Aceptar** para crearlo.

    :::image type="content" source="./media/dns-custom-domain/web-app.png" alt-text="Captura de pantalla de la página del conjunto de registros de la aplicación web.":::

    | Propiedad  | Value | Descripción |
    |---------- | ----- | ----------- |
    | Nombre | mywebserver | Este valor junto con la etiqueta de nombre de dominio es el FQDN del nombre de dominio personalizado. |
    | Tipo | CNAME | Use un registro CNAME que use un alias. Si el recurso usa una dirección IP, se usará un registro A. |
    | TTL | 1 | 1 se usa para 1 hora |
    | Unidad de TTL | Horas | Las horas se utilizan como medida de tiempo |
    | Alias | contoso.azurewebsites.net | El nombre DNS para el que va a crear el alias. En este ejemplo es el nombre DNS contoso.azurewebsites.net proporcionado de manera predeterminada para la aplicación web. |

1. Vuelva a la aplicación web y seleccione **Dominios personalizados** en *Configuración*. Luego seleccione **+ Agregar dominio personalizado**.

    :::image type="content" source="./media/dns-custom-domain/web-app-add-domain.png" alt-text="Captura de pantalla del botón Agregar dominio personalizado para la aplicación web.":::  

1. En la página **Agregar dominio personalizado**, escriba el registro CNAME en el campo de texto **Dominio personalizado** y seleccione **Validar**. Si se encuentra el registro, aparece el botón **Agregar dominio personalizado**. Seleccione **Agregar dominio personalizado** para agregar el alias.

    :::image type="content" source="./media/dns-custom-domain/web-app-cname.png" alt-text="Captura de pantalla de la página Agregar dominio personalizado para la aplicación web.":::

1. Una vez completado el proceso, ejecute **nslookup** para comprobar que la resolución de nombres funciona.

    :::image type="content" source="./media/dns-custom-domain/app-service-nslookup.png" alt-text="Captura de pantalla de nslookup para la aplicación web."::: 

Para más información acerca de cómo asignar un dominio personalizado a App Service, visite [Asignación de un nombre DNS personalizado existente a Azure Web Apps](../app-service/app-service-web-tutorial-custom-domain.md?toc=%dns%2ftoc.json).

Para información sobre cómo migrar un nombre de DNS activo, consulte [Migración de un nombre de DNS activo a Azure App Service](../app-service/manage-custom-dns-migrate-domain.md).

Si tiene que comprar un dominio personalizado para su instancia de App Service, consulte [Comprar un nombre de dominio personalizado para Azure Web Apps](../app-service/manage-custom-dns-buy-domain.md).

## <a name="blob-storage"></a>Blob Storage

Los siguientes pasos le guiarán en el proceso de configuración de un registro CNAME para una cuenta de Blob Storage mediante el método asverify. Este método asegura que no haya ningún tiempo de inactividad.

1. Vaya a **Cuentas de almacenamiento**, seleccione la cuenta de almacenamiento y seleccione **Redes** en *Configuración*. A continuación, seleccione la pestaña **Dominio personalizado**. Tenga en cuenta el FQDN del paso 2, este nombre se usa para crear el primer registro CNAME.

    :::image type="content" source="./media/dns-custom-domain/blob-storage.png" alt-text="Captura de pantalla de los dominios personalizados para la cuenta de almacenamiento.":::

1. Vaya a la zona DNS y seleccione **+ Conjunto de registros**. Escriba la información siguiente en la página **Agregar conjunto de registros** y seleccione **Aceptar** para crearlo.

    :::image type="content" source="./media/dns-custom-domain/storage-account-record.png" alt-text="Captura de pantalla de la página del conjunto de registros de la cuenta de almacenamiento.":::

    | Propiedad | Value | Descripción |
    | -------- | ----- | ----------- |
    | Nombre | asverify.mystorageaccount | Este valor junto con la etiqueta de nombre de dominio es el FQDN del nombre de dominio personalizado. |
    | Tipo | CNAME | Use un registro CNAME que use un alias. |
    | TTL | 1 | 1 se usa para 1 hora |
    | Unidad de TTL | Horas | Las horas se utilizan como medida de tiempo |
    | Alias | asverify.contoso.blob.core.windows.net | El nombre DNS para el que va a crear el alias. En este ejemplo es el nombre DNS asverify.contoso.blob.core.windows.net proporcionado de manera predeterminada para la cuenta de almacenamiento. |

1. Vuelva a la cuenta de almacenamiento, seleccione **Redes** y, a continuación, la pestaña **Dominio personalizado**. Escriba en el cuadro de texto el alias que creó, sin el prefijo asverify, active **Usar validación CNAME indirecta** y seleccione **Guardar**. 

    :::image type="content" source="./media/dns-custom-domain/blob-storage-add-domain.png" alt-text="Captura de pantalla de la página Agregar dominio personalizado de la cuenta de almacenamiento.":::

1. Vuelva a la zona DNS y cree un registro CNAME sin el prefijo asverify.  Después de ese momento, ya es seguro eliminar el registro CNAME con el prefijo asverify.

    :::image type="content" source="./media/dns-custom-domain/storage-account-record-set.png" alt-text="Captura de pantalla del registro de la cuenta de almacenamiento sin el prefijo asverify.":::

1. Para validar la resolución de DNS, ejecute `nslookup`.

Para más información acerca de la asignación de un dominio personalizado a un punto de conexión de Blob Storage visite [Configurar un nombre de dominio personalizado para el punto de conexión de Blob Storage](../storage/blobs/storage-custom-domain-name.md?toc=%dns%2ftoc.json)

## <a name="azure-cdn"></a>Azure CDN

Los siguientes pasos le guiarán en el proceso de configuración de un registro CNAME para un punto de conexión de CDN mediante el método cdnverify. Este método asegura que no haya ningún tiempo de inactividad.

1. Vaya al perfil de CDN y seleccione el punto de conexión con el que está trabajando. Seleccione **+ Dominio personalizado**. Observe el **nombre de host del punto de conexión** ya que este valor es el registro al que señala el registro CNAME.

    :::image type="content" source="./media/dns-custom-domain/cdn.png" alt-text="Captura de pantalla de la página Dominio personalizado de CDN.":::

1. Vaya a la zona DNS y seleccione **+ Conjunto de registros**. Escriba la información siguiente en la página **Agregar conjunto de registros** y seleccione **Aceptar** para crearlo.

    :::image type="content" source="./media/dns-custom-domain/cdn-record.png" alt-text="Captura de pantalla de la página Conjunto de registros de CDN.":::

    | Propiedad | Value | Descripción |
    | -------- | ----- | ----------- |
    | Nombre | cdnverify.mycdnendpoint | Este valor junto con la etiqueta de nombre de dominio es el FQDN del nombre de dominio personalizado. |
    | Tipo | CNAME | Use un registro CNAME que use un alias. |
    | TTL | 1 | 1 se usa para 1 hora |
    | Unidad de TTL | Horas | Las horas se utilizan como medida de tiempo |
    | Alias | cdnverify.contoso.azureedge.net | El nombre DNS para el que va a crear el alias. En este ejemplo es el nombre DNS cdnverify.contoso.azureedge.net proporcionado de manera predeterminada para el punto de conexión de CDN. |

1. Vuelva al punto de conexión de CDN y seleccione **+ Dominio personalizado**. Escriba el alias del registro CNAME sin el prefijo cdnverify y seleccione **Agregar**.

    :::image type="content" source="./media/dns-custom-domain/cdn-add.png" alt-text="Captura de pantalla de la página Agregar un dominio personalizado para un punto de conexión de CDN.":::

1. Vuelva a la zona DNS y cree un registro CNAME sin el prefijo cdnverify.  Después de ese momento, ya es seguro eliminar el registro CNAME con el prefijo cdnverify.

    :::image type="content" source="./media/dns-custom-domain/cdn-record-set.png" alt-text="Captura de pantalla del registro de CDN sin el prefijo cdnverify.":::

Para más información sobre CDN y los procedimientos para configurar un dominio personalizado sin el paso de registro intermedio, visite [Asignación del contenido de Azure CDN a un dominio personalizado](../cdn/cdn-map-content-to-custom-domain.md?toc=%dns%2ftoc.json).

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [configurar búsquedas inversas de DNS para servicios hospedados en Azure](dns-reverse-dns-for-azure-services.md).
