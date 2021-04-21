---
title: Restauración de un grupo de SQL dedicado
description: Guía para la restauración de un grupo de SQL dedicado eliminado.
author: joannapea
manager: igorstan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql
ms.date: 10/29/2020
ms.author: joanpo
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: f2fb6f809794781559683907a806e6d30ca9bed6
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107567985"
---
# <a name="restore-an-existing-dedicated-sql-pool"></a>Restauración de un grupo de SQL dedicado

En este artículo, aprenderá a restaurar un grupo de SQL dedicado existente en Azure Synapse Analytics mediante Azure Portal y Synapse Studio. Este artículo se aplica a las restauraciones y a las restauraciones geográficas. 

## <a name="restore-an-existing-dedicated-sql-pool-through-the-synapse-studio"></a>Restauración de un grupo de SQL dedicado existente mediante Synapse Studio

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Vaya al área de trabajo de Synapse. 
3. En Introducción > abra Synapse Studio, seleccione **Abrir**.

    ![ Synapse Studio](../media/sql-pools/open-synapse-studio.png)
4. En el panel de navegación izquierdo, seleccione **Datos**.
5. Seleccione **Administrar grupos**. 
6. Seleccione **+ Nuevo** para crear un nuevo grupo de SQL dedicado. 
7. En la pestaña Configuración adicional, seleccione un punto de restauración desde el que restaurar. 

    Si desea realizar una restauración geográfica, seleccione el área de trabajo y el grupo de SQL dedicado que quiere recuperar. 

8. Seleccione **Puntos de restauración automáticos** o **Puntos de restauración definidos por el usuario**. 

    ![Puntos de restauración](../media/sql-pools/restore-point.PNG)

    Si el grupo de SQL dedicado no tiene ningún punto de restauración automática, espere unas horas o cree un punto de restauración definido por el usuario antes de llevar a cabo la restauración. En el caso de los puntos de restauración definidos por el usuario, seleccione uno existente o cree uno nuevo.

    Si va a restaurar una copia de seguridad geográfica, simplemente seleccione el área de trabajo que se encuentra en la región de origen y el grupo de SQL dedicado que quiere restaurar. 

9. Seleccione **Revisar + crear**.

## <a name="restore-an-existing-dedicated-sql-pool-through-the-azure-portal"></a>Restauración de un grupo de SQL dedicado existente mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Vaya al grupo de SQL dedicado desde el que desea restaurar.
3. En la parte superior de la hoja de información general, seleccione **Restaurar**.

    ![ Introducción a la restauración](../media/sql-pools/restore-sqlpool-01.png)

4. Seleccione **Puntos de restauración automáticos** o **Puntos de restauración definidos por el usuario**. 

    Si el grupo de SQL dedicado no tiene ningún punto de restauración automática, espere unas horas o cree un punto de restauración definido por el usuario antes de llevar a cabo la restauración. 

    Si desea realizar una restauración geográfica, seleccione el área de trabajo y el grupo de SQL dedicado que quiere recuperar. 

5. Seleccione **Revisar + crear**.

## <a name="next-steps"></a>Pasos siguientes

- [Crear un punto de restauración](sqlpool-create-restore-point.md)
