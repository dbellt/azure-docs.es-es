---
title: 'Registro de un nuevo dispositivo: Azure IoT Edge | Microsoft Docs'
description: Registre un único dispositivo IoT Edge en IoT Hub para el aprovisionamiento manual con claves simétricas o certificados X.509.
author: kgremban
manager: philmea
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 10/06/2020
ms.author: kgremban
ms.openlocfilehash: b5d761cfa947b3fd4e5f718e603219c650e8dd72
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107481878"
---
# <a name="register-an-iot-edge-device-in-iot-hub"></a>Registro de un dispositivo IoT Edge en IoT Hub

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

En este artículo se brindan los pasos para registrar un nuevo dispositivo IoT Edge en IoT Hub.

Todos los dispositivos que se conectan a IoT Hub tienen un identificador de dispositivo que se usa para realizar un seguimiento de las comunicaciones de la nube al dispositivo o del dispositivo a la nube. Configure un dispositivo con su información de conexión, que incluye el nombre de host del centro de IoT, el identificador de dispositivo y la información que el dispositivo usa para autenticarse en IoT Hub.

Los pasos de este artículo le guían a través de un proceso denominado aprovisionamiento manual, donde conecta un único dispositivo a su centro de IoT. Para el aprovisionamiento manual, tiene dos opciones para autenticar dispositivos IoT Edge:

* **Clave simétrica**: cuando se crea una identidad de dispositivo nueva en IoT Hub, el servicio crea dos claves. Debe colocar una de las claves en el dispositivo y este presenta la clave a IoT Hub al autenticarse.

  Este método de autenticación es más rápido para comenzar, pero no es tan seguro.

* **X.509 autofirmado**: cree dos certificados de identidad X.509 y colóquelos en el dispositivo. Cuando se crea una identidad de dispositivo nueva en IoT Hub, se proporcionan huellas digitales desde ambos certificados. Cuando el dispositivo se autentica en IoT Hub, presenta un certificado, y IoT Hub comprueba que el certificado coincide con su huella digital.

  Este método de autenticación es más seguro y se recomienda para escenarios de producción.

En este artículo se tratan ambos métodos de autenticación.

Si tiene muchos dispositivos para configurar y no quiere aprovisionar manualmente cada uno, siga uno de los artículos a continuación para obtener información sobre cómo funciona IoT Edge con IoT Hub Device Provisioning Service:

* [Creación y aprovisionamiento de dispositivos IoT Edge mediante certificados X.509](how-to-auto-provision-x509-certs.md)
* [Creación y aprovisionamiento de dispositivos IoT Edge con un TPM](how-to-auto-provision-simulated-device-linux.md)
* [Creación y aprovisionamiento de dispositivos IoT Edge mediante claves simétricas](how-to-auto-provision-symmetric-keys.md)

## <a name="prerequisites"></a>Requisitos previos

# <a name="portal"></a>[Portal](#tab/azure-portal)

Un [centro de IoT](../iot-hub/iot-hub-create-through-portal.md) gratuito o estándar en la suscripción de Azure.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* Un [centro de IoT](../iot-hub/iot-hub-create-through-portal.md) gratuito o estándar en la suscripción de Azure
* Tener [Visual Studio Code](https://code.visualstudio.com/)
* [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) para Visual Studio Code

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

* Un [centro de IoT](../iot-hub/iot-hub-create-using-cli.md) gratuito o estándar en la suscripción de Azure.
* La [CLI de Azure](/cli/azure/install-azure-cli) en su entorno. Como mínimo, la versión de la CLI de Azure debe ser la 2.0.70 o posterior. Use `az --version` para asegurarse. Esta versión admite comandos az extension e introduce la plataforma de comandos de Knack.

---

## <a name="option-1-register-with-symmetric-keys"></a>Opción 1: Registro con claves simétricas

Puede usar varias herramientas para registrar un nuevo dispositivo IoT Edge en IoT Hub y recuperar su cadena de conexión, en función de sus preferencias.

# <a name="portal"></a>[Portal](#tab/azure-portal)

En su centro de IoT en Azure Portal, los dispositivos IoT Edge se crean y administran por separado de los dispositivos IOT que no están habilitados para Edge.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a IoT Hub.

1. En el panel izquierdo, seleccione **IoT Edge** en el menú y, a continuación, seleccione **Agregar un dispositivo IoT Edge**.

   ![Agrear un dispositivo IoT Edge desde Azure Portal](./media/how-to-register-device/portal-add-iot-edge-device.png)

1. En la página **Crear un dispositivo**, proporcione la información siguiente:

   * Cree un identificador de dispositivo descriptivo.
   * Seleccione **Clave simétrica** como el tipo de autenticación.
   * Use la configuración predeterminada para la generación automática de claves de autenticación y para conectar el dispositivo nuevo a su centro.

1. Seleccione **Guardar**.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

### <a name="sign-in-to-access-your-iot-hub"></a>Iniciar sesión para acceder a IoT Hub

Puede usar las extensiones de Azure IoT para Visual Studio Code para realizar operaciones con IoT Hub. Para que estas operaciones funcionen, debe iniciar sesión en su cuenta de Azure y seleccionar su centro.

1. En Visual Studio Code, abra la vista **Explorer** (Explorador).
1. En la parte inferior de Explorer, expanda la sección **Azure IoT Hub**.

   ![Sección Expansión de los dispositivos de Azure IoT Hub](./media/how-to-register-device/azure-iot-hub-devices.png)

1. Haga clic en los puntos suspensivos **...** en el encabezado de la sección **Azure IoT Hub**. Si no ve el botón de los puntos suspensivos, mantenga el ratón sobre el encabezado o haga clic en él.
1. Elija **Select IoT Hub** (Seleccionar IoT Hub).
1. Si no ha iniciado sesión en su cuenta de Azure, siga las indicaciones para hacerlo.
1. Seleccione su suscripción a Azure.
1. Seleccione IoT Hub.

### <a name="register-a-new-device-with-visual-studio-code"></a>Registro de un nuevo dispositivo en Visual Studio Code

1. En el explorador de Visual Studio Code, expanda la sección **Azure IoT Hub**.
1. Haga clic en los puntos suspensivos **...** en el encabezado de la sección **Azure IoT Hub**. Si no ve el botón de los puntos suspensivos, mantenga el ratón sobre el encabezado o haga clic en él.
1. Seleccione **Create IoT Edge Device** (Crear un dispositivo de IoT Edge).
1. En el cuadro de texto que aparece, otorgue a su dispositivo un identificador.

En la pantalla de salida, consulte el resultado del comando. Se imprime la información del dispositivo, que incluye el valor de **deviceId** que proporcionó, y el valor de **connectionString** que puede usar para conectar su dispositivo físico a su instancia de IoT Hub.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el comando [az iot hub device-identity create](/cli/azure/iot/hub/device-identity) para crear una nueva identidad de dispositivo en su centro de IoT. Por ejemplo:

   ```azurecli
   az iot hub device-identity create --device-id [device id] --hub-name [hub name] --edge-enabled
   ```

Este comando incluye tres parámetros:

* `--device-id` o `-d`: proporcione un nombre descriptivo que sea único dentro de su centro de IoT.
* `--hub-name` o `-n`: proporcione el nombre de su centro de IoT.
* `--edge-enabled` o `--ee`: Declare que se trata de un dispositivo de IoT Edge.

   ![az iot hub device-identity create output](./media/how-to-register-device/create-edge-device-cli.png)

---

Ahora que tiene un dispositivo registrado en IoT Hub, recupere la cadena de conexión que usa para completar la instalación y el aprovisionamiento del entorno de ejecución de Azure IoT Edge. Siga los pasos que se describen más adelante en este artículo para [ver los dispositivos registrados y recuperar las cadenas de conexión](#view-registered-devices-and-retrieve-connection-strings).

## <a name="option-2-register-with-x509-certificates"></a>Opción 2: Registro con certificados X.509

El aprovisionamiento manual con certificados X.509 requiere IoT Edge, versión 1.0.10 o posterior.

En el caso de la autenticación mediante certificados X.509, la información de autenticación de cada dispositivo se proporciona en forma de *huellas digitales* tomadas de los certificados de identidad del dispositivo. Estas huellas digitales se proporcionan a IoT Hub en el momento de registrar el dispositivo para que el servicio pueda reconocer el dispositivo al conectarse.

### <a name="create-certificates-and-thumbprints"></a>Creación de certificados y huellas digitales

Al aprovisionar un dispositivo IoT Edge con certificados X.509, se usa lo que se denomina un *certificado de identidad del dispositivo*. Este certificado solo se usa para aprovisionar un dispositivo IoT Edge y autenticarlo en Azure IoT Hub. Es un certificado de hoja que no firma otros certificados. El certificado de identidad del dispositivo es independiente de los certificados de la entidad de certificación (CA) que el dispositivo IoT Edge presenta a los módulos o dispositivos de nivel inferior para la verificación. Para obtener más información sobre cómo se usan los certificados de CA en los dispositivos IoT Edge, consulte [Información sobre los certificados de Azure IoT Edge](iot-edge-certs.md).

Necesita los siguientes archivos para el aprovisionamiento manual con X.509:

* Dos certificados de identidad del dispositivo con sus certificados de clave privada coincidentes en los formatos .cer o .pem.

  Se proporciona un conjunto de archivos de certificado y clave al entorno de ejecución de IoT Edge. Al crear los certificados de identidad del dispositivo, establezca el nombre común del certificado (CN) en el id. de dispositivo que quiere que tenga el dispositivo en el centro de IoT.

* Huellas digitales tomadas de ambos certificados de identidad del dispositivo.

  Los valores de huella digital son 40 caracteres hexadecimales para hashes de tipo SHA-1 o 64 caracteres hexadecimales para hashes de tipo SHA-256. Ambas huellas digitales se proporcionan a IoT Hub en el momento del registro del dispositivo.

Si no tiene certificados disponibles, puede proceder a la [Creación de certificados de demostración para probar las características de dispositivo IoT Edge](how-to-create-test-certificates.md). Siga las instrucciones de ese artículo para configurar scripts de creación de certificados, crear un certificado de CA raíz y luego crear dos certificados de identidad del dispositivo IoT Edge.

Una manera de recuperar la huella digital de un certificado es con el siguiente comando openssl:

```cmd
openssl x509 -in <certificate filename>.pem -text -fingerprint
```

### <a name="register-a-new-device"></a>Registro de un dispositivo nuevo

Puede usar varias herramientas para registrar un dispositivo IoT Edge nuevo en IoT Hub y cargar sus huellas digitales de certificado.

# <a name="portal"></a>[Portal](#tab/azure-portal)

En su centro de IoT en Azure Portal, los dispositivos IoT Edge se crean y administran por separado de los dispositivos IOT que no están habilitados para Edge.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a IoT Hub.

1. En el panel izquierdo, seleccione **IoT Edge** en el menú y, a continuación, seleccione **Agregar un dispositivo IoT Edge**.

   ![Agrear un dispositivo IoT Edge desde Azure Portal](./media/how-to-register-device/portal-add-iot-edge-device.png)

1. En la página **Crear un dispositivo**, proporcione la información siguiente:

   * Cree un id. de dispositivo descriptivo. Anote este id. de dispositivo porque lo usará en la sección siguiente.
   * Seleccione **X.509 autofirmado** como el tipo de autenticación.
   * Proporcione las huellas digitales de certificado de identidad principal y secundaria. Los valores de huella digital son 40 caracteres hexadecimales para hashes de tipo SHA-1 o 64 caracteres hexadecimales para hashes de tipo SHA-256.

1. Seleccione **Guardar**.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Actualmente, la extensión de Azure IoT para Visual Studio Code no admite el registro de dispositivos con certificados X.509.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el comando [az iot hub device-identity create](/cli/azure/iot/hub/device-identity) para crear una nueva identidad de dispositivo en su centro de IoT. Por ejemplo:

   ```azurecli
   az iot hub device-identity create --device-id [device id] --hub-name [hub name] --edge-enabled --auth-method x509_thumbprint --primary-thumbprint [SHA thumbprint] --secondary-thumbprint [SHA thumbprint]
   ```

Este comando incluye varios parámetros:

* `--device-id` o `-d`: proporcione un nombre descriptivo que sea único de su centro de IoT. Anote este id. de dispositivo porque lo usará en la sección siguiente.
* `hub-name` o `-n`: proporcione el nombre de su centro de IoT.
* `--edge-enabled` o `--ee`: declare que se trata de un dispositivo IoT Edge.
* `--auth-method` o `--am`: declare el tipo de autorización que usará el dispositivo. En este caso, vamos a usar huellas digitales de certificado X.509.
* `--primary-thumbprint` o `--ptp`: proporcione una huella digital de certificado X.509 que se usará como clave principal.
* `--secondary-thumbprint` o `--stp`: proporcione una huella digital de certificado X.509 que se usará como clave secundaria.

---

Ahora que tiene un dispositivo registrada en IoT Hub, está listo para instalar y aprovisionar el entorno de tiempo de ejecución de IoT Edge en el dispositivo. Los dispositivos IoT Edge que se autentican con certificados X.509 no usan cadenas de conexión, por lo que puede continuar con el paso siguiente:

* [Instalación o desinstalación de Azure IoT Edge para Linux](how-to-install-iot-edge.md)
* [Instalación o desinstalación de Azure IoT Edge para Windows](how-to-install-iot-edge-windows-on-windows.md)

## <a name="view-registered-devices-and-retrieve-connection-strings"></a>Vista de dispositivos registrados y recuperación de cadenas de conexión

Los dispositivos que usan la autenticación de claves simétricas necesitan sus cadenas de conexión para completar la instalación y el aprovisionamiento del entorno de ejecución de Azure IoT Edge.

Los dispositivos que usan la autenticación de certificados X.509 no necesitan cadenas de conexión. En su lugar, estos dispositivos necesitan el nombre de su centro de IoT, su nombre de dispositivo y sus archivos de certificado para completar la instalación y el aprovisionamiento del entorno de ejecución de Azure IoT Edge.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Todos los dispositivos habilitados para Edge que se conectan al centro de IoT se enumeran en la página **IoT Edge**.

![Uso de Azure Portal para visualizar todos los dispositivos IoT Edge del centro de IoT](./media/how-to-register-device/portal-view-devices.png)

Cuando esté listo para configurar el dispositivo, necesitará la cadena de conexión que vincula el dispositivo físico con su identidad en el centro de IoT.

Los dispositivos que se autentican con claves simétricas tienen sus cadenas de conexión disponibles para copiarse en el portal.

1. En la página **IoT Edge** del portal, haga clic en el identificador de dispositivo en la lista de dispositivos de IoT Edge.
2. Copie el valor de la **cadena de conexión primaria** o la **cadena de conexión secundaria**.

# <a name="visual-studio-code"></a>Tener [Visual Studio Code](#tab/visual-studio-code)

### <a name="view-iot-edge-devices-with-visual-studio-code"></a>Creación de dispositivos IoT Edge con Visual Studio Code

Todos los dispositivos que se conectan a IoT Hub se enumeran en la sección **Azure IoT Hub** del explorador de Visual Studio Code. Los dispositivos de IoT Edge se distinguen de los dispositivos que no son Edge con un icono diferente y por el hecho de que los módulos **$edgeAgent** y **$edgeHub** se implementan en cada dispositivo IoT Edge.

![Uso de VS Code para visualizar todos los dispositivos IoT Edge del centro de IoT](./media/how-to-register-device/view-devices.png)

### <a name="retrieve-the-connection-string-with-visual-studio-code"></a>Recuperación de la cadena de conexión con Visual Studio Code

Cuando esté listo para configurar el dispositivo, necesitará la cadena de conexión que vincula el dispositivo físico con su identidad en el centro de IoT.

1. Haga clic con el botón derecho en el identificador del dispositivo en la sección **Azure IoT Hub**.
1. Seleccione **Copy Device Connection String** (Copiar la cadena de conexión del dispositivo).

   La cadena de conexión se copiará al portapapeles.

También puede seleccionar **Get Device Info** (Obtener información del dispositivo), si hace clic con el botón derecho del mouse en el menú para ver toda la información del dispositivo, incluyendo la cadena de conexión, en la ventana de salida.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

### <a name="view-iot-edge-devices-with-the-azure-cli"></a>Creación de dispositivos IoT Edge con la CLI de Azure

Use el comando [az iot hub device-identity list](/cli/azure/iot/hub/device-identity) para ver todos los dispositivos en su centro de IoT. Por ejemplo:

   ```azurecli
   az iot hub device-identity list --hub-name [hub name]
   ```

Los dispositivos que están registrados como dispositivos IoT Edge tendrán la propiedad **capabilities.iotEdge** establecida en **true**.

### <a name="retrieve-the-connection-string-with-the-azure-cli"></a>Recuperación de la cadena de conexión con la CLI de Azure

Cuando esté listo para configurar el dispositivo, necesitará la cadena de conexión que vincula el dispositivo físico con su identidad en el centro de IoT. Use el comando [az iot hub device-identity connection-string show](/cli/azure/iot/hub/device-identity/connection-string) para devolver la cadena de conexión para un único dispositivo:

   ```azurecli
   az iot hub device-identity connection-string show --device-id [device id] --hub-name [hub name]
   ```

>[!TIP]
>El comando `connection-string show` se presentó en la versión 0.9.8 de la extensión de Azure IoT, en reemplazo del comando `show-connection-string` en desuso. Si recibe un error al ejecutar este comando, asegúrese de que la versión de la extensión esté actualizada a 0.9.8 o posterior. Para obtener más información y las actualizaciones más recientes, consulte [Extensión de Microsoft Azure IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension).

El valor del parámetro `device-id` distingue mayúsculas y minúsculas.

Al copiar la cadena de conexión que se va a usar en un dispositivo, no incluya las comillas alrededor de la cadena de conexión.

---

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene un dispositivo registrada en IoT Hub, está listo para instalar y aprovisionar el entorno de tiempo de ejecución de IoT Edge en el dispositivo.

* [Instalación o desinstalación de Azure IoT Edge para Linux](how-to-install-iot-edge.md)
* [Instalación o desinstalación de Azure IoT Edge para Windows](how-to-install-iot-edge-windows-on-windows.md)