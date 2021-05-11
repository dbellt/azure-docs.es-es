---
title: Administración de aplicaciones lógicas en Azure Portal
description: Edite, habilite, deshabilite o elimine aplicaciones lógicas mediante Azure Portal.
services: logic-apps
ms.suite: integration
author: lauradolan
ms.author: ladolan
ms.reviewer: estfan, jonfan, logicappspm
ms.topic: conceptual
ms.custom: mvc
ms.date: 04/23/2021
ms.openlocfilehash: 4c4a3b1e5d165681e921d2fadeadc5dea9633d41
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108162912"
---
# <a name="manage-logic-apps-in-the-azure-portal"></a>Administración de aplicaciones lógicas en Azure Portal

Puede administrar aplicaciones lógicas mediante [Azure Portal](https://portal.azure.com) o [Visual Studio](manage-logic-apps-with-visual-studio.md). En este artículo se muestra cómo editar, deshabilitar, habilitar o eliminar aplicaciones lógicas en Azure Portal. Si no está familiarizado con Azure Logic Apps, consulte [¿Qué es Azure Logic Apps?](logic-apps-overview.md)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Elija una aplicación lógica existente. Para obtener información sobre cómo crear una aplicación lógica mediante Azure Portal, consulte [Inicio rápido: Creación del primer flujo de trabajo mediante Azure Logic Apps: Azure Portal](quickstart-create-first-logic-app-workflow.md).

<a name="find-logic-app"></a>

## <a name="find-and-open-a-logic-app"></a>Búsqueda y apertura de una aplicación lógica

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

1. En el cuadro de búsqueda del portal, escriba `logic apps` y seleccione **Logic Apps**.

1. En la lista de aplicaciones lógicas, busque la aplicación lógica mediante la exploración o el filtrado de la lista.

1. Para abrir la aplicación lógica, seleccione la aplicación que quiere administrar.

## <a name="view-logic-app-properties"></a>Visualización de las propiedades de la aplicación lógica

1. En Azure Portal, [busque y abra la aplicación lógica](#find-logic-app).

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Propiedades**.

1. En el panel **Propiedades**, puede ver y copiar la siguiente información sobre la aplicación lógica:

   * **Nombre**
   * **Identificador del recurso**
   * **Grupos de recursos**
   * **Ubicación**
   * **Tipo** 
   * **Subscription Name**
   * **Subscription ID**
   * **Punto de conexión de acceso**
   * **Direcciones IP salientes en tiempo de ejecución**
   * **Direcciones IP de puntos de conexión de acceso**
   * **Direcciones IP salientes del conector**

<a name="disable-enable-logic-apps"></a>

## <a name="disable-or-enable-logic-apps"></a>Deshabilitación o habilitación de la aplicaciones lógicas

Para evitar que el desencadenador se active la próxima vez que se cumpla una condición desencadenadora, deshabilite la aplicación lógica. En Azure Portal, puede habilitar o deshabilitar una [sola aplicación lógica](#disable-enable-single-logic-app) o [varias aplicaciones lógicas a la vez](#disable-or-enable-multiple-logic-apps). La deshabilitación de una aplicación lógica tiene los siguientes efectos en las instancias de flujo de trabajo:

* El servicio Logic Apps continúa todas las ejecuciones en curso y pendientes hasta que finalizan. Según el volumen o el trabajo pendiente, este proceso puede tardar en completarse.

* El servicio Logic Apps no crea ni ejecuta nuevas instancias de flujo de trabajo.

* El desencadenador no se activará la próxima vez que se cumplan sus condiciones. Sin embargo, el estado del desencadenador recuerda el punto en el que se detuvo la aplicación lógica. Por lo tanto, si reactiva la aplicación lógica, el desencadenador se activará para todos los elementos no procesados desde la última ejecución.

  Para evitar que el desencadenador se active para los elementos no procesados desde la última ejecución, borre el estado del desencadenador antes de reactivar la aplicación lógica:

  1. En la aplicación lógica, edite cualquier parte del desencadenador del flujo de trabajo.
  1. Guarde los cambios. Este paso restablece el estado actual del desencadenador.
  1. [Reactive la aplicación lógica](#disable-enable-single-logic-app).

<a name="disable-enable-single-logic-app"></a>

### <a name="disable-or-enable-a-single-logic-app"></a>Deshabilitación o habilitación de una sola aplicación lógica

1. En Azure Portal, [busque y abra la aplicación lógica](#find-logic-app).

1. En el menú de la aplicación lógica, seleccione **Información general**.

   * Para deshabilitar la aplicación lógica, en la barra de herramientas del panel Información general, seleccione **Deshabilitar**.
   * Para habilitar la aplicación lógica, en la barra de herramientas del panel Información general, seleccione **habilitar**.

     > [!NOTE]
     > Si la aplicación lógica ya está deshabilitada, solo verá la opción **Habilitar**.
     > Si la aplicación lógica ya está habilitada, solo verá la opción **Deshabilitar**.

1. Para confirmar si la operación se ha completado correctamente o no, abra la lista **Notificaciones** (icono de campana) en la barra de herramientas principal de Azure.

<a name="disable-or-enable-multiple-logic-apps"></a>

### <a name="disable-or-enable-multiple-logic-apps"></a>Deshabilitación o habilitación de varias aplicaciones lógicas

1. En el cuadro de búsqueda principal de Azure Portal, escriba `logic apps` y seleccione **Logic Apps**.

1. En la página **Aplicaciones lógicas,** revise la columna **Estado** de la aplicación lógica.

1. En la columna de casillas, seleccione las aplicaciones lógicas que quiera detener o iniciar.

   * Para detener las aplicaciones lógicas en ejecución seleccionadas, haga clic en **Deshabilitar/Detener** en la barra de herramientas del panel Información general. Confirme la selección.
   * Para iniciar las aplicaciones lógicas detenidas seleccionadas, haga clic en **Habilitar/Iniciar** en la barra de herramientas del panel Información general.

1. Para confirmar si la operación se ha completado correctamente o no, abra la lista **Notificaciones** (icono de campana) en la barra de herramientas principal de Azure.

   > [!TIP]
   > Si no aparece el estado correcto, actualice la página.

<a name="delete-logic-apps"></a>

## <a name="delete-logic-apps"></a>Eliminación de aplicaciones lógicas

Puede eliminar una sola aplicación lógica o varias a la vez. Eliminar una aplicación lógica afecta a las instancias de flujo de trabajo de las maneras siguientes:

* El servicio Logic Apps hace todo lo posible por cancelar las ejecuciones en curso y pendientes.

  Incluso con un gran volumen o trabajo pendiente, la mayoría de las ejecuciones se cancelan antes de que finalicen o se inicien. Sin embargo, el proceso de cancelación puede tardar en completarse. Mientras tanto, pueden seleccionarse algunas ejecuciones para su ejecución mientras el servicio se encarga del proceso de cancelación.

* El servicio Logic Apps no crea ni ejecuta nuevas instancias de flujo de trabajo.

* Si elimina un flujo de trabajo y, luego, vuelve a crear el mismo flujo de trabajo, el flujo de trabajo recreado no tendrá los mismos metadatos que el flujo de trabajo eliminado. Tiene que volver a guardar todo flujo de trabajo que haya llamado al flujo de trabajo eliminado. De este modo, el autor de la llamada obtiene la información correcta para el flujo de trabajo recreado. De lo contrario, las llamadas al flujo de trabajo recreado producirán un error `Unauthorized`. Este comportamiento también se aplica a los flujos de trabajo que usan artefactos en cuentas de integración y a flujos de trabajo que llaman a Azure Functions.

1. En el cuadro de búsqueda principal de Azure Portal, escriba `logic apps` y seleccione **Logic Apps**.

1. En la lista **Aplicaciones lógicas**, en la columna de casillas, seleccione una o varias aplicaciones lógicas que quiera eliminar. En la barra de herramientas, seleccione **Eliminar**.

1. Cuando aparezca el cuadro de confirmación, escriba `yes` y seleccione **Eliminar**.

1. Para confirmar si la operación se ha completado correctamente o no, abra la lista **Notificaciones** (icono de campana) en la barra de herramientas principal de Azure.

<a name="manage-logic-app-versions"></a>

## <a name="manage-logic-app-versions"></a>Administración de versiones de aplicaciones lógicas

Puede usar Azure Portal para el control de versiones de las aplicaciones lógicas. Puede encontrar el historial de versiones de una aplicación lógica y restaurar versiones anteriores.

<a name="find-version-history"></a>

### <a name="find-and-view-previous-versions"></a>Búsqueda y visualización de versiones anteriores

1. En Azure Portal, [busque y abra la aplicación lógica](#find-logic-app).

1. En el menú de la aplicación lógica, en **Herramientas de desarrollo**, seleccione **Versiones**.

   ![Captura de pantalla de la aplicación lógica en Azure Portal, que muestra la selección de la página Versiones en las herramientas de desarrollo.](./media/manage-logic-apps-with-azure-portal/logic-apps-menu-versions.png)

1. En la lista seleccione la **versión** de la aplicación lógica que va a administrar. Puede especificar el identificador de la **versión** en la barra de búsqueda para filtrar la lista.

1. En la página **Historial de versiones**, verá los detalles de la versión anterior en modo de solo lectura. Puede seleccionar entre los modos **Diseñador** de Logic Apps y **vista Código**.

   ![Captura de pantalla de la página History version (Historial de versiones) de Logic Apps, que muestra las opciones de vista de código y vista de diseñador.](./media/manage-logic-apps-with-azure-portal/history-version.png)

<a name="promote-previous-versions"></a>

### <a name="promote-previous-versions"></a>Restauración de versiones anteriores

1. En el historial de versiones de la aplicación lógica, [busque y seleccione la versión que desea promocionar](#find-version-history).

1. En la página **Historial de versiones**, seleccione **Promover**.

   ![Captura de pantalla del historial de versiones de la aplicación lógica, que muestra el botón para promocionar una versión anterior.](./media/manage-logic-apps-with-azure-portal/promote-button.png)

1. En la página **Diseñador de Logic Apps** que se abre, edite la versión que va a promocionar según sea necesario. Puede cambiar entre los modos **Diseñador** y **vista de Código**. También puede actualizar **Parámetros**, **Plantillas** y **Conectores**.

   ![Captura de pantalla del Diseñador de aplicaciones lógicas, que muestra el botón para promocionar una versión anterior de una aplicación lógica.](./media/manage-logic-apps-with-azure-portal/promote-page.png)

1. Para guardar las actualizaciones y finalizar la promoción de la versión anterior, seleccione **Guardar**. (Si desea cancelar los cambios, seleccione **Descartar**). 

   Cuando vuelva a [visualizar el historial de versiones de la aplicación lógica](#find-version-history), la versión promocionada aparece al principio de la lista con un nuevo identificador.

## <a name="next-steps"></a>Pasos siguientes

* [Supervisión de aplicaciones lógicas](monitor-logic-apps.md)
