---
title: Configuración de un laboratorio con Adobe Creative Cloud mediante Azure Lab Services | Microsoft Docs
description: Aprenda a configurar un laboratorio para las clases de artes digitales y multimedia que usan Adobe Creative Cloud.
author: nicolela
ms.topic: article
ms.date: 04/21/2021
ms.author: nicolela
ms.openlocfilehash: 210fa647f12a786e357902bef3bbb3b20cca8077
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111962649"
---
# <a name="set-up-a-lab-for-adobe-creative-cloud"></a>Configuración de un laboratorio para Adobe Creative Cloud
[Adobe Creative Cloud](https://www.adobe.com/creativecloud.html) es una colección de aplicaciones de escritorio y servicios web que se usan en la fotografía, el diseño, el vídeo, la web y la experiencia de usuario (UX) y muchas otras disciplinas.  Universidades y colegios de educación primaria y secundaria usan Creative Cloud en las clases de artes digitales y multimedia.  Algunos de los procesos multimedia de Creative Cloud pueden requerir una mayor potencia de cálculo y visualización (GPU) que la que ofrece una tableta, un portátil o una estación de trabajo normales.  Con Azure Lab Services, tiene flexibilidad para elegir entre varios tamaños de máquina virtual (VM), incluidos los tamaños de GPU.

En este artículo, le mostraremos cómo configurar una clase que usa Creative Cloud.

## <a name="licensing"></a>Licencias
Para usar Creative Cloud en una máquina virtual de laboratorio, debe utilizar [licencias de usuario con nombre](https://helpx.adobe.com/enterprise/kb/technical-support-boundaries-virtualized-server-based.html#main_Licensing_considerations), que es el único tipo de licencia que admite la implementación en una máquina virtual.  Cada máquina virtual de laboratorio tiene acceso a Internet para que los alumnos puedan activar las aplicaciones de Creative Cloud mediante el inicio de sesión en el software.  Una vez que un alumno inicia sesión, su token de autenticación se almacena en caché en el perfil de usuario para que no tenga que volver a iniciar sesión en su máquina virtual.  Para más información, lea el [artículo de Adobe sobre licencias](https://helpx.adobe.com/enterprise/using/licensing.html).

## <a name="lab-configuration"></a>Configuración del laboratorio
Para configurar este laboratorio, para empezar, necesita una suscripción a Azure y una cuenta de laboratorio. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar. Una vez que obtenga una suscripción a Azure, puede crear una nueva cuenta de laboratorio en Azure Lab Services. Para más información sobre la creación de una cuenta de laboratorio, consulte el tutorial sobre la [configuración de una cuenta de laboratorio](./tutorial-setup-lab-account.md). También puede usar una cuenta de laboratorio existente.

### <a name="lab-account-settings"></a>Configuración de la cuenta de laboratorio

Habilite la configuración que se describe en la tabla siguiente para la cuenta de laboratorio. Para obtener más información sobre cómo habilitar imágenes de marketplace, consulte el artículo sobre la [especificación de las imágenes de Marketplace disponibles para los creadores de laboratorios](./specify-marketplace-images.md).

| Configuración de la cuenta de laboratorio | Instructions |
| ------------------- | ------------ |
|Imagen de Marketplace| Habilite la imagen de Windows 10 para usarla en la cuenta de laboratorio.|

### <a name="lab-settings"></a>Configuración del laboratorio

El tamaño de la máquina virtual que debe usar para el laboratorio depende de los tipos de proyectos que vayan a crear los alumnos.  La mayoría de las [aplicaciones de Creative Cloud](https://helpx.adobe.com/creative-cloud/system-requirements.html) admiten la aceleración basada en GPU y requieren una GPU para que las características funcionen correctamente.  Para seleccionar el tamaño de máquina virtual adecuado, se recomienda probar los proyectos que vayan a crear los alumnos para garantizar un rendimiento adecuado.  En la tabla siguiente se muestra el [tamaño de máquina virtual](./administrator-guide.md#vm-sizing) recomendado para usar con Creative Cloud.  

| Configuración del laboratorio | Valor/instrucciones |
| ------------ | ------------------ |
|Tamaño de la máquina virtual| **GPU pequeña (visualización)** .  Esta máquina virtual es más adecuada para visualización remota, streaming, juegos y codificación mediante plataformas como OpenGL y DirectX.|  
|Imagen de máquina virtual| Windows 10 |

> [!NOTE]
> El tamaño de máquina virtual de **GPU pequeña (visualización)** está configurado para permitir una experiencia gráfica de alto rendimiento y cumple los [requisitos del sistema de Adobe de cada aplicación](https://helpx.adobe.com/creative-cloud/system-requirements.html).  Asegúrese de elegir la opción de GPU pequeña (visualización) y no GPU pequeña (proceso).  Para obtener más información sobre este tamaño de máquina virtual, consulte el artículo sobre la [configuración de un laboratorio con GPU](./how-to-setup-lab-gpu.md).

## <a name="template-virtual-machine-configuration"></a>Configuración de plantilla de máquina virtual

### <a name="creative-cloud-deployment-package"></a>Paquete de implementación de Creative Cloud
La instalación de Creative Cloud requiere el uso de un paquete de implementación. Normalmente, el departamento de TI crea este paquete mediante la consola de administración de Adobe.  Cuando TI crea el paquete de implementación, también tienen la opción de habilitar el autoservicio.  Hay varias maneras de habilitar el autoservicio en el paquete de implementación:
-    Crear un paquete de autoservicio
-    Crear un paquete administrado con privilegios elevados de autoservicio activados

Con el autoservicio habilitado, no se instala la colección completa de aplicaciones de Creative Cloud.  Al contrario, los alumnos pueden instalar las aplicaciones por su cuenta mediante la aplicación de escritorio de Creative Cloud.  Estas son algunas ventajas clave de este enfoque:
- La instalación entera de Creative Cloud ocupa unos 25 GB.  Que los alumnos instalen solo las aplicaciones que necesitan a la carta les ayuda a optimizar el espacio en disco. Las máquinas virtuales de laboratorio tienen un tamaño de disco de 128 GB.
- Puede optar por instalar un subconjunto de las aplicaciones en la máquina virtual de plantilla antes de publicarla.  De este modo, las máquinas virtuales de los alumnos tendrán algunas aplicaciones instaladas de forma predeterminada y los alumnos pueden agregar más aplicaciones por su cuenta cuando sea necesario.
- Puede evitar volver a publicar la máquina virtual de plantilla porque los alumnos pueden instalar aplicaciones adicionales en su máquina virtual en cualquier momento durante la vigencia del laboratorio.  De lo contrario, el responsable de TI o el profesor tendrían que instalar aplicaciones adicionales en la máquina virtual de plantilla y volver a publicarla.  Como consecuencia, las máquinas virtuales de los alumnos se restablecen y se pierden los trabajos que no se hayan guardado externamente.

Si usa un paquete de implementación administrado con el autoservicio deshabilitado, los alumnos no tendrán la posibilidad de instalar sus propias aplicaciones.  En este caso, el departamento de TI debe especificar las aplicaciones de Creative Cloud que se instalarán.

Para más información, lea los [pasos de Adobe para crear un paquete](https://helpx.adobe.com/enterprise/admin-guide.html/enterprise/using/create-nul-packages.ug.html).

### <a name="install-creative-cloud"></a>Instalación de Creative Cloud
Después de crear la máquina de plantilla, siga los pasos que se indican a continuación para configurar la máquina virtual (VM) de plantilla del laboratorio con Creative Cloud.
1. Inicie la máquina virtual de plantilla y conéctese con RDP.
1. Para instalar Creative Cloud, descargue el paquete de implementación que le ha dado el equipo de TI o directamente desde la [consola de administración de Adobe](https://adminconsole.adobe.com/).
1. Ejecute el archivo del paquete de implementación.  En función de si el autoservicio está habilitado o deshabilitado, se instalará la aplicación de escritorio de Creative Cloud o las aplicaciones de Creative Cloud especificadas.
Para más información, lea los [pasos de implementación de Adobe](https://helpx.adobe.com/enterprise/admin-guide.html/enterprise/using/deploy-packages.ug.html).
1. Una vez configurada la máquina virtual de plantilla, [publique la imagen de la máquina virtual de plantilla](how-to-create-manage-template.md) que se usa para crear todas las máquinas virtuales de los alumnos en el laboratorio.

### <a name="storage"></a>Storage
Como se mencionó anteriormente, las máquinas virtuales de laboratorio de Azure tienen un tamaño de disco de 128 GB.  Si los alumnos necesitan almacenamiento adicional para guardar recursos multimedia grandes o necesitan acceder a recursos multimedia compartidos, se debe considerar el uso de almacenamiento de archivos externo.  Para más información, consulte los siguientes artículos:
-    [Uso de almacenamiento de archivos externo en servicios de laboratorio](how-to-attach-external-storage.md)
-    [Instalación y configuración de OneDrive](./how-to-prepare-windows-template.md#install-and-configure-onedrive)

### <a name="save-template-vm-image"></a>Guardar la imagen de la máquina virtual de plantilla
Considere la posibilidad de guardar la máquina virtual de plantilla para usarla más adelante.  Para guardar la máquina virtual de plantilla, consulte [Guardar una imagen en Shared Image Gallery](./how-to-use-shared-image-gallery.md#save-an-image-to-the-shared-image-gallery).
- Cuando el autoservicio esté *habilitado*, la imagen de la máquina virtual de plantilla tendrá instalado el escritorio de Creative Cloud.  Luego, los profesores pueden reutilizar esta imagen para crear laboratorios y elegir qué aplicaciones de Creative Cloud van a instalar.  De esta forma, se reduce la sobrecarga sobre el departamento de TI, ya que los profesores pueden configurar laboratorios de forma independiente y tener control total sobre la instalación de las aplicaciones de Creative Cloud que necesiten para sus clases.
- Cuando el autoservicio esté *deshabilitado*, la imagen de la máquina virtual de plantilla ya tendrá instaladas las aplicaciones de Creative Cloud especificadas.  Los profesores pueden reutilizar esta imagen para crear laboratorios, sin embargo, no podrán instalar aplicaciones de Creative Cloud adicionales.

## <a name="cost"></a>Coste

En esta sección, se examinará una posible estimación de costos de esta clase.  Se usará una clase de 25 alumnos con 20 horas de clase programadas.  Además, cada alumno obtiene una cuota de 10 horas para deberes o tareas fuera del tiempo de clase programado. El tamaño de la máquina virtual que hemos elegido es el de **GPU pequeña (visualización)** , que tiene 160 unidades de laboratorio.

25 alumnos \* (20 horas programadas + 10 horas de cuota) \* 160 unidades de laboratorio * 0,01 USD por hora = 1200,00 USD

>[!IMPORTANT]
> La estimación de costos solo se utiliza con fines de ejemplo.  Para conocer los detalles actuales sobre los precios, consulte [Precios de Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).  

## <a name="next-steps"></a>Pasos siguientes

Los pasos siguientes son comunes a la configuración de cualquier laboratorio.

- [Creación y administración de plantillas](how-to-create-manage-template.md)
- [Incorporación de usuarios](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Establecimiento de la cuota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Establecimiento de la programación](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Vínculos de registro por correo electrónico para los alumnos](how-to-configure-student-usage.md#send-invitations-to-users)