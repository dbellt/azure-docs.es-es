---
title: Restauración de Oracle Database
description: Obtenga información sobre cómo restaurar Oracle Database en BareMetal Infrastructure mediante SnapCenter.
ms.topic: how-to
ms.subservice: baremetal-oracle
ms.date: 05/07/2021
ms.openlocfilehash: 3d7f417b6881fd44d67011d33e4fdde87ff2a6e1
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576372"
---
# <a name="restore-oracle-database"></a>Restauración de Oracle Database

En este artículo, veremos los pasos para restaurar una instancia de Oracle Database en BareMetal Infrastructure mediante SnapCenter.

Tiene varias opciones para restaurar Oracle Database en BareMetal Infrastructure. Se recomienda consultar la [Matriz de restauración de Oracle](oracle-high-availability-recovery.md#choose-your-method-of-recovery) antes de llevar a cabo la restauración. Esta matriz puede ayudarle a elegir el método de restauración más adecuado dado el tiempo de recuperación y la posible pérdida de datos. 

En general, se restauran las instantáneas más actuales para los volúmenes de registro de datos y archivos, ya que el objetivo es restaurar la base de datos al último punto de recuperación conocido. La restauración de instantáneas es un proceso permanente. 

>[!IMPORTANT]
>Es necesario tener mucho cuidado para asegurarse de restaurar la instantánea adecuada. La restauración a partir de una instantánea elimina todas las demás instantáneas y sus datos asociados. Esto incluye incluso las instantáneas más recientes que la que seleccione para la restauración. Por lo tanto, se recomienda ser conservador al abordar el proceso de restauración. No dude en usar una instantánea más reciente en primer lugar si hay alguna inquietud sobre qué instantánea se debe recuperar.

## <a name="restore-database-locally-with-restored-archive-logs"></a>Restauración local de la base de datos con registros de archivo restaurados

Antes de intentar la recuperación, la base de datos debe desconectarse si aún no lo está. Una vez que haya comprobado que la base de datos no se está ejecutando en ningún nodo de Oracle Real Application Clusters (RAC), está listo para comenzar. En primer lugar, restaurará los registros de archivo. 

1. Identifique las copias de seguridad disponibles. En SnapCenter, seleccione **Resources** (Recursos) en el menú de la izquierda.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/restore-database-available-backups.png" alt-text="Captura de pantalla que muestra la vista de bases de datos.":::

2. Seleccione de la lista la base de datos que quiere restaurar. La base de datos contendrá la lista de grupos de recursos creados y sus directivas asociadas.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/available-backups-list.png" alt-text="Captura de pantalla que muestra la lista completa de copias de seguridad disponibles para restaurar.":::

3. Luego se muestra una lista de copias de seguridad principales. Las copias de seguridad se identifican por su nombre de copia de seguridad y su tipo: Log (Registro) o Data (Datos). Controle primero la restauración de registros, ya que SnapCenter no está diseñado para restaurar directamente los volúmenes de registro de archivos. Identifique el volumen de registro de archivos que requiere restauración. Normalmente, el volumen que se va a restaurar es la copia de seguridad más reciente, ya que la mayoría de las recuperaciones pondrán al día todos los registros de archivo de la última copia de seguridad de archivos de datos correcta conocida.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/primary-backups-list.png" alt-text="Captura de pantalla que muestra una lista de las copias de seguridad principales.":::

4. Después de seleccionar el volumen de registro de archivos, se habilita la opción de montaje en la esquina superior derecha de la lista de copia de seguridad. Seleccione **Montar**. En la página Mount backups (Montar copias de seguridad), en la lista desplegable, seleccione el host que montará la copia de seguridad. Se puede seleccionar cualquier nodo RAC, pero el host de recuperación tanto de los registros de archivos como de los archivos de datos debe ser el mismo. Copie la ruta de acceso de montaje, ya que se usará en el próximo paso para recuperar los archivos de datos. Seleccione **Montar**.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/restore-database-mount-backups.png" alt-text="Captura de pantalla que muestra las copias de seguridad que se van a montar.":::

5. A diferencia de los trabajos de copia de seguridad, el visor de trabajos de esta página no muestra el estado del proceso de montaje. Para ver el estado del montaje, seleccione **Monitor** (Supervisión) en el menú izquierdo. A continuación, se resaltará el estado de todos los trabajos que se han ejecutado. La operación de montaje debe ser la primera entrada. También puede navegar a la ruta de acceso de montaje copiada para comprobar la existencia de los registros de archivo.

    El host seleccionado para montar ese sistema de archivos también recibirá una entrada en /etc/fstab con la ruta de acceso de montaje anterior. Se puede quitar una vez que se complete la recuperación.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/restore-database-mount-output.png" alt-text="Captura de pantalla que muestra la salida del montaje.":::

6. A continuación, restaure los archivos de datos y los archivos de control. Como antes, seleccione **Resources** (Recursos) en el menú de la izquierda y seleccione la base de datos que se va a restaurar. Seleccione la copia de seguridad de datos que se va a restaurar. De nuevo, normalmente la copia de seguridad restaurada es la más reciente. Esta vez, cuando seleccione la copia de seguridad de datos, se habilitará la opción de restauración. Seleccione **Restaurar**.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/restore-database-selected-data-backup.png" alt-text="Captura de pantalla de la copia de seguridad de datos que se va a restaurar.":::

7. En la pestaña **Restore Scope** (Ámbito de restauración), en la lista desplegable, seleccione el mismo host que eligió anteriormente para montar los archivos de registro. Asegúrese de que **All Datafiles** (Todos los archivos de datos) esté seleccionado y active la casilla **Control files** (Archivos de control). Seleccione **Next** (Siguiente).

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/restore-database-recovery-scope.png" alt-text="Captura de pantalla que muestra los detalles del ámbito de restauración de los archivos de datos.":::

8. En la pestaña **Recovery Scope** (Ámbito de recuperación), se indica el número de cambio del sistema (SCN) de la copia de seguridad del registro que se ha montado. **Until SCN** (Hasta SC) ahora también está seleccionado. 
 
    El SCN se puede encontrar en la lista de todas las copias de seguridad para cada copia de seguridad. La ubicación del punto de montaje copiada anteriormente también se especifica en **Specify external archive log files locations** (Especificar ubicaciones externas de archivos de registro de archivo). Actualmente, el abordaje de más de una ubicación de restauración de registros de archivo está fuera del ámbito de este documento. Seleccione **Next** (Siguiente).

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/restore-database-choose-recovery-scope.png" alt-text="Captura de pantalla que muestra las opciones de ámbito de recuperación.":::

9. En la pestaña **PreOps**, actualmente no se recomienda ningún script como parte del proceso previo a la restauración. Seleccione **Next** (Siguiente).

10. En la pestaña PostOps, de nuevo, no se admite ningún script como parte del proceso posterior a la restauración. Active la casilla **Open the database or container database in READ-WRITE mode after recovery** (Abrir la base de datos o la base de datos de contenedor en modo LECTURA-ESCRITURA después de la recuperación). Seleccione **Next** (Siguiente).

11. En la pestaña **Notification** (Notificación), si hay disponible un servidor SMTP y quiere recibir una notificación por correo electrónico del proceso de restauración, rellene la opción de correo electrónico. Seleccione **Next** (Siguiente).

12. En la pestaña **Summary** (Resumen), compruebe que todos los detalles sean correctos. Seleccione **Finalizar**.

13. Para ver el estado de restauración, seleccione el trabajo de restauración en la pantalla inferior **Activity** (Actividad). Para seguir el progreso, puede seleccionar las flechas verdes para abrir cada subsección de restauración y su progreso.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/restore-job-details.png" alt-text="Captura de pantalla que muestra los detalles del trabajo de restauración.":::

    Una vez que se complete la restauración, todos los pasos cambiarán a marcas de verificación. Seleccione **Cerrar**.

    Las filas quitadas de la base de datos simple creada ahora se verifican como restauradas.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/output-of-restored-files.png" alt-text="Captura de pantalla que muestra la verificación de salida de los archivos restaurados.":::

## <a name="clone-database-locally-or-remotely"></a>Clonación de la base de datos de manera local o remota

La clonación de la base de datos es similar al proceso de restauración. La operación de clonación de una base de datos se puede usar de manera local o remota en función del resultado que se quiera. La clonación de la base de datos en la ubicación de recuperación ante desastres (DR) es útil para varios fines, incluidas las pruebas de conmutación por error de recuperación ante desastres y el almacenamiento provisional para control de calidad de los datos de producción. Se supone que la ubicación de recuperación ante desastres se usa para pruebas, desarrollo, control de calidad, entre otros.

### <a name="create-a-clone"></a>Creación de un clon

La creación de un clon es una característica de SnapCenter que permite usar una instantánea como referencia a un momento dado para capturar un conjunto similar de datos entre el volumen primario y el volumen clonado mediante punteros. A continuación, se permiten las lecturas y escrituras en el volumen clonado, que se expande solo mediante escrituras, mientras que las lecturas siguen teniendo lugar en el volumen primario. Esta característica permite crear un conjunto "duplicado" de datos a disposición de un host sin interferir con los datos que existen en el volumen primario. 

Esto resulta especialmente útil para las pruebas de recuperación ante desastres, ya que un sistema de archivos temporal se puede poner en marcha en función de las mismas instantáneas que se usarán en una recuperación real. Puede comprobar los datos y que las aplicaciones funcionen según lo previsto y, a continuación, apagar la prueba de recuperación ante desastres, sin afectar a los volúmenes de recuperación ante desastres ni a la replicación.

Estos son los pasos para clonar una base de datos. 

1. En SnapCenter, seleccione **Resources** (Recursos) y, después, seleccione la base de datos que quiere clonar. Si va a crear el clon localmente, vaya al paso siguiente. Si va a restaurar el clon en la ubicación secundaria, seleccione las copias de seguridad encima del cuadro Mirror Copies (Copias reflejadas).

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/clone-database-manage-copies.png" alt-text="Diagrama de copias administradas de bases de datos clonadas.":::

2. Seleccione la copia de seguridad de datos adecuada de la lista de copias de seguridad proporcionada. Seleccione **Clonar**.

    >[!NOTE]
    >La copia de seguridad de datos se debe haber creado antes que la marca de tiempo o el número de cambio del sistema (SCN) si se requiere un clon de marca de tiempo. 

3. En la pestaña **name** (Nombre), escriba el nombre del **SID** del clon. Compruebe que **Source Volume** (Volumen de origen) y **Destination Volume** (Volumen de destino) para los registros de archivos y datos coinciden con lo que Microsoft Operations indicó para la asignación de recuperación ante desastres. Seleccione **Next** (Siguiente).

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/provide-clone-database-sid.png" alt-text="Captura de pantalla que muestra dónde escribir el SID de la base de datos.":::

4. En la pestaña **Locations** (Ubicaciones), las ubicaciones de los archivos de datos, los archivos de control y los registros de rehacer están donde la operación de clonación creará los sistemas de archivos necesarios para la recuperación. Se recomienda dejarlas tal y como están. La exploración de alternativas está fuera del ámbito de este documento. Asegúrese de que el host adecuado esté seleccionado para la ubicación de la restauración. Seleccione **Next** (Siguiente).

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/clone-database-select-host.png" alt-text="Captura de pantalla que muestra cómo seleccionar un host para clonar.":::

5. En la pestaña **Credentials** (Credenciales), los valores de Credentials (Credenciales) y Oracle Home Settings (Configuración de Oracle Home) se extraen de la ubicación de producción existente. Se recomienda dejarlos tal y como están, a menos que sepa que la ubicación secundaria tiene valores diferentes a la ubicación principal. Seleccione **Next** (Siguiente).

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/clone-database-credentials.png" alt-text="Captura de pantalla para especificar las credenciales de la base de datos que se va a clonar.":::

6. En la pestaña **PreOps**, actualmente no se admiten scripts previos a la recuperación. Seleccione **Next** (Siguiente).

7. En la pestaña **PostOps**, si la base de datos solo se recuperará hasta un momento o SCN dados, seleccione el botón de radio adecuado y agregue la marca de tiempo o el SCN. SnapCenter recuperará la base de datos hasta que llegue a ese punto. De lo contrario, deje seleccionada la opción **Until Cancel** (Hasta cancelar). Seleccione **Next** (Siguiente).

8. En la pestaña **Notification** (Notificación), escriba la información de SMTP necesaria si quiere enviar un correo electrónico de notificación cuando se complete la clonación. Seleccione **Next** (Siguiente).

9. En la pestaña **Summary** (Resumen), compruebe que se ha indicado el valor de **Clone SID** (SID de clonación) adecuado y se ha seleccionado el host correcto. Desplácese hacia abajo para asegurarse de que se ha especificado el ámbito de recuperación adecuado. Seleccione **Finalizar**.

10. El trabajo de clonación se mostrará en el menú emergente activo en la parte inferior de la pantalla. Seleccione la actividad de clonación para mostrar los detalles del trabajo. Una vez que haya finalizado la actividad, la página Job Details (Detalles del trabajo) mostrará todas las marcas de verificación verdes e indicará la hora de finalización. La clonación normalmente tarda entre 7 y 10 minutos.

11. Una vez que se complete el trabajo, cambie al host que se usa como destino para el clon y compruebe los puntos de montaje mediante cat /etc/fstab. Esta comprobación garantiza que existan los puntos de montaje adecuados para la base de datos que se enumeraron durante el asistente para la clonación, y también resalta el SID de base de datos especificado en el asistente. En el ejemplo siguiente, el SID es dbsc4, como lo indican los puntos de montaje del host.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/clone-database-switch-to-target-host.png" alt-text="Captura de pantalla del comando para cambiar al host de destino.":::

12. En el host, escriba **oraenv** y presione **ENTRAR**. Escriba el SID creado recientemente.

    Depende del usuario comprobar que la base de datos se restaure correctamente. Sin embargo, los siguientes pasos adicionales se basan en la base de datos creada por un usuario que no sea Oracle. 

13. Escriba **sqlplus / as sysdba**. Puesto que la tabla se creó con otro usuario, se escriben automáticamente el nombre de usuario y la contraseña originales no válidos. Escriba el nombre de usuario y la contraseña correctos. El símbolo de SQL> aparecerá una vez que el inicio de sesión se haya completado correctamente.

Escriba una consulta de la base de datos básica para comprobar que se reciban los datos adecuados. En el ejemplo siguiente, hemos usado los registros de archivo para poner al día la base de datos. A continuación se muestra que los registros de archivo se usaron correctamente, ya que la entrada clonetest se creó después de crear la copia de seguridad de datos. Por lo tanto, si los registros de archivo no se han puesto al día, esa entrada no aparece en la lista.

```sql
SQL> select * from acolvin.t;

COL1
-----------------------------------------
COL2
---------------
first insert
17-DEC-20

log restore
17-DEC-20

clonetest
18-DEC-20

COL1
-----------------------------------------
COL2
---------------
after first insert
17-DEC-20

next insert
17-DEC-20

final insert
18-DEC-20

COL1
-----------------------------------------
COL2
---------------
BILLY
17-DEC-20

7 rows selected.

```

### <a name="delete-a-clone"></a>Eliminación de un clon

Es importante eliminar un clon una vez que haya terminado con él. Si tiene previsto seguir usándolo, debe dividir el clon. No se puede eliminar una instantánea que es el elemento primario de un clon, y se omite como parte del recuento de retención. Si no elimina los clones a medida que termina con ellos, el número de instantáneas que mantenga podría usar un almacenamiento excesivo. 

Un clon dividido copia los datos del volumen y la instantánea existentes en un nuevo volumen. Este proceso desactiva la relación entre el clon y la instantánea primaria, lo que permite eliminar la instantánea cuando se alcanza su número de retención. Si ya se ha alcanzado el número de retención, la instantánea se eliminará en la siguiente instantánea. Aun así, un clon dividido también incurre en costos de almacenamiento.

Cuando se crea un clon, la pestaña de recursos de esa base de datos mostrará un clon presente, ya sea de manera local o remota. Para eliminar el clon:

1. En la pestaña **Resources** (Recursos), seleccione el cuadro que contiene el clon que quiere eliminar.

2. El clon aparece en la página Secondary Mirror Clone(s) (Clones reflejados secundarios). En este ejemplo, el clon está en la ubicación secundaria. Seleccione **Delete** (Eliminar) en la esquina superior derecha de la lista de clones.

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/delete-clone-secondary-mirror-clones.png" alt-text="Captura de pantalla de los clones reflejados secundarios.":::

3. Antes de la ejecución, asegúrese de haber salido de SQLPLUS. Seleccione **Aceptar**.

4. Para ver el progreso del trabajo, seleccione **Jobs** (Trabajos) en el menú de la izquierda. Una marca de verificación verde muestra cuando se ha eliminado el clon.

5. Una vez que se haya eliminado el clon, también puede ser útil desmontar los registros de archivo que se usaron como parte del proceso de clonación, si procede. De nuevo en la lista de copias de seguridad adecuadas (de donde creó el clon), la lista de copias de seguridad se puede ordenar según si una copia de seguridad se ha montado o no. El algoritmo de ordenación no es perfecto y no siempre ordenará todos los elementos juntos, pero en general el orden será el correcto. El volumen siguiente se montó previamente. Seleccione **Unmount** (Desmontar).

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/delete-clone-unmount-backup.png" alt-text="Captura de pantalla que muestra la lista de copias de seguridad.":::

6. Seleccione **Aceptar**. Para ver el estado del trabajo de desmontaje, seleccione **Monitor** (Supervisión) el menú de la izquierda. Aparecerá una marca de verificación verde cuando se desmonte la copia de seguridad.

### <a name="split-a-clone"></a>División de un clon

La división de un clon crea una copia del volumen primario mediante la replicación de todos los datos del volumen primario hasta el momento en que se creó la instantánea usada para crear el clon. Este proceso separa el volumen primario del volumen clonado y quita la retención en la instantánea usada para crear el volumen clonado. A continuación, la instantánea se puede eliminar como parte de la directiva de retención.

Dividir un clon resulta útil para rellenar los datos en el entorno de producción o en el entorno de recuperación ante desastres. La división permite que los nuevos volúmenes funcionen independientemente del volumen primario.

>[!NOTE]
>El proceso de división de un clon no se puede invertir ni cancelar.

Para dividir un clon:

1. Seleccione la base de datos que ya contiene un clon.

2. Una vez que haya seleccionado la ubicación del clon, se muestra en la lista de la página Secondary Mirror Clone(s) (Clones reflejados secundarios).

3. Justo encima de la lista de clones, seleccione **Clone Split** (División de clones). SnapCenter no permite ningún cambio como parte del proceso de división, por lo que en la página siguiente simplemente se muestran los cambios que tendrán lugar. Seleccione **Inicio**.

    >[!NOTE]
    >El proceso de división puede tardar bastante tiempo en función de la cantidad de datos que se tienen que copiar, del diseño de la base de datos en el almacenamiento y del nivel de actividad de almacenamiento.

Una vez finalizado el proceso, el clon que se dividió se quita de la lista de copias de seguridad, y la instantánea asociada al clon ahora es libre de quitarse como parte del plan normal de retención en SnapCenter.

## <a name="restore-database-remotely-after-disaster-recovery-event"></a>Restauración remota de una base de datos después de un evento de recuperación ante desastres

Actualmente, SnapCenter no está diseñado para automatizar el proceso de conmutación por error. Si se produce un evento de recuperación ante desastres, se requieren scripts de recuperación de REST o de Microsoft Operations para restaurar la base de datos en la ubicación secundaria. El detalle del proceso de ejecución de los scripts de recuperación de REST está fuera del ámbito de este documento actualmente.

## <a name="restart-all-rac-nodes-after-restore"></a>Reinicio de todos los nodos de RAC después de una restauración

Después de restaurar la base de datos en SnapCenter, solo está activa en el nodo de RAC que seleccionó al restaurar la base de datos, como se muestra a continuación. En este ejemplo, restauramos la base de datos en bn6sc2 para demostrar que puede seleccionar cualquier nodo de RAC para completar la restauración.

:::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/restore-database-example-output-rac-node.png" alt-text="Captura de pantalla que muestra que la base de datos se restauró en bn6sc2.":::

Inicie los nodos de RAC restantes mediante el comando _srvctl start database_. Una vez que finalice el comando, compruebe el estado en el que participan todos los nodos de RAC.

:::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/restore-database-srvctl-start-database-command.png" alt-text="Captura de pantalla que muestra el comando srvctl start database para reiniciar los nodos de RAC restantes.":::

## <a name="unmount-log-archive-volume"></a>Desmontaje del volumen de archivos del registro

Después de clonar o restaurar una base de datos, el volumen de archivos del registro debe desmontarse, ya que solo se usa para restaurar la base de datos y luego ya no se usa. 

1. En la pestaña **Resources** (Recursos), para seleccionar la lista de copias de seguridad adecuada en la base de datos, elija la lista local o remota.

2. Una vez que haya seleccionado la lista de copias de seguridad adecuada, ordene la lista en función de si una copia de seguridad se ha montado o no. El algoritmo de ordenación no es perfecto y no siempre ordenará todos los elementos juntos, pero arrojará resultados útiles. 
 
3. Una vez que encuentre el volumen que quiere que se montó previamente, seleccione **Unmount** (Desmontar). Seleccione **Aceptar**.

4. Para ver el estado del trabajo de desmontaje, puede seleccionar **Monitor** (Supervisión) el menú de la izquierda. Aparecerá una marca de verificación verde cuando se desmonte el volumen.

>[!NOTE]
>Al seleccionar el desmontaje, se quita la entrada de /etc/fstab en el host en el que está montada. El desmontaje también libera la copia de seguridad para su eliminación según sea necesario como parte de la directiva de retención establecida en la directiva de protección.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la alta disponibilidad y la recuperación ante desastres para Oracle en BareMetal:

> [!div class="nextstepaction"]
> [Alta disponibilidad y recuperación ante desastres para Oracle en BareMetal](concepts-oracle-high-availability.md)
