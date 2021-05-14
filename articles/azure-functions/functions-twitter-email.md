---
title: Creación de una función que se integre con Azure Logic Apps
description: Cree una función que se integre con Azure Logic Apps y Azure Cognitive Services. El flujo de trabajo resultante clasifica las opiniones por tweet y envía notificaciones por correo electrónico.
author: craigshoemaker
ms.assetid: 60495cc5-1638-4bf0-8174-52786d227734
ms.topic: tutorial
ms.date: 04/10/2021
ms.author: cshoe
ms.custom: devx-track-csharp, mvc, cc996988-fb4f-47
ms.openlocfilehash: 3517835859de82117de07ad67cdf8027960ab777
ms.sourcegitcommit: aa00fecfa3ad1c26ab6f5502163a3246cfb99ec3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107388688"
---
# <a name="tutorial-create-a-function-to-integrate-with-azure-logic-apps"></a>Tutorial: Creación de una función para integrarla con Azure Logic Apps

Azure Functions se integra con Azure Logic Apps en el diseñador de Logic Apps. Esta integración permite usar la capacidad de proceso de Azure Functions en las orquestaciones con otros servicios tanto de Azure como de terceros.

En este tutorial se muestra cómo crear un flujo de trabajo para analizar la actividad de Twitter. A medida que se evalúan los tweets, el flujo de trabajo envía notificaciones cuando se detectan opiniones positivas.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un recurso de API de Cognitive Services.
> * Crear una función que clasifica las opiniones de tweet.
> * Crear una aplicación de lógica que se conecta a Twitter.
> * Agregar detección de opiniones a la aplicación lógica.
> * Conectar la aplicación lógica a la función.
> * Enviar un correo electrónico en función de la respuesta de la función.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de [Twitter](https://twitter.com/) activa.
* Una cuenta de [Outlook.com](https://outlook.com/) (para enviar las notificaciones).

> [!NOTE]
> Si quiere usar el conector de Gmail, solo las cuentas empresariales de G-Suite pueden usar este conector sin restricciones en las aplicaciones lógicas. Si tiene una cuenta de consumidor de Gmail, puede usar el conector de Gmail solo con aplicaciones y servicios específicos aprobados por Google, o puede [crear una aplicación cliente de Google para usarla en la autenticación con el conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). <br><br>Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

## <a name="create-text-analytics-resource"></a>Creación de recursos de Text Analytics

Cognitive Services APIs están disponibles en Azure como recursos individuales. Use API Text Analytics para detectar la opinión de los tweets que se han enviado.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Haga clic en **Crear un recurso** en la esquina superior izquierda de Azure Portal.

1. En _Categorías_, seleccione **IA y Machine Learning**

1. En _Text Analytics_, seleccione **Crear**.

1. Escriba los siguientes valores en la pantalla _Create Text Analytics_ (Crear Text Analytics).

    | Configuración | Value | Observaciones |
    | ------- | ----- | ------- |
    | Subscription | El nombre de la suscripción de Azure | |
    | Resource group | Cree un grupo de recursos denominado  **tweet-sentiment-tutorial** | Más adelante, eliminará este grupo de recursos para quitar todos los recursos creados en este tutorial. |
    | Region | Seleccione la región más cercana a la suya | |
    | Nombre | **TweetSentimentApp** | |
    | Plan de tarifa | Seleccione **Free F0** (F0 gratis) | |

1. Seleccione **Revisar + crear**.

1. Seleccione **Crear**.

1. Una vez finalizada la implementación, seleccione **Ir al recurso**.

## <a name="get-text-analytics-settings"></a>Obtención de la configuración de Text Analytics

Con el recurso de Text Analytics, copiará varios valores y los reservará para usarlos más adelante.

1. Seleccione **Keys and Endpoint** (Claves y punto de conexión).

1. Copie la **clave 1**, para lo que debe hacer clic en el icono al final del cuadro de entrada.

1. Pegue el valor en un editor de texto.

1. Copie el **punto de conexión**, para lo que debe hacer clic en el icono al final del cuadro de entrada.

1. Pegue el valor en un editor de texto.

## <a name="create-the-function-app"></a>Crear la aplicación de función

1. En el cuadro de búsqueda superior, busque y seleccione **Aplicación de funciones**.

1. Seleccione **Crear**.

1. Escriba los siguientes valores:

    | Configuración | Valor sugerido | Observaciones |
    | ------- | ----- | ------- |
    | Subscription | El nombre de la suscripción de Azure | |
    | Resource group | **tweet-sentiment-tutorial** | Use el mismo nombre de grupo de recursos en todo este tutorial. |
    | Nombre de la aplicación de funciones | **TweetSentimentAPI** + un sufijo único | Los nombres de las aplicaciones de funciones son únicos globalmente. Los caracteres válidos son `a-z` (no distingue mayúsculas de minúsculas), `0-9` y `-`. |
    | Publicar | **Código** | |
    | Pila en tiempo de ejecución | **.NET** | El código de función proporcionado está en C#. |
    | Versión | Seleccione el número de versión más reciente | |
    | Region | Seleccione la región más cercana a la suya | |

1. Seleccione **Revisar + crear**.

1. Seleccione **Crear**.

1. Una vez finalizada la implementación, seleccione **Ir al recurso**.

## <a name="create-an-http-triggered-function"></a>Creación de una función desencadenada con HTTP  

1. En el menú de la izquierda de la ventana _Funciones_, seleccione **Functions**.

1. Seleccione **Agregar** en el menú superior y escriba los valores siguientes.

    | Configuración | Value | Observaciones |
    | ------- | ----- | ------- |
    | Entorno de desarrollo | **Desarrollo en el portal** | |
    | Plantilla | **Desencadenador HTTP** | |
    | Nueva función | **TweetSentimentFunction** | Este es el nombre de la función. |
    | lista desplegable | **Function** | |

1. Seleccione el botón **Agregar**.

1. Seleccione el botón **Code + Test** (Código y prueba).

1. Pegue el siguiente código en la ventana del editor de código.

    ```csharp
    #r "Newtonsoft.Json"
    
    using System;
    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;
    
    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {
    
        string requestBody = String.Empty;
        using (StreamReader streamReader =  new  StreamReader(req.Body))
        {
            requestBody = await streamReader.ReadToEndAsync();
        }
    
        dynamic score = JsonConvert.DeserializeObject(requestBody);
        string value = "Positive";
    
        if(score < .3)
        {
            value = "Negative";
        }
        else if (score < .6) 
        {
            value = "Neutral";
        }
    
        return requestBody != null
            ? (ActionResult)new OkObjectResult(value)
           : new BadRequestObjectResult("Pass a sentiment score in the request body.");
    }
    ```

    Se pasa una puntuación de opinión a la función, que devuelve un nombre de categoría para el valor.

1. Seleccione el botón **Guardar** en la barra de herramientas para guardar los cambios.

    > [!NOTE]
    > Para probar la función, seleccione **Test/Run** (Probar o ejecutar) en el menú superior. En la pestaña _Entrada_, escriba el valor `0.9` en el cuadro de entrada _Cuerpo_ y, después, seleccione **Ejecutar**. Compruebe que se devuelve el valor _Positivo_ en el cuadro _Contenido de respuesta HTTP_ de la sección _Salida_.

A continuación, cree una aplicación lógica que se integre con Azure Functions, Twitter y Cognitive Services API.

## <a name="create-a-logic-app"></a>Creación de una aplicación lógica

1. En el cuadro de búsqueda superior, busque y seleccione **Aplicaciones lógicas**.

1. Seleccione **Agregar**.

1. Seleccione **Consumo** y escriba los valores siguientes.

    | Configuración | Valor sugerido |
    | ------- | --------------- |
    | Subscription | El nombre de la suscripción de Azure |
    | Resource group | **tweet-sentiment-tutorial** |
    | Nombre de la aplicación lógica | **TweetSentimentApp** |
    | Region | Seleccione la región más cercana, preferiblemente la misma que seleccionó en los pasos anteriores. |

    Deje los valores predeterminados en los restantes valores.

1. Seleccione **Revisar + crear**.

1. Seleccione **Crear**.

1. Una vez finalizada la implementación, seleccione **Ir al recurso**.

1. Seleccione el botón **Aplicación lógica en blanco**.

    :::image type="content" source="media/functions-twitter-email/blank-logic-app-button.png" alt-text="Botón Aplicación lógica en blanco":::

1. Seleccione el botón **Guardar** en la barra de herramientas para guardar el progreso.

Ahora puede usar el diseñador de aplicaciones lógicas para agregar servicios y desencadenadores a la aplicación.

## <a name="connect-to-twitter"></a>Conexión a Twitter

Cree una conexión a Twitter para que la aplicación pueda realizar sondeos para saber si hay nuevos tweets.

1. Busque **Twitter** en el cuadro de búsqueda superior.

1. Seleccione el icono **Twitter**.

1. Seleccione el desencadenador **Cuando se publica un tweet nuevo**.

1. Escriba los siguientes valores para configurar la conexión.

    | Configuración |  Value |
    | ------- | ---------------- |
    | Nombre de conexión | **MyTwitterConnection** |
    | Tipo de autenticación | **Usar aplicación compartida predeterminada** |

1. Seleccione **Iniciar sesión**.

1. Siga las indicaciones de la ventana emergente para completar el inicio de sesión en Twitter.

1. A continuación, escriba los siguientes valores en el cuadro _Cuando se publica un tweet nuevo_.

    | Configuración | Value |
    | ------- | ----- |
    | Buscar texto | **#my-twitter-tutorial** |
    | ¿Con qué frecuencia quiere comprobar si hay elementos? | **15** en el cuadro de texto y <br> **Minuto** en la lista desplegable |

1. Seleccione el botón **Guardar** en la barra de herramientas para guardar el progreso.

A continuación, conéctese a Text Analytics para detectar la opinión de los tweets recopilados.

## <a name="add-text-analytics-sentiment-detection"></a>Adición de la detección de opiniones de Text Analytics

1. Seleccione **Nuevo paso**.

1. Busque **Text Analytics** en el cuadro de búsqueda.

1. Seleccione el icono **Text Analytics**.

1. Seleccione **Detectar sentimiento** y escriba los siguientes valores.

    | Configuración | Value |
    | ------- | ----- |
    | Nombre de conexión | **TextAnalyticsConnection** |
    | Clave de cuenta | Pegue la clave de cuenta de Text Analytics que reservó anteriormente. |
    | Dirección URL del sitio | Pegue el punto de conexión de Text Analytics que reservó anteriormente. |

1. Seleccione **Crear**.

1. Haga clic dentro del cuadro _Agregar nuevo parámetro_ y active la casilla situada junto a **documentos**, que aparece en el menú emergente.

1. Haga clic en el cuadro de texto _Id- 1 de documentos_ para abrir el elemento emergente de contenido dinámico.

1. En el cuadro de búsqueda _contenido dinámico_, busque el **identificador** y haga clic en **Id. de tweet**.

1. Haga clic en el cuadro de texto _Text- 1 de documentos_ para abrir el elemento emergente de contenido dinámico.

1. En el cuadro de búsqueda _contenido dinámico_, busque el **texto** y haga clic en **Texto del tweet**.

1. En **Elegir una acción**, escriba **Análisis de texto** y, después, haga clic en la acción **Detectar sentimiento**.

1. Seleccione el botón **Guardar** en la barra de herramientas para guardar el progreso.

El cuadro _Detectar sentimiento_ debe ser similar al de la captura de pantalla siguiente.

:::image type="content" source="media/functions-twitter-email/detect-sentiment.png" alt-text="Configuración de Detectar sentimiento":::

## <a name="connect-sentiment-output-to-function-endpoint"></a>Conexión de la salida de opiniones con el punto de conexión de la función

1. Seleccione **Nuevo paso**.

1. Busque **Azure Functions** en el cuadro de búsqueda.

1. Seleccione el icono de **Azure Functions**.

1. Busque el nombre de la función en el cuadro de búsqueda. Si ha seguido las instrucciones anteriores, el nombre de la función comenzará por **TweetSentimentAPI**.

1. Seleccione el icono de la función.

1. Seleccione el elemento **TweetSentimentFunction**.

1. Haga clic en el cuadro _Cuerpo de la solicitud_ y seleccione el elemento de _puntuación_ **Detectar sentimiento** en la ventana emergente.

1. Seleccione el botón **Guardar** en la barra de herramientas para guardar el progreso.

## <a name="add-conditional-step"></a>Adición de un paso condicional

1. Seleccione el botón **Agregar una acción**.

1. Haga clic en el cuadro _Control_ y busque y seleccione **Control** en la ventana emergente.

1. Seleccione **Condición**.

1. Haga clic en el cuadro _Elegir un valor_ y seleccione el elemento _Cuerpo_ de **TweetSentimentFunction** en la ventana emergente.

1. Escriba **Positivo** en el cuadro _Elegir un valor_.

1. Seleccione el botón **Guardar** en la barra de herramientas para guardar el progreso.

## <a name="add-email-notifications"></a>Agregar notificaciones por correo electrónico

1. En el cuadro _True_, seleccione el botón **Agregar una acción**.

1. Busque **Office 365 Outlook** en el cuadro de texto y selecciónelo.

1. Busque **send (enviar)** y seleccione **Send an email** (Enviar un correo electrónico) en el cuadro de texto.

1. Haga clic en el botón **Sign in** (Iniciar sesión).

1. Siga las indicaciones de la ventana emergente para completar el inicio de sesión en Office 365 Outlook.

1. Escriba su dirección de correo electrónico en el cuadro _Para_.

1. Haga clic en el cuadro _Asunto_ y, después, en el elemento **Cuerpo** de _TweetSentimentFunction_. Si el elemento _Cuerpo_ no se muestra en la lista, haga clic en el vínculo **Ver más** para expandir la lista de opciones.

1. Después del elemento _Cuerpo_ en _Asunto_, escriba el texto **Tweet from:** (Tweet de:).

1. Después del texto _Tweet from:_ (Tweet de: ), vuelva a hacer clic en el cuadro y seleccione **User name** (Nombre de usuario) en la lista de opciones de _Cuando se publica un tweet nuevo_.

1. Haga clic en el cuadro _Cuerpo_ y seleccione **Texto del tweet** en la lista de opciones de _Cuando se publica un tweet nuevo_. Si el elemento _Texto del tweet_ no se muestra en la lista, haga clic en el vínculo **Ver más** para expandir la lista de opciones.

1. Seleccione el botón **Guardar** en la barra de herramientas para guardar el progreso.

El cuadro de correo electrónico debería ser como el de esta captura de pantalla.

:::image type="content" source="media/functions-twitter-email/email-notification.png" alt-text="Notificación de correo electrónico":::

## <a name="run-the-workflow"></a>Ejecución del flujo de trabajo

1. Desde su cuenta de Twitter, publique un tweet con el siguiente texto: **I'm enjoying #my-twitter-tutorial** (Me gusta #my-twitter-tutorial).

1. Vuelva al diseñador de aplicaciones lógicas y seleccione el botón **Ejecutar**.

1. Busque en el correo electrónico un mensaje del flujo de trabajo.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para limpiar todos los servicios y cuentas de Azure creados durante este tutorial, elimine el grupo de recursos.

1. Busque **Grupos de recursos** en el cuadro de búsqueda superior.

1. Seleccione **tweet-sentiment-tutorial**.

1. Seleccione **Eliminar grupo de recursos**.

1. Escriba **tweet-sentiment-tutorial** en el cuadro de texto.

1. Seleccione el botón **Eliminar**.

Si lo desea, puede volver a su cuenta de Twitter y eliminar los tweets de prueba de la fuente.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una API sin servidor mediante Azure Functions](functions-create-serverless-api.md)
