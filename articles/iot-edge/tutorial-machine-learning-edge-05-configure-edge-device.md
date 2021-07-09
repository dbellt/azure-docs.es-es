---
title: 'Tutorial: Configuración de un dispositivo Azure IoT Edge: Aprendizaje automático en IoT Edge'
description: En este tutorial configurará una máquina virtual de Azure que ejecuta Linux como un dispositivo Azure IoT Edge que actúa como puerta de enlace transparente.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 2/5/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: amqp
ms.openlocfilehash: e5e1556b0b4960850c955f3d52c34396d1363b2a
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110535760"
---
# <a name="tutorial-configure-an-azure-iot-edge-device"></a>Tutorial: Configuración de un dispositivo IoT Edge

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

En este artículo, se configurará una máquina virtual de Azure que ejecuta Linux para que sea un dispositivo Azure IoT Edge que actúa como puerta de enlace transparente. La configuración de una puerta de enlace transparente permite que los dispositivos se conecten a Azure IoT Hub a través de la puerta de enlace sin necesidad de saber que dicha puerta existe. Al mismo tiempo, un usuario que interactúa con los dispositivos de IoT Hub no es consciente del dispositivo de puerta de enlace intermedio. En última instancia, se agregará análisis perimetral al sistema mediante la adición de módulos de IoT Edge a la puerta de enlace transparente.

>[!NOTE]
>Los conceptos de este tutorial se aplican a todas las versiones de IoT Edge, pero el dispositivo de ejemplo que se crea para probar el escenario utiliza la versión 1.1 de IoT Edge.

Los pasos descritos en este artículo los realiza normalmente un desarrollador en la nube.

En esta sección del tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear certificados para permitir que el dispositivo de puerta de enlace se conecte de forma segura a los dispositivos de nivel inferior.
> * Crear un dispositivo IoT Edge.
> * Crear una máquina virtual de Azure para simular el dispositivo IoT Edge.

## <a name="prerequisites"></a>Requisitos previos

Este artículo forma parte de un tutorial sobre el uso de Azure Machine Learning en IoT Edge. Cada artículo de la serie se basa en el trabajo del artículo anterior. Si ha llegado a este artículo directamente, consulte el [primer artículo](tutorial-machine-learning-edge-01-intro.md) de la serie.

## <a name="create-certificates"></a>Creación de certificados

Para que un dispositivo funcione como puerta de enlace, debe poder conectarse de forma segura a dispositivos de bajada. Con IoT Edge puede usar una infraestructura de clave pública (PKI) para configurar conexiones seguras entre dispositivos. En este caso, vamos a permitir que un dispositivo IoT de bajada se conecte a un dispositivo IoT Edge que actúa como puerta de enlace transparente. Para mantener una seguridad razonable, el dispositivo de bajada debe confirmar la identidad del dispositivo IoT Edge. Para obtener más información acerca de cómo los dispositivos IoT Edge usan certificados, consulte [Información de uso de los certificados de Azure IoT Edge](iot-edge-certs.md).

En esta sección, se crean los certificados autofirmados mediante una imagen de Docker que luego se compila y ejecuta. Se ha decidido usar una imagen de Docker para completar este paso, ya que reduce el número de pasos necesarios para crear los certificados en la máquina de desarrollo de Windows. Para saber qué se automatiza con la imagen de Docker, consulte [Creación de certificados de demostración para probar las características del dispositivo IoT Edge](how-to-create-test-certificates.md).

1. Inicie sesión en la máquina virtual de desarrollo.
1. Cree una carpeta con la ruta de acceso y el nombre **c:\edgeCertificates**.

1. Inicie **Docker para Windows** desde el menú Inicio de Windows, en caso de que no se esté ejecutando ya.

1. Abra Visual Studio Code.

1. Seleccione **Archivo** > **Abrir carpeta** y, después, seleccione **C:\\source\\IoTEdgeAndMlSample\\CreateCertificates**.

1. En el panel **Explorador**, haga clic con el botón derecho en el archivo **dockerfile** y seleccione **Compilar imagen**.

1. En el cuadro de diálogo, acepte el valor predeterminado del nombre de la imagen y la etiqueta: **createcertificates: latest**.

    ![Captura de pantalla en la que se muestra la creación de certificados en Visual Studio Code.](media/tutorial-machine-learning-edge-05-configure-edge-device/create-certificates.png)

1. Espere a que la compilación finalice.

    > [!NOTE]
    > Es posible que vea la advertencia de que falta una clave pública. Puede hacer caso omiso de esta advertencia sin ningún problema. Asimismo, verá una advertencia de seguridad que le recomienda comprobar o restablecer los permisos en la imagen, puede hacer caso omiso para esta imagen.

1. En la ventana de terminal de Visual Studio Code, ejecute el contenedor createcertificates.

    ```cmd
    docker run --name createcertificates --rm -v c:\edgeCertificates:/edgeCertificates createcertificates /edgeCertificates
    ```

1. Docker le solicitará el acceso a la unidad **c:\\** . Seleccione **Share it** (Compartir).

1. Escriba sus credenciales cuando se le solicite.

1. Una vez que finaliza la ejecución del contenedor, compruebe los siguientes archivos en **c:\\edgeCertificates**:

    * c:\\edgeCertificates\\certs\\azure-iot-test-only.root.ca.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device-full-chain.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device.cert.pfx
    * c:\\edgeCertificates\\private\\new-edge-device.key.pem

## <a name="upload-certificates-to-azure-key-vault"></a>Carga de certificados a Azure Key Vault

Para almacenar nuestros certificados de forma segura y para que se pueda acceder a ellos desde varios dispositivos, se cargarán en Azure Key Vault. Como puede ver en la lista anterior, tenemos dos tipos de archivos de certificado: PFX y PEM. El archivo PFX se tratará como certificados de Key Vault que se van a cargar en Key Vault. Los archivos PEM son texto sin formato y se tratarán como secretos de Key Vault. Se usará la instancia de Key Vault asociada al área de trabajo de Azure Machine Learning que se creó mediante la ejecución de cuadernos de [Jupyter Notebook](tutorial-machine-learning-edge-04-train-model.md#run-the-jupyter-notebooks).

1. En [Azure Portal](https://portal.azure.com), vaya al área de trabajo de Azure Machine Learning.

1. En la página de información general del área de trabajo de Machine Learning, busque el nombre de la instancia de **Key Vault**.

    ![Captura de pantalla que muestra cómo se copia el nombre del almacén de claves.](media/tutorial-machine-learning-edge-05-configure-edge-device/find-key-vault-name.png)

1. En el equipo de desarrollo, cargue los certificados a Key Vault. Reemplace **\<subscriptionId\>** y **\<keyvaultname\>** por la información del recurso.

    ```powershell
    c:\source\IoTEdgeAndMlSample\CreateCertificates\upload-keyvaultcerts.ps1 -SubscriptionId <subscriptionId> -KeyVaultName <keyvaultname>
    ```

1. Cuando se le solicite, inicie sesión en Azure.

1. El script se ejecutará durante unos minutos y, como salida, proporcionará una lista de las nuevas entradas de Key Vault.

    ![Captura de pantalla que muestra la salida del script de Key Vault.](media/tutorial-machine-learning-edge-05-configure-edge-device/key-vault-entries-output.png)

## <a name="register-an-iot-edge-device"></a>Registro de un dispositivo de IoT Edge

Para conectar un dispositivo Azure IoT Edge a un centro de IoT, primero es preciso registrar un dispositivo en el centro. Tomamos la cadena de conexión de la identidad del dispositivo en la nube y la usamos para configurar el tiempo de ejecución en nuestro dispositivo IoT Edge. Después de que un dispositivo configurado se conecte al centro de conectividad, se pueden implementar módulos y enviar mensajes. También se puede cambiar la configuración del dispositivo IoT Edge físico modificando la identidad del dispositivo correspondiente en IoT Hub.

Para este tutorial, se ha registrado la nueva identidad del dispositivo mediante Visual Studio Code. Estos pasos también se pueden completar mediante Azure Portal o la CLI de Azure. Independientemente del método que elija, asegúrese de obtener la cadena de conexión del dispositivo IoT Edge. La cadena de conexión del dispositivo se puede encontrar en la página de detalles del dispositivo en Azure Portal.

1. Abra Visual Studio Code en el equipo de desarrollo.

1. Expanda el marco **Azure IoT Hub** desde la vista **Explorador** de Visual Studio Code.

1. Seleccione los puntos suspensivos y seleccione **Create IoT Edge Device** (Crear dispositivo IoT Edge).

1. Asigne un nombre al dispositivo. Por comodidad, aquí se usa el nombre **aaTurbofanEdgeDevice**, con el fin de que al ordenar los nombres aparezca el primero de la lista de dispositivos.

1. El nuevo dispositivo aparece en la lista de dispositivos.

    ![Captura de pantalla que muestra una vista del dispositivo en el Explorador de Visual Studio Code.](media/tutorial-machine-learning-edge-05-configure-edge-device/iot-hub-devices-list.png)

## <a name="deploy-an-azure-virtual-machine"></a>Implementación de una máquina virtual de Azure

Aquí se usa una máquina virtual con Ubuntu 18.04 LTS con el runtime de Azure IoT Edge instalado y configurado. La implementación usa una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/overview.md) que se mantiene en el repositorio de proyectos [iotedge-vm-deploy](https://github.com/Azure/iotedge-vm-deploy). Aprovisiona el dispositivo IoT Edge que registró en el paso anterior mediante la cadena de conexión que se especifica en la plantilla.

La máquina virtual se puede implementar mediante Azure Portal o la CLI de Azure. Se mostrarán los pasos de Azure Portal. Para más información, consulte [Ejecución de Azure IoT Edge en máquinas virtuales Ubuntu](how-to-install-iot-edge-ubuntuvm.md).

### <a name="deploy-using-deploy-to-azure-button"></a>Implementación con el botón Implementar en Azure

1. Para usar la plantilla de ARM `iotedge-vm-deploy` para implementar una máquina virtual con Ubuntu 18.04 LTS, haga clic en el botón siguiente:

    [![Botón Implementar en Azure de iotedge-vm-deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fazure%2Fiotedge-vm-deploy%2Fmaster%2FedgeDeploy.json)

1. En la ventana que se acaba de abrir, rellene los campos de formulario disponibles.

   | Campo | Descripción |
   | - | - |
   | **Suscripción** | suscripción de Azure activa en la que se va a implementar la máquina virtual. |
   | **Grupos de recursos** | grupo de recursos existente o recién creado que va a contener la máquina virtual y sus recursos asociados. |
   | **Prefijo de etiqueta DNS** | valor requerido (de su elección) que se usa para colocar el nombre de host de la máquina virtual a modo de prefijo. |
   | **Nombre de usuario administrador** | nombre de usuario al que se proporcionarán privilegios raíz en la implementación. |
   | **Cadena de conexión del dispositivo** | [cadena de conexión de un dispositivo](./how-to-register-device.md) que se ha creado en la instancia de [IoT Hub](../iot-hub/about-iot-hub.md) de su elección. |
   | **Tamaño de VM** | [tamaño](../cloud-services/cloud-services-sizes-specs.md) de la máquina virtual que se va a implementar.
   | **Versión del sistema operativo Ubuntu** | versión del sistema operativo Ubuntu que se va a instalar en la máquina virtual base. |
   | **Ubicación** | [región geográfica](https://azure.microsoft.com/global-infrastructure/locations/) donde se va a implementar la máquina virtual. Este valor se establece de forma predeterminada en la ubicación del grupo de recursos seleccionado. |
   | **Tipo de autenticación** | elija **sshPublicKey** o **password**, según cuál sea su preferencia. |
   | **Contraseña o clave de administrador** | valor de la clave pública SSH o de la contraseña en función del tipo de autenticación que se haya escogido. |

1. Cuando se hayan rellenado todos los campos, active la casilla situada en la parte inferior de la página para aceptar los términos y seleccione **Revisar y crear**, y **Crear** para iniciar la implementación.

1. En Azure Portal, vaya a la máquina virtual. Puede encontrarla a través del grupo de recursos o seleccionando **Máquinas virtuales** en **Servicios de Azure** en la página de aterrizaje del portal.

1. Anote el **nombre DNS** de la máquina virtual. Lo necesitará para iniciar sesión en la máquina virtual.

## <a name="connect-to-your-iot-edge-device"></a>Conexión al dispositivo IoT Edge

1. Abra un símbolo del sistema y use el siguiente comando para iniciar sesión en la máquina virtual. Escriba su propia información de nombre de usuario y nombre DNS según los datos de la sección anterior.

    ```bash
    ssh <adminUsername>@<DNS_name>
    ```

1. Cuando se le solicite validar la autenticidad del host, escriba **Sí** y seleccione **Entrar**.

1. Cuando se le solicite, indique la contraseña.

1. Ubuntu muestra un mensaje de bienvenida y, después, debería aparecer una solicitud como `<username>@<machinename>:~$`.

## <a name="download-key-vault-certificates"></a>Descarga de los certificados de Key Vault

Anteriormente en este artículo, cargamos certificados en Key Vault para que estén disponibles para el dispositivo IoT Edge y para nuestro dispositivo hoja. El dispositivo hoja es un dispositivo de nivel inferior que usa el dispositivo IoT Edge como puerta de enlace para comunicarse con IoT Hub.

El dispositivo hoja se tratará más adelante en el tutorial. En esta sección, descargue los certificados en el dispositivo IoT Edge.

1. En la sesión SSH en la máquina virtual con Linux, inicie sesión en Azure con la CLI de Azure.

    ```azurecli
    az login
    ```

1. Se le pedirá que abra un explorador en una página [de inicio de sesión de dispositivo de Microsoft](https://microsoft.com/devicelogin) y que escriba un código único. Puede realizar estos pasos en el equipo local. Cierre la ventana del explorador cuando haya terminado la autenticación.

1. Cuando se autentique correctamente, inicie sesión en la VM de Linux y enumere las suscripciones de Azure.

1. Defina la suscripción de Azure que quiere usar con los comandos de la CLI de Azure.

    ```azurecli
    az account set --subscription <subscriptionId>
    ```

1. Cree un directorio en la VM para los certificados.

    ```bash
    sudo mkdir /edgeMlCertificates
    ```

1. Descargue los certificados que guardó en el almacén de claves: new-edge-device-full-chain.cert.pem, new-edge-device.key.pem y azure-iot-test-only.root.ca.cert.pem.

    ```azurecli
    key_vault_name="<key vault name>"
    sudo az keyvault secret download --vault-name $key_vault_name --name new-edge-device-full-chain-cert-pem -f /edgeMlCertificates/new-edge-device-full-chain.cert.pem
    sudo az keyvault secret download --vault-name $key_vault_name --name new-edge-device-key-pem -f /edgeMlCertificates/new-edge-device.key.pem
    sudo az keyvault secret download --vault-name $key_vault_name --name azure-iot-test-only-root-ca-cert-pem -f /edgeMlCertificates/azure-iot-test-only.root.ca.cert.pem
    ```

## <a name="update-the-iot-edge-device-configuration"></a>Actualización de la configuración del dispositivo IoT Edge

El entorno de ejecución de IoT Edge usa el archivo /etc/iotedge/config.yaml para conservar su configuración. Es necesario actualizar la información siguiente de este archivo:

* **Certificados**: los certificados que se usarán para las conexiones realizadas con los dispositivos de bajada
* **Nombre de host**: el nombre de dominio completo del dispositivo IoT Edge de la máquina virtual.

Actualice los certificados y el nombre de host mediante la edición directa del archivo config.yaml.

1. Abra el archivo config.yaml.

    ```bash
    sudo nano /etc/iotedge/config.yaml
    ```

1. Actualice la sección de certificados del archivo config.yaml, para lo que se quita el símbolo **#** que aparece al principio y se configura la ruta para que el archivo tenga un aspecto similar al siguiente ejemplo:

    ```yaml
    certificates:
      device_ca_cert: "/edgeMlCertificates/new-edge-device-full-chain.cert.pem"
      device_ca_pk: "/edgeMlCertificates/new-edge-device.key.pem"
      trusted_ca_certs: "/edgeMlCertificates/azure-iot-test-only.root.ca.cert.pem"
    ```

    Asegúrese de que la línea **certificates:** no tiene ningún espacio en blanco al principio y que a todos los certificados anidados se le aplica una sangría de dos espacios.

    Al hacer clic con el botón derecho en nano, se pegará el contenido del portapapeles en la posición actual del cursor. Para reemplazar la cadena, use las flechas del teclado para ir a la cadena que desea reemplazar, elimine la cadena y haga clic con el botón derecho para pegar la nueva cadena desde el búfer.

1. En Azure Portal, vaya a la máquina virtual. Copie el nombre DNS (FQDN de la máquina) de la sección **Introducción**.

1. Pegue el nombre de dominio completo en la sección de nombre del host del archivo config.yml. Asegúrese de que el nombre está escrito en minúsculas.

    ```yaml
    hostname: '<machinename>.<region>.cloudapp.azure.com'
    ```

1. Guarde y cierre el archivo, para lo que debe seleccionar **Ctrl + X**, **Y,** y **Entrar**.

1. Reinicie el demonio de IoT Edge.

    ```bash
    sudo systemctl restart iotedge
    ```

1. Compruebe el estado del demonio de IoT Edge. Después del comando, escriba **: q** para salir.

    ```bash
    systemctl status iotedge
    ```

## <a name="troubleshooting"></a>Solución de problemas

Si ve errores (texto en color con el prefijo "\[ERROR\]") en el estado, examine los registros del demonio para obtener información detallada del error.

   ```bash
   journalctl -u iotedge --no-pager --no-full
   ```

Para más información sobre cómo solucionar errores, consulte la página de [solución de problemas](troubleshoot.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Este tutorial forma parte de una serie en la que cada artículo complementa el trabajo realizado en los anteriores. No borre ningún recurso hasta que complete el tutorial final.

## <a name="next-steps"></a>Pasos siguientes

Se acaba de completar la configuración de una máquina virtual de Azure como puerta de enlace transparente de IoT Edge. Comenzamos con la generación de certificados de prueba, que se cargaron en Key Vault. A continuación, usamos un script y una plantilla de Resource Manager para implementar la máquina virtual con la imagen Ubuntu Server 16.04 LTS + runtime de Azure IoT Edge" desde Azure Marketplace. Con la máquina virtual en funcionamiento, nos conectamos mediante SSH. Luego, iniciamos sesión en Azure y descargamos los certificados de Key Vault. Realizamos varias actualizaciones en la configuración del entorno de ejecución de Azure IoT Edge, para lo que actualizamos el archivo config.yaml.

Consulte el artículo siguiente para compilar módulos de IoT Edge.

> [!div class="nextstepaction"]
> [Creación e implementación de módulos personalizados de IoT Edge](tutorial-machine-learning-edge-06-custom-modules.md)
