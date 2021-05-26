---
title: Instalación y ejecución de contenedores de Docker para Translator API
titleSuffix: Azure Cognitive Services
description: Use un contenedor de Docker para Translator API para traducir texto.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: how-to
ms.date: 05/25/2021
ms.author: lajanuar
recommendations: false
keywords: entorno local, Docker, contenedor, identificar
ms.openlocfilehash: 93cf53bb53c5a86ff6b4b073105c08c595fcf182
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110480259"
---
# <a name="install-and-run-translator-containers-preview"></a>Instalación y ejecución de contenedores de Translator (versión preliminar)

  Los contenedores permiten ejecutar algunas características del servicio Translator en un entorno propio. Los contenedores son excelentes para requisitos específicos de control de datos y seguridad. En este artículo va a aprender a descargar, instalar y ejecutar un contenedor de Translator.

Los contenedores de Translator permiten compilar una arquitectura de aplicación de traductor optimizada tanto para sólidas capacidades en la nube como para localidad perimetral.

> [!IMPORTANT]
>
> * El contenedor de Translator está en versión preliminar controlada y, para usarlo, debe enviar una solicitud en línea para su aprobación. Vea [Solicitud de aprobación para ejecutar un contenedor](#request-approval-to-run-container) a continuación para obtener más información.
> * El contenedor de Translator admite características limitadas en comparación con las ofertas en la nube.  Vea **Contenedor: Translate** para obtener más detalles.

<!-- markdownlint-disable MD033 -->

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará una [**cuenta de Azure**](https://azure.microsoft.com/free/cognitive-services/) activa.  En caso de no tener ninguna, puede crear una [**cuenta gratuita**](https://azure.microsoft.com/free/).

También va a necesitar lo siguiente para usar contenedores de Translator:

| Obligatorio | Propósito |
|--|--|
| Conocimientos sobre Docker | <ul><li>Debe tener una comprensión básica de conceptos de Docker, como los registros, los repositorios, los contenedores y las imágenes de contenedor, así como conocimientos de [terminología y comandos](/dotnet/architecture/microservices/container-docker-introduction/docker-terminology) básicos de `docker`.</li></ul> |
| Motor de Docker | <ul><li>Necesita que el motor de Docker esté instalado en un [equipo host](#host-computer). Docker dispone de paquetes que configuran el entorno de Docker en [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) y [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Para conocer los principios básicos de Docker y de los contenedores, consulte [Introducción a Docker](https://docs.docker.com/engine/docker-overview/).</li><li> Docker debe configurarse para permitir que los contenedores se conecten con Azure y envíen datos de facturación a dicho servicio. </li><li> En **Windows**, Docker también debe estar configurado de forma que admita contenedores de **Linux**.</li></ul> |
| Recurso de Translator | <ul><li>Recurso de Azure [Translator](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextTranslation) cuya región no es "global", clave de API asociada y URI de punto de conexión. Ambos valores son necesarios para iniciar el contenedor y se pueden encontrar en la página de información general del recurso.</li></ul>|
|||

|Opcionales|Propósito|
|---------|----------|
|CLI (interfaz de la línea de comandos) de Azure |<ul><li> La [CLI de Azure](/cli/azure/install-azure-cli) permite usar un conjunto de comandos en línea para crear y administrar recursos de Azure. Está disponible para instalarse en entornos Windows, macOS y Linux, y se puede ejecutar en un contenedor de Docker y Azure Cloud Shell.</li></ul> |
|||

## <a name="required-elements"></a>Elementos obligatorios

Todos los contenedores de Cognitive Services requieren tres elementos principales:

* **Valor de aceptación de los términos de licencia**. Contrato de licencia para el usuario final establecido con un valor de `Eula=accept`.

* **Clave de API** y **Dirección URL de punto de conexión**.  La clave de API se usa para iniciar el contenedor. Para recuperar los valores de clave de API y dirección URL de punto de conexión, vaya a la página _Claves y punto de conexión_ del recurso de Translator y seleccione el icono `Copy to clipboard` <span class="docon docon-edit-copy x-hidden-focus"></span>.

> [!IMPORTANT]
>
> * Las claves de suscripción se usan para acceder a la API de Cognitive Services. No comparta las claves. Almacénelas de forma segura, por ejemplo, con Azure Key Vault. También se recomienda regenerar estas claves periódicamente. Solo se necesita una clave para realizar una llamada API. Al volver a generar la primera clave, puede usar la segunda clave para seguir teniendo acceso al servicio.

## <a name="host-computer"></a>Equipo host

[!INCLUDE [Host Computer requirements](../../../../includes/cognitive-services-containers-host-computer.md)]

## <a name="container-requirements-and-recommendations"></a>Recomendaciones y requisitos del contenedor

En la tabla siguiente se describen las especificaciones mínima y recomendada para los contenedores de Translator. Se necesitan al menos 2 gigabytes (GB) de memoria y cada CPU debe ser de 2,6 gigahercios (GHz) como mínimo o más. y memoria, en gigabytes (GB), para asignar a cada instancia de Translator. En la tabla siguiente se describen la asignación mínima y recomendada de recursos para cada contenedor de Translator.

| Contenedor | Mínima |Recomendado | Par de idiomas |
|-----------|---------|---------------|----------------------|
| Translator conectado |2 núcleos, 2 GB de memoria |4 núcleos, 8 GB de memoria | 4 |
|||

Para cada par de idiomas, se recomienda tener 2 GB de memoria. De manera predeterminada, el contenedor sin conexión de Translator tiene cuatro pares de idiomas. El núcleo y la memoria se corresponden con los valores de `--cpus` y `--memory` que se usan como parte del comando `docker run`.

> [!NOTE]
>
> * El núcleo de CPU y la memoria corresponden a los valores `--cpus` y `--memory`, que se usan como parte del comando docker run.
>
> * Las especificaciones mínima y recomendada se basan en los límites de Docker, no en los recursos del equipo host.

## <a name="request-approval-to-run-container"></a>Solicitud de aprobación para ejecutar un contenedor

Rellene y envíe la [**solicitud de Azure Cognitive Services de servicios controlados**](https://aka.ms/csgate-translator) para solicitar acceso al contenedor.

[!INCLUDE [Request access to public preview](../../../../includes/cognitive-services-containers-request-access.md)]

## <a name="get-container-images-with-docker-commands"></a>Obtención de imágenes de contenedor con **comandos de Docker**

> [!IMPORTANT]
>
> * Los comandos de Docker de las secciones siguientes usan la barra diagonal inversa, `\`, como un carácter de continuación de línea. Puede quitarla o reemplazarla en función de los requisitos del sistema operativo del host.
> * Para poder ejecutar el contenedor, las opciones `EULA`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará.

Use el comando [docker run](https://docs.docker.com/engine/reference/commandline/run/) para descargar una imagen de contenedor de Microsoft Container Registry y ejecutarla.

```Docker
docker run --rm -it -p 5000:80 --memory 12g --cpus 4 \
-v /mnt/d/TranslatorContainer:/usr/local/models \
-e apikey={API_KEY} \
-e eula=accept \
-e billing={ENDPOINT_URI} \
-e Languages=en,fr,es,ar,ru  \
mcr.microsoft.com/azure-cognitive-services/translator/text-translation
```

El comando anterior:

* Descarga y ejecuta un contenedor de Translator desde la imagen de contenedor.
* Asigna 12 gigabytes (GB) de memoria y cuatro núcleos de CPU.
* Expone el puerto TCP 5000 y asigna un seudo-TTY para el contenedor.
* Acepta el contrato de usuario final
* Configura el punto de conexión de facturación
* Descarga modelos de traducción para los idiomas inglés, francés, español, árabe y ruso
* Una vez que se produce la salida, quita automáticamente el contenedor. La imagen del contenedor sigue estando disponible en el equipo host.

### <a name="run-multiple-containers-on-the-same-host"></a>Ejecución de varios contenedores en el mismo host

Si tiene pensado ejecutar varios contenedores con puertos expuestos, asegúrese de que ejecuta cada contenedor con un puerto expuesto diferente. Por ejemplo, ejecute el primer contenedor en el puerto 5000 y el segundo en el puerto 5001.

Puede tener este contenedor y un contenedor de Azure Cognitive Services diferente en ejecución simultáneamente en el HOST. También puede tener varios contenedores del mismo contenedor de Cognitive Services en ejecución.

## <a name="query-the-containers-translator-endpoint"></a>Consulta del punto de conexión de Translator del contenedor

 El contenedor proporciona una API de punto de conexión de Translator basada en REST. Esta es una solicitud de ejemplo:

```curl
curl -X POST "http://localhost:5000/translate?api-version=3.0&from=en&to=zh-HANS"
    -H "Content-Type: application/json" -d "[{'Text':'Hello, what is your name?'}]"
```

> [!NOTE]
> Si intenta la solicitud cURL POST antes de que el contenedor esté listo, obtiene una respuesta *El servicio no está disponible temporalmente*. Espere hasta que el contenedor esté listo e inténtelo de nuevo.

## <a name="stop-the-container"></a>Detención del contenedor

[!INCLUDE [How to stop the container](../../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshoot"></a>Solución de problemas

### <a name="validate-that-a-container-is-running"></a>Comprobación de que un contenedor está en ejecución

Hay varias maneras de comprobar que el contenedor se está ejecutando:

* El contenedor proporciona una página principal en `\` como validación visual de que se está ejecutando.

* Puede abrir el explorador web que prefiera e ir a la dirección IP externa y el puerto expuesto del contenedor en cuestión. Use las distintas direcciones URL de solicitud para validar que el contenedor se está ejecutando. Las direcciones URL de solicitud de ejemplo que se enumeran a continuación son `http://localhost:5000`, pero el contenedor específico puede variar. Tenga en cuenta que va a ir a la **dirección IP externa** y el puerto expuesto del contenedor.

| URL de la solicitud | Propósito |
|--|--|
| `http://localhost:5000/` | El contenedor proporciona una página principal. |
| `http://localhost:5000/ready` | Solicitado con GET. Proporciona una comprobación de que el contenedor está listo para aceptar una consulta en el modelo.  Esta solicitud se puede usar con los [sondeos de ejecución y preparación](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) de Kubernetes. |
| `http://localhost:5000/status` | Solicitado con GET.  Comprueba si la clave de API usada para iniciar el contenedor es válida sin generar una consulta de punto de conexión. Esta solicitud se puede usar con los [sondeos de ejecución y preparación](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) de Kubernetes. |
| `http://localhost:5000/swagger` | El contenedor cuenta con un completo conjunto de documentación sobre los puntos de conexión y una característica de **prueba**. Esta característica le permite especificar la configuración en un formulario HTML basado en web y realizar la consulta sin necesidad de escribir código. Una vez que la consulta devuelve resultados, se proporciona un ejemplo del comando CURL para mostrar los encabezados HTTP y el formato de cuerpo requeridos. |

:::image type="content" source="../../../../includes/media/cognitive-services-containers-api-documentation/container-webpage.png" alt-text="Página principal del contenedor":::

## <a name="text-translation-code-samples"></a>Ejemplos de código de traducción de texto

### <a name="translate-text-with-swagger"></a>Traducción de texto con Swagger

#### <a name="english-leftrightarrow-german"></a>Inglés &leftrightarrow; Alemán

Vaya a la página de Swagger: <http://localhost:5000/swagger/index.html>

1. Seleccione **POST /translate**.
1. Haga clic en **Probar**.
1. Introduzca el parámetro **Desde** como `en`.
1. Introduzca el parámetro **Hasta** como `de`.
1. Introduzca el parámetro **api-version** como `3.0`.
1. En **texts**, reemplace `string` por el siguiente código JSON.

```json
  [
        {
            "text": "hello, how are you"
        }
  ]
```

Seleccione **Ejecutar** y las traducciones resultantes se mostrarán en **Cuerpo de respuesta**. Debería esperar algo parecido a la siguiente respuesta:

```json
"translations": [
      {
          "text": "hallo, wie geht es dir",
          "to": "de"
      }
    ]
```

### <a name="translate-text-with-python"></a>Traducción de texto con Python

```python
import requests, json

url = 'http://localhost:5000/translate?api-version=3.0&from=en&to=fr'
headers = { 'Content-Type': 'application/json' }
body = [{ 'text': 'Hello, how are you' }]

request = requests.post(url, headers=headers, json=body)
response = request.json()

print(json.dumps(
    response,
    sort_keys=True,
     indent=4,
     ensure_ascii=False,
     separators=(',', ': ')))
```

### <a name="translate-text-with-cnet-console-app"></a>Traducción de texto con la aplicación de consola de C#/.NET

Inicie Visual Studio y cree una aplicación de consola. Edite el archivo `*.csproj` para agregar el nodo `<LangVersion>7.1</LangVersion>` (especifica C# 7.1). Agregue el paquete NuGet [Newtoonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/), versión 11.0.2.

En `Program.cs`, reemplace todo el código existente por lo siguiente:

```csharp
using Newtonsoft.Json;
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;

namespace TranslateContainer
{
    class Program
    {
        const string ApiHostEndpoint = "http://localhost:5000";
        const string TranslateApi = "/translate?api-version=3.0&from=en&to=de";

        static async Task Main(string[] args)
        {
            var textToTranslate = "Sunny day in Seattle";
            var result = await TranslateTextAsync(textToTranslate);

            Console.WriteLine(result);
            Console.ReadLine();
        }

        static async Task<string> TranslateTextAsync(string textToTranslate)
        {
            var body = new object[] { new { Text = textToTranslate } };
            var requestBody = JsonConvert.SerializeObject(body);

            var client = new HttpClient();
            using (var request =
                new HttpRequestMessage
                {
                    Method = HttpMethod.Post,
                    RequestUri = new Uri($"{ApiHostEndpoint}{TranslateApi}"),
                    Content = new StringContent(requestBody, Encoding.UTF8, "application/json")
                })
            {
                // Send the request and await a response.
                var response = await client.SendAsync(request);

                return await response.Content.ReadAsStringAsync();
            }
        }
    }
}
```

## <a name="summary"></a>Resumen

En este artículo ha conocido conceptos y flujos de trabajo para la descarga, instalación y ejecución de contenedores de Translator. Ahora sabe que:

* Translator proporciona contenedores de Linux para Docker.
* Las imágenes de contenedor se descargan desde el registro de contenedor y se ejecutan en Docker.
* Puede usar la API de REST para llamar a la operación "translate" en el contenedor de Translator si especifica el URI de host del contenedor.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Obtenga más información sobre contenedores de Azure Cognitive Services](/azure/cognitive-services/containers/index?context=/azure/cognitive-services/translator/context/context)
