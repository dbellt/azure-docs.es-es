---
title: Configuración de un contenedor para Form Recognizer
titleSuffix: Azure Applied AI Services
description: Vea cómo configurar el contenedor Form Recognizer para analizar los datos de formularios y tablas.
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: aahi
ms.openlocfilehash: 444e59aa969950eadb8532849ae10b97412555ab
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110374567"
---
# <a name="configure-form-recognizer-containers"></a>Configuración de contenedores de Form Recognizer

[!INCLUDE [Form Recognizer containers limit](includes/container-limit.md)]

Con los contenedores de Azure Form Recognizer, puede crear una arquitectura de aplicación optimizada para aprovechar las sólidas funcionalidades de la nube y la localidad del perímetro.

El entorno de ejecución de los contenedores de Form Recognizer se configura mediante los argumentos del comando `docker run`. Este contenedor tiene varias opciones de configuración necesarias, así como otras opcionales. Para ver algunos ejemplos, consulte la sección ["Comandos de ejemplo de docker run"](#example-docker-run-commands). La configuración específica del contenedor es la configuración de facturación.

## <a name="configuration-settings"></a>Parámetros de configuración

[!INCLUDE [Container shared configuration settings table](../../../includes/cognitive-services-containers-configuration-shared-settings-table.md)]

> [!IMPORTANT]
> Los valores de configuración [`ApiKey`](#apikey-configuration-setting), [`Billing`](#billing-configuration-setting) y [`Eula`](#eula-setting) se usan juntos. Debe proporcionar valores válidos para los tres o el contenedor no se iniciará. Para obtener más información sobre el uso de estas opciones de configuración para crear instancias de un contenedor, consulte [Facturación](form-recognizer-container-howto.md#billing).

## <a name="apikey-configuration-setting"></a>Opción de configuración ApiKey

La opción de configuración `ApiKey` especifica la clave de recurso de Azure usada para realizar un seguimiento de la información de facturación del contenedor. El valor de ApiKey debe ser una clave válida para el recurso _Form Recognizer_ que se especifica para `Billing` en la sección de configuración de facturación.

Puede encontrar esta opción en Azure Portal, en **Form Recognizer Resource Management** (Administración de recursos de Form Recognizer), debajo de **Claves**.

## <a name="applicationinsights-setting"></a>Opción de configuración ApplicationInsights

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Opción de configuración Billing

El valor de configuración `Billing` especifica el URI de punto de conexión del recurso _Form Recognizer_ en Azure que se usa para medir la información de facturación del contenedor. El valor de esta opción de configuración debe ser un URI de punto de conexión válido para un recurso de _Form Recognizer_ en Azure. El contenedor informa sobre el uso cada 10 a 15 minutos.

Puede encontrar esta configuración en Azure Portal, en **Form Recognizer Overview** (Información general de Form Recognizer), debajo de **Punto de conexión**.

|Obligatorio| Nombre | Tipo de datos | Descripción |
|--|------|-----------|-------------|
|Sí| `Billing` | String | Identificador URI del punto de conexión de facturación. Para más información sobre cómo obtener el URI de facturación, consulte la [recopilación de los parámetros necesarios](form-recognizer-container-howto.md#gathering-required-parameters). Para más información y para obtener una lista completa de los puntos de conexión regionales, consulte [Nombres de subdominios personalizados para Cognitive Services.](../cognitive-services-custom-subdomains.md) |

## <a name="eula-setting"></a>Opción de configuración Eula

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Opción de configuración Fluentd

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Configuración de las credenciales del proxy HTTP

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>Opción de configuración Logging

[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]


## <a name="mount-settings"></a>Configuración de montaje

Utilice montajes de enlace para leer y escribir datos hacia y desde el contenedor. Puede especificar un montaje de entrada o un montaje de salida mediante la opción `--mount` del comando [`docker run`](https://docs.docker.com/engine/reference/commandline/run/).

El contenedor Form Recognizer requiere un montaje de entrada y un montaje de salida. El montaje de entrada puede ser de solo lectura y es necesario para acceder a los datos que se usan para el entrenamiento y la puntuación. El montaje de salida se debe poder escribir y se usará para almacenar los modelos y los datos temporales.

La sintaxis exacta de la ubicación de montaje del host varía según el sistema operativo del host. Además, la ubicación de montaje del [equipo host](form-recognizer-container-howto.md#the-host-computer) puede no ser accesible debido a un conflicto entre los permisos de la cuenta de servicio de Docker y los permisos de la ubicación de montaje del host.

|Opcional| Nombre | Tipo de datos | Descripción |
|-------|------|-----------|-------------|
|Obligatorio| `Input` | String | Destino del montaje de entrada. El valor predeterminado es `/input`.    <br><br>Ejemplo:<br>`--mount type=bind,src=c:\input,target=/input`|
|Obligatorio| `Output` | String | Destino del montaje de salida. El valor predeterminado es `/output`.  <br><br>Ejemplo:<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Comandos de ejemplo de docker run

Los ejemplos siguientes usan las opciones de configuración para ilustrar cómo escribir y usar comandos `docker run`. Una vez en ejecución, el contenedor continúa ejecutándose hasta que lo [detenga](form-recognizer-container-howto.md#stop-the-container).

* **Carácter de continuación de línea**: Los comandos de Docker de las secciones siguientes usan una barra diagonal inversa (\\) como carácter de continuación de línea. Puede quitarla o reemplazarla en función de los requisitos del sistema operativo del host.
* **Orden de los argumentos**: no cambie el orden de los argumentos a menos que esté muy familiarizado con los contenedores de Docker.

Reemplace {_argument_name_} en la tabla siguiente por sus propios valores:

| Marcador de posición | Value |
|-------------|-------|
| **{FORM_RECOGNIZER_API_KEY}** | La clave que se usa para iniciar el contenedor. Está disponible en la página de claves de Form Recognizer de Azure Portal. |
| **{FORM_RECOGNIZER_ENDPOINT_URI}** | El valor de URI del punto de conexión de facturación está disponible en Azure Portal, en la página de información general de Form Recognizer.|
| **{COMPUTER_VISION_API_KEY}** | La clave está disponible en la página Claves de Computer Vision API de Azure Portal.|
| **{COMPUTER_VISION_ENDPOINT_URI}** | El punto de conexión de facturación. Si usa un recurso de Computer Vision de la nube, el valor del URI está disponible en la página de información general de Computer Vision API de Azure Portal. Si usa un contenedor *cognitive-services-recognize-text*, utilice la dirección URL del punto de conexión de facturación que se pasa al contenedor en el comando `docker run`. |

Consulte [Recopilación de los parámetros obligatorios](form-recognizer-container-howto.md#gathering-required-parameters) para más información sobre cómo obtener estos valores.

[!INCLUDE [cognitive-services-custom-subdomains-note](../../../includes/cognitive-services-custom-subdomains-note.md)]

> [!IMPORTANT]
> Para ejecutar el contenedor, especifique las opciones `Eula`, `Billing` y `ApiKey`; de lo contrario, el contenedor no se iniciará. Para obtener más información, vea [Facturación](#billing-configuration-setting).

## <a name="form-recognizer-container-docker-examples"></a>Ejemplos de Docker del contenedor de Form Recognizer

Los siguientes ejemplos de Docker son para el contenedor de Form Recognizer.

### <a name="basic-example-for-form-recognizer"></a>Ejemplo básico de Form Recognizer

```Docker
docker run --rm -it -p 5000:5000 --memory 8g --cpus 2 \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={FORM_RECOGNIZER_ENDPOINT_URI} \
ApiKey={FORM_RECOGNIZER_API_KEY} \
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

### <a name="logging-example-for-form-recognizer"></a>Ejemplo de registro de Form Recognizer

```Docker
docker run --rm -it -p 5000:5000 --memory 8g --cpus 2 \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={FORM_RECOGNIZER_ENDPOINT_URI} \
ApiKey={FORM_RECOGNIZER_API_KEY} \
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
Logging:Console:LogLevel:Default=Information
```

## <a name="next-steps"></a>Pasos siguientes

* Revise [Instalación y ejecución de contenedores](form-recognizer-container-howto.md).
