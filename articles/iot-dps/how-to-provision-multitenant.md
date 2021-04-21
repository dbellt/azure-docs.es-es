---
title: Cómo aprovisionar dispositivos para el multiinquilinato en Azure IoT Hub Device Provisioning Service
description: Cómo aprovisionar dispositivos para el multiinquilinato con su instancia de Device Provisioning Service (DPS)
author: wesmc7777
ms.author: wesmc
ms.date: 04/10/2019
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.openlocfilehash: 04ab4ae4d3de6f33f800011b9b0802665d5fb16e
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107228337"
---
# <a name="how-to-provision-for-multitenancy"></a>Cómo aprovisionar para el multiinquilinato 

En este artículo se muestra cómo aprovisionar de forma segura varios dispositivos de claves simétricas para un grupo de centros de IoT mediante una [directiva de asignación](concepts-service.md#allocation-policy). Las directivas de asignación definidas por el servicio de aprovisionamiento admiten una gran variedad de escenarios de asignación. Dos escenarios comunes son:

* **Geolocalización o geolatencia**: puesto que un dispositivo se mueve entre ubicaciones, la latencia de red se mejora mediante su aprovisionamiento en el centro de IoT más cercano a cada ubicación. En este escenario, se selecciona un grupo de centros de IoT, que abarcan varias regiones, para las inscripciones. La directiva de asignación **Latencia más baja** está seleccionada para estas inscripciones. Esta directiva hace que Device Provisioning Service evalúe la latencia del dispositivo y determine el centro de IoT más cercano del grupo de centros de IOT. 

* **Multiinquilinato**: puede que los dispositivos que se usan en una solución de IoT se tengan que asignar a un centro de IoT o un grupo de centros de IoT específicos. La solución puede requerir que todos los dispositivos de un inquilino determinado se comuniquen con un grupo específico de centros de IoT. En algunos casos, un inquilino puede tener centros de IoT y requerir que los dispositivos se asignen a sus centros.

Es habitual combinar estos dos escenarios. Por ejemplo, una solución de IoT multiinquilino normalmente asignará los dispositivos de los inquilinos mediante un grupo de centros de IoT repartidos en regiones. Estos dispositivos de los inquilinos se pueden asignar al centro de IoT de ese grupo que tiene la menor latencia según la ubicación geográfica.

En este artículo, se usa un ejemplo de dispositivo simulado del [SDK de C para Azure IoT](https://github.com/Azure/azure-iot-sdk-c) para demostrar cómo aprovisionar dispositivos en un escenario multiinquilino entre regiones. En este artículo, llevará a cabo los siguientes pasos:

> [!div class="checklist"]
> * Usar la CLI de Azure para crear dos centros de IoT regionales (**Oeste de EE. UU.** y **Este de EE. UU.**).
> * Crear una inscripción multiinquilino.
> * Usar la CLI de Azure para crear dos máquinas virtuales de Linux regionales para actuar como dispositivos en las mismas regiones (**Oeste de EE. UU.** y **Este de EE. UU.**).
> * Configurar el entorno de desarrollo para el SDK de C para Azure IoT en ambas máquinas virtuales de Linux.
> * Simular los dispositivos para ver que están aprovisionados para el mismo inquilino en la región más cercana.


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]


## <a name="prerequisites"></a>Requisitos previos

- Finalización de la guía de inicio rápido [Configuración de Azure IoT Hub Device Provisioning Service con Azure Portal](./quick-setup-auto-provision.md).
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="create-two-regional-iot-hubs"></a>Creación de dos centros de IoT regionales

En esta sección, usará Azure Cloud Shell para crear dos centros de IoT regionales en las regiones **Oeste de EE. UU.** y **Este de EE. UU.** para un inquilino.


1. Use Azure Cloud Shell para crear un grupo de recursos con el comando [az group create](/cli/azure/group#az-group-create). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

    En el ejemplo siguiente, se crea un grupo de recursos denominado *contoso-us-resource-group* en la región *eastus*. Le recomendamos que use este grupo para todos los recursos que crearemos en el artículo. De esta forma, será más fácil limpiar al finalizar.

    ```azurecli-interactive 
    az group create --name contoso-us-resource-group --location eastus
    ```

2. Use Azure Cloud Shell para crear un centro de IoT en la región **eastus** con el comando [az iot hub create](/cli/azure/iot/hub#az-iot-hub-create). El centro de IoT se agregará al grupo *contoso-us-resource-group*.

    En el ejemplo siguiente, se crea un centro de IoT denominado *contoso-east-hub* en la ubicación *eastus*. Debe usar su propio nombre único de centro en lugar de **contoso-east-hub**.

    ```azurecli-interactive 
    az iot hub create --name contoso-east-hub --resource-group contoso-us-resource-group --location eastus --sku S1
    ```
    
    Este comando puede tardar varios minutos en completarse.

3. Use Azure Cloud Shell para crear un centro de IoT en la región **westus** con el comando [az iot hub create](/cli/azure/iot/hub#az-iot-hub-create). Este centro de IoT también se agregará al grupo *contoso-us-resource-group*.

    En el ejemplo siguiente, se crea un centro de IoT denominado *contoso-west-hub* en la ubicación *westus*. Debe usar su propio nombre único de centro en lugar de **contoso-west-hub**.

    ```azurecli-interactive 
    az iot hub create --name contoso-west-hub --resource-group contoso-us-resource-group --location westus --sku S1
    ```

    Este comando puede tardar varios minutos en completarse.



## <a name="create-the-multitenant-enrollment"></a>Creación de la inscripción multiinquilino

En esta sección, creará un grupo de inscripción para los dispositivos del inquilino.  

Para que sea más sencillo, en este artículo se usa la [atestación de clave simétrica](concepts-symmetric-key-attestation.md) con la inscripción. Con el fin de obtener una solución más segura, considere la posibilidad de usar la [atestación de certificado X.509](concepts-x509-attestation.md) con una cadena de confianza.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la instancia de Device Provisioning Service.

2. Seleccione la pestaña **Administrar inscripciones** y, luego, haga clic en el botón **Agregar grupo de inscripciones**, situado en la parte superior de la página. 

3. En **Agregar grupo de inscripciones**, escriba la siguiente información y haga clic en el botón **Guardar**.

    **Nombre del grupo**: escriba **contoso-us-devices**.

    **Tipo de atestación**: seleccione **clave simétrica**.

    **Generar claves automáticamente**: esta casilla ya debe estar seleccionada.

    **Seleccione cómo quiere asignar los dispositivos a los centros**: seleccione **Latencia más baja**.

    ![Agregar grupo de inscripción multiinquilino para la atestación de clave simétrica](./media/how-to-provision-multitenant/create-multitenant-enrollment.png)


4. En **Agregar grupo de inscripciones**, haga clic en **Link a new IoT hub** (Vincular un nuevo centro de IoT) para vincular sus dos centros regionales.

    **Suscripción**: si tiene varias suscripciones, elija aquella en la que ha creado los centros de IoT regionales.

    **Centro de IoT**: seleccione uno de los centros regionales que ha creado.

    **Directiva de acceso**: elija **iothubowner**.

    ![Vinculación de los centros de IoT regionales con el servicio de aprovisionamiento](./media/how-to-provision-multitenant/link-regional-hubs.png)


5. Una vez que se han vinculado ambos centros de IoT regionales, debe seleccionarlos para el grupo de inscripción y hacer clic en **Guardar** para crear el grupo de centros de IoT regionales para la inscripción.

    ![Creación del grupo de centros regionales para la inscripción](./media/how-to-provision-multitenant/enrollment-regional-hub-group.png)


6. Después de guardar la inscripción, vuelva a abrirla y apunte la **clave principal**. Primero debe guardar la inscripción para que se generen las claves. Esta clave se usará para generar claves de dispositivo únicas para ambos dispositivos simulados más adelante.


## <a name="create-regional-linux-vms"></a>Creación de máquinas virtuales regionales de Linux

En esta sección, creará dos máquinas virtuales regionales de Linux. Estas máquinas virtuales ejecutarán un ejemplo de simulación de dispositivo de cada región para demostrar el aprovisionamiento de dispositivos de inquilino desde ambas regiones.

Para que la limpieza sea más sencilla, estas máquinas virtuales se agregarán al mismo grupo de recursos que contiene los centros de IoT que se han creado: *contoso-us-resource-group*. En cambio, las máquinas virtuales se ejecutarán en regiones diferentes (**Oeste de EE. UU.** y **Este de EE. UU.**).

1. En Azure Cloud Shell, ejecute el siguiente comando para crear una máquina virtual de la región **Este de EE. UU.** después de realizar los siguientes cambios de parámetro en el comando:

    **--name**: escriba un nombre único para su máquina virtual de dispositivo regional del **Este de EE. UU.** 

    **--admin-username**: use su propio nombre de usuario administrador.

    **--admin-password**: use su propia contraseña de administrador.

    ```azurecli-interactive
    az vm create \
    --resource-group contoso-us-resource-group \
    --name ContosoSimDeviceEast \
    --location eastus \
    --image Canonical:UbuntuServer:18.04-LTS:18.04.201809110 \
    --admin-username contosoadmin \
    --admin-password myContosoPassword2018 \
    --authentication-type password
    ```

    Este comando tardará varios minutos en completarse. Cuando se haya completado el comando, apunte el valor de **publicIpAddress** de la máquina virtual de la región Este de EE. UU.

1. En Azure Cloud Shell, ejecute el comando para crear una máquina virtual de la región **Oeste de EE. UU.** después de realizar los siguientes cambios de parámetro en el comando:

    **--name**: escriba un nombre único para su máquina virtual de dispositivo regional de **Oeste de EE. UU.** 

    **--admin-username**: use su propio nombre de usuario administrador.

    **--admin-password**: use su propia contraseña de administrador.

    ```azurecli-interactive
    az vm create \
    --resource-group contoso-us-resource-group \
    --name ContosoSimDeviceWest \
    --location westus \
    --image Canonical:UbuntuServer:18.04-LTS:18.04.201809110 \
    --admin-username contosoadmin \
    --admin-password myContosoPassword2018 \
    --authentication-type password
    ```

    Este comando tardará varios minutos en completarse. Cuando se haya completado el comando, apunte el valor de **publicIpAddress** de la máquina virtual de la región Oeste de EE. UU.

1. Abra dos shells de línea de comandos. Conéctese a una de las máquinas virtuales regionales en cada shell mediante SSH. 

    Pase su nombre de usuario administrador y la dirección IP pública que ha apuntado para la máquina virtual como parámetros para conectarse mediante SSH. Escriba la contraseña de administrador cuando se le pida.

    ```bash
    ssh contosoadmin@1.2.3.4

    contosoadmin@ContosoSimDeviceEast:~$
    ```

    ```bash
    ssh contosoadmin@5.6.7.8

    contosoadmin@ContosoSimDeviceWest:~$
    ```



## <a name="prepare-the-azure-iot-c-sdk-development-environment"></a>Preparación del entorno de desarrollo del SDK de C para Azure IoT

En esta sección, clonará el SDK de C para Azure IoT en cada máquina virtual. El SDK contiene un ejemplo que simulará el aprovisionamiento de dispositivos de un inquilino desde cada región.

1. Para cada máquina virtual, instale **CMake**, **g++**, **gcc** y [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) mediante los siguientes comandos:

    ```bash
    sudo apt-get update
    sudo apt-get install cmake build-essential libssl-dev libcurl4-openssl-dev uuid-dev git-all
    ```

1. Busque el nombre de etiqueta de la [versión más reciente](https://github.com/Azure/azure-iot-sdk-c/releases/latest) del SDK.

1. Clone el [SDK de C para Azure IoT](https://github.com/Azure/azure-iot-sdk-c) en ambas máquinas virtuales.  Use la etiqueta que encontró en el paso anterior como valor del parámetro `-b`:

    ```bash
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Esta operación puede tardar varios minutos en completarse.

1. Para ambas máquinas virtuales, cree una carpeta **cmake** dentro del repositorio y cambie a esa carpeta.

    ```bash
    mkdir ~/azure-iot-sdk-c/cmake
    cd ~/azure-iot-sdk-c/cmake
    ```

1. Para ambas máquinas virtuales, ejecute el siguiente comando para compilar una versión del SDK específica para su plataforma de cliente de desarrollo. 

    ```bash
    cmake -Dhsm_type_symm_key:BOOL=ON -Duse_prov_client:BOOL=ON  ..
    ```

    Una vez realizada la compilación, las últimas líneas de salida tendrán un aspecto similar al siguiente:

    ```bash
    -- IoT Client SDK Version = 1.2.9
    -- Provisioning client ON
    -- Provisioning SDK Version = 1.2.9
    -- target architecture: x86_64
    -- Checking for module 'libcurl'
    --   Found libcurl, version 7.58.0
    -- Found CURL: curl
    -- target architecture: x86_64
    -- target architecture: x86_64
    -- target architecture: x86_64
    -- target architecture: x86_64
    -- iothub architecture: x86_64
    -- target architecture: x86_64
    -- Configuring done
    -- Generating done
    -- Build files have been written to: /home/contosoadmin/azure-iot-sdk-c/cmake
    ```    


## <a name="derive-unique-device-keys"></a>Derivación de las claves de dispositivo únicas

Al usar la atestación de clave simétrica con las inscripciones de grupos, no se usan las claves de grupo de inscripción directamente. En su lugar, se crea una clave derivada única para cada dispositivo tal y como se ha mencionado en la sección relativa a las [inscripciones de grupo con claves simétricas](concepts-symmetric-key-attestation.md#group-enrollments).

Para generar la clave del dispositivo, use la clave maestra del grupo para calcular un [HMAC-SHA256](https://wikipedia.org/wiki/HMAC) del identificador de registro único del dispositivo y convierta el resultado en formato Base64.

No incluya la clave maestra del grupo en el código del dispositivo.

Use el ejemplo de shell de Bash para crear una clave de dispositivo derivada para cada dispositivo con **openssl**.

- Reemplace el valor de **KEY** por el de la **clave principal** que ha apuntado anteriormente para la inscripción.

- Reemplace el valor de **REG_ID** por el de su propio identificador de registro único de cada dispositivo. Use caracteres alfanuméricos en minúsculas y guiones ("-") para definir ambos identificadores.

Ejemplo de generación de claves de dispositivo para *contoso-simdevice-east*:

```bash
KEY=rLuyBPpIJ+hOre2SFIP9Ajvdty3j0EwSP/WvTVH9eZAw5HpDuEmf13nziHy5RRXmuTy84FCLpOnhhBPASSbHYg==
REG_ID=contoso-simdevice-east

keybytes=$(echo $KEY | base64 --decode | xxd -p -u -c 1000)
echo -n $REG_ID | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64
```

```bash
p3w2DQr9WqEGBLUSlFi1jPQ7UWQL4siAGy75HFTFbf8=
```

Ejemplo de generación de claves de dispositivo para *contoso-simdevice-west*:

```bash
KEY=rLuyBPpIJ+hOre2SFIP9Ajvdty3j0EwSP/WvTVH9eZAw5HpDuEmf13nziHy5RRXmuTy84FCLpOnhhBPASSbHYg==
REG_ID=contoso-simdevice-west

keybytes=$(echo $KEY | base64 --decode | xxd -p -u -c 1000)
echo -n $REG_ID | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64
```

```bash
J5n4NY2GiBYy7Mp4lDDa5CbEe6zDU/c62rhjCuFWxnc=
```


Cada dispositivo del inquilino usará su clave de dispositivo derivada e identificador de registro único para realizar la atestación de clave simétrica con el grupo de inscripción durante el aprovisionamiento a los centros de IoT del inquilino.




## <a name="simulate-the-devices-from-each-region"></a>Simulación de los dispositivos desde cada región


En esta sección, actualizará un ejemplo de aprovisionamiento en el SDK de C para Azure IoT para ambas máquinas virtuales regionales. 

El código de ejemplo simula una secuencia de arranque de dispositivo que envía la solicitud de aprovisionamiento a la instancia de Device Provisioning Service. La secuencia de arranque hará que se reconozca y se asigne el dispositivo al centro de IoT más cercano según la latencia.

1. En Azure Portal, seleccione la pestaña **Información general** para su servicio Device Provisioning y anote el valor de **_Ámbito de id_**.

    ![Extracción de información del punto de conexión del servicio Device Provisioning desde la hoja del portal](./media/quick-create-simulated-device-x509/extract-dps-endpoints.png) 

1. Abra **~/azure-iot-sdk-c/provisioning\_client/samples/prov\_dev\_client\_sample/prov\_dev\_client\_sample.c** para editar en ambas máquinas virtuales.

    ```bash
    vi ~/azure-iot-sdk-c/provisioning_client/samples/prov_dev_client_sample/prov_dev_client_sample.c
    ```

1. Busque la constante `id_scope` y reemplace el valor por el valor de **Ámbito de id.** que copió anteriormente. 

    ```c
    static const char* id_scope = "0ne00002193";
    ```

1. Busque la definición de la función `main()` en el mismo archivo. Asegúrese de que la variable `hsm_type` esté establecida en `SECURE_DEVICE_TYPE_SYMMETRIC_KEY` tal como se muestra a continuación para que coincida con el método de atestación del grupo de inscripción. 

    Guarde los cambios de los archivos en ambas máquinas virtuales.

    ```c
    SECURE_DEVICE_TYPE hsm_type;
    //hsm_type = SECURE_DEVICE_TYPE_TPM;
    //hsm_type = SECURE_DEVICE_TYPE_X509;
    hsm_type = SECURE_DEVICE_TYPE_SYMMETRIC_KEY;
    ```

1. En ambas máquinas virtuales, encuentre la llamada a `prov_dev_set_symmetric_key_info()` en **prov\_dev\_client\_sample.c** que se marcó como comentario.

    ```c
    // Set the symmetric key if using they auth type
    //prov_dev_set_symmetric_key_info("<symm_registration_id>", "<symmetric_Key>");
    ```

    Quite la marca del comentario en las llamadas de función y reemplace los valores de marcador de posición (incluidos los corchetes angulares) por los identificadores de registro únicos y las claves de dispositivo derivada para cada dispositivo. Las claves que se muestran a continuación son solo para fines ilustrativos. Use las claves que generó anteriormente.

    Este de EE. UU:
    ```c
    // Set the symmetric key if using they auth type
    prov_dev_set_symmetric_key_info("contoso-simdevice-east", "p3w2DQr9WqEGBLUSlFi1jPQ7UWQL4siAGy75HFTFbf8=");
    ```

    Oeste de EE. UU.:
    ```c
    // Set the symmetric key if using they auth type
    prov_dev_set_symmetric_key_info("contoso-simdevice-west", "J5n4NY2GiBYy7Mp4lDDa5CbEe6zDU/c62rhjCuFWxnc=");
    ```

    Guarde los archivos.

1. En ambas máquinas virtuales, vaya a la carpeta de ejemplo que se muestra a continuación y compile el ejemplo.

    ```bash
    cd ~/azure-iot-sdk-c/cmake/provisioning_client/samples/prov_dev_client_sample/
    cmake --build . --target prov_dev_client_sample --config Debug
    ```

1. Una vez que la compilación se realice correctamente, ejecute **prov\_dev\_client\_sample.exe** en ambas máquinas virtuales para simular un dispositivo de inquilino de cada región. Tenga en cuenta que cada dispositivo se asigna al centro de IoT del inquilino más cercano a las regiones del dispositivo simulado.

    Ejecución de la simulación:
    ```bash
    ~/azure-iot-sdk-c/cmake/provisioning_client/samples/prov_dev_client_sample/prov_dev_client_sample
    ```

    Salida de ejemplo de la máquina virtual de Este de EE. UU.:

    ```bash
    contosoadmin@ContosoSimDeviceEast:~/azure-iot-sdk-c/cmake/provisioning_client/samples/prov_dev_client_sample$ ./prov_dev_client_sample
    Provisioning API Version: 1.2.9

    Registering Device

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service: contoso-east-hub.azure-devices.net, deviceId: contoso-simdevice-east
    Press enter key to exit:

    ```

    Salida de ejemplo de la máquina virtual de Oeste de EE. UU.:
    ```bash
    contosoadmin@ContosoSimDeviceWest:~/azure-iot-sdk-c/cmake/provisioning_client/samples/prov_dev_client_sample$ ./prov_dev_client_sample
    Provisioning API Version: 1.2.9

    Registering Device

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service: contoso-west-hub.azure-devices.net, deviceId: contoso-simdevice-west
    Press enter key to exit:
    ```



## <a name="clean-up-resources"></a>Limpieza de recursos

Si tiene previsto seguir trabajando con los recursos creados en este artículo, puede dejarlos. Si no tiene la intención de seguir usando el recurso, siga estos pasos para eliminar todos los recursos creados en este artículo para evitar cargos innecesarios.

En estos pasos se supone que ha creado todos los recursos de este artículo como se ha indicado en el mismo grupo de recursos denominado **contoso-us-resource-group**.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible. El grupo de recursos y todos los recursos contenidos en él se eliminan permanentemente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados. Si ha creado una instancia de IoT Hub en un grupo de recursos ya existente que contiene recursos que desea conservar, puede eliminar solo esa instancia en lugar de eliminar todo el grupo de recursos.
>

Para eliminar el grupo de recursos por nombre:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y haga clic en **Grupos de recursos**.

2. En el cuadro de texto **Filtrar por nombre...**, escriba el nombre del grupo de recursos que contiene los recursos: **contoso-us-resource-group**. 

3. A la derecha del grupo de recursos de la lista de resultados, haga clic en **...** y, a continuación, en **Eliminar grupo de recursos**.

4. Se le pedirá que confirme la eliminación del grupo de recursos. Escriba otra vez el nombre del grupo de recursos para confirmar y haga clic en **Eliminar**. Transcurridos unos instantes, el grupo de recursos y todos los recursos que contiene se eliminan.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre el reaprovisionamiento, consulte

> [!div class="nextstepaction"]
> [Conceptos sobre el reaprovisionamiento de dispositivos de IoT Hub](concepts-device-reprovision.md)

* Para más información sobre el desaprovisionamiento, consulte
> [!div class="nextstepaction"]
> [Desaprovisionamiento de dispositivos aprovisionados automáticamente](how-to-unprovision-devices.md)