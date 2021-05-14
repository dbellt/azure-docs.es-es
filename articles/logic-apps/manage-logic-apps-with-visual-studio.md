---
title: Edición y administración de aplicaciones lógicas mediante Visual Studio con Cloud Explorer
description: Editar, actualizar, administrar e implementar aplicaciones lógicas, así como agregarlas al control de código fuente, mediante Visual Studio con Cloud Explorer
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, jonfan, logicappspm
ms.topic: conceptual
ms.custom: mvc
ms.date: 04/23/2021
ms.openlocfilehash: 443dd0a1172c98b67282b50659ffeb3611470413
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108163776"
---
# <a name="manage-logic-apps-with-visual-studio"></a>Administración de aplicaciones lógicas con Visual Studio

Aunque puede crear, editar, administrar e implementar aplicaciones lógicas en [Azure Portal](https://portal.azure.com), también puede usar Visual Studio cuando quiera agregar las aplicaciones lógicas al control de código fuente, publicar versiones diferentes y crear plantillas de [Azure Resource Manager](../azure-resource-manager/management/overview.md) para distintos entornos de implementación. Con Visual Studio Cloud Explorer, puede encontrar y administrar las aplicaciones lógicas junto con otros recursos de Azure. Por ejemplo, puede abrir, descargar, editar, ejecutar, ver el historial de ejecución, deshabilitar y habilitar las aplicaciones lógicas que ya están implementadas en Azure Portal. Si es la primera vez que usa Azure Logic Apps en Visual Studio, obtenga información sobre [cómo crear aplicaciones lógicas con Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

También puede [administrar las aplicaciones lógicas con PowerShell en Azure Portal](manage-logic-apps-with-azure-portal.md).

> [!IMPORTANT]
> Al implementar o publicar una aplicación lógica desde Visual Studio se sobrescribe la versión de esa aplicación en Azure Portal. Por tanto, si hace cambios en Azure Portal que desea mantener, asegúrese de [actualizar la aplicación lógica en Visual Studio](#refresh) desde Azure Portal antes de volver a implementar o publicar desde Visual Studio.

<a name="requirements"></a>

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Descargue e instale estas herramientas si no las tiene aún:

  * [Visual Studio 2019, 2017 o 2015 Community Edition o superior](https://aka.ms/download-visual-studio). Este inicio rápido usa Visual Studio Community 2017, que es gratuito.

    > [!IMPORTANT]
    > Al instalar Visual Studio de 2019 o 2017, asegúrese de que selecciona la carga de trabajo **desarrollo de Azure**.
    > Para obtener más información, vea [Administración de los recursos asociados con las cuentas de Azure en Visual Studio Cloud Explorer](/visualstudio/azure/vs-azure-tools-resources-managing-with-cloud-explorer).

    Para instalar Cloud Explorer para Visual Studio 2015, [descargue Cloud Explorer desde Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=MicrosoftCloudExplorer.CloudExplorerforVisualStudio2015). Para obtener más información, vea [Administración de los recursos asociados con las cuentas de Azure en Visual Studio Cloud Explorer (2015)](/visualstudio/azure/vs-azure-tools-resources-managing-with-cloud-explorer?view=vs-2015&preserve-view=true).

  * [SDK de Azure (2.9.1 o posterior)](https://azure.microsoft.com/downloads/)

  * [Azure PowerShell](https://github.com/Azure/azure-powershell#installation)

  * Las últimas herramientas de Azure Logic Apps para la extensión de Visual Studio para la versión que necesita:

    * [Visual Studio 2019](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2019)

    * [Visual Studio 2017](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2017)

    * [Visual Studio 2015](https://aka.ms/download-azure-logic-apps-tools-visual-studio-2015)

    Puede descargar e instalar Herramientas de Azure Logic Apps directamente desde Visual Studio Marketplace o aprender [a instalar esta extensión desde Visual Studio](/visualstudio/ide/finding-and-using-visual-studio-extensions). Asegúrese de reiniciar Visual Studio después de finalizar la instalación.

  * Para usar suscripciones de Azure Government con Visual Studio, consulte estos temas para ver opciones de instalación adicionales:

    * Visual Studio 2019: [Inicio rápido: Conexión a Azure Government con Visual Studio](../azure-government/documentation-government-connect-vs.md)

    * Visual Studio 2017: [Introducción a la extensión Azure Environment Selector de Visual Studio](https://devblogs.microsoft.com/azuregov/introducing-the-azure-environment-selector-visual-studio-extension/), que puede descargar e instalar desde [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=SteveMichelotti.AzureEnvironmentSelector).

* Acceso a la Web mediante el diseñador integrado de Logic Apps

  El diseñador requiere una conexión a Internet para crear recursos en Azure y leer las propiedades y los datos de los conectores de la aplicación lógica.

<a name="find-logic-apps-vs"></a>

## <a name="find-logic-apps"></a>Búsqueda de aplicaciones lógicas

En Visual Studio, puede encontrar todas las aplicaciones lógicas que están asociadas con la suscripción de Azure y que están implementadas en Azure Portal mediante Cloud Explorer.

1. Abra Visual Studio. En el menú **Ver**, seleccione **Cloud Explorer**.

1. En Cloud Explorer, seleccione el icono **Account Management** (Administración de cuentas). Seleccione la suscripción de Azure asociada con las aplicaciones lógicas y, luego, elija **Aplicar**. Por ejemplo:

   ![Seleccionar "Administración de cuentas"](./media/manage-logic-apps-with-visual-studio/account-management-select-Azure-subscription.png)

1. Junto al icono de **Administración de cuentas**, seleccione **Resource Types** (Tipos de recursos). En la suscripción de Azure, expanda **Logic Apps** para que pueda ver todas las aplicaciones lógicas implementadas asociadas a la suscripción.

Luego, abra la aplicación lógica en el editor de aplicaciones lógicas.

<a name="open-designer"></a>

## <a name="open-logic-apps-in-visual-studio"></a>Abrir aplicaciones lógicas en Visual Studio

En Visual Studio, puede abrir las aplicaciones lógicas anteriormente creadas e implementadas de manera directa mediante Azure Portal o como proyectos de grupo de recursos de Azure con Visual Studio.

1. [Abra Cloud Explorer y busque su aplicación lógica](#find-logic-apps-vs).

1. En el menú contextual de la aplicación lógica, seleccione **Abrir con el editor de aplicaciones lógicas**.

   > [!TIP]
   > Si no tiene este comando en Visual Studio 2019, compruebe que tiene las actualizaciones más recientes de Visual Studio.

   ![Abrir aplicación lógica implementada desde Azure Portal](./media/manage-logic-apps-with-visual-studio/open-logic-app-in-editor.png)

   Una vez que la aplicación lógica se abre en el diseñador de Logic Apps, en la parte inferior del diseñador, puede seleccionar **Vista Código** para poder revisar la estructura de definición de aplicación lógica subyacente. Si quiere crear una plantilla de implementación para la aplicación lógica, obtenga información sobre [cómo descargar una plantilla de Azure Resource Manager](#download-logic-app) para esa aplicación lógica. Obtenga más información sobre las [plantillas de Resource Manager](../azure-resource-manager/templates/overview.md).

<a name="download-logic-app"></a>

## <a name="download-from-azure"></a>Descarga de Azure

Puede [descargar](../azure-resource-manager/templates/export-template-portal.md#export-template-from-a-resource) aplicaciones lógicas desde [Azure Portal](https://portal.azure.com) y guardarlas como [plantillas de Azure Resource Manager](../azure-resource-manager/management/overview.md). Luego puede editar de manera local las plantillas con Visual Studio y personalizar las aplicaciones lógicas para distintos entornos de implementación.  Al descargar las aplicaciones lógicas, se *parametrizan* automáticamente sus definiciones dentro de las [plantillas de Resource Manager](../azure-resource-manager/templates/overview.md), que también usan la notación de objetos JavaScript (JSON).

1. En Visual Studio, mediante Cloud Explorer, [abra la aplicación lógica que quiere descargar de Azure](#open-designer).

1. En el menú contextual de la aplicación lógica, seleccione **Abrir con el editor de aplicaciones lógicas**.

   > [!TIP]
   > Si no tiene este comando en Visual Studio 2019, compruebe que tiene las actualizaciones más recientes de Visual Studio.

   La aplicación lógica se abre en el Diseñador de aplicaciones lógicas.

1. En la barra de herramientas del diseñador, seleccione **Descargar**.

   ![Descargar la aplicación lógica desde Azure Portal](./media/manage-logic-apps-with-visual-studio/download-logic-app-from-portal.png)

1. Cuando se le pida una ubicación, vaya a esa ubicación y guarde la plantilla de Resource Manager para la definición de aplicaciones lógicas en formato de archivo JSON (.json).

   La definición de aplicaciones lógicas aparece en la subsección `resources` de la plantilla de Resource Manager. Ahora puede editar la definición de aplicaciones lógicas y la plantilla de Resource Manager con Visual Studio. También puede agregar la plantilla como un [proyecto de grupo de recursos de Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md) a una solución de Visual Studio. Aprenda sobre los [proyectos de grupo de recursos de Azure para aplicaciones lógicas en Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

<a name="link-integration-account"></a>

## <a name="link-to-integration-account"></a>Vínculo a la cuenta de integración

Para crear aplicaciones lógicas en escenarios de integración empresarial de negocio a negocio (B2B), puede vincular la aplicación lógica a una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) creada anteriormente que exista en la misma región que la aplicación lógica. Una cuenta de integración contiene artefactos B2B, como entidades, acuerdos, esquemas y asignaciones, y permite que la aplicación lógica use conectores B2B para la validación de XML y la codificación o descodificación de archivos sin formato. Aunque puede [crear este vínculo mediante Azure Portal](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md#link-account), también puede usar Visual Studio después de cumplir los [requisitos previos](#requirements), y la aplicación lógica existirá como un archivo JSON (.json) dentro de un [proyecto de grupo de recursos de Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md). Aprenda sobre los [proyectos de grupo de recursos de Azure para aplicaciones lógicas en Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#create-resource-group-project).

1. En Visual Studio, abra el proyecto de grupo de recursos de Azure que contiene la aplicación lógica.

1. En el Explorador de soluciones, abra el menú contextual del archivo **<nombre de la aplicación lógica>.json** y seleccione **Open With Logic App Designer** (Abrir con el Diseñador de aplicación lógica). (Teclado: CTRL + L)

   ![Abrir el archivo .json de la aplicación lógica con el Diseñador de aplicación lógica](./media/manage-logic-apps-with-visual-studio/open-logic-app-designer.png)

   > [!TIP]
   > Si no tiene este comando en Visual Studio 2019, compruebe que tiene las actualizaciones más recientes de Visual Studio y la extensión para las herramientas de Azure Logic Apps.

1. Asegúrese de que el Diseñador de aplicación lógica tiene el foco, para ello, seleccione la pestaña o la superficie del diseñador para que la ventana Propiedades muestre la propiedad **Cuenta de integración** de la aplicación lógica.

   ![Ventana Propiedades: propiedad "Cuenta de integración"](./media/manage-logic-apps-with-visual-studio/open-logic-app-properties-integration-account.png)

   > [!TIP]
   > Si aún no tiene abierta la ventana Propiedades, en el menú **Ver**, seleccione **Ventana Propiedades**. (Teclado: presione F4)

1. Abra la lista de la propiedad **Cuenta de integración** y seleccione la cuenta de integración que quiere vincular a la aplicación lógica, por ejemplo:

   ![Abrir la lista de propiedades "Cuenta de integración"](./media/manage-logic-apps-with-visual-studio/select-integration-account.png)

1. Cuando haya terminado, no olvide guardar la solución de Visual Studio.

Cuando se establece la propiedad **Cuenta de integración** en Visual Studio y se guarda la aplicación lógica como una plantilla de Azure Resource Manager, esa plantilla también incluye una declaración de parámetros para la cuenta de integración seleccionada. Para más información sobre los parámetros de plantilla y las aplicaciones lógicas, consulte [Introducción: Automatización de la implementación de aplicaciones lógicas](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#template-parameters)

<a name="change-location"></a>

## <a name="change-deployment-location"></a>Cambio de la ubicación de implementación

En Visual Studio, si la aplicación lógica existe como un archivo JSON (.json) en un [proyecto de grupo de recursos de Azure](../azure-resource-manager/templates/create-visual-studio-deployment-project.md) que se usa para automatizar la implementación, esa aplicación lógica se establece en un tipo de ubicación y en una ubicación específica. Esta ubicación puede ser una región de Azure o un [entorno del servicio de integración (ISE)](connect-virtual-network-vnet-isolated-environment.md) existente.

Para cambiar el tipo de ubicación o la propia ubicación de la aplicación lógica, tiene que abrir el archivo de definición de flujo de trabajo (.json) de la aplicación lógica desde Explorador de soluciones con el Diseñador de aplicación lógica. Estas propiedades no se pueden cambiar mediante Cloud Explorer.

> [!IMPORTANT]
> El cambio del tipo de ubicación de **Región** a [**Entorno del servicio de integración**](connect-virtual-network-vnet-isolated-environment-overview.md) afecta al [modelo de precios](logic-apps-pricing.md#fixed-pricing) de la aplicación lógica que se usa para la facturación, a los [límites](logic-apps-limits-and-config.md#integration-account-limits), la [compatibilidad con la cuenta de integración](connect-virtual-network-vnet-isolated-environment-overview.md#ise-skus), etc. Antes de seleccionar un tipo de ubicación diferente, asegúrese de que comprende el impacto que tendrá en la aplicación lógica.

1. En Visual Studio, abra el proyecto de grupo de recursos de Azure que contiene la aplicación lógica.

1. En el Explorador de soluciones, abra el menú contextual del archivo `<logic-app-name>.json` y, después, seleccione **Abrir con el Diseñador de aplicación lógica**. (Teclado: CTRL + L)

   ![Abrir el archivo .json de la aplicación lógica con el Diseñador de aplicación lógica](./media/manage-logic-apps-with-visual-studio/open-logic-app-designer.png)

   > [!TIP]
   > Si no tiene este comando en Visual Studio 2019, compruebe que tiene las actualizaciones más recientes de Visual Studio y la extensión para las herramientas de Azure Logic Apps.

1. Asegúrese de que el Diseñador de aplicación lógica tiene el foco, para ello, seleccione la pestaña o la superficie del diseñador para que la ventana Propiedades muestre las propiedades **Elegir tipo de ubicación** y **Ubicación** de la aplicación lógica. El tipo de ubicación del proyecto se establece en **Región** o **Entorno del servicio de integración**.

   ![Ventana Propiedades: propiedades "Elegir tipo de ubicación" y "Ubicación"](./media/manage-logic-apps-with-visual-studio/open-logic-app-properties-location.png)

   > [!TIP]
   > Si aún no tiene abierta la ventana Propiedades, en el menú **Ver**, seleccione **Ventana Propiedades**. (Teclado: presione F4)

1. Para cambiar el tipo de ubicación, abra la lista de la propiedad **Elegir tipo de ubicación** y seleccione el tipo de ubicación que quiere.

   Por ejemplo, si el tipo de ubicación es **Entorno del servicio de integración**, puede seleccionar **Región**.

   ![Propiedad "Elegir tipo de ubicación": cambio del tipo de ubicación](./media/manage-logic-apps-with-visual-studio/change-location-type.png)

1. Para cambiar la ubicación específica, abra la lista de la propiedad **Ubicación**. En función del tipo de ubicación, seleccione la ubicación que quiere, por ejemplo:

   * Seleccione una región de Azure diferente:

     ![Abrir la lista de la propiedad "Ubicación" y seleccionar otra región de Azure](./media/manage-logic-apps-with-visual-studio/change-azure-resource-group-region.png)

   * Seleccione otro ISE:

     ![Abrir la lista de la propiedad "Ubicación" y seleccionar otro ISE](./media/manage-logic-apps-with-visual-studio/change-integration-service-environment.png)

1. Cuando haya terminado, no olvide guardar la solución de Visual Studio.

Al cambiar el tipo de ubicación o la propia ubicación en Visual Studio y guardar la aplicación lógica como una plantilla de Azure Resource Manager, esa plantilla también incluye una declaración de parámetros para esa ubicación y ese tipo de ubicación. Para más información sobre los parámetros de plantilla y las aplicaciones lógicas, consulte [Introducción: Automatización de la implementación de aplicaciones lógicas](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#template-parameters)

<a name="refresh"></a>

## <a name="refresh-from-azure"></a>Actualización desde Azure

Si edita la aplicación lógica en Azure Portal y quiere mantener esos cambios, asegúrese de actualizar la versión de esa aplicación en Visual Studio con esos cambios.

* En Visual Studio, en la barra de herramientas del Diseñador de aplicaciones lógicas, seleccione **Actualizar**.

  O bien

* En Visual Studio Cloud Explorer, abra el menú contextual de la aplicación lógica y seleccione **Actualizar**.

![Actualización de aplicación lógica con actualizaciones](./media/manage-logic-apps-with-visual-studio/refresh-logic-app-with-updates-from-portal.png)

## <a name="publish-logic-app-updates"></a>Publicación de actualizaciones de la aplicación lógica

Cuando esté listo para implementar las actualizaciones de la aplicación lógica desde Visual Studio a Azure, en la barra de herramientas del Diseñador de aplicaciones lógicas, seleccione **Publicar**.

![Publicar la aplicación lógica actualizada en Azure Portal](./media/manage-logic-apps-with-visual-studio/publish-logic-app-to-azure-portal.png)

## <a name="manually-run-your-logic-app"></a>Ejecución manual de la aplicación lógica

Puede desencadenar manualmente una aplicación lógica implementada en Azure desde Visual Studio. En la barra de herramientas del Diseñador de aplicaciones lógicas, seleccione **Ejecutar desencadenador**.

![Ejecutar manualmente el desencadenador para la aplicación lógica](./media/manage-logic-apps-with-visual-studio/manually-run-logic-app.png)

## <a name="review-run-history"></a>Revisar el historial de ejecución.

Para comprobar el estado y diagnosticar problemas con las ejecuciones de la aplicación lógica, puede revisar los detalles, como las entradas y las salidas, para esas ejecuciones en Visual Studio.

1. En Cloud Explorer, abra el menú contextual de la aplicación lógica y seleccione **Abrir el historial de ejecución**.

   ![Abrir el historial de ejecución de la aplicación lógica](./media/manage-logic-apps-with-visual-studio/open-run-history-for-logic-app.png)

1. Para ver los detalles de una ejecución específica, haga doble clic en ella. Por ejemplo:

   ![Ver información acerca de una ejecución específica](./media/manage-logic-apps-with-visual-studio/view-run-history-details.png)
  
   > [!TIP]
   > Para ordenar la tabla por propiedad, seleccione el encabezado de columna de esa propiedad.

1. Expanda los pasos cuyas entradas y salidas desea revisar, por ejemplo:

   ![Ver las entradas y salidas de cada paso](./media/manage-logic-apps-with-visual-studio/view-run-history-inputs-outputs.png)

<a name="disable-enable-logic-apps"></a>

## <a name="disable-or-enable-logic-apps"></a>Deshabilitación o habilitación de la aplicaciones lógicas

Para evitar que el desencadenador se active la próxima vez que se cumpla una condición desencadenadora, deshabilite la aplicación lógica. La deshabilitación de una aplicación lógica tiene los siguientes efectos en las instancias de flujo de trabajo:

* El servicio Logic Apps continúa todas las ejecuciones en curso y pendientes hasta que finalizan. Según el volumen o el trabajo pendiente, este proceso puede tardar en completarse.

* El servicio Logic Apps no crea ni ejecuta nuevas instancias de flujo de trabajo.

* El desencadenador no se activará la próxima vez que se cumplan sus condiciones.

* El estado del desencadenador recuerda el punto en el que se detuvo la aplicación lógica. Por lo tanto, si reactiva la aplicación lógica, el desencadenador se activará para todos los elementos no procesados desde la última ejecución.

  Para evitar que el desencadenador se active con elementos no procesados desde la última ejecución, borre el estado del desencadenador antes de reactivar la aplicación lógica:

  1. En la aplicación lógica, edite cualquier parte del desencadenador del flujo de trabajo.
  1. Guarde los cambios. Este paso restablece el estado actual del desencadenador.
  1. [Reactive la aplicación lógica](#enable-logic-apps).

<a name="disable-logic-apps"></a>

### <a name="disable-logic-apps"></a>Deshabilitación de aplicaciones lógicas

En Cloud Explorer, abra el menú contextual de la aplicación lógica y seleccione **Deshabilitar**.

![Deshabilitar la aplicación lógica en Cloud Explorer](./media/manage-logic-apps-with-visual-studio/disable-logic-app-cloud-explorer.png)

<a name="enable-logic-apps"></a>

### <a name="enable-logic-apps"></a>Habilitación de aplicaciones lógicas

En Cloud Explorer, abra el menú contextual de la aplicación lógica y seleccione **Habilitar**.

![Habilitar la aplicación lógica en Cloud Explorer](./media/manage-logic-apps-with-visual-studio/enable-logic-app-cloud-explorer.png)

<a name="delete-logic-apps"></a>

## <a name="delete-logic-apps"></a>Eliminación de aplicaciones lógicas

Eliminar una aplicación lógica afecta a las instancias de flujo de trabajo de las maneras siguientes:

* El servicio Logic Apps hace todo lo posible por cancelar las ejecuciones en curso y pendientes.

  Incluso con un gran volumen o trabajo pendiente, la mayoría de las ejecuciones se cancelan antes de que finalicen o se inicien. Sin embargo, el proceso de cancelación puede tardar en completarse. Mientras tanto, pueden seleccionarse algunas ejecuciones para su ejecución mientras el entorno en tiempo de ejecución se encarga del proceso de cancelación.

* El servicio Logic Apps no crea ni ejecuta nuevas instancias de flujo de trabajo.

* Si elimina un flujo de trabajo y, luego, vuelve a crear el mismo flujo de trabajo, el flujo de trabajo recreado no tendrá los mismos metadatos que el flujo de trabajo eliminado. Tiene que volver a guardar todo flujo de trabajo que haya llamado al flujo de trabajo eliminado. De este modo, el autor de la llamada obtiene la información correcta para el flujo de trabajo recreado. De lo contrario, las llamadas al flujo de trabajo recreado producirán un error `Unauthorized`. Este comportamiento también se aplica a los flujos de trabajo que usan artefactos en cuentas de integración y a flujos de trabajo que llaman a Azure Functions.

Para eliminar la aplicación lógica desde Azure Portal, en Cloud Explorer, abra el menú contextual de la aplicación lógica y seleccione **Eliminar**.

![Eliminar la aplicación lógica desde Azure Portal](./media/manage-logic-apps-with-visual-studio/delete-logic-app-from-azure-portal.png)

## <a name="troubleshooting"></a>Solución de problemas

Cuando se abre el proyecto de aplicación lógica en el Diseñador de aplicaciones lógicas, es posible que no tenga la opción de seleccionar la suscripción a Azure. En cambio, la aplicación lógica se abre con una suscripción a Azure que no es la que quiere usar. Este comportamiento ocurre porque después de abrir el archivo .json de una aplicación lógica, Visual Studio almacena en caché la primera suscripción seleccionada para su uso futuro. Para solucionar este problema, pruebe uno de estos pasos:

* Cambie el nombre del archivo .json de la aplicación lógica. La caché de suscripción depende del nombre del archivo.

* Para eliminar las suscripciones previamente seleccionadas de *todas* las aplicaciones lógicas de la solución, elimine la carpeta de configuración oculta de Visual Studio (.vs) del directorio de la solución. Esta ubicación almacena la información de suscripción.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a administrar las aplicaciones lógicas implementadas con Visual Studio. A continuación, obtenga información sobre cómo personalizar las definiciones de aplicaciones lógicas para la implementación:

> [!div class="nextstepaction"]
> [Creación de definiciones de aplicaciones lógicas en JSON](../logic-apps/logic-apps-author-definitions.md)
