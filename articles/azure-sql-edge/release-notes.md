---
title: Notas de la versión de Azure SQL Edge
description: Notas de la versión en las que se detallan las novedades y los cambios en las imágenes de Azure SQL Edge.
keywords: notas de la versión SQL Server
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
ms.subservice: ''
author: VasiyaKrishnan
ms.author: vakrishn
ms.reviewer: sstein
ms.date: 11/24/2020
ms.openlocfilehash: 6218715878ec40fecee79f1c93bf2ca1820af007
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106277018"
---
# <a name="azure-sql-edge-release-notes"></a>Notas de la versión de Azure SQL Edge 

En este artículo se describen las novedades y los cambios en cada nueva versión de Azure SQL Edge.

## <a name="azure-sql-edge-103"></a>Azure SQL Edge 1.0.3

Número de versión del motor de SQL: 15.0.2000.1554

### <a name="fixes"></a>Correcciones

- Actualización del entorno en tiempo de ejecución de ONNX a la versión 1.5.3
- Actualización a Microsoft.SqlServer.DACFx versión 150.5084.2
- Correcciones de errores varios  
   
## <a name="azure-sql-edge-102"></a>Azure SQL Edge 1.0.2

Número de versión del motor de SQL: 15.0.2000.1554

### <a name="fixes"></a>Correcciones

- Transmisión de T-SQL  
   - Corrección de la propiedad y los permisos de los objetos de transmisión
   - Mejoras en el registro con la rotación de registros y la adición de prefijos a los registros
   - Azure Stream Analytics: Mejoras en el registro, mejora del código de error y los mensajes de error en los adaptadores 

- ONNX
    - Correcciones de errores en el escenario de consulta paralela y de limpieza de modelo
    - Entorno en tiempo de ejecución de ONNX actualizado a la versión 1.5.1

## <a name="azure-sql-edge-101"></a>Azure SQL Edge 1.0.1

Número de versión del motor de SQL: 15.0.2000.1553

### <a name="whats-new"></a>Novedades

- Permitir las expresiones Date_Bucket definidas en columnas calculadas.

### <a name="fixes"></a>Correcciones

- Corrección de la directiva de retención para quitar una tabla donde la directiva de retención está habilitada con tiempo de espera infinito.
- Compatibilidad con la implementación de DacFx para las características de la directiva de retención y transmisión. 
- Corrección de implementación de DacFx para habilitar la implementación desde la carpeta anidada en una dirección URL de SAS. 
- Corrección de PREDICT para admitir nombres de columna largos en mensajes de error

## <a name="azure-sql-edge-100-rtm"></a>Azure SQL Edge 1.0.0 (RTM)

Número de versión del motor de SQL: 15.0.2000.1552

### <a name="whats-new"></a>Novedades
- Imágenes de contenedor basadas en Ubuntu 18.04 
- Compatibilidad con la sintaxis `IGNORE NULL` y `RESPECT NULL` con funciones `LAST_VALUE()` y `FIRST_VALUE()`. 
- Mejoras de confiabilidad de PREDICT con ONNX.
- Compatibilidad con la limpieza basada en la directiva de retención de datos:
   - Compatibilidad del búfer en anillo con la tarea de limpieza de la retención para la solución de problemas.
- Compatibilidad de nuevas características: 
   - Recuperación rápida
   - Optimización automática de consultas.
   - Escenarios de ejecución en paralelo.
- Mejoras de ahorro de energía en modo de bajo consumo.
- Compatibilidad con la transmisión de nuevas características. 
   - [Ventanas de instantánea](/stream-analytics-query/snapshot-window-azure-stream-analytics): nuevo tipo de ventana que permite agrupar por los eventos que llegan al mismo tiempo.
   - [TopOne](/stream-analytics-query/topone-azure-stream-analytics) y [CollectTop](/stream-analytics-query/collecttop-azure-stream-analytics) pueden habilitarse como funciones analíticas. Puede devolver registros ordenados por la columna que quiera. No es necesario que formen parte de una ventana. 
   - Mejoras a [MATCH_RECOGNIZE](/stream-analytics-query/match-recognize-stream-analytics). 

### <a name="fixes"></a>Correcciones
- Mensajes de error adicionales y detalles para solucionar problemas de operaciones de transmisión de T-SQL. 
- Mejoras para conservar la duración de la batería en modo inactivo. 
- Correcciones del motor de transmisión de T-SQL: 
   - Limpieza de los trabajos de transmisión detenidos. 
   - Correcciones para la localización. 
   - Mejora del control de Unicode. 
   - Mejora de la capacidad de depuración de la transmisión de T-SQL para SQL Edge, lo que permite a los usuarios consultar los errores de los trabajos desde get_streaming_job.
- Limpieza basada en la directiva de retención de datos: 
    - Correcciones para escenarios de creación y limpieza de directivas de retención.
- Correcciones en tareas de temporizador en segundo plano para mejorar el ahorro de energía para el modo de batería baja.

### <a name="known-issues"></a>Problemas conocidos 
- La función Date_Bucket de T-SQL no se puede usar en una columna calculada.


## <a name="ctp-23"></a>CTP 2.3
Número de versión del motor de SQL: 15.0.2000.1549
### <a name="whats-new"></a>Novedades
- Compatibilidad con orígenes personalizados en la función Date_Bucket(). 
- Compatibilidad con archivos BACPAC como parte de la implementación de SQL.
- Compatibilidad con la limpieza basada en la directiva de retención de datos:      
   - Compatibilidad de DDL para habilitar la directiva de retención. 
   - Limpieza para los procedimientos almacenados y tarea de limpieza en segundo plano.
   - Eventos ampliados para supervisar las tareas de limpieza.

### <a name="fixes"></a>Correcciones
- Mensajes de error adicionales y detalles para solucionar problemas de operaciones de transmisión de T-SQL. 
- Mejoras para conservar la duración de la batería en modo inactivo. 
- Motor de transmisión de T-SQL: 
   - Corrección del problema de marca de agua atascada con la ventana de salto de subtransmisión. 
   - Corrección del control de excepciones del marco para asegurarse de que se recopila como un error que pueda procesar el usuario.


## <a name="ctp-22"></a>CTP 2.2
Número de versión del motor de SQL: 15.0.2000.1546
### <a name="whats-new"></a>Novedades
- Compatibilidad con contenedores que no son raíz 
- Compatibilidad con la recopilación de datos de uso y diagnóstico. 
- Actualizaciones de transmisión de T-SQL:
   - Compatibilidad con los caracteres Unicode para nombres de objetos de secuencia

### <a name="fixes"></a>Correcciones
- Actualizaciones de transmisión de T-SQL:
   - Mejoras en la limpieza del proceso
   - Mejoras en el registro y los diagnósticos
- Mejora del rendimiento de la ingesta de datos

## <a name="ctp-21"></a>CTP 2.1 
Número de versión del motor de SQL: 15.0.2000.1545
### <a name="fixes"></a>Correcciones
- Los modelos PREDICT con ONNX pueden administrar el problema de CPUID en ARM. 
- Control mejorado de la ruta de acceso de error cuando se inicia la transmisión por secuencias de T-SQL.
- Corrección del valor del retraso de la marca de agua en las métricas de trabajo cuando no hay datos.
- Corrección de un problema con el adaptador de salida cuando este tiene un esquema de variables entre lotes.  

## <a name="ctp-20"></a>CTP 2.0 
Número de versión del motor de SQL: 15.0.2000.1401
### <a name="whats-new"></a>Novedades
-   Actualización del nombre de producto a *Azure SQL Edge*.
-  Función Date_Bucket:
    - Compatibilidad con el tipo Date, Time y DateTime.
- PREDICT con ONNX:
    - Requisito de ONNX para el parámetro RUNTIME.  
- Compatibilidad con la transmisión de T-SQL (versión preliminar limitada). 
 
### <a name="known-issues"></a>Problemas conocidos

- Problema: posibles errores al aplicar DACPAC al inicio debido a un problema de temporización.
- Solución alternativa: Reinicie SQL Server. De lo contrario, el contenedor volverá a intentar aplicar DACPAC.

### <a name="request-support"></a>Solicitud de soporte técnico
Puede solicitar soporte técnico en la [página de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest). Seleccione estos campos: 
- **Tipo de problema**: *Técnico* 
- **Servicio**: *IoT Edge*
- **Tipo de problema**: *Mi problema está relacionado con un módulo de IoT Edge*
- **Subtipo de problema**: *Azure SQL Edge*

:::image type="content" source="media/get-support/support-ticket.png" alt-text="Captura de pantalla que muestra una incidencia de soporte técnico de ejemplo.":::

## <a name="ctp-15"></a>CTP 1.5
Número de versión del motor de SQL: 15.0.2000.1331
### <a name="whats-new"></a>Novedades
- Función Date_Bucket:
    - Compatibilidad con el tipo DateTimeOffset.
- Modelos de PREDICT con ONNX:
  - Compatibilidad con NVARCHAR.
 
## <a name="ctp-14"></a>CTP 1.4
Número de versión del motor de SQL: 15.0.2000.1247
### <a name="whats-new"></a>Novedades
-   Modelos de PREDICT con ONNX:
    - Compatibilidad con VARCHAR.
    - Migración a la versión en tiempo de ejecución de ONNX 1.0 

- Se han habilitado las características siguientes:
  - Compatibilidad con CDC
  - Tabla de historial con compresión
  - Mayor factor de escala para la lectura anticipada del registro.
  - Modo por lotes frente a aplicación de filtro
  - Optimizaciones de lectura anticipada.
 
## <a name="ctp-13"></a>(CTP 1.3)
Número de versión del motor de SQL: 15.0.2000.1147
### <a name="whats-new"></a>Novedades
- Implementación del portal de Azure IoT: 
    - Compatibilidad con la implementación de imágenes de AMD64 y ARM
    - Compatibilidad con la creación de trabajos de streaming
    - Implementación de DACPAC.
- Modelos de PREDICT con ONNX:
    - Compatibilidad con tipos numéricos
- Se han habilitado las características siguientes:
    - Agregado de aplicación a examen de almacén de columnas
    - Recorridos cícliclos
- Trabajo de reducción del consumo de memoria y superficie.
