---
title: Conexión de una cuenta de Azure Purview 
description: Conecte una cuenta de Azure Purview a un área de trabajo de Synapse.
author: julieMSFT
ms.service: synapse-analytics
ms.subservice: ''
ms.topic: quickstart
ms.date: 12/16/2020
ms.author: jrasnick
ms.reviewer: jrasnick
ms.openlocfilehash: ad9250d022f02db06c9c5251c1fccfbfb730af67
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107567883"
---
# <a name="quickstartconnect-an-azure-purview-account-to-a-synapse-workspace"></a>Inicio rápido: Conexión de una cuenta de Azure Purview a un área de trabajo de Synapse. 


En este inicio rápido, registrará una cuenta de Azure Purview en un área de trabajo de Synapse. Esa conexión le permite detectar recursos de Azure Purview e interactuar con ellos mediante las funcionalidades de Synapse. 

Puede realizar las siguientes tareas en Synapse: 
- Usar el cuadro de búsqueda de la parte superior para buscar recursos de Purview basados en palabras clave. 
- Comprender los datos en función de los metadatos, linaje y anotaciones. 
- Conectar esos datos al área de trabajo con servicios vinculados o conjuntos de datos de integración. 
- Analizar esos conjuntos de datos con Synapse Apache Spark, Synapse SQL y Data Flow 

## <a name="prerequisites"></a>Requisitos previos 
- [Cuenta de Azure Purview](../../purview/create-catalog-portal.md) 
- [Área de trabajo de Synapse](../quickstart-create-workspace.md) 

## <a name="signin-toa-synapse-workspace"></a>Inicie sesión en un área de trabajo de Synapse 

Vaya a  [https://web.azuresynapse.net](https://web.azuresynapse.net) e inicie sesión en el área de trabajo. 

## <a name="permissions-for-connecting-an-azure-purview-account"></a>Permisos para conectar una cuenta de Azure Purview 

- Para conectar una cuenta de Azure Purview a un área de trabajo de Synapse, necesita un rol **Colaborador** en dicha área otorgado por la Administración de identidad y acceso de Azure Portal y necesita acceso a esa cuenta de Azure Purview. Para obtener más información, consulte [permisos de Azure Purview](../../purview/catalog-permissions.md).

## <a name="connect-an-azure-purview-account"></a>Conexión de una cuenta de Azure Purview  

- En el área de trabajo de Synapse, vaya a **Administrar** -> **Azure Purview**. Seleccione **Connect to a Purview account** (Conectarse a una cuenta de Purview). 
- Puede elegir **A partir de una suscripción de Azure** o **Especificar manualmente**. En **A partir de una suscripción de Azure**, puede seleccionar la cuenta a la que tiene acceso. 
- Una vez conectado, debería poder ver el nombre de la cuenta de Purview en la pestaña **Cuenta de Azure Purview**. 
- Puede usar la barra de búsqueda de la parte superior central del área de trabajo de Synapse para buscar datos. 

## <a name="nextsteps"></a>Pasos siguientes 

[Registro y análisis de recursos de Azure Synapse en Azure Purview](../../purview/register-scan-azure-synapse-analytics.md)

[Detección, conexión y exploración de datos en Synapse con Azure Purview](how-to-discover-connect-analyze-azure-purview.md)   
