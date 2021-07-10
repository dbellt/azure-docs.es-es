---
title: 'Tutorial: Aprovisionamiento de dispositivos X.509 para Azure IoT Hub con un módulo de seguridad de hardware (HSM) personalizado'
description: En este tutorial se usan grupos de inscripción. En este tutorial, aprenderá a aprovisionar dispositivos X.509 con un módulo de seguridad de hardware (HSM) personalizado y el SDK de dispositivos para C para Azure IoT Hub Device Provisioning Service (DPS).
author: wesmc7777
ms.author: wesmc
ms.date: 05/24/2021
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.custom: mvc
ms.openlocfilehash: 8e7d024d4d5b1e058e7a0b895faae5d2e7425f44
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110472134"
---
# <a name="tutorial-provision-multiple-x509-devices-using-enrollment-groups"></a>Tutorial: Aprovisionamiento de varios dispositivos X.509 mediante grupos de inscripción

En este tutorial, obtendrá información sobre cómo aprovisionar grupos de dispositivos IoT que usen certificados X.509 para la autenticación. Se ejecutará código del dispositivo de ejemplo del [SDK de C de Azure IoT](https://github.com/Azure/azure-iot-sdk-c) en la máquina de desarrollo para simular el aprovisionamiento de dispositivos X.509. En dispositivos reales, el código de dispositivo se implementará y se ejecutará desde el dispositivo IoT.

Asegúrese de completar al menos los pasos descritos en [Configuración de Azure IoT Hub Device Provisioning Service con Azure Portal](quick-setup-auto-provision.md) antes de continuar con este tutorial. Asimismo, si no está familiarizado con el proceso de aprovisionamiento automático, revise la información general sobre [Aprovisionamiento](about-iot-dps.md#provisioning-process). 

Azure IoT Hub Device Provisioning Service admite dos tipos de inscripciones para dispositivos de aprovisionamiento:

* [Grupos de inscripción](concepts-service.md#enrollment-group): usados para inscribir varios dispositivos relacionados.
* [Inscripciones individuales](concepts-service.md#individual-enrollment): usadas para inscribir un solo dispositivo.

Este tutorial es similar a los tutoriales anteriores que muestran cómo usar los grupos de inscripción para aprovisionar conjuntos de dispositivos. Sin embargo, en este tutorial se utilizarán certificados X.509 en lugar de las claves simétricas. Revise los tutoriales anteriores de esta sección para obtener un enfoque sencillo mediante [claves simétricas](./concepts-symmetric-key-attestation.md).

En este tutorial se muestra el [ejemplo de HSM personalizado](https://github.com/Azure/azure-iot-sdk-c/tree/master/provisioning_client/samples/custom_hsm_example) que proporciona una implementación de código auxiliar para interactuar con el almacenamiento seguro basado en hardware. Se usa un [módulo de seguridad de hardware (HSM)](./concepts-service.md#hardware-security-module) para el almacenamiento seguro basado en hardware de secretos de dispositivos. Un HSM se puede utilizar con una clave simétrica, un certificado X.509 o una atestación de TPM para proporcionar almacenamiento seguro para los secretos. No se requiere el almacenamiento en hardware de los secretos de dispositivo, pero es muy recomendable proteger la información confidencial, como la clave privada del certificado de dispositivo.


En este tutorial, va a completar los siguientes objetivos:

> [!div class="checklist"]
> * Crear una cadena de certificados de confianza para organizar un conjunto de dispositivos mediante certificados X.509.
> * Completar la prueba de posesión con un certificado de firma usado con la cadena de certificados.
> * Crear una nueva inscripción de grupo que use la cadena de certificados.
> * Configurar el entorno de desarrollo para el aprovisionamiento de un dispositivo con código del [SDK para C de Azure IoT](https://github.com/Azure/azure-iot-sdk-c).
> * Aprovisionar un dispositivo mediante la cadena de certificados con el ejemplo de módulo de seguridad de hardware (HSM) personalizado en el SDK.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Los siguientes requisitos previos corresponden a un entorno de desarrollo de Windows usado para simular los dispositivos. En el caso de Linux o macOS, consulte la sección correspondiente en [Preparación del entorno de desarrollo](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) en la documentación del SDK.

* [Visual Studio](https://visualstudio.microsoft.com/vs/) 2019 con la carga de trabajo ["Desarrollo para el escritorio con C++"](/cpp/ide/using-the-visual-studio-ide-for-cpp-desktop-development) habilitada. También se admiten Visual Studio 2015 y Visual Studio 2017. 

    En este artículo se usa Visual Studio para compilar el código de ejemplo del dispositivo que se implementaría en dispositivos IoT.  Esto no implica que se requiera Visual Studio en el propio dispositivo.

* Tener instalada la versión más reciente de [Git](https://git-scm.com/download/).

## <a name="prepare-the-azure-iot-c-sdk-development-environment"></a>Preparación del entorno de desarrollo del SDK de C para Azure IoT

En esta sección, preparará un entorno de desarrollo para compilar el [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c). El SDK incluye el código de ejemplo y las herramientas usadas para el aprovisionamiento de dispositivos X.509 con DPS.

1. Descargue el [sistema de compilación CMake](https://cmake.org/download/).

    **Antes** de comenzar la instalación de `CMake`, es importante que los requisitos previos de Visual Studio ([Visual Studio](https://visualstudio.microsoft.com/vs/) y la carga de trabajo [ "Desarrollo para el escritorio con C++"](/cpp/ide/using-the-visual-studio-ide-for-cpp-desktop-development)) estén instalados en la máquina. Una vez que los requisitos previos están en su lugar, y se ha comprobado la descarga, instale el sistema de compilación de CMake.

2. Busque el nombre de etiqueta de la [versión más reciente](https://github.com/Azure/azure-iot-sdk-c/releases/latest) del SDK de C de IoT de Azure.

3. Abra un símbolo del sistema o el shell de Bash de Git. Ejecute los siguientes comandos para clonar la versión más reciente del repositorio de GitHub del [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c). Use la etiqueta que encontró en el paso anterior como valor del parámetro `-b`:

    ```cmd/sh
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Esta operación puede tardar varios minutos en completarse.

4. Cree un subdirectorio `cmake` en el directorio raíz del repositorio de Git y vaya a esa carpeta. 

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

5. El directorio `cmake` que ha creado contendrá el ejemplo de HSM personalizado y el código de aprovisionamiento de dispositivos de ejemplo que usa el HSM personalizado para proporcionar la autenticación X.509. 

    Ejecute el siguiente comando en el directorio `cmake` para compilar una versión del SDK específica para su plataforma de desarrollo. La compilación incluirá una referencia al ejemplo de HSM personalizado. 

    Al especificar la ruta de acceso que se usa con `-Dhsm_custom_lib` a continuación, asegúrese de usar la ruta de acceso relativa al directorio `cmake` que ha creado anteriormente. La ruta de acceso relativa que se muestra a continuación es solo un ejemplo.

    ```cmd
    $ cmake -Duse_prov_client:BOOL=ON -Dhsm_custom_lib=/d/azure-iot-sdk-c/cmake/provisioning_client/samples/custom_hsm_example/Debug/custom_hsm_example.lib ..
    ```

    Si `cmake` no encuentra el compilador de C++, es posible que obtenga errores de compilación durante la ejecución del comando anterior. Si sucede, intente ejecutar este comando en el [símbolo del sistema de Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs).

    Una vez que la compilación se realiza correctamente, se generará una solución de Visual Studio en el directorio `cmake`. Las últimas líneas de la salida tienen un aspecto similar al siguiente resultado:

    ```cmd/sh
    $ cmake -Duse_prov_client:BOOL=ON -Dhsm_custom_lib=/d/azure-iot-sdk-c/cmake/provisioning_client/samples/custom_hsm_example/Debug/custom_hsm_example.lib ..
    -- Building for: Visual Studio 16 2019
    -- The C compiler identification is MSVC 19.23.28107.0
    -- The CXX compiler identification is MSVC 19.23.28107.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: D:/azure-iot-sdk-c/cmake
    ```

## <a name="create-an-x509-certificate-chain"></a>Creación de una cadena de certificados X.509

En esta sección, generará una cadena de certificados X.509 de tres certificados para probar cada dispositivo con este tutorial. Los certificados tendrán la siguiente jerarquía.

![Cadena de certificados de dispositivo del tutorial](./media/tutorial-custom-hsm-enrollment-group-x509/example-device-cert-chain.png#lightbox)

[Certificado raíz](concepts-x509-attestation.md#root-certificate): Completará la [prueba de posesión](how-to-verify-certificates.md) para verificar el certificado raíz. Esta verificación permitirá que DPS confíe en ese certificado y verifique los certificados firmados por él.

[Certificado intermedio](concepts-x509-attestation.md#intermediate-certificate): es habitual que se usen certificados intermedios para agrupar dispositivos de forma lógica por líneas de producto, divisiones de la compañía u otros criterios. En este tutorial se utilizará una cadena de certificados compuesta por un certificado intermedio. El certificado intermedio se firmará con el certificado raíz. Este certificado también se usará en el grupo de inscripción creado en DPS para agrupar lógicamente un conjunto de dispositivos. Esta configuración permite administrar un grupo completo de dispositivos que tienen certificados de dispositivo firmados por el mismo certificado intermedio. Puede crear grupos de inscripción para habilitar o deshabilitar un grupo de dispositivos. Para más información acerca de cómo deshabilitar un grupo de dispositivos, consulte [Denegación de un certificado de entidad de certificación X.509 intermedia o raíz usando un grupo de inscripción](how-to-revoke-device-access-portal.md#disallow-an-x509-intermediate-or-root-ca-certificate-by-using-an-enrollment-group).

[Certificados del dispositivo](concepts-x509-attestation.md#end-entity-leaf-certificate): los certificado del dispositivo (hoja) los firmará el certificado intermedio y se almacenarán en el dispositivo junto con su clave privada. Idealmente, estos elementos confidenciales se almacenarían de forma segura con un HSM. Cada dispositivo presentará su certificado y clave privada junto con la cadena de certificados al intentar el aprovisionamiento. 

#### <a name="create-root-and-intermediate-certificates"></a>Creación de certificados raíz e intermedios

Para crear las partes raíz e intermedia de la cadena de certificados:

> [!IMPORTANT]
> Use solo el enfoque del shell de Bash con este artículo. El uso de PowerShell es posible, pero no se trata en este artículo.


1. Abra un símbolo del sistema de Bash de Git. Complete los pasos 1 y 2 con las instrucciones del shell de bash que se encuentran en [Administración de certificados de entidad de certificación de prueba para ejemplos y tutoriales](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md#managing-test-ca-certificates-for-samples-and-tutorials).

    Esto crea un directorio de trabajo para los scripts de certificados y genera el certificado raíz e intermedio de ejemplo para la cadena de certificados mediante openssl. 
    
2. Observe que en el resultado se muestra la ubicación del certificado raíz autofirmado. Este certificado pasará la [prueba de posesión](how-to-verify-certificates.md) para comprobar la propiedad más adelante.

    ```output
    Creating the Root CA Certificate
    CA Root Certificate Generated At:
    ---------------------------------
        ./certs/azure-iot-test-only.root.ca.cert.pem
    
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                fc:cc:6b:ab:3b:9a:3e:fe
        Signature Algorithm: sha256WithRSAEncryption
            Issuer: CN=Azure IoT Hub CA Cert Test Only
            Validity
                Not Before: Oct 23 21:30:30 2020 GMT
                Not After : Nov 22 21:30:30 2020 GMT
            Subject: CN=Azure IoT Hub CA Cert Test Only
    ```        
    
3. Observe que en la salida se muestra la ubicación del certificado intermedio firmado o emitido por el certificado raíz. Este certificado se usará con el grupo de inscripción que creará más adelante.

    ```output
    Intermediate CA Certificate Generated At:
    -----------------------------------------
        ./certs/azure-iot-test-only.intermediate.cert.pem
    
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number: 1 (0x1)
        Signature Algorithm: sha256WithRSAEncryption
            Issuer: CN=Azure IoT Hub CA Cert Test Only
            Validity
                Not Before: Oct 23 21:30:33 2020 GMT
                Not After : Nov 22 21:30:33 2020 GMT
            Subject: CN=Azure IoT Hub Intermediate Cert Test Only
    ```    
    
#### <a name="create-device-certificates"></a>Creación de certificados de dispositivo

Para crear certificados de dispositivo firmados por el certificado intermedio en la cadena:

1. Ejecute el siguiente comando para crear un nuevo certificado de dispositivo (certificado hoja) con un nombre de firmante que proporcionará como un parámetro. Use el nombre de firmante de ejemplo que se proporciona para este tutorial, `custom-hsm-device-01`. Este nombre de firmante será el identificador del dispositivo IoT. 

    > [!WARNING]
    > No use un nombre de firmante que incluya espacios. Este nombre de firmante es el identificador del dispositivo IoT que se va a aprovisionar. Debe seguir las reglas de un identificador de dispositivo. Para más información, consulte [Propiedades de la id. de dispositivo](../iot-hub/iot-hub-devguide-identity-registry.md#device-identity-properties).

    ```cmd
    ./certGen.sh create_device_certificate_from_intermediate "custom-hsm-device-01"
    ```

    Observe que en la siguiente salida se muestra dónde se encuentra el nuevo certificado de dispositivo. El certificado intermedio firma (emite) el certificado de dispositivo.

    ```output
    -----------------------------------
    ./certs/new-device.cert.pem: OK
    Leaf Device Certificate Generated At:
    ----------------------------------------
        ./certs/new-device.cert.pem
    
    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number: 9 (0x9)
        Signature Algorithm: sha256WithRSAEncryption
            Issuer: CN=Azure IoT Hub Intermediate Cert Test Only
            Validity
                Not Before: Nov 10 09:20:33 2020 GMT
                Not After : Dec 10 09:20:33 2020 GMT
            Subject: CN=custom-hsm-device-01
    ```    
    
2. Ejecute el siguiente comando para crear un archivo .pem de la cadena de certificados completa que incluye el nuevo certificado de dispositivo para `custom-hsm-device-01`.

    ```Bash
    cd ./certs && cat new-device.cert.pem azure-iot-test-only.intermediate.cert.pem azure-iot-test-only.root.ca.cert.pem > new-device-01-full-chain.cert.pem && cd ..
    ```

    Use un editor de texto y abra el archivo de la cadena de certificados *./certs/new-device-01-full-chain.cert.pem*. El texto de la cadena de certificados contiene la cadena completa de los tres certificados. Usará este texto como la cadena de certificados en el código del dispositivo HSM personalizado más adelante en este tutorial para `custom-hsm-device-01`.

    El texto de la cadena completo tiene el formato siguiente:
 
    ```output 
    -----BEGIN CERTIFICATE-----
        <Text for the device certificate includes public key>
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
        <Text for the intermediate certificate includes public key>
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
        <Text for the root certificate includes public key>
    -----END CERTIFICATE-----
    ```

3. Observe que la clave privada del nuevo certificado de dispositivo se escribe en *./private/new-device.key.pem*. Cambie el nombre de este archivo de clave a *./private/new-device-01.key.pem* para el dispositivo `custom-hsm-device-01`. El dispositivo necesitará el texto de esta clave durante el aprovisionamiento. El texto se agregará al ejemplo de HSM personalizado más adelante.

    ```bash
    $ mv private/new-device.key.pem private/new-device-01.key.pem
    ```

    > [!WARNING]
    > El texto de los certificados solo contiene información de la clave pública. 
    >
    > Sin embargo, el dispositivo también debe tener acceso a la clave privada del certificado de dispositivo. Esto es necesario porque el dispositivo debe realizar la comprobación con esa clave en tiempo de ejecución al intentar el aprovisionamiento. La confidencialidad de esta clave es una de las principales razones por las que se recomienda usar el almacenamiento basado en hardware en un HSM real para ayudar a proteger las claves privadas.

4. Elimine el archivo *./certs/new-device.cert.pem* y repita los pasos del 1 al 3 para un segundo dispositivo con el id. de dispositivo `custom-hsm-device-02`. Debe eliminar el archivo *./certs/new-device.cert.pem* o se producirá un error en la generación del certificado para el segundo dispositivo. En este artículo, solo se usarán archivos de certificado de cadena completos. Utilice los siguientes valores para el segundo dispositivo:

    |   Descripción                 |  Value  |
    | :---------------------------- | :--------- |
    | Nombre del firmante                  | `custom-hsm-device-02` |
    | Archivo de cadena de certificados completa   | *./certs/new-device-02-full-chain.cert.pem* |
    | Nombre de archivo de clave privada          | *private/new-device-02.key.pem* |
    

## <a name="verify-ownership-of-the-root-certificate"></a>Verificación de la propiedad del certificado raíz

1. Cargue el certificado raíz (`./certs/azure-iot-test-only.root.ca.cert.pem`) y obtenga un código de verificación de DPS siguiendo las instrucciones descritas en [Registro de la parte pública de un certificado X.509 y obtención de un código de verificación](how-to-verify-certificates.md#register-the-public-part-of-an-x509-certificate-and-get-a-verification-code).

2. Una vez que tenga un código de verificación de DPS para el certificado raíz, ejecute el siguiente comando desde el directorio de trabajo de los scripts de certificado para generar un certificado de verificación.
 
    El código de comprobación que se proporciona aquí es solo un ejemplo. Use el código generado desde DPS.    

    ```Bash
    ./certGen.sh create_verification_certificate 1B1F84DE79B9BD5F16D71E92709917C2A1CA19D5A156CB9F    
    ```    

    Este script crea un certificado firmado por el certificado raíz con el nombre de firmante establecido en el código de verificación. Este certificado permite a DPS verificar que tiene acceso a la clave privada del certificado raíz. Observe la ubicación del certificado de verificación en la salida del script. Este certificado se genera con el formato `.pfx`.

    ```output
    Leaf Device PFX Certificate Generated At:
    --------------------------------------------
        ./certs/verification-code.cert.pfx
    ```

3. Como se menciona en [Carga del certificado de verificación firmado](how-to-verify-certificates.md#upload-the-signed-verification-certificate), cargue el certificado de verificación y haga clic en **Verificar** en DPS para completar la prueba de posesión del certificado raíz.


## <a name="update-the-certificate-store-on-windows-based-devices"></a>Actualización del almacén de certificados en dispositivos basados en Windows

En dispositivos que no usan Windows, puede pasar la cadena de certificados desde el código como el almacén de certificados.

En los dispositivos basados en Windows, debe agregar los certificados de firma (raíz e intermedio) a un [almacén de certificados](/windows/win32/secauthn/certificate-stores) de Windows. De lo contrario, los certificados de firma no se transportarán a DPS por un canal seguro con Seguridad de la capa de transporte (TLS).

> [!TIP]
> También es posible usar OpenSSL en lugar del canal seguro (Schannel) con el SDK de C. Para más información sobre el uso de OpenSSL, consulte [Uso de OpenSSL en el SDK](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md#using-openssl-in-the-sdk).

Para agregar los certificados de firma al almacén de certificados en dispositivos basados en Windows:

1. En un símbolo del sistema de Bash de Git, vaya al subdirectorio `certs` que contiene los certificados de firma y conviértalos a formato `.pfx` como se indica a continuación.

    Certificado raíz:

    ```bash
    winpty openssl pkcs12 -inkey ../private/azure-iot-test-only.root.ca.key.pem -in ./azure-iot-test-only.root.ca.cert.pem -export -out ./root.pfx
    ```
    
    Certificado intermedio:   

    ```bash
    winpty openssl pkcs12 -inkey ../private/azure-iot-test-only.intermediate.key.pem -in ./azure-iot-test-only.intermediate.cert.pem -export -out ./intermediate.pfx
    ```

2. Haga clic con el botón derecho en el botón **Inicio** de Windows. A continuación, haga clic con el botón izquierdo en **Ejecutar**. Escriba *certmgr.msc* y haga clic en **Aceptar** para iniciar el complemento MMC del administrador de certificados.

3. En el administrador de certificados, en **Certificados: usuario actual**, haga clic en **Entidades de certificación raíz de confianza**. A continuación, en el menú, haga clic en **Acción** > **Todas las tareas** > **Importar** para importar el archivo `root.pfx`.

    * Asegúrese de buscar por **Intercambio de información personal (.pfx)** .
    * Use `1234` como contraseña.
    * Coloque el certificado en el almacén de certificados **Entidades de certificación raíz de confianza**.

4. En el administrador de certificados, en **Certificados: usuario actual**, haga clic en **Entidades de certificación intermedias**. A continuación, en el menú, haga clic en **Acción** > **Todas las tareas** > **Importar** para importar el archivo `intermediate.pfx`.

    * Asegúrese de buscar por **Intercambio de información personal (.pfx)** .
    * Use `1234` como contraseña.
    * Coloque el certificado en el almacén de certificados **Entidades de certificación intermedias**.

Los certificados de firma ahora son de confianza en el dispositivo basado en Windows y se puede transportar la cadena completa a DPS.



## <a name="create-an-enrollment-group"></a>Creación de un grupo de inscripción

1. Inicie sesión en Azure Portal, seleccione el botón **Todos los recursos** en el menú de la izquierda y abra Device Provisioning Service.

2. Seleccione la pestaña **Administrar inscripciones** y, después, seleccione el botón **Agregar grupo de inscripciones** en la parte superior.

3. En el panel **Agregar grupo de inscripciones**, escriba la siguiente información y, a continuación, pulse el botón **Guardar**.

      ![Adición de un grupo de inscripción para la atestación X.509 en el portal](./media/tutorial-custom-hsm-enrollment-group-x509/custom-hsm-enrollment-group-x509.png#lightbox)

    | Campo        | Value           |
    | :----------- | :-------------- |
    | **Nombre de grupo** | Para este tutorial, escriba **custom-hsm-x509-devices**. |
    | **Tipo de atestación** | Seleccione **Certificado**. |
    | **Dispositivo de IoT Edge** | Seleccione **Falso**. |
    | **Tipo de certificado** | Seleccione **Certificado intermedio**. |
    | **Archivo .pem o .cer del certificado principal** | Vaya al certificado intermedio que creó anteriormente ( *./certs/azure-iot-test-only.intermediate.cert.pem*). |


## <a name="configure-the-provisioning-device-code"></a>Configuración del código del dispositivo de aprovisionamiento

En esta sección, actualizará el código de ejemplo con la información de la instancia de Device Provisioning Service. Si un dispositivo está autenticado, se asignará a un centro de IoT vinculado a la instancia de Device Provisioning Service configurada en esta sección.

1. En Azure Portal, seleccione la pestaña **Información general** de Device Provisioning Service y anote el valor de **_Ámbito de id_** .

    ![Extracción de información del punto de conexión del servicio Device Provisioning desde la hoja del portal](./media/quick-create-simulated-device-x509/extract-dps-endpoints.png) 

2. Inicie Visual Studio y abra el nuevo archivo de solución que se creó en el directorio `cmake` que creó en el directorio raíz del repositorio de Git azure-iot-sdk-c. El archivo de solución se llama `azure_iot_sdks.sln`.

3. En el Explorador de soluciones de Visual Studio, vaya a **Provisioning_Samples > prov_dev_client_sample > Archivos de código fuente** y abra *prov_dev_client_sample.c*.

4. Busque la constante `id_scope` y reemplace el valor por el valor de **Ámbito de id.** que copió anteriormente. 

    ```c
    static const char* id_scope = "0ne00000A0A";
    ```

5. Busque la definición de la función `main()` en el mismo archivo. Asegúrese de que la variable `hsm_type` está establecida en `SECURE_DEVICE_TYPE_X509`, tal como se muestra a continuación.

    ```c
    SECURE_DEVICE_TYPE hsm_type;
    //hsm_type = SECURE_DEVICE_TYPE_TPM;
    hsm_type = SECURE_DEVICE_TYPE_X509;
    //hsm_type = SECURE_DEVICE_TYPE_SYMMETRIC_KEY;
    ```

6. Haga clic con el botón derecho en el proyecto **prov\_dev\_client\_sample** y seleccione **Set as Startup Project** (Establecer como proyecto de inicio).


## <a name="configure-the-custom-hsm-stub-code"></a>Configuración del código auxiliar del HSM personalizado

Los detalles de la interacción con el almacenamiento seguro real basado en hardware varían en función del hardware. Como resultado, las cadenas de certificados usadas por los dispositivos simulados en este tutorial estarán codificada en el código auxiliar del HSM personalizado. En un escenario real, la cadena de certificados se almacenaría en el hardware del HSM real para proporcionar una mejor seguridad para la información confidencial. A continuación, se implementarán métodos similares a los métodos de código auxiliar usados en este ejemplo para leer los secretos de ese almacenamiento basado en hardware. 

Aunque el hardware de HSM no sea un requisito, se recomienda proteger la información confidencial, como la clave privada del certificado. Si el ejemplo llama a un HSM real, la clave privada no estará presente en el código fuente. La clave en el código fuente queda expuesta a cualquier persona que pueda ver el código. Únicamente se ha hecho así en este artículo con fines de aprendizaje.

Para actualizar el código auxiliar del HSM personalizado con el fin de simular la identidad del dispositivo con el identificador `custom-hsm-device-01`, complete los pasos siguientes:

1. En el Explorador de soluciones de Visual Studio, vaya a **Provisioning_Samples > custom_hsm_example > Archivos de código fuente** y abra *custom_hsm_example.c*.

2. Actualice el valor de cadena de la constante de cadena `COMMON_NAME` con el nombre común que utilizó al generar el certificado de dispositivo.

    ```c
    static const char* const COMMON_NAME = "custom-hsm-device-01";
    ```

3. En el mismo archivo, debe actualizar el valor de cadena de la cadena de constante `CERTIFICATE` con el texto de la cadena de certificados que guardó en *./certs/new-device-01-full-chain.cert.pem* después de generar los certificados.

    La sintaxis del texto del certificado debe seguir el patrón que aparece a continuación, sin espacios adicionales ni análisis realizados por Visual Studio.

    ```c
    // <Device/leaf cert>
    // <intermediates>
    // <root>
    static const char* const CERTIFICATE = "-----BEGIN CERTIFICATE-----\n"
    "MIIFOjCCAyKgAwIBAgIJAPzMa6s7mj7+MA0GCSqGSIb3DQEBCwUAMCoxKDAmBgNV\n"
        ...
    "MDMwWhcNMjAxMTIyMjEzMDMwWjAqMSgwJgYDVQQDDB9BenVyZSBJb1QgSHViIENB\n"
    "-----END CERTIFICATE-----\n"
    "-----BEGIN CERTIFICATE-----\n"
    "MIIFPDCCAySgAwIBAgIBATANBgkqhkiG9w0BAQsFADAqMSgwJgYDVQQDDB9BenVy\n"
        ...
    "MTEyMjIxMzAzM1owNDEyMDAGA1UEAwwpQXp1cmUgSW9UIEh1YiBJbnRlcm1lZGlh\n"
    "-----END CERTIFICATE-----\n"
    "-----BEGIN CERTIFICATE-----\n"
    "MIIFOjCCAyKgAwIBAgIJAPzMa6s7mj7+MA0GCSqGSIb3DQEBCwUAMCoxKDAmBgNV\n"
        ...
    "MDMwWhcNMjAxMTIyMjEzMDMwWjAqMSgwJgYDVQQDDB9BenVyZSBJb1QgSHViIENB\n"
    "-----END CERTIFICATE-----";        
    ```

    Actualizar este valor de cadena correctamente en este paso puede ser muy tedioso y estar sujeto a errores. Para generar la sintaxis correcta en el símbolo del sistema de Git Bash, copie y pegue los siguientes comandos de shell de bash en el símbolo del sistema de Git Bash y presione **Entrar**. Estos comandos generarán la sintaxis del valor de la constante de cadena `CERTIFICATE`.

    ```Bash
    input="./certs/new-device-01-full-chain.cert.pem"
    bContinue=true
    prev=
    while $bContinue; do
        if read -r next; then
          if [ -n "$prev" ]; then   
            echo "\"$prev\\n\""
          fi
          prev=$next  
        else
          echo "\"$prev\";"
          bContinue=false
        fi  
    done < "$input"
    ```

    Copie y pegue el texto del certificado de salida para el nuevo valor de constante. 


4. En el mismo archivo, también se debe actualizar el valor de cadena de la constante `PRIVATE_KEY` con la clave privada del certificado de dispositivo.

    La sintaxis del texto de la clave privada debe seguir el patrón que aparece a continuación, sin espacios adicionales ni análisis realizados por Visual Studio.

    ```c
    static const char* const PRIVATE_KEY = "-----BEGIN RSA PRIVATE KEY-----\n"
    "MIIJJwIBAAKCAgEAtjvKQjIhp0EE1PoADL1rfF/W6v4vlAzOSifKSQsaPeebqg8U\n"
        ...
    "X7fi9OZ26QpnkS5QjjPTYI/wwn0J9YAwNfKSlNeXTJDfJ+KpjXBcvaLxeBQbQhij\n"
    "-----END RSA PRIVATE KEY-----";
    ```

    Actualizar este valor de cadena correctamente en este paso puede ser también muy tedioso y estar sujeto a errores. Para generar la sintaxis correcta en el símbolo del sistema de Git Bash, copie y pegue los siguientes comandos del shell de bash y presione **Entrar**. Estos comandos generarán la sintaxis del valor de la constante de cadena `PRIVATE_KEY`.

    ```Bash
    input="./private/new-device-01.key.pem"
    bContinue=true
    prev=
    while $bContinue; do
        if read -r next; then
          if [ -n "$prev" ]; then   
            echo "\"$prev\\n\""
          fi
          prev=$next  
        else
          echo "\"$prev\";"
          bContinue=false
        fi  
    done < "$input"
    ```

    Copie y pegue el texto de la clave privada de salida para el nuevo valor de la constante. 

5. Guarde el archivo *custom_hsm_example.c*.

6. En el menú de Visual Studio, seleccione **Depurar** > **Iniciar sin depurar** para ejecutar la solución. Cuando se le pida que recompile el proyecto, seleccione **Sí** para recompilar el proyecto antes de su ejecución.

    La salida siguiente es un ejemplo de arranque correcto del dispositivo simulado `custom-hsm-device-01` y su conexión al servicio de aprovisionamiento. El dispositivo se ha asignado a un centro de IoT y se ha registrado:

    ```cmd
    Provisioning API Version: 1.3.9
    
    Registering Device
    
    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    
    Registration Information received from service: test-docs-hub.azure-devices.net, deviceId: custom-hsm-device-01
    Press enter key to exit:
    ```

7. En el portal, vaya hasta el centro de IoT vinculado a su servicio de aprovisionamiento y seleccione la pestaña **Dispositivos IoT**. Si el dispositivo X.509 se ha aprovisionado correctamente en el centro, su identificador de dispositivo aparecerá en la hoja **Dispositivos IoT** y en *ESTADO* aparecerá el valor **Habilitado**. Es posible que tenga que pulsar el botón **Actualizar** en la parte superior. 

    ![El dispositivo HSM personalizado se ha registrado con el centro de IoT.](./media/tutorial-custom-hsm-enrollment-group-x509/hub-provisioned-custom-hsm-x509-device.png) 

8. Repita los pasos 1 a 7 para un segundo dispositivo con id. de dispositivo `custom-hsm-device-02`. Utilice los siguientes valores para ese dispositivo:

    |   Descripción                 |  Value  |
    | :---------------------------- | :--------- |
    | `COMMON_NAME`                 | `"custom-hsm-device-02"` |
    | Cadena de certificados completa        | Generación del texto con `input="./certs/new-device-02-full-chain.cert.pem"` |
    | Clave privada                   | Generación del texto con `input="./private/new-device-02.key.pem"` |

    La salida siguiente es un ejemplo de arranque correcto del dispositivo simulado `custom-hsm-device-02` y su conexión al servicio de aprovisionamiento. El dispositivo se ha asignado a un centro de IoT y se ha registrado:

    ```cmd
    Provisioning API Version: 1.3.9
    
    Registering Device
    
    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    
    Registration Information received from service: test-docs-hub.azure-devices.net, deviceId: custom-hsm-device-02
    Press enter key to exit:
    ```


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado de probar y explorar este ejemplo de cliente de dispositivo, siga estos pasos para eliminar todos los recursos creados en este tutorial.

1. Cierre la ventana de salida de ejemplo del cliente del dispositivo en su máquina.
1. En el menú de la izquierda de Azure Portal, seleccione **Todos los recursos** y seleccione Device Provisioning Service. Abra **Administrar inscripciones** en el servicio y, a continuación, seleccione la pestaña **Grupos de inscripción**. Active la casilla situada junto al campo *Nombre del grupo* del grupo de dispositivos que ha creado en este tutorial y pulse el botón **Eliminar** situado en la parte superior del panel. 
1. Haga clic en **Certificados** en DPS. Para cada certificado que ha cargado y verificado en este tutorial, haga clic en el certificado y haga clic en el botón **Eliminar** para eliminarlo.
1. En el menú de la izquierda de Azure Portal, seleccione **Todos los recursos** y después su centro de IoT. Abra la opción **Dispositivos IoT** del centro. Seleccione la casilla situada junto al campo *Id. de dispositivo* del dispositivo que registró en este tutorial. Haga clic en el botón **Eliminar** situado en la parte superior del panel.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprovisionado un dispositivo X.509 con un HSM personalizado en el centro de IoT. Para obtener información sobre cómo aprovisionar dispositivos IoT en varios centros, continúe con el siguiente tutorial. 

> [!div class="nextstepaction"]
> [Tutorial: Aprovisionamiento de dispositivos en instancias de IoT Hub con equilibrio de carga](tutorial-provision-multiple-hubs.md)