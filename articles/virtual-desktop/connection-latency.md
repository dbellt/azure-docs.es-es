---
title: 'Latencia de la conexión para usuarios de Windows Virtual Desktop: Azure'
description: Latencia de la conexión para usuarios de Windows Virtual Desktop.
author: Heidilohr
ms.topic: conceptual
ms.date: 10/30/2019
ms.author: helohr
manager: femila
ms.openlocfilehash: 18fc276ec9d95a52c5bb5890cda313fc14fc7c5f
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108144338"
---
# <a name="determine-user-connection-latency-in-windows-virtual-desktop"></a>Determinación de la latencia de la conexión de los usuarios en Windows Virtual Desktop

Windows Virtual Desktop está disponible a nivel mundial. Los administradores pueden crear máquinas virtuales (VM) en cualquier región de Azure que deseen. La latencia de la conexión variará en función de la ubicación de los usuarios y las máquinas virtuales. Los servicios de Windows Virtual Desktop se implementarán continuamente en nuevas zonas geográficas para mejorar la latencia.

La [herramienta Estimador de experiencia de Windows Virtual Desktop](https://azure.microsoft.com/services/virtual-desktop/assessment/) puede ayudarle a determinar la mejor ubicación para optimizar la latencia de las máquinas virtuales. Se recomienda usar la herramienta cada dos o tres meses para asegurarse de que la ubicación óptima no ha cambiado a medida que Windows Virtual Desktop se implementa en nuevas áreas.

## <a name="interpreting-results-from-the-windows-virtual-desktop-experience-estimator-tool"></a>Interpretación de los resultados de la herramienta de estimador de experiencia de Windows Virtual Desktop

En Windows Virtual Desktop, la latencia de hasta 150 ms no debería afectar a la experiencia del usuario que no implica representación o vídeo. Las latencias entre 150 ms y 200 ms deben ser adecuadas para el procesamiento de texto. La latencia por encima de 200 ms puede afectar a la experiencia del usuario. 

Además, la conexión de Windows Virtual Desktop depende de la conexión a Internet de la máquina desde la que el usuario está utilizando el servicio. Los usuarios pueden perder la conexión o experimentar un retraso en la entrada en una de las siguientes situaciones:

 - El usuario no tiene una conexión a Internet local estable y la latencia es superior a 200 ms.
 - La red está saturada o tiene la velocidad limitada.

Le recomendamos que elija ubicaciones de VM que estén lo más cerca posible de sus usuarios. Por ejemplo, si el usuario se encuentra en la India pero la máquina virtual está en Estados Unidos, se producirá una latencia que afectará a la experiencia global del usuario. 

## <a name="azure-front-door"></a>Azure Front Door

Windows Virtual Desktop usa [Azure Front Door](https://azure.microsoft.com/services/frontdoor/) para redirigir la conexión de usuario a la puerta de enlace de Windows Virtual Desktop más cercana en función de la dirección IP de origen. Windows Virtual Desktop siempre usará la puerta de enlace de Windows Virtual Desktop que el cliente elija.

## <a name="next-steps"></a>Pasos siguientes

- Para comprobar cuál es la mejor ubicación de cara a una latencia óptima, consulte la [herramienta Estimador de experiencia de Windows Virtual Desktop](https://azure.microsoft.com/services/virtual-desktop/assessment/).
- Para información sobre los planes de precios, consulte [Precios de Windows Virtual Desktop](https://azure.microsoft.com/pricing/details/virtual-desktop/).
- Para empezar a trabajar con la implementación de Windows Virtual Desktop, consulte [nuestro tutorial](./create-host-pools-azure-marketplace.md).