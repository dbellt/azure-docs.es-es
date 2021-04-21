---
title: 'Mediciones de usuario reales con páginas web: Azure Traffic Manager'
description: En este artículo, aprenderá a configurar las páginas web para enviar Mediciones de usuario reales a Azure Traffic Manager.
services: traffic-manager
documentationcenter: traffic-manager
author: duongau
manager: kumud
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 04/06/2021
ms.author: duau
ms.openlocfilehash: 7ec91945c901f46d7036e8c474f9f5f0714ce65e
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106580384"
---
# <a name="how-to-send-real-user-measurements-to-azure-traffic-manager-using-web-pages"></a>Envío de medidas de usuarios reales a Azure Traffic Manager mediante páginas web

Para configurar las páginas web para enviar medidas de usuarios reales a Traffic Manager, obtenga una clave de Real User Measurements (RUM) e inserte el código generado en la página web.

## <a name="obtain-a-real-user-measurements-key"></a>Obtención de una clave de Real User Measurements

El servicio identifica las medidas que se realizan y envían a Traffic Manager desde la aplicación cliente mediante una cadena única, llamada **clave de Mediciones de usuario reales (RUM)** . Puede obtener una clave de RUM mediante Azure Portal, una API REST, PowerShell o la CLI de Azure.

Para obtener la clave RUM con Azure Portal:
1. En un explorador, inicie sesión en Azure Portal. Si aún no tiene una cuenta, puede registrarse para disfrutar de una evaluación gratuita de un mes.

1. En la barra de búsqueda del portal, busque el nombre del perfil de Traffic Manager que desea modificar y selecciónelo en los resultados que se muestran.

1. En la página del perfil de Traffic Manager, seleccione **Mediciones de usuario reales** en **Configuración**.

1. Seleccione **Generar clave** para crear una nueva clave de RUM.

    :::image type="content" source="./media/traffic-manager-create-rum-web-pages/generate-rum-key.png" alt-text="Captura de pantalla de cómo generar una clave."::: 

   **Figura 1: Generación de la clave de Real User Measurements**

1. La página muestra ahora la clave de RUM generada y un fragmento de código JavaScript que se debe insertar en la página HTML.

    :::image type="content" source="./media/traffic-manager-create-rum-web-pages/rum-javascript-code.png" alt-text="Captura de pantalla del código de JavaScript generado."::: 

    **Figura 2: Clave de Real User Measurements y JavaScript de medida**
 
1. Seleccione el botón **Copiar** para copiar el código JavaScript. 

> [!IMPORTANT]
> Use el código JavaScript generado para que la característica Real User Measurements funcione correctamente. Los cambios realizados en este script o en los scripts usados por Real User Measurements pueden llevar a comportamientos impredecibles.

## <a name="embed-the-code-to-an-html-web-page"></a>Inserción del código en una página web HTML

Después de haber obtenido la clave de RUM, el siguiente paso es insertar este código JavaScript copiado en una página HTML que visiten sus usuarios finales. La edición de HTML puede realizarse de muchas maneras y con distintas herramientas y flujos de trabajo. En este ejemplo se muestra cómo actualizar una página HTML para agregar este script. Puede usarlo como guía y adaptarlo a su flujo de trabajo de administración de código fuente HTML.

1. Abra la página HTML en un editor de texto.

1. Pegue el código JavaScript que copió en la sección anterior en la sección BODY del código HTML. El código copiado está en las líneas 8 y 9; consulte la figura 3.

    :::image type="content" source="./media/traffic-manager-create-rum-web-pages/real-user-measurement-embed-script.png" alt-text="Captura de pantalla del código JavaScript generado insertado en la página web."::: 

    **Figura 3: HTML simple con JavaScript insertado de Real User Measurements**

1. Guarde el archivo HTML y hospédelo en un servidor web conectado a Internet.

1. La próxima vez que se represente la página en un explorador web, el código JavaScript al que se hace referencia se descarga y el script ejecutará las operaciones de mediciones e informes.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre [Real User Measurements](traffic-manager-rum-overview.md).
- Aprenda [cómo funciona el Administrador de tráfico](traffic-manager-overview.md)
- Aprenda más sobre los [métodos de enrutamiento de tráfico](traffic-manager-routing-methods.md) que admite el Administrador de tráfico.
- Aprenda a [crear un perfil del Administrador de tráfico](./quickstart-create-traffic-manager-profile.md)
