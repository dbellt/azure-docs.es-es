---
title: Restauración de todos los archivos de un volumen con MARS
description: Aprenda a restaurar todos los archivos de un volumen mediante el agente de MARS.
ms.topic: conceptual
ms.date: 01/17/2021
ms.openlocfilehash: 1d04e9f77b9f92594def9381f973c999e96b2cb2
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107516508"
---
# <a name="restore-all-the-files-in-a-volume-using-the-mars-agent"></a>Restaure todos los archivos de un volumen mediante el agente de MARS.

En este artículo se explica cómo restaurar todos los archivos de copia de seguridad de un volumen completo mediante el Asistente de recuperación de datos del agente Microsoft Azure Recovery Services (MARS). Puede:

- Restaurar los archivos de copia de seguridad de un volumen en la misma máquina desde la que se realizaron las copias de seguridad.
- Restaurar todos los archivos de copia de seguridad de un volumen en una máquina alternativa.

>[!TIP]
>La opción **Volumen** recupera todos los datos de los que se ha realizado una copia de seguridad en un volumen especificado. Esta opción ofrece mayores velocidades de transferencia (hasta 40 MBps) y se recomienda para la recuperación de volúmenes completos o de datos de gran tamaño.
>
>La opción **Archivos y carpetas individuales** permite el acceso rápido a los datos de punto de recuperación. Es adecuado para recuperar archivos individuales y se recomienda para un tamaño total de menos de 80 GB. Ofrece velocidades de transferencia o copia de hasta 6 MBps durante la recuperación.

## <a name="volume-level-restore-to-the-same-machine"></a>Restauración en el nivel de volumen en la misma máquina

Los pasos siguientes le ayudarán a recuperar todos los archivos de copia de seguridad de un volumen:

1. Abra el complemento **Microsoft Azure Backup**. Si no conoce la ubicación donde se instaló el complemento, busque el equipo o servidor para **Microsoft Azure Backup**. La aplicación de escritorio debe aparecer en los resultados de búsqueda.

1. Seleccione **Recuperar datos** para iniciar el asistente.

    ![Menú Recuperar datos](./media/restore-all-files-volume-mars/recover.png)

1. En la página **Introducción**, para restaurar los datos en el mismo servidor o equipo, seleccione **Este servidor (nombre del servidor)**  > **Siguiente**.

    ![Página de introducción](./media/restore-all-files-volume-mars/same-machine-instant-restore.png)

1. En la página **Seleccionar modo de recuperación**, elija **Volumen** > **Siguiente**.

    ![Selección del modo de recuperación](./media/restore-all-files-volume-mars/select-recovery-mode.png)

1. En la página **Seleccionar volumen y fecha**, seleccione el volumen que quiere restaurar.

    En el calendario, seleccione un punto de recuperación. Las fechas en **negrita** indican la disponibilidad de al menos un punto de recuperación. Si están disponibles varios puntos de recuperación dentro de una única fecha, elija el punto de recuperación específico desde el menú desplegable **Hora**.

     ![Selección de volumen y fecha](./media/restore-all-files-volume-mars/select-volume-and-date.png)

1. En la página **Especificar opciones de recuperación**, configure el comportamiento de restauración.
    1. Elija el destino de recuperación:
        - **Ubicación original**: restaure los datos en la ruta de acceso original.
        - **Otra ubicación**: especifique una ubicación alternativa en la que restaurar los datos.
    1. Elija el comportamiento de **Cuando los elementos de la copia de seguridad ya están en el destino de la recuperación**:
        - **Crear copias para tener ambas versiones**: si ya existe un archivo con el mismo nombre, los datos del punto de recuperación se restaurarán como una copia. La copia tendrá un prefijo de nombre de archivo localizado con la hora del trabajo de restauración local en uno de los siguientes formatos:
            - `YYYY-MM-DD HH-mm Copy of <original file name>`
            - `YYYY-MM-DD HH-mm Copy (n) of <original file name>`
        - **Overwrite existing versions with recovered versions** (Sobrescribir las versiones existentes con las recuperadas): si ya existe un archivo con el mismo nombre, el contenido se reemplaza por los datos del punto de recuperación.
        - **No recuperar los elementos que ya existen en el destino de recuperación**: si ya existe un archivo con el mismo nombre, se omite.
    1. **Habilite la restauración de los permisos de la lista de control de acceso (ACL) en el archivo o la carpeta que se va a recuperar** si el archivo debe restaurarse con los permisos originales en el punto de recuperación.
        ![Especificación de las opciones de recuperación](./media/restore-all-files-volume-mars/specify-recovery-options.png)

1. Compruebe los detalles de recuperación en el panel **Confirmación** y seleccione **Recuperar**.

    ![Detalles de confirmación](./media/restore-all-files-volume-mars/confirmation-details.png)

1. En la página **Progreso de la recuperación**, supervise el progreso del trabajo de recuperación. También se puede cerrar el asistente de forma segura y la operación de recuperación continuará en segundo plano. Puede ver el progreso de nuevo si hace doble clic en el trabajo de recuperación en el panel.

## <a name="volume-level-restore-to-an-alternate-machine"></a>Restauración en el nivel de volumen en una máquina alternativa

Los pasos siguientes le ayudarán a recuperar todos los archivos de copia de seguridad de un volumen en una máquina alternativa. Si ha perdido todo el servidor, puede usar estos pasos para recuperar datos de Azure Backup.

Estos pasos incluyen la siguiente terminología:

- *Máquina de origen* : es la máquina original desde la que se realizó la copia de seguridad y que no está disponible actualmente.
- *Máquina de destino* : es la máquina en la que se recuperan los datos.
- *Almacén de ejemplo*: el almacén de Recovery Services en el que se registran la máquina de origen y la máquina de destino.

> [!NOTE]
> Las copias de seguridad no se pueden restaurar en una máquina de destino en la que se ejecuta una versión anterior del sistema operativo. Por ejemplo, una copia de seguridad perteneciente a un equipo con Windows 7 se puede restaurar en un equipo con Windows 7 o posterior. Una copia de seguridad tomada de un equipo con Windows 10 no se puede restaurar en un equipo con Windows 7.

1. Abra el complemento **Microsoft Azure Backup** en la máquina de destino.

1. Asegúrese de que tanto la máquina de destino como la máquina de origen están registradas en el mismo almacén de Recovery Services.

1. Seleccione **Recuperar datos** para abrir el **Asistente de recuperación de datos**.

    ![Captura de pantalla de Azure Backup, con la opción Recuperar datos resaltada (restauración en una máquina alternativa)](./media/backup-azure-restore-windows-server/recover.png)

1. En la página **Introducción**, seleccione **Otro servidor**.

    ![Captura de pantalla de la página de introducción al Asistente de recuperación de datos (restauración en una máquina alternativa)](./media/backup-azure-restore-windows-server/alternatemachine_gettingstarted_instantrestore.png)

1. Proporcione el archivo de credenciales de almacén que se corresponde con el almacén de ejemplo y seleccione **Siguiente**.

    Si el archivo de credenciales de almacén no es válido o ha expirado, [descargue un nuevo archivo de credenciales de almacén desde el almacén de ejemplo](backup-azure-file-folder-backup-faq.yml#where-can-i-download-the-vault-credentials-file-) en Azure Portal. Una vez proporcionada una credencial de almacén válida, aparecerá el nombre del almacén de Backup correspondiente.

1. En la página **Seleccionar servidor de copia de seguridad**, seleccione la máquina de origen en la lista de máquinas mostradas y proporcione la frase de contraseña. Luego, seleccione **Siguiente**.

    ![Captura de pantalla de la página Seleccionar servidor de copia de seguridad del Asistente de recuperación de datos (restauración en una máquina alternativa)](./media/backup-azure-restore-windows-server/alternatemachine_selectmachine_instantrestore.png)

1. En la página **Seleccionar modo de recuperación**, elija **Volumen** > **Siguiente**.

    ![Selección del modo de recuperación](./media/restore-all-files-volume-mars/select-recovery-mode.png)

1. En la página **Seleccionar volumen y fecha**, seleccione el volumen que quiere restaurar.

    En el calendario, seleccione un punto de recuperación. Las fechas en **negrita** indican la disponibilidad de al menos un punto de recuperación. Si están disponibles varios puntos de recuperación dentro de una única fecha, elija el punto de recuperación específico desde el menú desplegable **Hora**.

     ![Selección de volumen y fecha](./media/restore-all-files-volume-mars/select-volume-and-date.png)

1. En la página **Especificar opciones de recuperación**, configure el comportamiento de restauración.
    1. Elija el destino de recuperación:
        - **Ubicación original**: restaure los datos en la ruta de acceso original.
        - **Otra ubicación**: especifique una ubicación alternativa en la que restaurar los datos.
    1. Elija el comportamiento de **Cuando los elementos de la copia de seguridad ya están en el destino de la recuperación**:
        - **Crear copias para tener ambas versiones**: si ya existe un archivo con el mismo nombre, los datos del punto de recuperación se restaurarán como una copia. La copia tendrá un prefijo de nombre de archivo localizado con la hora del trabajo de restauración local en uno de los siguientes formatos:
            - `YYYY-MM-DD HH-mm Copy of <original file name>`
            - `YYYY-MM-DD HH-mm Copy (n) of <original file name>`
        - **Overwrite existing versions with recovered versions** (Sobrescribir las versiones existentes con las recuperadas): si ya existe un archivo con el mismo nombre, el contenido se reemplaza por los datos del punto de recuperación.
        - **No recuperar los elementos que ya existen en el destino de recuperación**: si ya existe un archivo con el mismo nombre, se omite.
    1. **Habilite la restauración de los permisos de la lista de control de acceso (ACL) en el archivo o la carpeta que se va a recuperar** si el archivo debe restaurarse con los permisos originales en el punto de recuperación.
        ![Especificación de las opciones de recuperación](./media/restore-all-files-volume-mars/specify-recovery-options.png)

1. Compruebe los detalles de recuperación en el panel **Confirmación** y seleccione **Recuperar**.

    ![Detalles de confirmación](./media/restore-all-files-volume-mars/confirmation-details.png)

1. En la página **Progreso de la recuperación**, supervise el progreso del trabajo de recuperación. También se puede cerrar el asistente de forma segura y la operación de recuperación continuará en segundo plano. Puede ver el progreso de nuevo si hace doble clic en el trabajo de recuperación en el panel.

## <a name="next-steps"></a>Pasos siguientes

- Ahora que ha recuperado los archivos y las carpetas, puede [administrar las copias de seguridad](backup-azure-manage-windows-server.md).
- Busque [Preguntas comunes acerca de la realización de copias de seguridad de archivos y carpetas](backup-azure-file-folder-backup-faq.yml).
