---
title: Creación de un punto de restauración definido por el usuario para un grupo de SQL dedicado
description: Obtenga información sobre cómo usar Azure Portal para crear un punto de restauración definido por el usuario para un grupo de SQL dedicado en Azure Synapse Analytics.
author: joannapea
manager: igorstan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 10/29/2020
ms.author: joanpo
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: b695f6c7aabc21541fcc48efed54bbecd022f65a
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107567934"
---
# <a name="user-defined-restore-points"></a>Puntos de restauración definidos por el usuario

En este artículo aprenderá a crear un punto de restauración definido por el usuario para un grupo de SQL dedicado en Azure Synapse Analytics mediante Azure Portal.

## <a name="create-user-defined-restore-points-through-the-azure-portal"></a>Creación de puntos de restauración definidos por el usuario mediante Azure Portal

Los puntos de restauración definidos por el usuario también se pueden crear mediante Azure Portal.

1. Inicie sesión en su cuenta de [Azure Portal](https://portal.azure.com/).

2. Vaya al grupo de SQL dedicado para el que quiere crear un punto de restauración.

3. Seleccione **Información general** en el panel izquierdo y **+ New restore point** (Nuevo punto de restauración). Si el botón Nuevo punto de restauración no está habilitado, asegúrese de que el grupo de SQL dedicado no está en pausa.

    ![Nuevo punto de restauración](../media/sql-pools/create-sqlpool-restore-point-01.png)

4. Especifique el nombre del punto de restauración definido por el usuario y haga clic en **Aplicar**. Los puntos de restauración definidos por el usuario tienen un período de retención predeterminado de siete días.

    ![Nombre de punto de restauración](../media/sql-pools/create-sqlpool-restore-point-02.png)

## <a name="next-steps"></a>Pasos siguientes

- [Restauración de un grupo de SQL dedicado](restore-sql-pool.md)

