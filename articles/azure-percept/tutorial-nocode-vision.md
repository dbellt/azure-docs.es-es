---
title: Creación de una solución de visión sin código en Azure Percept Studio
description: Aprenda a crear una solución de visión sin código en Azure Percept Studio y a implementarla en Azure Percept DK.
author: philmea
ms.author: philmea
ms.service: azure-percept
ms.topic: tutorial
ms.date: 02/10/2021
ms.custom: template-tutorial
ms.openlocfilehash: 35d39fabedb71e27e54d72fd8f5031bcd9510b7b
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108746034"
---
# <a name="create-a-no-code-vision-solution-in-azure-percept-studio"></a>Creación de una solución de visión sin código en Azure Percept Studio

Azure Percept Studio le permite crear e implementar soluciones personalizadas de visión computarizada, sin necesidad de código. En este artículo:

- Creará un proyecto de visión en [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819).
- Recopilará imágenes de entrenamiento con el kit de desarrollo.
- Etiquetará las imágenes de entrenamiento en [Custom Vision](https://www.customvision.ai/).
- Entrenará el modelo personalizado de detección y clasificación de objetos.
- Implementará el modelo en el kit de desarrollo.
- Mejorará el modelo mediante la configuración del nuevo entrenamiento.

Este tutorial es adecuado para desarrolladores con poca o ninguna experiencia en inteligencia artificial y para aquellos que acaban de empezar a usar Azure Percept.

## <a name="prerequisites"></a>Requisitos previos

- Kit de desarrollo Azure Percept DK
- [Suscripción de Azure](https://azure.microsoft.com/free/)
- Experiencia de instalación de Azure Percept DK: ya ha conectado el kit de desarrollo a una red Wi-Fi, creado una instancia de IOT Hub y conectado el kit a esta instancia.

## <a name="create-a-vision-prototype"></a>Creación de un prototipo de visión

1. Inicie el explorador y vaya a [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819).

1. En la página Información general, haga clic en la pestaña **Demostraciones y tutoriales**. :::image type="content" source="./media/tutorial-nocode-vision/percept-studio-overview-inline.png" alt-text="Pantalla Información general de Azure Percept Studio." lightbox="./media/tutorial-nocode-vision/percept-studio-overview.png":::

1. En **Vision tutorials and demos** (Tutoriales y demostraciones de visión), haga clic en **Create a vision prototype** (Crear un prototipo de visión).

    :::image type="content" source="./media/tutorial-nocode-vision/vision-tutorials-and-demos-inline.png" alt-text="Pantalla de demostraciones y tutoriales de Azure Percept Studio." lightbox="./media/tutorial-nocode-vision/vision-tutorials-and-demos.png":::

1. En la página **New Azure Percept Custom Vision prototype** (Nuevo prototipo de Custom Vision para Azure Percept), haga lo siguiente:

    1. En el cuadro **Project name** (Nombre del proyecto), escriba un nombre para el prototipo de visión.

    1. Escriba una descripción del prototipo de visión en el cuadro **Project description** (Descripción del proyecto).

    1. Seleccione **Azure Percept DK** en el menú desplegable **Device type** (Tipo de dispositivo).

    1. Seleccione un recurso en el menú desplegable **Resource** (Recurso) o haga clic en **Create a new resource** (Crear un nuevo recurso). Si opta por crear un nuevo recurso, haga lo siguiente en la ventana **Create** (Crear):
        1. Escriba un nombre para el nuevo recurso.
        1. Seleccione la suscripción de Azure.
        1. seleccione un grupo de recursos o cree uno.
        1. Seleccione la región que prefiera.
        1. Seleccione el plan de tarifa (se recomienda S0).
        1. En la parte inferior de la ventana, haga clic en **Create** (Crear).

        :::image type="content" source="./media/tutorial-nocode-vision/create-resource.png" alt-text="Ventana para crear recurso.":::

    1. En **Project type** (Tipo de proyecto), elija si el proyecto de visión realizará la detección de objetos o la clasificación de imágenes. Para más información sobre los tipos de proyecto, haga clic en **Help me choose** (Ayúdame a elegir).

    1. En **Optimization** (Optimización), seleccione si desea optimizar el proyecto para conseguir precisión, latencia de red baja o un equilibrio entre ambas opciones.

    1. Haga clic en el botón **Crear**.

        :::image type="content" source="./media/tutorial-nocode-vision/create-prototype.png" alt-text="Pantalla para crear prototipo de Custom Vision.":::

## <a name="connect-a-device-to-your-project-and-capture-images"></a>Conexión de un dispositivo al proyecto y captura de imágenes

Después de crear una solución de visión, debe agregar el kit de desarrollo y las instancias de IoT Hub correspondientes a ella.

1. Encienda el kit de desarrollo.

1. En el menú desplegable **IoT Hub**, seleccione la instancia de IoT Hub a la que conectó el kit de desarrollo durante la configuración rápida.

1. En el menú desplegable **Devices** (Dispositivos), seleccione el kit de desarrollo.

A continuación, debe cargar imágenes o capturarlas para entrenar el modelo de inteligencia artificial. Se recomienda cargar al menos 30 imágenes por tipo de etiqueta. Por ejemplo, si desea crear un detector de perros y gatos, debe cargar al menos 30 imágenes de perros y 30 imágenes de gatos. Para capturar imágenes con el módulo de sistema de visión del kit de desarrollo, haga lo siguiente:

1. En la ventana **Image capture** (Captura de imagen), seleccione **View Device Stream** (Ver flujo de dispositivo) para ver la secuencia de vídeo del módulo de sistema de visión.

1. Compruebe el flujo de vídeo para asegurarse de que la cámara del módulo de sistema esté correctamente alineada para tomar las imágenes de entrenamiento. Realice los ajustes necesarios.

1. En la ventana **Image capture** (Captura de imagen), haga clic en **Take photo** (Tomar foto).

    :::image type="content" source="./media/tutorial-nocode-vision/image-capture.png" alt-text="Pantalla Captura de imagen.":::

1. Como alternativa, configure una captura de imagen automatizada para recopilar una gran cantidad de imágenes a la vez; para ello, active la casilla **Automatic image capture** (Captura de imagen automática). Seleccione la velocidad de creación de imágenes preferida en **Capture rate** (Velocidad de captura) y el número total de imágenes que quiere recopilar en **Target** (Destino). Haga clic en **Set automatic capture** (Establecer captura automática) para iniciar el proceso de captura automática de imágenes.

    :::image type="content" source="./media/tutorial-nocode-vision/image-capture-auto.png" alt-text="Menú desplegable Captura de imagen automática.":::

Cuando tenga suficientes fotos, haga clic en **Next: Tag images and model training** (Siguiente: Etiquetar imágenes y entrenamiento del modelo) en la parte inferior de la pantalla. Todas las imágenes se guardarán en [Custom Vision](https://www.customvision.ai/).

> [!NOTE]
> Si opta por cargar imágenes de entrenamiento directamente en Custom Vision, tenga en cuenta que el tamaño del archivo de imagen no puede superar los 6 MB.

## <a name="tag-images-and-train-your-model"></a>Etiquetado de imágenes y entrenamiento del modelo

Antes de entrenar el modelo, agregue etiquetas a las imágenes.

1. En la página **Tag images and model training** (Etiquetar imágenes y entrenamiento del modelo), haga clic en **Open project in Custom Vision** (Abrir proyecto en Custom Vision).

1. En el lado izquierdo de la página **Custom Vision**, haga clic en **Untagged** (Sin etiquetas) en **Tags** (Etiquetas) para ver las imágenes que acaba de recopilar en el paso anterior. Seleccione una o varias de las imágenes sin etiquetar.

1. En la ventana **Image Detail** (Detalles de la imagen), haga clic en la imagen para comenzar el etiquetado. Si ha seleccionado la detección de objetos como tipo de proyecto, también debe dibujar un [rectángulo de selección](../cognitive-services/custom-vision-service/get-started-build-detector.md#upload-and-tag-images) alrededor de los objetos específicos que desea etiquetar. Ajuste el rectángulo de selección según sea necesario. Escriba la etiqueta de objeto y haga clic en **+** para aplicar la etiqueta. Por ejemplo, para crear una solución de visión que le notificara cuando una estantería de una tienda necesita reabastecerse, agregue la etiqueta "Estantería vacía" a las imágenes de estanterías vacías y agregue la etiqueta "Estantería llena" a las imágenes de estanterías llenas. Repita el procedimiento con todas las imágenes sin etiquetar.

    :::image type="content" source="./media/tutorial-nocode-vision/image-tagging.png" alt-text="Pantalla de etiquetado de imágenes de Custom Vision.":::

1. Después de etiquetar las imágenes, haga clic en el icono **X** en la esquina superior derecha de la ventana. Haga clic en **Tagged** (Etiquetada) en **Tags** (Etiquetas) para ver todas las imágenes recién etiquetadas.

1. Una vez que las imágenes estén etiquetadas, estará listo para entrenar el modelo de inteligencia artificial. Para ello, haga clic en **Train** (Entrenar) cerca de la parte superior de la página. Debe tener al menos 15 imágenes por tipo de etiqueta para entrenar el modelo (se recomienda usar al menos 30). El entrenamiento normalmente tarda unos 30 minutos, pero puede tardar más tiempo si el conjunto de imágenes es extremadamente grande.

    :::image type="content" source="./media/tutorial-nocode-vision/train-model.png" alt-text="Selección de imágenes de entrenamiento con el botón Entrenar resaltado.":::

1. Cuando se haya completado el entrenamiento, la pantalla mostrará el rendimiento del modelo. Para más información sobre la evaluación de estos resultados, consulte la [documentación sobre evaluación de modelos](../cognitive-services/custom-vision-service/get-started-build-detector.md#evaluate-the-detector). Después del entrenamiento, puede que también desee [probar el modelo](../cognitive-services/custom-vision-service/test-your-model.md) en imágenes adicionales y volver a entrenar según sea necesario. Cada vez que entrena el modelo, se guardará como una nueva iteración. Consulte la [documentación de Custom Vision](../cognitive-services/custom-vision-service/getting-started-improving-your-classifier.md) para más información sobre cómo mejorar el rendimiento del modelo.

    :::image type="content" source="./media/tutorial-nocode-vision/iteration.png" alt-text="Resultados del entrenamiento del modelo.":::

    > [!NOTE]
    > Si opta por probar el modelo en imágenes adicionales en Custom Vision, tenga en cuenta que el tamaño del archivo de imagen de prueba no puede superar los 4 MB.

Una vez que esté satisfecho con el rendimiento del modelo, cierre la pestaña del explorador para cerrar Custom Vision.

## <a name="deploy-your-ai-model"></a>Implementación del modelo de inteligencia artificial

1. Vuelva a la pestaña Azure Percept Studio y haga clic en **Next: Evaluate and deploy** (Siguiente: Evaluación e implementación) en la parte inferior de la pantalla.

1. La ventana **Evaluate and deploy** (Evaluar e implementar) mostrará el rendimiento de la iteración del modelo seleccionada. Seleccione la iteración que desea implementar en el kit de desarrollo en el menú desplegable **Model iteration** (Iteración del modelo) y haga clic en **Deploy model** (Implementar modelo) en la parte inferior de la pantalla.

    :::image type="content" source="./media/tutorial-nocode-vision/deploy-model-inline.png" alt-text="Pantalla de implementación de modelo." lightbox="./media/tutorial-nocode-vision/deploy-model.png":::

1. Después de implementar el modelo, vea la secuencia de vídeo del dispositivo para ver la inferencia de modelos en acción.

    :::image type="content" source="./media/tutorial-nocode-vision/view-device-stream.png" alt-text="Flujo de dispositivo que muestra el detector de auriculares en acción.":::

Después de cerrar esta ventana, puede volver atrás y editar el proyecto de visión en cualquier momento; para ello, haga clic en **Vision** en **AI Projects** (Proyectos de IA) en la página principal de Azure Percept Studio y seleccione el nombre de su proyecto de visión.

:::image type="content" source="./media/tutorial-nocode-vision/vision-project-inline.png" alt-text="Página del proyecto de visión." lightbox="./media/tutorial-nocode-vision/vision-project.png":::

## <a name="improve-your-model-by-setting-up-retraining"></a>Mejora del modelo mediante la configuración del nuevo entrenamiento

Una vez que haya entrenado el modelo y lo haya implementado en el dispositivo, puede mejorar el rendimiento del modelo si configura los parámetros de repetición del entrenamiento para capturar más datos de entrenamiento. Esta característica se usa para mejorar el rendimiento de un modelo entrenado, ya que ofrece la posibilidad de capturar imágenes en función de un intervalo de probabilidad. Por ejemplo, puede configurar el dispositivo para que solo capture imágenes de entrenamiento cuando la probabilidad sea baja. A continuación se ofrecen algunas [instrucciones adicionales](../cognitive-services/custom-vision-service/getting-started-improving-your-classifier.md) sobre cómo agregar más imágenes y equilibrar los datos de entrenamiento.

1. Para configurar la repetición del entrenamiento, vuelva al **Proyecto** y, a continuación, a **Resumen del proyecto**
1. En la pestaña **Captura de imágenes**, seleccione **Captura de imágenes automática** y **Configurar el nuevo entrenamiento**.
1. Puede configurar la captura de imágenes automática para recopilar una gran cantidad de imágenes a la vez; para ello, active la casilla **Captura de imágenes automática**.
1. Seleccione la velocidad de creación de imágenes preferida en **Capture rate** (Velocidad de captura) y el número total de imágenes que quiere recopilar en **Target** (Destino).
1. En la sección **Configurar el nuevo entrenamiento**, seleccione la iteración para la que desea capturar más datos de entrenamiento y, a continuación, seleccione el intervalo de probabilidad. Solo se cargarán en el proyecto las imágenes que cumplan la tasa de probabilidad.

    :::image type="content" source="./media/tutorial-nocode-vision/vision-image-capture.png" alt-text="Captura de imagen.":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha creado un nuevo recurso de Azure para este tutorial y ya no desea desarrollar ni usar la solución de visión, realice los pasos siguientes para eliminar el recurso:

1. Vaya a [Azure Portal](https://ms.portal.azure.com/).
1. Haga clic en **Todos los recursos**.
1. Haga clic en la casilla situada junto al recurso creado durante este tutorial. El tipo de recurso se mostrará como **Cognitive Services**.
1. Haga clic en el icono **Eliminar** situado cerca de la parte superior de la pantalla.

## <a name="video-walkthrough"></a>Tutorial en vídeo

Para ver un tutorial visual de los pasos descritos anteriormente, consulte el siguiente vídeo:

</br>

> [!VIDEO https://www.youtube.com/embed/9LvafyazlJM]

</br>

## <a name="next-steps"></a>Pasos siguientes

A continuación, consulte los artículos de procedimientos de visión para más información sobre las características adicionales de la solución en Azure Percept Studio.

<!--
Add links to how-to articles and oobe article.
-->