---
title: Creación y administración de proyectos
description: Busque, cree, administre y elimine proyectos en Azure Migrate.
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: how-to
ms.date: 11/23/2020
ms.openlocfilehash: 0c9102f8ca724e431bb478945c5f4ba0369643d6
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107714852"
---
# <a name="create-and-manage-projects"></a>Creación y administración de proyectos

En este artículo se describe cómo crear, administrar y eliminar [proyectos](migrate-services-overview.md). 

Azure Migrate clásico se retira en febrero de 2024. Después de febrero de 2024, ya no se admitirá la versión clásica de Azure Migrate y se eliminarán los metadatos de inventario del proyecto clásico. Si usa proyectos clásicos, elimine esos proyectos y siga los pasos para crear uno. Los proyectos o componentes clásicos no se pueden actualizar a Azure Migrate. Vea las [preguntas más frecuentes](./resources-faq.md#i-have-a-project-with-the-previous-classic-experience-of-azure-migrate-how-do-i-start-using-the-new-version) antes de iniciar el proceso de creación.

El proyecto se usa para almacenar los metadatos de detección, evaluación y migración recopilados del entorno que se va a evaluar o migrar. En un proyecto puede realizar un seguimiento de los recursos detectados, crear evaluaciones y organizar migraciones a Azure.  

## <a name="verify-permissions"></a>Comprobar los permisos

Compruebe que tenga los permisos correctos para crear un proyecto:

1. En Azure Portal, abra la suscripción correspondiente y seleccione  **Control de acceso (IAM)** .
2. En **Comprobar acceso**, busque la cuenta correspondiente y, después, selecciónela para ver los permisos. Debe tener permisos de *Colaborador* o *Propietario*. 


## <a name="create-a-project-for-the-first-time"></a>Creación de un proyecto por primera vez

Configure un nuevo proyecto en una suscripción de Azure.

1. En Azure Portal, busque *Azure Migrate*.
2. En **Servicios**, seleccione **Azure Migrate**.
3. En **Información general**, seleccione **Evaluar y migrar servidores**.

    :::image type="content" source="./media/create-manage-projects/assess-migrate-servers.png" alt-text="Opción en Información general para evaluar y migrar servidores":::

4. En **Servidores**, seleccione **Crear proyecto**.

    :::image type="content" source="./media/create-manage-projects/create-project.png" alt-text="Botón para empezar a crear el proyecto":::

5. En **Crear proyecto**, seleccione la suscripción y el grupo de recursos de Azure. Cree un grupo de recursos si no tiene ninguno.
6. En **Detalles del proyecto**, especifique el nombre del proyecto y la región geográfica en la que quiere crearlo.
    - La geografía solo se usa para almacenar los metadatos que se recopilan de los servidores locales. Puede seleccionar cualquier región de destino para la migración. 
    - Revise las zonas geográficas admitidas para nubes [públicas](migrate-support-matrix.md#supported-geographies-public-cloud) y [nubes gubernamentales](migrate-support-matrix.md#supported-geographies-azure-government). 


    > [!Note]
    > Use la sección de configuración **Avanzadas** para crear un proyecto de Azure Migrate con conectividad del punto de conexión privado. [Más información](how-to-use-azure-migrate-with-private-endpoints.md#create-a-project-with-private-endpoint-connectivity) 

7. Seleccione **Crear**.

     :::image type="content" source="./media/create-manage-projects/project-details.png" alt-text="Página para especificar la configuración del proyecto":::


Espere unos minutos a que el proyecto se implemente.

## <a name="create-a-project-in-a-specific-region"></a>Creación de un proyecto en una región específica

En el portal, puede seleccionar la geografía en la que quiere crear el proyecto. Si quiere crear el proyecto dentro de una región específica de Azure, use el siguiente comando de API para crearlo.

```rest
PUT /subscriptions/<subid>/resourceGroups/<rg>/providers/Microsoft.Migrate/MigrateProjects/<mymigrateprojectname>?api-version=2018-09-01-preview "{location: 'centralus', properties: {}}"
```

## <a name="create-additional-projects"></a>Creación de proyectos adicionales

Si ya tiene un proyecto y quiere crear otro, haga lo siguiente:  

1. En el [portal público de Azure](https://portal.azure.com) o [Azure Government](https://portal.azure.us), busque **Azure Migrate**.
2. En el panel de Azure Migrate, vaya a **Servidores** y seleccione **Cambiar** en la esquina superior derecha.

    :::image type="content" source="./media/create-manage-projects/switch-project.png" alt-text="Cambiar proyecto":::

3. Para crear un nuevo proyecto, seleccione **Haga clic aquí**.


## <a name="find-a-project"></a>Búsqueda de un programa

Busque un proyecto como se indica a continuación:

1. En [Azure Portal](https://portal.azure.com), busque *Azure Migrate*.
2. En el panel de Azure Migrate, vaya a **Servidores** y seleccione **Cambiar** en la esquina superior derecha.

    :::image type="content" source="./media/create-manage-projects/switch-project.png" alt-text="Cambiar a un proyecto existente":::

3. Seleccione la suscripción y el proyecto adecuados.


### <a name="find-a-classic-project"></a>Búsqueda de un proyecto clásico

Si creó el proyecto en la [versión anterior](migrate-services-overview.md#azure-migrate-versions) de Azure Migrate, haga lo siguiente para buscarlo:

1. En [Azure Portal](https://portal.azure.com), busque *Azure Migrate*.
2. En el panel de Azure Migrate, si ha creado un proyecto en la versión anterior, aparece un banner que hace referencia a los proyectos anteriores. Seleccione el banner.

    :::image type="content" source="./media/create-manage-projects/access-existing-projects.png" alt-text="Acceso a proyectos existentes":::

3. Revise la lista de proyectos anteriores.


## <a name="delete-a-project"></a>Eliminación de un proyecto

Realice la eliminación de la siguiente manera:

1. Abra el grupo de recursos de Azure en el que se ha creado el proyecto.
2. En la página del grupo de recursos, seleccione **Mostrar tipos ocultos**.
3. Seleccione el proyecto que quiere eliminar y los recursos asociados.
    - El tipo de recurso es **Microsoft.Migrate/migrateprojects**.
    - Si el grupo de recursos se utiliza de forma exclusiva en el proyecto, puede eliminarlo por completo.

Observe lo siguiente:

- Al eliminar un proyecto, se eliminan tanto el proyecto como los metadatos sobre los servidores detectados.
- Si usa la versión anterior de Azure Migrate, abra el grupo de recursos de Azure en el que se creó el proyecto. Seleccione el proyecto que quiere eliminar (el tipo de recurso es **Proyecto de migración**).
- Si usa el análisis de dependencias con una área de trabajo de Azure Log Analytics:
    - Si ha vinculado un área de trabajo de Log Analytics a la herramienta Server Assessment, el área de trabajo no se elimina automáticamente. La misma área de trabajo de Log Analytics se puede usar para varios escenarios.
    - Si desea eliminar el área de trabajo de Log Analytics, hágalo manualmente.
- La eliminación de un proyecto es irreversible. Los objetos eliminados no se pueden recuperar.

### <a name="delete-a-workspace-manually"></a>Eliminación manual de un área de trabajo

1. Vaya al área de trabajo de Log Analytics vinculada al proyecto.

    - Si no ha eliminado el proyecto, puede encontrar el vínculo al área de trabajo en **Essentials** > **Server Assessment**.
    :::image type="content" source="./media/create-manage-projects/loganalytics-workspace.png" alt-text="Área de trabajo de Azure Migrate":::
       
    - Si ya ha eliminado el proyecto, seleccione **Grupos de recursos** en el panel izquierdo de Azure Portal y busque el área de trabajo.
       
2. [Siga las instrucciones](../azure-monitor/logs/delete-workspace.md) para eliminar el área de trabajo.

## <a name="next-steps"></a>Pasos siguientes

Agregue herramientas de [evaluación](how-to-assess.md) o [migración](how-to-migrate.md) a los proyectos.