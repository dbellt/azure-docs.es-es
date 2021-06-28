---
title: 'Conector de Azure IoT para FHIR (versión preliminar): Guía de solución de problemas y procedimientos'
description: Cómo solucionar mensajes de error y problemas comunes del conector de Azure IoT para FHIR (versión preliminar) y copiar archivos de asignación
services: healthcare-apis
author: msjasteppe
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: troubleshooting
ms.date: 11/13/2020
ms.author: jasteppe
ms.openlocfilehash: 4e296f8d62afbebe49ebccc60654e26c46cbb2bf
ms.sourcegitcommit: 0beea0b1d8475672456da0b3a4485d133283c5ea
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/28/2021
ms.locfileid: "112991206"
---
# <a name="azure-iot-connector-for-fhir-preview-troubleshooting-guide"></a>Guía de solución de problemas del conector de Azure IoT para FHIR (versión preliminar)

En este artículo se proporcionan los pasos para solucionar condiciones y mensajes de error relacionados con el Conector de Azure IoT para Recursos Rápidos de Interoperabilidad en Salud (FHIR&#174;)*.  

También aprenderá a crear copias del archivo JSON de asignaciones de conversión del conector de Azure IoT para FHIR (por ejemplo: Dispositivo y FHIR).  

Puede usar las copias del archivo JSON de asignaciones de conversión para editar y archivar fuera de Azure Portal.  

> [!TIP]
> Si va a abrir una [incidencia de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/) para el conector de Azure IoT para FHIR, asegúrese de incluir copias de su archivo JSON de asignación de conversión para ayudar con el proceso de solución de problemas.

## <a name="device-and-fhir-conversion-mapping-json-template-validations-for-azure-iot-connector-for-fhir-preview"></a>Validaciones de plantilla JSON de asignación de conversión de dispositivo y FHIR para el conector de Azure IoT para FHIR (versión preliminar)
En esta sección, obtendrá información sobre el proceso de validación que realiza el conector de Azure IoT para FHIR con el fin de validar las plantillas JSON de asignación de conversión de dispositivo y FHIR antes de permitir que se guarden para su uso.  Estos elementos son necesarios en el archivo JSON de asignación de conversión de dispositivo y FHIR.

**Asignación de dispositivos**

|Elemento|Obligatorio|
|:-------|:------|
|TypeName|True|
|TypeMatchExpression|True|
|DeviceIdExpression|True|
|TimestampExpression|True|
|Values[].ValueName|True|
|Values[].ValueExpression|True|

> [!NOTE]
> Values[].ValueName y Values[].ValueExpression
>
> Estos elementos solo son necesarios si tiene una entrada de valor en la matriz; es válido no asignar ningún valor. Se usa cuando la telemetría que se envía es un evento. Por ejemplo: Cuando se coloca o se quita un dispositivo IoMT ponible. Los elementos no tienen valores, excepto un nombre que el conector de Azure IoT para FHIR empareja y emite. En la conversión FHIR, el conector de Azure IoT para FHIR lo asigna a un concepto codificable basado en el tipo semántico: no se rellenan los valores reales.

**Asignación de FHIR**

|Elemento|Obligatorio|
|:------|:-------|
|TypeName|True|

> [!NOTE]
> Este es el único elemento de asignación FHIR necesario validado en este momento.

## <a name="error-messages-and-fixes-for-azure-iot-connector-for-fhir-preview"></a>Mensajes de error y correcciones para el conector de Azure IoT para FHIR (versión preliminar)

|Message|Visualización|Condición|Fix| 
|-------|---------|---------|---|
|Invalid mapping name, mapping name should be device or FHIR (Nombre de asignación no válido, el nombre de la asignación debe ser Dispositivo o FHIR).|API|El tipo de asignación indicado no es Dispositivo ni FHIR.|Use uno de los dos tipos de asignación admitidos (por ejemplo: Dispositivo o FHIR).|
|Error de validación. Falta información necesaria o la que hay no es válida.|API y Azure Portal|Al intentar guardar una asignación de conversión falta información o elementos necesarios.|Agregue la información o el elemento de asignación de conversión que faltan, e intente guardar de nuevo la asignación de conversión.|
|Regenerate key parameters not defined (Los parámetros de la clave de regeneración no están definidos).|API|Solicitud de clave de regeneración.|Incluya los parámetros en la solicitud de clave de regeneración.|
|Reached the maximum number of IoT Connector instances that can be provisioned in this subscription (Se alcanzó el número máximo de instancias del conector de IoT que se pueden aprovisionar en esta suscripción).|API y Azure Portal|Se alcanzó la cuota de la suscripción del conector de Azure IoT para FHIR (el valor predeterminado es [2] por suscripción).|Elimine una de las instancias existentes del conector de Azure IoT para FHIR.  Use otra suscripción que no haya alcanzado la cuota de suscripción.  Solicite un aumento de la cuota de suscripción.|
|Move resource is not supported for IoT Connector enabled Azure API for FHIR resource (No se admite el movimiento de un recurso de Azure API for FHIR compatible con el conector de IoT).|API y Azure Portal|Intenta una operación de movimiento en un recurso de Azure API for FHIR que tiene una o más instancias del conector de Azure IoT para FHIR.|Elimine las instancias existentes del conector de Azure IoT para FHIR para realizar la operación de movimiento.|
|IoT Connector not provisioned (Conector de IoT no aprovisionado).|API|Intento de usar servicios secundarios (conexiones y asignaciones) cuando no se ha aprovisionado el elemento primario (conector de Azure IoT para FHIR).|Aprovisione un conector de Azure IoT para FHIR.|
|No se admite la solicitud.|API|No se admite la solicitud de API específica.|Use la solicitud de API correcta.|
|La cuenta no existe.|API|Intento de agregar un conector de Azure IoT para FHIR y el recurso de Azure API for FHIR no existe.|Cree el recurso de Azure API for FHIR y luego vuelva a intentar la operación.|
|Azure API for FHIR resource FHIR version is not supported for IoT Connector (La versión de FHIR del recurso de Azure API for FHIR no es compatible con el conector de IoT).|API|Intento de usar un conector de Azure IoT para FHIR con una versión incompatible del recurso de Azure API for FHIR.|Cree un nuevo recurso de Azure API for FHIR (versión R4) o use un recurso de Azure API for FHIR existente (versión R4).

## <a name="why-is-my-azure-iot-connector-for-fhir-preview-data-not-showing-up-in-azure-api-for-fhir"></a>¿Por qué los datos del conector de Azure IoT para FHIR (versión preliminar) no aparecen en Azure API for FHIR?

|Posibles problemas|Correcciones|
|----------------|-----|
|Todavía se están procesando los datos.|Los datos salen hacia Azure API for FHIR en lotes (cada 15 minutos aproximadamente).  Es posible que los datos se sigan procesando y que se necesite un tiempo adicional para que los datos se conserven en Azure API for FHIR.|
|No se ha configurado el archivo JSON de asignación de conversión del dispositivo.|Configure y guarde el archivo JSON conforme de asignación de conversión de dispositivo.|
|No se ha configurado el archivo JSON de asignación de conversión de FHIR.|Configure y guarde el archivo JSON conforme de asignación de conversión de FHIR.|
|El mensaje del dispositivo no contiene una expresión esperada definida en la asignación del dispositivo.|Compruebe que las expresiones JsonPath definidas en la asignación del dispositivo coincidan con los tokens definidos en el mensaje del dispositivo.|
|No se ha creado un recurso de dispositivo en Azure API for FHIR (tipo de resolución: solo Lookup)*.|Cree un recurso de dispositivo válido en Azure API for FHIR. Asegúrese de que el recurso de dispositivo contenga un identificador que coincida con el identificador de dispositivo proporcionado en el mensaje entrante.|
|No se ha creado un recurso de paciente en Azure API for FHIR (tipo de resolución: solo Lookup)*.|Cree un recurso de paciente válido en Azure API for FHIR.|
|La referencia Device.patient no está establecida o la referencia no es válida (tipo de resolución: solo Lookup)*.|Asegúrese de que el recurso de dispositivo contenga una [referencia](https://www.hl7.org/fhir/device-definitions.html#Device.patient) válida a un recurso de paciente.| 

\* Consulte [Inicio rápido: Implementación del conector de IoT (versión preliminar) con Azure Portal](iot-fhir-portal-quickstart.md#create-new-azure-iot-connector-for-fhir-preview) para conocer una descripción funcional de los tipos de resolución del conector de Azure IoT para FHIR (por ejemplo: Lookup o Create).

## <a name="use-metrics-to-troubleshoot-issues-in-azure-iot-connector-for-fhir-preview"></a>Uso de métricas para solucionar problemas con el Conector de Azure IoT para FHIR (versión preliminar)

El Conector de Azure IoT para FHIR genera varias métricas para proporcionar información sobre el proceso de flujo de datos. Una de las métricas admitidas se denomina *Total de errores*, que proporciona la cantidad de errores que se producen dentro de una instancia del Conector de Azure IoT para FHIR.

Cada error se registra con varias propiedades asociadas. Cada propiedad proporciona un aspecto distinto del error, lo que puede ayudar a identificar y solucionar problemas. En esta sección se muestran las distintas propiedades capturadas para cada error de la métrica *Total de errores* y los valores posibles de estas propiedades.

> [!NOTE]
> Puede ir a la métrica *Total de errores* para una instancia del Conector de Azure IoT para FHIR (versión preliminar) tal como se describe en la página [Métricas del Conector de Azure IoT para FHIR (versión preliminar)](iot-metrics-display.md).

Haga clic en el gráfico *Total de errores* y haga clic en el botón *Agregar filtro* para segmentar y desglosar la métrica de errores con cualquiera de las propiedades que se mencionan a continuación.

### <a name="the-operation-performed-by-the-azure-iot-connector-for-fhir-preview"></a>La operación realizada por el Conector de Azure IoT para FHIR (versión preliminar).

Esta propiedad representa la operación que estaba realizando el Conector de IoT cuando se generó el error. Por lo general, una operación representa la fase del flujo de datos durante el procesamiento de un mensaje del dispositivo. A continuación se muestra la lista de valores posibles para esta propiedad.

> [!NOTE]
> Puede leer más información sobre las distintas fases del flujo de datos del Conector de Azure IoT para FHIR (versión preliminar) [aquí](iot-data-flow.md).

|Fase del flujo de datos|Descripción|
|---------------|-----------|
|Configuración|Operación específica para configurar una instancia del Conector de IoT|
|Normalización|Fase del flujo de datos en la que se normalizan los datos del dispositivo|
|Agrupar|Fase del flujo de datos en la que se agrupan los datos normalizados|
|FHIRConversion|Fase del flujo de datos en la que los datos normalizados y agrupados se transforman en un recurso FHIR|
|Desconocido|Se desconoce el tipo de operación cuando se produjo un error|

### <a name="the-severity-of-the-error"></a>La gravedad del error

Esta propiedad representa la gravedad del error generado. A continuación se muestra la lista de valores posibles para esta propiedad.

|Gravedad|Descripción|
|---------------|-----------|
|Advertencia|Existen algunos problemas menores en el proceso del flujo de datos, pero el procesamiento del mensaje del dispositivo no se detiene|
|Error|El procesamiento de un mensaje del dispositivo específico encontró un error y otros mensajes pueden seguir ejecutándose según lo previsto|
|Crítico|Hay un problema en el nivel de sistema con el Conector de IoT y no se espera que se procese ningún mensaje|

### <a name="the-type-of-the-error"></a>El tipo de error

Esta propiedad indica una categoría para un error determinado, que básicamente representa una agrupación lógica para un tipo de error similar. A continuación se muestra la lista de valores posibles para esta propiedad.

|Tipo de error|Descripción|
|----------|-----------|
|DeviceTemplateError|Errores relacionados con las plantillas de asignación de dispositivo|
|DeviceMessageError|Se produjeron errores durante el procesamiento de un mensaje de dispositivo específico|
|FHIRTemplateError|Errores relacionados con las plantillas de asignación de FHIR|
|FHIRConversionError|Se produjeron errores al transformar un mensaje en un recurso FHIR|
|FHIRResourceError|Errores relacionados con los recursos existentes en el servidor FHIR al que hace referencia el Conector de IoT|
|FHIRServerError|Errores que se producen al comunicarse con el servidor FHIR|
|GeneralError|Todos los demás tipos de errores|

### <a name="the-name-of-the-error"></a>El nombre del error

Esta propiedad proporciona el nombre de un error específico. Esta es la lista de todos los nombres de error con su descripción y sus tipos de errores asociados, la gravedad y las fases del flujo de datos.

|Nombre del error|Descripción|Tipos de error|Gravedad del error|Fases del flujo de datos|
|----------|-----------|-------------|--------------|------------------|
|MultipleResourceFoundException|Error que se produce cuando se encuentran varios recursos de paciente o dispositivo en el servidor FHIR para los identificadores respectivos presentes en el mensaje del dispositivo|FHIRResourceError|Error|FHIRConversion|
|TemplateNotFoundException|No hay configurada una plantilla de asignación de FHIR o dispositivo con la instancia de un Conector de IoT|DeviceTemplateError, FHIRTemplateError|Crítico|Normalization, FHIRConversion|
|CorrelationIdNotDefinedException|El id. de la correlación no se especifica en la plantilla de asignación de dispositivo. CorrelationIdNotDefinedException es un error condicional que solo se produciría cuando la observación de FHIR debiera agrupar las medidas de un dispositivo con un id. de correlación, pero no se configuró correctamente|DeviceMessageError|Error|Normalización|
|PatientDeviceMismatchException|Este error se produce cuando el recurso de dispositivo del servidor FHIR tiene una referencia a un recurso de paciente que no coincide con el identificador del paciente presente en el mensaje|FHIRResourceError|Error|FHIRConversionError|
|PatientNotFoundException|El recurso FHIR del dispositivo asociado con el identificador del dispositivo presente en el mensaje del dispositivo no hace referencia a ningún recurso FHIR del paciente. Tenga en cuenta que este error solo se produce cuando la instancia del Conector de IoT está configurada con el tipo de resolución *Lookup*|FHIRConversionError|Error|FHIRConversion|
|DeviceNotFoundException|No existe ningún recurso de dispositivo en el servidor FHIR asociado con el identificador de dispositivo presente en el mensaje del dispositivo|DeviceMessageError|Error|Normalización|
|PatientIdentityNotDefinedException|Este error ocurre cuando la expresión de análisis del identificador del paciente del mensaje del dispositivo no está configurada en la plantilla de asignación del dispositivo o el identificador del paciente no está en el mensaje del dispositivo. Tenga en cuenta que este error solo se produce cuando el tipo de resolución del Conector de IoT está establecido en *Create*|DeviceTemplateError|Crítico|Normalización|
|DeviceIdentityNotDefinedException|Este error ocurre cuando la expresión de análisis del identificador del dispositivo del mensaje del dispositivo no está configurada en la plantilla de asignación del dispositivo o el identificador del dispositivo no está en el mensaje del dispositivo|DeviceTemplateError|Crítico|Normalización|
|NotSupportedException|Error al recibir un mensaje del dispositivo con un formato no compatible|DeviceMessageError|Error|Normalización|

## <a name="creating-copies-of-the-azure-iot-connector-for-fhir-preview-conversion-mapping-json"></a>Creación de copias del archivo JSON de asignación de conversión del conector de Azure IoT para FHIR (versión preliminar)

La copia de los archivos de asignación del conector de Azure IoT para FHIR puede ser útil para editar y archivar fuera del sitio web de Azure Portal.

Las copias de los archivos de asignación deben proporcionarse al servicio de soporte técnico de Azure al abrir una incidencia de soporte técnico para ayudar en la solución de problemas.

> [!NOTE]
> En este momento, JSON es el único formato admitido para los archivos de asignación de Dispositivo y FHIR.

> [!TIP]
> Obtenga más información sobre [el archivo JSON de asignación de conversión de Dispositivo y FHIR](iot-mapping-templates.md) del conector de Azure IoT para FHIR.

1. Seleccione **IoT Connector (preview)** (Conector de IoT [versión preliminar]) en la parte inferior izquierda del panel de recursos de Azure API for FHIR en la sección **Complementos**.

   :::image type="content" source="media/iot-troubleshoot/map-files-main-with-box.png" alt-text="IoT Connector1" lightbox="media/iot-troubleshoot/map-files-main-with-box.png":::

2. Seleccione el **Conector** del que va a copiar el archivo JSON de asignación de conversión.

   :::image type="content" source="media/iot-troubleshoot/map-files-select-connector-with-box.png" alt-text="IoT Connector2" lightbox="media/iot-troubleshoot/map-files-select-connector-with-box.png":::

> [!NOTE]
> Este proceso también se puede usar para copiar y guardar el contenido del archivo JSON de **Configure FHIR mapping** (Configurar asignación de FHIR).

3. Seleccione **Configure device mapping** (Configurar asignación de dispositivo).

    :::image type="content" source="media/iot-troubleshoot/map-files-select-device-with-box.png" alt-text="IoT Connector3" lightbox="media/iot-troubleshoot/map-files-select-device-with-box.png":::

4. Seleccione el contenido del archivo JSON y realice una operación de copia (por ejemplo: seleccione CTRL+C). 

   :::image type="content" source="media/iot-troubleshoot/map-files-select-device-json-with-box.png" alt-text="IoT Connector4" lightbox="media/iot-troubleshoot/map-files-select-device-json-with-box.png":::

5. Realice una operación de pegado (por ejemplo: seleccione CTRL+V) en un nuevo archivo dentro de un editor (por ejemplo: Visual Studio Code, Bloc de notas) y guarde el archivo con una extensión *.json.

> [!TIP]
> Si va a abrir una [incidencia de soporte técnico de Azure](https://azure.microsoft.com/support/create-ticket/) para el conector de Azure IoT para FHIR, asegúrese de incluir copias de su archivo JSON de asignación de conversión para ayudar con el proceso de solución de problemas.

## <a name="next-steps"></a>Pasos siguientes

Consulte las preguntas más frecuentes sobre el conector de Azure IoT para FHIR

>[!div class="nextstepaction"]
>[P+ del conector de Azure IoT para FHIR](fhir-faq.yml)

*En Azure Portal, el Conector de Azure IoT para FHIR se conoce como Conector de IoT (versión preliminar). FHIR es una marca registrada de HL7 y se usa con el permiso de HL7.