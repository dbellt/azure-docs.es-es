---
title: Configurar un laboratorio para enseñar ciencia de datos con Python y Jupyter Notebooks | Microsoft Docs
description: Aprenda a configurar un laboratorio para enseñar ciencia de datos mediante Python y Jupyter Notebooks
author: emaher
ms.topic: article
ms.date: 09/29/2020
ms.author: enewman
ms.openlocfilehash: 8f84edf29e6c4e3bd111deb5ea4bd479ea2d6140
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109716283"
---
# <a name="set-up-a-lab-to-teach-data-science-with-python-and-jupyter-notebooks"></a>Configurar un laboratorio para enseñar ciencia de datos con Python y Jupyter Notebooks
En este artículo se describe cómo configurar una máquina virtual (VM) de plantilla en servicios de laboratorio con las herramientas necesarias para enseñar a los alumnos a usar [Cuadernos de Jupyter Notebook](http://jupyter-notebook.readthedocs.io/) y cómo los alumnos pueden conectarse a sus cuadernos en sus máquinas virtuales (VM).

Cuadernos de Jupyter Notebook es un proyecto de código abierto que permite combinar fácilmente texto enriquecido y código fuente Python ejecutable en un solo lienzo denominado cuaderno. Al ejecutar un cuaderno, se genera un registro lineal de entradas y salidas. Dichas salidas pueden incluir texto, tablas de información, gráficos de dispersión y mucho más.

## <a name="set-up-the-lab"></a>Configuración del laboratorio

### <a name="lab-configuration"></a>Configuración del laboratorio
Para configurar este laboratorio, necesita acceso a una suscripción a Azure y una cuenta de laboratorio. Hable con el administrador de su organización para ver si puede obtener acceso a una suscripción de Azure existente. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

Una vez que tenga una suscripción de Azure, cree una nueva cuenta de laboratorio en Azure Lab Services siguiendo las instrucciones del tutorial: [Configuración de una cuenta de laboratorio](tutorial-setup-lab-account.md). También puede usar una cuenta de laboratorio existente.

### <a name="lab-account-settings"></a>Configuración de la cuenta de laboratorio
Habilite la configuración que se describe en la tabla siguiente para la cuenta de laboratorio. Para obtener más información sobre cómo habilitar imágenes de Marketplace, consulte [Especificación de las imágenes de Marketplace disponibles para los creadores de laboratorios](specify-marketplace-images.md).

| Configuración de la cuenta de laboratorio | Instructions |
| ------------------- | ------------ |
| Imagen de Marketplace | Dentro de su cuenta de laboratorio, habilite una de las imágenes de Azure Marketplace en función de las necesidades de su sistema operativo: <br/><ul><li>Data Science Virtual Machine: Windows Server 2019</li><li>Data Science Virtual Machine: Ubuntu 18.04</li></ul> |

> [!NOTE]
> En este artículo se usan las imágenes de Data Science VM disponibles en Azure Marketplace porque están preconfiguradas con Jupyter Notebook. Sin embargo, estas imágenes también incluyen muchas otras herramientas de desarrollo y modelado para la ciencia de datos. Si no quiere esas herramientas adicionales y quiere una configuración ligera con cuadernos de Jupyter Notebook, cree una imagen de máquina virtual personalizada. Para obtener un ejemplo, vea [Instalación de JupyperHub en Azure](http://tljh.jupyter.org/en/latest/install/azure.html). Una vez creada la imagen personalizada, puede cargarla en una galería de imágenes compartida para usar la imagen dentro de Azure Lab Services. Obtenga más información sobre el [uso de Shared Image Gallery en Azure Lab Services](how-to-attach-detach-shared-image-gallery.md). 

### <a name="lab-settings"></a>Configuración del laboratorio
Configure los valores de **Tamaño de máquina virtual** e **Imagen de máquina virtual**, tal como se muestra en la tabla siguiente al configurar un laboratorio de clase. Para obtener instrucciones sobre la creación de un laboratorio de clase, vea [Configuración de un laboratorio de clase](tutorial-setup-classroom-lab.md).

| Configuración del laboratorio | Valor/instrucciones |
| ------------ | ------------------ | 
| Tamaño de la máquina virtual | <p>El tamaño que elija aquí dependerá de la carga de trabajo que quiera ejecutar:</p><ul><li>Pequeño o medio: adecuado para una configuración básica del acceso a cuadernos de Jupyter Notebook</li><li>GPU pequeña (proceso): este tamaño es más adecuado para aplicaciones de proceso intensivo y uso intensivo de la red como, por ejemplo, la inteligencia artificial y el aprendizaje profundo.</li></ul> | 
| Imagen de máquina virtual | <p>Elija una de las imágenes siguientes en función de las necesidades del sistema operativo:</p><ul><li>[Data Science Virtual Machine: Windows Server 2019](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.dsvm-win-2019)</li><li>[Data Science Virtual Machine: Ubuntu 18.04](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804?tab=Overview)</li></ul> |

Al crear un laboratorio con el tamaño **GPU pequeña (proceso)** , tiene la opción de [instalar controladores de GPU](./how-to-setup-lab-gpu.md#ensure-that-the-appropriate-gpu-drivers-are-installed).  Esta opción instala los controladores de NVIDIA recientes y el kit de herramientas de arquitectura unificada de dispositivos de cómputo (CUDA), que son necesarios para habilitar la computación de alto rendimiento con la GPU.  Para obtener más información, consulte el artículo [Configurar un laboratorio con máquinas virtuales de GPU](./how-to-setup-lab-gpu.md).

### <a name="template-virtual-machine"></a>Máquina virtual de plantilla
Una vez creado un laboratorio, se creará una máquina virtual de plantilla en función del tamaño de la máquina virtual y la imagen que elija. Configure la máquina virtual de plantilla con todo lo que quiere proporcionar a los alumnos para esta clase. Para obtener más información, vea [Cómo administrar la máquina virtual de plantilla](how-to-create-manage-template.md). 

De forma predeterminada, las imágenes de Data Science VM incluyen muchos de los marcos de ciencia de datos y herramientas necesarios para este tipo de clase. Por ejemplo, las imágenes incluyen lo siguiente:

- [Jupyter Notebooks](http://jupyter-notebook.readthedocs.io/): aplicación web que permite a los científicos de datos tomar datos sin procesar, ejecutar cálculos y ver los resultados en el mismo entorno. Se ejecutará localmente en la máquina virtual de plantilla.  
- [Visual Studio Code](https://code.visualstudio.com/): entorno de desarrollo integrado (IDE) que proporciona una experiencia interactiva enriquecida al escribir y probar un cuaderno. Para obtener más información, consulte [Trabajar con cuadernos de Jupyter Notebook en Visual Studio Code](https://code.visualstudio.com/docs/python/jupyter-support).

Si usa el tamaño **GPU pequeña (proceso)** , se recomienda comprobar que las bibliotecas y los marcos de ciencia de datos estén configurados correctamente con la GPU.  Para configurar correctamente los marcos y las bibliotecas, es posible que deba instalar una versión diferente de los controladores de NVIDIA y el kit de herramientas de CUDA.  Por ejemplo, para validar que la GPU está configurada para TensorFlow, puede conectarse a la máquina virtual de la plantilla y ejecutar el siguiente código Python-TensorFlow en Jupyter Notebooks:

```python
import tensorflow as tf
from tensorflow.python.client import device_lib

print(device_lib.list_local_devices())
```

Si la salida del código anterior es similar a la siguiente, significa que la GPU no está configurada para TensorFlow:

```python
[name: "/device:CPU:0"
device_type: "CPU"
memory_limit: 268435456
locality {
}
incarnation: 15833696144144374634
]
```
Para configurar correctamente la GPU, debe consultar la documentación del marco o la biblioteca.  Siguiendo con el ejemplo anterior, TensorFlow proporciona las siguientes instrucciones:
- [Compatibilidad con GPU de TensorFlow](https://www.tensorflow.org/install/gpu)

Estas instrucciones cubren la versión necesaria de los [controladores de NVIDIA](https://www.nvidia.com/drivers) y el [kit de herramientas de CUDA](https://developer.nvidia.com/cuda-toolkit-archive).  Las instrucciones también incluyen la instalación de la [biblioteca de redes neuronales profundas de CUDA (cudDNN) de NVIDIA](https://developer.nvidia.com/cudnn).

Después de seguir los pasos de TensorFlow para configurar la GPU, al volver a ejecutar el código anterior, debería ver una salida similar a la siguiente:

```python
[name: "/device:CPU:0"
device_type: "CPU"
memory_limit: 268435456
locality {
}
incarnation: 15833696144144374634
, name: "/device:GPU:0"
device_type: "GPU"
memory_limit: 11154792128
locality {
  bus_id: 1
  links {
  }
}
incarnation: 2659412736190423786
physical_device_desc: "device: 0, name: NVIDIA Tesla K80, pci bus id: 0001:00:00.0, compute capability: 3.7"
]
```

### <a name="provide-notebooks-for-the-class"></a>Suministro de cuadernos para la clase
La tarea siguiente consiste en proporcionar a los alumnos los cuadernos que quiera que usen. Para proporcionar sus propios cuadernos, los puede guardar localmente en la máquina virtual de plantilla. 

Si quiere usar cuadernos de ejemplo de Azure Machine Learning, vea [Procedimientos para configurar un entorno con cuadernos de Jupyter Notebook](../machine-learning/how-to-configure-environment.md#jupyter). 

### <a name="optional-enable-graphical-desktop-for-linux"></a>Opcional: habilitación del escritorio gráfico para Linux 
La imagen de **Data Science Virtual Machine: Ubuntu** ya está aprovisionada con el servidor de X2GO y está lista para aceptar conexiones de cliente. No es necesario realizar más pasos al configurar la máquina virtual de plantilla. 

### <a name="publish-the-template-machine"></a>Publicación de la máquina de plantilla
Al publicar la plantilla, cada alumno registrado en el laboratorio obtendrá una copia de la máquina virtual de plantilla con todas las herramientas y cuadernos locales que se hayan configurado en él.

## <a name="how-students-connect-to-jupyter-notebooks"></a>¿Cómo se conectan los alumnos a los cuadernos de Jupyter Notebook?
Una vez que se publique la plantilla, cada alumno tendrá acceso a una máquina virtual que incluye todo lo que se ha configurado previamente para la clase, incluidos los cuadernos de Jupyter Notebook. En las secciones siguientes se muestran diferentes maneras que los alumnos tienen de conectarse a los cuadernos de Jupyter Notebook. 

### <a name="for-windows-vms"></a>Para las máquinas virtuales Windows
Si se han proporcionado máquinas virtuales Windows a los alumnos, deben conectarse a sus máquinas virtuales y usar cuadernos de Jupyter Notebook que estén disponibles de forma local en ellas. 

Para conectarse a una máquina virtual de Windows, un alumno puede usar una conexión a escritorio remoto (RDP). Para obtener pasos detallados, vea [Procedimientos para acceder a un laboratorio de clase](how-to-use-classroom-lab.md). 

Un alumno que use Mac o Chromebook puede seguir las instrucciones de los artículos siguientes para conectarse a Data Science Virtual Machine para Windows. 

- [Conexión a una máquina virtual mediante RDP en un equipo Mac](connect-virtual-machine-mac-remote-desktop.md)
- [Conexión a una máquina virtual mediante RDP en un equipo Chromebook](connect-virtual-machine-chromebook-remote-desktop.md).

### <a name="for-linux-vms"></a>Para las máquinas virtuales Linux
Si se han proporcionado máquinas virtuales Linux a los alumnos, hay varias opciones que pueden usar para conectarse a sus cuadernos de Jupyter Notebook en las máquinas virtuales:

- Acceder a cuadernos de Jupyter Notebook localmente después de conectarse a la máquina virtual.
    - SSH a la VM para sesiones de terminal
     - Conexión X2Go a la VM para las sesiones gráficas
- Usar la tunelización SSH para conectarse directamente desde el equipo local del alumno al servidor Jupyter en la máquina virtual. 

En las secciones siguientes se proporcionan detalles acerca de estas formas de conectarse a los cuadernos de Jupyter Notebook. 

#### <a name="ssh-to-virtual-machine"></a>SSH a la máquina virtual
Los alumnos pueden conectarse mediante SSH a sus máquinas virtuales Linux desde una sesión de terminal. Para obtener pasos detallados, vea [Procedimientos para acceder a un laboratorio de clase](how-to-use-classroom-lab.md). Si usan una máquina cliente Windows, deberán habilitar un cliente SSH mediante la descarga de [PuTTy](https://www.putty.org/) o la habilitación de [OpenSSH en Windows](/windows-server/administration/openssh/openssh_install_firstuse) a SSH desde el símbolo del sistema. 

1.  Inicie la máquina virtual.
2.  Una vez que la máquina virtual se esté ejecutando, haga clic en **Conectar**, que mostrará un cuadro de diálogo que proporciona la cadena de comando SSH, la cual tendrá el aspecto siguiente:
    
     ```shell
    ssh -p 12345 student@ml-lab-00000000-0000-0000-0000-000000000000.eastus2.cloudapp.azure.com
     ```
3.  Vaya al símbolo del sistema o terminal y pegue este comando y, después, presione **ENTRAR**.
4.  Escriba la contraseña para iniciar sesión en la máquina virtual. 

Una vez que los alumnos estén conectados a las máquinas virtuales, pueden acceder a los cuadernos de Jupyter Notebook y ejecutarlos de forma local.

#### <a name="x2go-to-virtual-machine"></a>X2Go a la máquina virtual
La imagen de **Data Science Virtual Machine: Ubuntu** ya está aprovisionada con el servidor de X2Go y está lista para aceptar conexiones de cliente. Para conectarse al escritorio gráfico de la máquina Linux, los alumnos deben seguir estos pasos de una sola vez para configurar X2Go en sus máquinas cliente:

1.  Descargue e instale el [cliente X2Go](https://wiki.x2go.org/doku.php/doc:installation:x2goclient) para su plataforma cliente.
2.  En el [portal de Azure Lab Services](https://labs.azure.com), asegúrese de que se ha iniciado la máquina virtual Linux a la que quiere conectarse.
3.  Una vez que la máquina virtual se esté ejecutando, haga clic en **Conectar**, que mostrará un cuadro de diálogo que proporciona la cadena de comando SSH, la cual tendrá el aspecto siguiente:

    ```
     ssh -p 12345 student@ml-lab-00000000-0000-0000-0000-000000000000.eastus2.cloudapp.azure.com
     ```    
4. Una vez que tenga esta información, abra la aplicación cliente de X2Go y cree una sesión nueva. 
5.  Rellene los valores siguientes en el panel **Preferencias de la sesión**:
    - **Nombre de sesión**: puede ser el que quiera, pero se recomienda usar el nombre de la máquina virtual de laboratorio.
     - **Host**: `ml-lab-00000000-0000-0000-0000-000000000000.eastus2.cloudapp.azure.com`
     - **Inicio de sesión**: alumno
     - **Puerto SSH**: 12345
     - **Tipo de sesión**: XFCE
6. Seleccione **Aceptar**. 

    > [!NOTE]
     > Al crear una sesión nueva de X2Go, asegúrese de usar el puerto SSH **no** el puerto RDP.

Ahora, para conectarse a la máquina virtual, siga estos pasos:    

1.  En el cliente de X2Go, haga doble clic en la máquina virtual a la que quiere conectarse. 

    ![Cliente de X2Go](./media/class-type-jupyter-notebook/x2go-client.png)
2. Escriba la contraseña para conectarse a la máquina virtual. Para finalizar la conexión, es posible que tenga que conceder a X2Go permiso para omitir el firewall.
3.  Ahora debería ver la interfaz gráfica de la instancia de Data Science Virtual Machine para Ubuntu.

#### <a name="ssh-tunnel-to-jupyter-server-on-the-vm"></a>Tunelización SSH al servidor de Jupyter en la máquina virtual
Es posible que algunos alumnos quieran conectarse directamente desde su equipo local al servidor de Jupyter dentro de sus máquinas virtuales. El protocolo SSH permite el reenvío de puertos entre el equipo local y un servidor remoto (en nuestro caso, la máquina virtual de laboratorio del alumno), de modo que una aplicación que se ejecuta en un determinado puerto del servidor se **tuneliza** al puerto de asignación en el equipo local. Los alumnos deben seguir estos pasos para realizar la tunelización SSH al servidor de Jupyter en sus máquinas virtuales de laboratorio:

1.  En el [portal de Azure Lab Services](https://labs.azure.com), asegúrese de que se ha iniciado la máquina virtual Linux a la que quiere conectarse.
2.  Una vez que la máquina virtual se esté ejecutando, haga clic en **Conectar**, que mostrará un cuadro de diálogo que proporciona la cadena de comandos SSH, la cual tendrá la cadena siguiente:

    ```bash
     ssh -p 12345 student@ml-lab-00000000-0000-0000-0000-000000000000.eastus2.cloudapp.azure.com
     ```
3. En el equipo local, inicie un terminal o símbolo del sistema y copie la cadena de conexión SSH en él. Después, agregue `-L 8888:localhost:8888` a la cadena de comandos, que crea el **túnel** entre los puertos. La cadena final debe tener este aspecto:

    ```bash
     ssh –L 8888:localhost:8888 -p 12345 student@ml-lab-b720853e-570f-49ac-9cb2-bd0bd2aeec35.eastus.cloudapp.azure.com
     ```
4. Presione **ENTRAR** para ejecutar el comando. 
5.  Cuando se le solicite, proporcione la contraseña para conectarse a la máquina virtual de laboratorio. 
6.  Una vez que esté conectado a la máquina virtual, inicie el servidor de Jupyter con este comando: 

    ```bash
     jupyter notebook
     ```
7. La ejecución del comando proporcionará una dirección URL en el terminal o el símbolo del sistema. La dirección URL debe tener el aspecto siguiente:

    ```bash
     http://localhost:8888/?token=8c09ecfc93e6a8cbedf9c66dffdae19670a64acc1d37
     ```
8. Pegue esta dirección URL en un explorador del equipo local para conectarse y trabajar en el cuaderno de Jupyter Notebook. 

    > [!NOTE]
    > Visual Studio Code también habilita una excelente [experiencia de edición de Jupyter Notebook](https://code.visualstudio.com/docs/python/jupyter-support). Se pueden seguir las instrucciones sobre [cómo conectarse a un servidor remoto de Jupyter](https://code.visualstudio.com/docs/python/jupyter-support#_connect-to-a-remote-jupyter-server) y usar la misma dirección URL del paso anterior para conectarse desde VS Code en lugar de hacerlo desde el explorador. 


## <a name="cost-estimate"></a>Estimación del costo
Vamos a cubrir una posible estimación de costos para esta clase. Usaremos una clase de 25 alumnos. Hay 20 horas de tiempo de clase programado. Además, cada alumno obtiene una cuota de 10 horas para deberes o tareas fuera del tiempo de clase programado. El tamaño de la máquina virtual que hemos elegido es el de GPU pequeña (proceso), que tiene 139 unidades de laboratorio. Si se quiere usar el tamaño Pequeño (20 unidades de laboratorio) o el Medio (42 unidades de laboratorio), se puede reemplazar la parte de la unidad de laboratorio de la ecuación siguiente por el número correcto.  

A continuación se incluye un ejemplo de una posible estimación del costo para esta clase: 25 alumnos * (20 horas programadas + 10 horas de cuota) * 139 unidades de laboratorio * 0,01 USD por hora = 1042,5 USD

Para más detalles sobre los precios, consulte [Precios de Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

## <a name="conclusion"></a>Conclusión
En este artículo, se describen los pasos para crear un laboratorio para una clase de Jupyter Notebook. Puede usar una configuración similar para otras clases de aprendizaje automático.

## <a name="next-steps"></a>Pasos siguientes

Los pasos siguientes son comunes a la configuración de cualquier laboratorio.

- [Creación y administración de plantillas](how-to-create-manage-template.md)
- [Incorporación de usuarios](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Establecimiento de la cuota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Establecimiento de la programación](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Vínculos de registro por correo electrónico para los alumnos](how-to-configure-student-usage.md#send-invitations-to-users)