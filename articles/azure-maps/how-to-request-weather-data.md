---
title: Solicitud de datos meteorológicos en tiempo real y previsiones mediante los servicios meteorológicos de Azure Maps
description: Aprenda a solicitar datos meteorológicos en tiempo real (actuales) y previsiones (minuto, hora, día) con los servicios meteorológicos de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 46d9847e8463d5413409ebcacc7cd62d68f13e65
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107259411"
---
# <a name="request-real-time-and-forecasted-weather-data-using-azure-maps-weather-services"></a>Solicitud de datos meteorológicos en tiempo real y previsiones mediante los servicios meteorológicos de Azure Maps

El [servicio Weather](/rest/api/maps/weather) de Azure Maps es un conjunto de API de RESTful que permite a los desarrolladores integrar datos y visualizaciones meteorológicos sumamente dinámicos históricos, en tiempo real y previstos en sus soluciones. En este artículo, se muestra cómo solicitar datos meteorológicos en tiempo real y previsiones.

Aprenderá a:

* Solicitar datos meteorológicos en tiempo real (actuales) mediante [Get Current Conditions API](/rest/api/maps/weather/getcurrentconditions).
* Solicitar alertas meteorológicas graves mediante [Get Severe Weather Alerts API](/rest/api/maps/weather/getsevereweatheralerts).
* Solicitar previsiones diarias mediante [Get Daily Forecast API](/rest/api/maps/weather/getdailyforecast).
* Solicitar previsiones por hora mediante [Get Hourly Forecast API](/rest/api/maps/weather/gethourlyforecast).
* Solicitar previsiones minuto a minuto mediante [Get Minute Forecast API](/rest/api/maps/weather/getminuteforecast).

En este vídeo se proporcionan ejemplos para realizar llamadas de REST al servicio Weather de Azure Maps.

</br>

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Azure-Maps-Weather-services-for-developers/player?format=ny]

## <a name="prerequisites"></a>Requisitos previos

1. [Cree una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).
2. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción. Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](./how-to-manage-authentication.md).

    >[!IMPORTANT]
    >[Get minute Forecast API](/rest/api/maps/weather/getminuteforecast) requiere una clave de plan de tarifa S1. Todas las demás API requieren una clave de plan de tarifa S0.

En este tutorial se usa la aplicación [Postman](https://www.postman.com/), pero puede elegir un entorno de desarrollo de API diferente.

## <a name="request-real-time-weather-data"></a>Solicitud de datos meteorológicos en tiempo real

[Get Current Conditions API](/rest/api/maps/weather/getcurrentconditions) devuelve condiciones meteorológicas detalladas, como precipitaciones, temperatura y viento para una ubicación de coordenadas determinada. Además, se pueden recuperar las observaciones de las últimas 6 o 24 horas para una ubicación determinada. La respuesta incluye detalles como la fecha y la hora de la observación, una breve descripción de las condiciones meteorológicas, el icono del tiempo, las marcas del indicador de precipitación y la temperatura. También se devuelven la temperatura y el índice de rayos UVA de RealFeel™.

En este ejemplo, usará [Get Current Conditions API](/rest/api/maps/weather/getcurrentconditions) para recuperar las condiciones meteorológicas actuales en las coordenadas ubicadas en Seattle, WA.

1. Abra la aplicación Postman. Cerca de la parte superior de la aplicación Postman, seleccione **New** (Nuevo). En la ventana **Create New** (Crear nuevo), seleccione **Collection** (Colección).  Asigne un nombre a la colección y seleccione el botón **Create** (Crear). Usará esta colección para el resto de los ejemplos de este documento.

2. Para crear la solicitud, seleccione **New** (Nuevo) otra vez. En la ventana **Create New** (Crear nuevo), seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en el paso anterior y haga clic en **Save** (Guardar).

3. Seleccione el método **GET** HTTP en la pestaña del generador y escriba la dirección URL siguiente. Para esta solicitud y otras solicitudes mencionadas en este artículo, reemplace `{Azure-Maps-Primary-Subscription-key}` por su clave de suscripción principal.

    ```http
    https://atlas.microsoft.com/weather/currentConditions/json?api-version=1.0&query=47.60357,-122.32945&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

4. Haga clic en el botón azul **Enviar**. El cuerpo de la respuesta contiene información meteorológica actual.

    ```json
    {
    "results": [
        {
            "dateTime": "2020-10-19T20:39:00+00:00",
            "phrase": "Cloudy",
            "iconCode": 7,
            "hasPrecipitation": false,
            "isDayTime": true,
            "temperature": {
                "value": 12.4,
                "unit": "C",
                "unitType": 17
            },
            "realFeelTemperature": {
                "value": 13.7,
                "unit": "C",
                "unitType": 17
            },
            "realFeelTemperatureShade": {
                "value": 13.7,
                "unit": "C",
                "unitType": 17
            },
            "relativeHumidity": 87,
            "dewPoint": {
                "value": 10.3,
                "unit": "C",
                "unitType": 17
            },
            "wind": {
                "direction": {
                    "degrees": 23.0,
                    "localizedDescription": "NNE"
                },
                "speed": {
                    "value": 4.5,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "windGust": {
                "speed": {
                    "value": 9.0,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "uvIndex": 1,
            "uvIndexPhrase": "Low",
            "visibility": {
                "value": 9.7,
                "unit": "km",
                "unitType": 6
            },
            "obstructionsToVisibility": "",
            "cloudCover": 100,
            "ceiling": {
                "value": 1494.0,
                "unit": "m",
                "unitType": 5
            },
            "pressure": {
                "value": 1021.2,
                "unit": "mb",
                "unitType": 14
            },
            "pressureTendency": {
                "localizedDescription": "Steady",
                "code": "S"
            },
            "past24HourTemperatureDeparture": {
                "value": -2.1,
                "unit": "C",
                "unitType": 17
            },
            "apparentTemperature": {
                "value": 15.0,
                "unit": "C",
                "unitType": 17
            },
            "windChillTemperature": {
                "value": 12.2,
                "unit": "C",
                "unitType": 17
            },
            "wetBulbTemperature": {
                "value": 11.3,
                "unit": "C",
                "unitType": 17
            },
            "precipitationSummary": {
                "pastHour": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past3Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past6Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past9Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past12Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past18Hours": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "past24Hours": {
                    "value": 0.4,
                    "unit": "mm",
                    "unitType": 3
                }
            },
            "temperatureSummary": {
                "past6Hours": {
                    "minimum": {
                        "value": 12.2,
                        "unit": "C",
                        "unitType": 17
                    },
                    "maximum": {
                        "value": 14.0,
                        "unit": "C",
                        "unitType": 17
                    }
                },
                "past12Hours": {
                    "minimum": {
                        "value": 12.2,
                        "unit": "C",
                        "unitType": 17
                    },
                    "maximum": {
                        "value": 14.0,
                        "unit": "C",
                        "unitType": 17
                    }
                },
                "past24Hours": {
                    "minimum": {
                        "value": 12.2,
                        "unit": "C",
                        "unitType": 17
                    },
                    "maximum": {
                        "value": 15.6,
                        "unit": "C",
                        "unitType": 17
                    }
                }
            }
        }
    ]
    }
    ```

## <a name="request-severe-weather-alerts"></a>Solicitud de alertas meteorológicas graves

[Get Severe Weather Alerts API de Azure Maps](/rest/api/maps/weather/getsevereweatheralerts) devuelve las alertas meteorológicas graves que están disponibles en todo el mundo desde las agencias meteorológicas gubernamentales oficiales y los principales proveedores de alertas meteorológicas globales y regionales. El servicio puede devolver información detallada, como el tipo de alerta, la categoría y el nivel, además de descripciones detalladas sobre las alertas graves activas en la ubicación solicitada, como huracanes, tormentas eléctricas, olas de calor o incendios forestales. Por ejemplo, los administradores de logística pueden visualizar las condiciones meteorológicas graves en un mapa, junto con las ubicaciones de la empresa y las rutas planeadas, y coordinarse mejor con los conductores y los trabajadores locales.

En este ejemplo, usará [Get Severe Weather Alerts API](/rest/api/maps/weather/getsevereweatheralerts) para recuperar las condiciones meteorológicas actuales en las coordenadas ubicadas en Cheyenne, WY.

>[!NOTE]
>En este ejemplo se recuperan alertas meteorológicas graves en el momento de redactar este documento. Es probable que ya no haya ninguna alerta meteorológica grave en la ubicación solicitada. Para recuperar datos de alertas graves reales al ejecutar este ejemplo, debe recuperar los datos de una ubicación de coordenadas diferente.

1. Abra la aplicación Postman, haga clic en **New** (Nuevo) y seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en la sección anterior o cree una nueva y, a continuación, seleccione **Save** (Guardar).

2. Seleccione el método **GET** HTTP en la pestaña del generador y escriba la dirección URL siguiente. Para esta solicitud y otras solicitudes mencionadas en este artículo, reemplace `{Azure-Maps-Primary-Subscription-key}` por su clave de suscripción principal.

    ```http
    https://atlas.microsoft.com/weather/severe/alerts/json?api-version=1.0&query=41.161079,-104.805450&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Haga clic en el botón azul **Enviar**. Si no hay ninguna alerta meteorológica grave, el cuerpo de la respuesta contendrá una matriz `results[]` vacía. Si hay alertas meteorológicas graves, el cuerpo de la respuesta contiene algo parecido a la siguiente respuesta JSON:

    ```json
    {
    "results": [
        {
            "countryCode": "US",
            "alertId": 2194734,
            "description": {
                "localized": "Red Flag Warning",
                "english": "Red Flag Warning"
            },
            "category": "FIRE",
            "priority": 54,
            "source": "U.S. National Weather Service",
            "sourceId": 2,
            "alertAreas": [
                {
                    "name": "Platte/Goshen/Central and Eastern Laramie",
                    "summary": "Red Flag Warning in effect until 7:00 PM MDT.  Source: U.S. National Weather Service",
                    "startTime": "2020-10-05T15:00:00+00:00",
                    "endTime": "2020-10-06T01:00:00+00:00",
                    "latestStatus": {
                        "localized": "Continue",
                        "english": "Continue"
                    },
                    "alertDetails": "...RED FLAG WARNING REMAINS IN EFFECT FROM 9 AM THIS MORNING TO\n7 PM MDT THIS EVENING FOR STRONG GUSTY WINDS AND LOW HUMIDITY...\n\n* WHERE...Fire weather zones 303, 304, 305, 306, 307, 308, 309,\n  and 310 in southeast Wyoming. Fire weather zone 313 in Nebraska.\n\n* WIND...West to northwest 15 to 30 MPH with gusts around 40 MPH.\n\n* HUMIDITY...10 to 15 percent.\n\n* IMPACTS...Any fires that develop will likely spread rapidly.\n  Outdoor burning is not recommended.\n\nPRECAUTIONARY/PREPAREDNESS ACTIONS...\n\nA Red Flag Warning means that critical fire weather conditions\nare either occurring now...or will shortly. A combination of\nstrong winds...low relative humidity...and warm temperatures can\ncontribute to extreme fire behavior.\n\n&&",
                    "alertDetailsLanguageCode": "en"
                }
            ]
            },...
        ]
    }
    ```

## <a name="request-daily-weather-forecast-data"></a>Solicitud de datos de previsiones meteorológicas diarias

[Get Daily Forecast API](/rest/api/maps/weather/getdailyforecast) devuelve la previsión meteorológica diaria detallada, como la temperatura y el viento. La solicitud puede especificar para cuántos días se devolverá esta información: 1, 5, 10, 15, 25 o 45 días para una ubicación de coordenadas determinada. La respuesta incluye detalles como la temperatura, el viento, las precipitaciones, la calidad del aire y el índice de rayos UVA.  En este ejemplo, se establece `duration=5` para solicitar la información de cinco días.

>[!IMPORTANT]
>En el plan de tarifa S0, puede solicitar una previsión diaria para los próximos 1, 5, 10 y 15 días. En el plan de tarifa S1, también puede solicitar una previsión diaria para los próximos 25 y 45 días.

En este ejemplo, usará [Get Daily Forecast API](/rest/api/maps/weather/getdailyforecast) para recuperar la previsión meteorológica de cinco días para las coordenadas ubicadas en Seattle, WA.

1. Abra la aplicación Postman, haga clic en **New** (Nuevo) y seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en la sección anterior o cree una nueva y, a continuación, seleccione **Save** (Guardar).

2. Seleccione el método **GET** HTTP en la pestaña del generador y escriba la dirección URL siguiente. Para esta solicitud y otras solicitudes mencionadas en este artículo, reemplace `{Azure-Maps-Primary-Subscription-key}` por su clave de suscripción principal.

    ```http
    https://atlas.microsoft.com/weather/forecast/daily/json?api-version=1.0&query=47.60357,-122.32945&duration=5&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Haga clic en el botón azul **Enviar**. El cuerpo de la respuesta contiene los datos de previsión meteorológica de cinco días. Por motivos de brevedad, la respuesta JSON siguiente muestra la previsión para el primer día.
    ```json
    {
    "summary": {
        "startDate": "2020-10-18T17:00:00+00:00",
        "endDate": "2020-10-19T23:00:00+00:00",
        "severity": 2,
        "phrase": "Snow, mixed with rain at times continuing through Monday evening and a storm total of 3-6 cm",
        "category": "snow/rain"
    },
    "forecasts": [
        {
            "date": "2020-10-19T04:00:00+00:00",
            "temperature": {
                "minimum": {
                    "value": -1.1,
                    "unit": "C",
                    "unitType": 17
                },
                "maximum": {
                    "value": 1.3,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "realFeelTemperature": {
                "minimum": {
                    "value": -6.0,
                    "unit": "C",
                    "unitType": 17
                },
                "maximum": {
                    "value": 0.5,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "realFeelTemperatureShade": {
                "minimum": {
                    "value": -6.0,
                    "unit": "C",
                    "unitType": 17
                },
                "maximum": {
                    "value": 0.7,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "hoursOfSun": 1.8,
            "degreeDaySummary": {
                "heating": {
                    "value": 18.0,
                    "unit": "C",
                    "unitType": 17
                },
                "cooling": {
                    "value": 0.0,
                    "unit": "C",
                    "unitType": 17
                }
            },
            "airAndPollen": [
                {
                    "name": "AirQuality",
                    "value": 23,
                    "category": "Good",
                    "categoryValue": 1,
                    "type": "Ozone"
                },
                {
                    "name": "Grass",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "Mold",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "Ragweed",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "Tree",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                },
                {
                    "name": "UVIndex",
                    "value": 0,
                    "category": "Low",
                    "categoryValue": 1
                }
            ],
            "day": {
                "iconCode": 22,
                "iconPhrase": "Snow",
                "hasPrecipitation": true,
                "precipitationType": "Mixed",
                "precipitationIntensity": "Light",
                "shortPhrase": "Chilly with snow, 2-4 cm",
                "longPhrase": "Chilly with snow, accumulating an additional 2-4 cm",
                "precipitationProbability": 90,
                "thunderstormProbability": 0,
                "rainProbability": 54,
                "snowProbability": 85,
                "iceProbability": 8,
                "wind": {
                    "direction": {
                        "degrees": 36.0,
                        "localizedDescription": "NE"
                    },
                    "speed": {
                        "value": 9.3,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "windGust": {
                    "direction": {
                        "degrees": 70.0,
                        "localizedDescription": "ENE"
                    },
                    "speed": {
                        "value": 25.9,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "totalLiquid": {
                    "value": 4.3,
                    "unit": "mm",
                    "unitType": 3
                },
                "rain": {
                    "value": 0.5,
                    "unit": "mm",
                    "unitType": 3
                },
                "snow": {
                    "value": 2.72,
                    "unit": "cm",
                    "unitType": 4
                },
                "ice": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "hoursOfPrecipitation": 9.0,
                "hoursOfRain": 1.0,
                "hoursOfSnow": 9.0,
                "hoursOfIce": 0.0,
                "cloudCover": 96
            },
            "night": {
                "iconCode": 29,
                "iconPhrase": "Rain and snow",
                "hasPrecipitation": true,
                "precipitationType": "Mixed",
                "precipitationIntensity": "Light",
                "shortPhrase": "Showers of rain and snow",
                "longPhrase": "A couple of showers of rain or snow this evening; otherwise, cloudy; storm total snowfall 1-3 cm",
                "precipitationProbability": 65,
                "thunderstormProbability": 0,
                "rainProbability": 60,
                "snowProbability": 54,
                "iceProbability": 4,
                "wind": {
                    "direction": {
                        "degrees": 16.0,
                        "localizedDescription": "NNE"
                    },
                    "speed": {
                        "value": 16.7,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "windGust": {
                    "direction": {
                        "degrees": 1.0,
                        "localizedDescription": "N"
                    },
                    "speed": {
                        "value": 35.2,
                        "unit": "km/h",
                        "unitType": 7
                    }
                },
                "totalLiquid": {
                    "value": 4.3,
                    "unit": "mm",
                    "unitType": 3
                },
                "rain": {
                    "value": 3.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "snow": {
                    "value": 0.79,
                    "unit": "cm",
                    "unitType": 4
                },
                "ice": {
                    "value": 0.0,
                    "unit": "mm",
                    "unitType": 3
                },
                "hoursOfPrecipitation": 4.0,
                "hoursOfRain": 1.0,
                "hoursOfSnow": 3.0,
                "hoursOfIce": 0.0,
                "cloudCover": 94
            },
            "sources": [
                "AccuWeather"
            ]
        },...
    ]
    }
    ```

## <a name="request-hourly-weather-forecast-data"></a>Solicitud de datos de previsión meteorológica por hora

[Get Hourly Forecast API](/rest/api/maps/weather/gethourlyforecast) devuelve la previsión meteorológica detallada por hora para las próximas 1, 12, 24 (1 día), 72 (3 días), 120 (5 días) y 240 horas (10 días) para la ubicación de coordenadas dada. La API devuelve detalles como la temperatura, la humedad, el viento, la precipitación y el índice de rayos UVA.

>[!IMPORTANT]
>En el plan de tarifa S0, puede solicitar una previsión horaria para las próximas 1, 12, 24 horas (1 día) y 72 horas (3 días). En el plan de tarifa S1, también puede solicitar una previsión por hora para las próximas 120 (5 días) y 240 horas (10 días).

En este ejemplo, usará [Get Hourly Forecast API](/rest/api/maps/weather/gethourlyforecast) para recuperar la previsión meteorológica por hora para las próximas 12 horas en las coordenadas ubicadas en Seattle, WA.

1. Abra la aplicación Postman, haga clic en **New** (Nuevo) y seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en la sección anterior o cree una nueva y, a continuación, seleccione **Save** (Guardar).

2. Seleccione el método **GET** HTTP en la pestaña del generador y escriba la dirección URL siguiente. Para esta solicitud y otras solicitudes mencionadas en este artículo, reemplace `{Azure-Maps-Primary-Subscription-key}` por su clave de suscripción principal.

    ```http
    https://atlas.microsoft.com/weather/forecast/hourly/json?api-version=1.0&query=47.60357,-122.32945&duration=12&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Haga clic en el botón azul **Enviar**. El cuerpo de la respuesta contiene los datos de previsión meteorológica para las próximas 12 horas. Por motivos de brevedad, la respuesta JSON siguiente muestra la previsión para la primera hora.

    ```json
    {
    "forecasts": [
        {
            "date": "2020-10-19T21:00:00+00:00",
            "iconCode": 12,
            "iconPhrase": "Showers",
            "hasPrecipitation": true,
            "precipitationType": "Rain",
            "precipitationIntensity": "Light",
            "isDaylight": true,
            "temperature": {
                "value": 14.7,
                "unit": "C",
                "unitType": 17
            },
            "realFeelTemperature": {
                "value": 13.3,
                "unit": "C",
                "unitType": 17
            },
            "wetBulbTemperature": {
                "value": 12.0,
                "unit": "C",
                "unitType": 17
            },
            "dewPoint": {
                "value": 9.5,
                "unit": "C",
                "unitType": 17
            },
            "wind": {
                "direction": {
                    "degrees": 242.0,
                    "localizedDescription": "WSW"
                },
                "speed": {
                    "value": 9.3,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "windGust": {
                "speed": {
                    "value": 14.8,
                    "unit": "km/h",
                    "unitType": 7
                }
            },
            "relativeHumidity": 71,
            "visibility": {
                "value": 9.7,
                "unit": "km",
                "unitType": 6
            },
            "cloudCover": 100,
            "ceiling": {
                "value": 1128.0,
                "unit": "m",
                "unitType": 5
            },
            "uvIndex": 1,
            "uvIndexPhrase": "Low",
            "precipitationProbability": 51,
            "rainProbability": 51,
            "snowProbability": 0,
            "iceProbability": 0,
            "totalLiquid": {
                "value": 0.3,
                "unit": "mm",
                "unitType": 3
            },
            "rain": {
                "value": 0.3,
                "unit": "mm",
                "unitType": 3
            },
            "snow": {
                "value": 0.0,
                "unit": "cm",
                "unitType": 4
            },
            "ice": {
                "value": 0.0,
                "unit": "mm",
                "unitType": 3
            }
        }...
    ]
    }
    ```
## <a name="request-minute-by-minute-weather-forecast-data"></a>Solicitud de datos de previsión meteorológica minuto a minuto

 [Get Minute Forecast API](/rest/api/maps/weather/getminuteforecast) devuelve las previsiones minuto a minuto de una ubicación determinada para los próximos 120 minutos. Los usuarios pueden solicitar previsiones meteorológicas en intervalos de 1, 5 y 15 minutos. La respuesta incluye detalles como el tipo de precipitación (como lluvia, nieve o una mezcla de ambos), la hora de inicio y el valor de intensidad de la precipitación (dBZ).

En este ejemplo, usará [Get Minute Forecast API](/rest/api/maps/weather/getminuteforecast) para recuperar la previsión meteorológica minuto a minuto en las coordenadas ubicadas en Seattle, WA. La previsión meteorológica se proporciona para los próximos 120 minutos. Nuestra consulta solicita que la previsión se proporcione a intervalos de 15 minutos, pero puede ajustar el parámetro para que sea de 1 o 5 minutos.

1. Abra la aplicación Postman, haga clic en **New** (Nuevo) y seleccione **Request** (Solicitud). Escriba un valor de **Request name** (Nombre de solicitud) para la solicitud. Seleccione la colección que creó en la sección anterior o cree una nueva y, a continuación, seleccione **Save** (Guardar).

2. Seleccione el método **GET** HTTP en la pestaña del generador y escriba la dirección URL siguiente. Para esta solicitud y otras solicitudes mencionadas en este artículo, reemplace `{Azure-Maps-Primary-Subscription-key}` por su clave de suscripción principal.

    ```http
    https://atlas.microsoft.com/weather/forecast/minute/json?api-version=1.0&query=47.60357,-122.32945&interval=15&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Haga clic en el botón azul **Enviar**. El cuerpo de respuesta contiene datos de previsión meteorológica para los próximos 120 minutos, en intervalos de 15 minutos.

    ```json
    {
    "summary": {
        "briefPhrase60": "No precipitation for at least 60 min",
        "shortPhrase": "No precip for 120 min",
        "briefPhrase": "No precipitation for at least 120 min",
        "longPhrase": "No precipitation for at least 120 min",
        "iconCode": 7
    },
    "intervalSummaries": [
        {
            "startMinute": 0,
            "endMinute": 119,
            "totalMinutes": 120,
            "shortPhrase": "No precip for %MINUTE_VALUE min",
            "briefPhrase": "No precipitation for at least %MINUTE_VALUE min",
            "longPhrase": "No precipitation for at least %MINUTE_VALUE min",
            "iconCode": 7
        }
    ],
    "intervals": [
        {
            "startTime": "2020-10-19T20:51:00+00:00",
            "minute": 0,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:06:00+00:00",
            "minute": 15,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:21:00+00:00",
            "minute": 30,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:36:00+00:00",
            "minute": 45,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T21:51:00+00:00",
            "minute": 60,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T22:06:00+00:00",
            "minute": 75,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T22:21:00+00:00",
            "minute": 90,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        },
        {
            "startTime": "2020-10-19T22:36:00+00:00",
            "minute": 105,
            "dbz": 0.0,
            "shortPhrase": "No Precipitation",
            "iconCode": 7,
            "cloudCover": 100
        }
        ]
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Conceptos de los servicios Weather de Azure Maps](./weather-services-concepts.md)

> [!div class="nextstepaction"]
> [API REST de los servicios Weather de Azure Maps](/rest/api/maps/weather)