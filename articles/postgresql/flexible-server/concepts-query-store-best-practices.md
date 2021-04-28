---
title: 'Procedimientos recomendados para usar el Almacén de consultas en Azure Database for PostgreSQL: servidor flexible'
description: 'En este artículo se describen los procedimientos recomendados para usar el Almacén de consultas en Azure Database for PostgreSQL: servidor flexible.'
author: ssen-msft
ms.author: ssen
ms.service: postgresql
ms.topic: conceptual
ms.date: 04/01/2021
ms.openlocfilehash: 9ee2dc4b3e8ea6a9f892adbc378e8e13b8bd8160
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107558803"
---
# <a name="best-practices-for-query-store---flexible-server"></a>Procedimientos recomendados para usar el Almacén de consultas: servidor flexible

**Se aplica a:** Azure Database for PostgreSQL: servidor flexible, versiones 11 y 12

En este artículo se describen los procedimientos recomendados para usar Almacén de consultas en Azure Database for PostgreSQL.

## <a name="set-the-optimal-query-capture-mode"></a>Establecimiento del modo de captura de consultas óptimo
Permite que Almacén de consultas capture los datos realmente relevantes. 

|**pg_qs.query_capture_mode** | **Escenario**|
|---|---|
|_Todo_  |Analice la carga de trabajo exhaustivamente en todas las consultas, así como sus frecuencias de ejecución y otras estadísticas. Identifique nuevas consultas en la carga de trabajo. Detecte si las consultas ad-hoc se usan para identificar oportunidades de parametrización automática o manual. _All_ (Todas) aumenta el costo de consumo de recursos. |
|_Top_ (Principales)  |Centre la atención en las consultas principales; las emitidos por los clientes.
|_None_ |Si se establece en Ninguno, el Almacén de consultas no capturará ninguna consulta nueva. Ya ha capturado un conjunto de consultas y el período de tiempo que desea investigar, y también desea eliminar los objetos innecesarios que pueden introducir otras consultas. La opción _None_ (Ninguna) es adecuada para entornos de pruebas y evaluación comparativa. Además, _None_ (Ninguna) debe usarse con precaución, ya que podría perder la oportunidad de realizar un seguimiento de las consultas nuevas importantes y de optimizarlas. |


> [!NOTE] 
> **pg_qs.query_capture_mode** reemplaza a **pgms_wait_sampling.query_capture_mode**. Si el valor de pg_qs.query_capture_mode es _None_ (Ninguna), la configuración de pgms_wait_sampling.query_capture_mode no tiene ningún efecto. 


## <a name="keep-the-data-you-need"></a>Conservación de los datos necesarios
El parámetro **pg_qs.retention_period_in_days** especifica en días el período de retención de datos para Almacén de consultas. Se eliminan los datos de consulta y las estadísticas anteriores. De forma predeterminada, Almacén de consultas está configurado para conservar los datos durante siete días. Evite mantener datos históricos que no va a usar. Aumente el valor si necesita mantener los datos más tiempo.


## <a name="next-steps"></a>Pasos siguientes
- Obtenga información sobre cómo obtener o establecer los parámetros mediante [Azure Portal](howto-configure-server-parameters-using-portal.md) o la [CLI de Azure](howto-configure-server-parameters-using-cli.md).
