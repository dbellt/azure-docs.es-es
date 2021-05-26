---
title: 'Procedimiento: Desarrollo de aplicaciones de Comandos personalizados: servicio de voz personalizado'
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo desarrollar y personalizar aplicaciones de Comandos personalizados. Estas aplicaciones de comandos de voz son más adecuadas para escenarios de realización de tareas o de comando y control.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 12/15/2020
ms.author: trbye
ms.openlocfilehash: 84253f897e8b85029ea4425a7919be4bf71028ed
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110451951"
---
# <a name="develop-custom-commands-applications"></a>Desarrollo de aplicaciones de Comandos personalizados

En este artículo de procedimientos, aprenderá a desarrollar y configurar aplicaciones de Comandos personalizados. La característica Comandos personalizados ayuda a la creación de aplicaciones de comandos de voz enriquecidas y optimizadas para las experiencias de interacción que priorizan la voz. La característica es más adecuada para escenarios de realización de tareas o de comando y control. Es especialmente adecuada para los dispositivos de Internet de las cosas (IoT) y para dispositivos ambientales y desatendidos.

En este artículo, creará una aplicación que puede encender y apagar un televisor, establecer la temperatura y establecer una alarma. Después de crear estos comandos básicos, obtendrá información sobre las siguientes opciones para personalizar comandos:

* Adición de parámetros a comandos
* Adición de configuraciones a parámetros de comandos
* Creación de reglas de interacción
* Creación de plantillas de generación de idioma para respuestas de voz
* Uso de herramientas de voces personalizadas

## <a name="create-an-application-by-using-simple-commands"></a>Creación de una aplicación mediante comandos simples

Empiece por crear una aplicación de Comandos personalizados vacía. Para obtener más información, consulte la [guía de inicio rápido](quickstart-custom-commands-application.md). En esta aplicación, en lugar de importar un proyecto, se crea un proyecto en blanco.

1. En el cuadro **Nombre**, escriba el nombre del proyecto *Smart-Room-Lite* (u otro nombre de su elección).
1. En la lista **Idioma**, seleccione **Inglés (Estados Unidos)** .
1. Seleccione o cree un recurso de LUIS.

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla que muestra la ventana "Nuevo proyecto".](media/custom-commands/create-new-project.png)

### <a name="update-luis-resources-optional"></a>Actualización de recursos de LUIS (opcional)

Puede actualizar el recurso de creación que seleccionó en la ventana **Nuevo proyecto**. También puede establecer un recurso de predicción.

Se usa un recurso de predicción para el reconocimiento cuando se publica la aplicación de Comandos personalizados. No necesita un recurso de predicción durante las fases de desarrollo y pruebas.

### <a name="add-a-turnon-command"></a>Adición de un comando TurnOn

En la aplicación de Comandos personalizados Smart-Room-Lite vacía que creó, agregue un comando. El comando procesará una expresión, `Turn on the tv`. Responderá con el mensaje `Ok, turning the tv on`.

1. Cree un nuevo comando; para ello, seleccione **Nuevo comando** en la parte superior del panel izquierdo. Se abre la ventana **Nuevo comando**.
1. En el campo **Nombre**, especifique el valor `TurnOn`.
1. Seleccione **Crear**.

En el panel central se muestran las propiedades del comando.

En la siguiente tabla se explican las propiedades de configuración del comando. Para más información, vea [Conceptos y definiciones de Comandos personalizados](./custom-commands-references.md).

| Configuración            | Descripción                                                                                                                 |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Oraciones de ejemplo | Expresiones de ejemplo que el usuario puede decir para desencadenar este comando.                                                                 |
| Parámetros       | Información necesaria para completar el comando.                                                                                |
| Reglas de finalización | Acciones que se van a realizar para finalizar el comando. Ejemplos: responder al usuario o comunicarse con un servicio web. |
| Reglas de interacción   | Otras reglas para controlar situaciones más específicas o complejas.                                                              |


> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra dónde crear un comando.](media/custom-commands/add-new-command.png)

#### <a name="add-example-sentences"></a>Adición de oraciones de ejemplo

En la sección **Oraciones de ejemplo**, se especifica un ejemplo de lo que el usuario puede decir.

1. En el panel central, seleccione **Example sentences** (Oraciones de ejemplo).
1. En el panel de la derecha, agregue ejemplos:

    ```
    Turn on the tv
    ```

1.  En la parte superior del panel, seleccione **Guardar**.

Aún no hay ningún parámetro, por lo que puede ir a la sección **Completion rules** (Reglas de finalización).

#### <a name="add-a-completion-rule"></a>Adición de una regla de finalización

A continuación, el comando necesita una regla de finalización. Esta regla informa al usuario que se está llevando a cabo una acción de cumplimiento.

Para obtener más información sobre las reglas y las reglas de finalización, vea [Conceptos y definiciones de Comandos personalizados](./custom-commands-references.md).

1. Seleccione la regla de finalización predeterminada **Listo**. Edítela de la manera siguiente:


    | Configuración    | Valor sugerido                          | Descripción                                        |
    | ---------- | ---------------------------------------- | -------------------------------------------------- |
    | **Nombre**       | `ConfirmationResponse`                  | Nombre que describe el propósito de la regla.          |
    | **Condiciones** | None                                     | Condiciones que determinan cuándo se puede ejecutar la regla.    |
    | **Acciones**    | **Send speech response (Enviar respuesta de voz)**  > **Simple editor (Editor sencillo)**  > **First variation (Primera variación)**  > `Ok, turning the tv on` | Acción que se realizará cuando la condición de la regla sea true. |

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla que muestra dónde crear una respuesta de voz.](media/custom-commands/create-speech-response-action.png)

1. Seleccione **Guardar** para guardar la acción.
1. De nuevo en la sección **Completion rules** (reglas de finalización), seleccione **Guardar** para guardar todos los cambios.

    > [!NOTE]
    > No tiene que usar la regla de finalización predeterminada que viene con el comando. Puede eliminar la regla de finalización predeterminada y agregar su propia regla.

### <a name="add-a-settemperature-command"></a>Adición de un comando SetTemperature

Ahora, agregue un comando más, `SetTemperature`. Este comando tomará una sola expresión (`Set the temperature to 40 degrees`) y responderá con el mensaje `Ok, setting temperature to 40 degrees`.

Para crear el nuevo comando, siga los pasos que usó para el comando `TurnOn`, pero use la oración de ejemplo `Set the temperature to 40 degrees`.

A continuación, edite las reglas de finalización de **Listo** de la siguiente manera:

| Configuración    | Valor sugerido                          |
| ---------- | ---------------------------------------- |
| **Nombre**  | `ConfirmationResponse`                  |
| **Condiciones** | None                                     |
| **Acciones**    | **Send speech response (Enviar respuesta de voz)**  > **Simple editor (Editor sencillo)**  > **First variation (Primera variación)**  > `Ok, setting temperature to 40 degrees` |

Seleccione **Guardar** para guardar todos los cambios del comando.

### <a name="add-a-setalarm-command"></a>Adición de un comando SetAlarm

Cree un comando `SetAlarm` nuevo. Use la oración de ejemplo `Set an alarm for 9 am tomorrow`. A continuación, edite las reglas de finalización de **Listo** de la siguiente manera:

| Configuración    | Valor sugerido                          |
| ---------- | ---------------------------------------- |
| **Nombre**  | `ConfirmationResponse`                  |
| **Condiciones** | None                                     |
| **Acciones**    | **Send speech response (Enviar respuesta de voz)**  > **Simple editor (Editor sencillo)**  > **First variation (Primera variación)**  > `Ok, setting an alarm for 9 am tomorrow` |

Seleccione **Guardar** para guardar todos los cambios del comando.

### <a name="try-it-out"></a>Prueba

Use el panel de prueba para probar el comportamiento de la aplicación:

1. En la esquina superior derecha, seleccione el icono **Entrenar**.
1. Cuando finalice el entrenamiento, seleccione **Probar**.

Pruebe los siguientes ejemplos de expresiones mediante el uso de voz o texto:

- Escriba: *set the temperature to 40 degrees* (establece la temperatura en 40 grados)
- La respuesta esperada: Ok, setting temperature to 40 degrees (de acuerdo, definiré la temperatura en 40 grados)
- Si escribe: *turn on the tv* (enciende el televisor)
- La respuesta esperada: Ok, turning the tv on (de acuerdo, encenderé el televisor)
- Escriba: *set an alarm for 9 am tomorrow* (establece una alarma para mañana a las 9 de la mañana)
- La respuesta esperada: Ok, setting an alarm for 9 am tomorrow (de acuerdo, estableceré una alarma para mañana a las 9:00)

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra la prueba en una interfaz de chat web.](media/custom-commands/create-basic-test-chat.png)

> [!TIP]
> En el panel de pruebas, puede seleccionar **Turn details** (Activar detalles) para obtener información sobre cómo se procesó esta entrada de voz/texto.

## <a name="add-parameters-to-commands"></a>Adición de parámetros a comandos

En esta sección, obtendrá información sobre cómo agregar parámetros a los comandos. Los comandos requieren parámetros para completar una tarea. En los escenarios complejos, los parámetros se pueden usar para definir las condiciones que desencadenan acciones personalizadas.

### <a name="configure-parameters-for-a-turnon-command"></a>Configuración de parámetros para un comando TurnOn

Empiece por editar el comando `TurnOn` existente para activar y desactivar varios dispositivos.

1. Ahora que el comando controlará los escenarios de encendido y apagado, cambie el nombre del comando a *TurnOnOff*.
   1. En el panel de la izquierda, seleccione el comando **TurnOn**. A continuación, junto a **Nuevo comando** en la parte superior del panel, seleccione el botón de puntos suspensivos ( **...** ).

   1. Seleccione **Rename** (Cambiar nombre). En la ventana **Rename command** (Cambiar nombre del comando), cambie el nombre a *TurOnOff*.

1. Agregue un parámetro nuevo al comando. El parámetro representa si el usuario quiere encender o apagar el dispositivo.
   1. En la parte superior del panel, seleccione **Agregar**. En el menú desplegable, seleccione **Parámetro**.
   1. En el panel de la derecha, en la sección **Parámetros**, en el cuadro **Nombre**, agregue `OnOff`.
   1. Seleccione **Requerido**. En la ventana **Add response for a required parameter** (Agregar respuesta para un parámetro requerido), seleccione **Simple editor** (Editor sencillo). En el campo **First variation** (Primera variación), agregue *On or Off?* ¿Encender o apagar?.
   1. Selecciona **Actualización**.

       > [!div class="mx-imgBorder"]
       > ![Captura de pantalla que muestra la sección "Add response for a required parameter" (Agregar respuesta para un parámetro requerido) con la pestaña "Simple editor" (Editor sencillo) seleccionada.](media/custom-commands/add-required-on-off-parameter-response.png)

   1. Configure las propiedades del parámetro mediante la tabla siguiente. Para obtener información sobre todas las propiedades de configuración de un comando, vea [Conceptos y definiciones de Comandos personalizados](./custom-commands-references.md).


       | Configuración      | Valor sugerido     | Descripción                                                      |
       | ------------------ | ----------------| ---------------------------------------------------------------------|
       | **Nombre**               | `OnOff`           | Nombre descriptivo del parámetro.                                                                           |
       | **Is Global** (Es global)          | No seleccionado       | Casilla que indica si el valor de este parámetro se aplica globalmente a todos los comandos de la aplicación.|
       | **Obligatorio**           | Seleccionado         | Casilla que indica si es obligatorio especificar un valor para este parámetro antes de finalice el comando. |
       | **Respuesta para el parámetro requerido**      |**Simple editor (Editor sencillo)**  > `On or Off?`      | Mensaje que pregunta el valor de este parámetro cuando no se conoce. |
       | **Tipo**               | **String**          | Tipo de parámetro; por ejemplo, Number, String, Date Time o Geography.   |
       | **Configuración**      | **Accept predefined input values from internal catalog** (Aceptar valores de entrada predefinidos del catálogo interno) | En el caso de las cadenas, esta configuración limita las entradas a un conjunto de valores posibles. |
       | **Predefined input values** (Valores de entrada predefinidos)     | `on`, `off`           | Conjunto de valores posibles y sus alias.         |


   1. Para agregar valores de entrada predefinidos, seleccione **agregar un** de entrada predefinido. En ventana **Nuevo elemento**, escriba *Nombre* como se muestra en la tabla anterior. En este caso, no usa alias, por lo que puede dejar este campo en blanco.

      > [!div class="mx-imgBorder"]
      > ![Captura de pantalla que muestra cómo crear un parámetro.](media/custom-commands/create-on-off-parameter.png)

   1. Seleccione **Guardar** para guardar todas las configuraciones del parámetro.

#### <a name="add-a-subjectdevice-parameter"></a>Adición de un parámetro SubjectDevice

1. Para agregar un segundo parámetro que represente el nombre de los dispositivos que se podrán controlar con este comando, seleccione **Agregar**. Use la configuración siguiente.


    | Configuración            | Valor sugerido       |
    | ------------------ | --------------------- |
    | **Nombre**               | `SubjectDevice`         |
    | **Is Global** (Es global)          | No seleccionado             |
    | **Obligatorio**           | Seleccionado               |
    | **Respuesta para el parámetro requerido**     | **Simple editor (Editor sencillo)**  > `Which device do you want to control?`    | 
    | **Tipo**               | **String**                |     
    | **Configuración**      | **Accept predefined input values from internal catalog** (Aceptar valores de entrada predefinidos del catálogo interno) | 
    | **Predefined input values** (Valores de entrada predefinidos) | `tv`, `fan`               |
    | **Alias** (`tv`)      | `television`, `telly`     |

1. Seleccione **Guardar**.

#### <a name="modify-example-sentences"></a>Modificación de oraciones de ejemplo

Para los comandos que usan parámetros, resulta útil agregar oraciones de ejemplo que cubran todas las combinaciones posibles. Por ejemplo:

* Información completa del parámetro: `turn {OnOff} the {SubjectDevice}`
* Información parcial del parámetro: `turn it {OnOff}`.
* Sin información del parámetro: `turn something`.

Las oraciones de ejemplo que usan diferentes grados de información permiten que la aplicación de Comandos personalizados resuelva las resoluciones individuales y las resoluciones de varias activaciones y desactivaciones mediante información parcial.

Teniendo en cuenta esta información, edite las oraciones de ejemplo para usar estos parámetros sugeridos:

```
turn {OnOff} the {SubjectDevice}
{SubjectDevice} {OnOff}
turn it {OnOff}
turn something {OnOff}
turn something
```

Seleccione **Guardar**.

> [!TIP]
> En el editor de oraciones de ejemplo, use llaves para hacer referencia a los parámetros. Por ejemplo, `turn {OnOff} the {SubjectDevice}`.
> Use una tabulación para la finalización automática respaldada por parámetros creados previamente.

#### <a name="modify-completion-rules-to-include-parameters"></a>Modificación de reglas de finalización para incluir parámetros

Modifique la regla de finalización existente `ConfirmationResponse`.

1. En la sección **Conditions** (Condiciones), seleccione **Add a condition** (Agregar una condición).
1. En la ventana **New Condition** (Nueva condición), en la lista **Type** (Tipo), seleccione **Required parameters** (Parámetros requeridos). En la lista siguiente, seleccione **OnOff** y **SubjectDevice**.
1. Seleccione **Crear**.
1. En la sección **Acciones**, edite la acción **Send speech response** (Enviar respuesta de voz) existente. Para ello, mantenga el puntero sobre ella y seleccione el botón de edición. Esta vez, use los parámetros `OnOff` y `SubjectDevice` creados recientemente:

    ```
    Ok, turning the {SubjectDevice} {OnOff}
    ```
1. Seleccione **Guardar**.

Para probar los cambios, seleccione el icono **Entrenar** en la parte superior del panel de la derecha.

Cuando finalice el entrenamiento, seleccione **Probar**. Aparece una ventana **Test your application** (Probar la aplicación). Pruebe las siguientes interacciones:

- Entrada: *Turn off the tv* (apaga el televisor)
- Salida: Ok, turning on the TV (de acuerdo, encenderé el televisor)
- Entrada: *turn off the television* (apaga el televisor)
- Salida: Ok, turning on the TV (de acuerdo, encenderé el televisor)
- Entrada: *turn it off* (apágalo)
- Salida: Which device do you want to control? (¿Qué dispositivo quiere controlar?)
- Entrada: *the tv* (el televisor)
- Salida: Ok, turning on the TV (de acuerdo, encenderé el televisor)

### <a name="configure-parameters-for-a-settemperature-command"></a>Configuración de parámetros para un comando SetTemperature

Modifique el comando `SetTemperature` para que pueda establecer la temperatura según las indicaciones del usuario.

Agregue un parámetro `Temperature`. Use la configuración siguiente:

| Configuración      | Valor sugerido     |
| ------------------ | ----------------|
| **Nombre**               | `Temperature`           |
| **Obligatorio**           | Seleccionado         |
| **Respuesta para el parámetro requerido**      | **Simple editor (Editor sencillo)**  > `What temperature would you like?`
| **Type**               | `Number`          |


Edite las expresiones de ejemplo para usar los siguientes valores.

```
set the temperature to {Temperature} degrees
change the temperature to {Temperature}
set the temperature
change the temperature
```

Edite las reglas de finalización existentes. Use la configuración siguiente.

| Configuración      | Valor sugerido     |
| ------------------ | ----------------|
| **Condiciones**         | **Parámetro obligatorio** > **Temperatura**           |
| **Acciones**           | **Send speech response (Enviar respuesta de voz)**  > `Ok, setting temperature to {Temperature} degrees` |

### <a name="configure-parameters-for-a-setalarm-command"></a>Configuración de parámetros para un comando SetAlarm

Agregue un parámetro llamado `DateTime`. Use la configuración siguiente.

   | Configuración                           | Valor sugerido                     |
   | --------------------------------- | ----------------------------------------|
   | **Nombre**                              | `DateTime`                               |
   | **Obligatorio**                          | Seleccionado                                 |
   | **Respuesta para el parámetro requerido**   | **Simple editor (Editor sencillo)**  > `For what time?`            |
   | **Tipo**                              | **DateTime**                                |
   | **Valores predeterminados de fecha**                     | Si falta la fecha, use la fecha de hoy.            |
   | **Valores predeterminados de hora**                     | Si falta la hora, use el inicio del día.     |


> [!NOTE]
> En este artículo se usan principalmente los tipos de parámetro String, Number y DateTime. Para obtener una lista de todos los tipos de parámetro admitidos y sus propiedades, vea [Conceptos y definiciones de Comandos personalizados](./custom-commands-references.md).


Edite las expresiones de ejemplo. Use los valores siguientes.

```
set an alarm for {DateTime}
set alarm {DateTime}
alarm for {DateTime}
```

Edite las reglas de finalización existentes. Use la configuración siguiente.

   | Configuración    | Valor sugerido                               |
   | ---------- | ------------------------------------------------------- |
   | **Acciones**    | **Send speech response (Enviar respuesta de voz)**  > `Ok, alarm set for {DateTime}`  |

Pruebe los tres comandos juntos mediante expresiones relacionadas con distintos comandos. (Puede cambiar entre los distintos comandos).

- Entrada: *Set an alarm* (Establece una alarma)
- Salida: For what time? (¿A qué hora?)
- Entrada: *Turn on the tv* (Enciende el televisor)
- Salida: Ok, turning the tv on (de acuerdo, encenderé el televisor)
- Entrada: *Set an alarm* (Establece una alarma)
- Salida: For what time? (¿A qué hora?)
- Entrada: *5 pm*
- Salida: Ok, alarm set for 2020-05-01 17:00:00 (De acuerdo, alarma establecida para el 05/01/2020 a las 17:00:00)

## <a name="add-configurations-to-command-parameters"></a>Adición de configuraciones a parámetros de comandos

En esta sección, obtendrá más información sobre la configuración avanzada de parámetros, incluido lo siguiente:

 - Cómo los valores de parámetro pueden pertenecer a un conjunto definido fuera de la aplicación de Comandos personalizados.
 - Cómo agregar cláusulas de validación según los valores de los parámetros.

### <a name="configure-a-parameter-as-an-external-catalog-entity"></a>Configuración de un parámetro como entidad de catálogo externa

La característica de Comandos personalizados permite configurar parámetros de tipo cadena para hacer referencia a los catálogos externos hospedados a través de un punto de conexión web. Esto le permite actualizar el catálogo externo de forma independiente sin editar la aplicación de Comandos personalizados. Este enfoque resulta útil en los casos en los que las entradas de catálogo son numerosas.

Vuelva a usar el parámetro `SubjectDevice` del comando `TurnOnOff`. La configuración actual de este parámetro es **Accept predefined inputs from internal catalog** (aceptar valores de entrada predefinidos del catálogo interno). Esta configuración se refiere a una lista estática de dispositivos de la configuración de parámetros. Traslade este contenido a un origen de datos externo que se pueda actualizar de forma independiente.

Para mover el contenido, empiece por agregar un nuevo punto de conexión web. En el panel de la izquierda, vaya a la sección **Puntos de conexión web**. Allí, agregue un nuevo punto de conexión web. Use la configuración siguiente.

| Configuración | Valor sugerido |
|----|----|
| **Nombre** | `getDevices` |
| **URL** | `https://aka.ms/speech/cc-sampledevices` |
| **Método** | **GET** |


Si el valor sugerido para la dirección URL no funciona, configure y hospede un punto de conexión web que devuelva un archivo JSON que conste de la lista los dispositivos que se pueden controlar. El punto de conexión web debe devolver un archivo JSON con el formato siguiente:

```json
{
    "fan" : [],
    "refrigerator" : [
        "fridge"
    ],
    "lights" : [
        "bulb",
        "bulbs",
        "light"
        "light bulb"
    ],
    "tv" : [
        "telly",
        "television"
        ]
}

```

A continuación, vaya a la página de configuración del parámetro **SubjectDevice**. Configure las siguientes propiedades.

| Configuración | Valor sugerido |
| ----| ---- |
| **Configuración** | **Aceptar entradas predefinidas del catálogo externo** |                               
| **Punto de conexión del catálogo** | `getDevices` |
| **Método** | **GET** |

Después, seleccione **Guardar**.

> [!IMPORTANT]
> No verá la opción para configurar un parámetro que acepte entradas de un catálogo externo, a menos que el punto de conexión web esté configurado en la sección **Punto de conexión web** del panel de la izquierda.

Pruébelo seleccionando **Entrenar**. Después de que finalice el entrenamiento, seleccione **Probar** y pruebe algunas interacciones.

* Entrada: *turn on* (enciende)
* Salida: Which device do you want to control? (¿Qué dispositivo quiere controlar?)
* Entrada: *lights* (luces)
* Salida: De acuerdo, apagaré las luces

> [!NOTE]
> Ahora puede controlar todos los dispositivos hospedados en el punto de conexión web. Pero aún debe entrenar la aplicación para probar los cambios nuevos y luego volver a publicar la aplicación.

### <a name="add-validation-to-parameters"></a>Adición de validación a los parámetros

Las *validaciones* son construcciones que se aplican a determinados tipos de parámetro y permiten configurar restricciones para el valor del parámetro. Le solicitan correcciones si los valores no se encuentran dentro de las restricciones. Para obtener una lista de tipos de parámetro que extienden la construcción de validación, vea [Conceptos y definiciones de Comandos personalizados](./custom-commands-references.md).

Pruebe las validaciones mediante el comando `SetTemperature`. Use los siguientes pasos para agregar una validación al parámetro `Temperature`.

1. En el panel de la izquierda, seleccione el comando **SetTemperature**.
1. En el panel central, seleccione **Temperatura**.
1. En el panel de la derecha, seleccione **Add a validation** (Agregar una validación).
1. En la ventana **New validation** (Nueva validación), configure la validación tal como se muestra en la tabla siguiente Seleccione **Crear**.


    | Configuración de parámetros | Valor sugerido | Descripción |
    | ---- | ---- | ---- |
    | **Valor mínimo** | `60` | En el caso de los parámetros numéricos, el valor mínimo que puede asumir este parámetro |
    | **Valor máximo** | `80` | En el caso de los parámetros numéricos, el valor máximo que puede asumir este parámetro |
    | **Error de respuesta** |  **Simple editor (Editor sencillo)**  > **First variation (Primera variación)**  > `Sorry, I can only set temperature between 60 and 80 degrees. What temperature do you want?` | Un mensaje para solicitar un valor nuevo si se produce un error en la validación |

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla en la que se muestra cómo agregar una validación de un intervalo.](media/custom-commands/add-validations-temperature.png)

Pruébelo seleccionando el icono **Entrenar** en la parte superior del panel de la derecha. Después de que finalice el entrenamiento, seleccione **Probar**. Pruebe algunas interacciones:

- Entrada: *Set the temperature to 72 degrees* (Establece la temperatura en 72 grados)
- Salida: Ok, setting temperature to 72 degrees (De acuerdo, definiendo la temperatura en 72 grados)
- Entrada: *Set the temperature to 45 degrees* (Establece la temperatura en 45 grados)
- Salida: Sorry, I can only set temperature between 60 and 80 degrees (el valor de temperatura solo puede establecerse entre 60 y 80 grados)
- Entrada: *make it 72 degrees instead* (cámbiala a 72 grados en su lugar)
- Salida: Ok, setting temperature to 72 degrees (De acuerdo, definiendo la temperatura en 72 grados)

## <a name="add-interaction-rules"></a>Adición de reglas de interacción

Las reglas de interacción son reglas *adicionales* que controlan situaciones más específicas o complejas. Aunque tiene la libertad de crear sus propias reglas de interacción, en este ejemplo, usará reglas de interacción para los siguientes escenarios:

* Confirmación de comandos
* Adición de correcciones de un paso a los comandos

Para obtener más información sobre las reglas de interacción, vea [Conceptos y definiciones de Comandos personalizados](./custom-commands-references.md).

### <a name="add-confirmations-to-a-command"></a>Adición de confirmaciones a un comando

Para agregar una confirmación, use el comando `SetTemperature`. Para conseguir la confirmación, use los pasos siguientes para crear reglas de interacción:

1. En el panel de la izquierda, seleccione el comando **SetTemperature**.
1. En el panel central, agregue reglas de interacción seleccionando **Agregar**. A continuación, seleccione **Reglas de interacción** > **Confirm command** (Confirmar comando).

    Esta acción agrega tres reglas de interacción. Las reglas solicitan al usuario que confirme la fecha y la hora de la alarma. Esperan una confirmación (sí o no) para la siguiente activación o desactivación.

    1. Modifique la regla de interacción **Confirm Command** (Confirmar comando) mediante la configuración siguiente:
        1. Cambie el nombre a **Confirm temperature** (Confirmar temperatura).
        1. Agregue una condición nueva: **Parámetros requeridos** > **Temperatura**.
        1. Agregue una nueva acción: **Tipo** > **Send speech response (Enviar respuesta de voz)**  > **Are you sure you want to set the temperature as {Temperature} degrees?** (¿Está seguro de que quiere definir la temperatura en {Temperature} grados?)
        1. En la sección **Expectations** (Expectativas), deje el valor predeterminado de **Expecting confirmation from user** (A la espera de confirmación del usuario).

         > [!div class="mx-imgBorder"]
         > ![Captura de pantalla que muestra cómo crear la respuesta de parámetro necesario.](media/custom-speech-commands/add-validation-set-temperature.png)


    1. Modifique la regla de interacción **Confirmation succeeded** (Confirmación correcta) para controlar una confirmación correcta (el usuario dijo sí).

          1. Cambie el nombre a **Confirmation temperature succeeded** (Confirmación de temperatura correcta).
          1. Deje la condición **Confirmation was successful** (La confirmación se realizó correctamente) existente.
          1. Agregue una condición nueva: **Tipo** > **Parámetros requeridos** > **Temperatura**.
          1. Deje el valor predeterminado de **Post-execution state** (Estado posterior a la ejecución) como **Execute completion rules** (Ejecutar reglas de finalización).

    1. Modifique la regla de interacción **Confirmation denied** (Confirmación denegada) para controlar los escenarios en los que se deniega la confirmación (el usuario dijo que no).

          1. Cambie el nombre a **Confirmation temperature denied** (Confirmación de temperatura denegada).
          1. Deje la condición **Confirmation was denied** (La confirmación se denegó) existente.
          1. Agregue una condición nueva: **Tipo** > **Parámetros requeridos** > **Temperatura**.
          1. Agregue una nueva acción: **Tipo** > **Send speech response (Enviar respuesta de voz)**  > **No problem (No hay problema). What temperature then?** (Entonces, ¿qué temperatura?).
          1. Cambie el valor predeterminado de **Post-execution state** (Estado posterior a la ejecución) a **Wait for user's input** (Esperar la entrada del usuario).

> [!IMPORTANT]
> En este artículo, se usa la funcionalidad de confirmación integrada. También puede agregar manualmente reglas de interacción de una en una.

Pruebe los cambios seleccionando **Entrenar**. Cuando finalice el entrenamiento, seleccione **Probar**.

- **Entrada**: *Set temperature to 80 degrees* (Establece la temperatura en 80 grados)
- **Salida**: ¿está seguro de que quiere establecer la temperatura en 80 grados?
- **Entrada**: *No*
- **Salida**: No se preocupe. What temperature then? (Entonces, ¿qué temperatura?)
- **Entrada**: *72 degrees* (72 grados)
- **Salida**: ¿está seguro de que quiere establecer la temperatura en 72 grados?
- **Entrada**: *Sí*
- **Salida**: Ok, setting temperature to 72 degrees (De acuerdo, definiré la temperatura en 72 grados)

### <a name="implement-corrections-in-a-command"></a>Implementación de correcciones en un comando

En esta sección, configurará una corrección de un paso. Esta corrección se usa después de que se haya ejecutado la acción de cumplimiento. También verá un ejemplo de cómo la corrección se habilita de forma predeterminada si el comando aún no se ha completado. Para agregar una corrección cuando no ha finalizado el comando, agregue el nuevo parámetro `AlarmTone`.

En el panel izquierdo, seleccione el comando **Setalarm**. A continuación, agregue el nuevo parámetro **AlarmTone**.

- **Nombre** > `AlarmTone`
- **Tipo** > **Cadena**
- **Valor predeterminado** > **Campanillas**
- **Configuración** > **Aceptar valores de entrada predefinidos del catálogo interno**
- **Predefined input values (Valores de entrada predefinidos)**  > **Campanillas**, **Tintineo** y **Eco** (Estos valores son entradas individuales predefinidas).


A continuación, actualice la respuesta del parámetro **DateTime** como **Ready to set alarm with tone as {AlarmTone}. For what time? (Listo para definir la alarma con el tono {AlarmTone} ¿A qué hora?)** . A continuación, modifique la regla de finalización como se indica a continuación:

1. Seleccione la regla de finalización existente **ConfirmationResponse**.
1. En el panel de la derecha, mantenga el mouse sobre la acción existente y seleccione **Editar**.
1. Actualice la respuesta de voz a `OK, alarm set for {DateTime}. The alarm tone is {AlarmTone}`.

> [!IMPORTANT]
> El tono de la alarma puede cambiar sin ninguna configuración explícita en un comando en curso. Por ejemplo, puede cambiar cuando el comando todavía no ha terminado. Hay una corrección habilitada de forma *predeterminada* para todos los parámetros del comando, independientemente del número de activación o desactivación si el comando aún no se ha completado.

#### <a name="implement-a-correction-when-a-command-is-finished"></a>Implementación de una corrección cuando finaliza un comando

La plataforma de Comandos personalizados también permite la corrección de un paso, incluso cuando el comando ha finalizado. Esta característica no está habilitada de forma predeterminada. Debe configurarse explícitamente.

Siga estos pasos para configurar una corrección de un paso:

1. En el comando **SetAlarm**, agregue una regla de interacción del tipo **Update previous command** (Actualizar el comando anterior) para actualizar la alarma establecida previamente. Cambie el nombre predeterminado de la regla de interacción a **Update previous alarm** (Actualizar la alarma anterior).
1. Deje la condición predeterminada: **Previous command needs to be updated** (El comando anterior debe actualizarse).
1. Agregue una condición nueva: **Tipo** > **Parámetro requerido** > **DateTime**.
1. Agregue una nueva acción: **Tipo** > **Send speech response (Enviar respuesta de voz)**  > **Simple editor (Editor sencillo)**  > **Updating previous alarm time to {DateTime} (Actualizando hora de alarma anterior a {DateTime})** .
1. Deje el valor predeterminado del **Post-execution state** (Estado posterior a la ejecución) como **Comando completado**.

Pruebe los cambios seleccionando **Entrenar**. Espere a que finalice el entrenamiento y seleccione **Probar**.

- **Entrada**: *Set an alarm* (Establece una alarma).
- **Salida**: Ready to set alarm with tone as Chimes. (Listo para establecer una alarma con el tono campanillas). For what time? (¿A qué hora?)
- **Entrada**: *Set an alarm with the tone as Jingle for 9 am tomorrow* (Establece una alarma con el tono Tintineo para mañana a las 9:00).
- **Salida**: OK, alarm set for 2020-05-21 09:00:00 (De acuerdo, alarma establecida para el 21/05/2020 a las 9:00:00) The alarm tone is Jingle. (El tono de alarma es Tintineo).
- **Entrada**: *No, 8 am* (No, 08:00).
- **Salida**: Updating previous alarm time to 2020-05-29 08:00 (actualizaré la hora de la alarma anterior al 29/05/2020 a las 8:00:00).

> [!NOTE]
> En una aplicación real, en la sección **Acciones** de esta regla de corrección, también deberá enviar una actividad al cliente o llamar a un punto de conexión HTTP para actualizar la hora de la alarma en el sistema. Esta acción solo debe ser responsable de actualizar la hora de la alarma. No debe ser responsable de ningún otro atributo del comando. En este caso, el atributo sería el tono de alarma.

## <a name="add-language-generation-templates-for-speech-responses"></a>Adición de plantillas de generación de idioma para respuestas de voz

Las plantillas de generación de idioma (LG) le permiten personalizar las respuestas enviadas al cliente. Introducen variedad en las respuestas. Puede lograr la generación de idiomas mediante:

* Plantillas de generación de idioma.
* Expresiones adaptables.

Las plantillas de Comandos personalizados están basadas en las [plantillas LG](/azure/bot-service/file-format/bot-builder-lg-file-format#templates) de Bot Framework. Dado que la característica Comandos personalizados crea una nueva plantilla LG cuando es necesario (para las respuestas de voz de los parámetros o acciones), no es necesario especificar el nombre de la plantilla LG.

Por lo tanto, no es necesario definir la plantilla de la siguiente manera:

 ```
    # CompletionAction
    - Ok, turning {OnOff} the {SubjectDevice}
    - Done, turning {OnOff} the {SubjectDevice}
    - Proceeding to turn {OnOff} {SubjectDevice}
 ```

En su lugar, defina el cuerpo de la plantilla sin el nombre, de esta manera:

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra un ejemplo del editor de plantillas.](./media/custom-commands/template-editor-example.png)


Este cambio ofrece variedad en las respuestas de voz que se envían al cliente. Para una expresión, la respuesta de voz correspondiente se elegirá de forma aleatoria entre las opciones proporcionadas.

Si aprovecha las plantillas LG, también puede definir respuestas de voz complejas para los comandos mediante expresiones adaptables. Para obtener más información, vea [Formato de las plantillas LG](/azure/bot-service/file-format/bot-builder-lg-file-format#templates).

De forma predeterminada, la característica Comandos personalizados admite todas sus funcionalidades, con las siguientes variaciones mínimas:

* En las plantillas LG, las entidades se representan como `${entityName}`. La característica Comandos personalizados no usa entidades. Pero puede usar parámetros como variables con la representación `${parameterName}` o la representación `{parameterName}`.
* La característica Comandos personalizados no admite la composición y la expansión de plantillas, ya que nunca se edita el archivo *.lg* directamente. Solo se editan las respuestas de las plantillas creadas automáticamente.
* La característica Comandos personalizados no admite las funciones personalizadas que LG inserta. Se admiten las funciones predefinidas.
* La característica Comandos personalizados no admite opciones, como `strict`, `replaceNull` y `lineBreakStyle`.

### <a name="add-template-responses-to-a-turnonoff-command"></a>Adición de respuestas de plantilla a un comando TurnOnOff

Use el comando `TurnOnOff` para agregar un nuevo parámetro. Use la configuración siguiente.

| Configuración            | Valor sugerido       |
| ------------------ | --------------------- |
| **Nombre**               | `SubjectContext`         |
| **Is Global** (Es global)          | No seleccionado             |
| **Obligatorio**           | No seleccionado               |
| **Tipo**               | **String**                |
| **Valor predeterminado**      | `all` |
| **Configuración**      | **Aceptar valores de entrada predefinidos del catálogo interno** |
| **Predefined input values** (Valores de entrada predefinidos) | `room`, `bathroom`, `all`|

#### <a name="modify-a-completion-rule"></a>Modificación de una regla de finalización

Edite la sección **Acciones** de la regla de finalización existente **ConfirmationResponse**. En la ventana **Editar acción**, cambie a **Template Editor** (Editor de plantillas). A continuación, reemplace el texto por el siguiente ejemplo.

```
- IF: @{SubjectContext == "all" && SubjectDevice == "lights"}
    - Ok, turning all the lights {OnOff}
- ELSEIF: @{SubjectDevice == "lights"}
    - Ok, turning {OnOff} the {SubjectContext} {SubjectDevice}
- ELSE:
    - Ok, turning the {SubjectDevice} {OnOff}
    - Done, turning {OnOff} the {SubjectDevice}
```

Entrene y pruebe la aplicación mediante la entrada y salida siguientes. Observe la variación de las respuestas. La variación de las respuestas se crea por las distintas alternativas para el valor de plantilla y por el uso de expresiones adaptables.

* Entrada: *turn on the tv* (Enciende el televisor)
* Salida: Ok, turning the tv on (de acuerdo, encenderé el televisor)
* Entrada: *turn on the tv* (Enciende el televisor)
* Salida: Done, turned on the tv (listo, encendí el televisor)
* Entrada: *turn off the lights* (Apaga las luces)
* Salida: Ok, turning all the lights off (de acuerdo, apagaré las luces)
* Entrada: *turn off room lights* (Apaga las luces de la habitación)
* Salida: Ok, turning off the room lights (de acuerdo, apagaré las luces de la habitación)

## <a name="use-a-custom-voice"></a>Uso de una voz personalizada

Otra manera de personalizar las respuestas de Comandos personalizados consiste en seleccionar una voz de salida. Siga estos pasos para cambiar la voz predeterminada por una voz personalizada:

1. En la aplicación de Comandos personalizados, en el panel de la izquierda, seleccione **Configuración**.
1. En el panel central, seleccione **Voz personalizada**.
1. En la tabla, seleccione una voz personalizada o una voz pública.
1. Seleccione **Guardar**.

> [!div class="mx-imgBorder"]
> ![Captura de pantalla que muestra oraciones y parámetros de ejemplo.](media/custom-commands/select-custom-voice.png)

> [!NOTE]
> En el caso de las voces públicas, los tipos neuronales solo están disponibles para regiones específicas. Para más información, consulte [Regiones del servicio de voz admitidas](./regions.md#neural-and-standard-voices).
>
> Puede crear voces personalizadas en la página del proyecto **Voz personalizada**. Para obtener información, consulte [Introducción a voz personalizada](./how-to-custom-voice.md).

Ahora la aplicación responderá en la voz seleccionada, en lugar de en la voz predeterminada.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [integrar la aplicación de Comandos personalizados](how-to-custom-commands-setup-speech-sdk.md) con una aplicación cliente mediante el SDK de Voz.
* [Configuración de la implementación continua](how-to-custom-commands-deploy-cicd.md) para una aplicación de Comandos personalizados mediante Azure DevOps.
