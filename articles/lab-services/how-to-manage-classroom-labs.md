---
title: Administración de laboratorios en Azure Lab Services | Microsoft Docs
description: Obtenga información sobre cómo crear y configurar un laboratorio de clase, ver todos los laboratorios, compartir el vínculo de registro con un usuario del laboratorio o eliminar un laboratorio.
ms.topic: how-to
ms.date: 06/26/2020
ms.openlocfilehash: c6acb9609abac15b9ff92250e3d5d44c585881cc
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107481810"
---
# <a name="manage-labs-in-azure-lab-services"></a>Administración de laboratorios en Azure Lab Services 
En este artículo se describe cómo crear y eliminar un laboratorio de clase. También se muestra cómo ver todos los laboratorios de una cuenta de laboratorio. 

## <a name="prerequisites"></a>Prerrequisitos
Para configurar un laboratorio de clase en una cuenta de laboratorio, debe ser miembro del rol **Creador de laboratorio** en la cuenta de laboratorio. La cuenta que usó para crear una cuenta de laboratorio se agrega automáticamente a este rol. Un propietario de laboratorio puede agregar otros usuarios al rol Creador de laboratorio mediante los pasos que se indican en el siguiente artículo: [Incorporación de un usuario al rol Creador de laboratorio](tutorial-setup-lab-account.md#add-a-user-to-the-lab-creator-role).

## <a name="create-a-classroom-lab"></a>Creación de un laboratorio educativo

1. Vaya al [sitio web de Azure Lab Services](https://labs.azure.com).
1. Seleccione **Iniciar sesión** y escriba las credenciales. Seleccione o escriba un **Id. de usuario** que sea miembro del rol **Creador de laboratorio** en la cuenta del laboratorio y escriba la contraseña. Azure Lab Services es compatible con cuentas profesionales y cuentas Microsoft. 
1. Seleccione **New lab** (Nuevo laboratorio). 
    
    ![Creación de un laboratorio educativo](./media/tutorial-setup-classroom-lab/new-lab-button.png)
1. En la ventana **Nuevo laboratorio**, lleve a cabo las siguientes acciones: 
    1. Especifique un **nombre** para el laboratorio. 
    1. Seleccione el **tamaño de las máquinas virtuales** que necesite para la clase. Para ver la lista de tamaños disponibles, consulte la sección [VM Sizes](#vm-sizes) (Tamaños de máquina virtual). 
    1. Seleccione la **imagen de máquina virtual**  que quiere usar para el laboratorio educativo. Si selecciona una imagen de Linux, verá una opción para **habilitar la conexión a Escritorio remoto**. Para más información, consulte [Habilitar conexión de escritorio remoto para Linux](how-to-enable-remote-desktop-linux.md).

        Si ha iniciado sesión con las credenciales del propietario de la cuenta de laboratorio, verá una opción para habilitar más imágenes para el laboratorio. Para más información, vea [Habilitar imágenes para un laboratorio](specify-marketplace-images.md#enable-images-at-the-time-of-lab-creation).
    1. Revise el **precio total por hora** mostrado en la página. 
    1. Seleccione **Guardar**.

        ![Captura de pantalla que muestra la ventana "Nuevo laboratorio".](./media/tutorial-setup-classroom-lab/new-lab-window.png)

        > [!NOTE]
        > Se muestra una opción para seleccionar una ubicación del laboratorio si la cuenta de laboratorio se configuró en la opción [Permitir al creador del laboratorio seleccionar la ubicación correspondiente](allow-lab-creator-pick-lab-location.md). 
4. En la página **Virtual machine credentials** (Credenciales de la máquina virtual), especifique las credenciales predeterminadas de todas las máquinas virtuales del laboratorio.
    1. Especifique el **nombre del usuario** para todas las máquinas virtuales del laboratorio.
    2. Especifique la **contraseña** del usuario. 

        > [!IMPORTANT]
        > Tome nota de ambos. No se volverán a mostrar.
    3. Deshabilite la opción **Usar la misma contraseña para todas las máquinas virtuales** si quiere que los alumnos establezcan sus propias contraseñas. Este paso es **opcional**. 

        Un formador puede elegir usar la misma contraseña para todas las VM en el laboratorio o permitir a los alumnos establecer contraseñas para sus VM. De forma predeterminada, esta configuración está habilitada para todas las imágenes Windows y Linux, excepto para Ubuntu. Al seleccionar la máquina virtual **Ubuntu**, esta opción está deshabilitada, por lo que se pedirá a los alumnos que establezcan una contraseña cuando inicien sesión por primera vez.  

        ![Ventana de nuevo laboratorio](./media/tutorial-setup-classroom-lab/virtual-machine-credentials.png)
    4. A continuación, seleccione **Next** (Siguiente) en la página **Virtual machine credentials** (Credenciales de la máquina virtual). 
5. En la página **Directivas de laboratorio**, siga estos pasos:
    1. Escriba el número de horas asignadas para cada usuario (**cuota para cada usuario**) fuera del tiempo programado para el laboratorio. 
    2. En la opción **Apagado automático de máquinas virtuales**, especifique si desea que la VM se apague automáticamente cuando el usuario se desconecte. También puede especificar cuánto tiempo debe esperar la VM para que el usuario se vuelva a conectar antes de que se apague automáticamente. Para obtener más información, consulte [Habilitación del apagado automático de las máquinas virtuales al desconectarse](how-to-enable-shutdown-disconnect.md).
    3. Después, seleccione **Finalizar**. 

        ![Cuota para cada usuario](./media/tutorial-setup-classroom-lab/quota-for-each-user.png)
    
5. Debería ver la siguiente pantalla, que muestra el estado de la creación de la máquina virtual de plantilla. La creación de la plantilla en el laboratorio tarda un máximo de 20 minutos. 

    ![Estado de la creación de la plantilla de máquina virtual](./media/tutorial-setup-classroom-lab/create-template-vm-progress.png)
8. En la página **Template** (Plantilla), siga estos pasos: Estos pasos son **opcionales** para el tutorial.

    1. Seleccione **Conectar** para conectarse a la plantilla de máquina virtual. Si se trata de una plantilla de máquina virtual Linux, decida si quiere conectarse mediante SSH un escritorio remoto de la GUI.  Se requiere una configuración adicional para usar un escritorio remoto de la GUI. Para más información, consulte el artículo [Habilitación del Escritorio remoto para máquinas virtuales Linux en un laboratorio de Azure Lab Services](how-to-use-remote-desktop-linux-student.md).
    1. Seleccione **Restablecer contraseña** para restablecer la contraseña de la máquina virtual. 
    1. Instale y configure el software en la plantilla de máquina virtual. 
    1. **Pare** la máquina virtual.  
    1. Escriba una **descripción** para la plantilla.
9.  En la página **Template** (Plantilla), seleccione **Publish** (Publicar) en la barra de herramientas. 

    ![Botón de publicación de plantilla](./media/tutorial-setup-classroom-lab/template-page-publish-button.png)

    > [!WARNING]
    > Una vez que publique, no se puede cancelar la publicación. 
10. En la página **Publish template** (Publicar plantilla), escriba el número de máquinas virtuales que desea crear en el laboratorio y seleccione **Publish** (Publicar). 

    ![Publicación de plantilla: número de máquinas virtuales](./media/tutorial-setup-classroom-lab/publish-template-number-vms.png)
11. Puede ver el **estado de la publicación** de la plantilla en la página. Este proceso puede tardar hasta una hora. 

    ![Publicar plantilla: progreso](./media/tutorial-setup-classroom-lab/publish-template-progress.png)
12. Cambie a la página **Virtual machine pool** (Grupo de máquinas virtuales), para lo que debe seleccionar Virtual machines (Máquinas virtuales) en el menú izquierdo o el icono de las máquinas virtuales. Confirme que ve las máquinas virtuales que están en el estado **Sin asignar**. Estas máquinas virtuales no están asignadas a los alumnos todavía. Deberían estar en estado **Detenida**. En esta pagina, puede iniciar una máquina virtual de un alumno, conectarse a la máquina virtual, detener la máquina virtual y eliminar la máquina virtual. Puede iniciarlas en esta página o dejar que los alumnos inicien las máquinas virtuales. 

    ![Máquinas virtuales en estado detenido](./media/tutorial-setup-classroom-lab/virtual-machines-stopped.png)

    En esta página se realizan las siguientes tareas (no siga estos pasos en el tutorial, solo tienen carácter informativo): 
    
    1. Para cambiar la capacidad del laboratorio (número de máquinas virtuales que contiene), seleccione **Lab capacity** (Capacidad del laboratorio) en la barra de herramientas.
    2. Para iniciar todas las máquinas virtuales a la vez, seleccione **Start All** (Iniciar todas) en la barra de herramientas. 
    3. Para iniciar una máquina virtual específica, seleccione la flecha hacia abajo en **Status** (Estado) y **Start** (Iniciar). También puede iniciar una máquina virtual si la selecciona en la primera columna y, a continuación, selecciona **Start** (Iniciar) en la barra de herramientas.                

### <a name="vm-sizes"></a>Tamaños de VM  

| Size | Núcleos | RAM | Descripción | 
| ---- | ----- | --- | ----------- | 
| Pequeña | 2 | 3,5 GB | Este tamaño es el más adecuado para la línea de comandos, apertura de navegador web, servidores web de poco tráfico, bases de datos pequeñas o medianas. |
| Media | 4 | 7 GB | Este tamaño es el más adecuado para bases de datos relacionales, análisis y almacenamiento en caché en memoria | 
| Mediano (virtualización anidada) | 4 | 16 GB | Este tamaño es el más adecuado para bases de datos relacionales, análisis y almacenamiento en caché en memoria. Este tamaño admite la virtualización anidada. <p>Este tamaño puede usarse en escenarios donde cada alumno necesita varias máquinas virtuales. Los formadores pueden usar la virtualización anidada para configurar varias máquinas virtuales anidadas de tamaño pequeño dentro de la máquina virtual. </p> |
| GPU pequeña (proceso) | 6 | 56 GB | <p>Este tamaño es más adecuado para aplicaciones de proceso intensivo y uso intensivo de la red; por ejemplo, aplicaciones de inteligencia artificial y aprendizaje profundo.</p><p>Azure Lab Services instala y configura automáticamente los controladores de GPU necesarios cuando se crea un laboratorio con imágenes de GPU. </p> | 
| GPU pequeña (visualización) | 6 | 56 GB | Este tamaño es más adecuado para visualización remota, streaming, juegos y codificación mediante plataformas como OpenGL y DirectX. | 
| grande | 8 | 16 GB | Este tamaño es ideal para aplicaciones que necesitan CPU más rápidas, un mejor rendimiento de los discos locales, bases de datos grandes y memorias caché grandes. |
| Grande (virtualización anidada) | 8 | 32 GB | Este tamaño es ideal para aplicaciones que necesitan CPU más rápidas, un mejor rendimiento de los discos locales, bases de datos grandes y memorias caché grandes. Este tamaño admite la virtualización anidada. |  
| GPU mediana (visualización) | 12 | 112 GB | Este tamaño es más adecuado para visualización remota, streaming, juegos y codificación mediante plataformas como OpenGL y DirectX. | 

> [!NOTE]
> Es posible que no vea algunos de estos tamaños de VM en la lista al crear un laboratorio de clase. La lista se rellena en función de la capacidad actual de la ubicación del laboratorio. Si el creador de la cuenta de laboratorio [permite que los creadores del laboratorio elijan una ubicación para el laboratorio](allow-lab-creator-pick-lab-location.md), puede intentar elegir otra ubicación para el laboratorio y ver si el tamaño de VM está disponible. 

## <a name="view-all-labs"></a>Visualización de todos los laboratorios

1. Vaya al [portal de Azure Lab Services](https://labs.azure.com).
1. Seleccione **Iniciar sesión**. Seleccione o escriba un **Id. de usuario** que sea miembro del rol **Creador de laboratorio** en la cuenta del laboratorio y escriba la contraseña. Azure Lab Services es compatible con cuentas profesionales y cuentas Microsoft. 

    [!INCLUDE [Select a tenant](./includes/multi-tenant-support.md)]
1. Confirme que ve todos los laboratorios en la cuenta de laboratorio seleccionada. En el icono del laboratorio, verá el número de máquinas virtuales del laboratorio y la cuota asignada a cada usuario (fuera del tiempo programado).

    ![Todos los laboratorios](./media/how-to-manage-classroom-labs/all-labs.png)
1. Use la lista desplegable de la parte superior para seleccionar una cuenta de laboratorio distinta. Verá los laboratorios en la cuenta de laboratorio seleccionada. 

## <a name="delete-a-classroom-lab"></a>Eliminación de un laboratorio de clase

1. En el icono del laboratorio, seleccione los tres puntos (...) de la esquina y, luego, seleccione **Delete** (Eliminar). 

    ![Botón Eliminar](./media/how-to-manage-classroom-labs/delete-button.png)
1. En el cuadro de diálogo **Delete lab** (Eliminar laboratorio), seleccione **Delete** (Eliminar) para continuar con la eliminación. 

## <a name="switch-to-another-classroom-lab"></a>Cambiar a otro laboratorio de clase

Para cambiar a otro laboratorio de clase desde el actual, seleccione la lista desplegable de laboratorios en la cuenta de laboratorio en la parte superior.

![Seleccione el laboratorio de lista desplegable en la parte superior](./media/how-to-manage-classroom-labs/switch-lab.png)

También puede crear un laboratorio con la opción **New lab** (Nuevo laboratorio) de esta lista desplegable. 

> [!NOTE]
> También puede usar el módulo Az.LabServices de PowerShell (versión preliminar) para administrar laboratorios. Para más información, consulte la [página principal de Az.LabServices en GitHub](https://github.com/Azure/azure-devtestlab/tree/master/samples/ClassroomLabs/Modules/Library).

Para cambiar a una cuenta de laboratorio diferente, seleccione la lista desplegable junto a la cuenta de laboratorio y seleccione la otra cuenta de laboratorio. 

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes:

- [Como propietario del laboratorio, configurar y publicar plantillas](how-to-create-manage-template.md)
- [Como propietario del laboratorio, configurar y controlar el uso de un laboratorio](how-to-configure-student-usage.md)
- [Como usuario de laboratorio, obtener acceso a los laboratorios](how-to-use-classroom-lab.md)

