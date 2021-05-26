---
title: Uso del widget del reproductor de Azure Video Analyzer
description: En este artículo de referencia se explica cómo agregar un widget del reproductor de Video Analyzer a la aplicación.
ms.service: azure-video-analyzer
ms.topic: reference
ms.date: 05/11/2021
ms.openlocfilehash: 12565f2ca1a86d3a9e57fef652abab9564cb7b5e
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388648"
---
# <a name="using-the-azure-video-analyzer-player-widget"></a>Uso del widget del reproductor de Azure Video Analyzer

En este tutorial aprenderá a usar el widget del reproductor de Azure Video Analyzer dentro de la aplicación.  Este código es un widget fácil de insertar que permitirá a los usuarios finales reproducir vídeo y desplazarse por las partes de un archivo de vídeo segmentado.  Para ello, generará una página HTML estática con el widget insertado y todas las piezas para que funcione.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Creación de un token
> * Enumeración de los vídeos
> * Obtención de la dirección URL base para reproducir un [recurso de aplicación de vídeo](./terminology.md#video)
> * Creación de una página con el reproductor
> * Paso del punto de conexión de streaming y el token al reproductor

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura

- [Componentes web](https://developer.mozilla.org/en-US/docs/Web/Web_Components)
- [TypeScript](https://www.typescriptlang.org)

## <a name="prerequisites"></a>Requisitos previos

Los requisitos previos de este tutorial son los siguientes:
* Una cuenta de Azure que tenga una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), en caso de que aún no lo haya hecho. 
* [Visual Studio Code](https://code.visualstudio.com/) u otro editor para el archivo HTML.
uno.
* [Grabación y reproducción continua de vídeo](./use-continuous-video-recording.md) o [Detección de movimiento y grabación de vídeo en dispositivos perimetrales](./detect-motion-record-video-clips-cloud.md)

## <a name="create-a-token"></a>Creación de un token

En esta sección, crearemos un token JWT que usaremos más adelante en el documento.  Usaremos una aplicación de ejemplo que generará el token JWT y le proporcionará todos los campos necesarios para crear la directiva de acceso.

> [!NOTE] 
> Si está familiarizado con cómo generar un token JWT basado en un certificado RSA o ECC, puede omitir esta sección.

1. Descargue la aplicación JWTTokenIssuer que puede encontrar [aquí](https://github.com/Azure-Samples/video-analyzer-iot-edge-csharp/tree/main/src/jwt-token-issuer/readme.md).

   > [!NOTE] 
   > Para más información sobre cómo configurar los valores de audiencia, consulte este [artículo](./access-policies.md).

2. Inicie Visual Studio Code y abra la carpeta que contiene el archivo *.sln.
3. En el panel del explorador, vaya al archivo program.cs.
4. Modifique la línea 77: cambie la audiencia al valor del punto de conexión de Video Analyzer seguido de /videos/* para que tenga el siguiente aspecto:

   ```
   https://{Azure Video Analyzer Account ID}.api.{Azure Long Region Code}.videoanalyzer.azure.net/videos/*
   ```
5. Modifique la línea 78: cambie el emisor al valor del emisor del certificado.  Ejemplo: https://contoso.com

   > [!NOTE] 
   > El punto de conexión de Video Analyzer se puede encontrar en la sección de información general del recurso de Video Analyzer en Azure. Tendrá que hacer clic en el vínculo "Vista JSON". 

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/player-widget/endpoint.png" alt-text="Widget del reproductor: punto de conexión":::
6. Guarde el archivo.
7. Pulse `F5` para ejecutar la aplicación JWTTokenIssuer.

Esto compilará y ejecutará la aplicación.  Después de la compilación, se ejecutará y creará un certificado mediante openssl.  También puede ejecutar el archivo JWTTokenIssuer.exe ubicado en la carpeta de depuración.  La ventaja de ejecutar la aplicación es que puede especificar las opciones de entrada de la siguiente manera:

- JwtTokenIssuer [--audience=<audience>] [--issuer=<issuer>] [--expiration=<expiration>] [--certificatePath=<filepath> --certificatePassword=<password>]

JWTTokenIssuer creará el token JWT y los siguientes componentes necesarios:

- `kty`; `alg`; `kid`; `n`; `e`

Asegúrese de copiar estos valores para su uso posterior.

## <a name="create-an-access-policy"></a>Creación de una directiva de acceso

Las directivas de acceso definen los permisos y la duración del acceso a una secuencia de vídeo determinada de Video Analyzer.  En este tutorial, configuraremos una directiva de acceso para Video Analyzer en Azure Portal.  

1. Inicie sesión en Azure Portal y vaya al grupo de recursos en el que se encuentra la cuenta de Video Analyzer.
1. Seleccione el recurso de Video Analyzer.
1. En Video Analyzer, seleccione Access Policies (Directivas de acceso).

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/player-widget/portal-access-policies.png" alt-text="Widget del reproductor: directivas de acceso en el portal":::    
1. Haga clic en New (Nueva) y escriba lo siguiente:

   > [!NOTE] 
   > Estos valores proceden de la aplicación JWTTokenIssuer creada en el paso anterior.

   - Nombre de la directiva de acceso: cualquier nombre

   - Emisor: debe coincidir con el emisor del token JWT. 

   - Audiencia: audiencia del token JWT; el valor predeterminado es ${System.Runtime.BaseResourceUrlPattern}.  Para más información sobre la audiencia y ${System.Runtime.BaseResourceUrlPattern}, consulte este [artículo](./access-policies.md).

   - Tipo de clave: kty (RSA) 

   - Algoritmo: los valores admitidos son RS256, RS384 y RS512

   - Identificador de clave: kid (generado a partir del certificado)

   - Valor N: para RSA, el valor N es el módulo.

   - Valor E: para RSA, el valor E es el exponente público.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/player-widget/access-policies-portal.png" alt-text="Widget del reproductor: directivas de acceso en el portal":::     
1. Haga click en `save` (Guardar).

## <a name="list-video-analyzer-video-resources"></a>Enumeración de los recursos de vídeo de Video Analyzer

A continuación, es necesario generar una lista de recursos de vídeo.  Esto se realiza mediante una llamada REST al punto de conexión de la cuenta que hemos usado anteriormente, autenticando con el token que hemos generado.

Hay muchas maneras de enviar una solicitud GET a una API REST, pero para ello vamos a usar una función de JavaScript.  En el código siguiente se usa un elemento [XMLHttpRequest](https://www.w3schools.com/xml/ajax_xmlhttprequest_create.asp) junto con los valores que almacenamos en los campos `clientApiEndpointUrl` y `token` de la página para enviar una solicitud `GET` sincrónica.  A continuación, el código toma la lista de vídeos resultante y los almacena en el área de texto `videoList` que hemos configurado en la página.

```javascript
function getVideos()
{
    var xhttp = new XMLHttpRequest();
    var getUrl = document.getElementById("clientApiEndpointUrl").value + "/videos?api-version=2021-05-01-preview";
    xhttp.open("GET", getUrl, false);
    xhttp.setRequestHeader("Authorization", "Bearer " + document.getElementById("token").value);
    xhttp.send();
    document.getElementById("videoList").value = xhttp.responseText.toString();
}
```

## <a name="add-the-video-analyzer-player-component"></a>Adición del componente del reproductor de Video Analyzer

Ahora que tenemos una dirección URL del punto de conexión de la API de cliente, un token y un nombre de vídeo, podemos agregar el reproductor a la página.

1. Incluya el módulo del propio reproductor agregando el paquete directamente a la página.  Puede incluir la dirección del paquete NPM en la aplicación o hacer que se inserte dinámicamente en tiempo de ejecución, como se muestra aquí:
   ```html
   <script async type="module" src="https://unpkg.com/@azure/video-analyzer-widgets"></script>
   ```
1. Agregue un elemento AVA-Player al documento:
   ```html
   <ava-player width="720px" id="avaPlayer"></ava-player>
   ```
1. Obtenga un vínculo al widget del reproductor de Video Analyzer que se encuentra en la página:
   ```javascript
   const avaPlayer = document.getElementById("avaPlayer");
   ```
1. Para configurar el reproductor con los valores que tiene, deberá configurarlos como un objeto, tal como se muestra aquí:
   ```javascript
   avaPlayer.configure( {
      token: document.getElementById("token").value,
      clientApiEndpointUrl: document.getElementById("clientApiEndpointUrl").value,
      videoName: document.getElementById("videoName").value
   } );
   ```
1. Carga del vídeo en el reproductor para comenzar
   ```javascript
   avaPlayer.load();
   ```

## <a name="put-it-all-together"></a>Colocación de todo junto

Si combinamos los elementos web anteriores, se obtiene la siguiente página HTML estática que nos permitirá usar un punto de conexión de la cuenta y un token para ver un vídeo.

```html
<html>
<head>
<title>Video Analyzer Player Widget Demo</title>
</head>
<script async type="module" src="https://unpkg.com/@azure/video-analyzer-widgets"></script>
<body>
<script>
    function getVideos()
    {
        var xhttp = new XMLHttpRequest();
        var getUrl = document.getElementById("clientApiEndpointUrl").value + "/videos?api-version=2021-05-01-preview";
        xhttp.open("GET", getUrl, false);
        xhttp.setRequestHeader("Authorization", "Bearer " + document.getElementById("token").value);
        xhttp.send();
        document.getElementById("videoList").value = xhttp.responseText.toString();
    }
    function playVideo() {
        const avaPlayer = document.getElementById("avaPlayer");
        avaPlayer.configure( {
            token: document.getElementById("token").value,
            clientApiEndpointUrl: document.getElementById("clientApiEndpointUrl").value,
            videoName: document.getElementById("videoName").value
        } );
        avaPlayer.load();
    }
</script>
Client API endpoint URL: <input type="text" id="clientApiEndpointUrl" /><br><br>
Token: <input type="text" id="token" /><br><br>
<button type="submit" onclick="getVideos()">Get Videos</button><br><br>
<textarea rows="20" cols="100" id="videoList"></textarea><br><br>
Video name: <input type="text" id="videoName" /><br><br>
<button type="submit" onclick="playVideo()">Play Video</button><br><br>
<ava-player width="720px" id="avaPlayer"></ava-player>
</body>
</html>
```

## <a name="host-the-page"></a>Hospedaje de la página

Puede probar esta página de forma local, pero puede querer probar una versión hospedada.  En caso de que no tenga una forma rápida de hospedar una página, estas son instrucciones sobre cómo hacerlo mediante [sitios web estáticos](../../storage/blobs/storage-blob-static-website.md) con Storage.  A continuación se muestra una versión condensada de [estas instrucciones más completas](../../storage/blobs/storage-blob-static-website-how-to.md) actualizadas para los archivos que se usan en este tutorial.

1. Creación de una cuenta de Storage
1. A la izquierda, en `Data management` (Administración de datos), haga clic en `Static website` (Sitio web estático).
1. Seleccione `Enable` (Habilitar) para habilitar el sitio web estático en la cuenta de almacenamiento.
1. En `Index document name` (Nombre del documento de índice), escriba `index.html`.
1. En `Error document path` (Ruta de acceso del documento de error), escriba `404.html`.
1. Seleccione `Save` (Guardar) en la parte superior.
1. Anote el valor de `Primary endpoint` (Punto de conexión principal) que aparece: este será su sitio web
1. Haga clic en el elemento `$web` situado sobre `Primary endpoint` (Punto de conexión principal).
1. Con el botón `Upload` (Cargar) de la parte superior, cargue la página HTML estática como `index.html`.

## <a name="play-a-video"></a>Reproducción de un vídeo

Ahora que tiene la página hospedada, vaya hasta ella y debería poder seguir estos pasos.

1. Coloque los valores de `Client API endpoint URL` (Dirección URL del punto de conexión de la API de cliente) y `Token`.
1. Presione `Get videos`
1. En la lista de vídeos, seleccione un nombre de vídeo y rellénelo en el campo de entrada `Video name` (Nombre del vídeo).
1. Presione `Play video`

## <a name="additional-details"></a>Detalles adicionales

### <a name="refreshing-the-access-token"></a>Actualización del token de acceso

El reproductor usa el token de acceso que generamos anteriormente para obtener un token de autorización de reproducción.  Los tokens expiran periódicamente, por lo que se debe actualizar.  Hay dos maneras de actualizar el token de acceso para el reproductor después de haber generado uno nuevo.

* Llamar activamente al método `setAccessToken` del widget
    ```typescript
    avaPlayer.setAccessToken('<NEW-ACCESS-TOKEN>');
    ```
* Escuchar el evento `TOKEN_EXPIRED` para tomar acciones
    ```typescript
    avaPlayer.addEventListener(PlayerEvents.TOKEN_EXPIRED, () => {
        avaPlayer.setAccessToken('<YOUR-NEW-TOKEN>');
    });
    ```

El evento `TOKEN_EXPIRED` se producirá 5 segundos antes de que expire el token.  Si va a configurar una escucha de eventos, se recomienda que lo haga antes de llamar a la función `load` del widget del reproductor.

### <a name="configuration-details"></a>Detalles de configuración

Hemos hecho una configuración sencilla para el reproductor anterior, pero admite una gama más amplia de opciones para los valores de configuración.  A continuación se muestran los campos admitidos:

| Nombre   | Tipo             | Descripción                         |
| ------ | ---------------- | ----------------------------------- |
| token  | string | Token JWT del widget |
| videoName | string | Nombre del recurso de vídeo  |
| clientApiEndpointUrl | string | Dirección URL del punto de conexión de la API de cliente |

### <a name="alternate-ways-to-load-the-code-into-your-application"></a>Formas alternativas de cargar el código en la aplicación

El paquete que se usa para obtener el código en la aplicación es un paquete NPM que puede encontrar [aquí](https://www.npmjs.com/package/video-analyzer-widgets).  Aunque en el ejemplo anterior se cargó la versión más reciente en tiempo de ejecución directamente desde el repositorio, también puede descargar e instalar el paquete localmente mediante:

```bash
npm install @azure/video-analyzer/widgets
```

O bien, puede importarlo dentro del código de la aplicación mediante:

```typescript
import { Player } from '@video-analyzer/widgets';
```

Si usa este método para importar, deberá crear el objeto del reproductor mediante programación una vez completada la importación.  En el ejemplo anterior, agregó el módulo a la página mediante la etiqueta HTML `ava-player`.  Para crear un objeto de reproductor mediante código, puede hacer lo siguiente en JavaScript:

```javascript
const avaPlayer = new ava.widgets.player();
```

O en TypeScript:

```typescript
const avaPlayer = new Player();
```

A continuación, debe agregarlo al código HTML:

```javascript
document.firstElementChild.appendChild(avaPlayer);
```

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre la [API de widget](https://github.com/Azure/video-analyzer/widgets)
