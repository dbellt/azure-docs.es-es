---
title: 'Inicio rápido: Creación de una aplicación de HoloLens con Unity y MRTK'
description: En este artículo de inicio rápido, aprenderá a crear una aplicación de HoloLens para Unity con MRTK y Object Anchors.
author: craigktreasure
manager: virivera
services: azure-object-anchors
ms.author: crtreasu
ms.date: 03/02/2021
ms.topic: quickstart
ms.service: azure-object-anchors
ms.openlocfilehash: 5cccfb2b750e9ea1a167f8bbbddcb2621bd19b5b
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112008092"
---
# <a name="quickstart-create-a-hololens-app-with-azure-object-anchors-in-unity-with-mrtk"></a>Inicio rápido: Creación de una aplicación de HoloLens con Azure Object Anchors, en Unity con MRTK

En este artículo de inicio rápido se creará una aplicación de HoloLens en Unity con [Azure Object Anchors](../overview.md). Azure Object Anchors es un servicio en la nube administrado que convierte recursos 3D en modelos de inteligencia artificial (IA) que permiten experiencias de realidad mixta que reconocen objetos para HoloLens. Cuando haya terminado, tendrá una aplicación de HoloLens compilada con Unity, que puede detectar objetos en el mundo físico.

Aprenderá a:

> [!div class="checklist"]
> * Preparar la configuración de compilación de Unity.
> * Exportar el proyecto HoloLens de Visual Studio.
> * Implementar la aplicación y ejecutarla en un dispositivo HoloLens 2.

[!INCLUDE [Unity quickstart prerequisites](../../../includes/object-anchors-quickstart-unity-prerequisites.md)]

[!INCLUDE [Unity device setup](../../../includes/object-anchors-quickstart-unity-device-setup.md)]

## <a name="open-the-sample-project"></a>Apertura del proyecto de ejemplo

[!INCLUDE [Clone Sample Repo](../../../includes/object-anchors-clone-sample-repository.md)]

[!INCLUDE [Download Unity Package](../../../includes/object-anchors-quickstart-unity-download-package.md)]

En Unity, abra el proyecto `quickstarts/apps/unity/mrtk`.

[!INCLUDE [Import Unity Package](../../../includes/object-anchors-quickstart-unity-import-package.md)]

[!INCLUDE [Unity build sample scene 1](../../../includes/object-anchors-quickstart-unity-build-sample-scene-1.md)]

Cuando un cuadro de diálogo "TMP Importer" (Importador de TMP) le pida que importe recursos de TextMesh Pro, seleccione "Import TMP Essentials" (Importar la información esencial de TMP) para hacerlo.
:::image type="content" source="./media/textmesh-pro-importer-dialog.png" alt-text="Importación de recursos de TextMesh Pro":::

[!INCLUDE [Unity build sample scene 2](../../../includes/object-anchors-quickstart-unity-build-sample-scene-2.md)]

[!INCLUDE [Unity build and deploy](../../../includes/object-anchors-quickstart-unity-build-deploy.md)]

### <a name="run-the-sample-app"></a>Ejecutar la aplicación de ejemplo

Encienda el dispositivo, seleccione **All Apps** (Todas las aplicaciones) y, a continuación, busque e inicie la aplicación. Después de la pantalla de presentación de Unity, debería ver un rectángulo delimitador. Puede usar la mano para moverse, escalar o girar el rectángulo delimitador. Coloque el rectángulo para cubrir el objeto que quiere detectar.

Abra el <a href="https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/README_HandMenu.html" target="_blank">menú de la mano</a> y seleccione **Lock SearchArea** (Bloquear SearchArea) para evitar que el rectángulo delimitador se siga moviendo. Seleccione **Start Search** (Iniciar búsqueda) para iniciar la detección de objetos. Cuando se detecte el objeto, se representará una malla en el objeto. Los detalles de una instancia detectada se mostrarán en la pantalla, como la marca de tiempo actualizada y la relación de cobertura de superficie. Seleccione **Stop Search** (Detener búsqueda) para detener el seguimiento, con lo que se quitarán todas las instancias detectadas.

#### <a name="the-app-menus"></a>Menús de la aplicación

También puede realizar otras acciones mediante el <a href="https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/README_HandMenu.html" target="_blank">menú de la mano</a>.

##### <a name="primary-menu"></a>Menú principal

* **Start Search / Stop Search** (Iniciar búsqueda / Detener búsqueda): Inicia o detiene el proceso de detección de objetos.
* **Toggle Spatial Mapping** (Alternar asignación espacial): Muestra u oculta la representación de la asignación espacial. Esta opción se puede usar para depuración si el examen se ha completado o no.
* **Tracker Settings** (Configuración de seguimiento): Alterna la activación del menú de configuración de seguimiento.
* **Search Area Settings** (Configuración del área de búsqueda): Alterna la activación del menú de configuración del área de búsqueda.
* **Start Tracing** (Iniciar seguimiento): Captura los datos de diagnóstico y los guarda en el dispositivo. Consulte más detalles en la sección **Depuración de problemas de detección y captura de diagnósticos**.
* **Upload Tracing** (Cargar seguimiento): Cargue los datos de diagnóstico en el servicio Object Anchors. Un usuario debe proporcionar su cuenta de suscripción en `subscription.json` y cargarlo en la carpeta `LocalState`. A continuación encontrará un archivo `subscription.json` de ejemplo.

    :::image type="content" source="./media/mrtk-hand-menu-primary.png" alt-text="Menú principal de la mano en Unity":::

##### <a name="tracker-settings-menu"></a>Menú de configuración de seguimiento

* **High Accuracy** (Alta precisión): Característica experimental que se usa para obtener una posición más precisa. Al habilitar esta opción, se requerirán más recursos del sistema durante la detección de objetos. La malla del objeto se representará en rosa en este modo. Seleccione este botón de nuevo para volver al modo de seguimiento normal.
* **Relaxed Vertical Alignment** (Alineación vertical relajada): Cuando esta opción está habilitada, permite detectar un objeto en un ángulo no vertical. Resulta útil para detectar objetos en rampas.
* **Allow Scale Change** (Permitir cambio de escala): Permite al rastreador cambiar el tamaño del objeto detectado en función de la información del entorno.
* **Control deslizante Coverage Ratio** (Relación de cobertura): Ajusta la proporción de puntos de superficie que deben coincidir para que el rastreador detecte un objeto.  Los valores inferiores permiten al rastreador detectar mejor los objetos que son difíciles de detectar para los sensores de HoloLens, como objetos oscuros u objetos muy reflectantes. Los valores superiores reducirán la frecuencia de las detecciones falsas.

    :::image type="content" source="./media/mrtk-hand-menu-tracker.png" alt-text="Menú de la mano de seguimiento en Unity":::

##### <a name="search-area-settings-menu"></a>Menú de configuración del área de búsqueda

* **Lock Search Area** (Bloquear área de búsqueda): Bloquee el rectángulo delimitador del área para evitar el movimiento accidental debido a las manos.
* **Auto-Adjust Search Area** (Ajustar área de búsqueda automáticamente): Permite que el área de búsqueda cambie de posición durante la detección de objetos.
* **Cycle Mesh** (Ciclo de malla): Recorre la visualización de las mallas cargadas dentro del área de búsqueda.  Esta opción puede ayudar a los usuarios a alinear el cuadro de búsqueda para los objetos difíciles de detectar.

    :::image type="content" source="./media/mrtk-hand-menu-search-area.png" alt-text="Menú de la mano del área de búsqueda en Unity":::

Ejemplo `subscription.json`:

```json
{
  "AccountId": "<your account id>",
  "AccountKey": "<your account key>",
  "AccountDomain": "<your account domain>"
}
```

[!INCLUDE [Unity setup Windows Device Portal](../../../includes/object-anchors-quickstart-unity-setup-device-portal.md)]

[!INCLUDE [Unity upload your model](../../../includes/object-anchors-quickstart-unity-upload-model.md)]

[!INCLUDE [Unity troubleshooting](../../../includes/object-anchors-quickstart-unity-troubleshooting.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conceptos: Información general del SDK](../concepts/sdk-overview.md)

> [!div class="nextstepaction"]
> [P+F](../faq.md)

> [!div class="nextstepaction"]
> [SDK de conversión](/dotnet/api/overview/azure/mixedreality.objectanchors.conversion-readme-pre)
