---
title: Problemas de compatibilidad con aplicaciones de terceros y Azure Synapse Analytics
description: Aquí se describen los problemas conocidos que las aplicaciones de terceros pueden encontrar con Azure Synapse.
services: synapse-analytics
author: periclesrocha
ms.service: synapse-analytics
ms.topic: troubleshooting
ms.subservice: sql
ms.date: 11/18/2020
ms.author: procha
ms.reviewer: jrasnick
ms.openlocfilehash: 3bad9d6464b41ff1b7ad03147d3a48c50c787cb0
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107568121"
---
# <a name="compatibility-issues-with-third-party-applications-and-azure-synapse-analytics"></a>Problemas de compatibilidad con aplicaciones de terceros y Azure Synapse Analytics

Las aplicaciones que se compilan para SQL Server funcionarán sin problemas con los grupos de SQL dedicados de Azure Synapse. Sin embargo, en algunos casos es posible que las características y los elementos del lenguaje que se usan habitualmente en SQL Server no estén disponibles en Azure Synapse o que se comporten de manera diferente.

En este artículo se enumeran los problemas conocidos que pueden aparecer al usar aplicaciones de terceros con Azure Synapse Analytics. 

## <a name="tableau-error-an-attempt-to-complete-a-transaction-has-failed-no-corresponding-transaction-found"></a>Error de Tableau: "Error al intentar completar una transacción. No se encontró ninguna transacción correspondiente."

A partir de la versión 10.0.11038.0 del grupo de SQL dedicado de Azure Synapse, algunas consultas de Tableau que realizan llamadas a procedimientos almacenados pueden generar un error que devuelve el siguiente mensaje: " **[Microsoft][Controlador ODBC 17 para SQL Server][SQL Server]111214; Error al intentar completar una transacción. No se encontró ninguna transacción correspondiente.** "

### <a name="cause"></a>Causa

Se trata de un problema en el grupo de SQL dedicado de Azure Synapse que provoca la introducción de nuevos procedimientos almacenados del sistema a los que llaman automáticamente los controladores ODBC y JDBC. Uno de esos procedimientos almacenados del sistema puede hacer que se anulen las transacciones abiertas si se produce un error en la ejecución. Este problema puede producirse en función de la lógica de la aplicación cliente.

### <a name="solution"></a>Solución
Los clientes que vean este problema en particular al usar una instancia de Tableau conectada a los grupos de SQL dedicados de Azure Synapse, deben configurar el valor FMTONLY en "YES" en la conexión SQL. En el caso de Tableau Desktop y Tableau Server, debe usar un archivo de personalización de origen de datos (TDC) de tabla para asegurarse de que esa instancia de Tableau pasa el parámetro al controlador.  

> [!NOTE] 
> Microsoft no proporciona soporte técnico para herramientas de terceros. Aunque hemos probado que esta solución funciona con Tableau Desktop 2020.3.2, debe usar esta solución según su propia capacidad.
>

* [Para obtener información sobre cómo realizar personalizaciones globales con un archivo TDC en Tableau Desktop, consulte la documentación de Tableau Desktop.](https://help.tableau.com/current/pro/desktop/en-us/odbc_customize.htm)
* [Para obtener información sobre cómo realizar personalizaciones globales con un archivo TDC en Tableau Server, consulte "Uso de un archivo .TDC con Tableau Server".](https://kb.tableau.com/articles/howto/using-a-tdc-file-with-tableau-server)

En el ejemplo siguiente se muestra un archivo TDC de Tableau que pasa el parámetro FMTONLY=YES a la cadena de conexión de SQL:

```json
<connection-customization class='azure_sql_dw' enabled='true' version='18.1'>
    <vendor name='azure_sql_dw' />
    <driver name='azure_sql_dw' />
    <customizations>        
        <customization name='odbc-connect-string-extras' value='UseFMTONLY=yes' />
    </customizations>
</connection-customization>
```
Para obtener más información sobre el uso de archivos TDC, póngase en contacto con el soporte técnico de Tableau. 

## <a name="see-also"></a>Vea también

* [Elementos del lenguaje T-SQL para un grupo de SQL dedicado en Azure Synapse Analytics.](./sql-data-warehouse-reference-tsql-language-elements.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json)
* [Instrucciones T-SQL admitidas para un grupo de SQL dedicado en Azure Synapse Analytics.](./sql-data-warehouse-reference-tsql-statements.md)