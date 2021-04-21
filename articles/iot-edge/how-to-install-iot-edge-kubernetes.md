---
title: Cómo instalar IoT Edge en Kubernetes | Microsoft Docs
description: Obtenga información sobre cómo instalar IoT Edge en Kubernetes mediante un entorno de clústeres de desarrollo local
author: kgremban
manager: philmea
ms.author: veyalla
ms.date: 04/26/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
monikerRange: iotedge-2018-06
ms.openlocfilehash: a5407c46be6726974a920b5dddd87639dc426d0d
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107587279"
---
# <a name="how-to-install-iot-edge-on-kubernetes-preview"></a>Cómo instalar IoT Edge en Kubernetes (versión preliminar)

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

IoT Edge puede integrarse con Kubernetes como un nivel de infraestructura resistente y altamente disponible. Aquí es donde esta compatibilidad encaja en una solución IoT Edge de alto nivel:

![Introducción a k8s](./media/how-to-install-iot-edge-kubernetes/kubernetes-model.png)

>[!TIP]
>Un buen modelo mental para esta integración es pensar en Kubernetes como otro entorno operativo en el que se pueden ejecutar las aplicaciones de IoT Edge, además de Linux y Windows.

## <a name="architecture"></a>Architecture 
En Kubernetes, IoT Edge proporciona una *definición de recursos personalizada* (CRD) para las implementaciones de cargas de trabajo perimetrales. El agente de IoT Edge asume el rol de *controlador de CRD*, que concilia el estado deseado administrado por la nube con el estado del clúster local.

El programador de Kubernetes, que mantiene la disponibilidad del módulo y elige su colocación, administra la duración del módulo. IoT Edge administra la plataforma de aplicaciones perimetrales que se ejecutan en la parte superior, conciliando continuamente el estado deseado especificado en IoT Hub con el estado en el clúster de servidores perimetrales. El modelo de aplicaciones sigue siendo el modelo conocido basado en los módulos y las rutas de IoT Edge. El controlador del agente de IoT Edge realiza la traducción *automática* del modelo de aplicaciones de IoT Edge a construcciones nativas de Kubernetes como, por ejemplo, pods, implementaciones, servicios, etc.

A continuación se muestra un diagrama de arquitectura de alto nivel:

![arquitectura de Kubernetes](./media/how-to-install-iot-edge-kubernetes/publicpreview-refresh-kubernetes.png)

Todos los componentes de la implementación perimetral se limitan a un espacio de nombres de Kubernetes específico del dispositivo, lo que posibilita compartir los mismos recursos de clúster entre varios dispositivos perimetrales y sus implementaciones.

>[!NOTE]
>IoT Edge en Kubernetes se encuentra en [versión preliminar pública](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="tutorials-and-references"></a>Tutoriales y referencias 

Consulte el [minisitio de documentación de la versión preliminar de IoT Edge en Kubernetes](https://aka.ms/edgek8sdoc) para obtener más información, incluidos tutoriales y referencias detallados.
