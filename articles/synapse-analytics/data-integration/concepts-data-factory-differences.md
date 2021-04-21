---
title: Diferencias con respecto a Azure Data Factory
description: Aprenda cómo las funcionalidades de integración de datos de Azure Synapse Analytics difieren de las de Azure Data Factory
services: synapse-analytics
author: kromerm
ms.service: synapse-analytics
ms.subservice: pipeline
ms.topic: conceptual
ms.date: 12/10/2020
ms.author: makromer
ms.reviewer: jrasnick
ms.openlocfilehash: 144bdf5e94f753090dd73e5839b6c1fd25f11811
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107567645"
---
# <a name="data-integration-in-azure-synapse-analytics-versus-azure-data-factory"></a>Integración de datos en Azure Synapse Analytics frente a Azure Data Factory

En Azure Synapse Analytics, las funcionalidades de integración de datos, como los flujos de datos y las canalizaciones de Synapse, se basan en las de Azure Data Factory. Para más información, vea [¿Qué es Azure Data Factory?](../../data-factory/introduction.md).


## <a name="available-features-in-adf--azure-synapse-analytics"></a>Características disponibles en ADF y Azure Synapse Analytics

Consulte la tabla siguiente para ver la disponibilidad de las características:

| Category                 | Característica    |  Azure Data Factory  | Azure Synapse Analytics |
| ------------------------ | ---------- | :------------------: | :---------------------: |
| **Integration Runtime**  | Uso de SSIS y SSIS Integration Runtime | ✓ | ✗ |
|                          | Compatibilidad con Integration Runtime entre regiones (flujos de datos) | ✓ | ✗ |
|                          | Uso compartido de Integration Runtime | ✓<br><small>*Puede compartirse entre diferentes factorías de datos* | ✗ |
|                          | Período de vida | ✓ | ✗ |
| **Actividades de canalizaciones** | Actividad del paquete de SSIS | ✓ | ✗ |
|                          | Compatibilidad con la actividad de Power Query | ✓ | ✓ |
| **Galería de plantillas y centro de conocimientos** | Plantillas de solución | ✓<br><small>*Galería de plantillas de Azure Data Factory* | ✓<br><small>*Centro de conocimientos del área de trabajo de Synapse* |
| **Integración del repositorio GIT** | Integración de GIT | ✓ | ✓ |
| **Supervisión**           | Supervisión de trabajos de Spark para el flujo de datos | ✗ | ✓<br><small>*Aproveche los grupos de Spark en Synapse* |
|                          | Integración con Azure Monitor | ✓ | ✗ |
| **Lineage** | Admite la publicación de datos de linaje de canalización en Purview.  | ✓ | ✗ |  

> [!Note]
> **Período de vida** es una configuración Azure Integration Runtime que permite que el clúster de Spark *permanezca semiactivo* durante un período de tiempo después de la ejecución del flujo de datos.
>


## <a name="next-steps"></a>Pasos siguientes

Para empezar a trabajar con la integración de datos en el área de trabajo de Synapse, aprenda cómo [ingerir datos en una cuenta de Azure Data Lake Storage Gen2](data-integration-data-lake.md).
