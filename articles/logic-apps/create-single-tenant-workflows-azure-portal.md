---
title: Creación de flujos de trabajo en Azure Logic Apps de un solo inquilino mediante Azure Portal
description: Cree flujos de trabajo automatizados que integren aplicaciones, datos, servicios y sistemas mediante Azure Logic Apps de un solo inquilino y Azure Portal.
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, azla
ms.topic: how-to
ms.date: 05/25/2021
ms.openlocfilehash: 8058f6a912981f44205a6a2739fe2be54141d3dd
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110368219"
---
# <a name="create-an-integration-workflow-using-single-tenant-azure-logic-apps-and-the-azure-portal"></a>Creación de un flujo de trabajo de integración mediante Azure Logic Apps de un solo inquilino y Azure Portal

En este artículo se muestra cómo crear un flujo de trabajo de integración automatizado de ejemplo que se ejecuta en el *entorno de Azure Logic Apps de un solo inquilino* mediante el tipo de recurso **Logic Apps (estándar)** . Si es nuevo en el nuevo modelo de un solo inquilino y el tipo de recurso de aplicación lógica, revise [Comparación de las opciones de un solo inquilino, multiinquilino y entorno del servicio de integración](single-tenant-overview-compare.md).

Aunque este flujo de trabajo de ejemplo está basado en la nube y solo tiene dos pasos, puede crear flujos de trabajo a partir de cientos de operaciones que pueden conectar una amplia variedad de aplicaciones, datos, servicios y sistemas en entornos híbridos, locales y en la nube. El flujo de trabajo de ejemplo comienza con el desencadenador Solicitud integrado y sigue con una acción de Office 365 Outlook. El desencadenador crea un punto de conexión al que se puede llamar para el flujo de trabajo y espera una solicitud HTTPS entrante de cualquier autor de llamada. Cuando el desencadenador recibe una solicitud y se activa, la siguiente acción se ejecuta mediante el envío de un correo electrónico a la dirección de correo electrónico especificada junto con las salidas seleccionadas del desencadenador.

> [!TIP]
> Si no tiene ninguna cuenta de Office 365, puede usar cualquier otra acción disponible que pueda enviar mensajes desde su cuenta de correo electrónico, como, por ejemplo, Outlook.com.
> 
> Para crear este flujo de trabajo de ejemplo mediante Visual Studio Code, siga los pasos descritos en [Creación de flujos de trabajo de integración mediante Azure Logic Apps de un solo inquilino y Visual Studio Code](create-single-tenant-workflows-visual-studio-code.md). 
> Ambas opciones proporcionan la funcionalidad para desarrollar, ejecutar e implementar flujos de trabajo de aplicaciones lógicas en los mismos tipos de entornos. 
> Sin embargo, con Visual Studio Code, puede desarrollar, probar y ejecutar flujos de trabajo *localmente* en el entorno de desarrollo.

![Captura de pantalla que muestra Azure Portal con el diseñador de flujos de trabajo para el recurso "Logic Apps (estándar)".](./media/create-single-tenant-workflows-azure-portal/azure-portal-logic-apps-overview.png)

A medida que avance, completará estas tareas de alto nivel:

* Crear el nuevo recurso de aplicación lógica y agregar un flujo de trabajo [*con estado*](single-tenant-overview-compare.md#stateful-stateless) en blanco.
* Agregar un desencadenador y acción.
* Desencadenar una ejecución de flujo de trabajo.
* Ver el historial de ejecución y de desencadenadores del flujo de trabajo.
* Habilitar o abrir Application Insights después de la implementación.
* Habilitar el historial de ejecución para flujos de trabajo sin estado.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Una [cuenta de Azure Storage](../storage/common/storage-account-overview.md). Si no tiene ninguna, puede crear una cuenta de almacenamiento de antemano o durante la creación de la aplicación lógica.

  > [!NOTE]
  > El tipo de recurso **Logic Apps (estándar)** cuenta con tecnología de Azure Functions y presenta [requisitos de almacenamiento similares a los de las aplicaciones de funciones](../azure-functions/storage-considerations.md). 
  > Los [flujos de trabajo con estado](single-tenant-overview-compare.md#stateful-stateless) realizan transacciones de almacenamiento, como usar colas para programar y almacenar estados de flujo de trabajo en tablas y blobs. Estas transacciones generan [cargos de almacenamiento](https://azure.microsoft.com/pricing/details/storage/). Para más información sobre cómo los flujos de trabajo con estado almacenan datos en el almacenamiento externo, revise [Flujos de trabajo con y sin estado](single-tenant-overview-compare.md#stateful-stateless).

* Para crear el mismo flujo de trabajo de ejemplo de este artículo, necesita una cuenta de correo electrónico de Office 365 Outlook que use una cuenta profesional o educativa de Microsoft para iniciar sesión.

  Si elige un [conector de correo electrónico diferente](/connectors/connector-reference/connector-reference-logicapps-connectors), como Outlook.com, puede seguir el ejemplo, ya que los pasos generales son los mismos. Sin embargo, las opciones pueden diferir de alguna manera. Por ejemplo, si usa el conector de Outlook.com, en su lugar use su cuenta personal de Microsoft para iniciar sesión.

* Para probar el flujo de trabajo de ejemplo de este artículo, necesita una herramienta que pueda enviar llamadas al punto de conexión creado por el desencadenador Solicitud. Si no dispone de esta herramienta, puede descargar, instalar y usar [Postman](https://www.postman.com/downloads/).

* Si crea los recursos de la aplicación lógica con una configuración que admite el uso de [Application Insights](../azure-monitor/app/app-insights-overview.md), tiene la opción de habilitar el registro de diagnóstico y el seguimiento para la aplicación lógica. Puede hacerlo al crear la aplicación lógica o después de la implementación. Debe tener una instancia de Application Insights, pero puede crear este recurso [con antelación](../azure-monitor/app/create-workspace-resource.md), al crear la aplicación lógica o después de la implementación.

<a name="create-logic-app-resource"></a>

## <a name="create-the-logic-app-resource"></a>Creación del recurso de aplicación lógica

1. En [Azure Portal](https://portal.azure.com), inicie sesión con las credenciales de su cuenta de Azure.

1. En el cuadro de búsqueda de Azure Portal, escriba `logic apps` y seleccione **Logic Apps**.

   ![Captura de pantalla que muestra el cuadro de búsqueda de Azure Portal con el término de búsqueda "logic apps" y el recurso "Logic Apps (estándar)" seleccionado.](./media/create-single-tenant-workflows-azure-portal/find-logic-app-resource-template.png)

1. En la página **Logic Apps**, seleccione **Agregar** > **Estándar**.

   Este paso crea un recurso de aplicación lógica que se ejecuta en el entorno de Logic Apps de un solo inquilino y usa el [modelo de precios de un solo inquilino](logic-apps-pricing.md#standard-pricing).

1. En la página **Crear aplicación lógica**, en la pestaña **Aspectos básicos**, proporcione la siguiente información sobre el recurso de aplicación lógica:

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Suscripción** | Sí | <*Azure-subscription-name*> | La suscripción de Azure que se usa para la aplicación lógica. |
   | **Grupo de recursos** | Sí | <*nombre del grupo de recursos de Azure*> | El grupo de recursos de Azure en el que se crea la aplicación lógica y los recursos relacionados. El nombre del recurso debe ser único entre las regiones y solo puede contener letras, números, guiones ( **-** ), caracteres de subrayado ( **_** ), paréntesis ( **()** ) y puntos ( **.** ). <p><p>En este ejemplo se crea un grupo de recursos denominado `Fabrikam-Workflows-RG`. |
   | **Nombre de la aplicación lógica** | Sí | <*nombre-de-la-aplicación-lógica*> | Nombre que se va a usar para la aplicación lógica. El nombre del recurso debe ser único entre las regiones y solo puede contener letras, números, guiones ( **-** ), caracteres de subrayado ( **_** ), paréntesis ( **()** ) y puntos ( **.** ). <p><p>En este ejemplo se crea una aplicación lógica denominada `Fabrikam-Workflows`. <p><p>**Nota**: El nombre de la aplicación lógica obtiene automáticamente el sufijo `.azurewebsites.net`, ya que el recurso **Logic Apps (estándar)** se basa en Azure Functions, que usa la misma convención de nomenclatura de la aplicación. |
   | **Publicar** | Sí | <*entorno-de-implementación*> | El destino de implementación de la aplicación lógica. De forma predeterminada, **Workflow** es la única opción del portal disponible actualmente y se implementa en Azure Logic Apps de un solo inquilino. <p><p>**Nota**: Azure crea un recurso de aplicación lógica vacío donde tiene que agregar el primer flujo de trabajo. |
   | **Región** | Sí | <*Azure-region*> | Región de Azure que se va a usar al crear el grupo de recursos y los recursos. <p><p>En este ejemplo se usa **West US**. |
   |||||

   Este es un ejemplo:

   ![Captura de pantalla que muestra Azure Portal y la página "Crear aplicación lógica".](./media/create-single-tenant-workflows-azure-portal/create-logic-app-resource-portal.png)

1. A continuación, en la pestaña **Hosting**, proporcione la información siguiente acerca de la solución de almacenamiento y el plan de hospedaje que se usará para la aplicación lógica.

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Cuenta de almacenamiento** | Sí | <*Azure-storage-account-name*> | La [cuenta de Azure Storage](../storage/common/storage-account-overview.md) que se usará para transacciones de almacenamiento. <p><p>Este nombre de recurso debe ser único en todas las regiones y tener de 3 a 24 caracteres (solo números y letras minúsculas). Seleccione una cuenta existente o cree una nueva. <p><p>En este ejemplo se crea una cuenta de almacenamiento denominada `fabrikamstorageacct`. |
   | **Tipo de plan** | Sí | <*plan-de-hospedaje*> | Plan de hospedaje que se usará para implementar la aplicación lógica. <p><p>Para más información, revise [Planes de hospedaje y planes de tarifa](logic-apps-pricing.md#standard-pricing). |
   | **Plan de Windows** | Sí | <*nombre-de-plan*> | El nombre de plan que se usará. Seleccione el nombre de un plan existente o proporcione un nombre para un plan nuevo. <p><p>Este ejemplo usa el nombre de `Fabrikam-Service-Plan`. |
   | **SKU y tamaño** | Sí | <*plan-de-tarifa*> | [Plan de tarifa](../app-service/overview-hosting-plans.md) que se usará para la aplicación lógica. La selección afecta a los precios, al proceso, a la memoria y al almacenamiento que se usan en la aplicación lógica y los flujos de trabajo. <p><p>Para cambiar el plan de tarifa predeterminado, seleccione **Cambiar tamaño**. Después, puede seleccionar otros planes de tarifa, en función de la carga de trabajo que necesite. <p><p>Para más información, revise [Planes de hospedaje y planes de tarifa](logic-apps-pricing.md#standard-pricing). |
   |||||

1. A continuación, si la configuración de la creación e implementación admite el uso de [Application Insights](../azure-monitor/app/app-insights-overview.md), puede habilitar opcionalmente el registro de diagnóstico y el seguimiento para la aplicación lógica.

   1. En la pestaña **Supervisión**, en **Application Insights**, establezca **Habilitar Application Insights** en **Sí**, si aún no está seleccionado.

   1. En la opción **Application Insights**, seleccione una instancia de Application Insights existente o, si quiere crear una nueva instancia, seleccione **Crear nueva** y proporcione el nombre que usará.

1. Una vez que Azure valide la configuración de la aplicación lógica, en la pestaña **Revisar y crear**, seleccione **Crear**.

   Por ejemplo:

   ![Captura de pantalla que muestra Azure Portal y la configuración del recurso nuevo de aplicación lógica.](./media/create-single-tenant-workflows-azure-portal/check-logic-app-resource-settings.png)

   > [!TIP]
   > Si recibe un error de validación después de seleccionar **Crear**, abra y revise los detalles del error. Por ejemplo, si la región seleccionada alcanza una cuota para los recursos que está intentando crear, es posible que tenga que probar otra región.

   Una vez que Azure finaliza la implementación, la aplicación lógica se activa y se ejecuta automáticamente, pero aún no hace nada porque el recurso está vacío y aún no ha agregado ningún flujo de trabajo.

1. En la página de finalización de la implementación, seleccione **Ir al recurso** para que pueda agregar un flujo de trabajo en blanco.

   ![Captura de pantalla que muestra Azure Portal y la implementación finalizada.](./media/create-single-tenant-workflows-azure-portal/logic-app-completed-deployment.png)

<a name="add-workflow"></a>

## <a name="add-a-blank-workflow"></a>Adición de un flujo de trabajo en blanco

Después de crear el recurso de aplicación lógica vacío, debe agregar su primer flujo de trabajo.

1. Después de que Azure abra el recurso, en el menú de la aplicación lógica, seleccione **Flujos de trabajo**. En la barra de herramientas **Flujos de trabajo**, seleccione **Agregar**.

   ![Captura de pantalla que muestra el menú de recursos de la aplicación lógica, en la que se ha seleccionado "Flujos de trabajo" y, en la barra de herramientas, "Agregar".](./media/create-single-tenant-workflows-azure-portal/logic-app-add-blank-workflow.png)

1. Una vez que se abra el panel **Nuevo flujo de trabajo**, proporcione un nombre para el flujo de trabajo y elija el tipo de estado, [**Con estado** o **Sin estado**](single-tenant-overview-compare.md#stateful-stateless). Seleccione **Crear** cuando haya terminado.

   En este ejemplo se agrega un flujo de trabajo con estado en blanco denominado `Fabrikam-Stateful-Workflow`. De forma predeterminada, el flujo de trabajo está habilitado, pero no hace nada hasta que se agrega un desencadenador y acciones.

   ![Captura de pantalla que muestra el flujo de trabajo con estado en blanco recién agregado "Fabrikam-Stateful-Workflow".](./media/create-single-tenant-workflows-azure-portal/logic-app-blank-workflow-created.png)

1. A continuación, abra el flujo de trabajo en blanco en el diseñador para que pueda agregar un desencadenador y una acción.

   1. En la lista de flujos de trabajo, seleccione el flujo de trabajo en blanco.

   1. En el menú del flujo de trabajo, en **Programador**, seleccione **Diseñador**.

      En la superficie del diseñador, la solicitud **Elija una operación** ya aparece y está seleccionada de forma predeterminada para que el panel **Agregar un desencadenador** también aparezca abierto.

      ![Captura de pantalla que muestra el diseñador de flujos de trabajo abierto con la solicitud "Elija una operación" seleccionada en la superficie del diseñador.](./media/create-single-tenant-workflows-azure-portal/opened-logic-app-designer-blank-workflow.png)

<a name="add-trigger-actions"></a>

## <a name="add-a-trigger-and-an-action"></a>Adición de un desencadenador y una acción

En este ejemplo se crea un flujo de trabajo que tiene estos pasos:

* El [desencadenar Solicitud](../connectors/connectors-native-reqres.md) integrado, **Cuando se recibe una solicitud HTTP**, que recibe llamadas o solicitudes entrantes y crea un punto de conexión al que pueden llamar otros servicios o aplicaciones lógicas.

* La [acción de Office 365 Outlook](../connectors/connectors-create-api-office365-outlook.md), **Enviar correo electrónico**.

### <a name="add-the-request-trigger"></a>Adición del desencadenador Request

Antes de poder agregar un desencadenador a un flujo de trabajo en blanco, asegúrese de que el diseñador de flujos de trabajo esté abierto y de que la solicitud **Elija una operación** esté seleccionada en la superficie del diseñador.

1. Junto a la superficie del diseñador, en el panel **Agregar un desencadenador**, en el cuadro de búsqueda **Elija una operación**, compruebe que la pestaña **Integrado** esté seleccionada. En esta pestaña se muestran los desencadenadores que se ejecutan de forma nativa en Azure Logic Apps.

1. En el cuadro de búsqueda **Elegir una operación**, escriba `when a http request` y seleccione el desencadenador Solicitud integrado denominado **Cuando se recibe una solicitud HTTP**.

   ![Captura de pantalla que muestra el diseñador y el panel **Agregar un desencadenador** con el desencadenador "Cuando se recibe una solicitud HTTP" seleccionado.](./media/create-single-tenant-workflows-azure-portal/find-request-trigger.png)

   Cuando el desencadenador aparece en el diseñador, se abre el panel de detalles del desencadenador para mostrar las propiedades, la configuración y otras acciones del desencadenador.

   ![Captura de pantalla que muestra el diseñador con el desencadenador "Cuando se recibe una solicitud HTTP" seleccionado y el panel de detalles del desencadenador abierto.](./media/create-single-tenant-workflows-azure-portal/request-trigger-added-to-designer.png)

   > [!TIP]
   > Si el panel de detalles no aparece, asegúrese de que el desencadenador esté seleccionado en el diseñador.

1. Si necesita eliminar un elemento del diseñador, [siga estos pasos para eliminar elementos del diseñador](#delete-from-designer).

1. Para guardar el trabajo, en la barra de herramientas del diseñador, seleccione **Guardar**.

   Cuando guarda un flujo de trabajo por primera vez y ese flujo de trabajo se inicia con un desencadenador de solicitud, el servicio Logic Apps genera automáticamente una dirección URL para un punto de conexión creado por el desencadenador de solicitud. Más adelante, al probar el flujo de trabajo, envía una solicitud a esta dirección URL, que activa el desencadenador e inicia la ejecución del flujo de trabajo.

### <a name="add-the-office-365-outlook-action"></a>Adición de la acción Office 365 Outlook

1. En el diseñador, en el desencadenador que agregó, seleccione el signo más ( **+** ) > **Agregar una acción**.

   Aparece la solicitud **Elija una operación** en el diseñador, y se vuelve a abrir el panel **Agregar una acción** para que pueda seleccionar la acción siguiente.

   > [!NOTE]
   > Si en el panel **Agregar una acción** se muestra el mensaje de error "Cannot read property 'filter' of undefined" (No se puede leer la propiedad "filter" de sin definir), guarde el flujo de trabajo, vuelva a cargar la página, vuelva a abrir el flujo de trabajo e inténtelo de nuevo.

1. En el panel **Agregar una acción**, en el cuadro de búsqueda **Elija una operación**, seleccione **Azure**. Esta pestaña muestra los conectores administrados que están disponibles y que se hospedan en Azure.

   > [!NOTE]
   > Si en el panel **Agregar una acción** se muestra el mensaje de error `The access token expiry UTC time '{token-expiration-date-time}' is earlier than current UTC time '{current-date-time}'`, guarde el flujo de trabajo, vuelva a cargar la página, vuelva a abrir el flujo de trabajo y pruebe a agregar la acción de nuevo.

   En este ejemplo se usa la acción de Office 365 Outlook denominada **Enviar correo electrónico (V2)** .

   ![Captura de pantalla que muestra el diseñador y el panel **Agregar una acción** con la acción "Enviar correo electrónico" de Office 365 Outlook seleccionada.](./media/create-single-tenant-workflows-azure-portal/find-send-email-action.png)

1. En el panel de detalles de la acción, en la pestaña **Crear conexión**, seleccione **Iniciar sesión** para que pueda crear una conexión a su cuenta de correo electrónico.

   ![Captura de pantalla que muestra el diseñador y el panel de detalles **Enviar correo electrónico (V2)** con "Iniciar sesión" seleccionado.](./media/create-single-tenant-workflows-azure-portal/send-email-action-sign-in.png)

1. Cuando se le solicite acceso a su cuenta de correo electrónico, inicie sesión con las credenciales de la cuenta.

   > [!NOTE]
   > Si aparece el mensaje de error `Failed with error: 'The browser is closed.'. Please sign in again`, compruebe si el explorador bloquea las cookies de terceros. Si estas cookies están bloqueadas, intente agregar `https://portal.azure.com` a la lista de sitios que pueden usar cookies. Si usa el modo de incógnito, asegúrese de que las cookies de terceros no estén bloqueadas mientras se trabaja en ese modo.
   > 
   > Si es necesario, vuelva a cargar la página, abra el flujo de trabajo, vuelva a agregar la acción de correo electrónico y pruebe a crear la conexión.

   Una vez que Azure crea la conexión, la acción **Enviar un correo electrónico** aparece en el diseñador y está seleccionada de forma predeterminada. Si la acción no está seleccionada, seleccione la acción para que su panel de detalles también esté abierto.

1. En el panel de detalles de la acción, en la pestaña **Parámetros**, proporcione la información necesaria para la acción, por ejemplo:

   ![Captura de pantalla que muestra el diseñador y el panel de detalles **Enviar correo electrónico** con "Parámetros" seleccionado.](./media/create-single-tenant-workflows-azure-portal/send-email-action-details.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **To** | Sí | <*your-email-address*> | El destinatario del correo electrónico, que puede ser su dirección de correo electrónico con fines de prueba. En este ejemplo se usa la dirección de correo electrónico ficticia `sophiaowen@fabrikam.com`. |
   | **Subject** | Sí | `An email from your example workflow` | El asunto del correo electrónico |
   | **Cuerpo** | Sí | `Hello from your example workflow!` | El contenido del cuerpo del correo electrónico. |
   ||||

   > [!NOTE]
   > Al hacer cambios en el panel de detalles de las pestañas **Configuración**, **Resultado estático** o **Ejecutar después de**, asegúrese de seleccionar **Listo** para confirmar dichos cambios antes de cambiar de pestaña o cambiar el foco al diseñador. De lo contrario, el diseñador no conservará los cambios.

1. No olvide guardar su trabajo. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

1. Si el entorno tiene requisitos de red estrictos o firewalls que limitan el tráfico, debe permitir la configuración de los permisos de las conexiones de desencadenador o acción que existan en el flujo de trabajo Para buscar los nombres de dominio completos, revise [Búsqueda de nombres de dominio para el acceso al firewall](#firewall-setup).

   En caso contrario, para probar el flujo de trabajo, [desencadene manualmente una ejecución](#trigger-workflow).

<a name="firewall-setup"></a>

## <a name="find-domain-names-for-firewall-access"></a>Búsqueda de nombres de dominio para el acceso al firewall

Antes de implementar el flujo de trabajo de la aplicación lógica en Azure Portal, si su entorno tiene estrictos requisitos de red o firewalls que limitan el tráfico, debe configurar los permisos de red o del firewall para las conexiones de desencadenador o acción que existan en la aplicación lógica.

Para buscar las direcciones IP entrantes y salientes que se usan en la aplicación lógica y los flujos de trabajo, siga estos pasos:

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Redes (versión preliminar)** .

1. En la página de redes, busque y revise las secciones **Tráfico entrante** y **Tráfico saliente**.

Para buscar los nombres de dominio completos (FQDN) de las conexiones, siga estos pasos:

1. En el menú de la aplicación lógica, en **Flujos de trabajo**, seleccione **Conexiones**. En la pestaña **Conexiones de API**, seleccione el nombre del recurso de la conexión, por ejemplo:

   ![Captura de pantalla que muestra el menú de Azure Portal y la aplicación lógica con el nombre de recurso de conexión "Conexiones" y "office365" seleccionado.](./media/create-single-tenant-workflows-azure-portal/logic-app-connections.png)

1. Expanda el explorador lo suficientemente para que cuando aparezca la **Vista JSON** en la esquina superior derecha del explorador, seleccione **Vista JSON**.

   ![Captura de pantalla que muestra Azure Portal y el panel de conexión de API con la opción "Vista JSON" seleccionada.](./media/create-single-tenant-workflows-azure-portal/logic-app-connection-view-json.png)

1. Busque, copie y guarde el valor de la propiedad `connectionRuntimeUrl` en un lugar seguro para que pueda configurar el firewall con esta información.

   ![Captura de pantalla que muestra el valor de la propiedad "connectionRuntimeUrl" seleccionado.](./media/create-single-tenant-workflows-azure-portal/logic-app-connection-runtime-url.png)

1. En cada conexión, repita los pasos pertinentes.

<a name="trigger-workflow"></a>

## <a name="trigger-the-workflow"></a>Desencadenador del flujo de trabajo

En este ejemplo, el flujo de trabajo se ejecuta cuando el desencadenador de solicitud recibe una solicitud de entrada, que se envía a la dirección URL del punto de conexión que crea el desencadenador. Cuando se guardó el flujo de trabajo por primera vez, el servicio Logic Apps generó automáticamente esta dirección URL. Por lo tanto, antes de poder enviar la solicitud para desencadenar el flujo de trabajo, debe buscar esta dirección URL.

1. En el diseñador de flujos de trabajo, seleccione el desencadenador de solicitud denominado **Cuando se recibe una solicitud HTTP**.

1. Una vez que se abra el panel de detalles, en la pestaña **Parámetros**, busque la propiedad **HTTP POST URL**. Para copiar la dirección URL generada, seleccione **Copiar dirección URL** (icono de copiar archivo) y guarde la dirección URL en algún otro lugar por ahora. La dirección URL sigue este formato:

   `http://<logic-app-name>.azurewebsites.net:443/api/<workflow-name>/triggers/manual/invoke?api-version=2020-05-01-preview&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=<shared-access-signature>`

   ![Captura de pantalla que muestra el diseñador con el desencadenador de solicitud y la dirección URL del punto de conexión en la propiedad "HTTP POST URL".](./media/create-single-tenant-workflows-azure-portal/find-request-trigger-url.png)

   En este ejemplo, la dirección URL es similar a la siguiente:

   `https://fabrikam-workflows.azurewebsites.net:443/api/Fabrikam-Stateful-Workflow/triggers/manual/invoke?api-version=2020-05-01-preview&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=xxxxxXXXXxxxxxXXXXxxxXXXXxxxxXXXX`

   > [!TIP]
   > También puede encontrar la dirección URL del punto de conexión en el panel **Información general** de la aplicación lógica en la propiedad **URL del flujo de trabajo**.
   >
   > 1. En el menú del recurso, seleccione **Información general**.
   > 1. En el panel **Información general**, busque la propiedad **URL del flujo de trabajo**.
   > 1. Para copiar la dirección URL del punto de conexión, mueva el puntero sobre el final del texto de la dirección URL del punto de conexión y seleccione **Copiar en el Portapapeles** (icono de copiar archivo).

1. Para probar la dirección URL enviando una solicitud, abra [Postman](https://www.postman.com/downloads/) o su herramienta preferida para crear y enviar solicitudes.

   Este ejemplo continúa con Postman. Para obtener más información, consulte la [Introducción a Postman](https://learning.postman.com/docs/getting-started/introduction/).

   1. En la barra de herramientas de Postman, seleccione **New** (Nuevo).

      ![Captura de pantalla que muestra Postman con el botón Nuevo seleccionado.](./media/create-single-tenant-workflows-azure-portal/postman-create-request.png)

   1. En el panel **Create New** (Crear nuevo), en **Building Blocks** (Bloques de creación), seleccione **Request** (Solicitud).

   1. En la ventana **Save Request** (Guardar solicitud), en **Request name** (Nombre de solicitud), escriba un nombre para la solicitud, por ejemplo, `Test workflow trigger`.

   1. En **Select a collection or folder to save to** (Seleccionar una colección o carpeta donde guardar), seleccione **Create Collection** (Crear colección).

   1. En **All Collections** (Todas las colecciones), escriba un nombre para la colección que se va a crear para organizar las solicitudes, presione ENTRAR y, a continuación, seleccione **Save to <*nombre-de-la-colección*>** (Guardar en <nombre de la colección>). En este ejemplo se usa `Logic Apps requests` como nombre de la colección.

      Se abre el panel de solicitudes de Postman para que pueda enviar una solicitud a la dirección URL de punto de conexión para el desencadenador de solicitud.

      ![Captura de pantalla que muestra Postman con el panel de solicitudes abierto.](./media/create-single-tenant-workflows-azure-portal/postman-request-pane.png)

   1. En el panel de solicitudes, en el cuadro de dirección que se encuentra junto a la lista de métodos, que actualmente muestra **GET** como método de solicitud predeterminado, pegue la dirección URL que copió anteriormente y seleccione **Enviar**.

      ![Captura de pantalla que muestra Postman y la dirección URL de punto de conexión en el cuadro de dirección con el botón Enviar seleccionado](./media/create-single-tenant-workflows-azure-portal/postman-test-endpoint-url.png)

      Cuando se activa el desencadenador, se ejecuta el flujo de trabajo de ejemplo y este envía un correo electrónico similar a este ejemplo:

      ![Captura de pantalla que muestra el correo electrónico de Outlook tal como se describe en el ejemplo.](./media/create-single-tenant-workflows-azure-portal/workflow-app-result-email.png)

<a name="view-run-history"></a>

## <a name="review-run-history"></a>Revisar el historial de ejecución.

En el caso de un flujo de trabajo con estado, después de cada ejecución del flujo de trabajo, puede ver el historial de ejecución, incluido el estado de la ejecución general, del desencadenador y de cada acción junto con sus entradas y salidas. En Azure Portal, el historial de ejecución y los historiales de desencadenadores aparecen en el nivel del flujo de trabajo, no en el nivel de la aplicación lógica. Para revisar los historiales de desencadenadores fuera del contexto del historial de ejecución, consulte [Revisión de historiales de desencadenadores](#view-trigger-histories).

1. En Azure Portal, en el menú del flujo de trabajo, seleccione **Información general**.

1. En el panel **Información general**, seleccione **Historial de ejecución**, que muestra el historial de ejecución de ese flujo de trabajo.

   ![Captura de pantalla que muestra el panel "Información general" del flujo de trabajo con "Historial de ejecución" seleccionado.](./media/create-single-tenant-workflows-azure-portal/find-run-history.png)

   > [!TIP]
   > Si el estado de ejecución más reciente no aparece, en la barra de herramientas del panel **Información general**, seleccione **Actualizar**. No se produce ninguna ejecución para un desencadenador que se omite debido a criterios que no se cumplen o si no se encuentra ningún dato.

   | Estado de la ejecución | Descripción |
   |------------|-------------|
   | **Anulado** | La ejecución se ha detenido o no ha finalizado debido a problemas externos, por ejemplo, una interrupción del sistema o una suscripción de Azure vencida. |
   | **Cancelado** | La ejecución se desencadenó e inició, pero recibió una solicitud de cancelación. |
   | **Erróneo** | Se ha producido un error en al menos una acción de la ejecución. No se ha configurado ninguna acción posterior en el flujo de trabajo para controlar el error. |
   | **Ejecución** | La ejecución se ha desencadenado y está en curso, pero este estado también puede aparecer para una ejecución que está limitada debido a [límites de acción](logic-apps-limits-and-config.md) o al [plan de precios actual](https://azure.microsoft.com/pricing/details/logic-apps/). <p><p>**Sugerencia**: Si se configura un [registro de diagnóstico](monitor-logic-apps-log-analytics.md), se puede obtener información sobre los eventos de limitación que se produzcan. |
   | **Correcto** | La ejecución se ha completado correctamente. Si se ha producido un error en alguna acción, se ha controlado mediante una acción posterior en el flujo de trabajo. |
   | **Tiempo de espera agotado** | Se ha agotado el tiempo de espera de la ejecución porque la duración actual ha superado el límite de duración de la ejecución, que se controla mediante [el valor **Retención del historial de ejecución en días**](logic-apps-limits-and-config.md#run-duration-retention-limits). La duración de una ejecución se calcula mediante su hora de inicio y su límite de duración en esa hora de inicio. <p><p>**Nota**: Si la duración de la ejecución también supera el *límite de retención del historial de ejecución* actual, que también se controla mediante el [valor **Retención del historial de ejecución en días**](logic-apps-limits-and-config.md#run-duration-retention-limits), la ejecución se borra del historial de ejecución mediante un trabajo de limpieza diaria. Con independencia de que la ejecución agote el tiempo de espera o se complete, el período de retención siempre se calcula mediante la hora de inicio de la ejecución y el límite de retención *actual*. Por tanto, si reduce el límite de duración de una ejecución en curso, se agota su tiempo de espera. Pero la ejecución se mantiene o se borra del historial de ejecución en función de si su duración supera el límite de retención. |
   | **En espera** | La ejecución no se ha iniciado o está en pausa, por ejemplo, debido a un flujo de trabajo anterior que sigue en ejecución. |
   |||

1. Para revisar el estado de cada paso de una ejecución, seleccione la ejecución que desea revisar.

   Se abre la vista de detalles de la ejecución y se muestra el estado de cada paso de la ejecución.

   ![Captura de pantalla que muestra la vista de detalles de la ejecución con el estado de cada paso del flujo de trabajo.](./media/create-single-tenant-workflows-azure-portal/review-run-details.png)

   Estos son los posibles estados que puede tener cada paso del flujo de trabajo:

   | Estado de la acción | Descripción |
   |---------------|-------------|
   | **Anulado** | La acción se ha detenido o no ha finalizado debido a problemas externos; por ejemplo, una interrupción del sistema o una suscripción de Azure vencida. |
   | **Cancelado** | La acción se estaba ejecutando, pero recibió una solicitud de cancelación. |
   | **Erróneo** | Se produjo un error en la acción. |
   | **Ejecución** | La acción se está ejecutando actualmente. |
   | **Omitido** | La acción se ha omitido porque no se han cumplido sus condiciones `runAfter`; por ejemplo, no se ha podido completar la acción anterior. Cada acción tiene un objeto `runAfter` donde puede configurar las condiciones que deben cumplirse antes de que se pueda ejecutar la acción actual. |
   | **Correcto** | La acción se realizó correctamente. |
   | **Se realizó correctamente con reintentos** | La acción se ha realizado correctamente, pero solo después de uno o más reintentos. Para revisar el historial de reintentos en la vista de detalles del historial de ejecución, seleccione esa acción para que pueda ver las entradas y salidas. |
   | **Tiempo de espera agotado** | La acción se detuvo debido al límite de tiempo de espera que especificó la configuración de la acción. |
   | **En espera** | Se aplica a una acción de webhook que está esperando una solicitud entrante de un autor de llamada. |
   |||

   [aborted-icon]: ./media/create-single-tenant-workflows-azure-portal/aborted.png
   [cancelled-icon]: ./media/create-single-tenant-workflows-azure-portal/cancelled.png
   [failed-icon]: ./media/create-single-tenant-workflows-azure-portal/failed.png
   [running-icon]: ./media/create-single-tenant-workflows-azure-portal/running.png
   [skipped-icon]: ./media/create-single-tenant-workflows-azure-portal/skipped.png
   [succeeded-icon]: ./media/create-single-tenant-workflows-azure-portal/succeeded.png
   [succeeded-with-retries-icon]: ./media/create-single-tenant-workflows-azure-portal/succeeded-with-retries.png
   [timed-out-icon]: ./media/create-single-tenant-workflows-azure-portal/timed-out.png
   [waiting-icon]: ./media/create-single-tenant-workflows-azure-portal/waiting.png

1. Para revisar las entradas y salidas de cada paso específico, seleccione ese paso.

   ![Captura de pantalla que muestra las entradas y salidas de la acción "Enviar un correo electrónico" seleccionada.](./media/create-single-tenant-workflows-azure-portal/review-step-inputs-outputs.png)

1. Para revisar aún más las entradas y salidas sin procesar para ese paso, seleccione **Mostrar entradas sin procesar** o **Mostrar salidas sin procesar**.

<a name="view-trigger-histories"></a>

## <a name="review-trigger-histories"></a>Revisión de historiales de desencadenadores

Para un flujo de trabajo con estado, puede revisar el historial de desencadenadores para cada ejecución, incluido el estado del desencadenador junto con las entradas y salidas, independientemente del [contexto del historial de ejecución](#view-run-history). En Azure Portal, el historial de desencadenadores y el historial de ejecución aparecen en el nivel del flujo de trabajo, no en el nivel de la aplicación lógica. Para encontrar estos datos históricos, siga estos pasos:

1. En Azure Portal, en el menú del flujo de trabajo, seleccione **Información general**.

1. En la página **Información general**, seleccione **Historiales del desencadenador**.

   En el panel **Historiales del desencadenador** se muestran los historiales de los desencadenadores de las ejecuciones del flujo de trabajo.

1. Para revisar un historial de desencadenadores específico, seleccione el identificador de la ejecución.

<a name="enable-open-application-insights"></a>

## <a name="enable-or-open-application-insights-after-deployment"></a>Habilitación o apertura de Application Insights después de la implementación

Durante la ejecución del flujo de trabajo, la aplicación lógica emite telemetría junto con otros eventos. Puede usar esta telemetría para obtener una mejor visibilidad de si el flujo de trabajo se ejecuta correctamente y cómo funciona el runtime de Logic Apps de varias maneras. Puede supervisar el flujo de trabajo mediante [Application Insights](../azure-monitor/app/app-insights-overview.md), que proporciona telemetría casi en tiempo real (métricas en directo). Esta funcionalidad puede ayudarle a investigar los errores y problemas de rendimiento con más facilidad si usa estos datos para diagnosticar problemas, configurar alertas y crear gráficos.

Si la configuración de la creación e implementación de la aplicación lógica admite el uso de [Application Insights](../azure-monitor/app/app-insights-overview.md), puede habilitar opcionalmente el registro de diagnóstico y el seguimiento para la aplicación lógica. Puede hacerlo al crear la aplicación lógica en Azure Portal o después de la implementación. Debe tener una instancia de Application Insights, pero puede crear este recurso [con antelación](../azure-monitor/app/create-workspace-resource.md), al crear la aplicación lógica o después de la implementación.

Para habilitar Application Insights en una aplicación lógica implementada o abrir el panel de Application Insights si ya está habilitado, siga estos pasos:

1. En Azure Portal, busque la aplicación lógica implementada.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Application Insights**.

1. Si Application Insights no se ha habilitado, en el panel **Application Insights**, seleccione **Activar Application Insights**. Una vez que se actualice el panel, en la parte inferior, seleccione **Aplicar** > **Sí**.

   Si se ha habilitado Application Insights, en el panel **Application Insights**, seleccione **Ver datos de Application Insights**.

Después de que se abra Application Insights, puede revisar varias métricas de la aplicación lógica. Para obtener más información, consulte estos temas:

* [Azure Logic Apps en ejecución en cualquier ubicación: supervisión con Application Insights: parte 1](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-monitor-with-application/ba-p/1877849)
* [Azure Logic Apps en ejecución en cualquier ubicación: supervisión con Application Insights: parte 2](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-monitor-with-application/ba-p/2003332)

<a name="enable-run-history-stateless"></a>

## <a name="enable-run-history-for-stateless-workflows"></a>Habilitación del historial de ejecución para flujos de trabajo sin estado

Para depurar un flujo de trabajo sin estado con más facilidad, puede habilitar el historial de ejecución para ese flujo de trabajo y luego deshabilitar el historial de ejecución cuando haya terminado. Siga estos pasos para Azure Portal, o bien, si está trabajando en Visual Studio Code, consulte [Creación de flujos de trabajo con y sin estado en Visual Studio Code](create-single-tenant-workflows-visual-studio-code.md#enable-run-history-stateless).

1. En [Azure Portal](https://portal.azure.com), busque y abra el recurso **Logic Apps (estándar)** .

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Configuración**.

1. En la pestaña **Configuración de la aplicación**, seleccione **Nueva configuración de la aplicación**.

1. En el panel **Agregar o editar la configuración de la aplicación**, en el cuadro **Nombre**, escriba el nombre de opción de operación: 

   `Workflows.{yourWorkflowName}.OperationOptions`

1. En el cuadro **Valor**, escriba el siguiente valor: `WithStatelessRunHistory`

   Por ejemplo:

   ![Captura de pantalla que muestra Azure Portal y el recurso Logic Apps (estándar) con el panel "Configuración" > "Nueva configuración de la aplicación" < "Agregar o editar la configuración de la aplicación" abierto y la opción "Workflow.{nombreDelFlujoDeTrabajo}.OperationOptions" establecida en "WithStatelessRunHistory".](./media/create-single-tenant-workflows-azure-portal/stateless-operation-options-run-history.png)

1. Para finalizar esta tarea, seleccione **Aceptar**. En el panel **Configuración**, seleccione **Guardar**.

1. Para deshabilitar el historial de ejecución cuando haya terminado, establezca la propiedad `Workflows.{yourWorkflowName}.OperationOptions`en `None` o elimine la propiedad y su valor.

<a name="delete-from-designer"></a>

## <a name="delete-items-from-the-designer"></a>Eliminación de elementos del diseñador

Para eliminar un elemento del flujo de trabajo del diseñador, siga cualquiera de estos pasos:

* Seleccione el elemento, abra el menú contextual del elemento (Mayús+F10) y seleccione **Eliminar**. Para confirmar, seleccione **Aceptar**.

* Seleccione el elemento y presione la tecla Supr. Para confirmar, seleccione **Aceptar**.

* Seleccione el elemento para que se abra el panel de detalles de ese elemento. En la esquina superior derecha del panel, abra el menú de puntos suspensivos ( **...** ) y seleccione **Eliminar**. Para confirmar, seleccione **Aceptar**.

  ![Captura de pantalla que muestra el elemento seleccionado en el diseñador con el panel de detalles abierto y con el botón de puntos suspensivos seleccionado, así como el comando "Eliminar".](./media/create-single-tenant-workflows-azure-portal/delete-item-from-designer.png)

  > [!TIP]
  > Si no está visible el menú de puntos suspensivos, expanda la ventana del explorador el ancho suficiente para que en el panel de detalles se muestre el botón de puntos suspensivos ( **...** ) en la esquina superior derecha.

<a name="restart-stop-start"></a>

## <a name="restart-stop-or-start-logic-apps"></a>Reinicio, detención o inicio de aplicaciones lógicas

Puede iniciar o detener [una sola aplicación lógica](#restart-stop-start-single-logic-app) o [varias a la vez](#stop-start-multiple-logic-apps). También puede reiniciar una sola aplicación lógica sin detenerla primero. Su aplicación lógica basada en un solo inquilino puede incluir varios flujos de trabajo, por lo que puede detener toda la aplicación lógica o [deshabilitar solo flujos de trabajo](#disable-enable-workflows).

> [!NOTE]
> Estas dos operaciones tienen efectos diferentes. Para obtener más información, consulte [Consideraciones para detener aplicaciones lógicas](#considerations-stop-logic-apps) y [Consideraciones para deshabilitar flujos de trabajo](#disable-enable-workflows).

<a name="considerations-stop-logic-apps"></a>

### <a name="considerations-for-stopping-logic-apps"></a>Consideraciones para detener aplicaciones lógicas

Detener una aplicación lógica afecta a las instancias de flujo de trabajo de las maneras siguientes:

* Azure Logic Apps cancela de inmediato todas las ejecuciones en curso y pendientes.

* Azure Logic Apps no crea ni ejecuta nuevas instancias de flujo de trabajo.

* Los desencadenadores no se activarán la próxima vez que se cumplan las condiciones. Sin embargo, los estados de los desencadenadores recuerdan los puntos donde se detuvo la aplicación lógica. Por lo tanto, si reinicia la aplicación, se activarán para todos los elementos sin procesar desde la última ejecución.

  Para evitar que cada flujo de trabajo se active con elementos no procesados desde la última ejecución, siga estos pasos para borrar el estado del desencadenador antes de reiniciar la aplicación lógica:

  1. En Azure Portal, busque y abra la aplicación lógica.
  1. En el menú de la aplicación lógica, en **Flujos de trabajo**, seleccione **Flujos de trabajo**.
  1. Abra un flujo de trabajo y edite cualquier parte de su desencadenador.
  1. Guarde los cambios. Este paso restablece el estado actual del desencadenador.
  1. Repita el procedimiento para cada flujo de trabajo.
  1. Cuando haya terminado, [reinicie la aplicación lógica](#restart-stop-start-single-logic-app).

<a name="restart-stop-start-single-logic-app"></a>

### <a name="restart-stop-or-start-a-single-logic-app"></a>Reinicio, detención o inicio de una sola aplicación lógica

1. En Azure Portal, busque y abra la aplicación lógica.

1. En el menú de la aplicación lógica, seleccione **Información general**.

   * Para reiniciar una aplicación lógica sin detenerla, seleccione **Reiniciar** en la barra de herramientas del panel "Información general".
   * Para detener una aplicación lógica en ejecución, seleccione **Detener** en la barra de herramientas del panel "Información general". Confirme la selección.
   * Para iniciar una aplicación lógica detenida, seleccione **Iniciar** en la barra de herramientas del panel "Información general".

   > [!NOTE]
   > Si la aplicación lógica ya está detenida, solo aparecerá la opción **Iniciar**. Si ya está en ejecución, solo aparecerá la opción **Detener**.
   > Puede reiniciar la aplicación lógica en cualquier momento.

1. Para confirmar si la operación se ha realizado correctamente o no, abra la lista **Notificaciones** (icono de campana) en la barra de herramientas principal de Azure.

<a name="stop-start-multiple-logic-apps"></a>

### <a name="stop-or-start-multiple-logic-apps"></a>Detención o inicio de varias aplicaciones lógicas

Puede detener o iniciar varias aplicaciones lógicas a la vez, pero no reiniciar varias aplicaciones lógicas sin detenerlas primero.

1. En el cuadro de búsqueda principal de Azure Portal, escriba `logic apps` y seleccione **Aplicaciones lógicas**.

1. En la página **Aplicaciones lógicas,** revise la columna **Estado** de la aplicación lógica.

1. En la columna de casillas de verificación, seleccione las aplicaciones lógicas que quiera detener o iniciar.

   * Para detener las aplicaciones lógicas en ejecución seleccionadas, haga clic en **Deshabilitar/Detener** en la barra de herramientas del panel "Información general". Confirme la selección.
   * Para iniciar las aplicaciones lógicas detenidas seleccionadas, haga clic en **Habilitar/Iniciar** en la barra de herramientas del panel Información general.

1. Para confirmar si la operación se ha realizado correctamente o no, abra la lista **Notificaciones** (icono de campana) en la barra de herramientas principal de Azure.

<a name="disable-enable-workflows"></a>

## <a name="disable-or-enable-workflows"></a>Deshabilitación o habilitación de flujos de trabajo

Para evitar que el desencadenador se active la próxima vez que se cumpla una condición desencadenadora, deshabilite su flujo de trabajo. Únicamente se puede deshabilitar o habilitar un solo flujo de trabajo cada vez. La deshabilitación de un flujo de trabajo tiene los siguientes efectos en sus instancias:

* Azure Logic Apps continúa todas las ejecuciones en curso y pendientes hasta que finalizan. Según el volumen o el trabajo pendiente, este proceso puede tardar en completarse.

* Azure Logic Apps no crea ni ejecuta nuevas instancias de flujo de trabajo.

* El desencadenador no se activará la próxima vez que se cumplan sus condiciones. Sin embargo, el estado del desencadenador recuerda el punto en el que se ha deshabilitado el flujo de trabajo. Por lo tanto, si vuelve a habilitar el flujo de trabajo, el desencadenador se activará para todos los elementos sin procesar desde la última ejecución.

  Para evitar que el desencadenador se active con elementos no procesados desde la última ejecución, borre el estado del desencadenador antes de reactivar el flujo de trabajo:

  1. En el flujo de trabajo, edite cualquier parte de su desencadenador.
  1. Guarde los cambios. Este paso restablece el estado actual del desencadenador.
  1. [Vuelva a activar el flujo de trabajo](#disable-enable-workflows).

> [!NOTE]
> Las operaciones de deshabilitación de flujos de trabajo y detención de aplicaciones lógicas tienen efectos diferentes. Para obtener más información, consulte [Consideraciones para detener aplicaciones lógicas](#considerations-stop-logic-apps).

<a name="disable-workflow"></a>

### <a name="disable-workflow"></a>Deshabilitación de un flujo de trabajo

1. En el menú de la aplicación lógica, en **Flujos de trabajo**, seleccione **Flujos de trabajo**. En la columna de casillas de verificación, seleccione el flujo de trabajo que quiera deshabilitar.

1. En la barra de herramientas del panel **Flujos de trabajo**, seleccione **Deshabilitar**.

1. Para confirmar si la operación se ha realizado correctamente o no, abra la lista **Notificaciones** (icono de campana) en la barra de herramientas principal de Azure.

<a name="enable-workflow"></a>

### <a name="enable-workflow"></a>Habilitación de un flujo de trabajo

1. En el menú de la aplicación lógica, en **Flujos de trabajo**, seleccione **Flujos de trabajo**. En la columna de casillas de verificación, seleccione el flujo de trabajo que quiera habilitar.

1. En la barra de herramientas del panel **Flujos de trabajo**, seleccione **Habilitar**.

1. Para confirmar si la operación se ha realizado correctamente o no, abra la lista **Notificaciones** (icono de campana) en la barra de herramientas principal de Azure.

<a name="delete"></a>

## <a name="delete-logic-apps-or-workflows"></a>Eliminación de aplicaciones lógicas o flujos de trabajo

Puede [eliminar una sola aplicación lógica o varias a la vez](#delete-logic-apps). Su aplicación lógica basada en un solo inquilino puede incluir varios flujos de trabajo, por lo que puede eliminar toda la aplicación lógica o [solo flujos de trabajo](#delete-workflows).

<a name="delete-logic-apps"></a>

### <a name="delete-logic-apps"></a>Eliminación de aplicaciones lógicas

Al eliminar una aplicación lógica, se cancelan de inmediato las ejecuciones en curso y pendientes, pero no se ejecutan tareas de limpieza en el almacenamiento que usa la aplicación.

1. En el cuadro de búsqueda principal de Azure Portal, escriba `logic apps` y seleccione **Aplicaciones lógicas**.

1. En la lista **Aplicaciones lógicas**, en la columna de casillas, seleccione una o varias aplicaciones lógicas que quiera eliminar. En la barra de herramientas, seleccione **Eliminar**.

1. Cuando aparezca el cuadro de confirmación, escriba `yes` y seleccione **Eliminar**.

1. Para confirmar si la operación se ha realizado correctamente o no, abra la lista **Notificaciones** (icono de campana) en la barra de herramientas principal de Azure.

<a name="delete-workflows"></a>

### <a name="delete-workflows"></a>Eliminación de flujos de trabajo

La detección de un flujo de trabajo tiene los siguientes efectos en sus instancias:

* Azure Logic Apps cancela de inmediato las ejecuciones en curso y pendientes, pero no ejecuta tareas de limpieza en el almacenamiento que usa la aplicación.

* Azure Logic Apps no crea ni ejecuta nuevas instancias de flujo de trabajo.

* Si elimina un flujo de trabajo y, luego, vuelve a crear el mismo flujo de trabajo, el flujo de trabajo recreado no tendrá los mismos metadatos que el flujo de trabajo eliminado. Tiene que volver a guardar todo flujo de trabajo que haya llamado al flujo de trabajo eliminado. De este modo, el autor de la llamada obtiene la información correcta para el flujo de trabajo recreado. De lo contrario, las llamadas al flujo de trabajo recreado producirán un error `Unauthorized`. Este comportamiento también se aplica a los flujos de trabajo que usan artefactos en cuentas de integración y a flujos de trabajo que llaman a funciones de Azure.

1. En Azure Portal, busque y abra la aplicación lógica.

1. En el menú de la aplicación lógica, en **Flujos de trabajo**, seleccione **Flujos de trabajo**. En la columna de casillas de verificación, seleccione los flujos de trabajo que quiera eliminar.

1. En la barra de herramientas, seleccione **Eliminar**.

1. Para confirmar si la operación se ha realizado correctamente o no, abra la lista **Notificaciones** (icono de campana) en la barra de herramientas principal de Azure.

<a name="recover-deleted"></a>

## <a name="recover-deleted-logic-apps"></a>Recuperación de aplicaciones lógicas eliminadas

Si usa el control de código fuente, puede volver a implementar sin problemas un recurso **Logic Apps (estándar)** eliminado en Azure Logic Apps de un solo inquilino. Sin embargo, si no usa el control de código fuente, pruebe los pasos siguientes para recuperar la aplicación lógica eliminada.

> [!NOTE]
> Antes de intentar recuperar la aplicación lógica eliminada, revise estas consideraciones:
>
> * Solo se pueden recuperar los recursos **Logic Apps (estándar)** eliminados que usan el plan de hospedaje **Flujo de trabajo estándar**. 
> No se pueden recuperar recursos **Logic Apps (consumo)** eliminados.
>
> * Si el flujo de trabajo comienza con el desencadenador de solicitud, la dirección URL de devolución de llamada de la aplicación lógica recuperada difiere de la dirección URL de la aplicación lógica eliminada.
>
> * El historial de ejecución de la aplicación lógica eliminada no está disponible en la aplicación lógica recuperada.

1. Confirme que la cuenta de almacenamiento de la aplicación lógica todavía existe. Si se eliminó la cuenta de almacenamiento, primero debe [recuperar la cuenta de almacenamiento eliminada](../storage/common/storage-account-recover.md).

1. En el menú de la cuenta de almacenamiento, en **Seguridad y redes**, seleccione **Claves de acceso**.

1. En la página **Claves de acceso**, copie la cadena de conexión principal de la cuenta y guárdela para su uso posterior; por ejemplo:

   `DefaultEndpointsProtocol=https;AccountName=<storageaccountname>;AccountKey=<accesskey>;EndpointSuffix=core.windows.net`

1. En el menú de la cuenta de almacenamiento, en **Almacenamiento de datos**, seleccione **Recursos compartidos de archivos**, copie el nombre del recurso compartido de archivos asociado a la aplicación lógica y guárdelo para su uso posterior.

1. Cree un nuevo recurso **Logic Apps (estándar)** con el mismo plan de hospedaje y el mismo plan de tarifa. Puede usar un nuevo nombre o reutilizar el nombre de la aplicación lógica eliminada.

1. Antes de continuar, detenga la aplicación lógica. En el menú de la aplicación lógica, seleccione **Información general**. En la barra de herramientas de la página **Información general**, seleccione **Detener**.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Configuración**.

1. En la página **Configuración**, actualice los siguientes valores de configuración de la aplicación y recuerde guardar los cambios cuando haya terminado.

   | Configuración de aplicación | Valor de reemplazo |
   |-------------|-------------------|
   | `AzureWebJobsStorage` | Reemplace el valor existente por la cadena de conexión copiada anteriormente de la cuenta de almacenamiento. |
   | `WEBSITE_CONTENTAZUREFILECONNECTIONSTRING` | Reemplace el valor existente por la cadena copiada anteriormente de la cuenta de almacenamiento. |
   | `WEBSITE_CONTENTSHARE` | Reemplace el valor existente por el nombre del recurso compartido de archivos copiado anteriormente. |
   |||

1. En el menú de la aplicación lógica, en **Flujos de trabajo**, seleccione **Conexiones**.

1. Abra cada conexión y, en **Configuración**, seleccione **Directivas de acceso**.

1. Elimine la directiva de acceso de la aplicación lógica eliminada y, a continuación, agregue una nueva directiva de acceso para la aplicación lógica de reemplazo.

1. Vuelva a la página **Configuración** de la aplicación lógica y agregue cualquier configuración personalizada que existiera en la aplicación lógica eliminada.

1. Cuando haya terminado, reinicie la aplicación lógica.

<a name="troubleshoot"></a>

## <a name="troubleshoot-problems-and-errors"></a>Solución de problemas y errores

<a name="missing-triggers-actions"></a>

### <a name="new-triggers-and-actions-are-missing-from-the-designer-picker-for-previously-created-workflows"></a>Faltan nuevos desencadenadores y acciones en el selector del diseñador para flujos de trabajo creados previamente

Azure Logic Apps de un solo inquilino admite acciones integradas para las operaciones de Azure Functions, las operaciones de Liquid y las operaciones de XML, como **Validación XML** y **Transformar XML**. Sin embargo, en el caso de las aplicaciones lógicas creadas con anterioridad, es posible que estas acciones no aparezcan en el diseñador para que las pueda seleccionar si la aplicación lógica usa una versión obsoleta del paquete de extensión `Microsoft.Azure.Functions.ExtensionBundle.Workflows`.

Para corregir este problema, siga estos pasos para eliminar la versión obsoleta de modo que el paquete de extensión pueda actualizarse automáticamente a la versión más reciente.

> [!NOTE]
> Esta solución específica solo se aplica a los recursos **Logic Apps (estándar)** que se crean con Azure Portal, no a las aplicaciones lógicas que se crean e implementan con Visual Studio Code y la extensión Azure Logic Apps (estándar). Vea [Faltan desencadenadores y acciones en el diseñador en Visual Studio Code](create-single-tenant-workflows-visual-studio-code.md#missing-triggers-actions).

1. Detenga la aplicación lógica en Azure Portal.

   1. En el menú de la aplicación lógica, seleccione **Introducción**.

   1. En la barra de herramientas del panel **Información general**, seleccione **Detener**.

1. En el menú de la aplicación lógica, en **Herramientas de desarrollo**, seleccione **Herramientas avanzadas**.

1. En el panel **Herramientas avanzadas**, seleccione **Ir**, que abre el entorno de Kudu para la aplicación lógica.

1. En la barra de herramientas de Kudu, abra el menú **Consola de depuración** y seleccione **CMD**.

   Se abre una ventana de consola, por lo que puede desplazarse a la carpeta de la agrupación mediante el símbolo del sistema. O bien, puede examinar la estructura de directorios que aparece sobre la ventana de la consola.

1. Busque la carpeta siguiente, que contiene las carpetas con versión para el paquete existente:

   `...\home\data\Functions\ExtensionBundles\Microsoft.Azure.Functions.ExtensionBundle.Workflows`

1. Elimine la carpeta de la versión del paquete existente. En la ventana de la consola, puede ejecutar este comando donde se reemplaza `{bundle-version}` por la versión existente:

   `rm -rf {bundle-version}`

   Por ejemplo: `rm -rf 1.1.3`

   > [!TIP]
   > Si recibe un error, como "Permiso denegado" o "Archivo en uso", actualice la página en el explorador y vuelva a intentar los pasos anteriores hasta que se elimine la carpeta.

1. En Azure Portal, vuelva a la página **Información general** de la aplicación lógica y seleccione **Reiniciar**.

   El portal obtiene y utiliza automáticamente el paquete más reciente.

## <a name="next-steps"></a>Pasos siguientes

Nos gustaría conocer sus experiencias con este escenario.

* En caso de errores o problemas, [dirija los problemas a GitHub](https://github.com/Azure/logicapps/issues).
* Si tiene preguntas, solicitudes, ideas y comentarios, [use este formulario de comentarios](https://aka.ms/lafeedback).
