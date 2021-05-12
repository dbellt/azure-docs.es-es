---
title: Visualización de la secuencia de vídeo RTSP de Azure Percept DK
description: Aprenda a ver la secuencia de vídeo RTSP desde Azure Percept DK.
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 02/12/2021
ms.custom: template-how-to
ms.openlocfilehash: 678c5520b002f22552a6262fe245783f88ee61eb
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108754152"
---
# <a name="view-your-azure-percept-dks-rtsp-video-stream"></a>Visualización de la secuencia de vídeo RTSP de Azure Percept DK

Siga esta guía para ver la secuencia de vídeo RTSP desde el dispositivo Azure Percept DK en Azure Percept Studio. La inferencia de los modelos de inteligencia artificial de visión implementados en el dispositivo se podrá ver en la secuencia web.

## <a name="prerequisites"></a>Prerrequisitos

- Kit de desarrollo Azure Percept DK
- [Suscripción de Azure](https://azure.microsoft.com/free/)
- [Experiencia de instalación de Azure Percept DK](./quickstart-percept-dk-set-up.md): ya ha conectado el kit de desarrollo a una red Wi-Fi, creado una instancia de IoT Hub y conectado el kit a esta instancia.

## <a name="view-the-rtsp-video-stream"></a>Visualización de la secuencia de vídeo RTSP

1. Conecte el kit de desarrollo.

1. Vaya a [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819).

1. En el lado izquierdo de la página de información general, haga clic en **Dispositivos**.

    :::image type="content" source="./media/how-to-view-video-stream/overview-devices-inline.png" alt-text="Pantalla de información general de Azure Percept Studio." lightbox="./media/how-to-view-video-stream/overview-devices.png":::

1. Seleccione el kit de desarrollo de la lista.

    :::image type="content" source="./media/how-to-view-video-stream/select-device.png" alt-text="Captura de pantalla de los dispositivos disponibles en Azure Percept Studio.":::

1. Haga clic en **Ver el flujo del dispositivo**.

    :::image type="content" source="./media/how-to-view-video-stream/view-device-stream.png" alt-text="Captura de pantalla de la página del dispositivo que muestra las acciones disponibles del proyecto de visión.":::

    Se abrirá una pestaña independiente que muestra la secuencia web en directo desde el dispositivo Azure Percept DK.

    :::image type="content" source="./media/how-to-view-video-stream/webstream.png" alt-text="Captura de pantalla de la secuencia web del dispositivo.":::

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo ver los [datos de telemetría de Azure Percept DK](./how-to-view-telemetry.md).