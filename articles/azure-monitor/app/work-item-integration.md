---
title: 'Integración de elementos de trabajo: Application Insights'
description: Aprenda a crear elementos de trabajo en GitHub o Azure DevOps con los datos de Application Insights insertados en ellos.
ms.topic: conceptual
ms.date: 04/22/2021
ms.openlocfilehash: 57eae6b0cda7a0f77a83ecbe453aca56eaf2504b
ms.sourcegitcommit: bd1a4e4df613ff24e954eb3876aebff533b317ae
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2021
ms.locfileid: "107930439"
---
# <a name="work-item-integration"></a>Integración de elementos de trabajo 

La funcionalidad de integración de elementos de trabajo le permite crear fácilmente en GitHub o Azure DevOps elementos de trabajo que tienen datos de Application Insights pertinentes insertados en ellos.

## <a name="create-and-configure-a-work-item-template"></a>Creación y configuración de una plantilla de elemento de trabajo

1. Para crear una plantilla de elemento de trabajo, vaya a su recurso de Application Insights y, a la izquierda, en *Configurar*, seleccione **Elementos de trabajo**; luego, en la parte superior, elija **Create a new template** (Crear nueva plantilla).

    :::image type="content" source="./media/work-item-integration/create-work-item-template.png" alt-text=" Captura de pantalla de la pestaña Elementos de trabajo con la opción Create a new template (Crear nueva plantilla) seleccionada." lightbox="./media/work-item-integration/create-work-item-template.png":::

    Si no existe ninguna plantilla, también puede crear una plantilla de elemento de trabajo desde la pestaña Detalles de la transacción completa. Seleccione un evento y, a la derecha, elija **Crear un elemento de trabajo**; luego, seleccione **Start with a workbook template** (Empezar con una plantilla de libro).

    :::image type="content" source="./media/work-item-integration/create-template-from-transaction-details.png" alt-text=" Captura de pantalla de la pestaña Detalles de la transacción completa con Crear un elemento de trabajo < Start with a workbook template (Empezar con una plantilla de libro) seleccionadas." lightbox="./media/work-item-integration/create-template-from-transaction-details.png":::

2. Después de seleccionar **Create a new template** (Crear nueva plantilla), puede elegir los sistemas de seguimiento, asignar un nombre al libro, crear un vínculo al sistema de seguimiento seleccionado y elegir una región para almacenar la plantilla (el valor predeterminado es la región en la que se encuentra el recurso de Application Insights). Los parámetros de dirección URL son la dirección URL predeterminada del repositorio, por ejemplo, `https://github.com/myusername/reponame` o `https://mydevops.visualstudio.com/myproject`.

    :::image type="content" source="./media/work-item-integration/create-workbook.png" alt-text=" Captura de pantalla de Create a new work item workbook template (Crear una plantilla de libro de elemento de trabajo).":::

    Puede establecer propiedades específicas de los elementos de trabajo directamente desde la propia plantilla. Esto incluye la persona asignada, la ruta de acceso de iteración, los proyectos, etc. en función del proveedor de control de versiones.

## <a name="create-a-work-item"></a>Crear un elemento de trabajo

 Puede acceder a la nueva plantilla desde cualquier detalle de transacción completa al que pueda acceder desde las pestañas Rendimiento, Errores, Disponibilidad, u otras.

1. Para crear un elemento de trabajo, vaya a Detalles de la transacción completa, seleccione un evento y, a continuación, elija **Crear elemento de trabajo** y la plantilla de elemento de trabajo.

    :::image type="content" source="./media/work-item-integration/create-work-item.png" alt-text=" Captura de pantalla de la pestaña Detalles de la transacción completa con Crear elemento de trabajo seleccionada." lightbox="./media/work-item-integration/create-work-item.png":::

1. Se abrirá una nueva pestaña en el explorador que le lleva al sistema de seguimiento seleccionado. En Azure DevOps, puede crear un error o una tarea y, en GitHub, puede crear un problema en el repositorio. Se crea automáticamente un elemento de trabajo con la información contextual proporcionada por Application Insights.

    :::image type="content" source="./media/work-item-integration/github-work-item.png" alt-text=" Captura de pantalla del problema de GitHub creado automáticamente" lightbox="./media/work-item-integration/github-work-item.png":::

    :::image type="content" source="./media/work-item-integration/azure-devops-work-item.png" alt-text=" Captura de pantalla de la creación automática de errores en Azure DevOps." lightbox="./media/work-item-integration/azure-devops-work-item.png":::

## <a name="edit-a-template"></a>Edición de una plantilla

Para editar la plantilla, vaya a la pestaña **Elementos de trabajo** en *Configurar* y seleccione el icono de lápiz situado junto al libro que quiere actualizar.

:::image type="content" source="./media/work-item-integration/edit-template.png" alt-text=" Captura de pantalla de la pestaña Elemento de trabajo con el icono de lápiz Editar seleccionado.":::

Seleccione Editar ![icono Editar](./media/work-item-integration/edit-icon.png) en la parte superior para empezar a editar la plantilla. Las plantillas de elemento de trabajo se basan en [libros de Azure Monitor](../visualize/workbooks-overview.md). La información del elemento de trabajo se genera mediante el lenguaje de consulta de palabras clave. Puede modificar las consultas para agregar más contexto esencial a su equipo. Cuando haya terminado de editar, guarde el libro; para ello, seleccione el icono Guardar ![icono Guardar](./media/work-item-integration/save-icon.png) en la barra de herramientas superior.

:::image type="content" source="./media/work-item-integration/edit-workbook.png" alt-text=" Captura de pantalla del libro de la plantilla de elemento de trabajo en modo de edición." lightbox="./media/work-item-integration/edit-workbook.png":::

Puede crear más de una configuración de elementos de trabajo y tener un libro personalizado adecuado para cada escenario. Los libros también pueden implementarse mediante Azure Resource Manager, lo que garantiza implementaciones estándar entre los entornos.