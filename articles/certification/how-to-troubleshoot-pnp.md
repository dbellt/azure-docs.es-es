---
title: Solución de problemas con un dispositivo IoT Plug and Play
description: Una guía de pasos de solución de problemas recomendados para los asociados que están certificando un dispositivo IoT Plug and Play.
author: nkuntjoro
ms.author: nikuntjo
ms.service: certification
ms.topic: how-to
ms.date: 04/15/2021
ms.custom: template-how-to
ms.openlocfilehash: 9b406e489fb83083d47f01e1483160181601d518
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107558854"
---
# <a name="troubleshoot-your-iot-plug-and-play-certification-project"></a>Solución de problemas del proyecto de certificación de IoT Plug and Play

Durante la fase Conectar y probar del proyecto de certificación de IoT Plug and Play, puede encontrarse con algunos escenarios que le impedirán superar las pruebas de Azure for IoT Certification Service (AICS).

## <a name="prerequisites"></a>Requisitos previos

- Debe haber iniciado sesión y tener un proyecto creado para su dispositivo en el [portal de Azure Certified Device](https://certify.azure.com). Para más información, consulte el [tutorial](tutorial-01-creating-your-project.md).

## <a name="when-aics-tests-arent-passing"></a>Cuando no se superan las pruebas de AICS

Es posible que la prueba de AICS no se supere debido a varias causas. Siga estos pasos para comprobar si hay problemas comunes y solucionar los problemas del dispositivo.

1. Vuelva a comprobar que el código del dispositivo esté configurando la carga del identificador de modelo durante el aprovisionamiento de DPS. Este es un requisito para que AICS valide el dispositivo.
1. Para ver los registros de telemetría de las series de pruebas anteriores, presione el botón `View Logs` para identificar lo que está causando un error en la prueba. Puede revisar tanto la mensajería de prueba como los datos sin procesar.  

    ![Revisión de los datos de prueba](./media/images/review-logs.png)

1. En algunos de los casos en los que los registros indican `Failed to get Digital Twin Model ID of device xx due to DeviceNotConnected`, pruebe a reiniciar el dispositivo y el proceso de aprovisionamiento de dispositivos.
1. Si las pruebas automatizadas siguen generando errores, puede `request a manual review` de los resultados que se sustituirán. De este modo, se desencadenará una solicitud de **validación manual** con el equipo de Azure Certified Device.  

    ![Solicitud de revisión manual](./media/images/request-manual-review.png)

## <a name="when-you-see-passed-with-warnings"></a>Cuando se muestra el mensaje "Se pasó con advertencias"

Mientras se ejecutan las pruebas, si recibe como resultado `Passed with warnings`, significa que no se recibió telemetría durante el período de prueba. Esto puede deberse a que la telemetría requería de intervalos de tiempo más largos o los desencadenadores externos no estaban disponibles. Puede continuar con el envío del dispositivo para su revisión, y durante el proceso el equipo de revisión determinará si la **validación manual** es necesaria en el futuro.

## <a name="when-you-need-help-with-the-model-repository"></a>Cuando necesita ayuda con el repositorio de modelos

Para los problemas de IoT Plug and Play relacionados con el repositorio de modelos, consulte nuestra [guía de la documentación acerca del repositorio de modelos de dispositivo](https://docs.microsoft.com/azure/iot-pnp/concepts-model-repository).

## <a name="next-steps"></a>Pasos siguientes

Esperamos que esta guía le ayude a continuar con su proceso de certificación de IoT Plug and Play. Una vez que haya superado AICS, podrá continuar con nuestros tutoriales para enviar y publicar el dispositivo.

- [Tutorial: Pruebas del dispositivo](tutorial-03-testing-your-device.md)
