---
title: Instalación de actualizaciones en un dispositivo Azure Stack Edge Pro con GPU | Microsoft Docs
description: Se describe cómo aplicar actualizaciones desde Azure Portal y la interfaz de usuario web local en un dispositivo Azure Stack Edge Pro con GPU y en el clúster de Kubernetes del dispositivo
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 05/27/2021
ms.author: alkohli
ms.openlocfilehash: 5fd8dad44aa085530426168961280c4a84162ecb
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110617679"
---
# <a name="update-your-azure-stack-edge-pro-gpu"></a>Actualización de la GPU de Azure Stack Edge Pro 

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

En este artículo, se describen los pasos necesarios para instalar actualizaciones en el dispositivo Azure Stack Edge Pro con GPU desde la interfaz de usuario web local y Azure Portal. Para mantener actualizado el dispositivo Azure Stack Edge Pro y el clúster de Kubernetes asociado, es necesario aplicar correcciones y actualizaciones de software.

El procedimiento descrito en este artículo se realizó con una versión de software diferente, pero el proceso sigue siendo el mismo para la versión de software actual.

> [!IMPORTANT]
> - La actualización **2105** es la actualización actual y corresponde a:
>   - Versión de software del dispositivo: **2.2.1606.3320**
>   - Versión del servidor de Kubernetes: **v1.20.2**
>   - Versión de IoT Edge: **0.1.0-beta14**
>   - Versión del controlador de GPU: **460.32.03**
>   - Versión de CUDA: **11.2**
>    
>    Para obtener más información sobre cuáles son las novedades de esta actualización, vaya a las [Notas de la versión](azure-stack-edge-gpu-2105-release-notes.md).
> - Para aplicar la actualización 2105, el dispositivo debe ejecutar la versión 2010. Si no ejecuta la versión mínima admitida, verá un error que dice que *no se puede instalar el paquete de actualización porque no se cumplen sus dependencias*.
> - Esta actualización necesita que se apliquen dos actualizaciones seguidas. Primero deben aplicarse las actualizaciones de software del dispositivo y, después, las de Kubernetes.
> - Tenga en cuenta que al instalar una actualización o revisión, se reiniciará el dispositivo. Esta actualización contiene las actualizaciones de software del dispositivo y las actualizaciones de Kubernetes. Como la GPU de Azure Stack Edge Pro es un dispositivo de un solo nodo, cualquier operación de E/S en curso se interrumpirá y el dispositivo experimenta un tiempo de inactividad de hasta una hora y media durante la actualización.

Para instalar actualizaciones en el dispositivo, primero tiene que configurar la ubicación del servidor donde se encuentran las actualizaciones. Una vez configurado el servidor, puede aplicar las actualizaciones utilizando la interfaz de usuario de Azure Portal o la interfaz de usuario web local.

En las secciones siguientes se describen todos estos pasos.

## <a name="configure-update-server"></a>Configuración del servidor de actualizaciones

1. En la interfaz de usuario web local, vaya a **Configuración** > **Servidor de actualización**.
   
    ![Configuración de las actualizaciones 1](./media/azure-stack-edge-gpu-install-update/configure-update-server-1.png)

2. En **Select update server type** (Seleccionar tipo de servidor de actualización), en la lista desplegable, elija el servidor de Microsoft Update (opción predeterminada) o Windows Server Update Services.  
   
    Si está realizando la actualización desde Windows Server Update Services, especifique el URI del servidor. El servidor de dicho URI implementará las actualizaciones en todos los dispositivos que tenga conectados.

    ![Configuración de las actualizaciones 2](./media/azure-stack-edge-gpu-install-update/configure-update-server-2.png)
    
    El servidor de WSUS se utiliza para administrar y distribuir actualizaciones utilizando una consola de administración. Un servidor WSUS también puede ser el origen de actualización para otros servidores WSUS de la organización. El servidor WSUS que actúa como origen de actualización es el servidor que precede en la cadena. En una implementación de WSUS, al menos un servidor WSUS de la red debe tener la capacidad de conectarse a Microsoft Update para obtener información sobre las actualizaciones disponibles. Como administrador puedes determinar, en función de la configuración y seguridad de red, la cantidad de servidores WSUS que quieres que se conecten directamente a Microsoft Update.
    
    Para más información, consulte [Windows Server Update Services (WSUS)](/windows-server/administration/windows-server-update-services/get-started/windows-server-update-services-wsus).

## <a name="use-the-azure-portal"></a>Uso de Azure Portal

Se recomienda instalar las actualizaciones mediante Azure Portal. El dispositivo busca actualizaciones automáticamente una vez al día. Una vez que las actualizaciones estén disponibles, verá una notificación en el portal. Puede descargar e instalar las actualizaciones.

> [!NOTE]
> Asegúrese de que el dispositivo esté en buen estado y que el estado se muestre como **Your device is running fine!** (El dispositivo funciona correctamente) antes de continuar con la instalación de las actualizaciones.

1. Cuando las actualizaciones estén disponibles para el dispositivo, verá una notificación. Seleccione la notificación o, en la barra de comandos superior, seleccione **Actualizar dispositivo**. Esto le permitirá aplicar las actualizaciones de software del dispositivo.

    ![Versión de software después de la actualización](./media/azure-stack-edge-gpu-install-update/portal-update-1.png)

2. En la hoja **Actualizaciones del dispositivo**, compruebe que ha revisado los términos de licencia asociados a las nuevas características en las notas de la versión.

    Puede optar por **descargar e instalar** las actualizaciones o simplemente **descargarlas**. Después, puede optar por instalar estas actualizaciones más adelante.

    ![Versión de software después de la actualización 2](./media/azure-stack-edge-gpu-install-update/portal-update-2-a.png)    

    Si desea descargar e instalar las actualizaciones, active la opción que permite que las actualizaciones se instalen automáticamente una vez completada la descarga.

    ![Versión de software después de la actualización 3](./media/azure-stack-edge-gpu-install-update/portal-update-2-b.png)

3. Se inicia la descarga de actualizaciones. Verá una notificación de que la descarga está en curso.

    ![Versión de software después de la actualización 4](./media/azure-stack-edge-gpu-install-update/portal-update-3.png)

    También se muestra un banner de notificación en Azure Portal. Indica el progreso de la descarga.

    ![Versión de software después de la actualización 5](./media/azure-stack-edge-gpu-install-update/portal-update-4.png)

    Puede seleccionar esta notificación o seleccionar **Actualizar dispositivo** para ver el estado detallado de la actualización.

    ![Versión de software después de la actualización 6](./media/azure-stack-edge-gpu-install-update/portal-update-5.png)


4. Una vez completada la descarga, el banner de notificación se actualiza para indicar la finalización. Si decide descargar e instalar las actualizaciones, la instalación se iniciará automáticamente.

    Si decide descargar solo las actualizaciones, seleccione la notificación para abrir la hoja **Actualizaciones del dispositivo**. Seleccione **Instalar**.
  
5. Verá una notificación de que la instalación está en curso. El portal también muestra una alerta informativa para indicar que la instalación está en curso. El dispositivo se queda sin conexión y pasa a modo de mantenimiento.
   
    ![Versión de software después de la actualización 10](./media/azure-stack-edge-gpu-install-update/portal-update-9.png)

6. Dado que se trata de un dispositivo de un nodo, el dispositivo se reiniciará una vez instaladas las actualizaciones. La alerta crítica durante el reinicio indica que el latido del dispositivo se ha perdido.

    ![Versión de software después de la actualización 11](./media/azure-stack-edge-gpu-install-update/portal-update-10.png)

    Seleccione la alerta para ver el evento de dispositivo correspondiente.
    
    ![Versión de software después de la actualización 12](./media/azure-stack-edge-gpu-install-update/portal-update-11.png)

7. Después del reinicio, el software del dispositivo terminará de actualizarse. Una vez completada la actualización, puede consultar en la interfaz de usuario web local para comprobar que el software del dispositivo se ha actualizado. No se ha actualizado la versión del software de Kubernetes.

    ![Versión de software después de la actualización 13](./media/azure-stack-edge-gpu-install-update/portal-update-12.png)

8. Verá un banner de notificación que indicará que hay disponibles actualizaciones del dispositivo. Seleccione este banner para empezar a actualizar el software de Kubernetes en el dispositivo. 

    ![Versión de software después de la actualización 13a](./media/azure-stack-edge-gpu-install-update/portal-update-13.png) 


    ![Versión de software después de la actualización 14](./media/azure-stack-edge-gpu-install-update/portal-update-14-a.png) 

    Si selecciona **Actualizar dispositivo** en la barra de comandos superior, podrá ver el progreso de las actualizaciones.  

    ![Versión de software después de la actualización 15](./media/azure-stack-edge-gpu-install-update/portal-update-14-b.png) 


8. El estado del dispositivo se actualiza a **Your device is running fine** (El dispositivo funciona correctamente) después de instalar las actualizaciones. 

    ![Versión de software después de la actualización 16](./media/azure-stack-edge-gpu-install-update/portal-update-15.png)

    Vaya a la interfaz de usuario web local y, a continuación, vaya a la página **Actualización de software**. Compruebe que la actualización de Kubernetes se ha instalado correctamente y que la versión del software así lo refleje.

    ![Versión de software después de la actualización 17](./media/azure-stack-edge-gpu-install-update/portal-update-16.png)


Una vez que las actualizaciones de software y Kubernetes del dispositivo se instalen correctamente, la notificación del banner desaparece. Ahora, el dispositivo tendrá la última versión del software del dispositivo y de Kubernetes.


## <a name="use-the-local-web-ui"></a>Uso de la interfaz de usuario web local

Se pueden seguir dos pasos con la interfaz de usuario web local:

* Descargar la actualización o la revisión
* Instalar la actualización o la revisión

En las siguientes secciones se describen estos pasos.


### <a name="download-the-update-or-the-hotfix"></a>Descargar la actualización o la revisión

Realice los pasos siguientes para descargar la actualización. Puede descargar la actualización desde la ubicación proporcionada por Microsoft o desde el Catálogo de Microsoft Update.


Realice los pasos siguientes para descargar la actualización desde el Catálogo de Microsoft Update.

1. Inicie el explorador y vaya a [https://catalog.update.microsoft.com](https://catalog.update.microsoft.com).

    ![Catálogo de búsqueda](./media/azure-stack-edge-gpu-install-update/download-update-1.png)

2. En el cuadro de búsqueda del Catálogo de Microsoft Update, escriba el número de Knowledge Base (KB) correspondiente a la revisión o los términos de la actualización que quiera descargar. Por ejemplo, escriba **Azure Stack Edge** y haga clic en **Buscar**.
   
    La lista de actualizaciones aparece como **Azure Stack Edge Update 2105**.
   
    <!--![Search catalog 2](./media/azure-stack-edge-gpu-install-update/download-update-2-b.png)-->

4. Seleccione **Descargar**. Hay dos paquetes para descargar: <!--KB 4616970 and KB 4616971--> uno para las actualizaciones de software del dispositivo (*SoftwareUpdatePackage.exe*) y otro para las actualizaciones de Kubernetes (*Kubernetes_Package.exe*) respectivamente. Descargue los paquetes en una carpeta del sistema local. También puede copiar la carpeta en un recurso compartido de red que sea accesible desde el dispositivo.

### <a name="install-the-update-or-the-hotfix"></a>Instalar la actualización o la revisión

Antes de instalar la actualización o revisión, compruebe que:

 - Tiene la actualización o la revisión descargada localmente en el host o que puede acceder a ella a través de un recurso compartido de red.
 - El estado del dispositivo es correcto, tal como se muestra en la página **información general** de la interfaz de usuario web local.

   ![actualizar dispositivo](./media/azure-stack-edge-gpu-install-update/local-ui-update-1.png)

Este procedimiento tarda aproximadamente 20 minutos en completarse. Realice los pasos siguientes para instalar la actualización o revisión.

1. En la interfaz de usuario web local, vaya a **Mantenimiento** > **Actualización de software**. Tome nota de la versión de software que se está ejecutando. 
   
   ![Actualización de dispositivo 2](./media/azure-stack-edge-gpu-install-update/local-ui-update-2.png)

2. Proporcione la ruta de acceso al archivo de actualización. Asimismo, también puede acceder al archivo de instalación de la actualización si está en un recurso compartido de red. Seleccione el archivo de actualización de software con el sufijo *SoftwareUpdatePackage.exe*.

   ![Actualización de dispositivo 3](./media/azure-stack-edge-gpu-install-update/local-ui-update-3-a.png)

3. Seleccione **Aplicar**.

   ![Actualización de dispositivo 4](./media/azure-stack-edge-gpu-install-update/local-ui-update-4.png)

4. Cuando se le pida confirmación, seleccione **Sí** para continuar. Dado que se trata de un dispositivo de un solo nodo, una vez aplicada la actualización, se reiniciará el dispositivo y habrá un tiempo de inactividad. 
   
   ![Actualización de dispositivo 5](./media/azure-stack-edge-gpu-install-update/local-ui-update-5.png)

5. Se inicia la actualización. Una vez que el dispositivo se actualice correctamente, este se reiniciará. La interfaz de usuario local no será accesible durante este tiempo.
   
6. Una vez completado el reinicio, se le llevará a la página de **inicio de sesión** . Para comprobar que el software del dispositivo se ha actualizado, en la interfaz de usuario de web local, vaya a **Mantenimiento** > **Actualización de software**. Con el lanzamiento actual, la versión de software que aparece debe ser **Azure Stack Edge 2105**. 

   <!--![update device 6](./media/azure-stack-edge-gpu-install-update/local-ui-update-6.png)-->

7. Ahora, va a actualizar la versión del software de Kubernetes. Repita los pasos anteriores. Especifique la ruta de acceso al archivo de actualización de Kubernetes que tenga el sufijo *Kubernetes_Package. exe*.  

   <!--![update device](./media/azure-stack-edge-gpu-install-update/local-ui-update-7.png)-->

8. Seleccione **Aplicar actualización**.

   ![Actualización de dispositivo 7](./media/azure-stack-edge-gpu-install-update/local-ui-update-8.png)

9. Cuando se le pida confirmación, seleccione **Sí** para continuar.

10. Una vez que la actualización de Kubernetes se haya instalado correctamente, no aparecerá ningún cambio en el software que figura en **Mantenimiento** > **Actualización de software**.


## <a name="next-steps"></a>Pasos siguientes

Más información sobre [administración del dispositivo Azure Stack Edge Pro](azure-stack-edge-manage-access-power-connectivity-mode.md).
