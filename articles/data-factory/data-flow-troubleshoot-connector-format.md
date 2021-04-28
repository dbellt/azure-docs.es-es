---
title: Solución de problemas de conectores y formatos en flujos de datos de asignación
description: Obtenga información sobre la solución de problemas de flujos de datos relacionados con conectores y formatos en Azure Data Factory.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 04/20/2021
ms.openlocfilehash: 1991436e2cc890b5c6a339f79e42536ced2fbd36
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739483"
---
# <a name="troubleshoot-connector-and-format-issues-in-mapping-data-flows-in-azure-data-factory"></a>Solución de problemas de conectores y formatos en flujos de datos de asignación en Azure Data Factory


En este artículo se exploran los métodos de solución de problemas relacionados con conectores y formatos para flujos de datos de asignación en Azure Data Factory (ADF).


## <a name="cosmos-db--json"></a>Cosmos DB y JSON

### <a name="support-customized-schemas-in-the-source"></a>Compatibilidad con esquemas personalizados en el origen

#### <a name="symptoms"></a>Síntomas
Si quiere usar el flujo de datos de ADF para mover o transferir datos desde Cosmos DB/JSON hacia otros almacenes de datos, es posible que falten algunas columnas de los datos de origen. 

#### <a name="cause"></a>Causa 
Para los conectores sin esquema (el número de columna, el nombre de columna y el tipo de datos de columna de cada fila pueden ser diferentes cuando se comparan con otros), de manera predeterminada, ADF usa filas de ejemplo (por ejemplo, datos de las 100 o 1000 filas principales) para inferir el esquema, y el resultado inferido se usará como esquema para leer los datos. Por lo tanto, si los almacenes de datos tienen columnas adicionales que no aparecen en las filas de ejemplo, los datos de estas columnas adicionales no se leen, mueven ni transfieren a almacenes de datos receptores.

#### <a name="recommendation"></a>Recomendación
Para sobrescribir el comportamiento predeterminado y traer campos adicionales, ADF da opciones para personalizar el esquema de origen. Puede especificar columnas adicionales o que faltan que podrían faltar en el resultado de inferencia del esquema en la proyección del origen del flujo de datos para leer los datos, y puede aplicar una de las siguientes opciones para establecer el esquema personalizado. En general, se prefiere la **opción 1**.

- **Opción 1**: En comparación con los datos de origen originales, que pueden ser un archivo, una tabla o un contenedor grandes que contengan millones de filas con esquemas complejos, puede crear una tabla o contenedor temporales con algunas filas que contengan todas las columnas que quiere leer y, a continuación, pasar a la siguiente operación: 

    1. Use la **Configuración de depuración** del origen del flujo de datos para hacer que **Import projection** (Importar proyección) con los archivos o tablas de ejemplo obtenga el esquema completo. Puede seguir los pasos de la siguiente imagen:<br/>

        ![Captura de pantalla que muestra la primera parte de la primera opción para personalizar el esquema de origen.](./media/data-flow-troubleshoot-connector-format/customize-schema-option-1-1.png)<br/>
         1. Seleccione **Configuración de depuración** en el lienzo del flujo de datos.
         1. En el panel emergente, seleccione **Sample table** (Tabla de ejemplo) en la pestaña **cosmosSource** y escriba el nombre de la tabla en el bloque **Tabla**.
         1. Haga clic en **Guardar** para guardar la configuración.
         1. Seleccione **Import projection** (Importar proyección).<br/>  
    
    1. Vuelva a cambiar la **Configuración de depuración** para usar el conjunto de datos de origen para el movimiento o la transformación de datos restantes. Puede seguir con los pasos de la imagen siguiente:<br/>

        ![Captura de pantalla que muestra la segunda parte de la primera opción para personalizar el esquema de origen.](./media/data-flow-troubleshoot-connector-format/customize-schema-option-1-2.png) <br/>   
         1. Seleccione **Configuración de depuración** en el lienzo del flujo de datos.
         1. En el panel emergente, seleccione **Source dataset** (Conjunto de datos de origen) en la pestaña **cosmosSource**.
         1. Haga clic en **Guardar** para guardar la configuración.<br/>
    
    Después, el entorno en tiempo de ejecución del flujo de datos de ADF respetará y usará el esquema personalizado para leer los datos del almacén de datos original. <br/>

- **Opción 2**: Si está familiarizado con el esquema y el lenguaje DSL de los datos de origen, puede actualizar manualmente el script de origen del flujo de datos para agregar columnas adicionales o que faltan para leer los datos. En la imagen siguiente se muestra un ejemplo: 

    ![Captura de pantalla que muestra la segunda opción para personalizar el esquema de origen.](./media/data-flow-troubleshoot-connector-format/customize-schema-option-2.png)

## <a name="next-steps"></a>Pasos siguientes
Para obtener más ayuda para solucionar problemas, consulte estos recursos:

*  [Solución de problemas de los flujos de datos de asignación en Azure Data Factory](data-flow-troubleshoot-guide.md)
*  [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)