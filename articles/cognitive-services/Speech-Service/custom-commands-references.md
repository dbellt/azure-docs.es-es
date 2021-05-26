---
title: 'Conceptos y definiciones de los Comandos personalizados: servicio de Voz'
titleSuffix: Azure Cognitive Services
description: En este artículo, obtendrá información sobre los conceptos y las definiciones de las aplicaciones de Comandos personalizados.
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: sausin
ms.openlocfilehash: c0694e7d136b7fd339362145179939d350eb23ef
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110475971"
---
# <a name="custom-commands-concepts-and-definitions"></a>Conceptos y definiciones de Comandos personalizados

Este artículo funcionará como referencia sobre los conceptos y las definiciones de las aplicaciones de Comandos personalizados.

## <a name="commands-configuration"></a>Configuración de comandos
Los comandos son los bloques de creación fundamentales de una aplicación de Comandos personalizados. Un comando es un conjunto de configuraciones necesarias para completar una tarea específica que ha definido un usuario.

### <a name="example-sentences"></a>Oraciones de ejemplo
Las expresiones de ejemplo son el conjunto de ejemplos que el usuario puede decir para desencadenar un comando determinado. Solo tiene que proporcionar ejemplos de expresiones, no una lista exhaustiva.

### <a name="parameters"></a>Parámetros
Los parámetros son información que los comandos requieren para completar una tarea. En los escenarios complejos, los parámetros también se pueden usar para definir las condiciones que desencadenan acciones personalizadas.

### <a name="completion-rules"></a>Reglas de finalización
Las reglas de finalización son una serie de reglas que se ejecutarán una vez que el comando esté listo para completarse; es decir, cuando se cumplan todas las condiciones de las reglas.

### <a name="interaction-rules"></a>Reglas de interacción
Las reglas de interacción son reglas adicionales para controlar situaciones más específicas o complejas. Puede agregar validaciones adicionales o configurar características avanzadas, tales como confirmaciones o correcciones de un paso. También puede crear sus propias reglas de interacción personalizadas.

## <a name="parameters-configuration"></a>Configuración de parámetros

Los parámetros son la información que los comandos requieren para completar una tarea. En los escenarios complejos, los parámetros también se pueden usar para definir las condiciones que desencadenan acciones personalizadas.

### <a name="name"></a>Nombre
Un parámetro se identifica mediante la propiedad de nombre. Siempre debe asignar un nombre descriptivo a los parámetros. Se puede hacer referencia a un parámetro en distintas secciones; por ejemplo, durante la construcción de condiciones, respuestas de voz u otras acciones.

### <a name="required"></a>Obligatorio
Esta casilla indica si es obligatorio especificar un valor para este parámetro para completar el comando. Debe configurar las respuestas que pedirán al usuario que proporcione un valor si el parámetro se marca como obligatorio.

Tenga en cuenta que, aunque configure un **valor predeterminado** para **parámetro obligatorio**, el sistema solicitará explícitamente el valor del parámetro.

### <a name="type"></a>Tipo
Comandos personalizados admite los siguientes tipos de parámetros:

* DateTime
* Geography
* Number
* String

Todos estos tipos de parámetros, excepto Geography, son compatibles con la configuración de valores predeterminados, que puede realizar desde el portal.

### <a name="configuration"></a>Configuración
La configuración es una propiedad de parámetro que solo está definida para el tipo Cadena. Se admiten los valores siguientes:

* **No**.
* **Accept full input** (Aceptar la entrada completa): Cuando está habilitado, un parámetro acepta cualquier expresión de entrada. Esta opción es útil cuando el usuario necesita un parámetro con una expresión completa. Un ejemplo son las direcciones postales.
* **Accept predefined input values from an external catalog (Aceptar valores de entrada predefinidos de un catálogo externo)** : este valor se usa para configurar un parámetro que puede asumir una gran variedad de valores. Un ejemplo es un catálogo de ventas. En este caso, el catálogo se hospeda en un punto de conexión web externo y se puede configurar de forma independiente.
* **Accept predefined input values from internal catalog** (Aceptar valores de entrada predefinidos del catálogo interno): este valor se usa para configurar un parámetro que puede asumir algunos valores. En este caso, los valores se deben configurar en Speech Studio.


### <a name="validation"></a>Validación
Las validaciones son construcciones que se aplican a determinados tipos de parámetro y permiten configurar restricciones para el valor de un parámetro. Actualmente, Comandos personalizados admite validaciones para los siguientes tipos de parámetros:

* DateTime
* Number

## <a name="rules-configuration"></a>Configuración de reglas
Una regla de Comandos personalizados se define mediante un conjunto de *condiciones* que, cuando se cumplen, ejecutan un conjunto de *acciones*. Una regla también permite configurar su *estado posterior a la ejecución* y las *expectativas* para el siguiente turno.

### <a name="types"></a>Tipos
Comandos personalizados admite las siguientes categorías de reglas:

* **Reglas de finalización**: estas reglas se deben ejecutar cuando se completa un comando. Se ejecutarán todas las reglas configuradas en esta sección cuyas condiciones se cumplan.
* **Reglas de interacción**: estas reglas se pueden usar para configurar validaciones personalizadas adicionales, confirmaciones o correcciones de un paso, así como para aplicar cualquier otra lógica de diálogo personalizada. Las reglas de interacción se evalúan durante el procesamiento de cada turno y se pueden usar para desencadenar reglas de finalización.

Las distintas acciones configuradas como parte de una regla se ejecutan en el orden en el que aparecen en el portal de creación.

### <a name="conditions"></a>Condiciones
Las condiciones son los requisitos que deben cumplirse para que se ejecute una regla. Las condiciones de una regla pueden ser de uno de los tipos siguientes:

* **Parameter value equals** (El valor del parámetro es igual a): el valor del parámetro configurado es igual a un valor específico.
* **No parameter value**  (Sin valor de parámetro): los parámetros configurados no deben tener ningún valor.
* **Parámetros obligatorios**: el parámetro configurado tiene un valor.
* **All required parameters** (Todos los parámetros obligatorios): todos los parámetros marcados como obligatorios tienen un valor.
* **Updated parameters** (Parámetros actualizados): uno o varios valores de parámetro se actualizaron como resultado de procesar la entrada actual (expresión o actividad).
* **Confirmation was successful** (Confirmación correcta): la expresión o actividad de entrada tuvo una confirmación correcta (sí).
* **Confirmation was denied** (La confirmación se denegó): la expresión o actividad no tuvo una confirmación correcta (no).
* **Previous command needs to be updated** (El comando anterior debe actualizarse): esta condición se usa en las instancias en las que quiere detectar una confirmación negada junto con una actualización. En segundo plano, esta condición se configura para cuando el motor de diálogo detecte una confirmación negativa en la que la intención sea la misma que la del turno anterior y el usuario haya respondido con una actualización.

### <a name="actions"></a>Acciones
* **Send speech response** (Enviar respuesta de voz): envía una respuesta de voz al cliente.
* **Update parameter value** (Actualizar valor del parámetro): actualiza el valor de un parámetro de comando a un valor especificado.
* **Clear parameter value** (Borrar valor del parámetro): borra el valor del parámetro de comando.
* **Call web endpoint** (Llamar al punto de conexión web): permite realizar una llamada a un punto de conexión web.
* **Send activity to client** (Enviar actividad al cliente): envía una actividad personalizada al cliente.

### <a name="expectations"></a>Expectativas
Las expectativas se usan para configurar sugerencias para el procesamiento de la siguiente entrada de usuario. Se admiten los siguientes tipos:

* **Expecting confirmation from user** (A la espera de confirmación del usuario): especifica que la aplicación espera una confirmación (sí/no) para la siguiente entrada del usuario.
* **Expecting parameter(s) input from user** (A la espera de entrada de parámetros) del usuario: esta expectativa especifica uno o más parámetros de comando que la aplicación espera de la entrada del usuario.

### <a name="post-execution-state"></a>Estado posterior a la ejecución
El estado posterior a la ejecución es el estado del diálogo después de procesar la entrada actual (expresión o actividad). Es uno de los siguientes tipos:

* **Keep current state** (Mantener estado actual): se mantiene solo el estado actual.
* **Complete the command** (Completar el comando): finaliza el comando y no se procesará ninguna regla adicional del comando.
* **Expecting parameter(s) input from user** (A la espera de la entrada de parámetros del usuario): ejecuta todas las reglas de finalización válidas.
* **Esperar la entrada del usuario**: espere a la siguiente entrada del usuario.



## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Consulte los ejemplos en GitHub](https://aka.ms/speech/cc-samples)
