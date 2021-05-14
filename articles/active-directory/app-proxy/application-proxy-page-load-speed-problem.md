---
title: Una aplicación de Azure Active Directory Application Proxy tarda mucho en cargarse
description: Solución de problemas de rendimiento de carga de páginas con Azure Active Directory Application Proxy
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/11/2017
ms.author: kenwith
ms.reviewer: asteen
ms.openlocfilehash: 37f48b824cbae35f844bcb9ad2d03b469ca2bcbf
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108186838"
---
# <a name="an-application-proxy-application-takes-too-long-to-load"></a>Tiempo excesivo de carga de una aplicación con el proxy de aplicación

Este artículo le ayuda a comprender por qué una aplicación de Azure AD Application Proxy puede tardar mucho tiempo en cargarse. También se explica lo que puede hacer para resolver este problema.

## <a name="overview"></a>Información general
Aunque las aplicaciones funcionan, pueden experimentar una latencia elevada. Puede haber ajustes que puede realizar en la topología de red para mejorar la velocidad. Para una evaluación de diferentes topologías, consulte el [documento de consideraciones de red](application-proxy-network-topology.md).

Además de la topología de red, no hay actualmente recomendaciones adicionales sobre el ajuste del rendimiento. A medida que el servicio Application Proxy se expande, podría llegar a un centro de datos que esté físicamente cercano. La mayor proximidad puede ayudar con la latencia. Para obtener una lista de centros de datos de Azure, consulte la [página de prueba de latencia](http://www.azurespeed.com/Azure/Latency). 

## <a name="feedback-on-application-proxy-data-center-locations"></a>Comentarios sobre las ubicaciones de los centros de datos de Proxy de aplicación 
Puede haber centros de datos de Azure que todavía no incluyan Application Proxy, pero que le supongan una mejora importante de la latencia. Envíe la ubicación del centro de datos a aadapfeedback@microsoft.com. Microsoft usa sus comentarios con fines de expansión.

Microsoft está trabajando en funcionalidades adicionales para mejorar la latencia. Tan pronto como estas mejoras estén disponibles, se actualizará la documentación.

## <a name="next-steps"></a>Pasos siguientes
[Trabajo con servidores proxy locales existentes](application-proxy-configure-connectors-with-proxy-servers.md)
