---
title: archivo de inclusión
description: archivo de inclusión
services: data-factory
author: nabhishek
ms.service: data-factory
ms.topic: include
ms.date: 10/09/2019
ms.author: abnarain
ms.openlocfilehash: 626074940ced57bdb1ae93f494b7a3847ef83a81
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96444991"
---
| Nombres de dominio                  | Puertos de salida | Descripción                              |
| ----------------------------- | -------------- | ---------------------------------------- |
| `*.servicebus.windows.net`    | 443            | Lo necesita el entorno de ejecución de integración autohospedado para la creación interactiva. |
| `{datafactory}.{region}.datafactory.azure.net`<br> o bien `*.frontend.clouddatahub.net` | 443            | El entorno de ejecución de integración autohospedado lo necesita para conectarse al servicio Data Factory. <br>En el caso de los nuevos Data Factory creados, busque el nombre de dominio completo de la clave del entorno de ejecución de integración autohospedado que se encuentra en el formato {datafactory}.{region}.datafactory.azure.net. En el caso de Data Factory anterior, si no ve el FQDN en la clave del entorno de ejecución de integración autohospedado, use *.frontend.clouddatahub.net en su lugar. |
| `download.microsoft.com`    | 443            | Lo necesita el entorno de ejecución de integración autohospedado para descargar las actualizaciones. Si ha deshabilitado la actualización automática, puede omitir la configuración de este dominio. |
| `*.core.windows.net`          | 443            | Lo usa el entorno de ejecución de integración autohospedado para conectarse a la cuenta de Azure Storage cuando se utiliza la característica [Copia almacenada provisionalmente](../articles/data-factory/copy-activity-performance.md#staged-copy). |
| `*.database.windows.net`      | 1433           | Solo es obligatorio cuando se copia desde o en Azure SQL Database o Azure Synapse Analytics. En caso contrario, es opcional. Use la característica de copia almacenada provisionalmente para copiar datos en SQL Database o Azure Synapse Analytics sin abrir el puerto 1433. |
| `*.azuredatalakestore.net`<br>`login.microsoftonline.com/<tenant>/oauth2/token`    | 443            | Solo es necesario cuando se copia desde o en Azure Data Lake Store, sino es opcional. |
