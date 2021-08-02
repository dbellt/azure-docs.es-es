---
title: Colaboración en Microsoft Teams con un equipo de incidentes de Azure Sentinel | Microsoft Docs
description: Aprenda a conectarse a Microsoft Teams desde Azure Sentinel para colaborar con otros usuarios del equipo mediante los datos de Azure Sentinel.
services: sentinel
documentationcenter: na
author: batamig
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/03/2021
ms.author: bagol
ms.openlocfilehash: fed86c1982e78c527c211d4f4ec530386e13425a
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110796902"
---
# <a name="collaborate-in-microsoft-teams-public-preview"></a>Colaboración en Microsoft Teams (versión preliminar pública)

Azure Sentinel admite una integración directa con [Microsoft Teams,](/microsoftteams/) lo que le permite pasar directamente al trabajo en equipo en incidentes específicos.


> [!IMPORTANT]
> La integración con Microsoft Teams se encuentra actualmente en **VERSIÓN PRELIMINAR**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

## <a name="overview"></a>Información general

La integración con Microsoft Teams directamente desde Azure Sentinel permite a los equipos colaborar sin problemas en toda la organización y colaborar con partes interesadas externas.

Use Microsoft Teams con un *equipo de incidentes* de Azure Sentinel para centralizar la comunicación y la coordinación entre el personal correspondiente. Los equipos de incidentes son especialmente útiles cuando se usan como un puente de conferencia dedicado para incidentes continuos de alta gravedad.

Las organizaciones que ya usan Microsoft Teams para la comunicación y la colaboración pueden usar la integración de Azure Sentinel para aportar datos de seguridad directamente a sus conversaciones y al trabajo diario. 

Un equipo de incidentes dispone siempre de los datos más actualizados y recientes de Azure Sentinel, lo que garantiza que los equipos tengan a mano los datos más importantes.

## <a name="use-an-incident-team-to-investigate"></a>Uso de un equipo de incidentes para investigar

Para investigar de forma conjunta con un *equipo de incidentes* integre Microsoft Teams directamente desde el incidente.

**Para crear el equipo de incidentes**:

1. En Azure Sentinel, en la cuadrícula **Administración de amenazas** > **Incidentes**, seleccione el incidente que está investigando actualmente.

1. En la parte inferior del panel de incidentes que aparece a la derecha, seleccione **Acciones** > **Crear equipo**.

    [ ![Cree un equipo para colaborar en un equipo de incidentes.](media/collaborate-in-microsoft-teams/create-team.png) ](media/collaborate-in-microsoft-teams/create-team.png#lightbox)

    El panel **Nuevo equipo** se abre a la derecha. Defina la siguiente configuración del equipo de incidentes:

    - **Nombre del equipo**: se define automáticamente como el nombre del incidente. Modifique el nombre según sea necesario para que pueda identificarlo fácilmente.
    - **Descripción**: escriba una descripción significativa del equipo de incidentes.
    - **Incorporación de grupos:** seleccione uno o varios grupos de Azure AD para agregarlos al equipo de incidentes. No se admiten usuarios individuales.

        > [!TIP]
        > Si trabaja con regularidad con los mismos equipos, puede seleccionar la estrella :::image type="icon" source="media/collaborate-in-microsoft-teams/save-as-favorite.png" border="false"::: para guardarlos como favoritos.
        >
        > Los favoritos se seleccionarán automáticamente la próxima vez que cree un equipo. Si quiere quitarlo del siguiente equipo que cree, seleccione **Eliminar** :::image type="icon" source="media/collaborate-in-microsoft-teams/delete-user-group.png" border="false"::: o vuelva a seleccionar la estrella :::image type="icon" source="media/collaborate-in-microsoft-teams/save-as-favorite.png" border="false"::: para quitar el equipo de sus favoritos por completo.
        >

1. Cuando haya terminado de agregar grupos, seleccione **Crear** para crear el equipo de incidentes.

    El panel de incidentes se actualiza, con un vínculo al nuevo equipo de incidentes bajo el título **Nombre del equipo**.

    [ ![Haga clic en el vínculo de integración de Teams agregado al incidente.](media/collaborate-in-microsoft-teams/teams-link-added-to-incident.jpg) ](media/collaborate-in-microsoft-teams/teams-link-added-to-incident.jpg#lightbox)


1. Seleccione el vínculo de **integración de Teams** para cambiar a Microsoft Teams, donde todos los datos sobre el incidente aparecen en la pestaña de la página **Incidente**.

    [ ![Página Incidente en Microsoft Teams.](media/collaborate-in-microsoft-teams/incident-in-teams.jpg) ](media/collaborate-in-microsoft-teams/incident-in-teams.jpg#lightbox)

Continúe la conversación sobre la investigación en Teams durante el tiempo que sea necesario. Tiene los detalles completos del incidente directamente en los equipos.

> [!TIP]
> Al [cerrar un incidente](tutorial-investigate-cases.md#closing-an-incident), se archiva el equipo de incidentes relacionado que ha creado en Microsoft Teams.
>
> Si el incidente se vuelve a abrir, el equipo de incidentes relacionado también se volverá a abrir en Microsoft Teams para que pueda continuar la conversación, justo donde lo dejó.
>

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte:

- [Tutorial: Investigación de incidentes con Azure Sentinel](tutorial-investigate-cases.md)
- [Introducción a los equipos y canales de Microsoft Teams](/microsoftteams/teams-channels-overview/)
