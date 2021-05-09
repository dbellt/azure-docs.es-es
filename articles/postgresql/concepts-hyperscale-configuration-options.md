---
title: Opciones de configuración de Hiperescala (Citus) para Azure Database for PostgreSQL
description: Opciones para un grupo de servidores Hiperescala (Citus), incluidas las regiones, el almacenamiento y el proceso de nodos.
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.custom: references_regions
ms.date: 04/29/2021
ms.openlocfilehash: 60f52a11aa1573e4d05bcdc1dd8308ec3cc96dac
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108316144"
---
# <a name="azure-database-for-postgresql--hyperscale-citus-configuration-options"></a>Opciones de configuración de Hiperescala (Citus) para Azure Database for PostgreSQL

## <a name="compute-and-storage"></a>Compute y Storage
 
Puede seleccionar la configuración de proceso y almacenamiento por separado para los nodos de trabajo y el nodo de coordinación en un grupo de servidores Hiperescala (Citus).  Los recursos de proceso se proporcionan como núcleos virtuales, que representan la CPU lógica del hardware subyacente. El tamaño de almacenamiento para el aprovisionamiento se refiere a la capacidad disponible para los nodos de trabajo y de coordinación en el grupo de servidores Hiperescala (Citus). El almacenamiento incluye archivos de base de datos, archivos temporales, registros de transacciones y registros de servidor PostgreSQL.

### <a name="standard-tier"></a>Nivel estándar
 
| Recurso              | Nodo de trabajo           | Nodo de coordinación      |
|-----------------------|-----------------------|-----------------------|
| Proceso, núcleos virtuales       | 4, 8, 16, 32, 64      | 4, 8, 16, 32, 64      |
| Memoria por núcleo virtual, GiB | 8                     | 4                     |
| Tamaño de almacenamiento, TiB     | 0,5, 1, 2             | 0,5, 1, 2             |
| Tipo de almacenamiento          | Uso general (SSD) | Uso general (SSD) |
| E/S                  | Hasta 3 IOPS/GiB      | Hasta 3 IOPS/GiB      |

La cantidad total de memoria RAM en un solo nodo de Hiperescala (Citus) se basa en el número seleccionado de núcleos virtuales.

| Núcleos virtuales | Un nodo de trabajo, GiB de RAM | Nodo de coordinación, GiB de RAM |
|--------|--------------------------|---------------------------|
| 4      | 32                       | 16                        |
| 8      | 64                       | 32                        |
| 16     | 128                      | 64                        |
| 32     | 256                      | 128                       |
| 64     | 432                      | 256                       |

La cantidad total de almacenamiento que se aprovisiona también define la capacidad de E/S disponible para cada nodo de coordinación y de trabajo.

| Tamaño de almacenamiento, TiB | Número máximo de IOPS |
|-------------------|--------------|
| 0.5               | 1536        |
| 1                 | 3072        |
| 2                 | 6148        |

Para todo el clúster de Hiperescala (Citus), las IOPS agregadas alcanzan los siguientes valores:

| Nodos de trabajo | 0,5 TiB, total de IOPS | 1 TiB, total de IOPS | 2 TiB, total de IOPS |
|--------------|---------------------|-------------------|-------------------|
| 2            | 3072               | 6144             | 12 296            |
| 3            | 4608               | 9216             | 18 444            |
| 4            | 6144               | 12 288            | 24 592            |
| 5            | 7680               | 15 360            | 30 740            |
| 6            | 9216               | 18 432            | 36 888            |
| 7            | 10 752              | 21 504            | 43 036            |
| 8            | 12 288              | 24 576            | 49 184            |
| 9            | 13 824              | 27 648            | 55 332            |
| 10           | 15 360              | 30 720            | 61 480            |
| 11           | 16 896              | 33 792            | 67 628            |
| 12           | 18 432              | 36 864            | 73 776            |
| 13           | 19 968              | 39 936            | 79 924            |
| 14           | 21 504              | 43 008            | 86 072            |
| 15           | 23 040              | 46 080            | 92 220            |
| 16           | 24 576              | 49 152            | 98 368            |
| 17           | 26 112              | 52 224            | 104 516           |
| 18           | 27 648              | 55 296            | 110 664           |
| 19           | 29 184              | 58 368            | 116 812           |
| 20           | 30 720              | 61 440            | 122 960           |

### <a name="basic-tier-preview"></a>Nivel Básico (versión preliminar)

> [!IMPORTANT]
> El nivel Básico de Hiperescala (Citus) está actualmente en versión preliminar.  Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
>
> Puede ver una lista completa de otras características nuevas en [Características en versión preliminar de Hiperescala (Citus) de PostgreSQL](hyperscale-preview-features.md).

El [nivel Básico](concepts-hyperscale-tiers.md) es un grupo de servidores con solo un nodo.  Dado que no hay una distinción entre los nodos de coordinación y de trabajo, es menos complicado elegir recursos de proceso y almacenamiento.

| Resource              | Opciones disponibles     |
|-----------------------|-----------------------|
| Proceso, núcleos virtuales       | 2, 4, 8               |
| Memoria por núcleo virtual, GiB | 4                     |
| Tamaño del almacenamiento, GiB     | 128, 256, 512         |
| Tipo de almacenamiento          | Uso general (SSD) |
| E/S                  | Hasta 3 IOPS/GiB      |

La cantidad total de memoria RAM en un solo nodo de Hiperescala (Citus) se basa en el número seleccionado de núcleos virtuales.

| Núcleos virtuales | GiB RAM |
|--------|---------|
| 2      | 8       |
| 4      | 16      |
| 8      | 32      |

La cantidad total de almacenamiento que se aprovisiona también define la capacidad de E/S disponible para el nodo de nivel Básico.

| Tamaño del almacenamiento, GiB | Número máximo de IOPS |
|-------------------|--------------|
| 128               | 384          |
| 256               | 768          |
| 512               | 1536        |

## <a name="regions"></a>Regions
Los grupos de servidores Hiperescala (Citus) están disponibles en las siguientes regiones de Azure:

* América:
    * Sur de Brasil
    * Centro de Canadá
    * Centro de EE. UU.
    * Este de EE. UU.*
    * Este de EE. UU. 2
    * Centro-Norte de EE. UU
    * Oeste de EE. UU. 2
* Asia Pacífico:
    * Este de Australia
    * Japón Oriental
    * Centro de Corea del Sur
    * Sudeste de Asia
* Europa:
    * Centro de Francia
    * Norte de Europa
    * Norte de Suiza
    * Sur de Reino Unido
    * Oeste de Europa

(\* = admite [características en versión preliminar](hyperscale-preview-features.md))

Es posible que algunas de estas regiones no se activen inicialmente en todas las suscripciones de Azure. Si desea utilizar una región de la lista anterior y no la ve en su suscripción, o si desea usar una región que no está en esta lista, abra una [solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="pricing"></a>Precios
Para conocer la información más actualizada sobre precios, consulte la [página de precios](https://azure.microsoft.com/pricing/details/postgresql/) del servicio.
Para ver el costo de la configuración deseada, en [Azure Portal](https://portal.azure.com/#create/Microsoft.PostgreSQLServer) se muestra el costo mensual en la pestaña **Configurar** según las opciones que seleccione. Si no tiene una suscripción de Azure, puede usar la calculadora de precios de Azure para obtener un precio estimado. En el sitio web [Calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/), seleccione **Agregar elementos**, expanda la categoría **Bases de datos** y elija **Azure Database for PostgreSQL: Hiperescala (Citus)** para personalizar las opciones.
 
## <a name="next-steps"></a>Pasos siguientes
Obtenga información sobre cómo [crear un grupo de servidores Hiperescala (Citus) en el portal](quickstart-create-hyperscale-portal.md).
