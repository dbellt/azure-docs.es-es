---
title: Extracción de información en Excel con Text Analytics y Power Automate
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo extraer texto de Excel sin tener que escribir código, con Text Analytics y Power Automate.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 05/12/2021
ms.author: aahi
ms.openlocfilehash: 81f56d29f0e0d08b2fed72ba970e4aa0ca0f2f1b
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110091881"
---
# <a name="extract-information-in-excel-using-text-analytics-and-power-automate"></a>Extracción de información en Excel con Text Analytics y Power Automate 

En este tutorial, creará un flujo de Power Automate para extraer texto de una hoja de cálculo de Excel sin tener que escribir código. 

Este flujo tomará una hoja de cálculo de problemas informados sobre un complejo de apartamento y los clasificará en dos categorías: plumbing y other. Además, se extraerán los nombres y números de teléfono de los inquilinos que los enviaron. Por último, el flujo anexará esta información a la hoja de Excel. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Usar Power Automate para crear un flujo
> * Cargar datos de Excel desde OneDrive para la Empresa
> * Extraer texto de Excel y enviarlo a la API Text Analytics 
> * Utilizar la información de la API para actualizar una hoja de Excel

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Microsoft Azure. [Cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/) o [inicie sesión](https://portal.azure.com/).
- Un recurso de Text Analytics. Si no tiene uno, puede [crearlo en Azure Portal](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) y usar el nivel gratuito para completar este tutorial.
- La [clave y el punto de conexión](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) que se generaron automáticamente durante el registro.
- Una hoja de cálculo que contiene los problemas de los inquilinos. En GitHub encontrará datos de ejemplo.
- Microsoft 365 con OneDrive para la Empresa.

## <a name="add-the-excel-file-to-onedrive-for-business"></a>Adición de un archivo de Excel a OneDrive para la Empresa

Descargue el archivo de Excel de ejemplo de [GitHub](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/TextAnalytics/sample-data/ReportedIssues.xlsx). Este archivo debe almacenarse en su cuenta de OneDrive para la Empresa.

:::image type="content" source="../media/tutorials/excel/example-data.png" alt-text="Ejemplos del archivo de Excel" lightbox="../media/tutorials/excel/example-data.png":::

Los problemas se informan en texto sin formato. Usaremos el reconocimiento de entidades con nombre de la API Text Analytics para extraer el nombre de la persona y su número de teléfono. A continuación, el flujo buscará la palabra "plumbing" en la descripción para clasificar los problemas. 

## <a name="create-a-new-power-automate-workflow"></a>Creación de un nuevo flujo de trabajo de Power Automate

Vaya al [sitio de Power Automate](https://preview.flow.microsoft.com/) e inicie sesión. Luego, haga clic en **Crear** y en **Flujo programado**.

:::image type="content" source="../media/tutorials/excel/flow-creation.png" alt-text="Pantalla de creación del flujo de trabajo" lightbox="../media/tutorials/excel/flow-creation.png":::

En la página **Build a scheduled cloud flow** (Crear un flujo de nube programado), inicialice el flujo con los campos siguientes:

|Campo |Value  |
|---------|---------|
|**Nombre de flujo**     | **Revisión programada** u otro nombre.         |
|**Starting** (iniciándose)     |  Escriba la fecha y hora actuales.       |
|**Repetir cada**     | **1 hora**        |

## <a name="add-variables-to-the-flow"></a>Adición de variables al flujo

Cree variables que representen la información que se agregará al archivo de Excel. Haga clic en **Nuevo paso** y busque **Inicializar variable**. Haga esto cuatro veces para crear cuatro variables.

:::image type="content" source="../media/tutorials/excel/initialize-variables.png" alt-text="Paso para inicializar variables" lightbox="../media/tutorials/excel/initialize-variables.png":::


Agregue la siguiente información a las variables que ha creado. Representan las columnas del archivo de Excel. Si las variables están contraídas, puede hacer clic en ellas para expandirlas.

| Acción |Nombre   | Tipo | Value |
|---------|---------|---|---|
| Inicializar la variable | var_person | String | Person |
| Inicializar la variable 2 | var_phone | String | Número de teléfono |
| Inicializar la variable 3 | var_plumbing | String | plumbing |
| Inicializar la variable 4 | var_other | String | Otros | 

:::image type="content" source="../media/tutorials/excel/flow-variables.png" alt-text="Información contenida en las variables de flujo" lightbox="../media/tutorials/excel/flow-variables.png":::

## <a name="read-the-excel-file"></a>Lectura del archivo de Excel

Haga clic en **Nuevo paso** y escriba **Excel**, a continuación, seleccione **Enumerar las filas de una tabla** de la lista de acciones.

:::image type="content" source="../media/tutorials/excel/list-excel-rows.png" alt-text="Incorporación de filas de Excel al flujo" lightbox="../media/tutorials/excel/list-excel-rows.png":::

Rellene los campos de esta sección para agregar el archivo de Excel al flujo. En este tutorial es necesario haber cargado el archivo en OneDrive para la Empresa.

:::image type="content" source="../media/tutorials/excel/list-excel-rows-options.png" alt-text="Rellenado de las filas de Excel en el flujo" lightbox="../media/tutorials/excel/list-excel-rows-options.png":::

Haga clic en **Nuevo paso** y agregue una acción **Aplicar a cada uno**.

:::image type="content" source="../media/tutorials/excel/add-apply-action.png" alt-text="Incorporación de una acción Aplicar a cada uno" lightbox="../media/tutorials/excel/add-apply-action.png":::

Haga clic dentro del cuadro **Seleccionar una salida de los pasos anteriores**. En el cuadro de contenido dinámico que aparece, seleccione **valor**.

:::image type="content" source="../media/tutorials/excel/select-output.png" alt-text="Selección de salida del archivo de Excel" lightbox="../media/tutorials/excel/select-output.png":::

## <a name="send-a-request-to-the-text-analytics-api"></a>Envío de una solicitud a la API Text Analytics

Si todavía no lo ha hecho, debe crear un [recurso de Text Analytics](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) en Azure Portal.

### <a name="create-a-text-analytics-connection"></a>Creación de una conexión de Text Analytics

En **Aplicar a cada uno**, haga clic en **Agregar una acción**. Vaya a la página de **clave y punto de conexión** del recurso de Text Analytics en Azure Portal y obtenga la clave y el punto de conexión del recurso de Text Analytics.

En el flujo, escriba la siguiente información para crear una conexión de Text Analytics.

> [!NOTE]
> Si ya ha creado una conexión de Text Analytics y desea cambiar los detalles de la conexión, haga clic en los puntos suspensivos de la esquina superior derecha y haga clic en **+ Agregar nueva conexión**.

| Campo           | Value                                                                                                             |
|-----------------|-------------------------------------------------------------------------------------------------------------------|
| Nombre de la conexión | Nombre para la conexión al recurso de Text Analytics. Por ejemplo, `TAforPowerAutomate`. |
| Clave de cuenta     | Clave del recurso de Text Analytics.                                                                                   |
| Dirección URL del sitio        | Punto de conexión del recurso de Text Analytics.                                                       |

:::image type="content" source="../media/tutorials/excel/add-credentials.png" alt-text="Incorporación de credenciales de Text Analytics a un flujo" lightbox="../media/tutorials/excel/add-credentials.png":::


## <a name="extract-the-excel-content"></a>Extracción del contenido de Excel 

Una vez que se cree la conexión, busque **Text Analytics** y seleccione **Reconocimiento de entidades con nombre**. Se extraerá la información de la columna de descripción del problema.

:::image type="content" source="../media/tutorials/excel/extract-info.png" alt-text="Extracción de las entidades de la hoja de Excel" lightbox="../media/tutorials/excel/extract-info.png":::

Haga clic en el campo **Texto** y seleccione **Description** de la ventana de contenido dinámico que aparece. Escriba `en` en Idioma y un nombre único como identificador de documento (es posible que tenga que hacer clic en **Mostrar opciones avanzadas**).

:::image type="content" source="../media/tutorials/excel/description-from-dynamic-content.png" alt-text="Obtención del texto de la columna de descripción de la hoja de Excel" lightbox="../media/tutorials/excel/description-from-dynamic-content.png":::


En **Aplicar a cada uno**, haga clic en **Agregar una acción** y cree otra acción **Aplicar a cada uno**. Haga clic dentro del cuadro de texto y seleccione **Documentos** en la ventana Contenido dinámico que aparece.

:::image type="content" source="../media/tutorials/excel/apply-to-each-documents.png" alt-text="Creación de otra acción Aplicar a cada uno." lightbox="../media/tutorials/excel/apply-to-each-documents.png":::


## <a name="extract-the-person-name"></a>Extracción del nombre de la persona

A continuación, buscará el tipo de entidad de persona en la salida de Text Analytics. En **Aplicar a cada uno 2**, haga clic en **Agregar una acción** y cree otra acción **Aplicar a cada uno**. Haga clic dentro del cuadro de texto y seleccione **Entidades** en la ventana de contenido dinámico que aparece.

:::image type="content" source="../media/tutorials/excel/add-apply-action-2.png" alt-text="Búsqueda de la entidad de persona en la salida de Text Analytics" lightbox="../media/tutorials/excel/add-apply-action-2.png":::


Dentro de la acción **Aplicar a cada uno 3** recién creada, haga clic en **Agregar una acción** y agregue un control **Condición**.

:::image type="content" source="../media/tutorials/excel/create-condition.png" alt-text="Incorporación de un control de condición a la acción Aplicar a cada uno 3" lightbox="../media/tutorials/excel/create-condition.png":::


En la ventana Condición, haga clic en el primer cuadro de texto. En la ventana Contenido dinámico, busque **Categoría** y selecciónela.

:::image type="content" source="../media/tutorials/excel/choose-entities-value.png" alt-text="Incorporación de una categoría a la condición de control" lightbox="../media/tutorials/excel/choose-entities-value.png":::


Asegúrese de que el segundo cuadro esté establecido en **es igual a**. A continuación, seleccione el tercer cuadro y busque `var_person` en la ventana de contenido dinámico. 

:::image type="content" source="../media/tutorials/excel/choose-variable-value.png" alt-text="Incorporación de la variable person" lightbox="../media/tutorials/excel/choose-variable-value.png":::


En la condición **En caso positivo**, escriba en Excel y, a continuación, seleccione **Actualizar una fila**.

:::image type="content" source="../media/tutorials/excel/yes-column-action.png" alt-text="Actualización de la condición yes" lightbox="../media/tutorials/excel/yes-column-action.png":::

Escriba la información de Excel y actualice los campos **Columna de clave**, **Valor de clave** y **PersonName**. Esto anexará el nombre detectado por la API a la hoja de Excel. 

:::image type="content" source="../media/tutorials/excel/yes-column-action-options.png" alt-text="Incorporación de la información de Excel" lightbox="../media/tutorials/excel/yes-column-action-options.png":::

## <a name="get-the-phone-number"></a>Obtención del número de teléfono

Para minimizar la acción **Aplicar a cada uno 3**, haga clic en el nombre. A continuación, agregue otra acción **Aplicar a cada uno** a **Aplicar a cada uno 2**, tal como lo ha hecho antes. Se le asignará el nombre **Aplicar a cada uno 4**. Seleccione el cuadro de texto y agregue **entidades** como salida para esta acción. 

:::image type="content" source="../media/tutorials/excel/add-apply-action-phone.png" alt-text="Incorporación de las entidades de la salida de Text Analytics a otra acción Aplicar a cada uno." lightbox="../media/tutorials/excel/add-apply-action-phone.png":::

Dentro de **Aplicar a cada uno 4**, agregue un control **Condición**. Se denominará **Condición 2**. En el primer cuadro de texto, busque y agregue **Categorías** en la ventana Contenido dinámico. Asegúrese de que el cuadro central esté establecido en **es igual a**. A continuación, en el cuadro de texto de la derecha, escriba `var_phone`. 

:::image type="content" source="../media/tutorials/excel/condition-2-options.png" alt-text="Incorporación de un segundo control de condición" lightbox="../media/tutorials/excel/condition-2-options.png":::

En la condición **En caso positivo**, agregue una acción **Actualizar una fila**. A continuación, escriba la información como se hizo anteriormente, para la columna de números de teléfono de la hoja de Excel. Esto anexará el número de teléfono detectado por la API a la hoja de Excel. 

:::image type="content" source="../media/tutorials/excel/condition-2-yes-column.png" alt-text="Incorporación de la información de Excel a la segunda condición if yes" lightbox="../media/tutorials/excel/condition-2-yes-column.png":::

## <a name="get-the-plumbing-issues"></a>Obtención de los problemas de fontanería

Para minimizar **Aplicar a cada uno 4**, haga clic en el nombre. A continuación, cree otra acción **Aplicar a cada uno** en la acción primaria. Seleccione el cuadro de texto y agregue **Entidades** como salida para esta acción en la ventana de contenido dinámico. 

:::image type="content" source="../media/tutorials/excel/add-apply-action-plumbing.png" alt-text="Creación de otra acción Aplicar a cada uno" lightbox="../media/tutorials/excel/add-apply-action-plumbing.png":::

A continuación, el flujo comprobará si la descripción del problema de la fila de la tabla de Excel contiene la palabra "plumbing". En caso afirmativo, se agregará "plumbing" en la columna IssueType. De lo contrario, escribiremos "other".

Dentro de la acción **Aplicar a cada uno 4**, agregue un control **Condición**. Se denominará **Condición 3**. En el primer cuadro de texto, busque y agregue **Descripción** desde la ventana de contenido dinámico. Asegúrese de que el cuadro central indique **contiene**. A continuación, en el cuadro de texto de la derecha, busque y seleccione `var_plumbing`. 

:::image type="content" source="../media/tutorials/excel/condition-3-options.png" alt-text="Creación de un nuevo control de condición" lightbox="../media/tutorials/excel/condition-3-options.png":::

En la condición **En caso positivo**, haga clic en **Agregar una acción** y seleccione **Actualizar una fila**. A continuación, escriba la información como antes. En la columna IssueType, seleccione `var_plumbing`. Esto aplicará una etiqueta "plumbing" a la fila.

En la condición **En caso negativo**, haga clic en **Agregar una acción** y seleccione **Actualizar una fila**. A continuación, escriba la información como antes. En la columna IssueType, seleccione `var_other`. Esto aplicará una etiqueta "other" a la fila.

:::image type="content" source="../media/tutorials/excel/plumbing-issue-condition.png" alt-text="Incorporación de información a ambas condiciones" lightbox="../media/tutorials/excel/plumbing-issue-condition.png":::

## <a name="test-the-workflow"></a>Probar el flujo de trabajo

En la esquina superior derecha de la pantalla, haga clic en **Guardar** y luego en **Probar**. En **Probar flujo**, seleccione **manualmente**. A continuación, haga clic en **Probar** y **Ejecutar flujo**.

El archivo de Excel se actualizará en su cuenta de OneDrive. Tendrá un aspecto como el siguiente.

:::image type="content" source="../media/tutorials/excel/updated-excel-sheet.png" alt-text="Prueba del flujo de trabajo y visualización de la salida" lightbox="../media/tutorials/excel/updated-excel-sheet.png":::

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Explorar más soluciones](../text-analytics-user-scenarios.md)
