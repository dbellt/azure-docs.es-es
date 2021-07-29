---
title: 'Administración de clientes potenciales con un punto de conexión HTTPS: marketplace comercial de Microsoft'
description: Aprenda a usar Power Automate y un punto de conexión HTTPS para administrar clientes potenciales de Microsoft AppSource y Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 05/21/2021
ms.openlocfilehash: f4ef8d7d29b65af4876912e6dbbd3827693737c6
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110450428"
---
# <a name="use-an-https-endpoint-to-manage-commercial-marketplace-leads"></a>Uso de un punto de conexión HTTPS para administrar clientes potenciales de marketplace comercial

Si el sistema de administración de relaciones con clientes (CRM) no se admite explícitamente en el Centro de partners para recibir clientes potenciales de Azure Marketplace y Microsoft AppSource, puede usar un punto de conexión HTTPS en [Power Automate](https://powerapps.microsoft.com/automate-processes/) para administrar estos clientes potenciales. Con un punto de conexión HTTPS, los clientes potenciales de marketplace comercial se pueden enviar como notificación por correo electrónico o pueden escribirse en un sistema CRM admitido por Power Automate.

En este artículo se explica cómo crear un nuevo flujo en Power Automate para generar la dirección URL de HTTP POST que usará para configurar clientes potenciales en el Centro de partners. También se incluyen los pasos para probar el flujo con [Postman](https://www.getpostman.com/downloads/).

>[!NOTE]
>El conector de Power Automate usado en estas instrucciones requiere una suscripción de pago de Power Automate. Asegúrese de que tiene una cuenta para ello antes de configurar este flujo.

## <a name="create-a-flow-by-using-power-automate"></a>Creación de un flujo con Power Automate

1. Abra la página web de [Power Automate](https://flow.microsoft.com/). Seleccione **Iniciar sesión**. Si aún no tiene una cuenta, seleccione **Regístrese gratis** para crear una.

1. Inicie sesión, seleccione **Mis flujos** y cambie el entorno de **Microsoft (valor predeterminado)** a su entorno de Dataverse (CRM).

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-https/my-flows-automated.png" alt-text="Muestra cómo iniciar sesión en &quot;Mis flujos&quot;.":::

1. En **+ Nuevo,** seleccione **+ Instant cloud flow** (Flujo de nube instantáneo).

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-https/https-my-flows-create-from-blank.png" alt-text="Muestra cómo seleccionar Mis flujos, seguido de Flujo de nube instantáneo.":::

1. Asigne un nombre al flujo y, en **Elija cómo se desencadena este flujo**, seleccione **Cuando se reciba una solicitud HTTP**.

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-https/https-my-flows-pick-request-trigger.png" alt-text="Muestra cómo crear el botón Omitir de la ventana del flujo automatizado.":::

1. Haga clic en el paso del flujo para expandirlo.

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-https/expand-flow-step.png" alt-text="Muestra cómo expandir el paso del flujo.":::

1. Use uno de los métodos siguientes para configurar el **esquema JSON del cuerpo de la solicitud**:

    - Copie el esquema JSON en el cuadro de texto **Esquema JSON de cuerpo de solicitud**.
    - Seleccione **Usar una carga de ejemplo para generar el esquema**. En el cuadro de texto **Escriba o pegue una carga JSON de ejemplo**, pegue el ejemplo JSON. Seleccione **Listo** para crear el esquema.

    **Esquema JSON**

    ```JSON
    {
      "$schema": "https://json-schema.org/draft-04/schema#",
      "definitions": {},
      "id": "http://example.com/example.json",
      "properties": {
        "ActionCode": {
          "id": "/properties/ActionCode",
          "type": "string"
        },
        "OfferTitle": {
          "id": "/properties/OfferTitle",
          "type": "string"
        },
        "LeadSource": {
          "id": "/properties/LeadSource",
          "type": "string"
        },
        "Description": {
          "id": "/properties/Description",
          "type": "string"
        },
        "UserDetails": {
          "id": "/properties/UserDetails",
          "properties": {
            "Company": {
              "id": "/properties/UserDetails/properties/Company",
              "type": "string"
            },
            "Country": {
              "id": "/properties/UserDetails/properties/Country",
              "type": "string"
            },
            "Email": {
              "id": "/properties/UserDetails/properties/Email",
              "type": "string"
            },
            "FirstName": {
              "id": "/properties/UserDetails/properties/FirstName",
              "type": "string"
            },
            "LastName": {
              "id": "/properties/UserDetails/properties/LastName",
              "type": "string"
            },
            "Phone": {
              "id": "/properties/UserDetails/properties/Phone",
              "type": "string"
            },
            "Title": {
              "id": "/properties/UserDetails/properties/Title",
              "type": "string"
            }
          },
          "type": "object"
        }
      },
      "type": "object"
    }
    ```

    **Ejemplo JSON**
    
    ```json
    {
      "UserDetails": {
        "FirstName": "Some",
        "LastName": "One",
        "Email": "someone@contoso.com",
        "Phone": "16175555555",
        "Country": "USA",
        "Company": "Contoso",
        "Title": "Esquire"
     },
      "LeadSource": "AzureMarketplace",
      "ActionCode": "INS",
      "OfferTitle": "Test Microsoft",
      "Description": "Test run through Power Automate"
    }
    ```

>[!NOTE]
>En este punto de la configuración, puede seleccionar conectarse a un sistema CRM o configurar una notificación por correo electrónico. Siga las instrucciones restantes en función de su elección.

### <a name="connect-to-a-crm-system"></a>Conexión a un sistema CRM

1. Seleccione **+ New step**(+ Nuevo paso).
1. Busque y seleccione un sistema CRM en el cuadro **Buscar todos los conectores y acciones**. La siguiente pantalla muestra **Crear un nuevo registro (Microsoft Dataverse)** como ejemplo.

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-https/create-new-record.png" alt-text="Muestra cómo crear un nuevo registro.":::

1. Seleccione la **pestaña Acciones** con la acción para **Agregar una nueva fila**.

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-https/select-create-new-record.png" alt-text="Muestra cómo seleccionar &quot;Crear un nuevo registro&quot;.":::

1. Seleccione **cliente potencial** en la lista desplegable **Nombre de la tabla**.

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-https/select-leads.png" alt-text="Muestra cómo seleccionar clientes potenciales.":::

1. Power Automate muestra un formulario para proporcionar información sobre clientes potenciales. Puede asignar elementos de la solicitud de entrada y elige agregar contenido dinámico. La pantalla siguiente muestra **OfferTitle** como ejemplo.

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-https/add-dynamic-content.png" alt-text="Muestra cómo agregar contenido dinámico.":::

1. Asigne los campos que quiera y seleccione **Guardar** para guardar el flujo. Se crea una dirección URL de HTTP POST, y se puede acceder a ella en la ventana **Cuando se recibe una solicitud HTTP**. Copie esta dirección URL mediante el control de copia, que se encuentra a la derecha de la dirección URL de HTTP POST. Es importante utilizar el control de copia para no omitir ninguna parte de la dirección URL completa. Guarde esta dirección URL, ya que la necesitará al configurar la administración de clientes potenciales en el portal de publicación.

    :::image type="content" source="media/commercial-marketplace-lead-management-instructions-https/when-http-request-received.png" alt-text="Muestra Cuando se recibe una solicitud HTTP.":::

### <a name="set-up-email-notification"></a>Configuración de notificaciones de correo electrónico

1. Ahora que ha completado el esquema JSON, seleccione **+ Nuevo paso**.
1. En **Elegir una acción**, seleccione **Acciones**.
1. En la pestaña **Acciones**, seleccione **Enviar un correo electrónico (Office 365 Outlook)** .

    >[!NOTE]
    >Si desea usar otro proveedor de correo electrónico, busque y seleccione **Enviarme una notificación por correo electrónico (Correo)** como acción en su lugar.

    ![Adición de una acción de correo electrónico](./media/commercial-marketplace-lead-management-instructions-https/https-request-received-send-email.png)

1. En la ventana **Enviar un correo electrónico**, configure los siguientes campos obligatorios:

   - **Para**: especifique al menos una dirección de correo electrónico válida, adonde se enviarán los clientes potenciales.
   - **Firmante**: Power Automate le ofrece la opción de agregar contenido dinámico como, por ejemplo, **LeadSource** en la siguiente pantalla. Empiece por escribir un nombre de campo. A continuación, seleccione la lista de selección de contenido dinámico en la ventana emergente. 

        >[!NOTE] 
        > Al agregar nombres de campo, puede agregar dos puntos (:) después de cada nombre y seleccionar **Entrar** para crear una nueva fila. Una vez que haya agregado los nombres de campos, puede agregar cada parámetro asociado desde la lista desplegable dinámica.

        ![Agregar una acción de correo electrónico mediante contenido dinámico](./media/commercial-marketplace-lead-management-instructions-https/add-email-using-dynamic-content.png)

   - **Cuerpo**: en la lista de selección Contenido dinámico, agregue la información que desee en el cuerpo del correo electrónico. Por ejemplo, nombre, apellidos, correo electrónico y empresa. Cuando haya terminado de configurar la notificación por correo electrónico, tendrá un aspecto similar al ejemplo de la siguiente pantalla.


       ![Ejemplo de notificación por correo electrónico](./media/commercial-marketplace-lead-management-instructions-https/send-an-email.png)

1. Seleccione **Guardar** para finalizar el flujo. Se crea una dirección URL de HTTP POST, y se puede acceder a ella en la ventana **Cuando se recibe una solicitud HTTP**. Copie esta dirección URL mediante el control de copia, que se encuentra a la derecha de la dirección URL de HTTP POST. Es importante utilizar este control para no omitir ninguna parte de la dirección URL completa. Guarde esta dirección URL, ya que la necesitará al configurar la administración de clientes potenciales en el portal de publicación.

   ![Dirección URL de HTTP POST](./media/commercial-marketplace-lead-management-instructions-https/http-post-url.png)

### <a name="testing"></a>Prueba

Puede probar la configuración con [Postman](https://app.getpostman.com/app/download/win64). Hay disponible una descarga en línea de Postman para Windows. 

1. Inicie Postman y seleccione **Nuevo** > **Solicitud** para configurar la herramienta de prueba. 

   ![Solicitud de configuración de la herramienta de prueba](./media/commercial-marketplace-lead-management-instructions-https/postman-request.png)

1. Rellene el formulario **Guardar solicitud** y, luego, guárdelo en la carpeta que ha creado.

   ![Formulario Guardar solicitud](./media/commercial-marketplace-lead-management-instructions-https/postman-save-to-test.png)

1. Seleccione **POST** en la lista desplegable. 

   ![Probar mi flujo](./media/commercial-marketplace-lead-management-instructions-https/test-my-flow.png)

1. Pegue la dirección URL de HTTP POST del flujo que creó en Power Automate, donde dice **Escribir la dirección URL de la solicitud**.

   ![Pegue la dirección URL de HTTP POST](./media/commercial-marketplace-lead-management-instructions-https/paste-http-post-url.png)

1. Regrese a [Power Automate](https://flow.microsoft.com/). Busque el flujo que creó para enviar clientes potenciales; para ello, vaya a **Mis flujos** en la barra de menús de Power Automate. Seleccione los puntos suspensivos junto al nombre del flujo para ver más opciones y elija **Editar**.


1. Seleccione **Probar** en la esquina superior derecha, elija **Yo realizaré la acción de desencadenamiento** y luego seleccione **Probar**. Verá una indicación en la parte superior de la pantalla que indica que se ha iniciado la prueba.

   ![Opción Yo realizaré la acción de desencadenamiento](./media/commercial-marketplace-lead-management-instructions-https/test-flow-trigger-action.png)

1. Vuelva a la aplicación Postman y seleccione **Enviar**.

   ![Botón Enviar](./media/commercial-marketplace-lead-management-instructions-https/postman-send.png)

1. Vuelva al flujo y compruebe el resultado. Si todo funciona según lo previsto, verá un mensaje que indica que el flujo se ha realizado correctamente.

   ![Comprobar los resultados](./media/commercial-marketplace-lead-management-instructions-https/my-flow-check-results.png)

1. También debe haber recibido un correo electrónico. Compruebe la bandeja de entrada del correo.

    >[!NOTE] 
    >Si no ve un correo electrónico de la prueba, compruebe las carpetas de correo no deseado y spam. En la siguiente pantalla, observará solo las etiquetas de campo que agregó al configurar la notificación por correo electrónico. Si se tratase de un cliente potencial real generado a partir de su oferta, también vería la información real del contacto de cliente potencial en el cuerpo y en la línea de asunto.

   ![Correo electrónico recibido](./media/commercial-marketplace-lead-management-instructions-https/email-received.png)

## <a name="configure-your-offer-to-send-leads-to-the-https-endpoint"></a>Configuración de la oferta para enviar clientes potenciales al punto de conexión HTTPS

Cuando esté listo para configurar la información de administración de clientes potenciales de la oferta en el portal de publicación, siga estos pasos:

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard/home).

1. Seleccione la oferta y vaya a la pestaña **Configuración de la oferta**.

1. En la sección **Clientes potenciales**, seleccione **Conectar**.

    :::image type="content" source="./media/commercial-marketplace-lead-management-instructions-https/customer-leads.png" alt-text="Clientes potenciales":::

1. En la ventana emergente **Detalles de conexión**, seleccione **Punto de conexión HTTPS** como **Destino de clientes potenciales**. Pegue la dirección URL de HTTP POST del flujo que creó siguiendo los pasos anteriores en el campo **Dirección URL del punto de conexión HTTPS**.
    ![Detalles de la conexión: Correo electrónico de contacto](./media/commercial-marketplace-lead-management-instructions-https/https-connection-details.png)

1. En **Correo electrónico de contacto**, escriba las direcciones de correo electrónico de las personas de su empresa que deban recibir notificaciones por correo electrónico cuando se reciba un nuevo cliente potencial. Puede proporcionar varias direcciones de correo electrónico separadas con un punto y coma.

1. Seleccione **Aceptar**.

Para asegurarse de que se ha conectado correctamente a un destino de clientes potenciales, seleccione el botón **Validar**. Si la validación es correcta, tendrá un cliente potencial de prueba en el destino de clientes potenciales.

>[!NOTE] 
>Debe terminar de configurar el resto de las partes de la oferta y publicarla para poder recibir clientes potenciales para la oferta.

Cuando se generan clientes potenciales, Microsoft envía los clientes potenciales al flujo. Los clientes potenciales se dirigen a la dirección de correo electrónico o al sistema CRM que ha configurado.
