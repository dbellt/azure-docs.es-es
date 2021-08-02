---
title: Conexión a Azure Analysis Services con Power BI | Microsoft Docs
description: Aprenda a conectarse a un servidor de Azure Analysis Services mediante Power BI. Una vez conectados, los usuarios pueden explorar los datos del modelo.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 5/25/2021
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 579f97deaadf8005d7a7986ff3b032909c28972e
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110496567"
---
# <a name="connect-with-power-bi"></a>Conexión con Power BI

Una vez que se ha creado un servidor en Azure y se ha implementado un modelo tabular, los usuarios de su organización pueden conectarse y comenzar a explorar los datos. 

> [!NOTE]
> Si se publica un modelo de Power BI Desktop en el servicio Power BI, en el servidor de Azure Analysis Services, asegúrese de que la propiedad  de servidor de intercalación que distingue mayúsculas de minúsculas no está seleccionada (valor predeterminado). La propiedad de servidor que distingue mayúsculas de minúsculas puede establecerse mediante SQL Server Management Studio.
> 
> 
  
## <a name="connect-in-power-bi-desktop"></a>Conexión en Power BI Desktop

1. En Power BI Desktop, haga clic en **Obtener datos** > **Azure** > **Base de datos de Azure Analysis Services**.

2. En **Servidor**, escriba el nombre del servidor. Asegúrese de incluir la dirección URL completa; por ejemplo, asazure://westcentralus.asazure.windows.net/advworks.

3. En **Base de datos**, si conoce el nombre de la base de datos de modelo tabular o la perspectiva a la que desea conectarse, péguelo aquí. En caso contrario, puede dejar este campo en blanco y seleccionar una base de datos o una perspectiva más adelante.

4. Seleccione una opción de conexión y presione **Conectar**. 

    Se admiten las opciones **Conectar en directo** e **Importar**. Sin embargo, se recomienda usar conexiones en directo porque el modo de importación tiene algunas limitaciones. En particular, durante la importación se puede afectar el rendimiento del servidor.
    
    Si tiene un modelo Power BI en [modo de almacenamiento mixto](/power-bi/transform-model/desktop-composite-models), la opción **Conectarse en directo** se reemplaza por la opción **[DirectQuery](/power-bi/connect-data/desktop-directquery-datasets-azure-analysis-services)** . Las conexiones dinámicas también se actualizan automáticamente a DirectQuery si el modelo cambia del modo de importación al modo de almacenamiento mixto.

5. Escriba sus credenciales de inicio de sesión, si se le solicitan. 

   > [!NOTE]
   > No se admiten las cuentas de código de acceso de un solo uso (OTP). 

6. En **Navegador**, expanda el servidor, seleccione el modelo o la perspectiva a la que desea conectarse y haga clic en **Conectar**. Haga clic en un modelo o una perspectiva para mostrar todos los objetos de esa vista.

    El modelo se abre en Power BI Desktop con un informe en blanco en la vista de informe. La lista de campos muestra todos los objetos del modelo no ocultos. El estado de la conexión se muestra en la esquina inferior derecha.

## <a name="connect-in-power-bi-service"></a>Conexión en Power BI (servicio)

1. Cree un archivo de Power BI Desktop que tenga una conexión en directo con el modelo del servidor.
2. En [Power BI](https://powerbi.microsoft.com), haga clic en **Obtener datos** > **Archivos**, busque el archivo .pbix y selecciónelo.

## <a name="request-memory-limit"></a>Límite de memoria de solicitud

Para proteger el rendimiento del sistema, se aplica un límite de memoria para todas las consultas emitidas por informes de Power BI en Azure Analysis Services, independientemente del [límite de memoria de consulta](/analysis-services/server-properties/memory-properties?view=azure-analysis-services-current&preserve-view=true) configurado en el servidor Azure Analysis Services. Los usuarios deben considerar la posibilidad de simplificar la consulta o sus cálculos si la consulta tiene un uso intensivo de memoria.

|                                                           | Límite de memoria de solicitud |
|-----------------------------------------------------------|----------------------|
| Conexión en vivo desde Power BI                            | 10 GB  |
| DirectQuery desde el informe de Power BI en el área de trabajo Compartida  | 1 GB   |
| DirectQuery desde el informe de Power BI en el área de trabajo Premium | 10 GB  |
| Preguntas y respuestas de Power BI | 100 MB |

## <a name="see-also"></a>Consulte también
[Conexión a Azure Analysis Services](analysis-services-connect.md)   
[Bibliotecas de cliente](/analysis-services/client-libraries?view=azure-analysis-services-current&preserve-view=true)
