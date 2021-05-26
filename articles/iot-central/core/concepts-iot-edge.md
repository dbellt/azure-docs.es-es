---
title: Azure IoT Edge y Azure IoT Central | Microsoft Docs
description: Aprenda a usar Azure IoT Edge con una aplicación de IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 02/19/2021
ms.topic: conceptual
ms.service: iot-central
services: iot-central
ms.custom:
- device-developer
- iot-edge
ms.openlocfilehash: 2233e6f40c1023f1b02543f4e234b00422f6f77f
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110077201"
---
# <a name="connect-azure-iot-edge-devices-to-an-azure-iot-central-application"></a>Conexión de dispositivos de Azure IoT Edge a una aplicación de Azure IoT Central

Azure IoT Edge mueve el análisis en la nube y la lógica de negocios personalizada a los dispositivos para que la organización pueda centrarse en las conclusiones empresariales, en lugar de en la administración de datos. Escale horizontalmente la solución de IoT mediante el empaquetado de la lógica de negocios en contenedores estándar, implemente esos contenedores en los dispositivos y supervíselos desde la nube.

En este artículo se describe:

* Procedimiento para conectar dispositivos IoT Edge a una aplicación de IoT Central.
* Procedimiento para usar IoT Central para administrar los dispositivos IoT Edge.

Para obtener más información acerca de IoT Edge, consulte [¿Qué es Azure IoT Edge?](../../iot-edge/about-iot-edge.md)

## <a name="iot-edge"></a>IoT Edge

IoT Edge está formado por tres componentes:

* Los *módulos de IoT Edge* son contenedores que ejecutan servicios de Azure, de asociados o código propio del usuario. Los módulos se implementan en los dispositivos de IoT Edge y se ejecutan de forma local en ellos. Para obtener más información, consulte [Información sobre los módulos Azure IoT Edge](../../iot-edge/iot-edge-modules.md).
* El *entorno en tiempo de ejecución de IoT Edge* se ejecuta en todos los dispositivos de IoT Edge y administra los módulos que se implementan en cada dispositivo. El tiempo de ejecución consta de dos módulos de IoT Edge: *agente de IoT Edge* y *centro de IoT Edge*. Para obtener más información, consulte [Información del entorno de ejecución de Azure IoT Edge y su arquitectura](../../iot-edge/iot-edge-runtime.md).
* Una *interfaz basada en la nube* permite supervisar y administrar los dispositivos de IoT Edge de forma remota. IoT Central es un ejemplo de una interfaz en la nube.

Un dispositivo IoT Edge puede ser:

* un dispositivo independiente compuesto por módulos;
* un *dispositivo de puerta de enlace* con dispositivos de nivel inferior que se conectan a él.

## <a name="iot-edge-as-a-gateway"></a>IoT Edge como puerta de enlace

Un dispositivo IoT Edge puede funcionar como una puerta de enlace que proporciona una conexión entre otros dispositivos de nivel inferior en la red y la aplicación de IoT Central.

Hay dos patrones de puerta de enlace:

* En el patrón de *puerta de enlace transparente*, el módulo del centro de IoT Edge se comporta como IoT Central y controla las conexiones de los dispositivos registrados en IoT Central. Los mensajes pasan de los dispositivos de nivel inferior a IoT Central como si no hubiera ninguna puerta de enlace entre ellos.

* En el patrón de la *puerta de enlace de traducción*, los dispositivos que no se pueden conectar a IoT Central por su cuenta, se conectan a un módulo de IoT Edge personalizado. El módulo del dispositivo IoT Edge procesa los mensajes entrantes del dispositivo de nivel inferior y, a continuación, los reenvía a IoT Central.

Los patrones de puerta de enlace transparente y de traducción no se excluyen mutuamente. Un solo dispositivo IoT Edge puede funcionar como puerta de enlace transparente y como puerta de enlace de traducción.

Para obtener más información acerca de los patrones de puerta de enlace de IoT Edge, consulte [Uso de un dispositivo IoT Edge como puerta de enlace](../../iot-edge/iot-edge-as-gateway.md).

### <a name="downstream-device-relationships-with-a-gateway-and-modules"></a>Relaciones de dispositivo de nivel inferior con una puerta de enlace y módulos

Los dispositivos de nivel inferior pueden conectarse a un dispositivo de puerta de enlace IoT Edge mediante el módulo del *centro de IoT Edge*. En este escenario, el dispositivo IoT Edge es una puerta de enlace transparente:

:::image type="content" source="media/concepts-iot-edge/gateway-transparent.png" alt-text="Diagrama de una puerta de enlace transparente" border="false":::

Los dispositivos de nivel inferior también pueden conectarse a un dispositivo de puerta de enlace IoT Edge mediante un módulo personalizado. En el escenario siguiente, los dispositivos de nivel inferior se conectan mediante un módulo personalizado *Modbus*. En este escenario, el dispositivo IoT Edge es una puerta de enlace de traducción:

:::image type="content" source="media/concepts-iot-edge/gateway-module.png" alt-text="Diagrama de conexión de módulo personalizado" border="false":::

En el diagrama siguiente se muestran las conexiones a un dispositivo de puerta de enlace IoT Edge mediante ambos tipos de módulos. En este escenario, el dispositivo IoT Edge es una puerta de enlace transparente y de traducción:

:::image type="content" source="media/concepts-iot-edge/gateway-module-transparent.png" alt-text="Diagrama de conexión mediante ambos módulos de conexión" border="false":::

Los dispositivos de nivel inferior se pueden conectar a un dispositivo de puerta de enlace IoT Edge mediante varios módulos personalizados. En el diagrama siguiente se muestran los dispositivos de nivel inferior que se conectan mediante un módulo personalizado Modbus, un módulo personalizado BLE y el módulo del *centro de IoT Edge*:

:::image type="content" source="media/concepts-iot-edge/gateway-two-modules-transparent.png" alt-text="Diagrama de conexión mediante varios módulos personalizados" border="false":::

## <a name="iot-edge-devices-and-iot-central"></a>Dispositivos IoT Edge e IoT Central

Los dispositivos IoT Edge pueden usar tokens de *firma de acceso compartido* o certificados X.509 para autenticarse con IoT Central. Puede registrar manualmente los dispositivos IoT Edge en IoT Central antes de que se conecten por primera vez, o bien usar Device Provisioning Service para controlar el registro. Para más información, consulte [Conexión a Azure IoT Central](concepts-get-connected.md).

IoT Central usa [plantillas de dispositivo](concepts-device-templates.md) para definir el modo en que IoT Central interactúa con un dispositivo. Por ejemplo, una plantilla de dispositivo especifica:

* Los tipos de telemetría y propiedades que envía un dispositivo para que IoT Central pueda interpretarlos y crear visualizaciones.
* Los comandos a los que responde un dispositivo, por lo que IoT Central puede mostrar una interfaz de usuario para que un operador la use a fin de invocar los comandos.

Un dispositivo IoT Edge puede enviar telemetría, sincronizar valores de propiedad y responder a comandos de la misma manera que un dispositivo estándar. Por lo tanto, un dispositivo IoT Edge necesita una plantilla de dispositivo en IoT Central.

### <a name="iot-edge-device-templates"></a>Plantillas de dispositivo de IoT Edge

Las plantillas de dispositivo de IoT Central usan modelos para describir las funcionalidades de los dispositivos. En el diagrama siguiente se muestra la estructura del modelo de un dispositivo IoT Edge:

:::image type="content" source="media/concepts-iot-edge/iot-edge-model.png" alt-text="Estructura del modelo para el dispositivo IoT Edge conectado a IoT Central" border="false":::

IoT Central modela un dispositivo IoT Edge de la siguiente manera:

* Cada plantilla de dispositivo IoT Edge tiene un modelo de funcionalidad.
* Para cada módulo personalizado incluido en el manifiesto de implementación, se genera un modelo de funcionalidad del módulo.
* Se establece una relación entre cada modelo de funcionalidad del módulo y un modelo de dispositivo.
* Un modelo de funcionalidad del módulo implementa una o varias interfaces de módulo.
* Cada interfaz del módulo contiene datos de telemetría, propiedades y comandos.

### <a name="iot-edge-deployment-manifests-and-iot-central-device-templates"></a>Manifiestos de implementación de IoT Edge y plantillas de dispositivos de IoT Central

En IoT Edge, puede implementar y administrar la lógica de negocios en forma de módulos. Los módulos de IoT Edge son la unidad más pequeña de cálculo que administra IoT Edge y pueden contener servicios de Azure, como Azure Stream Analytics, o su propio código específico de la solución.

En un *manifiesto de implementación* de IoT Edge se enumeran los módulos de IoT Edge que se van a implementar en el dispositivo y cómo configurarlos. Para obtener más información, consulte [Obtenga información sobre cómo implementar módulos y establecer rutas en IoT Edge](../../iot-edge/module-composition.md).

En Azure IoT Central, puede importar un manifiesto de implementación para crear una plantilla de dispositivo para el dispositivo IoT Edge.

El siguiente fragmento de código muestra un ejemplo del manifiesto de implementación de IoT Edge:

```json
{
  "modulesContent": {
    "$edgeAgent": {
      "properties.desired": {
        "schemaVersion": "1.0",
        "runtime": {
          "type": "docker",
          "settings": {
            "minDockerVersion": "v1.25",
            "loggingOptions": "",
            "registryCredentials": {}
          }
        },
        "systemModules": {
          "edgeAgent": {
            "type": "docker",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-agent:1.0.9",
              "createOptions": "{}"
            }
          },
          "edgeHub": {
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-hub:1.0.9",
              "createOptions": "{}"
            }
          }
        },
        "modules": {
          "SimulatedTemperatureSensor": {
            "version": "1.0",
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
              "createOptions": "{}"
            }
          }
        }
      }
    },
    "$edgeHub": {
      "properties.desired": {
        "schemaVersion": "1.0",
        "routes": {
            "route": "FROM /* INTO $upstream"
        },
        "storeAndForwardConfiguration": {
          "timeToLiveSecs": 7200
        }
      }
    },
    "SimulatedTemperatureSensor": {
      "properties.desired": {
           "SendData": true,
           "SendInterval": 10
      }
    }
  }
}
```

En el fragmento de código anterior, puede ver:

* Hay tres módulos. Los módulos del sistema de *agente de IoT Edge* y de *centro de IoT Edge* presentes en cada manifiesto de implementación. El módulo **SimulatedTemperatureSensor** personalizado.
* Las imágenes del módulo público se extraen de un repositorio de Azure Container Registry que no requiere credenciales para conectarse. Para las imágenes de módulo privado, establezca las credenciales del registro de contenedor que se usarán en la `registryCredentials`configuración del módulo de *agente de IoT Edge*.
* El módulo **SimulatedTemperatureSensor** personalizado tiene dos propiedades `"SendData": true` y `"SendInterval": 10`.

Al importar este manifiesto de implementación en una aplicación de IoT Central, se genera la siguiente plantilla de dispositivo:

:::image type="content" source="media/concepts-iot-edge/device-template.png" alt-text="Captura de pantalla que muestra la plantilla de dispositivo creada a partir del manifiesto de implementación.":::

En la captura de pantalla anterior puede ver:

* Un módulo denominado **SimulatedTemperatureSensor**. Los módulos del sistema de *agente de IoT Edge* y *centro de IoT Edge* no aparecen en la plantilla.
* Una interfaz denominada **management** que incluye dos propiedades de escritura llamadas **SendData** y **SendInterval**.

El manifiesto de implementación no incluye información sobre la telemetría que el módulo **SimulatedTemperatureSensor** envía o los comandos a los que responde. Agregue estas definiciones a la plantilla de dispositivo manualmente antes de publicarla.

Para más información, consulte [Tutorial: Incorporación de un dispositivo Azure IoT Edge a la aplicación Azure IoT Central](tutorial-add-edge-as-leaf-device.md).

### <a name="update-a-deployment-manifest"></a>Actualización de un manifiesto de implementación

Al reemplazar el manifiesto de implementación, todos los dispositivos IoT Edge conectados descargan el nuevo manifiesto y actualizan sus módulos. Sin embargo, IoT Central no actualiza las interfaces de la plantilla de dispositivo con ningún cambio en la configuración del módulo. Por ejemplo, si reemplaza el manifiesto mostrado en el fragmento de código anterior por el siguiente manifiesto, no verá automáticamente la propiedad **SendUnits** en la interfaz de **administración** de la plantilla de dispositivo. Agregue manualmente la nueva propiedad a la interfaz de **administración** para que IoT Central la reconozca:

```json
{
  "modulesContent": {
    "$edgeAgent": {
      "properties.desired": {
        "schemaVersion": "1.0",
        "runtime": {
          "type": "docker",
          "settings": {
            "minDockerVersion": "v1.25",
            "loggingOptions": "",
            "registryCredentials": {}
          }
        },
        "systemModules": {
          "edgeAgent": {
            "type": "docker",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-agent:1.0.9",
              "createOptions": "{}"
            }
          },
          "edgeHub": {
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-hub:1.0.9",
              "createOptions": "{}"
            }
          }
        },
        "modules": {
          "SimulatedTemperatureSensor": {
            "version": "1.0",
            "type": "docker",
            "status": "running",
            "restartPolicy": "always",
            "settings": {
              "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
              "createOptions": "{}"
            }
          }
        }
      }
    },
    "$edgeHub": {
      "properties.desired": {
        "schemaVersion": "1.0",
        "routes": {
            "route": "FROM /* INTO $upstream"
        },
        "storeAndForwardConfiguration": {
          "timeToLiveSecs": 7200
        }
      }
    },
    "SimulatedTemperatureSensor": {
      "properties.desired": {
           "SendData": true,
           "SendInterval": 10,
           "SendUnits": "Celsius"
      }
    }
  }
}
```

## <a name="deploy-the-iot-edge-runtime"></a>Implementación del entorno de ejecución de IoT Edge

Para obtener información sobre dónde puede ejecutar el entorno de ejecución de IoT Edge, consulte [Sistemas compatibles con Azure IoT Edge](../../iot-edge/support.md).

También puede instalar el entorno de ejecución de Azure IoT Edge en los siguientes entornos:

* [Instalación o desinstalación de Azure IoT Edge para Linux](../../iot-edge/how-to-install-iot-edge.md)
* [Instalación y aprovisionamiento de Azure IoT Edge para Linux en un dispositivo Windows (versión preliminar)](../../iot-edge/how-to-install-iot-edge-on-windows.md)
* [Ejecución de Azure IoT Edge en máquinas virtuales Ubuntu en Azure](../../iot-edge/how-to-install-iot-edge-ubuntuvm.md).

## <a name="iot-edge-gateway-devices"></a>Dispositivos de puerta de enlace de IoT Edge

Si seleccionó un dispositivo de IoT Edge como dispositivo de puerta de enlace, puede agregar relaciones de nivel inferior a los modelos de los dispositivos que desea conectar a él.

Para obtener más información, consulte [Conexión de dispositivos a través de una puerta de enlace transparente IoT Edge](how-to-connect-iot-edge-transparent-gateway.md).

## <a name="next-steps"></a>Pasos siguientes

El siguiente paso recomendado es aprender a [desarrollar sus propios módulos de IoT Edge](../../iot-edge/module-development.md).
