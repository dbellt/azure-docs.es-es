---
title: Conexión a GitHub
description: Uso de GitHub para especificar referencias de entidad de Common Data Model
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 06/03/2020
ms.author: jingwang
ms.openlocfilehash: 6446bb9e18a8ce9e69c2462570d06154ed4ed8ac
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107904454"
---
# <a name="use-github-to-read-common-data-model-entity-references"></a>Uso de GitHub para leer referencias de entidad de Common Data Model

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

El conector de GitHub en Azure Data Factory sirve únicamente para recibir el esquema de referencia de entidades para el formato de [Common Data Model](format-common-data-model.md) en el flujo de datos de asignación.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de GitHub.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en **GitHub**. | sí
| userName | Nombre de usuario de GitHub | sí |
| password | Contraseña de GitHub | sí |

## <a name="next-steps"></a>Pasos siguientes

Cree un [conjunto de datos de origen](data-flow-source.md) en el flujo de datos de asignación.