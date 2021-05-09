---
title: Actualización de dispositivos para el sistema operativo en tiempo real de Azure | Microsoft Docs
description: Introducción a la actualización de dispositivos para el sistema operativo en tiempo real de Azure
author: ValOlson
ms.author: valls
ms.date: 3/18/2021
ms.topic: tutorial
ms.service: iot-hub-device-update
ms.openlocfilehash: b6fcbf7d31e76965cfd8b2944b426dce94b8aa75
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108324516"
---
# <a name="device-update-for-azure-iot-hub-tutorial-using-azure-real-time-operating-system-rtos"></a>Tutorial de actualización de dispositivos para Azure IoT Hub mediante el sistema operativo en tiempo real (RTOS) de Azure

Este tutorial le guiará a través de la creación de Device Update Agent para IoT Hub en Azure RTOS NetX Duo. También proporciona API sencillas para que los desarrolladores integren la funcionalidad de actualización de dispositivos en sus aplicaciones. Consulte [ejemplos](https://github.com/azure-rtos/samples/tree/PublicPreview/ADU) de paneles clave de evaluación de semiconductores que incluyen las guías introductorias para aprender a configurar, crear e implementar las actualizaciones inalámbricas en los dispositivos.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Introducción
> * Etiquetado del dispositivo
> * Creación de un grupo de dispositivos
> * Implementación de una actualización basada en imágenes
> * Supervisión de la implementación de la actualización

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos
* Acceso a IoT Hub. Se recomienda usar el nivel S1 (Estándar) o superior.
* Una instancia de Device Update y una cuenta vinculada a IoT Hub. Si aún no lo ha hecho, siga la guía para [crear y vincular](/azure/iot-hub-device-update/create-device-update-account) una cuenta de Device Update.

## <a name="get-started"></a>Introducción

Cada proyecto de ejemplo de Azure RTOS específico de la placa contiene código y documentación sobre cómo usar Device Update para IoT Hub en él. 
1. Descargue los archivos de ejemplo específicos de la placa de los [ejemplos de Azure RTOS y Device Update](https://github.com/azure-rtos/samples/tree/PublicPreview/ADU).
2. Busque la carpeta "docs" en el ejemplo descargado.
3. En los documentos, siga los pasos para preparar los recursos de Azure, la cuenta y registrar los dispositivos IoT en ella.
5. A continuación, siga los documentos para crear una nueva imagen de firmware e importar el manifiesto para la placa.
6. Publique la imagen de firmware de publicación y el manifiesto en Device Update para IoT Hub.
7. Por último, descargue y ejecute el proyecto en el dispositivo.

Obtenga más información sobre [Azure RTOS](/azure/rtos/).  

## <a name="tag-your-device"></a>Etiquetado del dispositivo

1. Mantenga la aplicación del dispositivo en ejecución desde el paso anterior.
2. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya al centro de IoT.
3. En la opción "IoT Devices" (Dispositivos IoT) del panel de navegación izquierdo, busque el dispositivo IoT y vaya al dispositivo gemelo.
4. En Dispositivo gemelo, elimine cualquier valor de etiqueta de Device Update existente. Para ello, establézcalo en NULL.
5. Agregue un nuevo valor de etiqueta de Device Update como se muestra a continuación.

```JSON
    "tags": {
            "ADUGroup": "<CustomTagValue>"
            }
```

## <a name="create-update-group"></a>Creación de un grupo de actualización

1. Vaya a la instancia de IoT Hub que conectó anteriormente a la instancia de Device Update.
2. Seleccione la opción Actualizaciones del dispositivo en Administración de dispositivos automática en la barra de navegación izquierda.
3. Seleccione la pestaña Groups (Grupos) en la parte superior de la página. 
4. Seleccione el botón Add (Agregar) para crear un grupo.
5. Seleccione en la lista la etiqueta IoT Hub que ha creado en el paso anterior. Seleccione Create update group (Crear grupo de actualización).

   :::image type="content" source="media/create-update-group/select-tag.PNG" alt-text="Captura de pantalla que muestra la selección de etiquetas." lightbox="media/create-update-group/select-tag.PNG":::

[Más información](create-update-group.md) sobre cómo agregar etiquetas y crear grupos de actualización

## <a name="deploy-new-firmware"></a>Implementación de firmware nuevo

1. Una vez creado el grupo, debería ver que hay una actualización nueva disponible para el grupo de dispositivos, con un vínculo a la actualización en Actualizaciones pendientes. Es posible que tenga que actualizar una vez. 
2. Haga clic en la actualización disponible.
3. Confirme que se ha seleccionado el grupo correcto como grupo de destino. Programe la implementación y, a continuación, seleccione Deploy update (Implementar actualización).

   :::image type="content" source="media/deploy-update/select-update.png" alt-text="Select update" lightbox="media/deploy-update/select-update.png"::: (Seleccionar actualización)

4. Consulte el gráfico de compatibilidad. Debería ver que la actualización está en curso. 

   :::image type="content" source="media/deploy-update/update-in-progress.png" alt-text="Actualización en curso" lightbox="media/deploy-update/update-in-progress.png":::

5. Una vez que el dispositivo se haya actualizado correctamente, debería ver que el gráfico de compatibilidad y la actualización de los detalles de implementación reflejan lo mismo. 

   :::image type="content" source="media/deploy-update/update-succeeded.png" alt-text="Actualización correcta" lightbox="media/deploy-update/update-succeeded.png":::

## <a name="monitor-an-update-deployment"></a>Supervisión de una implementación de actualizaciones

1. Seleccione la pestaña Implementaciones en la parte superior de la página.

   :::image type="content" source="media/deploy-update/deployments-tab.png" alt-text="Pestaña Implementaciones" lightbox="media/deploy-update/deployments-tab.png":::

2. Seleccione la implementación que creó para ver los detalles de la implementación.

   :::image type="content" source="media/deploy-update/deployment-details.png" alt-text="Detalles de implementación" lightbox="media/deploy-update/deployment-details.png":::

3. Seleccione Actualizar para ver los detalles de estado más recientes. Continúe con este proceso hasta que el estado cambie a Succeeded (Correcto).

Ahora ha completado una actualización correcta de la imagen de un extremo a otro con Device Update para IoT Hub en un dispositivo Raspberry Pi 3 B+. 

## <a name="cleanup-resources"></a>Limpieza de recursos

Cuando ya no los necesite, limpie la cuenta de Device Update, la instancia, IoT Hub y el dispositivo IoT. 

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure RTOS y su funcionamiento con Azure IoT, vea https://azure.com/rtos.