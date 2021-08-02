---
title: Conectividad de dispositivos en Azure IoT Central | Microsoft Docs
description: En este artículo se presentan conceptos clave relacionados con la conectividad de dispositivos en Azure IoT Central
author: dominicbetts
ms.author: dobett
ms.date: 1/15/2020
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: philmea
ms.custom:
- amqp
- mqtt
- device-developer
ms.openlocfilehash: fb9c9f460b46f8dec741f4c22460cbe9d44c6a0e
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110791117"
---
# <a name="get-connected-to-azure-iot-central"></a>Conexión a Azure IoT Central

En este artículo se describe cómo se conectan los dispositivos a una aplicación de Azure IoT Central. Para que un dispositivo pueda intercambiar datos con IoT Central, debe hacer lo siguiente:

- *Autenticación*. La autenticación con la aplicación de IoT Central usa un _token de firma de acceso compartido (SAS)_ o un _certificado X.509_. Los certificados X.509 se recomiendan en entornos de producción.
- *Registro*. Los dispositivos deben estar registrados con la aplicación de IoT Central. Puede ver los dispositivos registrados en la página **Dispositivos** de la aplicación.
- *Asociación a una plantilla de dispositivo*. En una aplicación de IoT Central, las plantillas de dispositivo definen la interfaz de usuario que usan los operadores para ver y administrar los dispositivos conectados.

IoT Central admite los siguientes dos escenarios de registro de dispositivos:

- *Registro automático*. El dispositivo se registra automáticamente cuando se conecta por primera vez. Este escenario permite a los OEM fabricar dispositivos de forma masiva que puedan conectarse sin registrarse primero. Un OEM genera las credenciales de dispositivo adecuadas y configura los dispositivos en la fábrica. Opcionalmente, puede requerir que un operador apruebe el dispositivo antes de empezar a enviar datos. Este escenario requiere que configure una _inscripción de grupo_ X.509 o SAS en la aplicación.
- *Registro manual*. Los operadores registran dispositivos individuales en la página **Dispositivos**, o bien [importan un archivo .csv](howto-manage-devices.md#import-devices) a los dispositivos registrados de forma masiva. En este escenario, puede usar la _inscripción de grupo_ X.509 o SAS, o bien la _inscripción individual_ X.509 o SAS.

Los dispositivos que se conectan a IoT Central deben seguir las *convenciones de IoT Plug and Play*. Una de estas convenciones es que un dispositivo debe enviar el _identificador de modelo_ del modelo de dispositivo que implementa cuando se conecta. El identificador de modelo permite a la aplicación de IoT Central asociar el dispositivo a la plantilla de dispositivo correcta.

IoT Central usa el [servicio Azure IoT Hub Device Provisioning (DPS)](../../iot-dps/about-iot-dps.md) para administrar el proceso de conexión. Un dispositivo se conecta primero a un punto de conexión de DPS para recuperar la información necesaria para conectarse a la aplicación. De manera interna, la aplicación de IoT Central usa un centro de IoT para administrar la conectividad de los dispositivos. El uso de DPS permite:

- Que IoT Central admite la incorporación y la conexión de dispositivos a escala.
- Que usted pueda generar credenciales de dispositivo y configurar los dispositivos sin conexión sin tener que registrarlos primero en la interfaz de usuario de IoT Central.
- Que usted pueda usar identificadores de dispositivo propios para registrar estos en IoT Central. Esto simplifica la integración con sistemas del área de operaciones existentes.
- Una manera sencilla y coherente de conectar dispositivos a IoT Central.

En este artículo se describen los siguientes pasos de conexión de dispositivos:

- [Inscripción de grupo X.509](#x509-group-enrollment)
- [Inscripción de grupo SAS](#sas-group-enrollment)
- [Inscripción individual](#individual-enrollment)
- [Registro de dispositivos](#device-registration)
- [Asociación de un dispositivo a una plantilla de dispositivo](#associate-a-device-with-a-device-template)

## <a name="x509-group-enrollment"></a>Inscripción de grupo X.509

En un entorno de producción, se recomienda el uso de certificados X.509 como mecanismo de autenticación de dispositivos para IoT Central. Para más información consulte [Autenticación de dispositivos mediante certificados de entidades de certificación X.509](../../iot-hub/iot-hub-x509ca-overview.md).

Para conectar un dispositivo con un certificado X.509 a la aplicación, haga lo siguiente:

1. Cree un *grupo de inscripción* que use el tipo de atestación **Certificados (X.509)** .
1. Agregue y verifique un certificado X.509 intermedio o raíz en el grupo de inscripción.
1. Genere un certificado de hoja a partir del certificado raíz o intermedio en el grupo de inscripción. Envíe el certificado de hoja desde el dispositivo cuando se conecte a la aplicación.

Para más información, vea [Procedimiento para conectar dispositivos con certificados X.509](how-to-connect-devices-x509.md).

### <a name="for-testing-purposes-only"></a>Solo para fines de prueba

Solo para pruebas, puede usar las siguientes utilidades para generar certificados raíz, intermedio y de dispositivo:

- [Herramientas del SDK de dispositivo de Azure IoT Device Provisioning](https://github.com/Azure/azure-iot-sdk-node/blob/master/provisioning/tools/readme.md): colección de herramientas de Node.js que se puede usar para generar y verificar claves y certificados X.509.
- Si usa un dispositivo DevKit, esta [herramienta de la línea de comandos](https://aka.ms/iotcentral-docs-dicetool) genera un certificado de entidad de certificación que puede agregar a la aplicación de IoT Central para verificar los certificados.
- [Administración de certificados de entidad de certificación de prueba para ejemplos y tutoriales](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md): una colección de scripts de PowerShell y Bash para:
  - Crear una cadena de certificados.
  - Guardar los certificados como archivos .cer para cargarlos en la aplicación de IoT Central.
  - Usar el código de verificación de la aplicación de IoT Central para generar el certificado de verificación.
  - Crear certificados de hoja para los dispositivos mediante sus identificadores de dispositivo como parámetro en la herramienta.

## <a name="sas-group-enrollment"></a>Inscripción de grupo SAS

Para conectar un dispositivo con una clave SAS de dispositivo a la aplicación, haga lo siguiente:

1. Cree un *grupo de inscripción* que use el tipo de atestación **Firma de acceso compartido (SAS)** .
1. Copie la clave principal o secundaria del grupo desde el grupo de inscripción.
1. Use la CLI de Azure para generar una clave de dispositivo a partir de la clave de grupo:

    ```azurecli
    az iot central device compute-device-key --primary-key <enrollment group primary key> --device-id <device ID>
    ```

1. Use la clave de dispositivo generada cuando el dispositivo se conecte a la aplicación de IoT Central.

## <a name="individual-enrollment"></a>Inscripción individual

Para los clientes que conectan dispositivos que tienen sus propias credenciales de autenticación, use inscripciones individuales. Una inscripción individual es una entrada para un único dispositivo que tiene permiso para conectarse. Las inscripciones individuales pueden usar certificados X.509 de hoja o tokens de SAS (de un módulo de plataforma segura físico o virtual) como mecanismos de atestación. Un identificador de dispositivo puede contener letras, números y el carácter `-`. Para más información, consulte [Inscripción individual de DPS](../../iot-dps/concepts-service.md#individual-enrollment).

> [!NOTE]
> Cuando se crea una inscripción individual para un dispositivo, tiene prioridad sobre las opciones de inscripción de grupo predeterminadas de la aplicación de IoT Central.

### <a name="create-individual-enrollments"></a>Creación de inscripciones individuales

IoT Central admite los siguientes mecanismos de atestación para las inscripciones individuales:

- **Atestación de clave simétrica:** la atestación de clave simétrica es un enfoque sencillo para autenticar un dispositivo con una instancia de DPS. Para crear una inscripción individual que use claves simétricas, abra la página **Conexión del dispositivo** del dispositivo, seleccione **Inscripción individual** como método de conexión y **Firma de acceso compartido (SAS)** como mecanismo. Escriba las claves principal y secundaria codificadas en Base64 y guarde los cambios. Use el valor de **ID scope** (Ámbito de id.), **Device ID** (Id. de dispositivo) y la clave principal o secundaria para conectarse al dispositivo.

    > [!TIP]
    > Con fines de prueba, puede usar **OpenSSL** para generar claves codificadas en Base64: `openssl rand -base64 64`

- **Certificados X.509:** Para crear una inscripción individual con certificados X.509, abra la página **Device Connection** (Conexión de dispositivos), seleccione **Individual enrollment** (Inscripción individual) como método de conexión y **Certificates (X.509)** (Certificados [X.509]) como mecanismo. Los certificados de dispositivo usados con una entrada de inscripción individual tienen el requisito de que el emisor y el CN del firmante están establecidos en el identificador de dispositivo.

    > [!TIP]
    > Con fines de prueba, puede usar las [herramientas para el SDK de dispositivo de Azure IoT Device Provisioning para Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/tools) para generar un certificado autofirmado: `node create_test_cert.js device "mytestdevice"`

- **Atestación del Módulo de plataforma segura (TPM):** Un [TPM](../../iot-dps/concepts-tpm-attestation.md) es un tipo de módulo de seguridad de hardware. El uso de un TPM es una de las formas más seguras de conectar un dispositivo. En este artículo se supone que usa un TPM discreto, de firmware o integrado. Los TPM emulados por software son adecuados para prototipos o pruebas, pero no brindan el mismo nivel de seguridad que los TPM discretos, de firmware o integrados. No use TPM de software en producción. Para crear una inscripción individual que use un TPM, abra la página **Device Connection** (Conexión de dispositivos), seleccione **Individual enrollment** (Inscripción individual) como método de conexión y **TPM** como mecanismo. Escriba la clave de aprobación de TPM y guarde la información de conexión del dispositivo.

## <a name="device-registration"></a>Registro de dispositivos

Para que un dispositivo pueda conectarse a una aplicación de IoT Central, debe estar registrado en la aplicación:

- Los dispositivos pueden registrarse automáticamente al conectarse por primera vez. Para usar esta opción, debe usar la [inscripción de grupo X.509](#x509-group-enrollment) o la [inscripción de grupo SAS](#sas-group-enrollment).
- Un operador puede importar un archivo .csv para registrar de forma masiva una lista de dispositivos en la aplicación.
- Un operador puede registrar manualmente un dispositivo individual en la página **Dispositivos** de la aplicación.

IoT Central permite a los OEM fabricar dispositivos de forma masiva que pueden registrarse automáticamente. Un OEM genera las credenciales de dispositivo adecuadas y configura los dispositivos en la fábrica. Cuando un cliente enciende un dispositivo por primera vez, se conecta a DPS, que, a continuación, conecta automáticamente el dispositivo a la aplicación de IoT Central correcta. Opcionalmente, puede requerir que un operador apruebe el dispositivo antes de que este empiece a enviar datos a la aplicación.

> [!TIP]
> En la página **Administración > Conexión del dispositivo**, la opción **Aprobación automática** controla si un operador debe aprobar el dispositivo manualmente antes de que pueda empezar a enviar datos.

### <a name="automatically-register-devices-that-use-x509-certificates"></a>Registro automático de dispositivos que usan certificados X.509

1. Genere certificados de hoja para los dispositivos con el certificado raíz o intermedio que agregó al [grupo de inscripción X.509](#x509-group-enrollment). Use los identificadores de dispositivo como `CNAME` en los certificados de hoja. Un identificador de dispositivo puede contener letras, números y el carácter `-`.

1. Como OEM, cargue cada dispositivo con un identificador de dispositivo, un certificado de hoja X.509 generado y el valor **Ámbito de id.** de la aplicación. El código del dispositivo también debe enviar el identificador del modelo de dispositivo que implementa.

1. Al encender un dispositivo, primero se conecta a DPS para recuperar la información de conexión de IoT Central.

1. El dispositivo usa la información de DPS para conectarse y registrarse en la aplicación de IoT Central.

La aplicación de IoT Central usa el identificador de modelo enviado por el dispositivo para [asociar el dispositivo registrado a una plantilla de dispositivo](#associate-a-device-with-a-device-template).

### <a name="automatically-register-devices-that-use-sas-tokens"></a>Registro automático de dispositivos que usan tokens de SAS

1. Copie la clave principal del grupo desde el grupo de inscripción **SAS-IoT-Devices**:

    :::image type="content" source="media/concepts-get-connected/group-primary-key.png" alt-text="Clave principal del grupo desde el grupo de inscripción SAS-IoT-Devices":::

1. Use el comando `az iot central device compute-device-key` para generar las claves SAS del dispositivo. Use la clave principal del grupo del paso anterior. El identificador de dispositivo puede contener letras, números y el carácter `-`:

    ```azurecli
    az iot central device compute-device-key --primary-key <enrollment group primary key> --device-id <device ID>
    ```

1. Como OEM, cargue cada dispositivo con el identificador del dispositivo, la clave SAS de dispositivo generada y el valor de **Ámbito de id.** de la aplicación. El código del dispositivo también debe enviar el identificador del modelo de dispositivo que implementa.

1. Al encender un dispositivo, primero se conecta a DPS para recuperar la información de registro de IoT Central.

1. El dispositivo usa la información de DPS para conectarse y registrarse en la aplicación de IoT Central.

La aplicación de IoT Central usa el identificador de modelo enviado por el dispositivo para [asociar el dispositivo registrado a una plantilla de dispositivo](#associate-a-device-with-a-device-template).

### <a name="bulk-register-devices-in-advance"></a>Registro anticipado de dispositivos de forma masiva

Para registrar un gran número de dispositivos con la aplicación de IoT Central, use un archivo CSV para [importar identificadores y nombres de dispositivo](howto-manage-devices.md#import-devices).

Si los dispositivos usan tokens de SAS para realizar la autenticación, [exporte un archivo .csv desde la aplicación de IoT Central](howto-manage-devices.md#export-devices). El archivo CSV exportado incluye los identificadores de dispositivo y las claves SAS.

Si los dispositivos usan certificados X.509 para realizar la autenticación, genere certificados X.509 de hoja para los dispositivos con el certificado raíz o intermedio que cargó en el grupo de inscripción X.509. Use el identificador de dispositivo que importó como el valor `CNAME` en los certificados de hoja.

Los dispositivos deben usar el valor de **Ámbito de id.** de la aplicación y enviar un identificador de modelo cuando se conecten.

> [!TIP]
> Puede encontrar el valor de **Ámbito de id.** en **Administración > Conexión del dispositivo**.

### <a name="register-a-single-device-in-advance"></a>Registro anticipado de un único dispositivo

Este enfoque es útil cuando está experimentando con IoT Central o probado dispositivos. Seleccione **+ Nuevo** en la página **Dispositivos** para registrar un dispositivo individual. Puede usar las claves SAS de conexión de dispositivos para conectar el dispositivo a la aplicación de IoT Central. Copie la _clave SAS del dispositivo_ de la información de conexión de un dispositivo registrado:

![Claves SAS de un dispositivo individual](./media/concepts-get-connected/single-device-sas.png)

## <a name="associate-a-device-with-a-device-template"></a>Asociación de un dispositivo a una plantilla de dispositivo

IoT Central asocia automáticamente un dispositivo a una plantilla de dispositivo cuando el dispositivo se conecta. Un dispositivo envía un [identificador de modelo](../../iot-fundamentals/iot-glossary.md?toc=/azure/iot-central/toc.json&bc=/azure/iot-central/breadcrumb/toc.json#model-id) cuando se conecta. IoT Central usa el identificador de modelo para identificar la plantilla de dispositivo para ese modelo de dispositivo específico. El proceso de detección funciona de la siguiente manera:

1. Si la plantilla de dispositivo ya está publicada en la aplicación IoT Central, el dispositivo se le asocia.
1. Si la plantilla de dispositivo todavía no está publicada en la aplicación IoT Central, esta busca el modelo del dispositivo en el [repositorio de modelos público](https://github.com/Azure/iot-plugandplay-models). Si IoT Central encuentra el modelo, lo utiliza para generar una plantilla de dispositivo básica.
1. Si IoT Central no encuentra el modelo en el repositorio de modelos público, el dispositivo se marca como **Sin asociar**. Un operador puede crear una plantilla de dispositivo para el dispositivo y, a continuación, migrar el dispositivo sin asociar a la nueva plantilla de dispositivo.

En la captura de pantalla siguiente se muestra cómo ver el identificador de modelo de una plantilla de dispositivo en IoT Central. En una plantilla de dispositivo, seleccione un componente y luego **Editar identidad**:

:::image type="content" source="media/concepts-get-connected/model-id.png" alt-text="Captura de pantalla que muestra el identificador de modelo de la plantilla de dispositivo thermostat.":::

Puede ver el [modelo de thermostat](https://github.com/Azure/iot-plugandplay-models/blob/main/dtmi/com/example/thermostat-1.json) en el repositorio de modelos públicos. La definición del identificador de modelo tiene este aspecto:

```json
"@id": "dtmi:com:example:Thermostat;1"
```

## <a name="device-status-values"></a>Valores de estado del dispositivo

Cuando un dispositivo real se conecta a la aplicación de IoT Central, su estado de dispositivo cambia como sigue:

1. Primero, el estado del dispositivo es **Registrado**. Este estado significa que el dispositivo se ha creado en IoT Central y tiene un identificador. El dispositivo está registrado cuando:
    - Se agrega un nuevo dispositivo real en la página **Dispositivos**.
    - Se agrega un conjunto de dispositivos mediante **Importar** en la página **Dispositivos**.

1. El estado del dispositivo cambia a **Aprovisionado** cuando el dispositivo conectado a la aplicación de IoT Central con credenciales válidas completa el paso de aprovisionamiento. En este paso, el dispositivo usa DPS para recuperar automáticamente una cadena de conexión de IoT Hub que usa la aplicación de IoT Central. El dispositivo puede conectarse ahora a IoT Hub y empezar a enviar datos.

1. Los dispositivos los puede bloquear un operador. Cuando están bloqueados, no pueden enviar datos a la aplicación de IoT Central. Los dispositivos bloqueados tienen un estado de **Bloqueado**. Un operador debe restablecer el dispositivo para que pueda volver a enviar datos. Cuando un operador desbloquea un dispositivo, el estado vuelve a su valor anterior, **Registrado** o **Aprovisionado**.

1. Si el estado del dispositivo es **Waiting for Approval** (Esperando aprobación), significa que la opción **Auto approve** (Aprobación automática) está deshabilitada. Un operador debe aprobar explícitamente un dispositivo antes de que empiece a enviar datos. Los dispositivos no registrados manualmente en la página **Dispositivos**, pero conectados con credenciales válidas, tendrán el estado del dispositivo **En espera de aprobación**. Los operadores pueden aprobar estos dispositivos desde la página **Dispositivos** mediante el botón **Aprobar**.

1. Si el estado del dispositivo es **Unassociated** (No asociado), significa que el dispositivo que se conecta a IoT Central no tiene una plantilla de dispositivo asociada. Esta situación suele darse en los escenarios siguientes:

    - Se agrega un conjunto de dispositivos mediante la opción **Import** (Importar) de la página **Devices** (Dispositivos) sin especificar la plantilla de dispositivo.
    - Se registró un dispositivo manualmente en la página **Devices** (Dispositivos) sin especificar la plantilla de dispositivo. Después, el dispositivo se conecta con credenciales válidas.  

    Un operador puede asociar un dispositivo a una plantilla de dispositivo desde la página **Dispositivos** mediante el botón **Migrar**.

## <a name="device-connection-status"></a>Estado de conexión del dispositivo
Cuando un dispositivo o dispositivo perimetral se conecta mediante el protocolo MQTT, se muestran eventos _conectados_ y _desconectados_ para el dispositivo. Estos eventos no se envían mediante los envíos del dispositivo; IoT Central los genera internamente.

En el diagrama siguiente se muestra cómo, cuando se conecta un dispositivo, la conexión se registra al final de una ventana de tiempo. Si se producen varios eventos de conexión y desconexión, IoT Central registra el más cercano al final de la ventana de tiempo. Por ejemplo, si un dispositivo se desconecta y se vuelve a conectar dentro de la ventana de tiempo, IoT Central registra el evento de conexión. Actualmente, la ventana de tiempo es de aproximadamente un minuto.

:::image type="content" source="media/concepts-get-connected/device-connectivity-diagram.png" alt-text="Diagrama que muestra la ventana de eventos para eventos conectados y desconectados" border="false":::.

Puede incluir eventos de conexión y desconexión en las [exportaciones desde IoT Central](howto-export-data.md#set-up-data-export). Para más información, vea [Reacción a eventos de IoT Hub > Limitaciones de los eventos de dispositivo conectado y dispositivo desconectado](../../iot-hub/iot-hub-event-grid.md#limitations-for-device-connected-and-device-disconnected-events).

## <a name="sdk-support"></a>Compatibilidad con SDK

La oferta de SDK de dispositivos de Azure es la manera más fácil de implementar el código del dispositivo. En la actualidad, están disponibles los siguientes SDK:

- [SDK de Azure IoT para C](https://github.com/azure/azure-iot-sdk-c)
- [SDK de Azure IoT para Python](https://github.com/azure/azure-iot-sdk-python)
- [SDK de Azure IoT para Node.js](https://github.com/azure/azure-iot-sdk-node)
- [SDK de Azure IoT para Java](https://github.com/azure/azure-iot-sdk-java)
- [SDK de Azure IoT para .NET](https://github.com/azure/azure-iot-sdk-csharp)

### <a name="sdk-features-and-iot-hub-connectivity"></a>Características del SDK y la conectividad de IoT Hub

Toda comunicación del dispositivo con la instancia de IoT Hub utiliza las siguientes opciones de conectividad:

- [Mensajería del dispositivo a la nube](../../iot-hub/iot-hub-devguide-messages-d2c.md)
- [Mensajería de la nube a un dispositivo](../../iot-hub/iot-hub-devguide-messages-c2d.md)
- [Dispositivos gemelos](../../iot-hub/iot-hub-devguide-device-twins.md)

En la tabla siguiente se resume cómo las características de los dispositivos de Azure IoT Central se asignan a las características de IoT Hub:

| Azure IoT Central | Azure IoT Hub |
| ----------- | ------- |
| Telemetría | Mensajería de un dispositivo a la nube |
| Comandos sin conexión | Mensajería de la nube a un dispositivo |
| Propiedad | Propiedades notificadas de dispositivos gemelos |
| Propiedad (grabable) | Propiedades deseadas y notificadas de dispositivos gemelos |
| Get-Help | Métodos directos |

### <a name="protocols"></a>Protocolos

Los SDK de dispositivo admiten los siguientes protocolos de red para conectarse a un centro de IoT:

- MQTT
- AMQP
- HTTPS

Para obtener información sobre estos protocolos de diferencia y guía sobre cómo elegir uno, consulte [Elección de un protocolo de comunicación](../../iot-hub/iot-hub-devguide-protocols.md).

Si el dispositivo no puede usar ninguno de los protocolos admitidos, utilice Azure IoT Edge para realizar la conversión de protocolos. IoT Edge es compatible con otros escenarios de inteligencia perimetral para descargar el procesamiento desde la aplicación de Azure IoT Central.

## <a name="security"></a>Seguridad

Todos los datos intercambiados entre los dispositivos y la instancia de Azure IoT Central están cifrados. IoT Hub autentica todas las solicitudes de un dispositivo que se conecta a cualquiera de los puntos de conexión de IoT Hub orientados al dispositivo. Para evitar el intercambio de credenciales por la red, el dispositivo utiliza tokens firmados para autenticarse. Para más información, consulte [Control del acceso a IoT Hub](../../iot-hub/iot-hub-devguide-security.md).

## <a name="next-steps"></a>Pasos siguientes

Algunos pasos siguientes sugeridos son:

- Revise los [procedimientos recomendados](concepts-best-practices.md) para el desarrollo de dispositivos.
- Revisar algún código de ejemplo que muestre cómo usar los tokens de SAS en [Tutorial: Creación y conexión de una aplicación cliente a una aplicación de Azure IoT Central](tutorial-connect-device.md)
- Aprender el [procedimiento para conectar dispositivos con certificados X.509 mediante el SDK de dispositivo de Node.js para la aplicación de IoT Central](how-to-connect-devices-x509.md)
- Aprender a [supervisar la conectividad de dispositivos mediante la CLI de Azure](./howto-monitor-devices-azure-cli.md)
- Leer sobre los [dispositivos de Azure IoT Edge y Azure IoT Central](./concepts-iot-edge.md)
