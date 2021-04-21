---
title: Creación de flujos de trabajo de automatización basados en una programación con Azure
description: 'Tutorial: Creación de un flujo de trabajo de automatización periódico basado en una programación que se integra en servicios en la nube mediante el uso de Azure Logic Apps.'
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: tutorial
ms.custom: mvc
ms.date: 03/24/2021
ms.openlocfilehash: 0dab2f49043f3a59dd4f06633e3f0228fc39a424
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107792100"
---
# <a name="tutorial-create-schedule-based-and-recurring-automation-workflows-with-azure-logic-apps"></a>Tutorial: Creación de flujos de trabajo de automatización periódicos basados en programación con Azure Logic Apps

En este tutorial se muestra cómo crear una [aplicación lógica](../logic-apps/logic-apps-overview.md) que automatiza un flujo de trabajo que se ejecuta según una programación periódica. En concreto, esta aplicación lógica de ejemplo comprueba el tiempo de desplazamiento, incluido el tráfico, entre dos lugares y se ejecuta cada día por la mañana. Si el tiempo supera un límite específico, la aplicación lógica envía un correo electrónico con el tiempo de desplazamiento y el tiempo adicional necesario para llegar al destino. El flujo de trabajo incluye varios pasos, que se inician con un desencadenador basado en una programación seguido de una acción de Mapas de Bing, una acción de operaciones de datos, una acción de flujo de control y una acción de notificación por correo electrónico.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una aplicación lógica en blanco.
> * Agregar un desencadenador de periodicidad que especifique la programación de la aplicación lógica.
> * Agregar una acción de Bing Maps que obtenga el tiempo de desplazamiento para una ruta.
> * Agregar una acción que cree una variable, convierta el tiempo de desplazamiento de segundos a minutos y guarde ese resultado en la variable.
> * Agregar una condición que compara el tiempo de desplazamiento con un límite especificado.
> * Agregar una acción que envíe un correo electrónico si el tiempo de desplazamiento supera el límite.

Cuando haya terminado, la aplicación lógica se parecerá a este flujo de trabajo, en un alto nivel:

![Captura de pantalla que muestra la información general de alto nivel de un flujo de trabajo de aplicación lógica de ejemplo.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/check-travel-time-overview.png)

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* Una cuenta de correo electrónico con un proveedor de correo electrónico compatible con Logic Apps, como Office 365 Outlook, Outlook.com o Gmail. En el caso de otros proveedores, [consulte la lista de conectores que se muestra aquí](/connectors/). Este inicio rápido utiliza Office 365 Outlook con una cuenta profesional o educativa. Si utiliza una cuenta de correo electrónico diferente, los pasos generales siguen siendo los mismos, pero la interfaz de usuario podría ser ligeramente distinta.

  > [!IMPORTANT]
  > Si quiere usar el conector de Gmail, solo las cuentas empresariales de G-Suite pueden usar este conector sin restricciones en las aplicaciones lógicas. Si tiene una cuenta de consumidor de Gmail, puede usar este conector solo con servicios específicos aprobados por Google, o puede [crear una aplicación cliente de Google para usarla en la autenticación con el conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Para obtener el tiempo de desplazamiento para una ruta, necesita una clave de acceso para la API de Mapas de Bing. Para obtener esta clave, siga los pasos sobre [cómo obtener una clave de Mapas de Bing](/bingmaps/getting-started/bing-maps-dev-center-help/getting-a-bing-maps-key).

* Si la aplicación lógica necesita comunicarse a través de un firewall que limite el tráfico a direcciones IP específicas, ese firewall debe permitir el acceso para *ambas* direcciones IP, la [de entrada](logic-apps-limits-and-config.md#inbound) y la [de salida](logic-apps-limits-and-config.md#outbound), utilizadas por el servicio Logic Apps o por el entorno de ejecución en la región de Azure donde se encuentra la aplicación lógica. Si la aplicación lógica también usa [conectores administrados](../connectors/managed.md), como el conector de Outlook de Office 365 o el conector de SQL, o emplea [conectores personalizados](/connectors/custom-connectors/), el firewall también debe permitir el acceso a *todas* las [direcciones IP de salida del conector administrado](logic-apps-limits-and-config.md#outbound) en la región de Azure de la aplicación lógica.

## <a name="create-your-logic-app"></a>Creación de una aplicación lógica

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure. En la página principal de Azure, seleccione **Crear un recurso**.

1. En el menú de Azure Marketplace, seleccione **Integración** > **Logic App**.

   ![Captura de pantalla que muestra el menú de Azure Marketplace con las opciones "Integración" y "Logic App" seleccionadas.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-new-logic-app-resource.png)

1. En el panel de **Logic App**, proporcione la información que se describe aquí sobre la aplicación lógica que desea crear.

   ![Captura de pantalla que muestra el panel de creación de aplicaciones lógicas y la información que se debe proporcionar para la nueva aplicación lógica.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-logic-app-settings.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Suscripción** | <*Azure-subscription-name*> | El nombre de la suscripción de Azure. En este ejemplo se usa `Pay-As-You-Go`. |
   | **Grupos de recursos** | LA-TravelTime-RG | El nombre del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) que se utiliza para organizar recursos relacionados. En este ejemplo se crea un nuevo grupo de recursos llamado `LA-TravelTime-RG`. |
   | **Nombre** | LA TravelTime | El nombre de la aplicación lógica, que solo puede contener letras, números, guiones (`-`), caracteres de subrayado (`_`), paréntesis (`(`, `)`) y puntos (`.`). En este ejemplo se usa `LA-TravelTime`. |
   | **Ubicación** | Oeste de EE. UU. | La región en la que desea almacenar la información de la aplicación lógica. En este ejemplo se usa `West US`. |
   | **Log Analytics** | Off | Mantenga el valor **Off** para el registro de diagnóstico. |
   ||||

1. Seleccione **Revisar y crear** cuando haya terminado. Una vez que Azure valide la información sobre la aplicación lógica, seleccione **Crear**.

1. Una vez que Azure implemente la aplicación, seleccione **Ir al recurso**.

   Azure abre el panel de selección de plantillas de aplicaciones lógicas, que muestra un vídeo de introducción, los desencadenadores usados frecuentemente y los patrones de plantillas de aplicaciones lógicas.

1. Desplácese hacia abajo más allá del vídeo y de los desencadenadores frecuentes hasta la sección **Plantillas** y seleccione **Aplicación lógica en blanco**.

   ![Captura de pantalla que muestra el panel de selección de plantilla de aplicaciones lógicas con la opción "Aplicación lógica en blanco" seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-logic-app-template.png)

A continuación, agregue el [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) Periodicidad, que ejecuta el flujo de trabajo en función de una programación especificada. Cada aplicación lógica debe comenzar con un desencadenador, que se activa cuando sucede un evento específico o cuando hay nuevos datos que cumplen una condición determinada. Para más información, consulte [Creación de una nueva aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="add-the-recurrence-trigger"></a>Adición del desencadenador Periodicidad

1. En el cuadro de búsqueda del diseñador de Logic Apps, escriba `recurrence` y seleccione el desencadenador denominado **Periodicidad**.

   ![Captura de pantalla que muestra el cuadro de búsqueda del diseñador de Logic Apps que contiene el término de búsqueda "periodicidad" y, en la lista "Desencadenadores", aparece seleccionado el desencadenador "Periodicidad".](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-schedule-recurrence-trigger.png)

1. En la forma **Periodicidad**, seleccione el botón de **puntos suspensivos** ( **...** ) y luego **Cambiar de nombre**. Cambie el nombre del desencadenador por esta descripción:`Check travel time every weekday morning`

   ![Captura de pantalla que muestra el botón de puntos suspensivos seleccionado, la lista "Configuración" abierta y el comando "Cambiar de nombre" seleccionado.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/rename-recurrence-schedule-trigger.png)

1. En el desencadenador, cambie estas propiedades como se muestra a continuación.

   ![Captura de pantalla que muestra los cambios en el intervalo y la frecuencia del desencadenador.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/change-interval-frequency.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Intervalo** | Sí | 1 | Número de intervalos que se espera entre comprobaciones |
   | **Frecuencia** | Sí | Semana | Unidad de tiempo que se usa para la periodicidad. |
   |||||

1. En **Intervalo** y **Frecuencia**, abra la lista **Agregar nuevo parámetro** y seleccione estas propiedades para agregarlas al desencadenador.

   * **En estos días**
   * **A estas horas**
   * **En estos minutos**

   ![Captura de pantalla que muestra la lista "Agregar nuevo parámetro" abierta y las propiedades seleccionadas: "En estos días", "A estas horas" y "En estos minutos".](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-trigger-properties.png)

1. Ahora, establezca los valores de las propiedades adicionales como se muestra y se describe aquí.

   ![Captura de pantalla que muestra las propiedades adicionales establecidas en los valores tal y como se describe en la tabla siguiente.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/recurrence-trigger-property-values.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **En estos días** | Lunes,Martes,Miércoles,Jueves,Viernes | Este valor solo está disponible cuando se establece la **Frecuencia** en **Semana**. |
   | **A estas horas** | 7,8,9 | Este valor solo está disponible cuando se establece la **Frecuencia** en **Semana** o **Día**. Para esta periodicidad, seleccione las horas del día. En este ejemplo, la ejecución se realiza a las marcas de hora `7`, `8` y `9`. |
   | **En estos minutos** | 0,15,30,45 | Este valor solo está disponible cuando se establece la **Frecuencia** en **Semana** o **Día**. Para esta periodicidad, seleccione los minutos del día. Este ejemplo se inicia en la marca de hora cero y se ejecuta cada 15 minutos. |
   ||||

   Este desencadenador se activa los días de entre semana cada 15 minutos, a partir de las 7:00 a.m. y hasta las 9:45 p.m. El cuadro **Vista previa** muestra la programación de periodicidad. Para más información, consulte [Programación de tareas y flujos de trabajo](../connectors/connectors-native-recurrence.md) y [Acciones y desencadenadores de flujos de trabajo](../logic-apps/logic-apps-workflow-actions-triggers.md#recurrence-trigger).

1. Para ocultar por ahora los detalles del desencadenador, contraiga la forma haciendo clic dentro de la barra de título de la forma.

   ![Captura de pantalla que muestra la forma de desencadenador contraída.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/collapse-trigger-shape.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

La aplicación lógica ahora está activa en Azure Portal pero no hace nada, aparte de ejecutar el desencadenador basado en la programación especificada. Por lo tanto, agregue una acción que responda cuando se active el desencadenador.

## <a name="get-the-travel-time-for-a-route"></a>Obtención del tiempo de desplazamiento para una ruta

Ahora que ya tiene un desencadenador, agregue una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) que obtenga el tiempo de desplazamiento entre dos lugares. Logic Apps proporciona un conector para la API de Mapas de Bing de forma que pueda obtener fácilmente esta información. Antes de iniciar esta tarea, asegúrese de tener una clave de la API de Mapas de Bing tal como se describe en los requisitos previos de este tutorial.

1. En el diseñador de Logic Apps, seleccione **New step** (Nuevo paso).

1. En **Choose an operation** (Elegir una operación), seleccione **Standar** (Estándar). En el cuadro de búsqueda, escriba `bing maps` y seleccione la acción **Get Route** (Obtener ruta).

   ![Captura de pantalla que muestra la lista "Choose an operation" (Elegir una operación) filtrada por acciones de "Mapas de Bing" y la acción "Get route" (Obtener ruta) seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-get-route-action.png)

1. Si no tiene una conexión de Mapas de Bing, se le solicitará que cree una conexión. Proporcione los detalles de conexión tal y como se describe y, a continuación, seleccione **Create** (Crear).

   ![Captura de pantalla que muestra el cuadro conexión de Mapas de Bing con el nombre de conexión y la clave de API de Mapas de Bing especificados.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/create-maps-connection.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Nombre de la conexión** | Sí | BingMapsConnection | Proporcione un nombre para la conexión. En este ejemplo se usa `BingMapsConnection`. |
   | **Clave de API** | Sí | <*Bing-Maps-API-key*> | Escriba la clave de API de Mapas de Bing recibida previamente. Si no tiene una clave de Mapas de Bing, consulte [Getting a Bing Maps Key](/bingmaps/getting-started/bing-maps-dev-center-help/getting-a-bing-maps-key) (Obtención de una clave de Mapas de Bing). |
   |||||

1. Cambie el nombre de la acción por esta descripción: `Get route and travel time with traffic`.

1. En la acción, abra la lista **Add new parameter** (Agregar nuevo parámetro) y seleccione estas propiedades:

   * **Optimize** (Optimizar)
   * **Distance unit** (Unidad de distancia)
   * **Travel mode** (Modo de desplazamiento)

   ![Captura de pantalla que muestra la acción "Get route..." (Obtener ruta) con las propiedades "Optimize" (Optimizar), "Distance unit" (Unidad de distancia) y "Travel mode" (Modo de desplazamiento) seleccionadas.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-bing-maps-action-properties.png) 

1. Ahora, escriba los valores de las propiedades como se muestra y se describe aquí.

   ![Captura de pantalla que muestra valores de propiedades adicionales para la acción "Get route" (Obtener ruta).](./media/tutorial-build-scheduled-recurring-logic-app-workflow/get-route-action-settings.png) 

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Waypoint 1** (Punto de referencia 1) | Sí | <*ubicación-inicial*> | Origen de la ruta. En este ejemplo se especifica una dirección inicial de ejemplo. |
   | **Waypoint 2** (Punto de referencia 2) | Sí | <*ubicación-final*> | Destino de la ruta. En este ejemplo se especifica una dirección de destino de ejemplo. |
   | **Optimize** (Optimizar) | No | timeWithTraffic | Parámetro para optimizar la ruta, como distancia, tiempo de desplazamiento con tráfico actual, etc. Seleccione el valor del parámetro, **timeWithTraffic**. |
   | **Distance unit** (Unidad de distancia) | No | <*su preferencia*> | Unidad de distancia para la ruta. En este ejemplo se utiliza **Mile** (Milla) como unidad. |
   | **Travel mode** (Modo de desplazamiento) | No | Conducción | Modo de desplazamiento para la ruta. Seleccione el modo de **conducción**. |
   |||||

   Para más información acerca de estos parámetros y valores, consulte [Calcular una ruta](/bingmaps/rest-services/routes/calculate-a-route).

1. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

A continuación, cree una variable para que pueda convertir y almacenar el tiempo de desplazamiento actual en minutos, en lugar de en segundos. De este modo, puede evitar repetir la conversión y utilizar el valor más fácilmente en pasos posteriores. 

## <a name="create-a-variable-to-store-travel-time"></a>Creación de una variable para almacenar el tiempo de desplazamiento

Tal vez desee realizar operaciones en los datos del flujo de trabajo y usar los resultados en acciones posteriores. Para guardar estos resultados, de forma que pueda volver a utilizarlos o hacer referencia a ellos fácilmente, puede crear variables para almacenar los resultados después de procesarlos. Solo puede crear variables en el nivel superior de la aplicación lógica.

De forma predeterminada, la acción **Get route** (Obtener ruta) devuelve el tiempo de desplazamiento actual con el tráfico en segundos mediante la propiedad **Travel Duration Traffic** (Tráfico de duración del desplazamiento). Al convertir y almacenar este valor en minutos, facilita volver a utilizar el valor más adelante sin necesidad de convertirlo de nuevo.

1. En el diseñador, en la acción **Get route** (Obtener ruta), seleccione **New step** (Nuevo paso).

1. En **Choose an operation** (Elegir una operación), seleccione **Built-in** (Integrada). En el cuadro de búsqueda, escriba `variables` y seleccione la acción **Initialize variable** (Inicializar variable).

   ![Captura de pantalla que muestra la acción "Initialize variable" (Inicializar variable) seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-initialize-variable-action.png)

1. Cambie el nombre de esta acción por esta descripción: `Create variable to store travel time`

1. Proporcione esta información para la variable tal y como se muestra en esta tabla y en los pasos posteriores:

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Nombre** | Sí | travelTime | El nombre de la variable. En este ejemplo se usa `travelTime`. |
   | **Tipo** | Sí | Entero | Tipo de datos de la variable |
   | **Valor** | No | Expresión que convierte el tiempo de desplazamiento actual de segundos a minutos (consulte los pasos a continuación de esta tabla). | Valor inicial de la variable |
   |||||

   1. Para crear la expresión para el campo **Valor**, haga clic en el campo para que aparezca la lista de contenido dinámico. Si es necesario, amplíe el explorador hasta que aparezca la lista dinámica. En la lista de contenido dinámico, seleccione **Expression** (Expresión), que muestra el editor de expresiones.

      ![Captura de pantalla que muestra la acción "Initialize variable" (Inicializar variable) con el cursor dentro de la propiedad "Value" (Valor), que abre la lista de contenido dinámico.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings.png)

      La lista de contenido dinámico muestra las salidas de las acciones anteriores que están disponibles para seleccionarse como entradas para las acciones posteriores del flujo de trabajo. La lista de contenido dinámico incluye un editor de expresiones que puede usar para seleccionar las funciones que realizan operaciones en la expresión. Este editor de expresiones solo está disponible en la lista de contenido dinámico.

   1. En el editor de expresiones, escriba esta expresión: `div(,60)`

      ![Captura de pantalla que muestra el editor de expresiones con la expresión "div(,60)" especificada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-2.png)

   1. En la expresión, coloque el cursor entre el paréntesis de apertura ( **(** ) y la coma ( **,** ) y seleccione **Dynamic content** (Contenido dinámico).

      ![Captura de pantalla que muestra dónde colocar el cursor en la expresión "div(,60)" con la opción "Dynamic content" (Contenido dinámico) seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-3.png)

   1. En la lista de contenido dinámico, seleccione el valor de la propiedad **Travel Duration Traffic** (Tráfico de duración del desplazamiento).

      ![Captura de pantalla que muestra el valor de la propiedad "Travel Duration Traffic" (Tráfico de duración del desplazamiento) seleccionado.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-4.png)

   1. Una vez que el valor de la propiedad se resuelva dentro de la expresión, seleccione **Aceptar**.

      ![Captura de pantalla que muestra el botón Aceptar seleccionado.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-5.png)

      El campo **Valor** aparece ahora como se muestra aquí:

      ![Captura de pantalla que muestra la propiedad "Value" (Valor) con la expresión resuelta.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/initialize-variable-action-settings-6.png)

1. Guarde la aplicación lógica.

A continuación, agregue una condición que compruebe si el tiempo de desplazamiento actual es mayor que un límite específico.

## <a name="compare-the-travel-time-with-limit"></a>Comparación del tiempo de desplazamiento con límite

1. En la acción **Create variable to store travel time** (Creación de una variable para almacenar el tiempo de desplazamiento), seleccione **New step** (Nuevo paso).

1. En **Choose an operation** (Elegir una operación), seleccione **Built-in** (Integrada). En el cuadro de búsqueda, escriba `condition`. En la lista de acciones, seleccione la acción **Condition** (Condición).

   ![Captura de pantalla que muestra la acción "Condition" (Condición) seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-condition-action.png)

1. Cambie el nombre de la condición por esta descripción: `If travel time exceeds limit`

1. Cree una condición que compruebe si el valor de la propiedad **travelTime** (tiempo de desplazamiento) supera el límite especificado tal y como se describe aquí:

   1. En el lado izquierdo de la condición, haga clic en el cuadro **Choose a value** (Elegir un valor).

   1. En la lista de contenido dinámico que aparece, en **Variables**, seleccione la propiedad **travelTime**.

      ![Captura de pantalla que muestra el cuadro "Choose a value" (Elegir un valor) en el lado izquierdo de la condición con la lista de contenido dinámico abierta y la propiedad "travelTime" seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/build-condition-left-side.png)

   1. En el cuadro de comparación central, seleccione el operador **is greater than** (es mayor que).

   1. En el lado derecho de la condición, en el cuadro **Choose a value** (Elegir un valor), escriba este límite: `15`.

      Una vez que haya terminado, la condición debe ser parecida a la de este ejemplo:

      ![Captura de pantalla que muestra la condición finalizada para comparar el tiempo de desplazamiento con el límite especificado.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/build-condition-check-travel-time.png)

1. Guarde la aplicación lógica.

A continuación, agregue la acción que desea ejecutar cuando el tiempo de desplazamiento supere el límite.

## <a name="send-email-when-limit-exceeded"></a>Envío de un correo electrónico cuando se supere el límite

Ahora, agregue una acción que le enviará un correo electrónico cuando el tiempo de desplazamiento supere el límite. Este correo electrónico incluye el tiempo de desplazamiento actual y el tiempo adicional necesario para desplazarse por la ruta especificada.

1. En la rama **True** (Verdadero) de la condición, seleccione **Add an action** (Agregar una acción).

1. En **Choose an operation** (Elegir una operación), seleccione **Standard** (Estándar). En el cuadro de búsqueda, escriba `send email`. La lista devuelve muchos resultados, por lo que para filtrarla primero debe seleccionar el conector de correo electrónico que desee.

   Por ejemplo, si tiene una cuenta de correo electrónico de Outlook, seleccione el conector para su tipo de cuenta:

   * Para las cuentas profesionales o educativas de Azure, seleccione **Office 365 Outlook**.
   * Para las cuentas de Microsoft personales, seleccione **Outlook.com**.

   En este ejemplo se selecciona Office 365 Outlook.

   ![Captura de pantalla que muestra la lista "Choose an operation" (Elegir una operación) con la categoría "Standard" (Estándar) y el conector "Office 365 Outlook" seleccionado.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/add-action-send-email.png)

1. Cuando aparezcan las acciones del conector, seleccione la acción que envía correo electrónico, por ejemplo:

   ![Captura de pantalla que muestra la acción "Send an email" (Enviar un correo electrónico) seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-send-email-action.png)

1. Si aún no tiene una conexión, inicie sesión y autentique el acceso a su cuenta de correo electrónico cuando se le solicite.

   Azure Logic Apps crea una conexión a la cuenta de correo electrónico.

1. Cambie el nombre de la acción por esta descripción: `Send email with travel time`

1. En la propiedad **To** (Para), escriba la dirección de correo electrónico del destinatario. Para realizar pruebas, puede usar su dirección de correo electrónico.

1. En el cuadro **Subject** (Asunto), especifique el asunto del correo electrónico e incluya la variable **travelTime** mediante estos pasos:

   1. Escriba el texto `Current travel time (minutes):` con un espacio final. Mantenga el cursor en el cuadro **Subject** (Asunto) para que la lista de contenido dinámico permanezca abierta.

   1. En la lista de contenido dinámico, en el encabezado **Variables**, seleccione **See more** (Ver más) para que aparezca la variable denominada **travelTime**.

      ![Captura de pantalla que muestra la lista de contenido dinámico con la sección "Variables" y la opción "See more" (Ver más) seleccionadas.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/find-travelTime-variable.png)

      > [!NOTE]
      > La lista de contenido dinámico no muestra automáticamente la variable **travelTime** porque la propiedad **Subject** (Asunto) espera un valor de cadena, mientras que **travelTime** es un valor entero.

      ![Captura de pantalla que muestra la lista de contenido dinámico con la variable "travelTime" seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/select-travelTime-variable.png)

1. Para la propiedad **Body** (Cuerpo), especifique el contenido del cuerpo del correo electrónico siguiendo estos pasos:

   1. Escriba el texto `Add extra travel time (minutes):` con un espacio final. Mantenga el cursor en el cuadro **Body** (Cuerpo) para que la lista de contenido dinámico permanezca abierta.

   1. En la lista de contenido dinámico, seleccione **Expression** (Expresión), que muestra el editor de expresiones.

      ![Captura de pantalla que muestra la lista de contenido dinámico con la opción "Expression" (Expresión) seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings.png)

   1. En el editor de expresiones, escriba `sub(,15)` para que pueda calcular el número de minutos que superen el límite: 

      ![Captura de pantalla que muestra el editor de expresiones con la expresión "sub(,15)" especificada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-2.png)

   1. En la expresión, coloque el cursor entre el paréntesis de apertura ( **(** ) y la coma ( **,** ) y seleccione **Dynamic content** (Contenido dinámico).

      ![Captura de pantalla que muestra dónde colocar el cursor en la expresión "sub(,15)" con la opción "Dynamic content" (Contenido dinámico) seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-3.png)

   1. En **Variables**, seleccione **travelTime**.

      ![Captura de pantalla que muestra la lista de contenido dinámico con la variable "travelTime" seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-4.png)

   1. Una vez que la propiedad se resuelva dentro de la expresión, seleccione **Aceptar**.

      ![Captura de pantalla que muestra la lista de contenido dinámico con la opción Aceptar seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-5.png)

      La propiedad **Body** (Cuerpo) aparece ahora como se muestra aquí:

      ![Captura de pantalla que muestra la lista de contenido dinámico con la expresión resuelta en la propiedad "Body" (Cuerpo) de la acción de correo electrónico.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/send-email-body-settings-6.png)

1. Guarde la aplicación lógica.

A continuación, pruebe y ejecute la aplicación lógica, que ahora es similar a este ejemplo:

![Captura de pantalla que muestra el flujo de trabajo de aplicación lógica de ejemplo finalizado.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/check-travel-time-finished.png)

## <a name="run-your-logic-app"></a>Ejecución de la aplicación lógica

Para iniciar manualmente la aplicación lógica, seleccione **Ejecutar** en la barra de herramientas del diseñador.

* Si el tiempo de desplazamiento actual permanece en el límite, la aplicación lógica no hace nada más y espera al siguiente intervalo antes de volver a comprobar. 

* Pero, si el tiempo de desplazamiento actual supera el límite, recibirá un correo electrónico con el tiempo de desplazamiento actual y el número de minutos por encima del límite. Este es un correo electrónico de ejemplo que la aplicación lógica envía:

  ![Captura de pantalla que muestra un correo electrónico de ejemplo que informa del tiempo de desplazamiento actual y del tiempo de desplazamiento adicional que supera el límite especificado.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/received-example-email-notification.png)

  > [!TIP]
  > Si no recibe ningún correo electrónico, compruebe la carpeta de correo electrónico no deseado. El filtro de correo electrónico no deseado podría redirigir esta clase de correo. Si tampoco aparece allí y no está seguro de que la aplicación lógica se ejecutara correctamente, consulte el artículo de [solución de problemas en la aplicación lógica](../logic-apps/logic-apps-diagnosing-failures.md).

Enhorabuena, acaba de crear y ejecutar una aplicación lógica periódica basada en programación. 

Para crear otras aplicaciones lógicas que usen el desencadenador **Periodicidad**, consulte estas plantillas, que están disponibles tras crear una aplicación lógica:

* Enviarle recordatorios diarios.
* Eliminar blobs de Azure anteriores.
* Agregar mensaje a una cola de Azure Storage.

## <a name="clean-up-resources"></a>Limpieza de recursos

La aplicación lógica continúa ejecutándose hasta que la deshabilite o elimine la aplicación. Cuando ya no necesite el ejemplo de aplicación lógica, elimine el grupo de recursos que contiene la aplicación lógica y los recursos relacionados.

1. En el cuadro de búsqueda de Azure Portal, escriba el nombre del grupo de recursos que creó. En los resultados, en **Grupos de recursos**, seleccione el grupo de recursos.

   En este ejemplo se ha creado un grupo de recursos llamado `LA-TravelTime-RG`.

   ![Captura de pantalla que muestra el cuadro de búsqueda de Azure con "la-travel-time-rg" escrito y **LA-TravelTime-RG** seleccionado.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/find-resource-group.png)

   > [!TIP]
   > Si la página principal de Azure muestra el grupo de recursos en **Recursos recientes**, puede seleccionar el grupo desde la página principal.

1. En el menú del grupo de recursos, compruebe que se ha seleccionado **Información general**. En la barra de herramientas del panel **Información general**, elija **Eliminar grupo de recursos**.

   ![Captura de pantalla que muestra el panel "Información general" del grupo de recursos y, en la barra de herramientas del panel, la opción "Eliminar grupo de recursos" está seleccionada.](./media/tutorial-build-scheduled-recurring-logic-app-workflow/delete-resource-group.png)

1. En el panel de confirmación que aparece, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha creado una aplicación de lógica que comprueba el tráfico según una programación definida (por las mañanas los días de entre semana) y realiza alguna acción (envío de correo electrónico) cuando el tiempo de desplazamiento supera un límite especificado. Ahora, aprenda a crear una aplicación lógica que envía solicitudes de lista de correo para su aprobación mediante la integración de servicios de Azure, servicios de Microsoft y otras aplicaciones de software como servicio (SaaS).

> [!div class="nextstepaction"]
> [Administración de solicitudes de lista de distribución de correo](../logic-apps/tutorial-process-mailing-list-subscriptions-workflow.md)
