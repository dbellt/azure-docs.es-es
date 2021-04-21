---
title: 'Inicio rápido: Creación del primer flujo de trabajo de Logic Apps en Azure Portal'
description: Cree su primer flujo de trabajo automatizado de Logic Apps en Azure Portal mediante esta guía de inicio rápido. Conozca los aspectos básicos de la integración de sistemas y las soluciones de Enterprise Application Integration (EAI) en Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: quickstart
ms.custom: mvc
ms.date: 03/24/2021
ms.openlocfilehash: d05566c0734f95e14335c6165de0b2104fa19bc6
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107764862"
---
# <a name="quickstart-create-your-first-logic-apps-workflow---azure-portal"></a>Inicio rápido: Creación del primer flujo de trabajo de Logic Apps en Azure Portal

En esta guía de inicio rápido se explica cómo crear su primer flujo de trabajo en [Azure Logic Apps](logic-apps-overview.md) mediante [Azure Portal](https://portal.azure.com). En esta guía introductoria también se explican los conceptos básicos del servicio Logic Apps, incluido cómo crear una nueva aplicación lógica, agregar un desencadenador y una acción a la aplicación lógica y probar la aplicación lógica. Siga esta guía de inicio rápido para crear una aplicación lógica de ejemplo que comprueba regularmente una fuente RSS y envía una notificación por correo electrónico para los nuevos elementos. En la captura de pantalla siguiente se muestra el flujo de trabajo general de esta aplicación lógica de ejemplo:

![Captura de pantalla del diseñador de Logic Apps, que muestra la aplicación lógica de ejemplo en la que el desencadenador es una fuente RSS y la acción es enviar un correo electrónico.](./media/quickstart-create-first-logic-app-workflow/quickstart-workflow-overview.png)

<a name="prerequisites"></a>

Si desea obtener información sobre cómo crear y administrar la primera aplicación lógica mediante otras interfaces y aplicaciones, consulte estas otras guías de inicio rápido de Logic Apps: 

* [Inicio rápido: Creación y administración de aplicaciones lógicas mediante la CLI de Azure](quickstart-logic-apps-azure-cli.md)
* [Inicio rápido: Creación y administración de definiciones de flujo de trabajo de aplicaciones lógicas mediante Visual Studio Code](quickstart-create-logic-apps-visual-studio-code.md)
* [Inicio rápido: Creación y automatización de tareas, procesos y flujos de trabajo con Azure Logic Apps en Visual Studio](quickstart-create-logic-apps-with-visual-studio.md)

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Una cuenta de correo electrónico de un servicio admitido por Logic Apps (como Outlook de Office 365 o Outlook.com). Para otros proveedores de correo electrónico admitidos, [consulte la lista de conectores](/connectors/).

    > [!IMPORTANT]
    > Si va a usar el [conector de Gmail](/connectors/gmail/), tenga en cuenta que solo las cuentas de G Suite pueden usarlo sin restricciones en Logic Apps. Si tiene una cuenta de consumidor de Gmail, puede usar este conector solo con servicios específicos aprobados por Google, a menos que pueda [crear una aplicación cliente de Google para usarla en la autenticación con el conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Si la aplicación lógica necesita comunicarse a través de un firewall que limite el tráfico a direcciones IP específicas, ese firewall debe permitir el acceso para *ambas* direcciones IP, la [de entrada](logic-apps-limits-and-config.md#inbound) y la [de salida](logic-apps-limits-and-config.md#outbound), utilizadas por el servicio Logic Apps o por el entorno de ejecución en la región de Azure donde se encuentra la aplicación lógica. Si la aplicación lógica también usa [conectores administrados](../connectors/managed.md), como el conector de Outlook de Office 365 o el conector de SQL, o emplea [conectores personalizados](/connectors/custom-connectors/), el firewall también debe permitir el acceso a *todas* las [direcciones IP de salida del conector administrado](logic-apps-limits-and-config.md#outbound) en la región de Azure de la aplicación lógica.

<a name="create-logic-app"></a>

## <a name="create-your-logic-app"></a>Creación de una aplicación lógica

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure.

1. En el cuadro de búsqueda de Azure Portal, escriba `logic apps` y seleccione **Logic Apps**.

   ![Captura de pantalla que muestra el cuadro de búsqueda de Azure Portal con "logic apps" como término de la búsqueda y "Logic Apps" como el resultado de la búsqueda seleccionado.](./media/quickstart-create-first-logic-app-workflow/find-select-logic-apps.png)

1. En la página **Logic Apps**, seleccione en **Agregar**.

   ![Captura de pantalla de la página del servicio Logic Apps en Azure Portal, que muestra la lista de aplicaciones lógicas y el botón seleccionado, "Agregar".](./media/quickstart-create-first-logic-app-workflow/add-new-logic-app.png)

1. En el panel **Aplicación lógica**, proporcione los detalles básicos y la configuración de la aplicación lógica. Cree un nuevo [grupo de recursos](../azure-resource-manager/management/overview.md#terminology) para los fines de esta aplicación lógica de ejemplo.

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Nombre** | <*nombre-de-la-aplicación-lógica*> | Nombre de la aplicación lógica, que debe ser único en todas las regiones. El nombre solo puede contener letras, números, guiones (`-`), caracteres de subrayado (`_`), paréntesis (`(`, `)`) y puntos (`.`). En este ejemplo se usa "My-First-Logic-App". |
   | **Suscripción** | <*Azure-subscription-name*> | Nombre de la suscripción de Azure. |
   | **Grupos de recursos** | <*nombre del grupo de recursos de Azure*> | Nombre del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md#terminology) en el que va a crear la aplicación lógica. El nombre del grupo de recursos debe ser único en todas las regiones. En este ejemplo se usa "My-First-LA-RG". |
   | **Ubicación** | <*Azure-region*> | Región de Azure en la que desea almacenar la información de la aplicación lógica. En este ejemplo se utiliza "Oeste de EE. UU.". |
   | **Log Analytics** | Desactivado | Configuración para el registro de diagnóstico, que es **Desactivado** de forma predeterminada. Mantenga el valor **Desactivado** para este ejemplo. |
   ||||

   ![Captura de pantalla de la página de creación de aplicaciones lógicas, donde se muestra el panel con detalles de la nueva aplicación lógica.](./media/quickstart-create-first-logic-app-workflow/create-logic-app-settings.png)

1. Seleccione **Revisar y crear** cuando esté preparado. Confirme los detalles que proporcionó y seleccione **Crear**.

1. Una vez que Azure implemente correctamente la aplicación, seleccione **Ir al recurso**. O también puede buscar y seleccionar la aplicación lógica escribiendo el nombre en el cuadro de búsqueda.

   ![Captura de pantalla que muestra la página de implementación del recurso y el botón seleccionado, "Ir al recurso".](./media/quickstart-create-first-logic-app-workflow/go-to-new-logic-app-resource.png)

   El Diseñador de aplicación lógica se abre y muestra una página con un vídeo de introducción y desencadenadores utilizados frecuentemente. En **Plantillas**, elija **Blank Logic App**.

   ![Captura de pantalla del diseñador de Logic Apps, que muestra la galería de plantillas y la plantilla seleccionada, "Blank Logic App" (Aplicación lógica en blanco).](./media/quickstart-create-first-logic-app-workflow/choose-logic-app-template.png)

A continuación, [agregue un desencadenador a la aplicación lógica](#add-rss-trigger).

<a name="add-rss-trigger"></a>

## <a name="add-the-rss-trigger"></a>Agregar el desencadenador de RSS

Toda aplicación lógica debe comenzar con un [desencadenador](../logic-apps/logic-apps-overview.md#how-do-logic-apps-work), que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador encuentra nuevos elementos se activa y el motor de Logic Apps crea una instancia de la aplicación lógica que se inicia y ejecuta el flujo de trabajo. Si el desencadenador no encuentra nuevos elementos, no se activa y no crea una instancia ni ejecuta el flujo de trabajo en esta comprobación.

En este ejemplo de inicio rápido, después de [crear una aplicación lógica](#create-your-logic-app), se agrega un desencadenador que busca nuevos elementos en una fuente RSS y se activa cuando hay nuevos elementos. También puede crear aplicaciones lógicas con distintos tipos de desencadenadores, como en el tutorial [Creación de flujos de trabajo automatizados basados en aprobación mediante Azure Logic Apps](tutorial-process-mailing-list-subscriptions-workflow.md).

1. En el **Diseñador de aplicación lógica**, debajo del cuadro de búsqueda, seleccione **Todo**.

1. En el cuadro de búsqueda, escriba `rss` para buscar el conector de RSS. En la lista **Desencadenadores**, seleccione el desencadenador de RSS, **Cuando se publica un elemento de fuente**.

   ![Captura de pantalla que muestra el diseñador de aplicaciones lógicas con "rss" en el cuadro de búsqueda y el desencadenador de RSS seleccionado, "Cuando se publica un elemento de fuente".](./media/quickstart-create-first-logic-app-workflow/add-rss-trigger-new-feed-item.png)

1. Proporcione la dirección URL de la fuente RSS para el desencadenador. A continuación, defina la programación del desencadenador; para ello, establezca el intervalo y la frecuencia.

   | Propiedad | Value | Descripción |
   | -------- | ----- | ----------- |
   | **URL de fuente RSS** | <*URL-de-fuente-RSS*> | Dirección URL de la fuente RSS que desea supervisar. En este ejemplo se usa la fuente RSS del Wall Street Journal en `https://feeds.a.dj.com/rss/RSSMarketsMain.xml`. Sin embargo, para los fines de este ejemplo, puede usar cualquier fuente RSS que no requiera autorización HTTP. Elija una fuente RSS que publique con frecuencia, para después poder probar la aplicación lógica fácilmente. |
   | **Intervalo** | 1 | Número de intervalos que se espera entre comprobaciones de la fuente RSS. Este ejemplo utiliza intervalos de 1 minuto. |
   | **Frecuencia** | Minute | Unidad de tiempo para cada intervalo entre las comprobaciones de la fuente RSS. Este ejemplo utiliza intervalos de 1 minuto. |
   ||||

   ![Captura de pantalla que muestra el diseñador de aplicaciones lógicas con la configuración del desencadenador de RSS, la cual incluye la dirección URL, la frecuencia y el intervalo de RSS.](./media/quickstart-create-first-logic-app-workflow/add-rss-trigger-settings.png)

1. Para ocultar por ahora los detalles del desencadenador, haga clic dentro de la barra de título.

   ![Captura de pantalla que muestra el diseñador de aplicaciones lógicas con la forma contraída de la aplicación lógica.](./media/quickstart-create-first-logic-app-workflow/collapse-trigger-shape.png)

1. Guarde la aplicación lógica; para ello, seleccione **Guardar** en la barra de herramientas del diseñador.

La aplicación lógica está activa ahora, pero no hace nada salvo comprobar la fuente RSS. A continuación, [agregue una acción](#add-email-action) para definir lo que ocurre cuando se activa el desencadenador.

<a name="add-email-action"></a>

## <a name="add-the-send-email-action"></a>Adición de la acción "enviar correo electrónico"

Después de [agregar un desencadenador para la aplicación lógica](#add-rss-trigger), debe agregar una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) para determinar la respuesta cuando la aplicación lógica comprueba la fuente RSS y aparece un nuevo elemento. También puede crear aplicaciones lógicas con acciones mucho más complejas, como en el tutorial [Automatización de tareas para procesar correos electrónicos mediante Azure Logic Apps, Azure Functions y Azure Storage](./tutorial-process-email-attachments-workflow.md).

> [!NOTE]
> En este ejemplo se usa Outlook de Office 365 como servicio de correo electrónico. Si usa otro servicio de correo electrónico admitido en la aplicación lógica, la interfaz de usuario podría tener un aspecto diferente. Sin embargo, los conceptos básicos para conectarse a otro servicio de correo electrónico son los mismos.

1. En el desencadenador **Cuando se publica un elemento de fuente**, elija **Nuevo paso**.

   ![Captura de pantalla del diseñador de Logic Apps, que muestra un flujo de trabajo con el botón seleccionado, "Nuevo paso".](./media/quickstart-create-first-logic-app-workflow/add-new-step-under-trigger.png)

1. En **Elegir una acción** y en el cuadro de búsqueda, seleccione **Todas**.

1. En el cuadro de búsqueda, escriba `send an email` para buscar conectores que ofrezcan esta acción. Para filtrar la lista de acciones de una aplicación o un servicio específicos, puede seleccionar la aplicación o servicio primero.

   Por ejemplo, si usa una cuenta profesional o educativa de Microsoft y desea usar Office 365 Outlook, seleccione **Office 365 Outlook**. O, si usa una cuenta de Microsoft personal, puede seleccionar Outlook.com. Este ejemplo continúa con Office 365 Outlook:

   ![Captura de pantalla del diseñador de Logic Apps, que muestra el paso de acción con el conector de correo electrónico seleccionado, "Office 365 Outlook".](./media/quickstart-create-first-logic-app-workflow/select-connector.png)

   Ahora puede buscar más fácilmente y seleccionar la acción que desea usar, por ejemplo, `send an email`:

   ![Captura de pantalla del diseñador de Logic Apps, que muestra la lista de acciones filtradas para el conector de correo electrónico, "Office 365 Outlook".](./media/quickstart-create-first-logic-app-workflow/filtered-actions-list.png)

1. Si el conector de correo electrónico seleccionado le pide que autentique su identidad, complete ese paso ahora. Debe crear una conexión entre la aplicación lógica y el servicio de correo electrónico para que este ejemplo funcione. 

    > [!NOTE]
    > En este ejemplo se muestra la autenticación manual con el conector de Outlook de Office 365. Sin embargo, otros conectores podrían admitir distintos tipos de autenticación.
    > También puede controlar la autenticación de las aplicaciones lógicas de maneras diferentes, en función del caso de uso. Por ejemplo, si se usan plantillas de Azure Resource Manager para la implementación, es posible parametrizar para mejorar la seguridad de las entradas que cambian con frecuencia, como la información de la conexión. Para más información, consulte los temas siguientes:
   > * [Parámetros de plantilla para la implementación](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#template-parameters)
   > * [Autorización de conexiones de OAuth](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md#authorize-oauth-connections)
   > * [Autenticación y acceso con identidades administradas](../logic-apps/create-managed-service-identity.md)
   > * [Autenticación de conexiones para la implementación de aplicaciones lógicas](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md#authenticate-connections)

1. En la acción **Enviar un correo electrónico**, especifique la información que desea incluir en la notificación por correo electrónico.

   1. En el cuadro **Para**, escriba la dirección de correo electrónico del destinatario. Para este ejemplo, utilice su dirección de correo electrónico.

        > [!NOTE]
        > La lista **Agregar contenido dinámico** aparece al hacer clic dentro del cuadro **Para** y algunos otros cuadros de entrada del diseñador de Logic Apps. En este ejemplo se usa contenido dinámico en un paso posterior. La lista **Agregar contenido dinámico** muestra las salidas disponibles del paso anterior, que puede usar como entradas para la acción actual.

   1. En el cuadro de texto **Asunto**, escriba el asunto de la notificación por correo electrónico. Para este ejemplo, escriba el texto siguiente con un espacio en blanco al final: `New RSS item: `

      ![Captura de pantalla del diseñador de Logic Apps que muestra la acción "Enviar un correo electrónico" y el cursor dentro del cuadro de la propiedad "Asunto".](./media/quickstart-create-first-logic-app-workflow/send-email-subject.png)

   1. En la lista **Agregar contenido dinámico**, seleccione **Título de fuente**, que es la salida del desencadenador **Cuando se publica un elemento de fuente**. La notificación de correo electrónico utiliza esta salida para obtener el título del elemento RSS.

      ![Captura de pantalla del diseñador de aplicaciones lógicas con la acción "Enviar un correo electrónico" y el cursor dentro del cuadro de propiedad "Asunto" con una lista de contenido dinámico abierta y la salida seleccionada, "Título de fuente".](./media/quickstart-create-first-logic-app-workflow/send-email-subject-dynamic-content.png)

      > [!TIP]
      > En la lista de contenido dinámico, si no aparece ninguna salida del desencadenador **Cuando se publica un elemento de fuente**, junto al encabezado de la acción, seleccione **Ver más**.
      > 
      > ![Captura de pantalla del diseñador de aplicaciones lógicas con la lista de contenido dinámico abierta y la opción "Ver más" seleccionada para el desencadenador.](./media/quickstart-create-first-logic-app-workflow/dynamic-content-list-see-more-actions.png)

      Cuando haya terminado, el asunto del correo electrónico será similar a este ejemplo:

      ![Captura de pantalla del diseñador de aplicaciones lógicas con la acción "Enviar un correo electrónico" y un asunto de correo electrónico de ejemplo con la propiedad "Título de fuente" incluida.](./media/quickstart-create-first-logic-app-workflow/send-email-feed-title.png)

      Si aparece un bucle "Para cada uno" en el diseñador, habrá seleccionado un token de una matriz, como el token **categorías-elemento**. Para este tipo de token, el diseñador añade automáticamente este bucle alrededor de la acción que hace referencia a ese token. De este modo, la aplicación lógica realiza la misma acción en cada elemento de la matriz. Para quitar el bucle, elija el botón de **puntos suspensivos** (**...**) de la barra de título del bucle y luego **Eliminar**.

   1. En el cuadro **Cuerpo**, escriba el contenido del cuerpo del correo electrónico. En este ejemplo, el contenido incluye tres propiedades con un texto descriptivo para cada una: `Title:`, la propiedad **Título de fuente**; `Date published:`, la propiedad **Fuente publicada el**; y `Link:`, la propiedad **Vínculo de fuente principal**. Para agregar líneas en blanco en el cuadro de edición, presione Mayús + Entrar.

      | Propiedad | Descripción |
      |----------|-------------|
      | **Título de fuente** | Título del elemento |
      | **Fuente publicada el** | Fecha y hora de publicación del elemento |
      | **Vínculo de fuente principal** | Dirección URL del elemento |
      |||

      ![Captura de pantalla del diseñador de aplicaciones lógicas con la acción "Enviar un correo electrónico" y las propiedades seleccionadas dentro del cuadro "Cuerpo".](./media/quickstart-create-first-logic-app-workflow/send-email-body.png)

1. Guarde la aplicación lógica. En el menú del diseñador, seleccione **Guardar**.

A continuación, [pruebe que la aplicación lógica funciona](#test-logic-app).

<a name="test-logic-app"></a>

## <a name="run-your-logic-app"></a>Ejecución de la aplicación lógica

Después de crear la aplicación lógica de ejemplo, confirme que el flujo de trabajo está configurado correctamente. Puede esperar a que la aplicación lógica compruebe la fuente RSS según la programación especificada. O bien, puede ejecutar manualmente la aplicación lógica; para ello, seleccione **Ejecutar** en la barra de herramientas del diseñador de Logic Apps, tal como se muestra en la siguiente captura de pantalla. 

Si la fuente RSS tiene nuevos elementos, la aplicación lógica envía un correo electrónico para cada uno de ellos. En caso contrario, la aplicación lógica espera hasta el siguiente intervalo para comprobar de nuevo la fuente RSS. 

![Captura de pantalla del diseñador de aplicaciones lógicas que muestra el botón "Ejecutar" seleccionado en la barra de herramientas del diseñador.](./media/quickstart-create-first-logic-app-workflow/run-logic-app-test.png)

En la captura de pantalla siguiente se muestra una notificación por correo electrónico de ejemplo de esta aplicación lógica de ejemplo. El correo electrónico incluye los detalles de cada uno de los elementos de la fuente RSS seleccionados en el diseñador, así como el texto descriptivo agregado para cada uno.

![Captura de pantalla de Outlook que muestra el mensaje de correo electrónico de ejemplo recibido cuando aparece un nuevo elemento en la fuente RSS,con el título, fecha de publicación y vínculo del elemento.](./media/quickstart-create-first-logic-app-workflow/monitor-rss-feed-email.png)

Si no recibe correos electrónicos de notificación de la aplicación lógica según lo previsto:

* Compruebe la carpeta de correo no deseado de la cuenta de correo electrónico, por si el mensaje se ha filtrado incorrectamente.
* Asegúrese de que la fuente RSS que usa ha publicado elementos desde la última comprobación programada o manual.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado de probar esta aplicación lógica de ejemplo, limpie la aplicación lógica y los recursos relacionados; para ello, elimine el grupo de recursos que creó para este ejemplo.

> [!NOTE]
> Cuando se [elimina una aplicación lógica](manage-logic-apps-with-azure-portal.md#delete-logic-apps), no se crean instancias de nuevas ejecuciones. Todas las ejecuciones nuevas y pendientes se cancelan. Si tiene miles de ejecuciones, la cancelación puede tardar bastante tiempo en completarse.

1. En el cuadro de búsqueda de Azure, escriba `resource groups` y seleccione **Grupos de recursos**.

   ![Captura de pantalla del cuadro de búsqueda de Azure Portal con el término de búsqueda "Grupos de recursos".](./media/quickstart-create-first-logic-app-workflow/find-resource-groups.png)

1. Busque y seleccione el grupo de recursos de la aplicación lógica. En el panel **Información general**, elija **Eliminar grupo de recursos**.

   ![Captura de pantalla que muestra Azure Portal con el grupo de recursos seleccionado y el botón para "Eliminar grupo de recursos".](./media/quickstart-create-first-logic-app-workflow/delete-resource-group.png)

1. Cuando aparezca el panel de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

   ![Captura de pantalla de Azure Portal con el panel de confirmación y el nombre del grupo de recursos que se va a eliminar.](./media/quickstart-create-first-logic-app-workflow/delete-resource-group-2.png)

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado su primera aplicación lógica en Azure Portal para comprobar si hay actualizaciones en una fuente RSS según una programación y enviar una notificación por correo electrónico sobre cada nuevo elemento de la fuente. 

Para obtener información sobre cómo crear flujos de trabajo basados en programación más avanzados en Logic Apps, consulte el siguiente tutorial:

> [!div class="nextstepaction"]
> [Comprobación del tráfico con una aplicación lógica basada en una programación](../logic-apps/tutorial-build-schedule-recurring-logic-app-workflow.md)