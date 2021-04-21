---
title: Creación de flujos de trabajo automatizados basados en aprobación
description: 'Tutorial: Creación de un flujo de trabajo automatizado basado en aprobación que procesa las suscripciones de listas de distribución de correo mediante Azure Logic Apps'
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: tutorial
ms.custom: mvc
ms.date: 03/24/2021
ms.openlocfilehash: c6186e6e9f60e852f77943834bcd1ae3d526491d
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107777282"
---
# <a name="tutorial-create-automated-approval-based-workflows-by-using-azure-logic-apps"></a>Tutorial: Creación de flujos de trabajo automatizados basados en aprobación mediante Azure Logic Apps

En este tutorial se muestra cómo compilar una [aplicación lógica](../logic-apps/logic-apps-overview.md) de ejemplo que automatiza un flujo de trabajo basado en aprobación. En concreto, esta aplicación lógica de ejemplo procesa las solicitudes de suscripción para obtener una lista de distribución de correo que administra el servicio [MailChimp](https://mailchimp.com/). Esta aplicación lógica incluye varios pasos, que comienzan con la supervisión de una cuenta de correo electrónico para detectar las solicitudes, envía estas solicitudes para su aprobación, comprueba si la solicitud se aprueba o no, agrega miembros aprobados a la lista de distribución de correo y confirma si los nuevos miembros se agregan a la lista.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear una aplicación lógica en blanco.
> * Agregue un desencadenador que supervise las solicitudes de suscripción en los correos electrónicos.
> * Agregue una acción que envíe correos electrónicos para aprobar o rechazar estas solicitudes.
> * Agregue una condición que compruebe la respuesta de aprobación.
> * Agregue una acción que permita incorporar los miembros aprobados a la lista de distribución de correo.
> * Agregue una condición que compruebe si estos miembros se han unido correctamente a la lista.
> * Agregue una condición que envíe correos electrónicos que confirmen si estos miembros se han unido correctamente a la lista.

Cuando haya terminado, la aplicación lógica se parecerá a este flujo de trabajo, en un alto nivel:

![Información general de la aplicación lógica de alto nivel finalizada](./media/tutorial-process-mailing-list-subscriptions-workflow/tutorial-high-level-overview.png)

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta y una suscripción de Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Una cuenta de MailChimp en la que haya creado previamente una lista denominada "test-members-ML" en la que la aplicación lógica pueda agregar direcciones de correo electrónico para los miembros aprobados. Si no tiene una cuenta, [regístrese para obtener una cuenta gratuita](https://login.mailchimp.com/signup/) y aprenda a [crear una lista de MailChimp](https://us17.admin.mailchimp.com/lists/#).

* Una cuenta de correo electrónico con un proveedor de correo electrónico compatible con Logic Apps, como Office 365 Outlook, Outlook.com o Gmail. En el caso de otros proveedores, [consulte la lista de conectores que se muestra aquí](/connectors/). Este inicio rápido utiliza Office 365 Outlook con una cuenta profesional o educativa. Si utiliza una cuenta de correo electrónico diferente, los pasos generales siguen siendo los mismos, pero la interfaz de usuario podría ser ligeramente distinta.

* Una cuenta de correo electrónico de Office 365 Outlook o Outlook.com, que admita flujos de trabajo de aprobación. Este tutorial usa Office 365 Outlook. Si utiliza una cuenta de correo electrónico diferente, los pasos generales siguen siendo los mismos pero la interfaz de usuario podría ser ligeramente distinta.

* Si la aplicación lógica necesita comunicarse a través de un firewall que limite el tráfico a direcciones IP específicas, ese firewall debe permitir el acceso para *ambas* direcciones IP, la [de entrada](logic-apps-limits-and-config.md#inbound) y la [de salida](logic-apps-limits-and-config.md#outbound), utilizadas por el servicio Logic Apps o por el entorno de ejecución en la región de Azure donde se encuentra la aplicación lógica. Si la aplicación lógica también usa [conectores administrados](../connectors/managed.md), como el conector de Outlook de Office 365 o el conector de SQL, o emplea [conectores personalizados](/connectors/custom-connectors/), el firewall también debe permitir el acceso a *todas* las [direcciones IP de salida del conector administrado](logic-apps-limits-and-config.md#outbound) en la región de Azure de la aplicación lógica.

## <a name="create-your-logic-app"></a>Creación de una aplicación lógica

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure. En la página principal de Azure, seleccione **Crear un recurso**.

1. En el menú de Azure Marketplace, seleccione **Integración** > **Logic App**.

   ![Captura de pantalla que muestra el menú de Azure Marketplace con las opciones "Integración" y "Logic App" seleccionadas.](./media/tutorial-process-mailing-list-subscriptions-workflow/create-new-logic-app-resource.png)

1. En el panel de **Logic App**, proporcione la información que se describe aquí sobre la aplicación lógica que desea crear.

   ![Captura de pantalla que muestra el panel de creación de aplicaciones lógicas y la información que se debe proporcionar para la nueva aplicación lógica.](./media/tutorial-process-mailing-list-subscriptions-workflow/create-logic-app-settings.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Suscripción** | <*Azure-subscription-name*> | El nombre de la suscripción de Azure. En este ejemplo se usa `Pay-As-You-Go`. |
   | **Grupos de recursos** | LA-MailingList-RG | El nombre del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) que se utiliza para organizar recursos relacionados. En este ejemplo se crea un nuevo grupo de recursos llamado `LA-MailingList-RG`. |
   | **Nombre** | LA-MailingList | El nombre de la aplicación lógica, que solo puede contener letras, números, guiones (`-`), caracteres de subrayado (`_`), paréntesis (`(`, `)`) y puntos (`.`). En este ejemplo se usa `LA-MailingList`. |
   | **Ubicación** | Oeste de EE. UU. | La región en la que desea almacenar la información de la aplicación lógica. En este ejemplo se usa `West US`. |
   | **Log Analytics** | Off | Mantenga el valor **Off** para el registro de diagnóstico. |
   ||||

1. Seleccione **Revisar y crear** cuando haya terminado. Una vez que Azure valide la información sobre la aplicación lógica, seleccione **Crear**.

1. Una vez que Azure implemente la aplicación, seleccione **Ir al recurso**.

   Azure abre el panel de selección de plantillas de aplicaciones lógicas, que muestra un vídeo de introducción, los desencadenadores usados frecuentemente y los patrones de plantillas de aplicaciones lógicas.

1. Desplácese hacia abajo más allá del vídeo y de los desencadenadores frecuentes hasta la sección **Plantillas** y seleccione **Aplicación lógica en blanco**.

   ![Captura de pantalla que muestra el panel de selección de plantilla de aplicaciones lógicas con la opción "Aplicación lógica en blanco" seleccionada.](./media/tutorial-process-mailing-list-subscriptions-workflow/select-logic-app-template.png)

A continuación, agregue un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) de Outlook que escuche los mensajes de correo electrónico entrantes que tienen solicitudes de suscripción. Cada aplicación lógica debe comenzar con un desencadenador, que se activa cuando sucede un evento específico o cuando hay nuevos datos que cumplen una condición determinada. Para más información, consulte [Creación de una nueva aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="add-trigger-to-monitor-emails"></a>Agregar desencadenador para supervisar los correos electrónicos

1. En el cuadro de búsqueda del diseñador de Logic Apps, escriba `when email arrives` y seleccione el desencadenador denominado **Cuando llega un nuevo correo electrónico**.

   * Para las cuentas profesionales o educativas de Azure, seleccione **Office 365 Outlook**.
   * Para las cuentas de Microsoft personales, seleccione **Outlook.com**.

   En este ejemplo se selecciona Office 365 Outlook.

   ![Captura de pantalla que muestra el cuadro de búsqueda del diseñador de Logic Apps que contiene el término de búsqueda "cuando llega un correo electrónico" y aparece seleccionado el desencadenador "Cuando llega un nuevo correo electrónico".](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-new-email.png)

1. Si aún no tiene una conexión, inicie sesión y autentique el acceso a su cuenta de correo electrónico cuando se le solicite.

   Azure Logic Apps crea una conexión a la cuenta de correo electrónico.

1. En el desencadenador, especifique los criterios para comprobar si hay correo electrónico nuevo.

   1. Especifique la carpeta para comprobar los correos electrónicos y mantenga las demás propiedades establecidas en sus valores predeterminados.

      ![Captura de pantalla que muestra el diseñador con la acción "Cuando llega un nuevo correo electrónico" y la opción "Carpeta" establecida en "Bandeja de entrada".](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-set-up-email.png)

   1. Agregue la propiedad **Filtro de asunto** del desencadenador para poder filtrar los mensajes de correo electrónico en función de la línea de asunto. Abra la lista **Agregar nuevo parámetro** y seleccione **Filtro de asunto**.

      ![Captura de pantalla que muestra la lista "Agregar nuevo parámetro" abierta con la opción "Filtro de asunto" seleccionada.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-add-properties.png)

      Para más información acerca de las propiedades de este desencadenador, consulte la [referencia del conector de Office 365 Outlook](/connectors/office365/) o la [referencia del conector de Outlook.com](/connectors/outlook/).

   1. Después de que la propiedad aparezca en el desencadenador, escriba este texto: `subscribe-test-members-ML`

      ![Captura de pantalla que muestra la propiedad "Filtro de asunto" con el texto "subscribe-test-members-ML" escrito.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-subject-filter-property.png)

1. Para ocultar por ahora los detalles del desencadenador, contraiga la forma haciendo clic dentro de la barra de título de la forma.

   ![Captura de pantalla que muestra la forma de desencadenador contraída.](./media/tutorial-process-mailing-list-subscriptions-workflow/collapse-trigger-shape.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

La aplicación lógica estará activa, pero no hace más que comprobar el correo electrónico de entrada. Por lo tanto, agregue una acción que responda cuando se active el desencadenador.

## <a name="send-approval-email"></a>Enviar correo electrónico de aprobación

Ahora que tiene un desencadenador, agregue una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) que envíe un correo electrónico de aprobación o rechazo de la solicitud.

1. En el diseñador de Logic Apps, en el desencadenador **Cuando llega un nuevo correo electrónico**, seleccione **Nuevo paso**.

1. En **Choose an operation** (Elegir una operación), en el cuadro de búsqueda, escriba `send approval` y seleccione la acción **Send approval email** (Enviar correo electrónico de aprobación).

   ![Captura de pantalla que muestra la lista "Choose an operation" (Elegir una operación) filtrada por acciones de "aprobación" y la acción "Send approval email" (Enviar correo electrónico de aprobación) seleccionada.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-send-approval-email.png)

1. Ahora, escriba los valores de las propiedades especificadas como se muestra y se describe aquí. Deje las demás opciones con sus valores predeterminados. Para más información acerca de estas propiedades, consulte la [referencia del conector de Office 365 Outlook](/connectors/office365/) o la [referencia del conector de Outlook.com](/connectors/outlook/).

   ![Captura de pantalla que muestra las propiedades de "Send approval email" (Enviar correo electrónico de aprobación).](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-approval-email-settings.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **To** | <*approval-email-address*> | Dirección de correo electrónico del aprobador. Para realizar pruebas, puede usar su propia dirección de correo electrónico. En este ejemplo se usa la dirección de correo electrónico ficticia `sophiaowen@fabrikam.com`. |
   | **Subject** | `Approve member request for test-members-ML` | Un asunto de correo electrónico descriptivo |
   | **Opciones de usuario** | `Approve, Reject` | Asegúrese de que esta propiedad especifica las opciones de respuesta que puede seleccionar el aprobador, que son **Aprobar** o **Rechazar** de forma predeterminada. |
   ||||

   > [!NOTE]
   > Al hacer clic en algunos cuadros de edición, aparece la lista de contenido dinámico que, por el momento, puede omitir. Esta lista muestra las salidas de las acciones anteriores que están disponibles para seleccionarse como entradas para las acciones posteriores del flujo de trabajo.
 
1. Guarde la aplicación lógica.

A continuación, agregue una condición para comprobar la respuesta seleccionada del aprobador.

## <a name="check-approval-response"></a>Comprobación de la respuesta de aprobación

1. En la acción **Send approval email** (Enviar correo electrónico de aprobación), seleccione **Nuevo paso**.

1. En **Choose an operation** (Elegir una operación), seleccione **Built-in** (Integrada). En el cuadro de búsqueda, escriba `condition` y seleccione la acción **Condición**.

   ![Captura de pantalla que muestra el cuadro de búsqueda "Choose an operation" (Elegir una operación) con la opción "Built-in" (Integrada) seleccionada y "Condición" como término de búsqueda, mientras que la acción "Condición" aparece seleccionada.](./media/tutorial-process-mailing-list-subscriptions-workflow/select-condition-action.png)

1. En la barra de título de **Condición**, seleccione el botón de **puntos suspensivos** ( **...** ) y, a continuación, seleccione **Cambiar nombre**. Cambie el nombre de la condición por esta descripción: `If request approved`

   ![Captura de pantalla que muestra el botón de puntos suspensivos seleccionado, la lista "Configuración" abierta y el comando "Cambiar nombre" seleccionado.](./media/tutorial-process-mailing-list-subscriptions-workflow/rename-condition-description.png)

1. Cree una condición que compruebe si el aprobador ha seleccionado **Aprobar**.

   1. En el lado izquierdo de la condición, haga clic en el cuadro **Elegir un valor**.

   1. En la lista de contenido dinámico que aparece, en **Enviar correo electrónico de aprobación**, seleccione la propiedad **SelectedOption**.

      ![Captura de pantalla que muestra la lista de contenido dinámico, con la sección ""Send approval email" (Enviar correo electrónico de aprobación), donde aparece seleccionada la salida "SelectedOption".](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-approval-response.png)

   1. En el cuadro de comparación central, seleccione el operador **es igual a**.

   1. En el lado derecho de la condición, en el cuadro **Elegir un valor**, escriba este texto: `Approve`.

      Una vez que haya terminado, la condición debe ser parecida a la de este ejemplo:

      ![Captura de pantalla que muestra la condición finalizada para el ejemplo de solicitud aprobada.](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-approval-response-2.png)

1. Guarde la aplicación lógica.

A continuación, especifique la acción que realizará la aplicación lógica cuando el revisor apruebe la solicitud. 

## <a name="add-member-to-mailchimp-list"></a>Adición de miembros a la lista MailChimp

Ahora, agregue una acción que incorpore al miembro aprobado a la lista de distribución de correo.

1. En la rama **True** (Verdadero) de la condición, seleccione **Add an action** (Agregar una acción).

1. En el cuadro de búsqueda **Choose an operation** (Elegir una operación), escriba **Todo**. En el cuadro de búsqueda, escriba `mailchimp` y seleccione la acción **Agregar un miembro a una lista**.

   ![Captura de pantalla que muestra el cuadro "Choose an operation" (Elegir una operación) con el término de búsqueda "mailchimp" y la acción "Agregar un miembro a una lista" seleccionada.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-mailchimp-add-member.png)

1. Si todavía no tiene ninguna conexión a la cuenta de MailChimp, se le pedirá que inicie sesión.

1. En la acción **Agregar un miembro a una lista**, proporcione la información que se muestra y se describe aquí:

   ![Captura de pantalla que muestra la información de la acción "Agregar un miembro a una lista".](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-mailchimp-add-member-settings.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Identificador de la lista** | Sí | <*mailing-list-name*> | Seleccione el nombre de la lista de distribución de correo electrónico de MailChimp. En este ejemplo se usa `test-members-ML`. |
   | **Dirección de correo electrónico** | Sí | <*new-member-email-address*> | En la lista de contenido dinámico que se abre, en la sección **Cuando llega un nuevo correo electrónico**, seleccione **De**, que es la salida del desencadenador y especifica la dirección de correo electrónico del nuevo miembro. |
   | **Estado** | Sí | <*member-subscription-status*> | Seleccione el estado de la suscripción que establecer para el nuevo miembro. En este ejemplo se selecciona `subscribed` (Flujo de trabajo con estado). <p>Para más información, consulte [Manage subscribers with the MailChimp API](https://developer.mailchimp.com/documentation/mailchimp/guides/manage-subscribers-with-the-mailchimp-api/) (Administración de suscriptores con MailChimp API). |
   |||||

   Para más información sobre las propiedades de la acción **Agregar un miembro a una lista**, consulte la [referencia del conector MailChimp](/connectors/mailchimp/).

1. Guarde la aplicación lógica.

A continuación, agregue una condición para comprobar si el nuevo miembro se ha unido correctamente a la lista de distribución de correo electrónico. De este modo, la aplicación lógica podrá notificarle si esta operación se ha realizado correctamente o si se ha producido un error.

## <a name="check-for-success-or-failure"></a>Comprobación de si la operación se ha realizado correctamente o se ha producido un error

1. En la rama **True**, en la acción **Agregar un miembro a una lista**, seleccione **Agregar una acción**.

1. En **Choose an operation** (Elegir una operación), seleccione **Built-in** (Integrada). En el cuadro de búsqueda, escriba `condition` y seleccione la acción **Condición**.

1. Cambie el nombre de la condición por esta descripción: `If add member succeeded`

1. Cree una condición que compruebe si la unión del miembro aprobado a la lista de distribución de correo electrónico se ha realizado correctamente o se ha producido un error:

   1. En el lado izquierdo de la condición, haga clic en el cuadro **Elegir un valor**. En la lista de contenido dinámico que aparece, en **Agregar un miembro a una lista** seleccione la propiedad **Status** (Estado).

      Por ejemplo, la condición es similar a la de este ejemplo:

      ![Captura de pantalla que muestra el cuadro "Elegir un valor" de la parte izquierda de la condición, con "Status" (Estado) especificado.](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-added-member.png)

   1. En el cuadro de comparación central, seleccione el operador **es igual a**.

   1. En el lado derecho de la condición, en el cuadro **Elegir un valor**, escriba este texto: `subscribed`.

      Una vez que haya terminado, la condición debe ser parecida a la de este ejemplo:

      ![Captura de pantalla que muestra la condición finalizada para comprobar si la suscripción se ha realizado correctamente o si se ha producido un error.](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-added-member-2.png)

A continuación, configure los correos electrónicos que se enviarán si la unión del miembro aprobado a la lista de distribución de correo electrónico se ha realizado correctamente o si se ha producido un error.

## <a name="send-email-if-member-added"></a>Envío de correo electrónico si se ha agregado el miembro

1. En la condición **If add member succeeded** (Si la incorporación del miembro se ha realizado correctamente), en la rama **True**, seleccione **Agregar una acción**.

   ![Captura de pantalla que muestra la rama "True" de la condición "If add member succeeded" (Si la incorporación del miembro se ha realizado correctamente) con la opción "Agregar una acción" seleccionada.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-success.png)

1. En el cuadro de búsqueda **Choose an operation** (Elegir una operación), escriba `outlook send email` y seleccione la acción **Send an email** (Enviar un correo electrónico).

   ![Captura de pantalla que muestra el cuadro de búsqueda "Choose an operation" (Elegir una operación)" con la opción "outlook send email" (Outlook envía correos electrónicos) especificada y la acción "Send an email" (Enviar un correo electrónico) seleccionada para la notificación.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-success-2.png)

1. Cambie el nombre de la acción por esta descripción: `Send email on success`

1. En la acción **Send email on success** (Enviar correo electrónico si la operación es correcta), proporcione la información que se muestra y se describe aquí:

   ![Captura de pantalla que muestra la acción "Send email on success" (Enviar correo electrónico si la operación es correcta) y la información proporcionada para el correo electrónico que indica que la operación se ha realizado correctamente.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-success-settings.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Cuerpo** | Sí | <*success-email-body*> | El contenido del cuerpo del correo electrónico de confirmación de unión correcta. Para este tutorial, siga estos pasos: <p>1. Escriba este texto con un espacio final: `New member has joined "test-members-ML":`. <p>2. En la lista de contenido dinámico que aparece, seleccione la propiedad **Email Address** (Dirección de correo electrónico). <p>**Nota**: Si esta propiedad no aparece, junto al encabezado de sección **Agregar un miembro a una lista**, seleccione **Ver más**. <p>3. En la siguiente fila, escriba este texto con un espacio final: `Member opt-in status: `. <p>4. En la lista de contenido dinámico, en **Agregar un miembro a una lista** seleccione la propiedad **Status** (Estado). |
   | **Subject** | Sí | <*success-email-subject*> | El asunto del correo electrónico de confirmación de que la unión se ha realizado correctamente. Para este tutorial, siga estos pasos: <p>1. Escriba este texto con un espacio final: `Success! Member added to "test-members-ML": `. <p>2. En la lista de contenido dinámico, en **Agregar un miembro a una lista** seleccione la propiedad **Email Address** (Dirección de correo electrónico). |
   | **To** | Sí | <*your-email-address*> | La dirección de correo electrónico a la que enviar el correo electrónico de confirmación de que la unión se ha realizado correctamente. Para realizar pruebas, puede usar su propia dirección de correo electrónico. |
   |||||

1. Guarde la aplicación lógica.

## <a name="send-email-if-member-not-added"></a>Envío de correo electrónico si el miembro no se ha agregado

1. En la condición **If add member succeeded** (Si la incorporación del miembro se ha realizado correctamente), en la rama **False**, seleccione **Agregar una acción**.

   ![Captura de pantalla que muestra la rama "False" de la condición "If add member succeeded" (Si la incorporación del miembro se ha realizado correctamente) con la opción "Agregar una acción" seleccionada.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-failed.png)

1. En el cuadro de búsqueda **Choose an operation** (Elegir una operación), escriba `outlook send email` y seleccione la acción **Send an email** (Enviar un correo electrónico).

   ![Captura de pantalla que muestra el cuadro de búsqueda "Choose an operation" (Elegir una operación)" con la opción "outlook send email" (Outlook envía correos electrónicos) especificada y la acción "Send an email" (Enviar un correo electrónico) seleccionada.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-failed-2.png)

1. Cambie el nombre de la acción por esta descripción: `Send email on failure`

1. Especifique la información acerca de esta acción como se indica aquí:

   ![Captura de pantalla que muestra la acción "Send email on failure" (Enviar correo electrónico si la operación no es correcta) y la información proporcionada para el correo electrónico que indica que la operación no se ha realizado correctamente.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-failed-settings.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Cuerpo** | Sí | <*body-for-failure-email*> | El contenido del cuerpo del correo electrónico de error. Para este tutorial, escriba este texto: <p>`Member might already exist. Check your MailChimp account.` |
   | **Subject** | Sí | <*subject-for-failure-email*> | El asunto del correo electrónico de error. Para este tutorial, siga estos pasos: <p>1. Escriba este texto con un espacio final: `Failed, member not added to "test-members-ML": `. <p>2. En la lista de contenido dinámico, en **Agregar un miembro a una lista** seleccione la propiedad **Email Address** (Dirección de correo electrónico). |
   | **To** | Sí | <*your-email-address*> | La dirección de correo electrónico a la que enviar el correo electrónico de error. Para realizar pruebas, puede usar su propia dirección de correo electrónico. |
   |||||

1. Guarde la aplicación lógica. 

A continuación, pruebe la aplicación lógica, que ahora es similar a este ejemplo:

![Captura de pantalla que muestra el flujo de trabajo de aplicación lógica de ejemplo finalizado.](./media/tutorial-process-mailing-list-subscriptions-workflow/tutorial-high-level-completed.png)

## <a name="run-your-logic-app"></a>Ejecución de la aplicación lógica

1. Envíese a sí mismo una solicitud de correo electrónico para unirse a la lista de distribución de correo electrónico. Espere a que la solicitud aparezca en la bandeja de entrada.

1. Para iniciar manualmente la aplicación lógica, seleccione **Ejecutar** en la barra de herramientas del diseñador. 

   Si el correo electrónico tiene un asunto que coincide con el del filtro de asunto del desencadenador, la aplicación lógica le enviará un correo electrónico para que apruebe la solicitud de suscripción.

1. En el correo electrónico de aprobación que reciba, seleccione **Aprobar**.

1. Si la dirección de correo electrónico del suscriptor no existe en la lista de distribución correspondiente, la aplicación lógica agregará la dirección de esta persona y enviará un correo electrónico parecido al de este ejemplo:

   ![Captura de pantalla que muestra el correo electrónico de ejemplo para una suscripción correcta.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-member-mailing-list-success.png)

   Si la aplicación lógica no puede agregar al suscriptor, recibirá un correo electrónico parecido al de este ejemplo:

   ![Captura de pantalla que muestra el correo electrónico de ejemplo para una suscripción que no se ha realizado correctamente.](./media/tutorial-process-mailing-list-subscriptions-workflow/add-member-mailing-list-failed.png)

  > [!TIP]
  > Si no recibe ningún correo electrónico, compruebe la carpeta de correo electrónico no deseado. El filtro de correo electrónico no deseado podría redirigir esta clase de correo. Si tampoco aparece allí y no está seguro de que la aplicación lógica se ejecutara correctamente, consulte el artículo de [solución de problemas en la aplicación lógica](../logic-apps/logic-apps-diagnosing-failures.md).

Enhorabuena, acaba de crear y ejecutar una aplicación lógica que integra la información de Azure, los servicios de Microsoft y otras aplicaciones de SaaS.

## <a name="clean-up-resources"></a>Limpieza de recursos

La aplicación lógica continúa ejecutándose hasta que la deshabilite o elimine la aplicación. Cuando ya no necesite el ejemplo de aplicación lógica, elimine el grupo de recursos que contiene la aplicación lógica y los recursos relacionados.

1. En el cuadro de búsqueda de Azure Portal, escriba el nombre del grupo de recursos que creó. En los resultados, en **Grupos de recursos**, seleccione el grupo de recursos.

   En este ejemplo se ha creado un grupo de recursos llamado `LA-MailingList-RG`.

   ![Captura de pantalla que muestra el cuadro de búsqueda de Azure con "la-mailinglist-rg" escrito y **LA-MailingList-RG** seleccionado.](./media/tutorial-process-mailing-list-subscriptions-workflow/find-resource-group.png)

   > [!TIP]
   > Si la página principal de Azure muestra el grupo de recursos en **Recursos recientes**, puede seleccionar el grupo desde la página principal.

1. En el menú del grupo de recursos, compruebe que se ha seleccionado **Información general**. En la barra de herramientas del panel **Información general**, elija **Eliminar grupo de recursos**.

   ![Captura de pantalla que muestra el panel "Información general" del grupo de recursos y, en la barra de herramientas del panel, la opción "Eliminar grupo de recursos" está seleccionada.](./media/tutorial-process-mailing-list-subscriptions-workflow/delete-resource-group.png)

1. En el panel de confirmación que aparece, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una aplicación lógica que administra las aprobaciones de las solicitudes de la lista de distribución de correo electrónico. Ahora, aprenda a crear una aplicación lógica que procesa y almacena datos adjuntos de correo electrónico mediante la integración de servicios de Azure, como Azure Storage y Azure Functions.

> [!div class="nextstepaction"]
> [Procesamiento de archivos adjuntos de correo electrónico](../logic-apps/tutorial-process-email-attachments-workflow.md)
