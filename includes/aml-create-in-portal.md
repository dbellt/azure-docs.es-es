---
title: archivo de inclusión
description: archivo de inclusión
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 05/11/2021
ms.openlocfilehash: 42af7691d7401e397e7e0d9819fded60b3821b26
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109785353"
---
1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con las credenciales de la suscripción de Azure.

1. En la esquina superior izquierda de Azure Portal, seleccione las tres barras y, luego, **+Crear un recurso**.

    :::image type="content" source="media/aml-create-in-portal/create-resource.png" alt-text="Captura de pantalla que muestra +Crear un recurso.":::

1. Use la barra de búsqueda para encontrar **Machine Learning**.

1. Seleccione **Machine Learning**.

   :::image type="content" source="media/aml-create-in-portal/machine-learning.png" alt-text="Captura de pantalla que muestra los resultados de la búsqueda para seleccionar Machine Learning.":::


1. En el panel **Machine Learning**, seleccione **Crear** para comenzar.

1. Especifique la siguiente información para configurar la nueva área de trabajo:

   Campo|Descripción
   ---|---
   Nombre del área de trabajo |Escriba un nombre único que identifique el área de trabajo. En este ejemplo, se usa **docs-ws**. Los nombres deben ser únicos en el grupo de recursos. Utilice un nombre que sea fácil de recordar y que se diferencie del de las áreas de trabajo creadas por otros.
   Subscription |Seleccione la suscripción de Azure que quiera usar.
   Resource group | Use un grupo de recursos existente en su suscripción o escriba un nombre para crear un nuevo grupo de recursos. Un grupo de recursos almacena los recursos relacionados con una solución de Azure. En este ejemplo, se usa **docs-aml**. 
   Location | Seleccione la ubicación más cercana a los usuarios y los recursos de datos para crear el área de trabajo.

1. Después de configurar el área de trabajo, seleccione **Revisar y crear**.
1. Seleccione **Crear** para crear el área de trabajo.

   > [!Warning]
   > La creación del área de trabajo en la nube puede tardar varios minutos.

   Cuando finalice el proceso, aparecerá un mensaje de implementación correcta.
 
 1. Para ver la nueva área de trabajo, seleccione **Ir al recurso**.
 1. En la vista del portal del área de trabajo, seleccione **Iniciar Studio** para ir a Azure Machine Learning Studio. 

