---
title: Deshabilitación de copias de seguridad de replicación geográfica
description: Guía de procedimientos para deshabilitar copias de seguridad geográficas para un grupo de SQL dedicado (anteriormente SQL DW) en Azure Synapse Analytics
services: synapse-analytics
author: joannapea
manager: igorstan
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 01/06/2021
ms.author: joanpo
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 2cebee3ad9b515c6f40529fe5d25da687fd53687
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109786292"
---
# <a name="disable-geo-backups-for-a-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Deshabilitación de copias de seguridad geográficas para un [grupo de SQL dedicado (anteriormente SQL DW)](sql-data-warehouse-overview-what-is.md) en Azure Synapse Analytics

En este artículo, aprenderá a deshabilitar copias de seguridad geográficas para un grupo de SQL dedicado (anteriormente SQL DW) mediante Azure Portal.

## <a name="disable-geo-backups-through-azure-portal"></a>Deshabilitación de copias de seguridad geográficas mediante Azure Portal

Siga estos pasos para deshabilitar copias de seguridad geográficas de un grupo de SQL dedicado (anteriormente SQL DW):

> [!NOTE]
> Si deshabilita las copias de seguridad geográficas, ya no podrá recuperar el grupo de SQL dedicado (anteriormente SQL DW) en otra región de Azure. 
> 

1. Inicie sesión en su cuenta de [Azure Portal](https://portal.azure.com/).
1. Seleccione el recurso de grupo de SQL dedicado (anteriormente SQL DW) para el que desea deshabilitar las copias de seguridad geográficas. 
1. En **Configuración**, en el panel de navegación izquierdo, seleccione **Directiva de copia de seguridad geográfica**.

   ![Deshabilitación de copias de seguridad geográficas](./media/sql-data-warehouse-restore-from-geo-backup/disable-geo-backup-1.png)

1. Para deshabilitar copias de seguridad geográficas, seleccione **Deshabilitado**. 

   ![Copia de seguridad geográfica deshabilitada](./media/sql-data-warehouse-restore-from-geo-backup/disable-geo-backup-2.png)

1. Seleccione *Guardar* para asegurarse de que se guarda la configuración. 

   ![Guardado de la configuración de copias de seguridad geográficas](./media/sql-data-warehouse-restore-from-geo-backup/disable-geo-backup-3.png)

## <a name="next-steps"></a>Pasos siguientes

- [Restauración de un grupo de SQL dedicado existente (anteriormente SQL DW)](sql-data-warehouse-restore-active-paused-dw.md)
- [Restauración de un grupo de SQL dedicado eliminado (anteriormente SQL DW)](sql-data-warehouse-restore-deleted-dw.md)
