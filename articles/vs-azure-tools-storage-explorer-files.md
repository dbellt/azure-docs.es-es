---
title: Uso del Explorador de Storage con Azure Files | Microsoft Docs
description: Aprenda a usar el Explorador de Storage para trabajar con archivos y recursos compartidos de archivos.
services: storage
documentationcenter: na
author: cawaMS
manager: paulyuk
editor: ''
ms.assetid: ''
ms.service: storage
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/09/2017
ms.author: cawa
ms.openlocfilehash: b0387aaecd45a2f09acc8d9ad7af230a62da6ed1
ms.sourcegitcommit: c1b0d0b61ef7635d008954a0d247a2c94c1a876f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109628507"
---
# <a name="using-storage-explorer-with-azure-files"></a>Uso del Explorador de Storage con Azure Files

Azure Files es un servicio que ofrece recursos compartidos de archivos en la nube mediante el protocolo Bloque de mensajes del servidor (SMB) estándar. Se admiten SMB 2.1 y SMB 3.0. Con Azure Files, puede migrar aplicaciones heredadas basadas en recursos compartidos de archivos a Azure con rapidez y sin necesidad de costosas reescrituras. File Storage se puede usar para exponer datos públicamente o para almacenar los datos de la aplicación de manera privada. En este artículo, aprenderá a usar el Explorador de Storage (versión preliminar) para trabajar con recursos compartidos de archivos y archivos.

## <a name="prerequisites"></a>Prerequisites

Para completar los pasos de este artículo, necesitará:

- [Descarga e instalación del Explorador de Storage](https://www.storageexplorer.com/)

- [Conectarse a una cuenta o a un servicio de almacenamiento de Azure](./vs-azure-tools-storage-manage-with-storage-explorer.md#connect-to-a-storage-account-or-service)

## <a name="create-a-file-share"></a>Creación de un recurso compartido de archivos

Todos los archivos deben residir en un recurso compartido de archivos, que no es más que una agrupación lógica de archivos. Una cuenta puede contener un número ilimitado de recursos compartidos de archivos y cada uno de ellos puede almacenar un número ilimitado de archivos.

Los siguientes pasos muestran cómo crear un recurso compartido de archivos en el Explorador de Storage.

1. Abra el Explorador de Storage.

1. En el panel izquierdo, expanda la cuenta de almacenamiento en la que desea crear el recurso compartido de archivos

1. Haga clic con el botón derecho en **Recursos compartidos de archivos** y (en el menú contextual) seleccione **Create File Share** (Crear recurso compartido de archivos).

    ![Creación de un recurso compartido de archivos](media/vs-azure-tools-storage-explorer-files/image1.png)

1. Aparecerá un cuadro de texto debajo de la carpeta **Recursos compartidos de archivos**. Escriba el nombre del recurso compartido de archivos. Para ver una lista de reglas y restricciones en la nomenclatura de recursos compartidos de archivos, consulte la sección [Reglas de nomenclatura de recursos compartidos](./storage/blobs/storage-quickstart-blobs-dotnet.md).

    ![Nomenclatura de recurso compartido](media/vs-azure-tools-storage-explorer-files/image2.png)

1. Presione **Entrar** cuando termine para crear el recurso compartido de archivos o **Esc** para cancelar la operación. Una vez que el recurso compartido de archivos se haya creado correctamente, se mostrará en la carpeta **Recursos compartidos de archivos** de la cuenta de almacenamiento seleccionada.

    ![El nuevo recurso compartido](media/vs-azure-tools-storage-explorer-files/image3.png)

## <a name="view-a-file-shares-contents"></a>Visualización del contenido de un recurso compartido de archivos

Los recursos compartidos de archivos contienen archivos y carpetas (que también pueden contener archivos).

Los siguientes pasos muestran cómo ver el contenido de recurso compartido de archivos en el Explorador de Storage:+

1. Abra el Explorador de Storage.

1. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el recurso compartido de archivos que desea ver.

1. Expanda la opción **Recursos compartidos de archivos** de la cuenta de almacenamiento.

1. Haga clic con el botón derecho en el recurso compartido de archivos que desea ver y (en el menú contextual) seleccione **Abrir**. También puede hacer doble clic en el recurso compartido de archivos que desea ver.

    ![Abrir recurso compartido](media/vs-azure-tools-storage-explorer-files/image4.png)

1. El panel principal mostrará el contenido del recurso compartido de archivos.
    
    ![Captura de pantalla del panel principal de un recurso compartido de archivos en el Explorador de Storage, que muestra el contenido del recurso compartido.](media/vs-azure-tools-storage-explorer-files/image5.png)

## <a name="delete-a-file-share"></a>Eliminación de un recurso compartido de archivos

Los recursos compartidos de archivos se pueden crear y eliminar fácilmente si fuera necesario. (para ver cómo eliminar los archivos individuales, consulte la sección [Administración de archivos en un recurso compartido de archivos](./vs-azure-tools-storage-explorer-blobs.md#managing-blobs-in-a-blob-container)).

Los siguientes pasos muestran cómo eliminar un recurso compartido de archivos en el Explorador de Storage:

1. Abra el Explorador de Storage.

1. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el recurso compartido de archivos que desea ver.

1. Expanda la opción **Recursos compartidos de archivos** de la cuenta de almacenamiento.

1. Haga clic con el botón derecho en el recurso compartido de archivos que desea eliminar y (en el menú contextual) seleccione **Delete** (Eliminar). También puede presionar **Suprimir** para eliminar el recurso compartido de archivos actualmente seleccionado.

    ![Eliminar](media/vs-azure-tools-storage-explorer-files/image6.png)

1. Haga clic en **Sí** en el cuadro de diálogo de confirmación.
    
    ![Cuadro de diálogo Confirmación](media/vs-azure-tools-storage-explorer-files/image7.png)

## <a name="copy-a-file-share"></a>Copia de un recurso compartido de archivos

El Explorador de Storage permite copiar un recurso compartido de archivos en el Portapapeles y, después, pegarlo en otra cuenta de almacenamiento. (para ver cómo copiar los archivos individuales, consulte la sección [Administración de archivos en un recurso compartido de archivos](./vs-azure-tools-storage-explorer-blobs.md#managing-blobs-in-a-blob-container)).

Los siguientes pasos muestran cómo copiar un recurso compartido de archivos de una cuenta de almacenamiento a otra.

1. Abra el Explorador de Storage.

1. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el recurso compartido de archivos que desea copiar.

1. Expanda la opción **Recursos compartidos de archivos** de la cuenta de almacenamiento.

1. Haga clic con el botón derecho en el recurso compartido de archivos que desea copiar y (en el menú contextual) seleccione **Copy File Share** (Copar recurso compartido de archivos).

    ![Copiar recurso compartido de archivos](media/vs-azure-tools-storage-explorer-files/image8.png)

1. Haga clic con el botón derecho en la cuenta de almacenamiento de "destino" deseada en la que desea pegar el recurso compartido de archivos y (en el menú contextual) seleccione **Paste File Share**(Pegar recurso compartido de archivos).

    ![Pegar recurso compartido de archivos](media/vs-azure-tools-storage-explorer-files/image9.png)

## <a name="get-the-sas-for-a-file-share"></a>Obtención de la SAS de un recurso compartido de archivos

Una [firma de acceso compartido (SAS)](./storage/common/storage-sas-overview.md) ofrece acceso delegado a los recursos en la cuenta de almacenamiento. Esto significa que puede conceder permisos limitados de los clientes a objetos en su cuenta de almacenamiento durante un período específico y con un conjunto determinado de permisos sin tener que compartir las claves de acceso a las cuentas.

Los siguientes pasos muestran cómo crear una SAS para un recurso compartido de archivos:+

1. Abra el Explorador de Storage.

1. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el recurso compartido de archivos del que desea obtener una SAS.

1. Expanda la opción **Recursos compartidos de archivos** de la cuenta de almacenamiento.

1. Haga clic con el botón derecho en el recurso compartido de archivos y (en el menú contextual) seleccione **Get Shared Access Signature**(Obtener firma de acceso compartido).

    ![Obtener firma de acceso compartido](media/vs-azure-tools-storage-explorer-files/image10.png)

1. En el cuadro de diálogo **Firma de acceso compartido** , especifique la directiva, las fechas de inicio y vencimiento, la zona horaria y los niveles de acceso que desea para el recurso.

    ![Cuadro de diálogo SAS](media/vs-azure-tools-storage-explorer-files/image11.png)

1. Cuando haya terminado de especificar las opciones de SAS, seleccione **Crear**.

1. Después, un segundo cuadro de diálogo **Firma de acceso compartido** mostrará el recurso compartido de archivos junto con la dirección URL y las QueryStrings que se pueden utilizar para acceder al recurso de almacenamiento. Seleccione **Copiar** junto a la dirección URL que desea copiar al Portapapeles.
    
    ![Segundo cuadro de diálogo SAS](media/vs-azure-tools-storage-explorer-files/image12.png)

1. Cuando haya terminado, seleccione **Cerrar**.

## <a name="manage-access-policies-for-a-file-share"></a>Administración de directivas de acceso para un recurso compartido de archivos

Los siguientes pasos muestran cómo administrar (agregar y quitar) las directivas de acceso de un recurso compartido de archivos:+. Las directivas de acceso se utilizan para crear las direcciones URL de SAS que los usuarios pueden utilizar para acceder al recurso de Azure Files durante un período definido.

1. Abra el Explorador de Storage.

1. En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el recurso compartido de archivos cuyas directivas de acceso desea administrar.

1. Expanda la opción **Recursos compartidos de archivos** de la cuenta de almacenamiento.

1. Seleccione el recurso compartido de archivos deseado y (en el menú contextual) seleccione **Manage Access Policies**(Administrar directivas de acceso).

    ![Menú contextual Administrar directivas de acceso](media/vs-azure-tools-storage-explorer-files/image13.png)

1. El cuadro de diálogo **Directivas de acceso** enumerará las directivas de acceso creadas para el recurso compartido de archivos seleccionado.
    
    ![Directivas de acceso](media/vs-azure-tools-storage-explorer-files/image14.png)

1. Siga estos pasos en función de la tarea de administración de directivas de acceso:
    
    - **Agregar una nueva directiva de acceso**: seleccione **Agregar**. Una vez generada, el cuadro de diálogo **Directivas de acceso** mostrará la directiva de acceso recién agregada (con la configuración predeterminada).

    - **Editar una directiva de acceso**: realice las modificaciones que desee y seleccione **Guardar**.

    - **Quitar una directiva de acceso**: seleccione **Quitar** junto a la directiva de acceso que desea quitar.

1. Cree una nueva dirección URL de SAS mediante la directiva de acceso que creó anteriormente:
    
    ![Obtener SAS](media/vs-azure-tools-storage-explorer-files/image15.png)
    
    ![Nombre y propiedades de SAS](media/vs-azure-tools-storage-explorer-files/image16.png)

## <a name="managing-files-in-a-file-share"></a>Administración de archivos en un recurso compartido de archivos

Una vez que haya creado un recurso compartido de archivos, puede cargar un archivo en él, descargar un archivo en el equipo local, abrir un archivo en el equipo local, etc.

Los siguientes pasos muestran cómo administrar los archivos (y carpetas) en un recurso compartido de archivos.

1.  Abra el Explorador de Storage.

1.  En el panel izquierdo, expanda la cuenta de almacenamiento que contiene el recurso compartido de archivos que desea administrar.

1.  Expanda la opción **Recursos compartidos de archivos** de la cuenta de almacenamiento.

1.  Haga doble clic en el recurso compartido de archivos que desea ver.

1.  El panel principal mostrará el contenido del recurso compartido de archivos.

    ![Captura de pantalla del panel principal del recurso compartido de archivos myazurefileshare en el Explorador de Storage, que muestra el contenido del recurso compartido con la primera carpeta seleccionada.](media/vs-azure-tools-storage-explorer-files/image17.png)

1.  El panel principal mostrará el contenido del recurso compartido de archivos.

1.  Siga estos pasos según la tarea que desea realizar:

    - **Cargar archivos en un recurso compartido de archivos**

        a.  En la barra de herramientas del panel principal, seleccione **Cargar** y, luego, **Cargar archivos** en el menú desplegable.

        ![Carga de archivos](media/vs-azure-tools-storage-explorer-files/image18.png)
        
        b. En el cuadro de diálogo **Cargar archivos**, seleccione el botón de puntos suspensivos ( **...** ) a la derecha del cuadro de texto **Archivos** para seleccionar los archivos que desea cargar.

        ![Adición de archivos](media/vs-azure-tools-storage-explorer-files/image19.png)

        c. Seleccione **Cargar**.

    - **Cargar una carpeta en un recurso compartido de archivos**
        
        a. En la barra de herramientas del panel principal, seleccione **Cargar**, y luego **Cargar carpeta** en el menú desplegable.

        ![Menú Cargar carpeta](media/vs-azure-tools-storage-explorer-files/image20.png)

        b. En el cuadro de diálogo **Cargar carpeta**, seleccione el botón de puntos suspensivos ( **...** ) a la derecha del cuadro de texto **Carpeta** para seleccionar la carpeta cuyo contenido desea cargar.

        c. Opcionalmente, especifique la carpeta de destino en la que se cargará el contenido de la carpeta seleccionada. Si la carpeta de destino no existe, se creará.

        d. Seleccione **Cargar**.

    - **Descargar un archivo en el equipo local**
        
        a. Seleccione el archivo que desea descargar.
        
        b. En la barra de herramientas del panel principal, seleccione **Descargar**.
        
        c. En el cuadro de diálogo **Specify where to save the downloaded blob** (Especificar dónde se guarda el archivo descargado), especifique la ubicación en que desea descargar el archivo y el nombre que desee darle.

        d. Seleccione **Guardar**.

    - **Abrir un archivo en el equipo local**
        
        a.  Seleccione el archivo que desea abrir.
        
        b.  En la barra de herramientas del panel principal, seleccione **Abrir**.
        
        c.  El archivo se descargará y abrirá mediante la aplicación asociada con el tipo de archivo subyacente del archivo.

    - **Copiar un archivo en el Portapapeles**

        a. Seleccione el archivo que desea copiar.

        b. En la barra de herramientas del panel principal, seleccione **Copiar**.

        c. En el panel izquierdo, navegue a otro recurso compartido de archivos y haga doble clic en él para verlo en el panel principal.

        d. En la barra de herramientas del panel principal, seleccione **Pegar** para crear una copia del archivo.

    - **Eliminar un archivo**

        a. Seleccione el archivo que desea eliminar.

        b. En la barra de herramientas del panel principal, seleccione **Eliminar**.

        c. Haga clic en **Sí** en el cuadro de diálogo de confirmación.

## <a name="next-steps"></a>Pasos siguientes

- Vea las [notas de la versión y los vídeos más recientes del Explorador de Storage](https://www.storageexplorer.com/).

- Obtenga información acerca de cómo [crear aplicaciones con blobs, tablas, colas y archivos de Azure](https://azure.microsoft.com/documentation/services/storage/).
