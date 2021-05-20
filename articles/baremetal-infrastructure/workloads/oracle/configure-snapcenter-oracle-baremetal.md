---
title: Configuración de SnapCenter para Oracle en BareMetal Infrastructure
description: Aprenda a configurar SnapCenter para Oracle en BareMetal Infrastructure.
ms.topic: how-to
ms.subservice: workloads
ms.date: 05/05/2021
ms.openlocfilehash: f12ae9eae62ecb47cafe5d555b5173a75c66200b
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109520827"
---
# <a name="configure-snapcenter-for-oracle-on-baremetal-infrastructure"></a>Configuración de SnapCenter para Oracle en BareMetal Infrastructure

En este artículo, haremos un recorrido por los pasos para configurar SnapCenter de NetApp para ejecutar Oracle en BareMetal Infrastructure.

## <a name="add-storage-hosts-to-snapcenter"></a>Adición de hosts de almacenamiento a SnapCenter

En primer lugar, vamos a agregar hosts de almacenamiento a SnapCenter. 

Cuando inicie la sesión en SnapCenter y guarde la exención de seguridad, se iniciará la aplicación. Inicie sesión en SnapCenter en la máquina virtual (VM) con las credenciales de Active Directory.

https://\<hostname\>:8146/

Ahora está listo para agregar tanto una ubicación de almacenamiento de producción como una ubicación de almacenamiento secundaria.

### <a name="add-the-production-storage-location"></a>Adición de la ubicación de almacenamiento de producción

Para agregar la máquina virtual de almacenamiento (SVM) de producción:

1. Seleccione **Storage Systems** (Sistemas de almacenamiento) en el menú de la izquierda y seleccione **+ New** (Nuevo).

2. Escriba la información en **Add Storage System** (Agregar sistema de almacenamiento):

      - Storage System (Sistema de almacenamiento): escriba la dirección IP de la SVM, proporcionada por Microsoft Operations.
      - Escriba el nombre de usuario creado; el valor predeterminado es **snap center**.
      - Escriba la contraseña creada cuando Microsoft Operations [modificó la contraseña del cliente mediante REST](set-up-snapcenter-to-route-traffic.md#steps-to-integrate-snapcenter).
      - Deje la opción **Send Autosupport notification for failed operations to storage system** (Enviar notificación de soporte automático para las operaciones con error al sistema de almacenamiento) desmarcada.
      - Seleccione **Log SnapCenter events to syslog** (Registrar eventos de SnapCenter en syslog).

3. Seleccione **Submit** (Enviar).

    Una vez comprobado el almacenamiento, la dirección IP del sistema de almacenamiento agregado se muestra con el nombre de usuario indicado.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/oracle-baremetal-snapcenter-add-production-storage-complete.png" alt-text="Captura de pantalla que muestra la dirección IP del sistema de almacenamiento agregado.":::

    El valor predeterminado es un SVM por inquilino. Si un cliente tiene varios inquilinos, se recomienda configurar todas las SVM aquí en SnapCenter.

### <a name="add-secondary-storage-location"></a>Adición de una ubicación de almacenamiento secundaria

Para agregar la ubicación de almacenamiento de recuperación ante desastres (DR), se deben emparejar las suscripciones de cliente tanto en la ubicación principal como en la de recuperación ante desastres. Para recibir asistencia, póngase en contacto con Microsoft Operations.

Agregar una ubicación de almacenamiento secundaria es similar a agregar la ubicación de almacenamiento principal. Sin embargo, una vez emparejadas las ubicaciones principal y de recuperación ante desastres, el acceso a la ubicación de almacenamiento secundaria debe comprobarse haciendo ping al almacenamiento en la ubicación secundaria. 

>[!NOTE]
>Si el ping no se completa correctamente, suele deberse a que no existe una ruta predeterminada en el host para la LAN virtual (VLAN) del cliente.

Una vez comprobado el ping, repita los pasos que siguió para agregar el almacenamiento principal, solo que ahora para la ubicación de recuperación ante desastres en un host de recuperación ante desastres. Se recomienda usar el mismo certificado en ambas ubicaciones, pero no es obligatorio.

## <a name="set-up-oracle-hosts"></a>Configuración de hosts de Oracle

>[!NOTE]
>Este proceso es para todos los hosts de Oracle, independientemente de su ubicación: producción o recuperación ante desastres.

1. Antes de agregar los hosts a SnapCenter e instalar los complementos de SnapCenter, Java 1.8 tiene que estar instalado. Antes de continuar, compruebe que Java 1.8 esté instalado en cada host.

2. Cree el mismo usuario no raíz en cada uno de los hosts de Oracle Real Application Clusters (RAC) y agréguelo a /etc/sudoers. Tendrá que especificar una nueva contraseña.

3. Una vez creado el usuario, debe agregarse a /etc/sudoers con un conjunto explícito de permisos. Para encontrar estos permisos, examine la siguiente ubicación en la VM de SnapCenter: C:\ProgramData\NetApp\SnapCenter\Package Repository, y abra el archivo oracle\_checksum.

4. Copie el conjunto de comandos adecuado, en función del paquete de sudo, donde LINUXUSER se reemplaza por el nombre de usuario recién creado. 

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/netapp-snapcenter-oracle-checksum-details.png" alt-text="Captura de pantalla de los detalles del archivo oracle_checksum.":::

    El conjunto de comandos para el paquete de sudo 1.8.7 o superior se copia en /etc/sudoers.

5. Una vez que se haya agregado el usuario a sudoers, en SnapCenter, seleccione **Settings** (Configuración) > **Credential** (Credencial) >  **New** (Nuevo).

6. Rellene el cuadro Credential (Credencial) de la forma siguiente:

    - **Credential Name** (Nombre de credencial): Escriba un nombre que identifique el nombre de usuario y sudoers.
    - **Authentication** (Autenticación): Linux.
    - **Username** (Nombre de usuario): Escriba el nombre de usuario recién creado.
    - **Contraseña**: <Enter Password>
    - Active la casilla **Use sudo privileges** (Usar privilegios de sudo).
    
7. Seleccione **Aceptar**.

8. Seleccione **Hosts** en el panel de navegación izquierdo y, después seleccione **+ Add** (Agregar).

9. Escriba los siguientes valores para **Add Host** (Agregar host):

    - **Host Type** (Tipo de host): Linux
    - **Host Name** (Nombre de host): Escriba el nombre de dominio completo del host RAC principal o la dirección IP del host RAC principal.
    - **Credentials** (Credenciales): Seleccione la lista desplegable y elija las nuevas credenciales creadas para sudoers.
    - Active la casilla **Oracle Database**.

    >[!NOTE]
    >Tiene que escribir una de las direcciones IP del host de Oracle reales. No se admite indicar una VIP de nodo ni IP de SCAN.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/snapcenter-add-hosts-details.png" alt-text="Captura de pantalla que muestra los detalles del nuevo host.":::
    
10. Seleccione **More Options** (Más opciones). Asegúrese de que **Add all hosts in the Oracle RAC** (Agregar todos los hosts en Oracle RAC) esté seleccionado. Seleccione **Save** (Guardar) y, luego, **Submit** (Enviar).

11. El instalador de complementos ahora intentará comunicarse con la dirección IP proporcionada. Si la comunicación se establece correctamente, la identidad del host de Oracle RAC proporcionado se confirma al seleccionar **Confirm and Submit** (Confirmar y enviar).

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/snapcenter-add-hosts-confirm-fingerprint.png" alt-text="Captura de pantalla que muestra la huella digital del nuevo host.":::

12. Una vez confirmado el host de Oracle RAC inicial, SnapCenter intentará comunicarse y confirmar todos los demás servidores de Oracle RAC como parte del clúster. Seleccione **Confirm Others and Submit** (Confirmar otros y enviar).

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/snapcenter-add-hosts-fingerprint-authentication.png" alt-text="Captura de pantalla que muestra la autenticación de la huella digital.":::

    Aparece una pantalla de estado para los hosts administrados que le permite saber que SnapCenter está instalando el complemento de Oracle seleccionado. Para comprobar el progreso de la instalación, consulte los registros ubicados en /opt/NetApp/snapcenter/scc/logs/DISCOVER-PLUGINS\_0.log en cualquiera de los hosts de Oracle RAC. 

    Cuando los complementos se hayan instalado correctamente, aparecerá la siguiente pantalla.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/snapcenter-installed-plugins.png" alt-text="Captura de pantalla que muestra todos los complementos de SnapCenter instalados.":::

## <a name="add-credentials-for-oracle-recovery-manager"></a>Adición de credenciales para Oracle Recovery Manager

El método de autenticación del catálogo de Oracle Recovery Manager (RMAN) permite autenticarse según la base de datos del catálogo de RMAN. Si ha configurado un mecanismo de catálogo externo y ha registrado la base de datos para catalogar una base de datos, debe agregar la autenticación del catálogo de RMAN. Las credenciales de RMAN se pueden agregar en cualquier momento.

1. En SnapCenter, seleccione **Settings** (Configuración) en el menú de la izquierda y, a continuación, seleccione **Credential** (Credencial). Seleccione **New** (Nuevo) en la esquina superior derecha.

2. Escriba **Credential Name** (Nombre de la credencial) para llamar a las credenciales de RMAN almacenadas en SnapCenter. En la lista desplegable **Authentication** (Autenticación), seleccione **Oracle RMAN Catalog for Authentication** (Catálogo de Oracle RMAN para la autenticación). Escriba su nombre de usuario y contraseña. Seleccione **Aceptar**.

3. Una vez agregadas las credenciales, la configuración de la base de datos debe modificarse en SnapCenter. Seleccione la base de datos en los recursos. Seleccione **Database Settings** (Configuración de la base de datos) en la esquina superior derecha de la ventana principal.

4. En la pantalla Database Settings, seleccione **Configure Database** (Configurar base de datos).

5. Expanda **Configure RMAN Catalog Settings** (Configurar opciones del catálogo de RMAN). Seleccione la lista desplegable **Use Existing Credential** (Usar credencial existente) establecida anteriormente para esta base de datos y seleccione la opción adecuada. Agregue el nombre de TNS para esta base de datos individual. Seleccione **Aceptar**.

    >[!NOTE]
    >Se deben crear credenciales diferentes en el paso anterior para cada combinación única de nombre de usuario y contraseña. Si lo prefiere, SnapCenter aceptará un único conjunto de credenciales para todas las bases de datos.
    >
    >Aunque se hayan agregado credenciales de RMAN a la base de datos, RMAN no se cataloga a menos que Catalog backup with Oracle Recovery Manager (RMAN) (Copia de seguridad del catálogo con Oracle Recovery Manager [RMAN]) también esté seleccionada para cada directiva de protección, como se describe en la sección siguiente sobre la creación de directivas de protección (según las directivas de copia de seguridad).

## <a name="create-protection-policies"></a>Creación de directiva de protección

Una vez que los hosts se hayan agregado correctamente a SnapCenter, estará listo para crear las directivas de protección. 

Seleccione **Resources** (Recursos) en el menú izquierdo. SnapCenter presentará todas las bases de datos identificadas que existían en los hosts de RAC. Los tipos de instancia única para bn6p1X y grid se omiten como parte del esquema de protección. El estado de todas debe ser "Not Protected" (Sin protección) en este momento.

Como se describe en la [información general](netapp-snapcenter-integration-oracle-baremetal.md#oracle-disaster-recovery-architecture), los distintos tipos de archivo tienen diferentes frecuencias de instantáneas que permiten un RPO local de aproximadamente 15 minutos al hacer copias de seguridad de registros de archivo cada 15 minutos. A continuación, se hace una copia de seguridad de los archivos de datos a intervalos más largos, como cada una hora o más. Por lo tanto, se crean dos directivas diferentes.

Con las bases de datos de RAC identificadas, se crean varias directivas de protección, incluida una directiva para archivos de datos y archivos de control, y otra para los registros de archivo.

### <a name="datafiles-protection-policy"></a>Directiva de protección de archivos de datos

Siga estos pasos para crear una directiva de protección de archivos de datos.

1. En SnapCenter, seleccione **Settings** (Configuración) en el menú de la izquierda y, a continuación, seleccione **Policies** (Directivas) en el menú superior. Seleccione **Nuevo**.

2. Seleccione el botón de radio para **Datafiles and control files** (Archivos de datos y archivos de control) y desplácese hacia abajo.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/snapcenter-new-database-backup-policy.png" alt-text="Captura de pantalla que muestra los detalles de la nueva directiva de copia de seguridad de base de datos.":::

3. Seleccione el botón de radio para **Hourly** (Cada hora). Si se quiere la integración con RMAN con fines de catálogo y ya se han agregado las credenciales de RMAN, active la casilla **Catalog backup with Oracle Recovery Manager (RMAN)** (Copia de seguridad del catálogo con Oracle Recovery Manager [RMAN]). Seleccione **Next** (Siguiente). Si la copia de seguridad del catálogo está seleccionada, [se deben agregar credenciales de RMAN](#add-credentials-for-oracle-recovery-manager) para que se produzca la catalogación.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/snapcenter-database-backup-policy-frequency.png" alt-text="Captura de pantalla que muestra las opciones para elegir la frecuencia de programación en la directiva de copia de seguridad.":::

    >[!IMPORTANT]
    >Se debe hacer una selección para la directiva de programación, incluso si se quiere una frecuencia de copia de seguridad que no sea cada hora o diaria, etc. La frecuencia de copia de seguridad real se establece más adelante en el proceso. No seleccione "None" (Ninguna), a menos que todas las copias de seguridad de esta directiva sean solo a petición.

4. Seleccione **Retention** (Retención) en el menú de la izquierda. Hay dos tipos de valores de retención que se establecen para una directiva de copia de seguridad. El primer valor de retención es el número máximo de instantáneas a petición que se tienen que conservar. En función de las directivas de copia de seguridad, se mantiene un número establecido de instantáneas, como el ejemplo siguiente, donde 48 instantáneas a petición se mantienen durante 14 días. Establezca la directiva adecuada de retención a petición según las directivas de copia de seguridad.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/snapcenter-new-database-backup-policy-retention.png" alt-text="Captura de pantalla que muestra la configuración de retención de la directiva de copia de seguridad de bases de datos.":::

5. El siguiente conjunto de valores de retención es el número programado de instantáneas que se mantienen en función de la entrada anterior, ya sea cada hora, diaria, semanal, etc. Seleccione **Next** (Siguiente).

6. En la pantalla **Replication** (Replicación), active la casilla **Update SnapMirror after creating a local snapshot copy** (Actualizar SnapMirror después de crear una copia de instantánea local) y seleccione **Next** (Siguiente). Las demás entradas se dejan en sus valores predeterminados.

    >[!NOTE]
    >Actualmente, SnapVault no se admite en el entorno de Oracle BareMetal Infrastructure.

    Para agregar la replicación más adelante, puede volver a la pantalla Protection Policy (Directiva de protección) y seleccionar **Modify protection policy** (Modificar directiva de protección) y **Replication** (Replicación) en el menú de la izquierda.

7. La página **Script** es donde se escriben los scripts que tienen que ejecutarse antes o después de que se haga la copia de seguridad de Oracle. Actualmente, no se admiten scripts como parte del proceso de SnapCenter. Seleccione **Next** (Siguiente).

8. Seleccione **Verification** (V) en el menú de la izquierda. Si quiere tener la posibilidad de verificar la integridad de la capacidad de recuperación de las instantáneas, active la casilla junto a Hourly (Cada hora). Actualmente no se admiten comandos de script de verificación. Seleccione **Next** (Siguiente).

    >[!NOTE]
    >La ubicación y la programación reales del proceso de verificación se agregan en una sección posterior, Asignación de directivas de protección a los recursos.

9. En la página **Summary** (Resumen), compruebe que todos los valores se hayan especificado según lo previsto y seleccione **Finish** (Finalizar).

### <a name="archive-logs-protection-policy"></a>Directiva de protección de registros de archivo

Siga los pasos anteriores de nuevo para crear la directiva de protección de registros de archivo. En el paso 2, seleccione el botón de radio para **Archive logs** (Registros de archivo) lugar de "Datafiles and control files" (Archivos de datos y archivos de control).

## <a name="assign-protection-policies-to-resources"></a>Asignación de directivas de protección a los recursos

Una vez creada una directiva, tiene que asociarse a un recurso para que SnapCenter empiece a ejecutarse dentro de esa directiva.

### <a name="datafiles-resource-group"></a>Grupo de recursos de archivos de datos

1. Seleccione **Resources** (Recursos) en el menú de la izquierda y, a continuación, seleccione **New Resource Group** (Nuevo grupo de recursos) en la esquina superior derecha de la ventana principal.

2. Agregue el nombre de este grupo de recursos en **Name** y las etiquetas para facilitar la capacidad de búsqueda.

    >[!NOTE]
    >Se recomienda seleccionar **Use custom name format for Snapshot copy** (Usar formato de nombre personalizado para la copia de instantáneas) y agregar las siguientes entradas: **$ResourceGroup**, **$Policy** y **$ScheduleType**. Esto garantiza que el prefijo de instantánea cumpla el estándar de SnapCenter y que el nombre de la instantánea brinde los detalles necesarios de un vistazo. Si deja desactivada la opción **Use custom name format for Snapshot copy** (Usar formato de nombre personalizado para copia de instantáneas), cualquier entrada que se escriba en **Name** (Nombre) se convertirá en el prefijo de las instantáneas creadas. Asegúrese de que el nombre especificado identifique la base de datos y lo que se protege, por ejemplo, los archivos de datos y los archivos de control. 

3. En Backup settings (Configuración de copia de seguridad), agregue **/u95** para excluir los registros de archivo.

4. En la página Resource (Recurso), mueva todas las bases de datos protegidas por la misma directiva de protección de copia de seguridad de "Available Resources" (Recursos disponibles) a "Selected Resources" (Recursos seleccionados). No agregue las instancias de base de datos de Oracle para cada host, solo las bases de datos. Seleccione **Next** (Siguiente).

5. Seleccione la directiva de protección para los archivos de datos y los archivos de control que creó anteriormente. Después de seleccionar la directiva, seleccione el lápiz en Configure schedules (Configurar programaciones).

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/new-datafiles-resource-group-available-policies.png" alt-text="Captura de pantalla que muestra la selección de la directiva de protección para configurar las programaciones.":::

6. Agregue la programación para que se ejecute la directiva seleccionada. Asegúrese de que la fecha de inicio sea posterior a la fecha y hora actuales.

    >[!NOTE]
    >No es necesario que la frecuencia programada coincida con la frecuencia seleccionada al crear la directiva. Para Hourly (Cada hora), se recomienda dejar "Repeat every 1 hours" (Repetir cada 1 hora). La hora de inicio servirá como hora de inicio cada hora en que se produzca la copia de seguridad. Asegúrese de que la programación para proteger los archivos de datos no se superponga con la programación para proteger los registros de archivo, ya que solo una copia de seguridad puede estar activa en un momento dado.

7. La verificación garantiza que la instantánea creada se pueda usar. El proceso de verificación es amplio, incluida la creación de clones de todos los volúmenes de base de datos de Oracle, el montaje de la base de datos en el host principal y la verificación de su capacidad de recuperación. Este proceso consume tiempo y recursos. Si quiere configurar la programación de verificación, seleccione el signo **+** .

    >[!NOTE]
    >El proceso de verificación ocupa recursos en el host durante una gran cantidad de tiempo. Se recomienda que, si agrega la verificación, lo haga en un host de la ubicación secundaria, si está disponible.
    
    En la captura de pantalla siguiente se muestra la verificación de un nuevo grupo de recursos que no tenía habilitada la replicación, y las instantáneas replicadas cuando se creó. 

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/new-resource-group-verification-schedules.png" alt-text="Captura de pantalla que muestra cómo configurar la programación de verificación para el grupo de recursos.":::

    Si la replicación está habilitada y se usará un host en la ubicación de recuperación ante desastres para la verificación, vaya al paso 2 en el siguiente apartado. Este paso le permite cargar localizadores secundarios para verificar las copias de seguridad en la ubicación secundaria.

### <a name="add-verification-schedule-for-new-resource-group"></a>Adición de una programación de verificación para el nuevo grupo de recursos

1. Seleccione **Run verification after backup** (Ejecutar verificación después de la copia de seguridad) o **Run scheduled verification** (Ejecutar verificación programada) y seleccione una frecuencia programada previamente de la lista desplegable. Si la recuperación ante desastres está habilitada, puede seleccionar **Verify on secondary storage location** (Verificar en la ubicación de almacenamiento secundaria) para ejecutar la frecuencia en la ubicación de recuperación ante desastres, lo que reduce la presión de recursos en producción. Seleccione **Aceptar**.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/new-resource-group-add-verification-schedules.png" alt-text="Captura de pantalla que muestra cómo agregar una programación de verificación.":::

    Si la verificación no es necesaria o ya está configurada localmente, vaya directamente a la configuración de SMTP (paso 5) más adelante.

2. Si se ha habilitado la actualización de SnapMirror al crear la directiva de protección de archivos de datos y se ha agregado una ubicación de almacenamiento secundaria, SnapCenter detecta la replicación entre las dos ubicaciones. En ese caso, aparece una pantalla que le permite cargar localizadores secundarios para verificar las copias de seguridad en la ubicación secundaria. Seleccione **Load locators** (Localizadores de carga).

3. Después de seleccionar los localizadores de carga, SnapCenter presentará las relaciones de SnapMirror que encontró que contienen los archivos de datos y los archivos de control. Deben coincidir con el marco de recuperación ante desastres indicado por Microsoft Operations. Seleccione el lápiz en Configure Schedules (Configurar programaciones).

4. Active la casilla **Verify on secondary storage location** (Verificar en la ubicación de almacenamiento secundaria). 

5. Si hay disponible un servidor SMTP, SnapCenter puede enviar un correo electrónico a los administradores de SnapCenter. Escriba lo siguiente si quiere que se envíe un correo electrónico.

    - **Email Preference** (Preferencia de correo electrónico): Escriba su preferencia para la frecuencia de recepción de correos electrónicos.
    - **From** (De): Escriba la dirección de correo electrónico que SnapCenter usará para enviar correos electrónicos.
    - **To** (Para): Escriba la dirección de correo electrónico a la que SnapCenter enviará correos electrónicos.
    - **Subject** (Asunto): Escriba el asunto que SnapCenter usará al enviar el correo electrónico.
    - Seleccione la casilla **Attach job report** (Adjuntar informe de trabajo).
    - Seleccione **Next** (Siguiente).

6. Revise la configuración en la página Summary (Resumen) y seleccione **Finish** (Finalizar).

Una vez creado un grupo de recursos, para identificar el grupo de recursos:

1. Seleccione **Resources** (Recursos) en el menú izquierdo.
2. Seleccione el menú desplegable de la ventana principal junto a View (Ver) y seleccione **Resource Group** (Grupo de recursos).
 
    Si este es su primer grupo de recursos, solo estará presente el grupo de recursos recién creado para los archivos de datos y archivos de control. Si además ha creado un grupo de recursos de registros de archivo, también lo verá.

### <a name="archive-log-resource-group"></a>Grupo de recursos de registros de archivo

Para asignar directivas de protección al grupo de recursos de registros de archivo, siga los mismos pasos que siguió al asignar directivas de protección al grupo de recursos de archivos de datos. Las únicas diferencias son:

- En el paso 3, no agregue **/u95** para excluir los registros de archivo; deje este campo en blanco.
- En el paso 6, se recomienda hacer una copia de seguridad de los registros de archivo cada 15 minutos.
- La pestaña de verificación está en blanco para los registros de archivo. 

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre la creación de una copia de seguridad a petición de Oracle Database con SnapCenter:

> [!div class="nextstepaction"]
> [Crear copias de seguridad a petición en SnapCenter](create-on-demand-backup-oracle-baremetal.md)
