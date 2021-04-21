---
title: 'Tutorial: Creación de un equilibrador de carga entre regiones mediante Azure Portal'
titleSuffix: Azure Load Balancer
description: Comience a trabajar con este tutorial para la implementación de Azure Load Balancer entre regiones con Azure Portal.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: tutorial
ms.date: 02/24/2021
ms.openlocfilehash: 16320021ede4a4e285c4e1973c166d2cdf643c4a
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107529525"
---
# <a name="tutorial-create-a-cross-region-azure-load-balancer-using-the-azure-portal"></a>Tutorial: Creación de una instancia de Azure Load Balancer entre regiones mediante Azure Portal

Un equilibrador de carga entre regiones garantiza que un servicio está disponible globalmente en varias regiones de Azure. Si se produce un error en una región, el tráfico se enruta al siguiente equilibrador de carga regional correcto más cercano.  

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un equilibrador de carga entre regiones.
> * Crear un grupo de back-end que contenga dos equilibradores de carga regionales.
> * Cree una regla de equilibrador de carga.
> * Probar el equilibrador de carga.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

> [!IMPORTANT]
> Azure Load Balancer entre regiones se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prerrequisitos

- Suscripción a Azure.
- Dos instancias de Azure Load Balancer con SKU **Estándar** con grupos de back-end implementados en dos regiones de Azure diferentes.
    - Para obtener información sobre cómo crear un equilibrador de carga estándar regional y máquinas virtuales para los grupos de back-end, consulte [Inicio rápido: Uso de Azure Portal para crear un equilibrador de carga público para equilibrar la carga de máquinas virtuales](quickstart-load-balancer-standard-public-portal.md).
        - Anexe **-R1** y **-R2** al nombre de los equilibradores de carga, las máquinas virtuales y otros recursos de cada región. 

## <a name="sign-in-to-azure-portal"></a>Inicio de sesión en Azure Portal

[Inicie sesión](https://portal.azure.com) en Azure Portal.

## <a name="create-cross-region-load-balancer"></a>Creación de un equilibrador de carga entre regiones

En esta sección, va a crear un equilibrador de carga entre regiones y una dirección IP pública.

1. Seleccione **Crear un recurso**. 
2. En el cuadro de búsqueda, escriba **Equilibrador de carga**. Seleccione **Equilibrador de carga** en los resultados de la búsqueda.
3. En la página **Equilibrador de carga**, seleccione **Crear**.
4. En la pestaña **Conceptos básicos** de la página **Crear equilibrador de carga**, escriba o seleccione la siguiente información: 

    | Configuración                 | Value                                              |
    | ---                     | ---                                                |
    | Subscription               | Seleccione su suscripción.    |    
    | Resource group         | Seleccione **Crear nuevo** y escriba **CreateCRLBTutorial-rg** en el cuadro de texto.|
    | Nombre                   | Escriba **myLoadBalancer-CR**.                                   |
    | Region         | Seleccione **(EE. UU.) Oeste de EE. UU.** .                                        |
    | Tipo          | Seleccione **Público**.                                        |
    | SKU           | Deje el valor predeterminado **Estándar**. |
    | Nivel           | Seleccione **Global**. |
    | Dirección IP pública | Seleccione **Crear nuevo**.|
    | Nombre de la dirección IP pública | Escriba **myPublicIP-CR** en el cuadro de texto.|
    | Preferencia de enrutamiento| Seleccione **Red de Microsoft**. </br> Para obtener más información sobre la preferencia de enrutamiento, vea [¿Qué es la preferencia de enrutamiento (versión preliminar)?](../virtual-network/routing-preference-overview.md) |

    > [!NOTE]
    > El equilibrador de carga entre regiones solo se puede implementar en las siguientes regiones de inicio: **Este de EE. UU. 2, Oeste de EE. UU., Oeste de Europa, Sudeste de Asia, Centro de EE. UU., Norte de Europa, Este de Asia**. Para más información, consulte **https://aka.ms/homeregionforglb**.


3. Acepte los valores predeterminados en los demás valores y seleccione **Revisar y crear**.

4. En la pestaña **Revisar + crear**, seleccione **Crear**.   

    :::image type="content" source="./media/tutorial-cross-region-portal/create-cross-region.png" alt-text="Creación de un equilibrador de carga entre regiones" border="true":::

## <a name="create-backend-pool"></a>Creación de un grupo de back-end

En esta sección va a agregar dos equilibradores de carga estándar regionales al grupo de back-end del equilibrador de carga entre regiones.

> [!IMPORTANT]
> Para completar estos pasos, asegúrese de que se han implementado en la suscripción dos equilibradores de carga regionales con grupos de back-end.  Para más información, consulte **[Inicio rápido: Uso de Azure Portal para crear un equilibrador de carga público para equilibrar la carga de máquinas virtuales](quickstart-load-balancer-standard-public-portal.md)** .

Cree el grupo de direcciones de back-end **myBackendPool-CR** para incluir los equilibradores de carga estándar regionales.

1. Seleccione **Todos los servicios** en el menú de la izquierda, seleccione **Todos los recursos** y, después, en la lista de recursos, seleccione **myLoadBalancer-CR**.

2. En **Configuración**, seleccione **Grupos de back-end** y, a continuación, seleccione **Agregar**.

3. En la página **Agregar un grupo de back-end**, en nombre, escriba **myBackendPool-CR**.

4. Seleccione **Agregar**.

4. Seleccione **myBackendPool-CR**.

5. En **Equilibradores de carga**, seleccione el cuadro desplegable en **Equilibrador de carga**.

5. Seleccione **myLoadBalancer-R1** o el nombre del equilibrador de carga de la región 1.

6. Seleccione el cuadro desplegable en **Configuración de IP de front-end**. Elija **LoadBalancerFrontEnd**.

7. Repita los pasos del 4 al 6 para agregar **myLoadBalancer-R2**.

8. Seleccione **Agregar**.

    :::image type="content" source="./media/tutorial-cross-region-portal/add-to-backendpool.png" alt-text="Adición de equilibradores de carga regionales al grupo de back-end" border="true":::

## <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga

En esta sección va a crear una regla de equilibrador de carga:

* Llamada **myHTTPRule**.
* En el front-end llamado **LoadBalancerFrontEnd**.
* A la escucha en el **puerto 80**.
* Dirige el tráfico con equilibrio de carga al back-end llamado **myBackendPool-CR** en el **puerto 80**.

    > [!NOTE]
    > El puerto de front-end debe coincidir con el puerto de back-end y el puerto de front-end de los equilibradores de carga regionales en el grupo de back-end.

1. Seleccione **Todos los servicios** en el menú de la izquierda, seleccione **Todos los recursos** y, después, en la lista de recursos, seleccione **myLoadBalancer-CR**.

2. En **Configuración**, seleccione **Reglas de equilibrio de carga** y, a continuación, seleccione **Agregar**.

3. Use estos valores para configurar la regla de equilibrio de carga:
    
    | Configuración | Value |
    | ------- | ----- |
    | Nombre | Escriba **myHTTPRule**. |
    | Versión de la dirección IP | Seleccione **IPv4**. |
    | Dirección IP del front-end | Seleccione **LoadBalancerFrontEnd**. |
    | Protocolo | seleccione **TCP**. |
    | Port | Escriba **80**.|
    | Puerto back-end | Escriba **80**. |
    | Grupo back-end | Seleccione **MyBackendPool**.|
    | Tiempo de espera de inactividad (minutos) | Mueva el control deslizante a **15**. |
    | Restablecimiento de TCP | Seleccione **Habilitado**. |

4. Deje el resto de valores predeterminados y después seleccione **Aceptar**.

    :::image type="content" source="./media/tutorial-cross-region-portal/create-lb-rule.png" alt-text="Creación de reglas del equilibrador de carga" border="true":::

## <a name="test-the-load-balancer"></a>Prueba del equilibrador de carga

En esta sección va a probar el equilibrador de carga entre regiones. Se conectará a la dirección IP pública en un explorador web.  Detendrá las máquinas virtuales de uno de los grupos de back-end del equilibrador de carga regional y observará la conmutación por error.

1. Busque la dirección IP pública de Load Balancer en la pantalla **Información general**. Seleccione **Todos los servicios** en el menú de la izquierda, seleccione **Todos los recursos** y, después, seleccione **myPublicIP-CR**.

2. Copie la dirección IP pública y péguela en la barra de direcciones del explorador. La página predeterminada del servidor web de IIS se muestra en el explorador.

    :::image type="content" source="./media/tutorial-cross-region-portal/test-cr-lb-1.png" alt-text="Prueba del equilibrador de carga" border="true":::

3. Detenga las máquinas virtuales del grupo de back-end de uno de los equilibradores de carga regionales.

4. Actualice el explorador web y observe la conmutación por error de la conexión al otro equilibrador de carga regional.

    :::image type="content" source="./media/tutorial-cross-region-portal/test-cr-lb-2.png" alt-text="Prueba del equilibrador de carga después de la conmutación por error" border="true":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no los necesite, elimine el grupo de recursos, el equilibrador de carga y todos los recursos relacionados. 

Para ello, seleccione el grupo de recursos **CreateCRLBTutorial-rg** que contiene los recursos y, a continuación, seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha:

* Creado un equilibrador de carga entre regiones.
* Agregado equilibradores de carga regionales al grupo de back-end del equilibrador de carga entre regiones.
* Creado una regla de equilibrio de carga.
* Probó el equilibrador de carga.

Para más información sobre el equilibrador de carga entre regiones, consulte:
> [!div class="nextstepaction"]
> [Equilibrador de carga entre regiones (versión preliminar)](cross-region-overview.md)
