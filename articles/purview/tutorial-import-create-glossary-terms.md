---
title: 'Tutorial: Creación e importación de términos de glosario en Azure Purview (versión preliminar)'
description: En este tutorial se describe cómo crear términos de glosario, agregar términos de glosario a un recurso e importar términos de glosario.
author: shsandeep123
ms.author: sandeepshah
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: tutorial
ms.date: 12/01/2020
ms.openlocfilehash: b240806b934cae54810cf9dfe1a6c1f369cede7e
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107587517"
---
# <a name="tutorial-create-and-import-glossary-terms-in-azure-purview-preview"></a>Tutorial: Creación e importación de términos de glosario en Azure Purview (versión preliminar)

> [!IMPORTANT]
> Azure Purview se encuentra actualmente en VERSIÓN PRELIMINAR. Los [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) incluyen términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

Un glosario es una herramienta importante para mantener y organizar el catálogo. Puede crear el glosario definiendo nuevos términos o importando una lista de términos y aplicando luego esos términos a sus recursos.

Este tutorial constituye la *parte 5 de una serie de tutoriales de cinco partes* en la que aprenderá los aspectos básicos de cómo administrar la gobernanza de datos en un patrimonio de datos mediante Azure Purview. Este tutorial se basa en el trabajo completado en la [parte 4: Exploración de conjuntos de recursos, detalles, esquemas y clasificaciones en Azure Purview (versión preliminar)](tutorial-schemas-and-classifications.md).

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear términos del glosario.
> * Agregar términos del glosario a un recurso.
> * Importar términos del glosario.

## <a name="prerequisites"></a>Prerrequisitos

* Complete el [Tutorial: Exploración de conjuntos de recursos, detalles, esquemas y clasificaciones en Azure Purview (versión preliminar)](tutorial-schemas-and-classifications.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-glossary-terms"></a>Creación de términos del glosario

Puede crear términos técnicos o empresariales en el glosario. También puede anotar los recursos aplicándoles términos.

A continuación se muestra un resumen de algunos de los campos más comunes de la página **Términos del glosario**:

* **Estado**: asigne un estado al término (**Borrador**, **Aprobado**, **Expirado** o **Alerta**).

* **Definición**: escriba una definición del término.

* **Acronym** (Acrónimo): escriba una versión abreviada del término con las letras iniciales de cada palabra.

* **Sinónimos**: seleccione otros términos con las mismas definiciones o similares.

* **Related Terms** (Términos relacionados): seleccione otros términos en el glosario que estén relacionados con este, pero que tengan diferentes definiciones. Algunos ejemplos son términos técnicos que están relacionados con un término empresarial, un nombre de código u otros términos que deben asociarse con el término.

Cree un término de glosario siguiendo estos pasos:

1. Vaya al recurso de Azure Purview en Azure Portal y seleccione **Abrir Purview Studio**. Se le dirigirá automáticamente a la página principal de Purview Studio.

1. Seleccione **Glosario** en el panel izquierdo para abrir la página **Términos del glosario**.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/glossary-terms-page.png" alt-text="Captura de pantalla que muestra la página Términos del glosario.":::

1. Seleccione **Nuevo término** > **Valores predeterminados del sistema** > **Continuar**.

1. En la pestaña **Información general** de la página **Nuevo término**, escriba el **Nombre** para el nuevo término: *Finanzas*.

1. Escriba la **Definición**: *Este término representa información financiera para Contoso Inc.*

1. En la lista desplegable **Estado**, seleccione **Aprobado**.

1. Cuando haya finalizado, seleccione **Crear**.

    :::image type="content" source="./media/tutorial-import-create-glossary-terms/enter-new-glossary-term.png" alt-text="Captura de pantalla que muestra cómo crear un nuevo término del glosario.":::

## <a name="add-glossary-terms-to-an-asset"></a>Incorporación de términos del glosario a un recurso

1. Utilice los pasos aprendidos en la [parte 1 de esta serie de tutoriales](tutorial-scan-data.md) para buscar un recurso. Por ejemplo, **Contoso_CashFlow_{N}.csv**.

1. En la página de detalles del recurso, seleccione **Editar**.

1. En la lista desplegable **Términos del glosario** de la pestaña **Información general**, seleccione **Finanzas** y, a continuación, **Guardar**.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/add-glossary-term-to-asset.png" alt-text="Captura de pantalla que muestra cómo agregar un término del glosario a un recurso.":::

1. Vaya a la sección **Términos del glosario** en la pestaña **Información general** y observe que el término *Finanzas* se haya aplicado al recurso.

## <a name="import-glossary-terms"></a>Importación de términos del glosario

Para importar términos o actualizar los términos existentes de forma masiva, cargue una plantilla rellenada previamente en el glosario.

En este procedimiento, va a importar términos del glosario mediante un archivo .csv:

1. Tenga en cuenta dónde ha almacenado el archivo denominado *StarterKitTerms.csv*, que forma parte del Starter Kit descargado en la [parte 1 de esta serie de tutoriales](tutorial-scan-data.md).

   Este archivo contiene una lista de términos rellenados previamente que son pertinentes para su patrimonio de datos.

 > [!Important]
   > La dirección de correo electrónico de los administradores y expertos en el archivo .CSV debe ser la dirección principal del usuario del grupo de AAD. Todavía no se admite el correo electrónico alternativo, el nombre principal de usuario y los correos electrónicos que no son de AAD. Debe reemplazar las direcciones de correo electrónico por la dirección principal de AAD de su organización.

1. Para empezar a importar, seleccione **Glosario** y, a continuación, **Import terms** (Importar términos).

    :::image type="content" source="./media/tutorial-import-create-glossary-terms/import-glossary-terms-select.png" alt-text="Captura de pantalla que muestra cómo importar términos del glosario.":::

1. En la página **Import terms** (Importar términos), seleccione **Valores predeterminados del sistema** y, después, **Continuar**.

1. Seleccione **Examinar**, vaya a la ubicación en la que ha descargado *StarterKitTerms.csv* y, a continuación, seleccione **Abrir**.

1. Seleccione **Aceptar** para empezar a importar los términos.

   Una vez finalizada la importación, los nuevos términos del glosario se muestran en la página **Términos del glosario**.

1. Consulte cada uno de los términos recién importados para ver cómo están definidos.

## <a name="create-custom-term-templates"></a>Creación de plantillas de términos personalizados

En esta sección, aprenderá a crear una plantilla de términos personalizados con atributos personalizados y a importar datos mediante un archivo csv de plantilla.

Hay varias plantillas de términos del sistema existentes, que puede ver seleccionando **Glosario** > **Manage term templates (Administrar plantillas de términos)**  > **Sistema**.

:::image type="content" source="./media/tutorial-import-create-glossary-terms/system-term-templates.png" alt-text="Plantillas de términos del sistema.":::

Para crear una nueva plantilla de términos personalizados, siga estos pasos:

1. Seleccione **Glosario** en el menú de la izquierda.
1. Seleccione **Manage term templates (Administrar plantillas de términos)**  > **Personalizar** > **New term template (Nueva plantilla de términos)**

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/create-new-custom-term-template.png" alt-text="Creación de una nueva plantilla de términos personalizados.":::

En la pantalla **New term template** (Nueva plantilla de términos), realice los pasos siguientes:

1. Escriba el **Nombre de plantilla**: `Sensitivity level`.
1. Especifique la descripción que desee, como `Indicates the level of sensitivity for this data.`
1. Seleccione **+ Nuevo atributo** a fin de abrir un cuadro de diálogo para agregar atributos.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/new-term-template-screen-start.png" alt-text="Inicio de pantalla de una nueva plantilla de términos.":::

1. En la pantalla **Nuevo atributo**, escriba los parámetros siguientes:

   |Configuración|Valor sugerido|
   |---------|-----------|
   |Nombre del atributo |es información confidencial|
   |Tipo de campo | Opción única|
   |Mark as required (Marcar como obligatorio) | Active esta casilla.|
   |+ Agregar una opción | Agregue dos opciones. "Sí" y "No".|

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/add-new-attribute.png" alt-text="Incorporación de un nuevo atributo.":::

1. Repita los pasos anteriores para agregar otro atributo con el nombre `can be shared externally`. Una vez agregados los dos atributos, seleccione **Crear**.

## <a name="import-terms-from-a-template"></a>Importación de términos desde una plantilla

A continuación, importe un nuevo término con la plantilla **Sensitivity level** (Nivel de confidencialidad) que ha creado. 

1. En la pantalla **Términos del glosario**, seleccione **Import terms** (Importar términos).

1. Seleccione **Sensitivity level** (Nivel de confidencialidad) en la pantalla **Import terms** (Importar términos). Seleccione **Continuar**.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/select-sensitivity-level.png" alt-text="Selección del nivel de sensibilidad.":::

1. La plantilla de términos para **Sensitivity level** (Nivel de confidencialidad) contiene los atributos predeterminados del sistema, así como los nuevos atributos que ha agregado: `can be shared externally` y `is sensitive information`. Seleccione el archivo **Download a sample .CSV**.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/download-sample-csv-file.png" alt-text="Archivo Download sample csv.":::

1. Se descarga un archivo de plantilla para que pueda editar y cargar el nuevo término del glosario con los atributos de cliente. Abra el archivo CSV que ha descargado. Agregue nuevos términos y sus valores adecuados como nuevas filas en el archivo CSV.

   :::image type="content" source="./media/tutorial-import-create-glossary-terms/create-term-in-csv.png" alt-text="Creación de un término en un archivo csv.":::

   Cuando se cargue el archivo, se crearán los términos que se enumeran en la columna **Related Terms** (Términos relacionados) o **Sinónimos** que no existen todavía. Se crearán con la plantilla **Valores predeterminados del sistema**.

1. Para cargar el archivo, vuelva a la pantalla **Import terms** (Importar términos) y seleccione **Examinar** junto a la casilla **Select the complete .CSV file to upload** (Seleccionar el archivo .CSV completo para cargar). Seleccione el archivo en el cuadro de diálogo que se abre y, después, **Aceptar**.

1. Los nuevos términos se muestran ahora en la pantalla **Términos del glosario**. Para ver los detalles de los nuevos términos, haga clic en el nombre del término en la lista.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
>
> * Crear términos del glosario.
> * Agregar términos del glosario a un recurso.
> * Importar términos del glosario.

Continúe en el siguiente artículo para ver información detallada sobre el catálogo.

> [!div class="nextstepaction"]
> [Descripción de Insights en Azure Purview](concept-insights.md)
