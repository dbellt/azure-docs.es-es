---
title: 'Inicio rápido: Requisitos previos del sistema'
description: En este inicio rápido obtendrá los requisitos previos del sistema necesarios para ejecutar Azure defender para IoT.
ms.date: 11/30/2020
ms.topic: quickstart
ms.openlocfilehash: 2aae849b6ee772c2aa29c680f3b107af3ed600b0
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106382953"
---
# <a name="quickstart-system-prerequisites"></a>Inicio rápido: Requisitos previos del sistema

En este artículo se enumeran los requisitos previos del sistema necesarios para ejecutar Azure Defender para IoT.

## <a name="prerequisites"></a>Requisitos previos

- Ninguno

## <a name="minimum-requirements"></a>Requisitos mínimos

- Conmutadores de red que admitan la supervisión del tráfico a través del puerto SPAN.
- Dispositivos de hardware para sensores NTA.
- El rol Colaborador en la suscripción de Azure. Solo se requiere durante la incorporación para definir los dispositivos confirmados y la conexión a Azure Sentinel.
- Rol **Colaborador** en IoT Hub (niveles Gratis o Estándar), para la administración conectada en la nube. Asegúrese de que la característica **Azure Defender para IoT** está habilitada.
- En cuanto a la compatibilidad con Defender-IoT-micro-agent en el nivel de dispositivo, los agentes de Defender para IoT admiten una lista cada vez mayor de dispositivos y plataformas. Consulte la [lista de plataformas admitidas](how-to-deploy-agent.md).

## <a name="supported-service-regions"></a>Regiones de servicio admitidas

Defender para IoT enruta todo el tráfico de todas las regiones europeas hasta el centro de datos regional de Oeste de Europa. Enruta el tráfico de las regiones restantes al centro de datos regional de Centro de EE. UU.

Para más información, consulte [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=iot-hub).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Identificación de los dispositivos necesarios](how-to-identify-required-appliances.md)
