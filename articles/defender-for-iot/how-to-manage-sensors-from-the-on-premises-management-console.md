---
title: Administración de sensores desde la consola de administración local
description: Obtenga información sobre cómo administrar los sensores desde la consola de administración, incluida la actualización de las versiones de los sensores, la inserción de la configuración del sistema en sensores y también la habilitación y deshabilitación de motores en sensores.
ms.date: 04/22/2021
ms.topic: how-to
ms.openlocfilehash: 2a1f1dbd69013db63365b64e255225c465b67238
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108734865"
---
# <a name="manage-sensors-from-the-management-console"></a>Administración de sensores desde la consola de administración

En este artículo se describe cómo administrar los sensores desde la consola de administración, lo que incluye:

- Inserción de la configuración del sistema en sensores

- Habilitación y deshabilitación de motores en sensores

- Actualización de las versiones de los sensores

## <a name="push-configurations"></a>Inserción de configuraciones

Puede definir varias configuraciones del sistema y aplicarlas automáticamente a los sensores que están conectados a la consola de administración. De este modo se ahorra tiempo y se ayuda a garantizar una configuración optimizada en los sensores de su empresa.

Puede definir la siguiente configuración del sistema de sensores desde la consola de administración:

- Servidor de correo

- Supervisión de MIB del protocolo simple de administración de redes

- Active Directory

- Configuración de DNS

- Subredes

- Alias de puerto

Para aplicar la configuración del sistema:

1. En el panel izquierdo de la consola, seleccione **Configuración del sistema**.

1. En el panel **Configure Sensors** (Configurar sensores), seleccione una de las opciones.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/sensor-system-setting-options.png" alt-text="Opciones de configuración del sistema para un sensor.":::

   En el ejemplo siguiente se describe cómo definir los parámetros del servidor de correo para los sensores de la empresa.

1. Seleccione **Servidor de correo**.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/edit-system-settings-screen.png" alt-text="Seleccione el servidor de correo en la pantalla Configuración del sistema.":::

1. Seleccione un sensor de la izquierda.

1. Establezca los parámetros del servidor de correo y seleccione **Duplicar**. Cada elemento del árbol del sensor aparece con una casilla junto a él.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/check-off-each-sensor.png" alt-text="Asegúrese de que las casillas de los sensores están seleccionadas.":::

1. En el árbol del sensor, seleccione los elementos a los que desea aplicar la configuración.

1. Seleccione **Guardar**.

## <a name="update-versions"></a>Actualización de versiones

Puede actualizar varios sensores simultáneamente desde la consola de administración local.

Para actualizar varios sensores:

1. Vaya a [Azure Portal](https://portal.azure.com/).

1. Vaya a Azure Defender para IoT.

1. Vaya a la página **Actualizaciones**.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/update-screen.png" alt-text="Captura de pantalla de la vista del panel Actualizaciones.":::

1. Seleccione **Descargar** en la sección **Sensores** y guarde el archivo.

1. Inicie sesión en la consola de administración y seleccione **Configuración del sistema**.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/admin-system-settings.png" alt-text="Captura de pantalla del menú Administración para seleccionar Configuración del sistema.":::

1. Seleccione los sensores que se van a actualizar en la sección **Sensor Engine Configuration** (Configuración del motor de sensores) y, a continuación, seleccione **Automatic Updates** (Actualizaciones automáticas).

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/sensors-select.png" alt-text="Dos sensores que muestran el modo de aprendizaje y las actualizaciones automáticas.":::

1. Seleccione **Save changes** (Guardar los cambios).

1. En el sensor, seleccione **System Settings** (Configuración del sistema) y, a continuación, seleccione **Update** (Actualizar).

    :::image type="content" source="media/how-to-manage-individual-sensors/upgrade-pane-v2.png" alt-text="Captura de pantalla del panel de actualización.":::

9. Se abre un cuadro de diálogo **Cargar archivo**. Cargue el archivo que descargó de la página **Actualizaciones**.

    :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/upload-file.png" alt-text="Seleccione el botón Examinar para cargar el archivo.":::

Puede supervisar el estado de actualización de cada sensor en la ventana **Site Management** (Administración del sitio).

:::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/progress.png" alt-text="Observe el progreso de la actualización.":::

### <a name="update-sensors-from-the-on-premises-management-console"></a>Actualización de sensores desde la consola de administración local

Puede ver el estado de actualización de los sensores desde la consola de administración. Si la actualización no se ha podido realizar, puede volver a intentar actualizar el sensor desde la consola de administración local (versiones 2.3.5 y posteriores).

Para actualizar el sensor desde la consola de administración local:

1. Inicie sesión en la consola de administración local y vaya a la página **Sites Management** (Administración de sitios).

1. Busque los sensores que muestren **Failed** (Error) en la columna Update Progress (Progreso de la actualización) y seleccione el botón Download (Descargar). 

    :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/download-update-button.png" alt-text="Seleccione el icono de descarga para intentar descargar e instalar la actualización del sensor.":::

Puede supervisar el estado de actualización de cada sensor en la ventana **Site Management** (Administración del sitio).

:::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/progress.png" alt-text="Observe el progreso de la actualización.":::

Si no puede actualizar el sensor, póngase en contacto con el equipo de soporte técnico al cliente para obtener ayuda.

## <a name="update-threat-intelligence-packages"></a>Actualización de paquetes de inteligencia sobre amenazas 

El paquete de datos de la inteligencia sobre amenazas se proporciona con cada nueva versión de Defender para IoT o, si es necesario, entre versiones. El paquete contiene firmas (incluidas las firmas de malware), CVE y otro contenido de seguridad. 

Puede cargar manualmente este archivo desde la página **Actualizaciones** del portal de Defender para IoT y actualizarlo automáticamente en los sensores. 

Para actualizar los datos de inteligencia sobre amenazas: 

1. Vaya a la página **Actualizaciones** de Defender para IoT. 

1. Descargue y guarde el archivo.

1. Inicie sesión en la consola de administración. 

1. En el menú lateral, seleccione **Configuración del sistema**. 

1. Seleccione los sensores que deben recibir la actualización en la sección **Sensor Engine Configuration** (Configuración del motor de sensores).  

1. En la sección **Select Threat Intelligence Data** (Seleccionar datos de inteligencia sobre amenazas), seleccione el signo más ( **+** ). 

1. Cargue el paquete que descargó de la página **Actualizaciones** de Defender para IoT.

## <a name="understand-sensor-disconnection-events"></a>Descripción de los eventos de desconexión de sensores

En la ventana de **Site Manager** se muestra la información de desconexión si los sensores se desconectan de la consola de administración local asignada. Está disponible la siguiente información de desconexión de sensor:

- "The on-premises management console cannot process data received from the sensor" (La consola de administración local no puede procesar los datos recibidos del sensor).

- "Times drift detected (Se ha detectado un desfase de tiempo). The on-premises management console has been disconnected from sensor" (La consola de administración local se ha desconectado del sensor).

- "Sensor not communicating with on-premises management console (El sensor no se comunica con la consola de administración local). Check network connectivity or certificate validation" (Compruebe la conectividad de red o la validación de certificados).

:::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/edit-system-settings-screen.png" alt-text="Captura de pantalla de la vista de la zona 1.":::

Puede enviar alertas a terceros con información sobre los sensores desconectados. Para obtener más información, consulte [Reenvío de alertas de error de sensores](how-to-manage-individual-sensors.md#forward-sensor-failure-alerts).

## <a name="enable-or-disable-sensors"></a>Habilitación o deshabilitación de sensores

Los sensores están protegidos por cinco motores de Defender para IoT. Puede habilitar o deshabilitar los motores para los sensores conectados.

| Motor | Descripción | Escenario de ejemplo |
|--|--|--|
| Motor de infracción de protocolo | Se produce una infracción de protocolo cuando la estructura de paquetes o los valores de campo no cumplen la especificación del protocolo. | Alerta "Illegal MODBUS Operation (Function Code Zero)" [Operación MODBUS no válida (código de función cero)]. Esta alerta indica que un dispositivo principal envió una solicitud con el código de función 0 a un dispositivo secundario. Esto no se permite según la especificación del protocolo y es posible que el dispositivo secundario no controle la entrada correctamente. |
| Motor de infracción de directiva | Se produce una infracción de directiva con una desviación del comportamiento de base de referencia definido en la directiva aprendida o configurada. | Alerta "Unauthorized HTTP User Agent" (Agente de usuario HTTP no autorizado). Esta alerta indica que una aplicación que no se ha aprendido o aprobado por la directiva se usa como cliente HTTP en un dispositivo. Puede tratarse de una nueva aplicación o explorador web de ese dispositivo. |
| Motor de malware | El motor de malware detecta actividades de red malintencionadas. | Alerta "Suspicion of Malicious Activity (Stuxnet)" [Sospecha de actividad malintencionada (Stuxnet)]. Esta alerta indica que el sensor detectó una actividad de red sospechosa que se sabe que está relacionada con el malware Stuxnet, que es una amenaza persistente avanzada dirigida a las redes de control industrial y de SCADA. |
| Motor de anomalías | El motor de malware detecta una anomalía en el comportamiento de la red. | "Periodic Behavior in Communication Channel" (Comportamiento periódico en el canal de comunicación). Se trata de un componente que inspecciona las conexiones de red y busca el comportamiento cíclico o periódico de la transmisión de datos, que es común en redes industriales. |
| Motor operativo | Este motor detecta incidentes operativos o entidades que no funcionan correctamente. | Alerta `Device is Suspected to be Disconnected (Unresponsive)`. Esta alerta se desencadena cuando un dispositivo no responde a las solicitudes durante un período predefinido. Podría indicar un apagado, desconexión o funcionamiento incorrecto del dispositivo.
|

Para habilitar o deshabilitar los motores para los sensores conectados:

1. En el panel izquierdo de la consola, seleccione **Configuración del sistema**.

1. En la sección **Sensor Engine Configuration** (Configuración del motor de sensores), seleccione **Habilitar** o **Deshabilitar** para los motores.
         
1. Seleccione **GUARDAR CAMBIOS**.

   Aparece un signo de exclamación rojo si no coinciden los motores habilitados en uno de los sensores de la empresa. Es posible que el motor se haya deshabilitado directamente desde el sensor.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/red-exclamation-example.png" alt-text="Los motores habilitados no coinciden."::: 

## <a name="define-sensor-backup-schedules"></a>Definición de programaciones de copia de seguridad de sensores

Puede programar copias de seguridad de sensores y realizar copias de seguridad de sensores a petición desde la consola de administración local. Esto ayuda a la protección frente a errores de disco duro y pérdida de datos.

- Elementos incluidos en la copia de seguridad: Configuraciones y datos.

- Elementos no incluidos en la copia de seguridad: Archivos PCAP y registros. Puede realizar copias de seguridad y restaurar los PCAP y los registros manualmente.

De forma predeterminada, se realiza automáticamente una copia de seguridad de los sensores a las 3:00 AM a diario. La característica de programación de copias de seguridad de los sensores le permite recopilar estas copias de seguridad y almacenarlas en la consola de administración local o en un servidor de copia de seguridad externo. La copia de los archivos de sensores en la consola de administración local se realiza a través de un canal cifrado.

:::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/sensor-backup-schedule-screen.png" alt-text="Una vista de la pantalla de copia de seguridad de sensores.":::

Cuando se cambia la ubicación predeterminada de la copia de seguridad de los sensores, la consola de administración local recupera automáticamente los archivos de la nueva ubicación en el sensor o en una ubicación externa, siempre que la consola tenga permiso para acceder a la ubicación. 

Cuando los sensores no están registrados en la consola de administración local, el cuadro de diálogo **Sensor Backup Schedule** (Programación de copia de seguridad de sensores) indica que no hay ningún sensor administrado.  

El proceso de restauración es el mismo, independientemente de dónde se almacenen los archivos.

### <a name="backup-storage-for-sensors"></a>Almacenamiento de la copia de seguridad de sensores

Puede usar la consola de administración local para mantener hasta nueve copias de seguridad para cada sensor administrado, siempre que los archivos de copia de seguridad no superen el espacio de copia de seguridad máximo asignado. 

El espacio disponible se calcula en función del modelo de consola de administración con el que trabaja: 

- **Modelo de producción**: el almacenamiento predeterminado es de 40 GB; el límite es de 100 GB. 

- **Modelo medio**: el almacenamiento predeterminado es de 20 GB; el límite es de 50 GB. 

- **Modelo portátil**: el almacenamiento predeterminado es de 10 GB; el límite es de 25 GB. 

- **Modelo fino**: el almacenamiento predeterminado es de 2 GB; el límite es de 4 GB. 

- **Modelo resistente**: el almacenamiento predeterminado es de 10 GB; el límite es de 25 GB. 

La asignación predeterminada se muestra en el cuadro de diálogo **Sensor Backup Schedule** (Programación de copia de seguridad de sensores). 

:::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/edit-mail-server-configuration.png" alt-text="Pantalla de edición de la configuración del servidor de correo electrónico.":::

No hay ningún límite de almacenamiento cuando se realiza una copia de seguridad en un servidor externo. Sin embargo, debe definir un límite de asignación superior en el campo **Sensor Backup Schedule** (Programación de copia de seguridad de sensores) > **Ruta de acceso personalizada**. Se admiten los siguientes números y caracteres: `/, a-z, A-Z, 0-9, and _`. 

A continuación, se muestra información sobre la superación de los límites de almacenamiento de asignación:

- Si supera el espacio de almacenamiento asignado, no se realiza la copia de seguridad del sensor. 

- Si realiza una copia de seguridad de más de un sensor, la consola de administración intenta recuperar los archivos de sensor de los sensores administrados.  

- Si la recuperación de un sensor supera el límite, la consola de administración intenta recuperar información de copia de seguridad del siguiente sensor. 

Cuando supera el número de copias de seguridad retenidas que se ha definido, se elimina el archivo de copia de seguridad más antiguo para acomodar el nuevo.

El nombre de los archivos de copia de seguridad de sensores se asigna automáticamente con el siguiente formato: `<sensor name>-backup-version-<version>-<date>.tar`. Por ejemplo: `Sensor_1-backup-version-2.6.0.102-2019-06-24_09:24:55.tar`. 

Para realizar una copia de seguridad de sensores:

1. Seleccione **Schedule Sensor Backup** (Programar copia de seguridad de sensores) en la ventana **Configuración del sistema**. Los sensores que la consola de administración local administra aparecen en el cuadro de diálogo **Sensor Backup Schedule** (Programación de copia de seguridad de sensores).  

1. Habilite la opción **Collect Backups** (Recopilar copias de seguridad).  

1. Seleccione un intervalo de calendario, una fecha y una zona horaria. El formato de hora se basa en un reloj de 24 horas. Por ejemplo, especifique las 6:00 PM como **18:00**. 

1. En el campo **Backup Storage Allocation** (Asignación de almacenamiento de copia de seguridad), especifique el almacenamiento que desea asignar a las copias de seguridad. Se le notificará si supera el espacio máximo.

1. En el campo **Retain Last** (Conservar las últimas), indique el número de copias de seguridad por sensor que desea conservar. Cuando se supera el límite, se elimina la copia de seguridad más antigua.  

1. Elija una ubicación de copia de seguridad:  

   - Para realizar una copia de seguridad en la consola de administración local, deshabilite la opción **Ruta de acceso personalizada**. La ubicación predeterminada es `/var/cyberx/sensor-backups`.  

   - Para realizar una copia de seguridad en un servidor externo, habilite la opción **Ruta de acceso personalizada** y especifique una ubicación. Se admiten los siguientes números y caracteres: `/, a-z, A-Z, 0-9, and, _`. 

1. Seleccione **Guardar**. 

Para realizar la copia de seguridad inmediatamente: 

- Seleccione **Hacer Copia Ahora**. La consola de administración local crea y recopila archivos de copia de seguridad de sensores. 

### <a name="receiving-backup-notifications-for-sensors"></a>Recepción de notificaciones de copia de seguridad para sensores 

En el cuadro de diálogo **Sensor Backup Schedule** (Programación de copia de seguridad de sensores) y en el registro de copia de seguridad se indica la información sobre los errores y las copias de seguridad.  

:::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/sensor-location.png" alt-text="Visualización de los sensores, dónde se encuentran y toda la información pertinente.":::

Pueden producirse errores debido a:    

- No se encuentra ningún archivo de copia de seguridad. 

- Se encontró un archivo, pero no se puede recuperar.  

- Hay un error de conexión de red. 

- No hay suficiente espacio asignado a la consola de administración local para completar la copia de seguridad.  

Puede enviar una notificación por correo electrónico, actualizaciones de syslog y notificaciones del sistema cuando se produce un error. Para ello, cree una regla de reenvío en **Notificaciones del sistema**. 

### <a name="restoring-sensors"></a>Restauración de sensores 

Puede restaurar las copias de seguridad desde la consola de administración local y mediante la CLI.  

Para restaurar desde la consola: 

- Seleccione **Restore Image** (Restaurar imagen) en la ventana de configuración **Sensor System** (Sistema de sensores).

  :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/restore.png" alt-text="Restaure una copia de seguridad de la imagen.":::

  A continuación, la consola muestra los errores de restauración.  

Para realizar la restauración con la CLI: 

- Inicie sesión en una cuenta administrativa y escriba `$ sudo cyberx-xsense-system-restore`. 

### <a name="save-a-sensor-backup-to-an-external-smb-server"></a>Procedimiento para guardar la copia de seguridad de un sensor en un servidor SMB externo

Si quiere configurar un servidor SMB para que pueda guardar una copia de seguridad de un sensor en una unidad externa: 

1. Cree una carpeta compartida en el servidor SMB externo. 

1. Obtenga la ruta de acceso de la carpeta, el nombre de usuario y la contraseña necesarios para acceder al servidor SMB. 

1. En Defender para IoT, cree un directorio para las copias de seguridad: 

   `sudo mkdir /<backup_folder_name_on_server>` 

   `sudo chmod 777 /<backup_folder_name_on_server>/` 

1. Edite fstab:  

   `sudo nano /etc/fstab` 

   `add - //<server_IP>/<folder_path> /<backup_folder_name_on_cyberx_server> cifs rw,credentials=/etc/samba/user,vers=3.0,uid=cyberx,gid=cyberx,file_mode=0777,dir_mode=0777 0 0` 

1. Edite o cree credenciales para uso compartido. Estas son las credenciales para el servidor SMB: 

   `sudo nano /etc/samba/user` 

1. Agregue:  

   `username=<user name>` 

   `password=<password>` 

1. Monte el directorio: 

   `sudo mount -a` 

1. Configure un directorio de copia de seguridad en la carpeta compartida en Defender para IoT:  

   `sudo nano /var/cyberx/properties/backup.properties` 

1. Establezca `Backup.shared_location` en `<backup_folder_name_on_cyberx_server>`.

## <a name="see-also"></a>Consulte también

[Administración de sensores individuales](how-to-manage-individual-sensors.md)
