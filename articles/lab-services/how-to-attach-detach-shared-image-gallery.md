---
title: Asociación o desasociación de una galería de imágenes compartidas en Azure Lab Services | Microsoft Docs
description: En este artículo se describe cómo asociar una galería de imágenes compartidas a un aula laboratorio en Azure Lab Services.
ms.topic: article
ms.date: 09/11/2020
ms.openlocfilehash: 2a460e25d59d2c5fcde341212e7912882f5fe111
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110190696"
---
# <a name="attach-or-detach-a-shared-image-gallery-in-azure-lab-services"></a>Asociación o desasociación de una galería de imágenes compartidas en Azure Lab Services
En este artículo se muestra cómo asociar o desasociar una galería de imágenes compartidas en una cuenta de laboratorio. 

> [!NOTE]
> Cuando [guarda una imagen de plantilla de un laboratorio](how-to-use-shared-image-gallery.md#save-an-image-to-the-shared-image-gallery) de Azure Lab Services en una galería de imágenes compartidas, la imagen se carga en la galería como una imagen especializada. Las [imágenes especializadas](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images) conservan la información específica de la máquina y los perfiles de usuario. Todavía puede cargar directamente una imagen generalizada en la galería fuera de Azure Lab Services. 
>
> Un creador de laboratorio puede crear una plantilla de máquina virtual basada en imágenes generalizadas y especializadas en Azure Lab Services. 

## <a name="scenarios"></a>Escenarios
Estos son los pares de escenarios compatibles con esta característica: 

- Un administrador de la cuenta de laboratorio adjunta una galería de imágenes compartidas a la cuenta de laboratorio y carga una imagen en la galería de imágenes compartidas fuera del contexto de un laboratorio. A continuación, los creadores del laboratorio pueden usar esa imagen de la galería de imágenes compartidas para crear laboratorios. 
- El administrador de una cuenta de laboratorio adjunta una galería de imágenes compartidas a la cuenta de laboratorio. Un creador del laboratorio (instructor) guarda la imagen personalizada de su laboratorio en la galería de imágenes compartidas. A continuación, otros creadores del laboratorio pueden seleccionar esta imagen de la galería de imágenes compartidas para crear una plantilla para sus laboratorios. 

    Cuando una imagen se guarda en una galería de imágenes compartida, Azure Lab Services replica la imagen guardada en otras regiones disponibles en la misma [ubicación geográfica](https://azure.microsoft.com/global-infrastructure/geographies/). Ello garantiza que la imagen está disponible para los laboratorios creados en otras regiones de la misma ubicación geográfica. Guardar las imágenes en una galería de imágenes compartida conlleva un costo adicional, lo que incluye el costo de todas las imágenes replicadas. Este costo es independiente del costo de uso de Azure Lab Services. Para más información sobre los precios de Shared Image Gallery, vea [Introducción a la galería de imágenes compartidas - Facturación](../virtual-machines/shared-image-galleries.md#billing).

> [!IMPORTANT]
> Al usar una instancia de Shared Image Gallery, Azure Lab Services solo admite imágenes con menos de 128 GB de espacio en disco del sistema operativo. Las imágenes con más de 128 GB de espacio en disco o varios discos no se mostrarán en la lista de imágenes de máquina virtual durante la creación del laboratorio.

## <a name="configure-at-the-time-of-lab-account-creation"></a>Configuración al crear la cuenta de laboratorio
Al crear una cuenta de laboratorio, puede asociarle una galería de imágenes compartidas. Puede seleccionar una galería de imágenes compartidas existente en la lista desplegable o bien crear una. Para crear y adjuntar una galería de imágenes compartidas a la cuenta de laboratorio, seleccione **Crear nuevo**, escriba un nombre para la galería y seleccione **Aceptar**. 

![Configuración de la Galería de imágenes compartidas en el momento de crear la cuenta de laboratorio](./media/how-to-use-shared-image-gallery/new-lab-account.png)

## <a name="configure-after-the-lab-account-is-created"></a>Configuración después de crear la cuenta de laboratorio
Una vez que se ha creado la cuenta de laboratorio, puede realizar las tareas siguientes:

- Creación y asociación de una galería de imágenes compartidas
- Asociar una galería de imágenes compartidas a la cuenta de laboratorio
- Desasociar una galería de imágenes compartidas de la cuenta de laboratorio

## <a name="create-and-attach-a-shared-image-gallery"></a>Creación y asociación de una galería de imágenes compartidas
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los servicios** en el menú de la izquierda. Seleccione **Lab Services** en la sección **DEVOPS**. Si selecciona la estrella (`*`) junto a **Lab Services**, se agrega a la sección **FAVORITOS** en el menú de la izquierda. A partir de la próxima vez, seleccione **Lab Services** en **FAVORITOS**.

    ![Todos los servicios -> Lab Services](./media/tutorial-setup-lab-account/select-lab-accounts-service.png)
3. Seleccione la cuenta de laboratorio para ver la página **Cuenta de laboratorio**. 
4. Seleccione **Galería de imágenes compartidas** en el menú de la izquierda y **+ Crear** en la barra de herramientas.  

    ![Botón para crear la galería de imágenes compartidas](./media/how-to-use-shared-image-gallery/new-shared-image-gallery-button.png)
5. En la ventana **Crear galería de imágenes compartidas**, escriba un **nombre** para la galería y seleccione **Aceptar**. 

    ![Ventana Crear galería de imágenes compartidas](./media/how-to-use-shared-image-gallery/create-shared-image-gallery-window.png)

    Azure Lab Services crea la galería de imágenes compartidas y la asocia a la cuenta de laboratorio. Todos los laboratorios creados en esta cuenta de laboratorio tienen acceso a la galería de imágenes compartidas adjunta. 

    ![Galería de imágenes adjunta](./media/how-to-use-shared-image-gallery/image-gallery-in-list.png)

    En el panel inferior, verá imágenes en la galería de imágenes compartidas. En esta galería nueva no hay ninguna imagen. Al cargar imágenes a la galería, las verá en esta página.     

    Todas las imágenes de la galería de imágenes compartidas adjunta están habilitadas de forma predeterminada. Puede habilitar o deshabilitar imágenes concretas si las selecciona en la lista y usa el botón **Enable selected images** (Habilitar imágenes seleccionadas) o **Disable selected images** (Deshabilitar imágenes seleccionadas).

## <a name="attach-an-existing-shared-image-gallery"></a>Asociación de una galería de imágenes compartidas existente
En el procedimiento siguiente se muestra cómo asociar una galería de imágenes compartidas existente a una cuenta de laboratorio. 

1. En la página **Cuenta de laboratorio**, seleccione **Galería de imágenes compartidas** en el menú de la izquierda y **Asociar** en la barra de herramientas. 

    ![Galería de imágenes compartidas: botón Agregar](./media/how-to-use-shared-image-gallery/sig-attach-button.png)
5. En la página **Asociar una galería de imágenes compartidas existente**, seleccione la galería de imágenes compartidas y después **Aceptar**.

    ![Selección de una galería existente](./media/how-to-use-shared-image-gallery/select-image-gallery.png)
6. Verá la pantalla siguiente: 

    ![Mi galería en la lista](./media/how-to-use-shared-image-gallery/my-gallery-in-list.png)
    
    En este ejemplo, todavía no hay ninguna imagen en la galería de imágenes compartidas.

    La identidad de Azure Lab Services se agrega como colaborador a la galería de imágenes compartidas asociada al laboratorio. Permite a formadores y administradores de TI guardar imágenes de máquina virtual en la galería de imágenes compartidas. Todos los laboratorios creados en esta cuenta de laboratorio tienen acceso a la galería de imágenes compartidas adjunta. 

    Todas las imágenes de la galería de imágenes compartidas adjunta están habilitadas de forma predeterminada. Puede habilitar o deshabilitar imágenes concretas si las selecciona en la lista y usa el botón **Enable selected images** (Habilitar imágenes seleccionadas) o **Disable selected images** (Deshabilitar imágenes seleccionadas). 

## <a name="detach-a-shared-image-gallery"></a>Desasociación de una galería de imágenes compartidas
Solo se puede asociar una galería de imágenes compartidas a un laboratorio. Si quiere asociar otra galería de imágenes compartidas, desasocie la actual antes de asociar la nueva. Para desasociar una galería de imágenes compartidas del laboratorio, seleccione **Desasociar** en la barra de herramientas y confirme la operación de desasociación. 

![Desasociación de la galería de imágenes compartidas de la cuenta de laboratorio](./media/how-to-use-shared-image-gallery/detach.png)

## <a name="next-steps"></a>Pasos siguientes
Para obtener información sobre cómo guardar una imagen de laboratorio en la galería de imágenes compartidas o cómo usar una imagen de la galería para crear una máquina virtual, vea [Procedimientos para usar la galería de imágenes compartidas](how-to-use-shared-image-gallery.md).

Para llevar una imagen personalizada de Windows a la galería de imágenes compartidas fuera del contexto de un laboratorio, consulte [Transporte de una imagen personalizada de Windows a la galería de imágenes compartidas](upload-custom-image-shared-image-gallery.md).

Para obtener más información sobre las galerías de imágenes compartidas en general, vea [Galería de imágenes compartidas](../virtual-machines/shared-image-galleries.md).