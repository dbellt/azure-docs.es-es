---
title: Conexión de Azure Percept DK a través de SSH
description: Aprenda a configurar una conexión SSH en Azure Percept DK con PuTTY
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 03/18/2021
ms.custom: template-how-to
ms.openlocfilehash: 4ee194e6d501d967d37453763c9cd4b25d1bedbe
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108754206"
---
# <a name="connect-to-your-azure-percept-dk-over-ssh"></a>Conexión de Azure Percept DK a través de SSH

Siga los pasos que se indican a continuación para configurar una conexión SSH a Azure Percept DK mediante OpenSSH o [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

## <a name="prerequisites"></a>Requisitos previos

- Un equipo host basado en Windows, Linux u OS X con funcionalidad Wi-Fi
- Un cliente SSH (consulte la siguiente sección para obtener la guía de instalación)
- Un Azure Percept DK (kit de desarrollo)
- Una cuenta de SSH, creada durante la [experiencia de instalación de Azure Percept DK](./quickstart-percept-dk-set-up.md)

## <a name="install-your-preferred-ssh-client"></a>Instalación del cliente SSH preferido

Si el equipo host ejecuta Linux u OS X, los servicios SSH están incluidos en esos sistemas operativos y se pueden ejecutar sin una aplicación cliente independiente. Consulte la documentación del producto del sistema operativo para más información sobre cómo ejecutar servicios SSH.

Si el equipo host ejecuta Windows, es posible que tenga dos opciones de cliente de SSH entre las que elegir: OpenSSH y PuTTY.

### <a name="openssh"></a>OpenSSH

Windows 10 incluye un cliente SSH integrado llamado OpenSSH que se puede ejecutar con un comando simple en un símbolo del sistema. Si está disponible, se recomienda usar OpenSSH con Azure Percept. Para comprobar si el equipo Windows tiene instalado OpenSSH, siga estos pasos:

1. Vaya a **Configuración** -> **de inicio**.

1. Seleccione **Aplicaciones**.

1. En **Aplicaciones y Características**, seleccione **Características opcionales**.

1. Escriba **Cliente OpenSSH** en la barra de búsqueda de **Características instaladas**. Si no aparece OpenSSH, el cliente ya está instalado y puede pasar a la sección siguiente. Si no ve OpenSSH, seleccione **Agregar una característica**.

    :::image type="content" source="./media/how-to-ssh-into-percept-dk/open-ssh-install.png" alt-text="Captura de pantalla de la configuración que muestra el estado de instalación OpenSSH.":::

1. Seleccione **Cliente OpenSSH** y, luego, **Instalar**. Ahora puede pasar a la sección siguiente. Si OpenSSH no está disponible para instalarse en el equipo, siga los pasos que se indican a continuación para instalar PuTTY, un cliente SSH de terceros.

### <a name="putty"></a>PuTTY

Si el equipo Windows no incluye OpenSSH, se recomienda usar [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). Para descargar e instalar PuTTY, realice los pasos siguientes:

1. Vaya a la [Página de descarga de PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

1. En **Archivos de paquete**, seleccione el archivo .msi de 32 bits o 64 bits para descargar el instalador. Si no está seguro de qué versión elegir, consulte las [Preguntas más frecuentes](https://www.chiark.greenend.org.uk/~sgtatham/putty/faq.html#faq-32bit-64bit).

1. Seleccione el instalador para iniciar el proceso de instalación. Siga las indicaciones según sea necesario.

1. Felicidades. Ha instalado correctamente el cliente SSH de PuTTY.

## <a name="initiate-the-ssh-connection"></a>Iniciar la conexión SSH

1. Encendido de Azure Percept DK.

1. Si el kit de desarrollo ya está conectado a una red a través de Ethernet o Wi-Fi, vaya al paso siguiente. De lo contrario, conecte el equipo host directamente al punto de acceso Wi-Fi del kit de desarrollo. Como la conexión a cualquier otra red Wi-Fi, abra la configuración de red e Internet en el equipo, seleccione la red que se indica y escriba la contraseña de red cuando se le solicite:

    - **Nombre de red**: en función de la versión del sistema operativo del kit de desarrollo, el nombre del punto de acceso Wi-Fi es **scz-xxxx** o **apd-xxxx** (donde "xxxx" son los cuatro últimos dígitos de la dirección MAC del kit de desarrollo)
    - **Contraseña**: puede encontrarla en la Tarjeta de bienvenida que se incluye en el kit de desarrollo

    > [!WARNING]
    > Mientras está conectado al punto de acceso Wi-Fi de Azure Percept DK, el equipo host perderá temporalmente su conexión a Internet. Se interrumpirán las llamadas de videoconferencia activas, la transmisión web u otras experiencias basadas en la red.

1. Complete el proceso de conexión SSH según el cliente SSH.

### <a name="using-openssh"></a>Uso de OpenSSH

1. Abra un símbolo del sistema (**Inicie** -> **el símbolo del sistema**).

1. Escriba lo siguiente en el símbolo del sistema:

    ```console
    ssh [your ssh user name]@[IP address]
    ```

    Si su equipo está conectado al punto de acceso Wi-Fi del kit de desarrollo, la dirección IP será 10.1.1.1. Si el kit de desarrollo está conectado a través de Ethernet, utilice la dirección IP local del dispositivo, que puede obtener en el centro o el enrutador Ethernet. Si el kit de desarrollo está conectado a través de Wi-Fi, debe utilizar la dirección IP que se ha asignado al kit de desarrollo durante la [experiencia de configuración de Azure Percept DK](./quickstart-percept-dk-set-up.md).

    > [!TIP]
    > Si el kit de desarrollo está conectado a una red Wi-Fi pero no conoce su dirección IP, vaya a Azure Percept Studio y [abra la secuencia de vídeo del dispositivo](./how-to-view-video-stream.md). La barra de direcciones de la pestaña explorador de secuencia de vídeo mostrará la dirección IP del dispositivo.

1. Escriba la contraseña SSH cuando se le solicite.

    :::image type="content" source="./media/how-to-ssh-into-percept-dk/open-ssh-prompt.png" alt-text="Captura de pantalla del inicio de sesión del símbolo del sistema SSH abierto.":::

1. Si esta es la primera vez que se conecta al kit de desarrollo a través de OpenSSH, puede que también se le pida que acepte la clave del host. Escriba **sí** para aceptar la clave.

1. Felicidades. Se ha conectado correctamente al kit de desarrollo a través de SSH.

### <a name="using-putty"></a>Usar PuTTY

1. Abra PuTTY. Escriba lo siguiente en la ventana de **Configuración de PuTTY** y seleccione **Abrir** para SSH en el kit de desarrollo:

    1. Nombre de host: [dirección IP]
    1. Puerto: 22
    1. Tipo de conexión: SSH

    El **Nombre de host** es la dirección IP del kit de desarrollo. Si su equipo está conectado al punto de acceso Wi-Fi del kit de desarrollo, la dirección IP será 10.1.1.1. Si el kit de desarrollo está conectado a través de Ethernet, utilice la dirección IP local del dispositivo, que puede obtener en el centro o el enrutador Ethernet. Si el kit de desarrollo está conectado a través de Wi-Fi, debe utilizar la dirección IP que se ha asignado al kit de desarrollo durante la [experiencia de configuración de Azure Percept DK](./quickstart-percept-dk-set-up.md).

    > [!TIP]
    > Si el kit de desarrollo está conectado a una red Wi-Fi pero no conoce su dirección IP, vaya a Azure Percept Studio y [abra la secuencia de vídeo del dispositivo](./how-to-view-video-stream.md). La barra de direcciones de la pestaña explorador de secuencia de vídeo mostrará la dirección IP del dispositivo.

    :::image type="content" source="./media/how-to-ssh-into-percept-dk/ssh-putty.png" alt-text="Captura de pantalla de la ventana de configuración PuTTY.":::

1. Se abrirá un terminal PuTTY. Cuando se le solicite, escriba su nombre de usuario y contraseña de SSH en el terminal.

1. Felicidades. Se ha conectado correctamente al kit de desarrollo a través de SSH.

## <a name="next-steps"></a>Pasos siguientes

Después de conectarse correctamente a Azure Percept DK mediante SSH, puede realizar una serie de tareas, entre las que se incluyen la [solución de problemas](./troubleshoot-dev-kit.md) y las [actualizaciones USB](./how-to-update-via-usb.md).