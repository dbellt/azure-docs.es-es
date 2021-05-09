---
title: 'Portal para la asociación de aplicaciones en formato MSIX para Windows Virtual Desktop: Azure'
description: Configuración de la asociación de aplicaciones en formato MSIX para Windows Virtual Desktop mediante Azure Portal.
author: Heidilohr
ms.topic: how-to
ms.date: 04/13/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: da084bfab39eb0e12d90f25f3658dc4507ca63dc
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107906663"
---
# <a name="set-up-msix-app-attach-with-the-azure-portal"></a>Configuración de la asociación de aplicaciones en formato .MSIX con Azure Portal

Este artículo le guiará a través de la configuración de la asociación de aplicaciones en formato MSIX en un entorno de Windows Virtual Desktop.

## <a name="requirements"></a>Requisitos

Esto es lo que necesita para configurar la asociación de aplicaciones en formato MSIX:

- Una implementación de Windows Virtual Desktop en funcionamiento. Para obtener información sobre cómo implementar Windows Virtual Desktop (clásico), consulte [Creación de un inquilino en Windows Virtual Desktop](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md). Para obtener información sobre cómo implementar Windows Virtual Desktop con la integración de Azure Resource Manager, consulte [Creación de un grupo de hosts con Azure Portal](./create-host-pools-azure-marketplace.md).
- Un grupo de hosts de Windows Virtual Desktop con al menos un host de sesión activo.
- La herramienta de empaquetado MSIX.
- Una aplicación empaquetada en formato MSIX expandida en una imagen MSIX que se carga en un recurso compartido de archivos.
- Un recurso compartido de archivos en la implementación de Windows Virtual Desktop donde se almacenará el paquete MSIX.
- El recurso compartido de archivos en el que cargó la imagen MSIX también debe ser accesible para todas las máquinas virtuales (VM) del grupo de hosts. Los usuarios necesitarán permisos de solo lectura para acceder a la imagen.
- Si el certificado no es de confianza pública, siga las instrucciones de [Instalación de certificados](app-attach.md#install-certificates).
- Actualmente, la nube de Azure Government no admite la asociación de aplicaciones en formato MSIX.

## <a name="turn-off-automatic-updates-for-msix-app-attach-applications"></a>Desactivación de las actualizaciones automáticas para las aplicaciones de asociación de aplicaciones en formato MSIX

Antes de comenzar, debe deshabilitar las actualizaciones automáticas para las aplicaciones de asociación de aplicaciones en formato MSIX. Para deshabilitar las actualizaciones automáticas, deberá ejecutar estos comandos en un símbolo del sistema con privilegios elevados:

```cmd
rem Disable Store auto update:

reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Automatic app update" /Disable
Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable

rem Disable Content Delivery auto download apps that they want to promote to users:

reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f

reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f

```

## <a name="configure-the-msix-app-attach-management-interface"></a>Configuración de la interfaz de administración de la asociación de aplicaciones en formato MSIX

A continuación, debe descargar y configurar la interfaz de administración de la asociación de aplicaciones en formato MSIX en Azure Portal.

Para configurar la interfaz de administración:

1. [Abra Azure Portal](https://portal.azure.com).
2. Si recibe un mensaje que le pregunta si la extensión es de confianza, seleccione **Permitir**.

      > [!div class="mx-imgBorder"]
      > ![Captura de pantalla de la ventana extensiones que no son de confianza. La opción "Permitir" está resaltada en rojo.](media/untrusted-extensions.png)


## <a name="add-an-msix-image-to-the-host-pool"></a>Agregación una imagen de MSIX al grupo de hosts

Deberá agregar la imagen de MSIX al grupo de hosts.

Para agregar la imagen de MSIX:

1. Abra Azure Portal.

2. Escriba **Windows Virtual Desktop** en la barra de búsqueda y seleccione el nombre del servicio.

      > [!div class="mx-imgBorder"]
      > ![Captura de pantalla de un usuario que selecciona "Windows Virtual Desktop" en el menú desplegable de la barra de búsqueda en Azure Portal. "Windows Virtual Desktop" está resaltado en rojo.](media/find-and-select.png)

3. Seleccione el grupo de hosts en el que va a colocar las aplicaciones de MSIX.

4. Seleccione los **paquetes de MSIX** para abrir la cuadrícula de datos que tiene todos los **paquetes de MSIX** que se han agregado actualmente al grupo de hosts.

5. Seleccione **+ Agregar** para abrir la pestaña **Add MSIX package** (Agregar paquete de MSIX).

6. En la pestaña **Add MSIX package** (Agregar paquete de MSIX), escriba los valores siguientes:

      - En la **ruta de acceso de la imagen de MSIX**, escriba una ruta de acceso UNC válida que apunte a la imagen de MSIX en el recurso compartido de archivos. (Por ejemplo, `\\storageaccount.file.core.windows.net\msixshare\appfolder\MSIXimage.vhd`). Cuando haya terminado, seleccione **Agregar** para consultar el contenedor de MSIX y comprobar si la ruta de acceso es válida.

      - En el **paquete de MSIX**, seleccione el nombre del paquete de MSIX que vaya a usar en el menú desplegable. Este menú solo se rellenará si ha especificado una ruta de acceso de imagen válida en la **ruta de acceso de la imagen de MSIX**.

      - En el caso de las **aplicaciones de paquetes**, asegúrese de que la lista contiene todas las aplicaciones de MSIX que quiera que estén disponibles para los usuarios en el paquete de MSIX.

      - Opcionalmente, puede escribir un **nombre para mostrar** si quiere que el paquete tenga un usuario más descriptivo en las implementaciones de usuario.

      - Asegúrese de que la **versión** tenga el número de versión correcto.

      - Seleccione el **tipo de registro** que quiera usar. El que use dependerá de sus necesidades:

    - El **registro a petición** pospone el registro completo de la aplicación de MSIX hasta que el usuario inicia la aplicación. Este es el tipo de registro que se recomienda usar.

    - El **bloqueo de inicio de sesión** solo realiza el registro mientras el usuario inicia sesión. No se recomienda este tipo porque puede dar lugar a tiempos de inicio de sesión más prolongados para los usuarios.

7.  En **Estado**, seleccione su estado preferido.
    -  El estado **Activo** permite a los usuarios interactuar con el paquete.
    -  El estado **Inactivo** hace que Windows Virtual Desktop ignore el paquete y no lo entregue a los usuarios.

8. Cuando finalice, seleccione **Agregar**.

## <a name="publish-msix-apps-to-an-app-group"></a>Publicación de aplicaciones en formato MSIX en un grupo de aplicaciones

A continuación, tendrá que publicar las aplicaciones en el paquete. Tendrá que hacer esto en los grupos de aplicaciones de escritorio y de aplicación remota.

Si ya tiene una imagen de MSIX, vaya a la opción [Publish MSIX apps to an app group](#publish-msix-apps-to-an-app-group) (Publicar aplicaciones de MSIX en un grupo de aplicaciones). Si quiere probar las aplicaciones heredadas, siga las instrucciones que aparecen en [Creación de un paquete de MSIX a partir de un instalador de escritorio en una máquina virtual](/windows/msix/packaging-tool/create-app-package-msi-vm/) para convertir la aplicación heredada en un paquete de MSIX.

Para publicar las aplicaciones:

1. En el proveedor de recursos de Windows Virtual Desktop, seleccione la pestaña **Grupos de aplicaciones**.

2. Seleccione el grupo de aplicaciones en el que quiera publicar las aplicaciones.

   >[!NOTE]
   >Las aplicaciones de MSIX pueden entregarse con la aplicación de MSIX adjunta a los grupos de aplicaciones de escritorio y de aplicaciones remotas.

3. Una vez en el grupo de aplicaciones, seleccione la pestaña **Aplicaciones**. En la cuadrícula de **Aplicaciones** se mostrarán todas las aplicaciones existentes en el grupo de aplicaciones.

4. Seleccione **+ Agregar** para abrir la pestaña **Agregar aplicación**.

      > [!div class="mx-imgBorder"]
      > ![Captura de pantalla de un usuario que selecciona "+ Agregar" para abrir la pestaña para agregar aplicaciones.](media/select-add.png)

5. En **Origen de la aplicación**, elija el origen de la aplicación.
    - Si usa un grupo de aplicaciones de escritorio, elija **Paquete de MSIX**.
      
      > [!div class="mx-imgBorder"]
      > ![Captura de pantalla de un cliente que selecciona el paquete de MSIX del menú desplegable del origen de la aplicación. El paquete de MSIX está resaltado en rojo.](media/select-source.png)
    
    - Si usa un grupo de aplicaciones remotas, elija una de las siguientes opciones:
        
        - Menú Inicio
        - Ruta de acceso de la aplicación
        - Paquete MSIX

    - En **Nombre de la aplicación**, escriba un nombre descriptivo para la aplicación.

    También puede configurar las siguientes características opcionales:
   
    - En el **nombre para mostrar**, escriba un nuevo nombre para el paquete que verán los usuarios.

    - En **Descripción**, escriba una breve descripción del paquete de la aplicación.

    - Si usa un grupo de aplicaciones remotas, también puede configurar estas opciones:

        - **Ruta de icono**
        - **Índice de icono**
        - **Mostrar en fuente web**

6. Cuando finalice, seleccione **Guardar**.

>[!NOTE]
>Cuando se asigna un usuario al grupo de aplicaciones remotas y al grupo de aplicaciones de escritorio desde el mismo grupo de hosts, el grupo de aplicaciones de escritorio se mostrará en la fuente.

## <a name="assign-a-user-to-an-app-group"></a>Asignación de un usuario a un grupo de aplicaciones

Después de asignar las aplicaciones de MSIX a un grupo de aplicaciones, deberá conceder a los usuarios acceso a ellas. Puede asignar el acceso agregando usuarios o grupos de usuarios a un grupo de aplicaciones con aplicaciones de MSIX publicadas. Para ello, siga las instrucciones que se detallan en [Administración de grupos de aplicaciones con Azure Portal](manage-app-groups.md) para asignar usuarios a un grupo de aplicaciones.

>[!NOTE]
>Las aplicaciones remotas de asociación de aplicaciones en formato MSIX pueden desaparecer de la fuente al probar las aplicaciones remotas durante la versión preliminar pública. Las aplicaciones no aparecen porque el grupo de hosts que usa en el entorno de evaluación recibe la atención de un agente de Escritorio remoto en el entorno de producción. Dado que el agente de Escritorio remoto en el entorno de producción no registra la presencia de las aplicaciones remotas de asociación de aplicaciones en formato MSIX, las aplicaciones no aparecerán en la fuente.

## <a name="change-msix-package-state"></a>Cambio del estado del paquete de MSIX

A continuación, deberá cambiar el estado del paquete de MSIX a **Activo** o **Inactivo**, en función de lo que quiera hacer con el paquete. Los paquetes activos son paquetes que los usuarios pueden usar una vez que se publican. Los paquetes inactivos se omiten en Windows Virtual Desktop, por lo que los usuarios no pueden interactuar con las aplicaciones dentro de ellos.

### <a name="change-state-with-the-applications-list"></a>Cambio del estado con la lista de aplicaciones

Para cambiar el estado del paquete con la lista de aplicaciones:

1. Vaya al grupo de hosts y seleccione **Paquetes de MSIX**. Debería ver una lista de todos los paquetes de MSIX existentes en el grupo de hosts.

2. Seleccione los paquetes de MSIX cuyos estados debe cambiar y, a continuación, seleccione **Cambiar estado**.

### <a name="change-state-with-update-package"></a>Cambio de estado con el paquete de actualización

Para cambiar el estado del paquete con un paquete de actualización:

1. Vaya al grupo de hosts y seleccione **Paquetes de MSIX**. Debería ver una lista de todos los paquetes de MSIX existentes en el grupo de hosts.

2. Seleccione el nombre del paquete cuyo estado quiere cambiar en la lista de paquetes de MSIX. Se abrirá la pestaña **Paquete de actualización**.

3. Cambie el **Estado** a **Inactivo** o **Activo** y, a continuación, seleccione **Guardar**.

## <a name="change-msix-package-registration-type"></a>Cambio del tipo de registro del paquete de MSIX

Para cambiar el tipo de registro del paquete:

1. Seleccione **Paquetes de MSIX**. Debería ver una lista de todos los paquetes de MSIX existentes en el grupo de hosts.

2. Seleccione el **nombre del paquete en** la **cuadrícula de paquetes de MSIX** para abrir la hoja de actualización del paquete.

3. Cambie el **tipo de registro** que quiera mediante el botón **On-demand/Log on blocking** (Bloqueo a petición o al iniciar sesión) y seleccione **Guardar**.

## <a name="remove-an-msix-package"></a>Eliminación de un paquete de MSIX

Para quitar un paquete de MSIX del grupo de hosts:

1. Seleccione **Paquetes de MSIX**.  Debería ver una lista de todos los paquetes de MSIX existentes en el grupo de hosts.

2. Seleccione los puntos suspensivos que están en el lado derecho del nombre del paquete que quiera eliminar y, a continuación, seleccione **Quitar**.

## <a name="remove-msix-apps"></a>El de aplicaciones de MSIX

Para quitar aplicaciones individuales de MSIX del paquete:

1. Vaya al grupo de hosts y seleccione **Grupos de aplicaciones**.

2. Seleccione el grupo de aplicaciones del que quiera quitar las aplicaciones de MSIX.

3. Haga clic en la pestaña **Aplicaciones**.

4. Seleccione la aplicación que quiera quitar y, a continuación, seleccione **Quitar**.

## <a name="next-steps"></a>Pasos siguientes

Pregunte a la comunidad sobre esta característica en el espacio de [TechCommunity sobre Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).

También puede dejar comentarios sobre Windows Virtual Desktop en el [Concentrador de comentarios de Windows Virtual Desktop](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app).

Estos son algunos otros artículos que pueden resultarle útiles:

- [Glosario sobre la conexión de aplicaciones en formato MSIX](app-attach-glossary.md)
- [Preguntas frecuentes sobre la asociación de aplicaciones en formato MSIX](app-attach-faq.md)
