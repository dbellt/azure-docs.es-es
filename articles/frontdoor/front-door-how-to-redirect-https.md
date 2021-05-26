---
title: Crear una instancia de Front Door con redirección de HTTP a HTTPS mediante Azure Portal
description: Aprenda a crear una instancia de Front Door con el tráfico redirigido de HTTP a HTTPS mediante Azure Portal.
services: front-door
author: duongau
ms.service: frontdoor
ms.topic: how-to
ms.date: 09/30/2020
ms.author: duau
ms.openlocfilehash: e3e2de040c80692824c648ee15f6ae0c97e2786c
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110453324"
---
# <a name="create-a-front-door-with-http-to-https-redirection-using-the-azure-portal"></a>Crear una instancia de Front Door con redirección de HTTP a HTTPS mediante Azure Portal

Puede usar Azure Portal para [crear una instancia de Front Door](quickstart-create-front-door.md) con un certificado para la terminación TLS. Una regla de enrutamiento se usa para redirigir el tráfico de HTTP a HTTPS.

## <a name="create-a-front-door-with-an-existing-web-app-resource"></a>Crear una instancia de Front Door con un recurso de aplicación web existente

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

1. Seleccione **Crear un recurso** en la esquina superior izquierda de Azure Portal.

1. Busque **Front Door** con la barra de búsqueda y, una vez que encuentre el tipo de recurso, seleccione **Crear**.

1. Elija una *suscripción* y luego use un grupo de recursos existente o cree uno nuevo. Seleccione **Siguiente** para ir a la pestaña de configuración.

    > [!NOTE]
    > La ubicación que se pide en la interfaz de usuario es solo para el grupo de recursos. La configuración de Front Door se implementará en todas las [ubicaciones POP de Azure Front Door](front-door-faq.yml#where-are-the-edge-locations-for-azure-front-door-).

    :::image type="content" source="./media/front-door-url-redirect/front-door-create-basics.png" alt-text="Configurar conceptos básicos de una nueva instancia de Front Door":::

1. La configuración de Front Door se realiza en tres pasos: la adición de un host de front-end predeterminado, la adición de servidores back-end en un grupo de back-end y, luego, la creación de reglas de enrutamiento para asignar el comportamiento de enrutamiento para el host de front-end. Seleccione el icono " **+** " en _Hosts de front-end_ para crear un host de front-end.

    :::image type="content" source="./media/front-door-url-redirect/front-door-designer.png" alt-text="Diseñador de configuración de Front Door":::

1. Escriba un nombre único global para el host de front-end predeterminado de la instancia de Front Door. Seleccione **Agregar** para ir al paso siguiente.

    :::image type="content" source="./media/front-door-url-redirect/front-door-create-frontend-host.png" alt-text="Agregar un host de front-end":::

### <a name="create-backend-pool"></a>Creación de un grupo de back-end

1. Seleccione el icono " **+** " en _Grupos de back-end_ para crear un grupo de back-end. Asigne un nombre al grupo de back-end y seleccione **Agregar un back-end**.

    :::image type="content" source="./media/front-door-url-redirect/front-door-designer-backend-pool.png" alt-text="Grupo de back-end del diseñador de configuración de Front Door":::

1. Seleccione el tipo de host de back-end como _App service_. Seleccione la suscripción donde se hospeda la aplicación web y, luego, seleccione la aplicación web específica en la lista desplegable para **Nombre de host de back-end**.

    :::image type="content" source="./media/front-door-url-redirect/front-door-create-backend-pool.png" alt-text="Agregar un back-end en un grupo de back-end":::

1. Seleccione **Agregar** para guardar el back-end y vuelva a seleccionar **Agregar** para guardar la configuración del grupo de back-end. 

## <a name="create-http-to-https-redirect-rule"></a>Creación de una ruta de redireccionamiento de HTTP a HTTPS

1. Seleccione el icono " **+** " en *Reglas de enrutamiento* para crear una ruta. Asigne un nombre a la ruta, por ejemplo, "HttpToHttpsRedirect" y establezca el campo *Protocolo aceptado* en **Solo HTTP**. Asegúrese de seleccionar el elemento adecuado de *Front-ends/dominios*.  

    :::image type="content" source="./media/front-door-url-redirect/front-door-designer-routing-rule.png" alt-text="Regla de enrutamiento del diseñador de configuración de Front Door":::

1. En la sección *Detalles de ruta*, establezca *Tipo de ruta* en **Redireccionamiento**. Asegúrese de que *Tipo de redireccionamiento* se establezca en **Encontrado (302)** y *Protocolo de redireccionamiento* en **Solo HTTPS**. 

    :::image type="content" source="./media/front-door-url-redirect/front-door-redirect-config-example.png" alt-text="Agregar una ruta de redireccionamiento de HTTP a HTTPS":::

1. Seleccione **Agregar** para guardar la regla de enrutamiento del redireccionamiento de HTTP a HTTPS.

## <a name="create-forwarding-rule"></a>Creación de regla de reenvío

1. Agregue otra regla de enrutamiento para controlar el tráfico HTTPS. Seleccione el signo " **+** " en *Reglas de enrutamiento* y asigne un nombre a la ruta, por ejemplo "DefaultForwardingRoute". Luego establezca el campo *Protocolos aceptados* en **Solo HTTPS**. Asegúrese de seleccionar el elemento adecuado de *Front-ends/dominios*.

1. En la sección Detalles de ruta, establezca *Tipo de ruta* en **Reenvío**. Asegúrese de que se seleccione el grupo de back-end correcto y de que *Protocolo de reenvío* se establezca en **Solo HTTPS**. 

    :::image type="content" source="./media/front-door-url-redirect/front-door-forward-route-example.png" alt-text="Agregar una ruta de reenvío para el tráfico HTTPS" border="false":::

1. Seleccione **Agregar** para guardar la regla de enrutamiento para el reenvío de solicitudes.

1. Seleccione **Revisar y crear** y luego **Crear** para crear el perfil de Front Door. Vaya al recurso una vez que lo haya creado.

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).
- Obtenga más información sobre la [redirección de URL en Front Door](front-door-url-redirect.md).
