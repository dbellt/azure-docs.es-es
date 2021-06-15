---
title: 'Ver resultados de pruebas del sistema e implementación: Custom Translator'
titleSuffix: Azure Cognitive Services
description: Cuando el entrenamiento se realice correctamente, revise las pruebas del sistema para analizar los resultados del entrenamiento. Si está satisfecho con los resultados del entrenamiento, envíe una solicitud de implementación para el modelo entrenado.
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: lajanuar
ms.topic: conceptual
ms.openlocfilehash: c2d97e9e9f0511a81c94e151f07d7d74ef437ca7
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111409981"
---
# <a name="view-system-test-results"></a>Ver resultados de pruebas del sistema

Cuando el entrenamiento se realice correctamente, revise las pruebas del sistema para analizar los resultados del entrenamiento. Si está satisfecho con los resultados del entrenamiento, envíe una solicitud de implementación para el modelo entrenado.

## <a name="system-test-results-page"></a>Página de resultados de pruebas del sistema

Seleccione un proyecto y, a continuación, seleccione la pestaña de modelos de ese proyecto, busque el modelo que quiere usar y, por último, seleccione la pestaña de pruebas.

En la pestaña de pruebas se muestra lo siguiente:

1.  **Resultados de pruebas del sistema:** el resultado del proceso de pruebas en los entrenamientos. El proceso de pruebas genera la puntuación BLEU.

    **Recuento de oraciones:** cuántas oraciones paralelas se usaron en el conjunto de pruebas.

     **Puntuación BLEU:** puntuación BLEU generada para un modelo tras la finalización del entrenamiento.

    **Estado:** indica si se ha completado el proceso de prueba o si está en curso.

    ![Resultados de pruebas del sistema](media/how-to/how-to-system-test-results.png)

2.  Haga clic en los resultados de pruebas del sistema y se mostrará la página de detalles del resultado de las pruebas. En esta página se muestra la traducción automática de las oraciones que formaban parte del conjunto de datos de prueba.

3.  La tabla en la página de detalles del resultado de pruebas tiene dos columnas: una para cada idioma en el par. La columna del idioma de origen muestra la oración que se va a traducir. La columna para el idioma de destino contiene dos oraciones por cada fila.

    **Ref:** esta oración es la traducción de referencia para la oración de origen tal como se especificó en el conjunto de datos de prueba.

    **MT:** esta oración es la traducción automática de la oración de origen realizada por el modelo creado después de llevar a cabo el entrenamiento.

    ![Comparación de los resultados de pruebas del sistema](media/how-to/how-to-system-test-results-2.png)

## <a name="download-test"></a>Descargar la prueba

Haga clic en el vínculo Download Translations (Descargar traducciones) para descargar un archivo zip. El archivo zip contiene las traducciones automáticas de las oraciones de origen en el conjunto de datos de pruebas.

![Descargar la prueba](media/how-to/how-to-system-test-download.png)

Este archivo zip descargado contiene tres archivos.

1.  **custom.mt.txt:** contiene la traducción automática de las oraciones del idioma de origen al idioma de destino que realizó el modelo entrenado con los datos del usuario.

2.  **ref.txt:** contiene las traducciones de las oraciones del idioma de origen proporcionadas por el usuario en el idioma de destino.

3.  **source.txt:** contiene las oraciones en el idioma de origen.

    ![Resultados de pruebas del sistema descargados](media/how-to/how-to-download-system-test.png)

## <a name="deploy-a-model"></a>Implementar un modelo

Para solicitar una implementación, haga lo siguiente:

1.  Seleccione un proyecto y vaya a la pestaña de modelos.

2. Para cada modelo entrenado correctamente, se muestra un botón "Implementar" si aún no se implementa.

    ![Captura de pantalla que resalta el botón Implementar para implementar un modelo.](media/how-to/how-to-deploy-model.png)

3.  Haga clic en Implementar.
4.  Seleccione **Implementado** para las regiones donde quiere que se implemente el modelo y haga clic en Guardar. Puede seleccionar **Implementado** para varias regiones.

    ![Captura de pantalla que muestra dónde puede implementar o anular la implementación de un modelo.](media/how-to/how-to-deploy-model-regions.png)

5.  Puede ver el estado del modelo en la columna "Estado".

>[!Note]
>Traductor personalizado es compatible con 10 modelos implementados dentro de un área de trabajo en cualquier momento dado.

## <a name="update-deployment-settings"></a>Actualizar la configuración de implementación

Para actualizar la configuración de implementación:

1.  Seleccione un proyecto y vaya a la pestaña **Modelos**.

2. En el caso de un modelo implementado correctamente, se muestra un botón **Actualizar**.

    ![Captura de pantalla que resalta el botón Actualizar para actualizar la configuración de la implementación.](media/how-to/how-to-update-undeploy-model.png)

3.  Seleccione **Actualizar**.
4.  Seleccione **Implementado** o **No implementado** para las regiones donde quiere que se implemente el modelo o se anule su implementación y, a continuación, haga clic en **Guardar**.

    ![Implementación de un modelo](media/how-to/how-to-undeploy-model.png)

>[!Note]
>Si selecciona **No implementado** para todas las regiones, se anula la implementación del modelo en todas las regiones y se coloca en un estado no implementado. Ahora está deshabilitado para su uso.

## <a name="next-steps"></a>Pasos siguientes

- Comience a utilizar el modelo de traducción personalizado implementado mediante [Microsoft Translator Text API V3](../reference/v3-0-translate.md?tabs=curl).
- Obtenga información sobre [cómo administrar la configuración](how-to-manage-settings.md) para compartir el área de trabajo y administrar la clave de suscripción.
