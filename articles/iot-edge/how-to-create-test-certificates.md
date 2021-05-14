---
title: 'Creación de certificados de prueba: Azure IoT Edge | Microsoft Docs'
description: Cree certificados de prueba y obtenga información sobre cómo instalarlos en un dispositivo Azure IoT Edge para preparar la implementación de producción.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 06/02/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: adfb46894e769a23a2ac48bdb4ac3e432d9cebce
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108139064"
---
# <a name="create-demo-certificates-to-test-iot-edge-device-features"></a>Creación de certificados de demostración para probar las características de dispositivo IoT Edge

[!INCLUDE [iot-edge-version-201806-or-202011](../../includes/iot-edge-version-201806-or-202011.md)]

Los dispositivos IoT Edge requieren certificados para una comunicación segura entre el entorno de ejecución, los módulos y los dispositivos de bajada.
Si no tiene una entidad de certificación para crear los certificados necesarios, puede usar certificados de demostración para probar las características de IoT Edge en el entorno de prueba.
En este artículo se describe la funcionalidad de los scripts de generación de certificados que IoT Edge proporciona para las pruebas.

Estos certificados expiran en 30 días y no deben usarse en ningún escenario de producción.

Puede crear certificados en cualquier máquina y, a continuación, copiarlos en el dispositivo IoT Edge.
Es más fácil usar el equipo principal para crear los certificados en lugar de generarlos en el propio dispositivo IoT Edge.
Mediante el uso del equipo principal, puede configurar los scripts una vez y, a continuación, usarlos para crear certificados para varios dispositivos.

Siga estos pasos para crear certificados de demostración para probar el escenario de IoT Edge:

1. [Configure scripts](#set-up-scripts) para la generación de certificados en el dispositivo.
2. [Cree el certificado de CA raíz](#create-root-ca-certificate) que usa para firmar todos los demás certificados del escenario.
3. Genere los certificados que necesita para el escenario que desea probar:
   * [Cree certificados de identidad de dispositivo de IoT Edge](#create-iot-edge-device-identity-certificates) para aprovisionar dispositivos con la autenticación de certificado X.509, ya sea manualmente o con IoT Hub Device Provisioning Service.
   * [Cree certificados de CA de dispositivo IoT Edge](#create-iot-edge-device-ca-certificates) para los dispositivos IoT Edge en escenarios de puerta de enlace.
   * [Cree certificados de dispositivo de bajada](#create-downstream-device-certificates) para la autenticación de dispositivos de bajada en un escenario de puerta de enlace.

## <a name="prerequisites"></a>Requisitos previos

Una máquina de desarrollo con Git instalado.

## <a name="set-up-scripts"></a>Configuración de script

El repositorio IoT Edge en GitHub incluye scripts de generación de certificados que puede usar para crear certificados de demostración.
En esta sección se ofrecen instrucciones para preparar los scripts para que se ejecuten en el equipo, ya sea en Windows o Linux.
Si tiene un equipo con Linux, vaya a [Configuración en Linux](#set-up-on-linux).

### <a name="set-up-on-windows"></a>Configuración en Windows

Para crear certificados de demostración en un dispositivo Windows, debe instalar OpenSSL, clonar los scripts de generación y configurarlos para que se ejecuten localmente en PowerShell.

#### <a name="install-openssl"></a>Instalación de OpenSSL

Instale OpenSSL para Windows en el equipo que usa para generar los certificados.
Si ya ha instalado OpenSSL en el dispositivo Windows, asegúrese de que openssl.exe está disponible en la variable de entorno PATH.

Hay varias maneras de instalar OpenSSL, como las siguientes:

* **Más fácil:** Descargue e instale cualquier [archivo binario de OpenSSL de terceros](https://wiki.openssl.org/index.php/Binaries), por ejemplo, de [OpenSSL en SourceForge](https://sourceforge.net/projects/openssl/). Agregue la ruta de acceso completa al archivo openssl.exe a la variable de entorno PATH.

* **Se recomienda que use:** Descargue el código fuente de OpenSSL y compile los archivos binarios en su máquina por sí mismo o a través de [vcpkg](https://github.com/Microsoft/vcpkg). Las instrucciones que aparecen a continuación usan vcpkg para descargar el código fuente, compilar e instalar OpenSSL en el equipo Windows con pasos fáciles.

   1. Vaya al directorio donde quiera instalar vcpkg. Siga las instrucciones para descargar e instalar [vcpkg](https://github.com/Microsoft/vcpkg).

   2. Después de instalar vcpkg, ejecute el siguiente comando desde un símbolo del sistema de Powershell para instalar el paquete de OpenSSL para Windows x64. Esta instalación suele tardar, aproximadamente, cinco minutos en completarse.

      ```powershell
      .\vcpkg install openssl:x64-windows
      ```

   3. Agregue `<vcpkg path>\installed\x64-windows\tools\openssl` a la variable de entorno PATH para que el archivo openssl.exe esté disponible para su invocación.

#### <a name="prepare-scripts-in-powershell"></a>Preparación de scripts en PowerShell

El repositorio de Git de Azure IoT Edge contiene scripts que puede usar para generar certificados de prueba.
En esta sección, se clona el repositorio de IoT Edge y se ejecutan los scripts.

1. Abra una ventana de Azure PowerShell en modo de administrador.

2. Clone el repositorio de Git de IoT Edge que contiene los scripts para generar los certificados de demostración. Use el comando `git clone` o [descargue el archivo ZIP](https://github.com/Azure/iotedge/archive/master.zip).

   ```powershell
   git clone https://github.com/Azure/iotedge.git
   ```

3. Vaya al directorio en el que quiere trabajar. En este artículo, llamaremos a este directorio *\<WRKDIR>* . Todos los certificados y claves se crearán en este directorio de trabajo.

4. Copie los archivos de configuración y de script del repositorio clonado en el directorio de trabajo.

   ```powershell
   copy <path>\iotedge\tools\CACertificates\*.cnf .
   copy <path>\iotedge\tools\CACertificates\ca-certs.ps1 .
   ```

   Si descargó el repositorio como un archivo ZIP, el nombre de la carpeta es `iotedge-master` y el resto de la ruta de acceso es igual.

5. Habilite PowerShell para ejecutar los scripts.

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
   ```

6. Lleve las funciones usadas por los scripts al espacio de nombres global de PowerShell.

   ```powershell
   . .\ca-certs.ps1
   ```

   La ventana de PowerShell mostrará una advertencia para indicar que los certificados generados por ese script son solo para fines de prueba y no se deben usar en escenarios de producción.

7. Compruebe que OpenSSL se ha instalado correctamente y asegúrese de que no haya conflictos de nombres con los certificados existentes. Si hay problemas, la salida del script debe describir cómo corregirlos en el sistema.

   ```powershell
   Test-CACertsPrerequisites
   ```

### <a name="set-up-on-linux"></a>Configuración en Linux

Para crear certificados de demostración en un dispositivo Windows, debe clonar los scripts de generación y configurarlos para que se ejecuten localmente en bash.

1. Clone el repositorio de Git de IoT Edge que contiene los scripts para generar los certificados de demostración.

   ```bash
   git clone https://github.com/Azure/iotedge.git
   ```

2. Vaya al directorio en el que quiere trabajar. Nos referiremos a este directorio en el artículo como *\<WRKDIR>* . Todos los archivos de certificados y claves se crearán en este directorio.
  
3. Copie los archivos de configuración y de script del repositorio de IoT Edge clonado en el directorio de trabajo.

   ```bash
   cp <path>/iotedge/tools/CACertificates/*.cnf .
   cp <path>/iotedge/tools/CACertificates/certGen.sh .
   ```

<!--
4. Configure OpenSSL to generate certificates using the provided script. 

   ```bash
   chmod 700 certGen.sh 
   ```
-->

## <a name="create-root-ca-certificate"></a>Creación de un certificado de CA raíz

El certificado de CA raíz se usa para hacer todos los demás certificados de demostración con el fin de probar un escenario de IoT Edge.
Puede seguir usando el mismo certificado de CA raíz para crear certificados de demostración para varios dispositivos IoT Edge o de bajada.

Si ya tiene un certificado de CA raíz en la carpeta de trabajo, no cree uno nuevo.
El nuevo certificado de CA raíz sobrescribirá el antiguo y los certificados de bajada que se realicen desde el antiguo dejarán de funcionar.
Si desea tener varios certificados de CA raíz, asegúrese de administrarlos en carpetas independientes.

Antes de continuar con los pasos de esta sección, siga los descritos en la sección [Configuración de scripts](#set-up-scripts) para preparar un directorio de trabajo con los scripts de generación de certificados de demostración.

### <a name="windows"></a>Windows

1. Vaya al directorio de trabajo donde colocó los scripts de generación de certificados.

1. Cree el certificado de entidad de certificación raíz y firme con él un certificado intermedio. Los certificados se colocan en el directorio de trabajo.

   ```powershell
   New-CACertsCertChain rsa
   ```

   Este comando de script crea varios archivos de certificado y clave, pero cuando los artículos solicitan el **certificado de CA raíz**, use el siguiente archivo:

   * `<WRKDIR>\certs\azure-iot-test-only.root.ca.cert.pem`

### <a name="linux"></a>Linux

1. Vaya al directorio de trabajo donde colocó los scripts de generación de certificados.

1. Cree el certificado de entidad de certificación raíz y un certificado intermedio.

   ```bash
   ./certGen.sh create_root_and_intermediate
   ```

   Este comando de script crea varios archivos de certificado y clave, pero cuando los artículos solicitan el **certificado de CA raíz**, use el siguiente archivo:

   * `<WRKDIR>/certs/azure-iot-test-only.root.ca.cert.pem`  

## <a name="create-iot-edge-device-identity-certificates"></a>Creación de certificados de identidad de dispositivo IoT Edge

Los certificados de identidad de dispositivo se usan para aprovisionar dispositivos IoT Edge si decide usar la autenticación de certificado X.509. Estos certificados funcionan tanto si usa el aprovisionamiento manual como el automático a través de Azure IoT Hub Device Provisioning Service (DPS).

Los certificados de identidad de dispositivo van en la sección **Provisioning** del archivo de configuración en el dispositivo IoT Edge.

Antes de continuar con los pasos de esta sección, siga los descritos en las secciones [Configuración de scripts](#set-up-scripts) y [Creación de certificados de CA raíz](#create-root-ca-certificate).

### <a name="windows"></a>Windows

Cree el certificado de identidad de dispositivo IoT Edge y una clave privada con el siguiente comando:

```powershell
New-CACertsEdgeDeviceIdentity "<name>"
```

El nombre que pase a este comando será el identificador del dispositivo IoT Edge en IoT Hub.

El nuevo comando de identidad de dispositivo crea varios archivos de certificado y clave, incluidos los tres que usará al crear una inscripción individual en DPS e instalar el tiempo de ejecución de IoT Edge:

* `<WRKDIR>\certs\iot-edge-device-identity-<name>-full-chain.cert.pem`
* `<WRKDIR>\certs\iot-edge-device-identity-<name>.cert.pem`
* `<WRKDIR>\private\iot-edge-device-identity-<name>.key.pem`

### <a name="linux"></a>Linux

Cree el certificado de identidad de dispositivo IoT Edge y una clave privada con el siguiente comando:

```bash
./certGen.sh create_edge_device_identity_certificate "<name>"
```

El nombre que pase a este comando será el identificador del dispositivo IoT Edge en IoT Hub.

El script crea varios archivos de certificado y clave, incluidos los tres que usará al crear una inscripción individual en DPS e instalar el tiempo de ejecución de IoT Edge:

* `<WRKDIR>\certs\iot-edge-device-identity-<name>-full-chain.cert.pem`
* `<WRKDIR>/certs/iot-edge-device-identity-<name>.cert.pem`
* `<WRKDIR>/private/iot-edge-device-identity-<name>.key.pem`

## <a name="create-iot-edge-device-ca-certificates"></a>Creación de certificados de CA de dispositivo IoT Edge

Cada dispositivo IoT Edge que va a producción necesita un certificado de CA de dispositivo al que se haga referencia desde el archivo de configuración.
El certificado de CA de dispositivo es responsable de la creación de certificados para los módulos que se ejecutan en el dispositivo.
También es necesario para los escenarios de puerta de enlace, ya que el certificado de entidad de certificación del dispositivo es la forma en que el dispositivo IoT Edge comprueba su identidad en los dispositivos de bajada.

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"
Los certificados de CA de dispositivo van en la sección **Certificado** del archivo config.yaml en el dispositivo IoT Edge.
:::moniker-end

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"
Los certificados de CA de dispositivo van en la sección **Edge CA** (CA de Edge) del archivo config.toml en el dispositivo IoT Edge.
:::moniker-end

Antes de continuar con los pasos de esta sección, siga los descritos en las secciones [Configuración de scripts](#set-up-scripts) y [Creación de certificados de CA raíz](#create-root-ca-certificate).

### <a name="windows"></a>Windows

1. Vaya al directorio de trabajo que contiene los scripts de generación de certificados y el certificado de CA raíz.

2. Cree el certificado de entidad de certificación del dispositivo IoT Edge y una clave privada con el siguiente comando. Proporcione un nombre para el certificado de entidad de certificación.

   ```powershell
   New-CACertsEdgeDevice "<CA cert name>"
   ```

   Este comando crea varios archivos de claves y certificados. El siguiente certificado y el par de claves deben copiarse en un dispositivo IoT Edge y hacerse referencia en el archivo de configuración:

   * `<WRKDIR>\certs\iot-edge-device-<CA cert name>-full-chain.cert.pem`
   * `<WRKDIR>\private\iot-edge-device-<CA cert name>.key.pem`

El nombre que se pasa al comando **New-CACertsEdgeDevice** no debe coincidir con el parámetro hostname del archivo de configuración o el id. de dispositivo en IoT Hub.

### <a name="linux"></a>Linux

1. Vaya al directorio de trabajo que contiene los scripts de generación de certificados y el certificado de CA raíz.

2. Cree el certificado de entidad de certificación del dispositivo IoT Edge y una clave privada con el siguiente comando. Proporcione un nombre para el certificado de entidad de certificación.

   ```bash
   ./certGen.sh create_edge_device_ca_certificate "<CA cert name>"
   ```

   Este comando de script crea varios archivos de claves y certificados. El siguiente certificado y el par de claves deben copiarse en un dispositivo IoT Edge y hacerse referencia en el archivo de configuración:

   * `<WRKDIR>/certs/iot-edge-device-<CA cert name>-full-chain.cert.pem`
   * `<WRKDIR>/private/iot-edge-device-<CA cert name>.key.pem`

El nombre que se pasa al comando **create_edge_device_ca_certificate** no debe coincidir con el parámetro hostname del archivo de configuración ni con el id. de dispositivo en IoT Hub.

## <a name="create-downstream-device-certificates"></a>Creación de certificados de dispositivo de bajada

Si está configurando un dispositivo IoT de bajada para un escenario de puerta de enlace y quiere usar la autenticación X.509, puede generar certificados de demostración para el dispositivo de bajada.
Si desea usar la autenticación de clave simétrica, no necesita crear ningún certificado adicional para el dispositivo de bajada.
Hay dos maneras de autenticar un dispositivo IoT mediante certificados X.509: mediante certificados autofirmados o mediante certificados firmados por una entidad de certificación (CA).
Para la autenticación de X.509 autofirmada, en ocasiones denominada autenticación de huella digital, tendrá que crear certificados para el dispositivo IoT.
Estos certificados contienen una huella digital que se comparte con IoT Hub para la autenticación.
Para la autenticación de certificados X.509 firmados por entidad de certificación (CA), necesita un certificado de entidad de certificación raíz registrado en IoT Hub y usarlo para firmar los certificados para el dispositivo IoT.
Todo dispositivo con un certificado emitido por el certificado de CA raíz o cualquiera de sus certificados intermedios tendrá permiso para autenticarse.

Los scripts de generación de certificados pueden ayudarle a crear certificados de demostración para probar cualquiera de estos escenarios de autenticación.

Antes de continuar con los pasos de esta sección, siga los descritos en las secciones [Configuración de scripts](#set-up-scripts) y [Creación de certificados de CA raíz](#create-root-ca-certificate).

### <a name="self-signed-certificates"></a>Certificados autofirmados

Al autenticar un dispositivo IoT con certificados autofirmados, debe crear certificados de dispositivo basados en el certificado de CA raíz de la solución.
Después, recupera una "huella digital" hexadecimal de los certificados que se van a proporcionar a IoT Hub.
El dispositivo IoT también necesita una copia de sus certificados de dispositivo para que pueda autenticarse con IoT Hub.

#### <a name="windows"></a>Windows

1. Vaya al directorio de trabajo que contiene los scripts de generación de certificados y el certificado de CA raíz.

2. Cree dos certificados (principal y secundario) para el dispositivo de bajada. Una convención de nomenclatura fácil de usar es crear los certificados con el nombre del dispositivo IoT y, después, la etiqueta principal o secundaria. Por ejemplo:

   ```PowerShell
   New-CACertsDevice "<device name>-primary"
   New-CACertsDevice "<device name>-secondary"
   ```

   Este comando de script crea varios archivos de claves y certificados. Los siguientes pares de certificados y claves deben copiarse en el dispositivo IoT de bajada y se puede hacer referencia a ellos en las aplicaciones que se conectan a IoT Hub:

   * `<WRKDIR>\certs\iot-device-<device name>-primary-full-chain.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-secondary-full-chain.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-primary.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-secondary.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-primary.cert.pfx`
   * `<WRKDIR>\certs\iot-device-<device name>-secondary.cert.pfx`
   * `<WRKDIR>\private\iot-device-<device name>-primary.key.pem`
   * `<WRKDIR>\private\iot-device-<device name>-secondary.key.pem`

3. Recupere la huella digital SHA1 (llamada huella digital en contextos de IoT Hub) de cada certificado. La huella digital es una cadena de caracteres hexadecimales de 40. Use el comando de openssl siguiente para ver el certificado y localizar la huella digital:

   ```PowerShell
   openssl x509 -in <WRKDIR>\certs\iot-device-<device name>-primary.cert.pem -text -fingerprint
   ```

   Ejecute este comando dos veces, una vez para el certificado principal y otra para el certificado secundario. Proporcione huellas digitales para ambos certificados cuando registre un nuevo dispositivo IoT mediante certificados X.509 autofirmados.

#### <a name="linux"></a>Linux

1. Vaya al directorio de trabajo que contiene los scripts de generación de certificados y el certificado de CA raíz.

2. Cree dos certificados (principal y secundario) para el dispositivo de bajada. Una convención de nomenclatura fácil de usar es crear los certificados con el nombre del dispositivo IoT y, después, la etiqueta principal o secundaria. Por ejemplo:

   ```bash
   ./certGen.sh create_device_certificate "<device name>-primary"
   ./certGen.sh create_device_certificate "<device name>-secondary"
   ```

   Este comando de script crea varios archivos de claves y certificados. Los siguientes pares de certificados y claves deben copiarse en el dispositivo IoT de bajada y se puede hacer referencia a ellos en las aplicaciones que se conectan a IoT Hub:

   * `<WRKDIR>/certs/iot-device-<device name>-primary-full-chain.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-secondary-full-chain.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-primary.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-secondary.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-primary.cert.pfx`
   * `<WRKDIR>/certs/iot-device-<device name>-secondary.cert.pfx`
   * `<WRKDIR>/private/iot-device-<device name>-primary.key.pem`
   * `<WRKDIR>/private/iot-device-<device name>-secondary.key.pem`

3. Recupere la huella digital SHA1 (llamada huella digital en contextos de IoT Hub) de cada certificado. La huella digital es una cadena de caracteres hexadecimales de 40. Use el comando de openssl siguiente para ver el certificado y localizar la huella digital:

   ```bash
   openssl x509 -in <WRKDIR>/certs/iot-device-<device name>-primary.cert.pem -text -fingerprint | sed 's/[:]//g'
   ```

   Proporcione las huellas digitales principal y secundaria cuando registre un nuevo dispositivo IoT mediante certificados X.509 autofirmados.

### <a name="ca-signed-certificates"></a>Certificados firmados por una entidad de certificación

Al autenticar un dispositivo IoT con certificados autofirmados, debe cargar el certificado de CA raíz de la solución en IoT Hub.
A continuación, realice una comprobación para demostrar IoT Hub que posee el certificado de CA raíz.
Por último, va a usar el mismo certificado de CA raíz para crear certificados de dispositivo que se colocarán en el dispositivo de IoT para que pueda autenticarse con IoT Hub.

Los certificados de esta sección son para los pasos descritos en [Configuración de la seguridad de X.509 en el centro de IoT de Azure](../iot-hub/tutorial-x509-scripts.md).

#### <a name="windows"></a>Windows

1. Cargue el archivo de certificado de CA raíz desde el directorio de trabajo, `<WRKDIR>\certs\azure-iot-test-only.root.ca.cert.pem`, a la instancia de IoT Hub.

2. Use el código proporcionado en Azure Portal para comprobar que es el propietario de ese certificado de CA raíz.

   ```PowerShell
   New-CACertsVerificationCert "<verification code>"
   ```

3. Cree una cadena de certificados para el dispositivo de bajada. Use el mismo identificador de dispositivo con el que está registrado el dispositivo en IoT Hub.

   ```PowerShell
   New-CACertsDevice "<device id>"
   ```

   Este comando de script crea varios archivos de claves y certificados. Los siguientes pares de certificados y claves deben copiarse en el dispositivo IoT de bajada y se puede hacer referencia a ellos en las aplicaciones que se conectan a IoT Hub:

   * `<WRKDIR>\certs\iot-device-<device id>.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device id>.cert.pfx`
   * `<WRKDIR>\certs\iot-device-<device id>-full-chain.cert.pem`  
   * `<WRKDIR>\private\iot-device-<device id>.key.pem`

#### <a name="linux"></a>Linux

1. Cargue el archivo de certificado de CA raíz desde el directorio de trabajo, `<WRKDIR>\certs\azure-iot-test-only.root.ca.cert.pem`, a la instancia de IoT Hub.

2. Use el código proporcionado en Azure Portal para comprobar que es el propietario de ese certificado de CA raíz.

   ```bash
   ./certGen.sh create_verification_certificate "<verification code>"
   ```

3. Cree una cadena de certificados para el dispositivo de bajada. Use el mismo identificador de dispositivo con el que está registrado el dispositivo en IoT Hub.

   ```bash
   ./certGen.sh create_device_certificate "<device id>"
   ```

   Este comando de script crea varios archivos de claves y certificados. Los siguientes pares de certificados y claves deben copiarse en el dispositivo IoT de bajada y se puede hacer referencia a ellos en las aplicaciones que se conectan a IoT Hub:

   * `<WRKDIR>/certs/iot-device-<device id>.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device id>.cert.pfx`
   * `<WRKDIR>/certs/iot-device-<device id>-full-chain.cert.pem`  
   * `<WRKDIR>/private/iot-device-<device id>.key.pem`