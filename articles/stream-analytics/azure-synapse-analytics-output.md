---
title: Salida de Azure Synapse Analytics desde Azure Stream Analytics
description: En este artículo se describe Azure Synapse Analytics como salida para Azure Stream Analytics.
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 6c61f378dd9121c727fc245d177e11921a8a8e26
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110094391"
---
# <a name="azure-synapse-analytics-output-from-azure-stream-analytics"></a>Salida de Azure Synapse Analytics desde Azure Stream Analytics

[Azure Synapse Analytics](https://azure.microsoft.com/services/synapse-analytics) es un servicio de análisis ilimitado que combina el almacenamiento de datos empresariales y el análisis de macrodatos. 

Los trabajos de Azure Stream Analytics se pueden generar en una tabla de grupos de SQL dedicados en Azure Synapse Analytics y pueden procesar velocidades de rendimiento de hasta 200 MB/s. Esta capacidad satisface las necesidades más exigentes de análisis en tiempo real y procesamiento de datos de ruta de acceso activa para cargas de trabajo de generación de informes y creación de paneles, entre otras.  

La tabla del grupo de SQL dedicado debe existir antes de poder agregarla como salida al trabajo de Stream Analytics. El esquema de tabla debe coincidir con los campos y los tipos en la salida del trabajo. 

> [!NOTE] 
> Para usar Azure Synapse Analytics como salida, asegúrese de que la cuenta de almacenamiento está configurada en el nivel de trabajo, no en el nivel de salida. Para cambiar la configuración de la cuenta de almacenamiento, en el menú **Configure** (Configurar) del trabajo de Stream Analytics, vaya a **Storage account settings** (Configuración de cuenta de almacenamiento). Use solo los tipos de cuenta de almacenamiento que admiten tablas: De uso general V2 y De uso general V1. Elija solo el nivel Estándar. El nivel Prémium no se admite en este escenario.

## <a name="output-configuration"></a>Configuración de salida

En la siguiente tabla se enumeran los nombres de propiedad y su descripción para crear una salida de Azure Synapse Analytics.

|Nombre de propiedad|Descripción|
|-|-|
|Alias de salida |Un nombre descriptivo usado en las consultas para dirigir la salida de la consulta a esta base de datos. |
|Base de datos |Nombre del grupo de SQL dedicado adonde se envía la salida. |
|Nombre de servidor |Nombre del servidor de Azure Synapse.  |
|Nombre de usuario |Nombre de usuario que tiene acceso de escritura a la base de datos. Stream Analytics admite solo la autenticación de SQL. |
|Contraseña |La contraseña para conectarse a la base de datos. |
|Tabla  | El nombre de la tabla donde se escribe la salida. El nombre de la tabla distingue mayúsculas de minúsculas. El esquema de esta tabla debe coincidir exactamente con el número y tipo de los campos que genera la salida del trabajo.|

## <a name="next-steps"></a>Pasos siguientes

* [Uso de identidades administradas para acceder a Azure SQL Database o Azure Synapse Analytics desde un trabajo de Azure Stream Analytics (versión preliminar)](sql-database-output-managed-identity.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)
