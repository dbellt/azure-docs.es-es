---
title: Tutorial de copia de datos mediante SMB desde Azure Data Box| Microsoft Docs
description: Aprenda a copiar datos a un dispositivo Azure Data Box mediante SMB
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 05/17/2021
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 68012cd27318822b8c18b281db967a26da3a15a2
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110095651"
---
# <a name="tutorial-copy-data-from-azure-data-box-via-smb"></a>Tutorial: Copia de datos de Azure Data Box a través de SMB

En este tutorial se describe cómo conectarse a Data Box y copiar datos desde este dispositivo en un servidor local mediante la interfaz de usuario web local. El dispositivo Data Box contiene los datos exportados desde la cuenta de Azure Storage.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Requisitos previos
> * Conexión a un dispositivo Data Box
> * Copiar datos de Data Box

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:

1. Ha realizado el pedido de Azure Data Box.
    - Si necesita un pedido de importación, consulte el [Tutorial: Realización de pedidos de Azure Data Box](data-box-deploy-ordered.md).
    - En caso de un pedido de exportación, consulte el [Tutorial: Realización de pedidos de Azure Data Box](data-box-deploy-export-ordered.md).
2. Ha recibido su dispositivo Data Box y el estado del pedido en el portal se actualiza a **Delivered** (Entregado).
3. Tiene un equipo host en el que quiere copiar los datos de Data Box. El equipo host debe:
   * Ejecutar un [sistema operativo admitido](data-box-system-requirements.md).
   * Estar conectado a una red de alta velocidad. Es muy recomendable tener una conexión de 10 GbE como mínimo. Si no hay disponible una conexión 10 GbE, use un vínculo de datos de 1 GbE, pero las velocidades de copia se verán afectadas.

## <a name="connect-to-data-box"></a>Conexión a un dispositivo Data Box

[!INCLUDE [data-box-shares](../../includes/data-box-shares.md)]

Si usa un equipo host Windows Server, realice los pasos siguientes para conectarse a su dispositivo Data Box.

1. El primer paso es autenticarse e iniciar sesión. Vaya a **Connect and copy** (Conectar y copiar). Seleccione **Get credentials** (Obtener credenciales) para obtener las credenciales de acceso a los recursos compartidos asociados con la cuenta de almacenamiento. 

    ![Obtención de las credenciales de recursos compartidos](media/data-box-deploy-export-copy-data/get-share-credentials-1.png)

2. En el cuadro de diálogo Access share and copy data (Acceder al recurso compartido y copiar datos), copie los valores de **Username** (Nombre de usuario) y **Password** (Contraseña) del recurso compartido. Seleccione **Aceptar**.
    
    ![Obtener credenciales del recurso compartido, acceder a recursos compartidos y copiar datos](media/data-box-deploy-export-copy-data/get-share-credentials-2.png)

3. Para acceder a los recursos compartidos asociados con la cuenta de almacenamiento (*exportbvtdataset2* en el ejemplo siguiente) desde el equipo host, abra una ventana de comandos. En el símbolo del sistema, escriba:

    `net use \\<IP address of the device>\<share name>  /u:<IP address of the device>\<user name for the share>`

    Dependiendo del formato de los datos, las rutas de acceso de los recursos compartidos son las siguientes:
    - Blob en bloques de Azure: `\\169.254.143.85\exportbvtdataset2_BlockBlob`
    - Blob en páginas de Azure: `\\169.254.143.85\exportbvtdataset2_PageBlob`
    - Azure Files: `\\169.254.143.85\exportbvtdataset2_AzFile`

4. Cuando se le solicite, escriba la contraseña del recurso compartido. En el ejemplo siguiente se muestra la conexión a un recurso compartido con el comando anterior.

    ```
    C:\Users\Databoxuser>net use \\169.254.143.85\exportbvtdataset2_BlockBlob /u:169.254.143.85\exportbvtdataset2
    Enter the password for 'exportbvtdataset2' to connect to '169.254.143.85':
    The command completed successfully.
    ```

5. Presione Windows + R. En la ventana **Ejecutar**, escriba `\\<device IP address>`. Seleccione **Aceptar** para abrir el Explorador de archivos.
    
    ![Conectar a un recurso compartido de archivos mediante el Explorador de archivos y especificar la IP del dispositivo](media/data-box-deploy-export-copy-data/connect-shares-file-explorer-1.png)

    Ahora debería ver los recursos compartidos como carpetas.
    
    ![Conectar a un recurso compartido de archivos mediante el Explorador de archivos y ver los recursos compartidos de archivos](media/data-box-deploy-export-copy-data/connect-shares-file-explorer-2.png)

    
Si usa un cliente Linux, utilice el siguiente comando para montar el recurso compartido SMB. El parámetro "vers" siguiente es la versión de SMB compatible con el host Linux. Conecte la versión adecuada en el siguiente comando. Para ver las versiones de SMB compatibles con Data Box, consulte [Sistemas de archivos compatibles para clientes Linux](./data-box-system-requirements.md#supported-file-transfer-protocols-for-clients). 

```console
sudo mount -t nfs -o vers=2.1 169.254.143.85:/exportbvtdataset2_BlockBlob /home/databoxubuntuhost/databox
```

## <a name="copy-data-from-data-box"></a>Copiar datos de Data Box

Una vez que esté conectado a los recursos compartidos de Data Box, el siguiente paso es copiar los datos.

[!INCLUDE [data-box-export-review-logs](../../includes/data-box-export-review-logs.md)]


 Después de haberse conectado al recurso compartido SMB, inicie una copia de datos. Puede usar cualquier herramienta de copia de archivos compatible con SMB, como Robocopy, para copiar los datos. Con Robocopy se pueden iniciar varios trabajos de copia. 


Para más información sobre el comando Robocopy, consulte [Robocopy and a few examples](https://social.technet.microsoft.com/wiki/contents/articles/1073.robocopy-and-a-few-examples.aspx) (Robocopy y algunos ejemplos).

Finalizada la copia, vaya al **Panel** y compruebe el espacio usado y libre en el dispositivo.

Ahora puede proceder al envío del dispositivo Data Box a Microsoft.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Data Box, como:

> [!div class="checklist"]
>
> * Requisitos previos
> * Conexión a un dispositivo Data Box
> * Copiar datos de Data Box

En el siguiente tutorial aprenderá a enviar su dispositivo Data Box a Microsoft.

> [!div class="nextstepaction"]
> [Envío de un dispositivo Data Box a Microsoft](./data-box-deploy-export-picked-up.md)