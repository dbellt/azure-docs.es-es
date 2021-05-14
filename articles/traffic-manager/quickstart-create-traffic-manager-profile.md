---
title: 'Inicio rápido: Creación de un perfil de alta disponibilidad de aplicaciones de Azure Traffic Manager en Azure Portal'
description: En este inicio rápido se describe cómo crear un perfil de Traffic Manager para crear una aplicación web de alta disponibilidad mediante Azure Portal.
services: traffic-manager
author: duongau
ms.author: duau
manager: twooley
ms.date: 04/19/2021
ms.topic: quickstart
ms.service: traffic-manager
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.custom:
- mode-portal
ms.openlocfilehash: 13b5925310c615461424f78d90ba9849c9bf58c5
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107727985"
---
# <a name="quickstart-create-a-traffic-manager-profile-using-the-azure-portal"></a>Inicio rápido: Creación de un perfil de Traffic Manager mediante Azure Portal

En esta guía de inicio rápido se describe cómo crear un perfil de Traffic Manager que ofrece alta disponibilidad para la aplicación web.

En esta guía de inicio rápido, obtendrá información sobre dos instancias de una aplicación web. Cada una de ellas se ejecuta en una región de Azure distinta. Creará un perfil de Traffic Manager según la [prioridad del punto de conexión](traffic-manager-routing-methods.md#priority-traffic-routing-method). El perfil dirige el tráfico de usuario al sitio principal que ejecuta la aplicación web. Traffic Manager supervisa continuamente la aplicación web. Si el sitio principal no está disponible, proporciona la conmutación automática por error al sitio de copia de seguridad.

:::image type="content" source="./media/quickstart-create-traffic-manager-profile/environment-diagram.png" alt-text="Diagrama del entorno de implementación de Traffic Manager" border="false":::

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) ahora.

## <a name="prerequisites"></a>Prerrequisitos

Esta guía de inicio rápido requiere que haya implementado dos instancias de una aplicación web en dos regiones de Azure distintas (*Este de EE. UU.* y *Oeste de Europa*). Cada una de ellas servirá como los puntos de conexión principal y de conmutación por error de Traffic Manager.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso**. Busque **Web App** y seleccione **Crear**.

1. En **Crear una aplicación web**, escriba o seleccione los valores siguientes en la pestaña **Aspectos básicos**:

    | Parámetro                 | Value |
    | ---                     | --- |
    | Suscripción            | Seleccione su suscripción. |    
    | Resource group          | Seleccione **Crear** y escriba *myResourceGroupTM1* en el cuadro de texto.|
    | Nombre                    | Escriba un **Nombre** único para la aplicación web. En este ejemplo se usa *myWebAppEastUS*. |
    | Publicar                 | Seleccione **Código**. |
    | Pila en tiempo de ejecución           | Seleccione **ASP.NET V4.7**. |
    | Sistema operativo        | Seleccione **Windows**. |
    | Region                  | Seleccione **Este de EE. UU**. |
    | Plan de Windows            | Seleccione **Crear** y escriba *myAppServicePlanEastUS* en el cuadro de texto. |
    | SKU y tamaño            | Seleccione **Standard S1 100 total ACU, 1.75-GB memory** (Estándar S1 100 ACU total, 1,75 GB de memoria). |
   
1. Seleccione la pestaña **Supervisión** o **Siguiente: Supervisión**.  En **Supervisión**, establezca **Application Insights** > **Habilitar Application Insights** en **No**.

1. Seleccione **Revisar y crear**.

1. Revise la configuración y, a continuación, seleccione **Crear**.  Cuando la aplicación web se implementa correctamente, crea un sitio web predeterminado.

1. Siga los pasos del 1 al 6 para crear una segunda aplicación web denominada *myWebAppWestEurope*. El nombre del **grupo de recursos** es *myResourceGroupTM2*, la **región** es *Oeste de Europa* y el **plan de App Service** es **myAppServicePlanWestEurope**. Todos los demás valores son los mismos que para *myWebAppEastUS*.

## <a name="create-a-traffic-manager-profile"></a>Crear un perfil de Traffic Manager

Cree un perfil de Traffic Manager que dirija el tráfico de los usuarios según la prioridad del punto de conexión.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso**. A continuación, busque **Perfil de Traffic Manager** y seleccione **Crear**.
1. En **Crear perfil de Traffic Manager**, escriba o seleccione estos valores:

    | Parámetro | Value |
    | --------| ----- |
    | Nombre | Escriba un nombre único para el perfil de Traffic Manager.|
    | Método de enrutamiento | Seleccione **Prioridad**.|
    | Subscription | Seleccione la suscripción en la que desea aplicar el perfil de Traffic Manager. |
    | Resource group | Seleccione *myResourceGroupTM1*.|
    | Location |Este valor hace referencia a la ubicación del grupo de recursos. No tiene ningún efecto sobre el perfil de Traffic Manager que se implementará globalmente.|

1. Seleccione **Crear**.

## <a name="add-traffic-manager-endpoints"></a>Incorporación de puntos de conexión de Traffic Manager

Agregue el sitio web en la región *Este de EE. UU.* como punto de conexión principal para enrutar todo el tráfico de usuario. Agregue el sitio web de *Oeste de Europa* como un punto de conexión de conmutación por error. Cuando el punto de conexión principal no está disponible, el tráfico se enruta automáticamente al punto de conexión de conmutación por error.

1. En la barra de búsqueda del portal, escriba el nombre del perfil de Traffic Manager que creó en la sección anterior.
1. Seleccione el perfil en los resultados de la búsqueda.
1. En **Perfil de Traffic Manager**, en la sección **Configuración**, seleccione **Puntos de conexión** y, a continuación, seleccione **Agregar**.

    :::image type="content" source="./media/quickstart-create-traffic-manager-profile/traffic-manager-endpoint-menu.png" alt-text="Captura de pantalla de la configuración de los puntos de conexión del perfil de Traffic Manager":::

1. Escriba o seleccione estas opciones:

    | Parámetro | Value |
    | ------- | ------|
    | Tipo | Seleccione **Punto de conexión de Azure**. |
    | Nombre | Escriba *myPrimaryEndpoint*. |
    | Tipo de recurso de destino | Seleccione **App Service**. |
    | Recurso de destino | Seleccione **Elegir un servicio de aplicaciones** > **Este de EE. UU.** |
    | Priority | Seleccione **1**. Todo el tráfico se dirige a este punto de conexión cuando está en buen estado. |

    :::image type="content" source="./media/quickstart-create-traffic-manager-profile/add-traffic-manager-endpoint.png" alt-text="Captura de pantalla de la adición de un punto de conexión al perfil de Traffic Manager.":::
    
1. Seleccione **Aceptar**.
1. Para crear un punto de conexión de conmutación por error para la segunda región de Azure, repita los pasos 3 y 4 con esta configuración:

    | Parámetro | Value |
    | ------- | ------|
    | Tipo | Seleccione **Punto de conexión de Azure**. |
    | Nombre | Escriba *myFailoverEndpoint*. |
    | Tipo de recurso de destino | Seleccione **App Service**. |
    | Recurso de destino | Seleccione **Elegir un servicio de aplicaciones** > **Oeste de Europa**. |
    | Priority | Seleccione **2**. Todo el tráfico se dirige a este punto de conexión de conmutación por error si el punto de conexión principal no está en estado correcto. |

1. Seleccione **Aceptar**.

Cuando haya terminado la adición de los dos puntos de conexión, estos se muestran en **Perfil de Traffic Manager**. Observe que el estado de supervisión ahora es **En línea**.

## <a name="test-traffic-manager-profile"></a>Prueba del perfil de Traffic Manager

En esta sección, comprobará el nombre de dominio del perfil de Traffic Manager. También configurará el punto de conexión principal para que no esté disponible. Por último, podrá ver que la aplicación web sigue estando disponible. Se debe a que Traffic Manager envía el tráfico al punto de conexión de conmutación por error.

### <a name="check-the-dns-name"></a>Comprobación del nombre DNS

1. En la barra de búsqueda del portal, busque el nombre del **perfil de Traffic Manager** que creó en la sección anterior.
1. Seleccione el perfil de Traffic Manager. Aparece la página **Información general**.
1. La hoja **Perfil de Traffic Manager** muestra el nombre DNS del perfil de Traffic Manager que acaba de crear.
  
    :::image type="content" source="./media/quickstart-create-traffic-manager-profile/traffic-manager-dns-name.png" alt-text="Captura de pantalla de la ubicación del nombre DNS de Traffic Manager.":::

### <a name="view-traffic-manager-in-action"></a>Ver a Traffic Manager en acción

1. En un explorador web, escriba el nombre DNS del perfil de Traffic Manager para ver el sitio web predeterminado de la aplicación web.

    > [!NOTE]
    > En el escenario de esta guía de inicio rápido, todas las solicitudes se enrutan al punto de conexión principal. Se ha establecido en **Prioridad 1**.

    :::image type="content" source="./media/quickstart-create-traffic-manager-profile/traffic-manager-test.png" alt-text="Captura de pantalla de la página web para confirmar la disponibilidad del perfil de Traffic Manager":::

1. Para ver la conmutación por error de Traffic Manager en acción, deshabilite el sitio principal:
    1. En la página Perfil de Traffic Manager, en la sección **Información general**, seleccione **myPrimaryEndpoint**.
    1. En *myPrimaryEndpoint*, seleccione **Deshabilitado** > **Guardar**.
    1. Cierre **myPrimaryEndpoint**. Observe que el estado ahora es **Deshabilitado**.
1. Copie el nombre DNS del perfil de Traffic Manager del paso anterior para ver el sitio web en una nueva sesión del explorador web.
1. Compruebe que la aplicación web sigue estando disponible.

El punto de conexión principal no está disponible, por lo que se ha enrutado al punto de conexión de conmutación por error.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, elimine los grupos de recursos, las aplicaciones web y todos los recursos relacionados. Para ello, seleccione cada elemento individual en el panel y seleccione **Eliminar** en la parte superior de cada página.

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado un perfil de Traffic Manager. Le permite dirigir el tráfico de usuario para aplicaciones web de alta disponibilidad. Para más información sobre el enrutamiento del tráfico, continúe con los tutoriales de Traffic Manager.

> [!div class="nextstepaction"]
> [Tutoriales de Traffic Manager](tutorial-traffic-manager-improve-website-response.md)
