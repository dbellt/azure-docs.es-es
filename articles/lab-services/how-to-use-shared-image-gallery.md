---
title: Usar una galería de imágenes compartidas en Azure Lab Services | Microsoft Docs
description: Aprenda a configurar una cuenta de laboratorio para usar una galería de imágenes compartidas para que un usuario pueda compartir una imagen con otro, y otro usuario pueda usar la imagen para crear una plantilla de máquina virtual en el laboratorio.
ms.topic: article
ms.date: 09/11/2020
ms.openlocfilehash: e4ebc0c865784922cc52d4f9b67c61f27908df55
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111970003"
---
# <a name="use-a-shared-image-gallery-in-azure-lab-services"></a>Usar una galería de imágenes compartidas en Azure Lab Services
En este artículo se muestra cómo los formadores y los administradores de laboratorio pueden guardar una imagen de máquina virtual de plantilla en una [galería de imágenes compartidas](../virtual-machines/shared-image-galleries.md) para que otras personas puedan usarla para crear laboratorios. 

> [!IMPORTANT]
> Al usar una instancia de Shared Image Gallery, Azure Lab Services solo admite imágenes con menos de 128 GB de espacio en disco del sistema operativo. Las imágenes con más de 128 GB de espacio en disco o varios discos no se mostrarán en la lista de imágenes de máquina virtual durante la creación del laboratorio.

## <a name="scenarios"></a>Escenarios
Estos son los pares de escenarios compatibles con esta característica: 

- Un administrador de la cuenta de laboratorio adjunta una galería de imágenes compartidas a la cuenta de laboratorio y carga una imagen en la galería de imágenes compartidas fuera del contexto de un laboratorio. A continuación, los creadores del laboratorio pueden usar esa imagen de la galería de imágenes compartidas para crear laboratorios. 
- El administrador de una cuenta de laboratorio adjunta una galería de imágenes compartidas a la cuenta de laboratorio. Un creador del laboratorio (instructor) guarda la imagen personalizada de su laboratorio en la galería de imágenes compartidas. A continuación, otros creadores del laboratorio pueden seleccionar esta imagen de la galería de imágenes compartidas para crear una plantilla para sus laboratorios. 

    Cuando una imagen se guarda en una galería de imágenes compartida, Azure Lab Services replica la imagen guardada en otras regiones disponibles en la misma [ubicación geográfica](https://azure.microsoft.com/global-infrastructure/geographies/). Ello garantiza que la imagen está disponible para los laboratorios creados en otras regiones de la misma ubicación geográfica. Guardar las imágenes en una galería de imágenes compartida conlleva un costo adicional, lo que incluye el costo de todas las imágenes replicadas. Este costo es independiente del costo de uso de Azure Lab Services. Para más información sobre los precios de Shared Image Gallery, vea [Introducción a la galería de imágenes compartidas - Facturación](../virtual-machines/shared-image-galleries.md#billing).
    
## <a name="prerequisites"></a>Requisitos previos
- Tener creada una galería de imágenes compartidas con [Azure PowerShell](../virtual-machines/shared-images-powershell.md) o con la [CLI de Azure](../virtual-machines/shared-images-cli.md).
- Haber asociado la galería de imágenes compartidas a la cuenta de laboratorio. Para obtener instrucciones detalladas, vea [Asociar o desasociar una galería de imágenes compartidas en Azure Lab Services](how-to-attach-detach-shared-image-gallery.md).


## <a name="save-an-image-to-the-shared-image-gallery"></a>Guardar una imagen en la galería de imágenes compartidas
Una vez que se ha asociado una galería de imágenes compartidas, un administrador de cuenta de laboratorio o un formador puede guardar una imagen en la galería de imágenes compartidas para que otros formaodres la puedan reutilizar. 

1. En la página **Template** (Plantilla) del laboratorio, seleccione **Export to Shared Image Gallery** (Exportar a Shared Image Gallery) en la barra de herramientas.

    ![Botón Save image (Guardar imagen)](./media/how-to-use-shared-image-gallery/export-to-shared-image-gallery-button.png)
2. En el cuadro de diálogo **Export to Shared Image Gallery** (Exportar a Shared Image Gallery), escriba un **nombre para la imagen** y seleccione **Export** (Exportar). 

    ![Cuadro de diálogo Export to Shared Image Gallery (Exportar a Shared Image Gallery)](./media/how-to-use-shared-image-gallery/export-to-shared-image-gallery-dialog.png)

3. Puede ver el progreso de esta operación en la página **Template** (Plantilla). Esta operación puede tardar algún tiempo. 

    ![Exportación en curso](./media/how-to-use-shared-image-gallery/exporting-image-in-progress.png)
4. Cuando la exportación se haya realizado correctamente, verá el mensaje siguiente:

    ![Exportación finalizada](./media/how-to-use-shared-image-gallery/exporting-image-completed.png)

    Después de guardar la imagen en la galería de imágenes compartidas, puede usar esa imagen de la galería al crear otro laboratorio. También puede cargar una imagen en la galería de imágenes compartidas fuera del contexto de un laboratorio. Para más información, consulte:

    - [Información general de Shared Image Gallery](../virtual-machines/shared-images-powershell.md)
    - [Carga de una imagen personalizada en Shared Image Gallery](upload-custom-image-shared-image-gallery.md)

    > [!IMPORTANT]
    > Cuando [guarda una imagen de plantilla de un laboratorio](how-to-use-shared-image-gallery.md#save-an-image-to-the-shared-image-gallery) de Azure Lab Services en una galería de imágenes compartidas, la imagen se carga en la galería como una **imagen especializada**. Las [imágenes especializadas](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images) conservan la información específica de la máquina y los perfiles de usuario. Todavía puede cargar directamente una imagen generalizada en la galería fuera de Azure Lab Services.    

## <a name="use-a-custom-image-from-the-shared-image-gallery"></a>Usar una imagen personalizada de Shared Image Gallery
Un formador puede elegir una imagen personalizada disponible en Shared Image Gallery para la plantilla de máquina virtual que se crea al configurar un nuevo laboratorio.

![Usar una imagen de máquina virtual de la galería](./media/how-to-use-shared-image-gallery/use-shared-image.png)

> [!NOTE]
> Puede crear una plantilla de máquina virtual basada en imágenes **generalizadas** y **especializadas** en Azure Lab Services.

### <a name="resave-a-custom-image-to-shared-image-gallery"></a>Volver a guardar una imagen personalizada en Shared Image Gallery

Después de haber creado un laboratorio a partir de una imagen personalizada de una instancia de Shared Image Gallery, puede realizar cambios en la imagen mediante la plantilla de máquina virtual y volver a exportar la imagen a Shared Image Gallery.  Al volver a exportar, tiene la opción de crear una nueva imagen o actualizar la imagen original. 

 ![Cuadro de diálogo Reexport to Shared Image Gallery (Volver a exportar a Shared Image Gallery)](./media/how-to-use-shared-image-gallery/reexport-to-shared-image-gallery-dialog.png) 

Si elige **Crear nueva imagen**, se crea una nueva [definición de imagen](../virtual-machines/shared-image-galleries.md#image-definitions).  Esto le permite guardar una imagen personalizada completamente nueva sin cambiar la imagen personalizada original que ya existe en Shared Image Gallery.

Si en su lugar elige **Actualizar imagen existente**, la definición de la imagen personalizada original se actualiza con una nueva [versión](../virtual-machines/shared-image-galleries.md#image-versions).  Lab Services usará automáticamente la versión más reciente la próxima vez que se cree un laboratorio mediante la imagen personalizada.

## <a name="next-steps"></a>Pasos siguientes
Para obtener información sobre cómo configurar Shared Image Gallery asociándola y desasociándola a una cuenta de laboratorio, consulte el artículo sobre [cómo asociar y desasociar Shared Image Gallery](how-to-attach-detach-shared-image-gallery.md).

Para llevar una imagen personalizada de Windows a Shared Image Gallery fuera del contexto de un laboratorio, consulte [Llevar una imagen personalizada a Shared Image Gallery](upload-custom-image-shared-image-gallery.md).

Para obtener más información sobre las galerías de imágenes compartidas en general, vea [Galería de imágenes compartidas](../virtual-machines/shared-image-galleries.md).