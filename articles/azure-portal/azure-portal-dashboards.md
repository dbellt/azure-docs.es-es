---
title: Crear un panel en el Azure Portal
description: En este artículo se describe cómo crear y personalizar un panel en Azure Portal.
ms.assetid: ff422f36-47d2-409b-8a19-02e24b03ffe7
ms.topic: how-to
ms.date: 05/06/2021
ms.openlocfilehash: 668aead7b448cd462fab80003e498e956a6484d2
ms.sourcegitcommit: 89c4843ec85d1baea248e81724781d55bed86417
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108794947"
---
# <a name="create-a-dashboard-in-the-azure-portal"></a>Crear un panel en el Azure Portal

Los paneles son una vista centrada y organizada de los recursos de nube de Azure Portal. Use los paneles como un área de trabajo donde puede supervisar los recursos e iniciar tareas rápidamente para las operaciones diarias. Por ejemplo, puede crear paneles personalizados basados en proyectos, tareas o roles de usuario.

Azure Portal proporciona un panel predeterminado como punto de partida. Puede editar el panel predeterminado y crear y personalizar paneles adicionales.

> [!NOTE]
> Cada usuario puede crear hasta 100 paneles privados. Si [publica y comparte el panel](azure-portal-dashboard-share-access.md), se implementará como un recurso de Azure en la suscripción y no contará para este límite.

En este artículo se describe cómo crear un nuevo panel y personalizarlo. Para más información sobre cómo compartir paneles, consulte [Uso compartido de paneles de Azure mediante el control de acceso basado en rol de Azure](azure-portal-dashboard-share-access.md).

## <a name="create-a-new-dashboard"></a>Creación de un panel

En este ejemplo se muestra cómo crear un nuevo panel privado con un nombre asignado. Todos los paneles son privados cuando se crean, aunque puede optar por publicar y compartir el panel con otros usuarios de la organización si lo desea.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En el menú de Azure Portal, seleccione **Panel**. Es posible que la vista predeterminada ya esté definida en el panel.

    ![Captura de pantalla de Azure Portal con el panel seleccionado.](./media/azure-portal-dashboards/portal-menu-dashboard.png)

1. Seleccione **Nuevo panel** y, a continuación, **Panel en blanco**.

    ![Captura de pantalla de las opciones del nuevo panel.](./media/azure-portal-dashboards/create-new-dashboard.png)

    Esta acción abre la **Galería de iconos**, desde la que puede seleccionar los iconos, y una cuadrícula vacía, desde donde podrá organizarlos.

1. Seleccione el texto **Mi panel** en la etiqueta del panel y escriba un nombre que le ayude a identificar fácilmente el panel personalizado.

    :::image type="content" source="media/azure-portal-dashboards/dashboard-name.png" alt-text="Captura de pantalla de una cuadrícula vacía con la Galería de iconos.":::

1. Para guardar el panel tal y como está, seleccione **Personalización finalizada** en el encabezado de la página. O bien, continúe con el paso 2 para agregar iconos y guardar el panel.

Ahora, la vista de panel muestra el panel nuevo. Seleccione la flecha situada junto al nombre del panel para ver los paneles disponibles. La lista puede incluir paneles que otros usuarios hayan creado y compartido.

## <a name="edit-a-dashboard"></a>Edición de paneles

Ahora vamos a editar el panel para agregar y organizar los iconos que representan los recursos de Azure y cambiar su tamaño.

### <a name="add-tiles-from-the-tile-gallery"></a>Adición de iconos desde la galería de iconos

Para agregar iconos a un panel, siga estos pasos:

1. Seleccione ![icono de edición](./media/azure-portal-dashboards/dashboard-edit-icon.png) **Editar** en el encabezado de la página del panel.

    ![Captura de pantalla del panel con la opción de edición resaltada.](./media/azure-portal-dashboards/dashboard-edit.png)

1. Examine la **Galería de iconos** o use el campo de búsqueda para buscar un icono determinado. Seleccione el icono que desea agregar al panel.

   :::image type="content" source="media/azure-portal-dashboards/dashboard-tile-gallery.png" alt-text="Captura de pantalla de la galería de iconos.":::

1. Seleccione **Agregar** para agregar el icono al panel con una ubicación y un tamaño predeterminados. O bien, arrastre el icono a la cuadrícula y colóquelo donde quiera. Agregue los iconos que desee, pero aquí tiene un par de ideas:

    - Agregue **Todos los recursos** para ver los recursos que ya ha creado.

    - Si trabaja con más de una organización, agregue el icono **Identidad de la organización** al panel para mostrar con claridad a qué organización pertenecen los recursos.

1. Si lo desea, [cambie el tamaño o reordene](#resize-or-rearrange-tiles) los iconos.

1. Para guardar los cambios, seleccione **Guardar** en el encabezado de la página. También puede obtener una vista previa de los cambios sin guardarlos; para ello, seleccione **Vista previa** en el encabezado de la página. Este modo de vista previa también le permite ver cómo afectan los [filtros](#set-and-override-dashboard-filters) a los iconos. En la pantalla de vista previa, puede seleccionar **Guardar** para conservar los cambios, **Descartar** para quitarlos o **Editar** para volver a las opciones de edición y realizar más cambios.

   :::image type="content" source="media/azure-portal-dashboards/dashboard-save.png" alt-text="Captura de pantalla de las opciones Vista previa, Guardar y Descartar.":::

> [!NOTE]
> Un icono de Markdown le permite mostrar contenido estático personalizado en el panel. Podrían ser instrucciones básicas, una imagen, un conjunto de hipervínculos o incluso la información de contacto. Para más información sobre el uso de un icono de Markdown, vea [Uso de un icono de Markdown en los paneles de Azure para mostrar contenido personalizado](azure-portal-markdown-tile.md).

### <a name="pin-content-from-a-resource-page"></a>Anclaje del contenido desde una página de recursos

Otra manera de agregar iconos al panel es directamente desde una página de recursos.

Muchas páginas de recursos incluyen una imagen de chincheta en el encabezado de página, lo que significa que puede anclar un icono que represente la página de origen. En algunos casos, una imagen de chincheta también puede aparecer para cada contenido específico dentro de una página, lo que significa que puede anclar un icono para ese contenido específico en lugar de para toda la página.

:::image type="content" source="media/azure-portal-dashboards/dashboard-pin-blade.png" alt-text="Captura de pantalla de la barra de comandos de la página con la imagen de chincheta.":::

Si selecciona esta imagen, puede anclar el icono a un panel privado o compartido existente. También puede crear un panel que incluya esta imagen de chincheta seleccionando **Crear nuevo**.

:::image type="content" source="media/azure-portal-dashboards/dashboard-pin-pane.png" alt-text="Captura de pantalla de las opciones para anclar al panel.":::

### <a name="resize-or-rearrange-tiles"></a>Reorganizar los iconos o cambiar su tamaño

Para cambiar el tamaño de un icono o para reorganizar los iconos de un panel, siga estos pasos:

1. Seleccione ![icono de edición](./media/azure-portal-dashboards/dashboard-edit-icon.png) **Editar** en el encabezado de la página.

1. Seleccione el menú contextual en la esquina superior derecha de un icono. A continuación, elija un tamaño de icono. Los iconos que admiten cualquier tamaño también incluyen un "identificador" en la esquina inferior derecha que le permite arrastrar el icono al tamaño que quiera.

    :::image type="content" source="media/azure-portal-dashboards/dashboard-tile-resize.png" alt-text="Captura de pantalla del panel con el menú de tamaño de icono abierto.":::

1. Seleccione un icono y arrástrelo a una nueva ubicación en la cuadrícula para organizar el panel.

### <a name="set-and-override-dashboard-filters"></a>Establecimiento e invalidación de filtros de panel

Cerca de la parte superior del panel, verá opciones para configurar los valores **Actualizar automáticamente** y **Configuración de hora** para los datos que se muestran en el panel, junto con una opción para agregar filtros adicionales.

:::image type="content" source="media/azure-portal-dashboards/dashboard-global-filters.png" alt-text="Captura de pantalla que muestra los filtros globales de un panel.":::

De forma predeterminada, los datos se actualizan cada hora. Para cambiar este valor, seleccione **Actualizar automáticamente** y elija un nuevo intervalo de actualización. Cuando haya realizado la selección, seleccione **Aplicar**.

La configuración de hora predeterminada es **Hora UTC**, que muestra los datos de las **Últimas 24 horas**. Para cambiar este valor, seleccione el botón y elija un nuevo intervalo de tiempo, la granularidad de tiempo o la zona horaria y, luego, seleccione **Aplicar**.

Para aplicar filtros adicionales, seleccione **Agregar filtros**. Las opciones que vea variarán en función de los iconos del panel. Por ejemplo, puede mostrar solo los datos correspondientes a una suscripción o una ubicación específicas. Seleccione el filtro que quiere usar y realice las selecciones. A continuación, el filtro se aplicará a los datos. Para quitarlo, seleccione la **X** en su botón.

Los iconos que admiten el filtrado tienen una imagen de un filtro ![imagen de un filtro](./media/azure-portal-dashboards/dashboard-filter.png) en su esquina superior izquierda. Algunos iconos permiten invalidar los filtros globales con filtros específicos de ese icono. Para ello, seleccione **Configure tile data** (Configurar datos de icono) en el menú contextual o seleccione la imagen de filtro y, luego, aplique los filtros deseados.

Si establece filtros para un icono determinado, la esquina izquierda de ese icono muestra una imagen de doble filtro, lo que indica que los datos de ese icono reflejan sus propios filtros.

:::image type="content" source="media/azure-portal-dashboards/dashboard-filter-override.png" alt-text="Captura de pantalla que muestra la imagen de un icono con una invalidación de filtro.":::

## <a name="modify-tile-settings"></a>Modificación de la configuración del icono

Puede que sea necesario configurar adicionalmente algunos iconos para que muestren la información necesaria. Por ejemplo, el icono **Gráfico de métricas** debe configurarse para mostrar una métrica de Azure Monitor. También puede personalizar los datos del icono para invalidar la configuración de tiempo predeterminada del panel.

## <a name="complete-tile-configuration"></a>Configuración completa del icono

Cualquier icono que deba configurarse muestra un banner hasta que personaliza. Por ejemplo, en el gráfico **Métricas**, el mensaje emergente dice **Editar en Métricas**. Otros mensajes emergentes pueden usar texto diferente, por ejemplo, **Configurar icono**.

Para personalizar el icono:

1. En el encabezado de página, seleccione **Guardar** para salir del modo de edición.

1. Seleccione el banner y, a continuación, realice la configuración necesaria.

    ![Captura de pantalla del icono que requiere configuración](./media/azure-portal-dashboards/dashboard-configure-tile.png)

### <a name="customize-time-span-for-a-tile"></a>Personalización del intervalo de tiempo de un icono

Los datos del panel muestran la actividad y las actualizaciones en función de los filtros globales. Algunos iconos le permitirán seleccionar un intervalo de tiempo diferente para un solo icono. Para ello, siga estos pasos:

1. Seleccione **Personalizar los datos del icono** en el menú contextual o la ![imagen de filtro](./media/azure-portal-dashboards/dashboard-filter.png) de la esquina superior izquierda del icono.

    ![Captura de pantalla del menú contextual del icono.](./media/azure-portal-dashboards/dashboard-customize-tile-data.png)

1. Active la casilla para **Anular la configuración de tiempo del panel en el nivel de icono**.

    ![Captura de pantalla del cuadro de diálogo para configurar las opciones de tiempo del icono.](./media/azure-portal-dashboards/dashboard-override-time-settings.png)

1. Elija el intervalo de tiempo que se mostrará para este icono. Puede elegir entre los últimos 30 minutos y los últimos 30 días o definir un intervalo personalizado.

1. Elija la granularidad de tiempo que se mostrará.  Puede mostrar incrementos de entre un minuto y un mes.

1. Seleccione **Aplicar**.

## <a name="delete-a-tile"></a>Eliminar un icono

Para quitar un icono de un panel, realice una de las acciones siguientes:

- Seleccione el menú contextual en la esquina superior derecha del icono y, a continuación, seleccione **Quitar del panel**.

- Seleccione ![icono de edición](./media/azure-portal-dashboards/dashboard-edit-icon.png) **Editar** para acceder al modo de personalización. Mantenga el puntero en la esquina superior derecha del icono y, a continuación, seleccione el icono de eliminación ![icono de eliminación](./media/azure-portal-dashboards/dashboard-delete-icon.png) para quitar el icono del panel.

   ![Captura de pantalla que muestra cómo quitar un icono del panel.](./media/azure-portal-dashboards/dashboard-delete-tile.png)

## <a name="clone-a-dashboard"></a>Clonar un panel

Para usar un panel existente como plantilla para uno nuevo, siga estos pasos:

1. Asegúrese de que la vista de panel muestra el panel que quiere copiar.

1. En el encabezado de página, seleccione ![icono para clonar](./media/azure-portal-dashboards/dashboard-clone.png) **Clonar**.

1. Se abre una copia del panel denominada **Clon de** *nombre del panel* en modo de edición. Utilice los pasos anteriores de este artículo para cambiar el nombre del panel y personalizarlo.

## <a name="publish-and-share-a-dashboard"></a>Publicar y compartir un panel

Al crear un panel, será privado de forma predeterminada, lo que significa que será la única persona que puede verlo. Para poner paneles a disposición de otros, puede publicarlos y compartirlos. Para más información, consulte [Uso compartido de paneles de Azure mediante el control de acceso basado en rol de Azure](azure-portal-dashboard-share-access.md).

### <a name="open-a-shared-dashboard"></a>Abrir un panel compartido

Para buscar y abrir un panel compartido, siga estos pasos:

1. Seleccione la flecha junto al nombre del panel.

1. Realice la selección a partir de la lista de paneles que se muestra. Si no se muestra el panel que desea abrir:

    1. seleccione **Examinar todos los paneles**.

        ![Captura de pantalla del menú de selección del panel](./media/azure-portal-dashboards/dashboard-browse.png)

    1. En el campo **Tipo**, seleccione **Paneles compartidos**.

        ![Captura de pantalla del menú de selección de todos los paneles](./media/azure-portal-dashboards/dashboard-browse-all.png)

    1. Seleccione una o varias suscripciones. También puede escribir texto para filtrar los paneles por nombre.

    1. Seleccione un panel en la lista de paneles compartidos.

## <a name="delete-a-dashboard"></a>Eliminar un panel

Para eliminar permanentemente un panel privado o compartido, siga estos pasos:

1. Seleccione el panel que quiera eliminar en la lista situada junto al nombre del panel.

1. Seleccione ![icono de eliminación](./media/azure-portal-dashboards/dashboard-delete-icon.png) **Eliminar** en el encabezado de página.

1. Para un panel privado, seleccione **Aceptar** en el cuadro de diálogo de confirmación para quitar el panel. Para un panel compartido, en el cuadro de diálogo de confirmación, active la casilla para confirmar que otros usuarios ya no podrán ver el panel publicado. Después, seleccione **Aceptar**.

    ![Captura de pantalla de la confirmación de la eliminación.](./media/azure-portal-dashboards/dashboard-delete-dash.png)

## <a name="recover-a-deleted-dashboard"></a>Recuperación de un panel eliminado

Si se encuentra en la nube global de Azure y elimina un panel _publicado_ en Azure Portal, puede recuperar el panel en el plazo de 14 días a partir de la eliminación. Para más información, consulte [Recuperación de un panel eliminado en Azure Portal](recover-shared-deleted-dashboard.md).

## <a name="next-steps"></a>Pasos siguientes

- [Uso compartido de paneles de Azure mediante el control de acceso basado en rol de Azure](azure-portal-dashboard-share-access.md)
- [Creación mediante programación de paneles de Azure](azure-portal-dashboards-create-programmatically.md)
