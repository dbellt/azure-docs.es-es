---
title: Conmutación por error y recuperación ante desastres de Azure Video Analyzer for Media (anteriormente Video Indexer)
titleSuffix: Azure Media Services
description: Aprenda cómo realizar la conmutación por error en una cuenta secundaria de Azure Video Analyzer for Media (anteriormente Video Indexer) si se produce un desastre o un error en un centro de datos regional.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.workload: ''
ms.topic: article
ms.custom: ''
ms.date: 07/29/2019
ms.author: juliako
ms.openlocfilehash: e3f1f09f58d79ce3927b137cae20db5e1d55af14
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386859"
---
# <a name="video-analyzer-for-media-failover-and-disaster-recovery"></a>Conmutación por error y recuperación ante desastres de Video Analyzer for Media

Azure Video Analyzer for Media (anteriormente Video Indexer) no proporciona una conmutación por error instantánea del servicio si se produce una interrupción o un error en un centro de centros regional. En este artículo se explica cómo configurar el entorno para una conmutación por error con el fin de garantizar la disponibilidad óptima de las aplicaciones y el tiempo de recuperación mínimo en caso de que se produzca un desastre.

Se recomienda que configure la continuidad empresarial y recuperación ante desastres (BCDR) entre las parejas regionales para beneficiarse de las directivas de aislamiento y disponibilidad de Azure. Para más información, consulte [Regiones emparejadas de Azure](../../best-practices-availability-paired-regions.md).

## <a name="prerequisites"></a>Prerrequisitos

Suscripción a Azure. Si aún no tiene una suscripción de Azure, regístrese para una [evaluación gratuita de Azure](https://azure.microsoft.com/free/).

## <a name="failover-to-a-secondary-account"></a>Conmutación por error a una cuenta secundaria

Para implementar BCDR, debe tener dos cuentas de Video Analyzer for Media para controlar la redundancia.

1. Cree dos cuentas de Video Analyzer for Media conectadas a Azure (consulte [Creación de una cuenta de Video Analyzer for Media](connect-to-azure.md)). Cree una cuenta para la región primaria y otra para la región de Azure emparejada.
1. Si se produce un error en la región primaria, cambie a la indexación mediante la cuenta secundaria.

> [!TIP]
> Puede automatizar los procesos de BCDR mediante la configuración de alertas de registro de actividad para las notificaciones de mantenimiento del servicio, según se explica en [Creación de alertas del registro de actividad en notificaciones del servicio](../../service-health/alerts-activity-log-service-notifications-portal.md).

Para información sobre el uso de varios inquilinos, consulte [Administración de varios inquilinos](manage-multiple-tenants.md). Para implementar BCDR, elija una de estas dos opciones: [Cuenta de Video Analyzer for Media por inquilino](./manage-multiple-tenants.md#video-analyzer-for-media-account-per-tenant) o [Suscripción de Azure por inquilino](./manage-multiple-tenants.md#azure-subscription-per-tenant).

## <a name="next-steps"></a>Pasos siguientes

[Administre una cuenta de Video Analyzer for Media conectada a Azure](manage-account-connected-to-azure.md).
