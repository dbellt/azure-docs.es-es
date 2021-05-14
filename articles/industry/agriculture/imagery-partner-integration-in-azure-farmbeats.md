---
title: Integración de asociados de imágenes
description: En este artículo se describe la integración de asociados de imágenes.
author: RiyazPishori
ms.topic: article
ms.date: 11/04/2019
ms.author: riyazp
ms.custom: has-adal-ref
ms.openlocfilehash: adef1cca40ab93555bb6bccb55b8a425689b7e89
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108768830"
---
# <a name="imagery-partner-integration"></a>Integración de asociados de imágenes

En este artículo se describe cómo usar el componente traductor de Azure FarmBeats para enviar datos de imágenes a FarmBeats. Los datos de imágenes agrarias pueden generarse desde diversos orígenes, como cámaras multiespectrales, satélites y drones. Los asociados de imágenes agrarias pueden integrarse con FarmBeats para proporcionar a los clientes mapas generados de forma personalizada para sus granjas.

Los datos, una vez disponibles, pueden visualizarse mediante el acelerador de FarmBeats y usarse para la fusión de datos y la creación de un modelo de aprendizaje automático/inteligencia artificial (ML/IA) por parte de las empresas agrarias o los integradores de sistemas de clientes.

FarmBeats ofrece la posibilidad de:

- Definir los tipos, el origen y el formato de archivo de imágenes personalizadas mediante instancias de /ExtendedType API.
- Ingerir datos de imágenes de varios orígenes mediante instancias de /Scene API y /SceneFile API.

La siguiente información va dirigida a obtener cualquier forma de imagen en el sistema FarmBeats.

Al seleccionar la sección de **imágenes de dron**, se abre un elemento emergente que muestra una imagen de alta resolución del ortomosaico del dron. Puede acceder al software del asociado, que ayuda a planear vuelos de dron y obtener datos sin procesar. El software del asociado se seguirá usando para el planeamiento de la ruta y la unión de imágenes de ortomosaico.

Los asociados del dron deben permitir que los clientes vinculen su cuenta a su instancia de FarmBeats en Azure.

Es necesario usar las siguientes credenciales en el software del asociado del dron para vincular FarmBeats:

- Punto de conexión de API
- Id. de inquilino
- Id. de cliente
- Secreto del cliente

## <a name="api-development"></a>Desarrollo de API

Las API contienen documentación técnica de Swagger. Consulte [Swagger](https://aka.ms/FarmBeatsDatahubSwagger) para obtener información sobre las API y las solicitudes y respuestas correspondientes.

## <a name="authentication"></a>Authentication

FarmBeats usa Microsoft Azure [Active Directory](../../app-service/overview-authentication-authorization.md) (Azure AD). Azure App Service proporciona compatibilidad integrada con la autenticación y la autorización. 

Para más información sobre Azure AD, vea [Azure Active Directory](../../app-service/overview-authentication-authorization.md).   

El centro de datos de FarmBeats usa la autenticación de portador, que necesita las credenciales siguientes:

- Id. de cliente
- Secreto del cliente
- Id. de inquilino

Con las credenciales anteriores, el autor de la llamada puede solicitar un token de acceso, que debe enviarse en las sucesivas solicitudes de API en la sección de encabezado de la siguiente manera:

```
headers = {"Authorization": "Bearer " + access_token, …} 
```

El ejemplo de código de Python siguiente recupera el token de acceso. El token se puede usar luego en las sucesivas llamadas API a FarmBeats.

```python
import requests
import json
import msal

# Your service principal App ID
CLIENT_ID = "<CLIENT_ID>"
# Your service principal password
CLIENT_SECRET = "<CLIENT_SECRET>"
# Tenant ID for your Azure subscription
TENANT_ID = "<TENANT_ID>"

AUTHORITY_HOST = 'https://login.microsoftonline.com'
AUTHORITY = AUTHORITY_HOST + '/' + TENANT_ID

ENDPOINT = "https://<yourfarmbeatswebsitename-api>.azurewebsites.net"
SCOPE = ENDPOINT + "/.default"

context = msal.ConfidentialClientApplication(CLIENT_ID, authority=AUTHORITY, client_credential=CLIENT_SECRET)
token_response = context.acquire_token_for_client(SCOPE)
# We should get an access token here
access_token = token_response.get('access_token')
```

## <a name="http-request-headers"></a>Encabezados de solicitud HTTP

Estos son los encabezados de solicitud más comunes que deben especificarse al realizar una llamada API al centro de datos de FarmBeats.

**Encabezado** | **Descripción y ejemplo**
--- | ---
Content-Type  | El formato de la solicitud (Content-Type: application/<format>). En el caso de las API de centro de datos de FarmBeats, el formato es JSON. Content-Type: application/json
Authorization | Especifica el token de acceso necesario para realizar una llamada API. Autorización: Bearer <Access-Token>
Accept  | El formato de respuesta. En el caso de las API de centro de datos de FarmBeats, el formato es JSON. Accept: application/json


## <a name="api-requests"></a>Solicitudes de API

Para realizar una solicitud de API REST, combine:

- El Método HTTP (GET, POST y PUT).
- La dirección URL del servicio de API.
- El URI del recurso (para consultar, enviar datos, actualizar o eliminar).
- Uno o varios encabezados de solicitud HTTP.

También se pueden incluir parámetros de consulta en las llamadas GET para filtrar y ordenar los datos en las respuestas y limitar su tamaño.

La siguiente solicitud de ejemplo se usa para obtener la lista de dispositivos:

```bash
curl -X GET "https://microsoft-farmbeats.azurewebsites.net/Device" -H
"Content-Type: application/json" -H
"Authorization: Bearer <Access-Token>”
```

La mayoría de llamadas GET, POST y PUT requieren un cuerpo de la solicitud JSON.

La siguiente solicitud de ejemplo se usa para crear un dispositivo. Este ejemplo tiene un código JSON de entrada con el cuerpo de la solicitud.


```bash
curl -X POST "https://microsoft-farmbeats.azurewebsites.net/Device" -H
"accept: application/json" -H
"Content-Type: application/json" -H "Authorization: Bearer <Access-Token>" -d
"{  \"deviceModelId\": \"ID123\",  \"hardwareId\": \"MHDN123\",  \"reportingInterval\": 900,
\"name\": \"Device123\",  \"description\": \"Test Device 123\",}"
```

## <a name="data-format"></a>Formato de datos

JSON es un formato de datos común independiente del lenguaje que proporciona una representación de texto simple de estructuras de datos arbitrarias. Para más información, consulte [JSON.org](https://JSON.org).

## <a name="ingest-imagery-into-farmbeats"></a>Ingesta de imágenes en FarmBeats

Una vez que el asociado tiene credenciales para conectarse al centro de datos de FarmBeats, realiza los siguientes pasos en el componente traductor.

1.  Crea un tipo extendido para los siguientes campos, de acuerdo con el tipo de imagen que se va a cargar:

    - **Scene Source** (Origen de escena): por ejemplo, drone_partner_name.
    - **Scene Type** (Tipo de escena): por ejemplo, dron.
    - **Scene File Type** (Tipo de archivo de escena): por ejemplo, índice de clorofila.
    - **Scene File Content Type** (Tipo de contenido del archivo de escena): por ejemplo, image/tiff.

2.  Llama a /Farms API para obtener la lista de granjas del sistema Azure FarmBeats.
3.  Proporciona al cliente la posibilidad de elegir una sola granja de la lista.

    El sistema del asociado debe mostrar la granja en el software del asociado para planear la ruta y el vuelo del dron, así como para recopilar imágenes.

4.  Llama a /Scene API y proporciona los detalles necesarios para crear una escena con un identificador único.
5.  Recibe una dirección URL de SAS de blob para cargar las imágenes necesarias en el centro de datos de FarmBeats, en el contexto de la granja elegida, en el sistema FarmBeats.

A continuación se muestra un flujo detallado de las llamadas API.

### <a name="step-1-extendedtype"></a>Paso 1: ExtendedType

Compruebe en /ExtendedType API si el tipo y el origen del archivo están disponibles en FarmBeats. Para ello, llame a GET en /ExtendedType API.

A continuación se indican los valores definidos por el sistema:

```json
{
  "items": [
    {
      "id": "ae2aa527-7d27-42e2-8fcf-841937c651d7",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:47:42.5910344Z",
      "key": "SceneFileType",
      "value": [
        "evi",
        "ndvi",
        "ndwi",
        "tci",
        "soil-moisture",
        "sensor-placement",
        "sentinel-b01",
        "sentinel-b02",
        "sentinel-b03",
        "sentinel-b04",
        "sentinel-b05",
        "sentinel-b06",
        "sentinel-b07",
        "sentinel-b08",
        "sentinel-b8a",
        "sentinel-b09",
        "sentinel-b10",
        "sentinel-b11",
        "sentinel-b12",
        "cloud-mask",
        "farm-mask",
        "image/tiff",
        "orthomosaic"
      ],
      "description": "List of scene file types available in system. User can add more values."
    },
    {
      "id": "b7824d2e-3e43-46d3-8fd4-ec7ba8918d84",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:45:44.8914691Z",
      "key": "SceneFileContentType",
      "value": [
        "image/tiff",
        "image/png",
        "image/jpeg",
        "text/csv",
        "text/plain",
        "text/tab-separated-values",
        "application/json",
        "application/octet-stream",
        "orthomosaic"
      ],
      "description": "List of scene file content types available in system. User can add more values."
    },
    {
      "id": "21d2924d-8b3e-4ce4-a7f8-318e85c0f7f2",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:45:37.5096445Z",
      "key": "SceneSource",
      "value": [
        "sentinel-l1c",
        "sentinel-l2a",
        "farmbeats-model",
        "dji",
        "SlantRange-3P",
        "DJI"
      ],
      "description": "List of scene sources available in system. User can add more values."
    },
    {
      "id": "6e09ca69-cacf-4b53-b63a-53c659aae4a4",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:45:34.7483899Z",
      "key": "SceneType",
      "value": [
        "base-bands",
        "sensor-placement",
        "soil-moisture",
        "evi",
        "ndwi",
        "ndvi",
        "drone",
        "orthomosaic-drone"
      ],
      "description": "List of scene types available in system. User can add more values."
    },
    {
      "id": "55d8436b-3a86-4bea-87ab-e1b51226525f",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-06-17T07:32:30.1135716Z",
      "key": "SensorMeasureType",
      "value": [
        "AmbientTemperature",
        "CO2",
        "Depth",
        "ElectricalConductivity",
        "LeafWetness",
        "Length",
        "LiquidLevel",
        "Nitrate",
        "O2",
        "PH",
        "Phosphate",
        "PointInTime",
        "Potassium",
        "Pressure",
        "RainGauge",
        "RelativeHumidity",
        "Salinity",
        "SoilMoisture",
        "SoilTemperature",
        "SolarRadiation",
        "State",
        "TimeDuration",
        "UVRadiation",
        "UVIndex",
        "Volume",
        "WindDirection",
        "WindRun",
        "WindSpeed",
        "Evapotranspiration",
        "PAR"
      ],
      "description": "List of sensor measure types available in system. User can add more values."
    },
    {
      "id": "0dfd6e6b-df58-428f-9ab8-a0674bfdcbe5",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-06-17T07:32:30.1135716Z",
      "key": "SensorMeasureUnit",
      "value": [
        "NoUnit",
        "Celsius",
        "Fahrenheit",
        "Kelvin",
        "Rankine",
        "Pascal",
        "Mercury",
        "PSI",
        "MilliMeter",
        "CentiMeter",
        "Meter",
        "Inch",
        "Feet",
        "Mile",
        "KiloMeter",
        "MilesPerHour",
        "MilesPerSecond",
        "KMPerHour",
        "KMPerSecond",
        "MetersPerHour",
        "MetersPerSecond",
        "Degree",
        "WattsPerSquareMeter",
        "KiloWattsPerSquareMeter",
        "MilliWattsPerSquareCentiMeter",
        "MilliJoulesPerSquareCentiMeter",
        "VolumetricWaterContent",
        "Percentage",
        "PartsPerMillion",
        "MicroMol",
        "MicroMolesPerLiter",
        "SiemensPerSquareMeterPerMole",
        "MilliSiemensPerCentiMeter",
        "Centibar",
        "DeciSiemensPerMeter",
        "KiloPascal",
        "VolumetricIonContent",
        "Liter",
        "MilliLiter",
        "Seconds",
        "UnixTimestamp",
        "MicroMolPerMeterSquaredPerSecond",
        "InchesPerHour"
      ],
      "description": "List of sensor measure units available in system. User can add more values."
    }
  ]
}
```

Este paso de configuración solo se realiza una vez. El ámbito de este nuevo tipo de escena se limita a la suscripción en la que se instala FarmBeats de Azure.

Por ejemplo, para agregar SceneSource: "SlantRange", realiza una operación PUT en el identificador de /ExtendedType API con la carga de entrada de la clave "SceneSource".

```json
{

      "key": "SceneSource",
      "value": [
        "sentinel-l1c",
        "sentinel-l2a",
        "farmbeats-model",
        "dji",
        "SlantRange-3P"
      ],
      "description": "List of scene sources available in system. User can add more values."
}

```

El campo verde es la nueva adición a los valores de origen de la escena definidos por el sistema.

### <a name="step-2-get-farm-details"></a>Paso 2: Obtención de detalles de la granja

Las escenas (archivos .tiff o .csv) se sitúan en el contexto de una granja. Para obtener los detalles de la granja, realice una operación GET en /Farm API. La API devuelve la lista de granjas que están disponibles en FarmBeats. Puede seleccionar la granja para la que desea ingerir los datos.

Respuesta de /Farm de GET:

```json
{
  "id": "07f3e09c-89aa-4619-8d50-e57fb083d8f9",
  "createdAt": "2019-11-01T13:55:41.8804663Z",
  "lastModifiedAt": "2019-11-01T13:55:41.8804663Z",
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [
        [
          78.34252251428694,
          17.402556352862675
        ],
        [
          78.34278000635095,
          17.407920852463022
        ],
        [
          78.34883106989963,
          17.40878079576528
        ],
        [
          78.3498181228054,
          17.403539173730607
        ],
        [
          78.34805859369493,
          17.39977166504127
        ],
        [
          78.34252251428694,
          17.402556352862675
        ]
      ]
    ]
  },
  "name": "Samplefarm",
  "properties": {
    "crops": "Corn",
    "address": "Sample street"
  }
}
 ```

### <a name="step-3-create-a-scene-id-post-call"></a>Paso 3: Creación de un identificador de escena (llamada POST)

Cree una nueva escena (archivo .tiff o .csv) con la información dada, que proporcione la fecha, la secuencia y el identificador de la granja con la que se asocia. Los metadatos asociados a la escena pueden definirse en las propiedades, que incluyen la duración y el tipo de medida.

Al crear una nueva escena, se crea un nuevo identificador de escena que está asociado a la granja. Una vez creado el valor del identificador de escena, el usuario puede usarlo para crear un nuevo archivo (.tiff o .csv) y almacenar su contenido.

Ejemplo de carga de entrada para la llamada POST en /Scene API:

```json
{
  "sceneId": "a0505928-c480-491b-ba31-d38285a28c1d",
  "type": "newtype",
  "contentType": "image/tiff",
  "name": "test scene file",
  "description": "test scene file description",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```

Respuesta de la API:

```json
{
  "id": "a0505928-c480-491b-ba31-d38285a28c1d",
  "createdAt": "2019-10-04T16:19:12.4838584Z",
  "lastModifiedAt": "2019-10-04T16:19:12.4838584Z",
  "type": "new type",
  "source": "SlantRange-3P",
  "farmId": "d41a33e7-b73e-480e-9279-0fcb3207332b",
  "date": "2019-10-04T16:13:39.064Z",
  "sequence": 5,
  "name": "test scene",
  "description": "test scene description",
  "properties": {}
}

```

**Creación de un archivo de escena**

El identificador de escena devuelto en el paso 3 es la entrada del archivo de escena. El archivo de escena devuelve un token de dirección URL de SAS, que es válido durante 24 horas.

Si el usuario necesita una manera programática de cargar una secuencia de imágenes, se puede usar el SDK de almacenamiento de blobs para definir un método mediante el identificador, la ubicación y la dirección URL del archivo de escena.

Ejemplo de carga de entrada para la llamada POST en /SceneFile API:

```json
{
  "sceneId": "a0505928-c480-491b-ba31-d38285a28c1d",
  "type": "newtype",
  "contentType": "image/tiff",
  "name": "test scene file",
  "description": "test scene file description",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```
Respuesta de la API:

```json
{
  "uploadSASUrl": "https://storagej2lho.blob.core.windows.net/farm-scene/2019/a0505928-c480-491b-ba31-d38285a28c1d/e91139a7-4ebd-4e2f-b17c-c677822dc840?sv=2018-03-28&sr=b&sig=%2F1426JkDcIFE5g3d%2BjOevCVMIn%2FJo9YKwBn3La5zL8Y%3D&se=2019-10-05T16%3A23%3A57Z&sp=w",
  "id": "e91139a7-4ebd-4e2f-b17c-c677822dc840",
  "createdAt": "2019-10-04T16:23:57.1192916Z",
  "lastModifiedAt": "2019-10-04T16:23:57.1192916Z",
  "blobUrl": "https://storagej2lho.blob.core.windows.net/farm-scene/2019/a0505928-c480-491b-ba31-d38285a28c1d/e91139a7-4ebd-4e2f-b17c-c677822dc840",
  "sceneId": "a0505928-c480-491b-ba31-d38285a28c1d",
  "type": "newtype",
  "contentType": "image/tiff",
  "name": "test scene file",
  "description": "test scene file description",
  "properties": {}
}


```

La llamada POST a /SceneFile API devuelve una dirección URL de carga de SAS, que se puede usar para cargar el archivo .csv o .tiff mediante el cliente o la biblioteca de Azure Blob Storage.


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la integración basada en la API REST, consulte [API REST](rest-api-in-azure-farmbeats.md).