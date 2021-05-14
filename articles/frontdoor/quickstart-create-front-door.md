---
title: 'Inicio rápido: Configuración de alta disponibilidad con Azure Front Door Service: Azure Portal'
description: En este inicio rápido se muestra cómo usar Azure Front Door Service para una aplicación web global de alta disponibilidad y alto rendimiento mediante Azure Portal.
services: front-door
documentationcenter: na
author: duongau
ms.author: duau
manager: KumudD
ms.date: 04/19/2021
ms.topic: quickstart
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.custom:
- mode-portal
ms.openlocfilehash: 74b7c16904c86751076d40056027999fe44cb868
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107727874"
---
# <a name="quickstart-create-a-front-door-for-a-highly-available-global-web-application"></a>Inicio rápido: Cree una instancia de Front Door para una aplicación web global de alta disponibilidad.

Empiece a trabajar con Azure Front Door mediante Azure Portal para configurar alta disponibilidad para una aplicación web.

En este inicio rápido, Azure Front Door agrupa dos instancias de una aplicación web que se ejecutan en regiones diferentes de Azure. Cree una configuración de Front Door basada en back-ends con la misma ponderación y prioridad. Esta configuración dirige el tráfico al sitio más cercano que ejecuta la aplicación. Azure Front Door supervisa continuamente la aplicación web. El servicio proporciona conmutación automática por error al siguiente sitio disponible si el sitio más cercano no lo está.

:::image type="content" source="media/quickstart-create-front-door/environment-diagram.png" alt-text="Diagrama del entorno de implementación de Front Door mediante el uso de Azure Portal." border="false":::

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-two-instances-of-a-web-app"></a>Creación de dos instancias de una aplicación web

Este inicio rápido requiere dos instancias de una aplicación web que se ejecuten en regiones diferentes de Azure. Ambas instancias de la aplicación web se ejecutan en modo *activo/activo*, por lo que cualquiera de ellas puede asumir el tráfico. Esta configuración difiere de una configuración *activa/en espera*, en la que una realiza la conmutación por error.

Si aún no tiene una aplicación web, use los pasos siguientes para configurar aplicaciones web de ejemplo.

1. Inicie sesión en Azure Portal en https://portal.azure.com.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** >  **WebApp**.

    :::image type="content" source="media/quickstart-create-front-door/front-door-create-web-app.png" alt-text="Creación de una aplicación web en Azure Portal":::

1. En la pestaña **Aspectos básicos** de la página **Crear aplicación web**, escriba o seleccione la información siguiente.

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | **Suscripción**               | Seleccione su suscripción. |    
    | **Grupos de recursos**       | Seleccione **Crear nuevo** y escriba *FrontDoorQS_rg1* en el cuadro de texto.|
    | **Nombre**                   | Escriba un **Nombre** único para la aplicación web. En este ejemplo se usa *WebAppContoso-1*. |
    | **Publicar** | Seleccione **Código**. |
    | **Pila del entorno en tiempo de ejecución**         | Seleccione **.NET Core 2.1 (LTS)** . |
    | **Sistema operativo**          | Seleccione **Windows**. |
    | **Región**           | Seleccione **Centro de EE. UU.** |
    | **Plan de Windows** | Seleccione **Crear nuevo** y escriba *myAppServicePlanCentralUS* en el cuadro de texto. |
    | **SKU y tamaño** | Seleccione **Estándar S1 100 ACU total, 1,75 GB de memoria**. |

1. Seleccione **Revisar y crear**, revise el **resumen** y, después, seleccione **Crear**. La implementación puede tardar varios minutos en completarse.

    :::image type="content" source="media/quickstart-create-front-door/create-web-app.png" alt-text="Revisar el resumen de la aplicación web":::

Una vez finalizada la implementación, cree una segunda aplicación web. Utilice el mismo procedimiento con los mismos valores, excepto los siguientes:

| Configuración          | Value     |
| ---              | ---  |
| **Grupos de recursos**   | Seleccione **Crear nuevo** y escriba *FrontDoorQS_rg2*. |
| **Nombre**             | Escriba un nombre único para la aplicación web, en este ejemplo, *WebAppContoso-2*  |
| **Región**           | Una región distinta, en este caso, *Este de EE. UU.* |
| **Plan de App Service** > **Plan de Windows**         | Seleccione **Nuevo**, escriba *myAppServicePlanEastUS* y seleccione **Aceptar**. |

## <a name="create-a-front-door-for-your-application"></a>Creación de una instancia de Front Door para una aplicación

Configure Azure Front Door para dirigir el tráfico del usuario en función de la latencia más baja entre los dos servidores de las aplicaciones web. Para empezar, agregue un host de front-end para Azure Front Door.

1. En la página principal o en el menú de Azure Portal, seleccione **Crear un recurso**. Seleccione **Redes** > **Ver todas** > **Front Door**.

1. En la pestaña **Conceptos básicos** de la página **Crear una instancia de Front Door**, escriba o seleccione la siguiente información y, a continuación, seleccione **Siguiente: Configuration** (Siguiente: Configuración).

    | Configuración | Value |
    | --- | --- |
    | **Suscripción** | Seleccione su suscripción. |    
    | **Grupos de recursos** | Seleccione **Crear nuevo** y escriba *FrontDoorQS_rg0* en el cuadro de texto.|
    | **Ubicación del grupo de recursos** | Seleccione **Centro de EE. UU.** |

1. En **Front-ends/dominios**, seleccione **+** para abrir **Agregar un host de front-end**.

1. En **Nombre de host**, escriba un nombre de host único global. En este ejemplo se usa *contoso-frontend*. Seleccione **Agregar**.

    :::image type="content" source="media/quickstart-create-front-door/add-frontend-host-azure-front-door.png" alt-text="Agregar un host de front-end para Azure Front Door":::

A continuación, cree un grupo de back-end que contenga las dos aplicaciones web.

1. Todavía en **Crear una instancia de Front Door**, en **Grupos de back-end**, seleccione **+** para abrir **Agregar un grupo de back-end**.

1. En **Nombre**, escriba *myBackendPool* y, a continuación, seleccione **Agregar un back-end**.

    :::image type="content" source="media/quickstart-create-front-door/front-door-add-backend-pool.png" alt-text="Agregar un grupo de back-end":::

1. En la hoja **Agregar un back-end**, seleccione la información siguiente y elija **Agregar**.

    | Configuración | Value |
    | --- | --- |
    | **Tipo de host de back-end** | Seleccione **App Service**. |   
    | **Suscripción** | Seleccione su suscripción. |    
    | **Nombre de host de back-end** | Seleccione la primera aplicación web que creó. En este ejemplo, la aplicación web era *WebAppContoso-1*. |

    **Deje todos los demás campos con el valor predeterminado.*

    :::image type="content" source="media/quickstart-create-front-door/front-door-add-a-backend.png" alt-text="Agregar un host de back-end a Front Door":::

1. Seleccione **Agregar un back-end** de nuevo. Seleccione la siguiente información y elija **Agregar**.

    | Configuración | Value |
    | --- | --- |
    | **Tipo de host de back-end** | Seleccione **App Service**. |   
    | **Suscripción** | Seleccione su suscripción. |    
    | **Nombre de host de back-end** | Seleccione la segunda aplicación web que creó. En este ejemplo, la aplicación web era *WebAppContoso-2*. |

    **Deje todos los demás campos con el valor predeterminado.*

1. Seleccione **Agregar** en la hoja **Agregar un grupo de back-end** para completar la configuración del grupo de back-end.

    :::image type="content" source="media/quickstart-create-front-door/front-door-add-backend-pool-complete.png" alt-text="Agregar un grupo de back-end para Azure Front Door":::

Por último, agregue una regla de enrutamiento. Una regla de enrutamiento asigna el host de front-end al grupo de back-end. La regla reenvía una solicitud para `contoso-frontend.azurefd.net` a **myBackendPool**.

1. Todavía en **Crear una instancia de Front Door**, en **Reglas de enrutamiento**, seleccione **+** para configurar una.

1. En **Agregar una regla**, en **Nombre**, escriba *LocationRule*. Acepte todos los valores predeterminados y, a continuación, seleccione **Agregar** para agregar la regla de enrutamiento.

    :::image type="content" source="media/quickstart-create-front-door/front-door-add-a-rule.png" alt-text="Agregar una regla a Front Door":::

   >[!WARNING]
   > **Debe** asegurarse de que todos los hosts de front-end de Front Door tienen una regla de enrutamiento con una ruta de acceso predeterminada (`\*`) asociada. Es decir, entre todas las reglas de enrutamiento debe haber al menos una para cada uno de los hosts de front-end que se definen en la ruta de acceso predeterminada (`\*`). Si no la hay, puede darse el caso de que el tráfico del usuario final no se enrute correctamente.

1. Seleccione **Revisar y crear** y, a continuación, **Crear**.

    :::image type="content" source="media/quickstart-create-front-door/configuration-azure-front-door.png" alt-text="Azure Front Door configurado":::

## <a name="view-azure-front-door-in-action"></a>Visualización de Front Door en acción

Una vez que cree una instancia de Front Door, la configuración tardará unos minutos en implementarse globalmente. Cuando lo haya hecho, acceda al host de front-end que ha creado. Vaya a `contoso-frontend.azurefd.net` en un explorador. La solicitud se enrutará automáticamente al servidor más cercano de entre los servidores especificados en el grupo de back-end.

Si ha creado estas aplicaciones en este inicio rápido, verá una página de información.

Para probar la conmutación por error global instantánea, realice estos pasos:

1. Abra un explorador como se indica más arriba y vaya a la dirección de front-end: `contoso-frontend.azurefd.net`.

1. En Azure Portal, busque y seleccione *App Services*. Desplácese hacia abajo hasta encontrar una de sus aplicaciones web, **WebAppContoso-1** en este ejemplo.

1. Seleccione la aplicación web y, a continuación, seleccione **Detener** y **Sí** para confirmar.

1. Actualice el explorador. Debería aparecer la misma página de información.

   >[!TIP]
   >Hay algo de retraso en estas acciones. Puede que tenga que actualizar de nuevo.

1. Busque la otra aplicación web y deténgala también.

1. Actualice el explorador. Esta vez debería aparecer un mensaje de error.

   :::image type="content" source="media/quickstart-create-front-door/web-app-stopped-message.png" alt-text="Las dos instancias de la aplicación web detenidas":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Una vez que haya terminado, puede eliminar todos los elementos que haya creado. La eliminación de un grupo de recursos también elimina su contenido. Si no tiene pensado utilizar esta instancia de Front Door, debe eliminar los recursos para evitar cargos innecesarios.

1. En Azure Portal, busque y seleccione **Grupos de recursos** o seleccione **Grupos de recursos** desde el menú de Azure Portal.

1. Filtre o desplácese hacia abajo hasta encontrar un grupo de recursos, por ejemplo **FrontDoorQS_rg0**.

1. Seleccione el grupo de recursos y, después, seleccione **Eliminar grupo de recursos**.

   >[!WARNING]
   >Esta acción es irreversible.

1. Escriba el nombre del grupo de recursos para confirmar y, a continuación, seleccione **Eliminar**.

Repita el procedimiento anterior con los otros dos grupos.

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo para aprender a agregar un dominio personalizado a Front Door.
> [!div class="nextstepaction"]
> [Agregar un dominio personalizado](front-door-custom-domain.md)
