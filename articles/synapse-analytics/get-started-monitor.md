---
title: 'Tutorial: Introducción a Azure Synapse Analytics: supervisión del área de trabajo de Synapse'
description: En este tutorial, aprenderá a supervisar actividades en el área de trabajo de Synapse.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: monitoring
ms.topic: tutorial
ms.date: 12/31/2020
ms.openlocfilehash: e2038ce9d14f4a80cf1a67321f3991c6d6f88bc4
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111558393"
---
# <a name="monitor-your-synapse-workspace"></a>Supervisión del área de trabajo de Synapse

En este tutorial, aprenderá a supervisar actividades en el área de trabajo de Synapse. Puede supervisar las actividades actuales e históricas de SQL, Apache Spark y Pipelines. 

## <a name="introduction-to-the-monitor-hub"></a>Introducción al centro Supervisar

Abra Synapse Studio y vaya al centro **Supervisar**. Aquí, puede ver un historial de todas las actividades que se realizan en el área de trabajo y las que están activas ahora. 

* En **Integration** (Integración), puede supervisar las canalizaciones, los desencadenadores y los entornos de ejecución de integración.
* En **Actividades**, puede supervisar las actividades de Spark y SQL. 

## <a name="integration"></a>Integración

1. Vaya a **Integration > Pipeline runs** (Integración > Ejecuciones de canalización). En esta vista, puede ver cada vez que se ha ejecutado una canalización en el área de trabajo. 
1. Busque la canalización que ejecutó en el paso anterior y haga clic en el **Nombre de canalización** para ver los detalles.
1. Haga clic en **Barra de ruta de navegación** cerca de la parte superior de Synapse Studio, haga clic en **Todas las ejecuciones de la canalización** para volver a la vista anterior.

## <a name="apache-spark-activities"></a>Actividades de Apache Spark

1. Vaya a **Actividades > Aplicaciones de Apache Spark**. Ahora puede ver todas las aplicaciones de Spark que se están ejecutando o que se han ejecutado en el área de trabajo.
1. Busque una aplicación que ya no esté en ejecución y haga clic en el **Nombre de aplicación**. Ahora puede ver los detalles de la aplicación de Spark.
1. Si está familiarizado con Apache Spark, puede encontrar la interfaz de usuario del servidor de historial de Apache Spark estándar haciendo clic en **Servidor de historial de Spark**.

## <a name="sql-activities"></a>Actividades de SQL

1. Vaya a **Actividades > Solicitudes de SQL**.
1. En esta vista puede ver las solicitudes de SQL.
1. Seleccione un **grupo** para supervisar con el filtro **Grupo**. Ahora puede ver todas las aplicaciones de SQL que se están ejecutando o que se han ejecutado en el área de trabajo de ese grupo.
1. Busque una solicitud de SQL específica y haga clic en el vínculo **Más** para ver el texto completo de la solicitud de SQL.

    > [!NOTE] 
    > Las solicitudes de SQL enviadas mediante Synapse Studio en un área de trabajo habilitada para un grupo de SQL dedicado (anteriormente SQL DW) se pueden ver en el centro de supervisión. Para todas las demás actividades de supervisión, puede ir a la supervisión de grupos de SQL dedicados de Azure Portal (anteriormente SQL DW).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Exploración del Centro de conocimientos](get-started-knowledge-center.md)
