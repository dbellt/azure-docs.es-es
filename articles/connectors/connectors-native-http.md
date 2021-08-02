---
title: Llamada a puntos de conexión de servicio mediante HTTP o HTTPS
description: Envío de solicitudes HTTP o HTTPS salientes a puntos de conexión de servicio desde Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm, azla
ms.topic: how-to
ms.date: 05/25/2021
tags: connectors
ms.openlocfilehash: 10c946010fa3caba14130c3c7055c711323ad93c
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110498298"
---
# <a name="call-service-endpoints-over-http-or-https-from-azure-logic-apps"></a>Llamada a puntos de conexión de servicio mediante HTTP o HTTPS desde Azure Logic Apps

Con [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y el desencadenador o la acción HTTP integrados, puede crear tareas y flujos de trabajo automatizados que envían solicitudes salientes a puntos de conexión de servicio mediante HTTP o HTTPS. En cambio, si desea recibir y responder llamadas HTTP o HTTPS entrantes, use el [desencadenador de solicitud o la acción de respuesta](../connectors/connectors-native-reqres.md) integrados.

Por ejemplo, para supervisar el punto de conexión de servicio de su sitio web puede comprobarlo según una programación específica. Cuando se produce un evento especificado en ese punto de conexión, por ejemplo, que su sitio web deje de funcionar, el evento desencadena el flujo de trabajo de la aplicación lógica y ejecuta las acciones de ese flujo de trabajo.

* [Agregue el desencadenador HTTP](#http-trigger) como primer paso del flujo de trabajo para comprobar o *sondear* un punto de conexión según una programación recurrente. Cada vez que el desencadenador comprueba el punto de conexión, el desencadenador llama a o envía una *solicitud* al punto de conexión. La respuesta del punto de conexión determina si el flujo de trabajo de la aplicación lógica se ejecuta. El desencadenador pasa todo el contenido de la respuesta del punto de conexión a las acciones en la aplicación lógica.

* Para llamar a un punto de conexión desde cualquier parte del flujo de trabajo, [agregue la acción HTTP](#http-action). La respuesta del punto de conexión determina cómo se ejecutan las acciones restantes de su flujo de trabajo.

En este artículo se muestra cómo usar el desencadenador HTTP y la acción HTTP para que la aplicación lógica pueda enviar llamadas salientes a otros servicios y sistemas.

Para obtener información sobre el cifrado, la seguridad y la autorización de llamadas entrantes para la aplicación lógica, como la [Seguridad de la capa de transporte (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security), conocida anteriormente como Capa de sockets seguros (SSL), certificados autofirmados o la [Autenticación abierta de Azure Active Directory Azure (Azure AD OAuth)](../active-directory/develop/index.yml), consulte [Acceso seguro y datos - Acceso para llamadas salientes a otros servicios y sistemas](../logic-apps/logic-apps-securing-a-logic-app.md#secure-outbound-requests).

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* La dirección URL del punto de conexión de destino al que quiere llamar.

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md). Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

* La aplicación lógica desde la que quiere llamar al punto de conexión de destino. Para comenzar con el desencadenador HTTP, [cree una aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para usar la acción de HTTP, inicie la aplicación lógica con el desencadenador que quiera. En este ejemplo se usa el desencadenador HTTP como primer paso.

<a name="http-trigger"></a>

## <a name="add-an-http-trigger"></a>Agregar un desencadenador HTTP

Este desencadenador integrado realiza una llamada HTTP a la dirección URL especificada para un punto de conexión y devuelve una respuesta.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Abra la aplicación lógica en blanco en el diseñador de aplicación lógica.

1. En el cuadro de búsqueda del diseñador, seleccione **Integrado**. En el cuadro de búsqueda, escriba `http` como filtro. En la lista **Desencadenadores**, seleccione el desencadenador **HTTP**.

   ![Selección del desencadenador HTTP](./media/connectors-native-http/select-http-trigger.png)

   En este ejemplo se cambia el nombre del desencadenador a "desencadenador HTTP" para que el paso tenga un nombre más descriptivo. En el ejemplo, más adelante se agrega una acción HTTP, y ambos nombres deben ser únicos.

1. Proporcione los valores de los [parámetros del desencadenador HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-trigger) que quiere incluir en la llamada al punto de conexión de destino. Establezca la periodicidad con la que quiere que el desencadenador compruebe el punto de conexión de destino.

   ![Introducción de los parámetros de desencadenador HTTP](./media/connectors-native-http/http-trigger-parameters.png)

   Si selecciona un tipo de autenticación distinto de **Ninguno**, la configuración de autenticación difiere en función de la selección. Para obtener más información sobre los tipos de autenticación disponibles para HTTP, consulte estos temas:

   * [Incorporación de la autenticación en las llamadas salientes](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound)
   * [Autenticar el acceso a los recursos de Azure mediante identidades administradas](../logic-apps/create-managed-service-identity.md)

1. Para agregar otros parámetros disponibles, abra la lista **Agregar nuevo parámetro** y seleccione los parámetros que quiera.

1. Continúe creando el flujo de trabajo de la aplicación lógica con acciones que se ejecuten cuando se activa el desencadenador.

1. Cuando haya terminado, recuerde guardar la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

<a name="http-action"></a>

## <a name="add-an-http-action"></a>Adición de una acción HTTP

Esta acción integrada realiza una llamada HTTP a la dirección URL especificada para un punto de conexión y devuelve una respuesta.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Abra la aplicación lógica en el Diseñador de aplicaciones lógicas.

   En este ejemplo se usa el desencadenador HTTP como primer paso.

1. En el paso en el que quiera agregar la acción HTTP, seleccione **Nuevo paso**.

   Para agregar una acción entre un paso y otro, mueva el puntero sobre la flecha entre ellos. Seleccione el signo más ( **+** ) que aparece y, luego, seleccione **Agregar una acción**.

1. En **Elegir una acción**, seleccione **Integrado**. En el cuadro de búsqueda, escriba `http` como filtro. En la lista **Acciones**, seleccione la acción **HTTP**.

   ![Selección de la acción de HTTP](./media/connectors-native-http/select-http-action.png)

   En este ejemplo se cambia el nombre de la acción por "acción HTTP" para que el paso tenga un nombre más descriptivo.

1. Proporcione los valores de los [parámetros de la acción HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-action) que quiere incluir en la llamada al punto de conexión de destino.

   ![Introducción de los parámetros de acción de HTTP](./media/connectors-native-http/http-action-parameters.png)

   Si selecciona un tipo de autenticación distinto de **Ninguno**, la configuración de autenticación difiere en función de la selección. Para obtener más información sobre los tipos de autenticación disponibles para HTTP, consulte estos temas:

   * [Incorporación de la autenticación en las llamadas salientes](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound)
   * [Autenticar el acceso a los recursos de Azure mediante identidades administradas](../logic-apps/create-managed-service-identity.md)

1. Para agregar otros parámetros disponibles, abra la lista **Agregar nuevo parámetro** y seleccione los parámetros que quiera.

1. Cuando haya terminado, recuerde guardar la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

## <a name="trigger-and-action-outputs"></a>Salidas de los desencadenadores y las acciones

Aquí tiene más información acerca de las salidas de un desencadenador o una acción HTTP, que devuelve esta información:

| Propiedad | Tipo | Descripción |
|----------|------|-------------|
| `headers` | Objeto JSON | Encabezados de la solicitud |
| `body` | Objeto JSON | Objeto con el contenido del cuerpo de la solicitud |
| `status code` | Entero | Código de estado de la solicitud |
|||

| status code | Descripción |
|-------------|-------------|
| 200 | Aceptar |
| 202 | Accepted |
| 400 | Solicitud incorrecta |
| 401 | No autorizado |
| 403 | Prohibido |
| 404 | No encontrado |
| 500 | Error interno del servidor Error desconocido. |
|||

<a name="single-tenant-authentication"></a>

## <a name="authentication-for-single-tenant-environment"></a>Autenticación para un entorno de inquilino único

Si tiene un recurso de **Aplicación lógica (estándar)** en Azure Logic Apps de un solo inquilino y desea usar una operación HTTP con cualquiera de los siguientes tipos de autenticación, asegúrese de completar los pasos de configuración adicionales para el tipo de autenticación correspondiente. De lo contrario, se produce un error en la llamada.

* [Certificado TSL/SSL](#tsl-ssl-certificate-authentication): agregue la configuración de la aplicación, `WEBSITE_LOAD_ROOT_CERTIFICATES`, y proporcione la huella digital del certificado TSL/SSL.

* [Certificado de cliente o Azure Active Directory Open Authentication (Azure AD OAuth) con el tipo de credencial "Certificado"](#client-certificate-authentication): agregue la configuración de la aplicación, `WEBSITE_LOAD_USER_PROFILE`, y establezca el valor en `1`.

<a name="tsl-ssl-certificate-authentication"></a>

### <a name="tslssl-certificate-authentication"></a>Autenticación con certificados TLS/SSL

1. En la configuración de la aplicación del recurso de aplicación lógica, [agregue o actualice la configuración de la aplicación](../logic-apps/edit-app-settings-host-settings.md#manage-app-settings), `WEBSITE_LOAD_ROOT_CERTIFICATES`.

1. Para el valor de configuración, proporcione la huella digital del certificado TLS/SSL como certificado raíz de confianza.

   `"WEBSITE_LOAD_ROOT_CERTIFICATES": "<thumbprint-for-TSL/SSL-certificate>"`

Por ejemplo, si trabaja en Visual Studio Code, siga estos pasos:

1. Abra el archivo **local.settings.json** del proyecto de aplicación lógica.

1. En el objeto JSON `Values`, agregue o actualice la configuración `WEBSITE_LOAD_ROOT_CERTIFICATES`:

   ```json
   {
      "IsEncrypted": false,
      "Values": {
         <...>
         "AzureWebJobsStorage": "UseDevelopmentStorage=true",
         "WEBSITE_LOAD_ROOT_CERTIFICATES": "<thumbprint-for-TSL/SSL-certificate>",
         <...>
      }
   }
   ```

Para más información, revise la siguiente documentación:

* [Edición de la configuración de host y aplicación para aplicaciones lógicas en Azure Logic Apps de inquilino único](../logic-apps/edit-app-settings-host-settings.md#manage-app-settings)
* [Certificados y App Service aislado](../app-service/environment/certificates.md#private-client-certificate)

<a name="client-certificate-authentication"></a>

### <a name="client-certificate-or-azure-ad-oauth-with-certificate-credential-type-authentication"></a>Certificado de cliente o Azure AD OAuth con autenticación de tipo de credencial "Certificado"

1. En la configuración de la aplicación del recurso de aplicación lógica, [agregue o actualice la configuración de la aplicación](../logic-apps/edit-app-settings-host-settings.md#manage-app-settings), `WEBSITE_LOAD_USER_PROFILE`.

1. Para el valor de configuración, especifique `1`.

   `"WEBSITE_LOAD_USER_PROFILE": "1"`

Por ejemplo, si trabaja en Visual Studio Code, siga estos pasos:

1. Abra el archivo **local.settings.json** del proyecto de aplicación lógica.

1. En el objeto JSON `Values`, agregue o actualice la configuración `WEBSITE_LOAD_USER_PROFILE`:

   ```json
   {
      "IsEncrypted": false,
      "Values": {
         <...>
         "AzureWebJobsStorage": "UseDevelopmentStorage=true",
         "WEBSITE_LOAD_USER_PROFILE": "1",
         <...>
      }
   }
   ```

Para más información, revise la siguiente documentación:

* [Edición de la configuración de host y aplicación para aplicaciones lógicas en Azure Logic Apps de inquilino único](../logic-apps/edit-app-settings-host-settings.md#manage-app-settings)
* [Certificados y App Service aislado](../app-service/environment/certificates.md#private-client-certificate)

## <a name="content-with-multipartform-data-type"></a>Contenido con el tipo multipart/form-data

Para controlar el contenido que tiene el tipo `multipart/form-data` en las solicitudes HTTP, puede agregar un objeto JSON que incluya los atributos `$content-type` y `$multipart` al cuerpo de la solicitud HTTP con este formato.

```json
"body": {
   "$content-type": "multipart/form-data",
   "$multipart": [
      {
         "body": "<output-from-trigger-or-previous-action>",
         "headers": {
            "Content-Disposition": "form-data; name=file; filename=<file-name>"
         }
      }
   ]
}
```

Por ejemplo, supongamos que tiene una aplicación lógica que envía una solicitud HTTP POST para un archivo de Excel a un sitio web mediante la API de ese sitio, que admite el tipo `multipart/form-data`. Este es el aspecto de esta acción:

![Datos de formulario de varias partes](./media/connectors-native-http/http-action-multipart.png)

Este es el mismo ejemplo que muestra la definición de JSON de la acción HTTP en la definición de flujo de trabajo subyacente:

```json
"HTTP_action": {
   "inputs": {
      "body": {
         "$content-type": "multipart/form-data",
         "$multipart": [
            {
               "body": "@trigger()",
               "headers": {
                  "Content-Disposition": "form-data; name=file; filename=myExcelFile.xlsx"
               }
            }
         ]
      },
      "method": "POST",
      "uri": "https://finance.contoso.com"
   },
   "runAfter": {},
   "type": "Http"
}
```

## <a name="content-with-applicationx-www-form-urlencoded-type"></a>Contenido con tipo application/x-www-form-urlencoded

Para proporcionar datos form-urlencoded en el cuerpo de una solicitud HTTP, debe especificar que los datos tienen el tipo de contenido `application/x-www-form-urlencoded`. En la acción o el desencadenador HTTP, agregue el encabezado `content-type`. Establezca el valor de encabezado en `application/x-www-form-urlencoded`.

Por ejemplo, imagine que tiene una aplicación lógica que envía una solicitud HTTP POST a un sitio web que admite el tipo `application/x-www-form-urlencoded`. Este es el aspecto de esta acción:

![Captura de pantalla que muestra una solicitud HTTP con el encabezado "content-type" establecido en "application/x-www-form-urlencoded"](./media/connectors-native-http/http-action-urlencoded.png)

<a name="asynchronous-pattern"></a>

## <a name="asynchronous-request-response-behavior"></a>Comportamiento asincrónico de la solicitud-respuesta

De forma predeterminada, todas las acciones basadas en HTTP de Azure Logic Apps siguen el [modelo asincrónico de operación](/azure/architecture/patterns/async-request-reply) estándar. Este patrón especifica que, después de que una acción HTTP llame a o envíe una solicitud a un punto de conexión, servicio, sistema o API, el receptor devolverá inmediatamente una respuesta ["202 ACCEPTED"](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.2.3). Este código confirma que el receptor aceptó la solicitud, pero no ha finalizado el procesamiento. La respuesta puede incluir un encabezado `location` que especifica la dirección URL y un identificador de actualización que el autor de la llamada puede usar para sondear o comprobar el estado de la solicitud asincrónica hasta que el receptor detenga el procesamiento y devuelva una respuesta de operación correcta ["200 OK"](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.2.1) u otra respuesta que no sea 202. Sin embargo, el autor de la llamada no tiene que esperar a que la solicitud finalice el procesamiento y puede continuar ejecutando la siguiente acción. Para más información, consulte [Diseño de la comunicación entre servicios para microservicios](/azure/architecture/microservices/design/interservice-communication#synchronous-versus-asynchronous-messaging).

* En el Diseñador de aplicación lógica, la acción HTTP (no el desencadenador) tiene una opción de configuración **Modelo asincrónico**, que está habilitada de forma predeterminada. Esta configuración especifica que el autor de la llamada no espera a que finalice el procesamiento y puede pasar a la siguiente acción, pero continúa comprobando el estado hasta que el procesamiento se detiene. Si está deshabilitada, esta configuración especifica que el autor de la llamada espera a que finalice el procesamiento antes de pasar a la siguiente acción.

  Para buscar esta configuración, siga estos pasos:

  1. En la barra de título de la acción HTTP, seleccione el botón de puntos suspensivos ( **...** ), que abre la configuración de la acción.

  1. Busque la opción de configuración **Modelo asincrónico**.

     ![Opción de configuración "Modelo asincrónico"](./media/connectors-native-http/asynchronous-pattern-setting.png)

* La definición de la notación de objetos JavaScript (JSON) subyacente de la acción HTTP sigue implícitamente el modelo asincrónico de operación.

<a name="disable-asynchronous-operations"></a>

## <a name="disable-asynchronous-operations"></a>Deshabilitación de las operaciones asincrónicas

En ocasiones, es posible que desee el comportamiento asincrónico de la acción HTTP en escenarios específicos, por ejemplo, si prefiere:

* [Evitar los tiempos de espera de HTTP para las tareas de ejecución prolongada](#avoid-http-timeouts)
* [Deshabilitar la comprobación de encabezados de ubicación](#disable-location-header-check)

<a name="turn-off-asynchronous-pattern-setting"></a>

### <a name="turn-off-asynchronous-pattern-setting"></a>Desactivación de la opción de configuración **Modelo asincrónico**

1. En el Diseñador de aplicación lógica, en la barra de título de la acción HTTP, seleccione el botón de puntos suspensivos ( **...** ), que abre la configuración de la acción.

1. Busque la opción de configuración **Modelo asincrónico**, establézcala en **Desactivada** si está habilitada y seleccione **Listo**.

   ![Deshabilitación de la opción de configuración "Modelo asincrónico"](./media/connectors-native-http/disable-asynchronous-pattern-setting.png)

<a name="add-disable-async-pattern-option"></a>

### <a name="disable-asynchronous-pattern-in-actions-json-definition"></a>Deshabilitación del modelo asincrónico en la definición JSON de la acción

En la definición de JSON subyacente de la acción HTTP, [agregar la opción de operación de `"DisableAsyncPattern"`](../logic-apps/logic-apps-workflow-actions-triggers.md#operation-options) a la definición de la acción para que la acción siga el modelo sincrónico de operación en su lugar. Para más información, consulte también [Ejecución de acciones en un modelo sincrónico de operación](../logic-apps/logic-apps-workflow-actions-triggers.md#disable-asynchronous-pattern).

<a name="avoid-http-timeouts"></a>

## <a name="avoid-http-timeouts-for-long-running-tasks"></a>Evitar los tiempos de espera de HTTP para las tareas de ejecución prolongada

Las solicitudes HTTP tienen un [límite de tiempo de espera](../logic-apps/logic-apps-limits-and-config.md#http-limits). Si tiene una acción HTTP de ejecución prolongada que agota el tiempo de espera debido a este límite, tiene estas opciones:

* [Deshabilite el modelo asincrónico de operación de la acción HTTP](#disable-asynchronous-operations) para que la acción no sondee continuamente ni compruebe el estado de la solicitud. En su lugar, la acción espera a que el receptor responda con el estado y los resultados una vez finalizado el procesamiento de la solicitud.

* Reemplace la acción HTTP por la [acción de webhook HTTP](../connectors/connectors-native-webhook.md), que espera a que el receptor responda con el estado y los resultados después de que la solicitud finalice el procesamiento.

<a name="disable-location-header-check"></a>

## <a name="disable-checking-location-headers"></a>Deshabilitación de la comprobación de encabezados de ubicación

Algunos puntos de conexión, servicios, sistemas o API devuelven una respuesta `202 ACCEPTED` que no tiene el encabezado `location`. Para evitar que una acción HTTP compruebe continuamente el estado de la solicitud cuando no existe el encabezado `location`, puede tener estas opciones:

* [Deshabilite el modelo asincrónico de operación de la acción HTTP](#disable-asynchronous-operations) para que la acción no sondee continuamente ni compruebe el estado de la solicitud. En su lugar, la acción espera a que el receptor responda con el estado y los resultados una vez finalizado el procesamiento de la solicitud.

* Reemplace la acción HTTP por la [acción de webhook HTTP](../connectors/connectors-native-webhook.md), que espera a que el receptor responda con el estado y los resultados después de que la solicitud finalice el procesamiento.

## <a name="known-issues"></a>Problemas conocidos

<a name="omitted-headers"></a>

### <a name="omitted-http-headers"></a>Encabezados HTTP omitidos

Si una acción o desencadenador HTTP incluye estos encabezados, Logic Apps quita estos encabezados del mensaje de respuesta generado sin mostrar ninguna advertencia o error:

* Encabezados `Accept-*`, excepto `Accept-version`
* `Allow`
* Encabezados `Content-*` excepto `Content-Disposition`, `Content-Encoding` y `Content-Type`, que se respetan cuando se usan las operaciones POST y PUT. Sin embargo, Logic Apps quita estos encabezados cuando se usa la operación Get.
* Encabezado `Cookie`, pero Logic Apps respeta cualquier valor que especifique mediante la propiedad **Cookie**.
* `Expires`
* `Host`
* `Last-Modified`
* `Origin`
* `Set-Cookie`
* `Transfer-Encoding`

Aunque Logic Apps no le impedirá guardar aplicaciones lógicas que usen una acción o desencadenador HTTP con estos encabezados, Logic Apps omite estos encabezados.

## <a name="connector-reference"></a>Referencia de conectores

Para obtener información técnica acerca de los parámetros de desencadenador y de acción, consulte las siguientes secciones:

* [Parámetros de desencadenador HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-trigger)
* [Parámetros de acción HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-action)

## <a name="next-steps"></a>Pasos siguientes

* [Acceso seguro y datos - Acceso para llamadas salientes a otros servicios y sistemas](../logic-apps/logic-apps-securing-a-logic-app.md#secure-outbound-requests)
* [Conectores de Logic Apps](../connectors/apis-list.md)
