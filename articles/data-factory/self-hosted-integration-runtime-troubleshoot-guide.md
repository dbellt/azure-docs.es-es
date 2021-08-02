---
title: Solución de problemas de un entorno de ejecución de integración autohospedado en Azure Data Factory
description: Conozca cómo solucionar problemas del entorno de ejecución de integración autohospedado en Azure Data Factory.
author: lrtoyou1223
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 05/31/2021
ms.author: lle
ms.openlocfilehash: 7abdd532e20a2514fcf96d97973a8fbfdd87d0df
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110796278"
---
# <a name="troubleshoot-self-hosted-integration-runtime"></a>Solución de problemas del entorno de ejecución de integración autohospedado

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se exploran los métodos comunes de solución de problemas del entorno de ejecución de integración (IR) autohospedado en Azure Data Factory.

## <a name="gather-self-hosted-ir-logs-from-azure-data-factory"></a>Recopilación de registros de IR autohospedado en Azure Data Factory

En el caso de las actividades con error que se ejecutan en IR autohospedado o IR compartido, Azure Data Factory admite la visualización y carga de los registros de errores. Para obtener el identificador del informe de error, siga las instrucciones que se indican aquí y, a continuación, escriba el identificador de informe para buscar problemas conocidos relacionados.

1. En Data Factory, seleccione **Ejecuciones de la canalización**.

1. En **Ejecuciones de actividad**, en la columna **Error**, seleccione el botón resaltado para mostrar los registros de actividad, tal como se muestra en la siguiente captura de pantalla:

    ![Captura de pantalla de la sección "Ejecuciones de actividad" en el panel "Todas las ejecuciones de la canalización".](media/self-hosted-integration-runtime-troubleshoot-guide/activity-runs-page.png)

    Los registros de actividad se muestran para la ejecución de actividad con errores.

    ![Captura de pantalla de los registros de actividad para la actividad con errores.](media/self-hosted-integration-runtime-troubleshoot-guide/send-logs.png) 
    
1. Para obtener más ayuda, seleccione **Enviar registros**.
 
   Se abre la ventana **Share the self-hosted integration runtime (IR) logs with Microsoft** (Compartir los registros de entorno de ejecución de integración [IR] autohospedado con Microsoft).

    ![Captura de pantalla de la ventana Compartir los registros de entorno de ejecución de integración (IR) autohospedado con Microsoft.](media/self-hosted-integration-runtime-troubleshoot-guide/choose-logs.png)

1. Seleccione los registros que quiere enviar. 
    * Para *IR autohospedado*, puede cargar los registros relacionados con la actividad con errores o todos los registros del nodo de IR autohospedado. 
    * En el caso de un *IR compartido*, solo puede cargar registros relacionados con la actividad con errores.

1. Cuando se carguen los registros, conserve un registro del identificador del informe para más adelante si necesita ayuda adicional para resolver el problema.

    ![Captura de pantalla del identificador de informe mostrado en la ventana progreso de la carga para los registros de IR.](media/self-hosted-integration-runtime-troubleshoot-guide/upload-logs.png)

> [!NOTE]
> Las solicitudes de visualización y carga de registros se ejecutan en todas las instancias de IR autohospedado en línea. Si falta algún registro, asegúrese de que todas las instancias de IR autohospedado estén en línea. 


## <a name="self-hosted-ir-general-failure-or-error"></a>Error general de IR autohospedado

### <a name="out-of-memory-issue"></a>Problema de memoria agotada

#### <a name="symptoms"></a>Síntomas

Se produce un error OutOfMemoryException (OOM) al intentar ejecutar una actividad de búsqueda con un IR vinculado o un IR autohospedado.

#### <a name="cause"></a>Causa

Una nueva actividad puede generar un error OOM si la máquina de IR experimenta un uso intensivo de memoria. El problema puede deberse a un gran volumen de actividad simultánea, y el error es así intencionadamente.

#### <a name="resolution"></a>Solución

Compruebe el uso de recursos y la ejecución de actividades simultáneas en el nodo del entorno de ejecución de integración. Ajuste el tiempo interno y del desencadenador de las ejecuciones de actividad para evitar demasiadas ejecuciones simultáneas en un único nodo del entorno de ejecución de integración.

### <a name="concurrent-jobs-limit-issue"></a>Problema de límite de trabajos simultáneos

#### <a name="symptoms"></a>Síntomas

Al intentar aumentar el límite de trabajos simultáneos desde la interfaz de Azure Data Factory, el proceso se bloquea con el estado *Actualizando*.

Escenario de ejemplo: El valor máximo de trabajos simultáneos está establecido actualmente en 24, y se quiere aumentar el número total para que los trabajos se puedan ejecutar con mayor rapidez. El valor mínimo que se puede especificar es 3, y el valor máximo es 32. Aumenta el valor de 24 a 32 y, luego, selecciona el botón **Actualizar**. El proceso se bloquea con el estado *Actualizando*, como se muestra en la captura de pantalla siguiente. Actualiza la página y el valor aún se muestra como 24. No se ha actualizado a 32 como era de esperar.

![Captura de pantalla del panel Nodos del entorno de ejecución de integración, que muestra el proceso atascado con el estado "Actualizando".](media/self-hosted-integration-runtime-troubleshoot-guide/updating-status.png)

#### <a name="cause"></a>Causa

El límite en el número de trabajos simultáneos depende del núcleo lógico y de la memoria del equipo. Intente ajustar el valor a la baja hasta un valor como 24 y, a continuación, vea el resultado.

> [!TIP] 
> -    Para obtener más información sobre el recuento de núcleos lógicos y determinar el número de núcleos lógicos de la máquina, consulte [Cuatro maneras de encontrar el número de núcleos de la CPU en Windows 10](https://www.top-password.com/blog/find-number-of-cores-in-your-cpu-on-windows-10/).
> -    Para obtener información sobre cómo calcular math.log, vaya a la [calculadora logarítmica](https://www.rapidtables.com/calc/math/Log_Calculator.html).


### <a name="self-hosted-ir-high-availability-ha-ssl-certificate-issue"></a>Problema de certificado SSL de alta disponibilidad (HA) de IR autohospedado

#### <a name="symptoms"></a>Síntomas

El nodo de trabajo de IR autohospedado notifica el siguiente error:

"Error al extraer los estados compartidos del nodo principal net.tcp://abc.cloud.corp.Microsoft.com:8060/ExternalService.svc/. Id. de actividad: XXXXX No se pudo compilar la cadena CN=abc.cloud.corp.Microsoft.com, OU=test, O=Microsoft de certificado X.509. El certificado que se utilizó tiene una cadena de confianza que no se puede comprobar. Reemplace el certificado o cambie el certificateValidationMode. La función de revocación no pudo comprobar la revocación debido a que el servidor de revocación estaba sin conexión".

#### <a name="cause"></a>Causa

Al tratar casos relacionados con un protocolo de enlace SSL/TLS, es posible que surjan algunos problemas relacionados con la comprobación de la cadena de certificados. 

#### <a name="resolution"></a>Solución

- A continuación, se muestra una manera rápida e intuitiva de solucionar los errores de compilación de la cadena de certificados X.509.
 
    1. Exporte el certificado, que debe comprobarse. Para ello, haga lo siguiente:
    
       a. En Windows, seleccione **Inicio**, empiece a escribir **certificados** y, a continuación, seleccione **Administrar certificados de equipo**.
       
       b. En el Explorador de archivos, en el panel izquierdo, busque el certificado que desea comprobar, haga clic con el botón derecho en él y, a continuación, seleccione **Todas las tareas** > **Exportar**.
    
        ![Captura de pantalla del control "Todas las tareas" > "Exportar" para un certificado en el panel "Administrar certificados de equipo".](media/self-hosted-integration-runtime-troubleshoot-guide/export-tasks.png)

    2. Copie el certificado exportado en el equipo cliente. 
    3. En el lado cliente, en una ventana del símbolo del sistema, ejecute el comando siguiente. No olvide reemplazar *\<certificate path>* y *\<output txt file path>* por las rutas de acceso reales.
    
        ```
        Certutil -verify -urlfetch    <certificate path>   >     <output txt file path> 
        ```

        Por ejemplo:

        ```
        Certutil -verify -urlfetch c:\users\test\desktop\servercert02.cer > c:\users\test\desktop\Certinfo.txt
        ```
    4. Compruebe los errores en el archivo TXT de salida. Puede encontrar el resumen de errores al final del archivo TXT.

        Por ejemplo: 

        ![Captura de pantalla de un resumen de errores al final del archivo TXT.](media/self-hosted-integration-runtime-troubleshoot-guide/error-summary.png)

        Si no ve ningún error al final del archivo de registro, como se muestra en la siguiente captura de pantalla, puede considerar que la cadena de certificados se ha compilado correctamente en la máquina cliente.
        
        ![Captura de pantalla de un archivo de registro que no muestra ningún error.](media/self-hosted-integration-runtime-troubleshoot-guide/log-file.png)      

- Si se configura una extensión de nombre de archivo de AIA (acceso a la información de entidad emisora), CDP (punto de distribución cRL) una OCSP (protocolo de estado de certificados en línea) en el archivo de certificado, puede comprobarlo de una manera más intuitiva:
 
    1. Para obtener esta información, compruebe los detalles del certificado, tal como se muestra en la captura de pantalla siguiente:
    
        ![Captura de pantalla de los detalles del certificado.](media/self-hosted-integration-runtime-troubleshoot-guide/certificate-detail.png)
    
    1. Ejecute el comando siguiente. Asegúrese de reemplazar *\<certificate path>* por la ruta de acceso real del certificado.
    
        ```
          Certutil   -URL    <certificate path> 
        ```
    
        Se abre la herramienta de recuperación de dirección URL. 
        
    1. Para comprobar los certificados con las extensiones de nombre de archivo AIA, CDP y OCSP, seleccione **Recuperar**.

        ![Captura de pantalla de la herramienta de recuperación de URL y el botón Recuperar.](media/self-hosted-integration-runtime-troubleshoot-guide/retrieval-button.png)
 
        Habrá creado la cadena de certificados correctamente si el estado del certificado de AIA es *Comprobado* y el estado del certificado de CDP u OCSP es *Comprobado*.

        Si hay un error al intentar recuperar AIA o CDP, trabaje con el equipo de redes para preparar la máquina cliente para conectarse a la dirección URL de destino. Será suficiente si se puede comprobar la ruta de acceso HTTP o la ruta de acceso del Protocolo ligero de acceso a directorios (LDAP).

### <a name="self-hosted-ir-could-not-load-file-or-assembly"></a>IR autohospedado no pudo cargar el archivo o ensamblado

#### <a name="symptoms"></a>Síntomas

Se muestra el siguiente mensaje de error:

"No se puede cargar el archivo o ensamblado 'XXXXXXXXXXXXXXXX, Version=4.0.2.0, Culture=neutral, PublicKeyToken=XXXXXXXXX' o una de sus dependencias. El sistema no encuentra el archivo especificado. Id. de actividad: 92693b45-b4bf-4fc8-89da-2d3dc56f27c3"
 
He aquí un mensaje de error más específico: 

"No se puede cargar el archivo o ensamblado 'System.ValueTuple, Version=4.0.2.0, Culture=neutral, PublicKeyToken=XXXXXXXXX' o una de sus dependencias. El sistema no encuentra el archivo especificado. Id. de actividad: 92693b45-b4bf-4fc8-89da-2d3dc56f27c3"

#### <a name="cause"></a>Causa

En Process Monitor, puede ver el siguiente resultado:

[![Captura de pantalla de la lista de rutas de acceso en Process Monitor.](media/self-hosted-integration-runtime-troubleshoot-guide/process-monitor.png)](media/self-hosted-integration-runtime-troubleshoot-guide/process-monitor.png#lightbox)

> [!TIP] 
> En Process Monitor, puede establecer filtros como se muestra en la captura de pantalla siguiente.
>
> El mensaje de error anterior indica que el archivo DLL System.ValueTuple no se encuentra en la carpeta de *caché global de ensamblados* (GAC) relacionada, en la carpeta *C:\Archivos de programa\Microsoft Integration Runtime\4.0\Gateway* ni en la carpeta *C:\Archivos de programa\Microsoft Integration Runtime\4.0\Shared*.
>
> Básicamente, el proceso carga el archivo DLL primero desde la carpeta *GAC*, luego de la carpeta *Shared* y, finalmente, de la carpeta *Gateway*. Por lo tanto, puede cargar el archivo DLL desde cualquier ruta de acceso que resulte útil.

<br>

![Captura de pantalla de la página "Filtro de monitor de procesos", donde se muestran los filtros del archivo DLL.](media/self-hosted-integration-runtime-troubleshoot-guide/set-filters.png)

#### <a name="resolution"></a>Solución

Puede ver que el archivo *System.ValueTuple.dll* se encuentra en la carpeta *C:\Archivos de programa\Microsoft Integration Runtime\4.0\Gateway\DataScan*. Para resolver el problema, copie el archivo *System.ValueTuple.dll* en la carpeta *C:\Archivos de programa\Microsoft Integration Runtime\4.0\Gateway*.

Puede usar el mismo método para resolver otros problemas de archivos o ensamblados que faltan.

#### <a name="more-information-about-this-issue"></a>Más información sobre este problema

La razón por la que se ve el archivo *System.ValueTuple.dll* en *%windir%\Microsoft.NET\assembly* y *%windir%\assembly* es que es un comportamiento de .NET. 

En el siguiente error, puede ver claramente que falta el ensamblado *System.ValueTuple*. Este problema surge cuando la aplicación intenta comprobar el ensamblado *System.ValueTuple.dll*.
 
"\<LogProperties>\<ErrorInfo>[{"Código":0,"Mensaje":"Se produjo una excepción en el inicializador de tipo de 'Npgsql.PoolManager'.","EventType":0,"Category":5,"Data":{},"MsgId":null,"ExceptionType":"System.TypeInitializationException","Source":"Npgsql","StackTrace":"","InnerEventInfos":[{"Código":0,"Mensaje":"No se puede cargar el archivo o ensamblado 'System.ValueTuple, Version=4.0.2.0, Culture=neutral, PublicKeyToken=XXXXXXXXX' ni una de sus dependencias. No se puede encontrar el archivo especificado.","EventType":0,"Category":5,"Data":{},"MsgId":null,"ExceptionType":"System.IO.FileNotFoundException","Source":"Npgsql","StackTrace":"","InnerEventInfos":[]}]}]\</ErrorInfo>\</LogProperties>"
 
Para obtener más información sobre GAC, consulte [Caché global de ensamblados](/dotnet/framework/app-domains/gac).


### <a name="self-hosted-integration-runtime-authentication-key-is-missing"></a>Falta la clave de autenticación del entorno de ejecución de integración autohospedado

#### <a name="symptoms"></a>Síntomas

El entorno de ejecución de integración autohospedado se queda sin conexión repentinamente sin una clave de autenticación, y el registro de eventos muestra el mensaje de error siguiente: 

"Authentication Key is not assigned yet" (La clave de autenticación no se ha asignado todavía)

![Captura de pantalla del panel de eventos del entorno de ejecución de integración que muestra que la clave de autenticación aún no se ha asignado.](media/self-hosted-integration-runtime-troubleshoot-guide/key-missing.png)

#### <a name="cause"></a>Causa

- Se eliminó el nodo de IR autohospedado o el IR autohospedado lógico en Azure Portal.
- Se realizó una desinstalación limpia.

#### <a name="resolution"></a>Solución

Si no corresponde ninguna de las causas anteriores, puede ir a la carpeta *%programdata%\Microsoft\Data Transfer\DataManagementGateway* para comprobar si se ha eliminado el archivo *Configurations*. Si se eliminó, siga las instrucciones del artículo de Netwrix sobre la [detección de quién eliminó un archivo de los servidores de archivos de Windows](https://www.netwrix.com/how_to_detect_who_deleted_file.html).

![Captura de pantalla del panel de detalles del registro de eventos para comprobar el archivo Configurations.](media/self-hosted-integration-runtime-troubleshoot-guide/configurations-file.png)


### <a name="cant-use-self-hosted-ir-to-bridge-two-on-premises-datastores"></a>No se puede usar IR autohospedado para enlazar dos almacenes de datos locales

#### <a name="symptoms"></a>Síntomas

Después de crear los IR autohospedados tanto para el almacén de datos de origen como para el de destino, quiere conectar los dos IR para finalizar una actividad de copia. Si los almacenes de datos están configurados en redes virtuales diferentes, o los almacenes de datos no pueden entender el mecanismo de la puerta de enlace, recibirá uno de los siguientes errores: 

* "The driver of source cannot be found in destination IR" (No se puede encontrar el controlador de origen en el IR de destino)
* "The source cannot be accessed by the destination IR" (El IR de destino no puede acceder al origen)
 
#### <a name="cause"></a>Causa

IR autohospedado está diseñado como nodo central de una actividad de copia, no como agente cliente que debe instalarse para cada almacén de datos.
 
En este caso, debe crear el servicio vinculado para cada almacén de datos con el mismo IR, y este debe poder acceder a ambos almacenes de datos a través de la red. No importa si el IR está instalado en el almacén de datos de origen, en el almacén de datos de destino o en una tercera máquina. Si se crean dos servicios vinculados con IR diferentes, pero se usan en la misma actividad de copia, se usa el IR de destino, y es necesario instalar los controladores para ambos almacenes de datos en la máquina del IR de destino.

#### <a name="resolution"></a>Solución

Instale los controladores para los almacenes de datos de origen y de destino en el IR de destino, y asegúrese de que este puede acceder al almacén de datos de origen.
 
Si el tráfico no puede atravesar la red entre dos almacenes de datos (por ejemplo, están configurados en dos redes virtuales), es posible que no finalice la copia en una actividad, incluso con el IR instalado. Si no puede finalizar la copia en una sola actividad, puede crear dos actividades de copia con dos IR, cada uno en una red virtual: 
* Copie un IR del almacén de datos 1 a Azure Blob Storage
* Copie otro entorno de ejecución de integración de Azure Blob Storage en el almacén de datos 2. 

Esta solución podría simular el requisito de usar el IR para crear un puente que conecte dos almacenes de datos desconectados.


### <a name="credential-sync-issue-causes-credential-loss-from-ha"></a>El problema de sincronización de credenciales provoca la pérdida de credenciales de alta disponibilidad

#### <a name="symptoms"></a>Síntomas

Si la credencial del origen de datos "XXXXXXXXXX" se elimina del nodo actual del entorno de ejecución de integración con la carga útil, recibirá el mensaje de error siguiente:

"Al eliminar el servicio de vínculos en Azure Portal, o bien la tarea tiene una carga incorrecta; vuelva a crear un nuevo servicio de vínculos con sus credenciales".

#### <a name="cause"></a>Causa

El IR autohospedado está integrado en el modo de alta disponibilidad con dos nodos, pero los nodos no están en un estado de sincronización de credenciales. Es decir, las credenciales almacenadas en el nodo del distribuidor no se sincronizan con otros nodos de trabajo. Si se produce una conmutación por error del nodo distribuidor al nodo de trabajo, y las credenciales solo existen en el nodo distribuidor anterior, se producirá un error en la tarea al intentar acceder a las credenciales, y recibirá el error anterior.

#### <a name="resolution"></a>Solución

La única manera de evitar este problema es asegurarse de que los dos nodos estén en un estado de sincronización de credenciales. Si no están sincronizados, tendrá que volver a escribir las credenciales para el nuevo distribuidor.


### <a name="cant-choose-the-certificate-because-the-private-key-is-missing"></a>No se puede elegir el certificado porque falta la clave privada

#### <a name="symptoms"></a>Síntomas

* Ha importado un archivo PFX al almacén de certificados.
* Al seleccionar el certificado a través de la interfaz de usuario de IR Configuration Manager, recibió el siguiente mensaje de error:

   "No se pudo cambiar el modo de cifrado de la comunicación de intranet. Es probable que el certificado '\<*certificate name*>' no tenga una clave pública que se pueda intercambiar o puede que el proceso no tenga derechos de acceso para la clave privada. Consulte la excepción interna para obtener más información".

    ![Captura de pantalla del panel configuración de Integration Runtime Configuration Manager, que muestra un mensaje de error "Falta una clave privada".](media/self-hosted-integration-runtime-troubleshoot-guide/private-key-missing.png)

#### <a name="cause"></a>Causa

- La cuenta de usuario tiene un nivel de privilegios bajo y no puede acceder a la clave privada.
- El certificado se generó como firma, pero no como intercambio de claves.

#### <a name="resolution"></a>Solución

* Para usar la interfaz de usuario, use una cuenta con los privilegios adecuados para acceder a la clave privada.  
* Para importar el certificado, ejecute el siguiente comando:
    
    ```
    certutil -importpfx FILENAME.pfx AT_KEYEXCHANGE
    ```

### <a name="self-hosted-integration-runtime-nodes-out-of-the-sync-issue"></a>Problema de nodos del entorno de ejecución de integración autohospedado sin sincronización

#### <a name="symptoms"></a>Síntomas

Los nodos del entorno de ejecución de integración autohospedado intentan sincronizar las credenciales entre nodos, pero se quedan bloqueados en el proceso y encuentran el mensaje de error siguiente al cabo de un tiempo:

"El nodo Integration Runtime (autohospedado) está intentando sincronizar las credenciales entre nodos. Esto podría tardar varios minutos".

>[!Note]
>Si este error aparece durante más de 10 minutos, compruebe la conectividad con el nodo de distribuidor.

#### <a name="cause"></a>Causa

El motivo es que los nodos de trabajo no tienen acceso a las claves privadas. Esto se puede confirmar a partir de los registros del entorno de ejecución de integración autohospedado a continuación:

`[14]0460.3404::05/07/21-00:23:32.2107988 [System] A fatal error occurred when attempting to access the TLS server credential private key. The error code returned from the cryptographic module is 0x8009030D. The internal error state is 10001.`

No tiene ningún problema con el proceso de sincronización al usar la autenticación de la entidad de servicio en el servicio vinculado de ADF. Sin embargo, al cambiar el tipo de autenticación a clave de cuenta, se inició el problema de sincronización. Esto se debe a que el servicio del entorno de ejecución de integración autohospedado se ejecuta en una cuenta de servicio (NT SERVICE\DIAHostService) y debe agregarse a los permisos de clave privada.
 

#### <a name="resolution"></a>Solución

Para solucionar este problema, debe agregar la cuenta de servicio del entorno de ejecución de integración autohospedado (NT SERVICE\DIAHostService) a los permisos de clave privada. Puede aplicar los pasos siguientes:

1. Abra el comando de ejecución de Microsoft Management Console (MMC).

    :::image type="content" source="./media/self-hosted-integration-runtime-troubleshoot-guide/management-console-run-command.png" alt-text="Captura de pantalla que muestra el comando de ejecución de MMC":::

1. En el panel de MMC, puede aplicar los pasos siguientes:

    :::image type="content" source="./media/self-hosted-integration-runtime-troubleshoot-guide/add-service-account-to-private-key-1.png" alt-text="Captura de pantalla que muestra el segundo paso para agregar una cuenta de servicio del entorno de ejecución de integración autohospedado a los permisos de clave privada" lightbox="./media/self-hosted-integration-runtime-troubleshoot-guide/add-service-account-to-private-key-1-expanded.png":::

    1. Seleccione **Archivo**.
    1. Elija **Agregar o quitar complemento** en el menú desplegable.
    1. Seleccione **Certificados** en el panel "Complementos disponibles".
    1. Seleccione **Agregar**.
    1. En el panel emergente "Complemento Certificados", elija **Cuenta de equipo**.
    1. Seleccione **Siguiente**.
    1. En el panel "Seleccionar equipo", elija **Equipo local: el equipo en el que se ejecuta esta consola**.
    1. Seleccione **Finalizar**.
    1. Seleccione **Aceptar** en el panel "Agregar o quitar complementos".

1. En el panel de MMC, siga estos pasos:

    :::image type="content" source="./media/self-hosted-integration-runtime-troubleshoot-guide/add-service-account-to-private-key-2.png" alt-text="Captura de pantalla que muestra el tercer paso para agregar una cuenta de servicio del entorno de ejecución de integración autohospedado a los permisos de clave privada" lightbox="./media/self-hosted-integration-runtime-troubleshoot-guide/add-service-account-to-private-key-2-expanded.png":::

    1. En la lista de carpetas izquierda, seleccione **Raíz de consola -> Certificados (Equipo local) -> Personal -> Certificados**.
    1. Haga clic con el botón derecho en **Microsoft Intune MDM beta**.
    1. Seleccione **Todas las tareas** en la lista desplegable.
    1. Seleccione **Administrar claves privadas**.
    1. Seleccione **Agregar** en "Nombre de grupo o usuario".
    1. Seleccione **NT SERVICE\DIAHostService** para concederle acceso de control total a este certificado, aplicarlo y asegurarlo. 
    1. Seleccione **Comprobar nombres** y, después, **Aceptar**.
    1. En el panel "Permisos", seleccione **Aplicar** y, a continuación, **Aceptar**.

## <a name="self-hosted-ir-setup"></a>Configuración de IR autohospedado

### <a name="integration-runtime-registration-error"></a>Error de registro del entorno de ejecución de integración 

#### <a name="symptoms"></a>Síntomas

En ocasiones, es posible que quiera ejecutar un IR autohospedado en otra cuenta por alguno de los siguientes motivos:
- La directiva de la compañía no permite la cuenta de servicio.
- Se requiere autenticación.

Después de cambiar la cuenta de servicio en el panel de servicio, es posible que el entorno de ejecución de integración deje de funcionar y reciba el siguiente mensaje de error:

"El nodo de Integration Runtime (autohospedado) ha encontrado un error durante el registro. No se puede conectar al servicio host de Integration Runtime (autohospedado)".

![Captura de pantalla de la ventana Integration Runtime Configuration Manager, que muestra un error de registro de IR.](media/self-hosted-integration-runtime-troubleshoot-guide/ir-registration-error.png)

#### <a name="cause"></a>Causa

Muchos recursos solo se conceden a la cuenta de servicio. Al cambiar la cuenta de servicio a otra cuenta, los permisos de todos los recursos dependientes siguen siendo los mismos.

#### <a name="resolution"></a>Solución

Vaya al registro de eventos de Integration Runtime para comprobar el error.

![Captura de pantalla del registro de eventos de IR, que indica que hubo un error de tiempo de ejecución.](media/self-hosted-integration-runtime-troubleshoot-guide/ir-event-log.png)

* Si el error en el registro de eventos es "UnauthorizedAccessException", haga lo siguiente:

    1. Compruebe la cuenta del servicio de inicio de sesión *DIAHostService* en el panel del servicio de Windows.

        ![Captura de pantalla del panel de propiedades de la cuenta de servicio de inicio de sesión.](media/self-hosted-integration-runtime-troubleshoot-guide/logon-service-account.png)

    1. Compruebe si la cuenta de servicio de inicio de sesión tiene permisos de lectura y escritura en la carpeta *%programdata%\Microsoft\DataTransfer\DataManagementGateway*.

        - De manera predeterminada, si no se ha cambiado la cuenta de inicio de sesión del servicio, debe tener los permisos de lectura y escritura.

            ![Captura de pantalla del panel de permisos del servicio.](media/self-hosted-integration-runtime-troubleshoot-guide/service-permission.png)

        - Si ha cambiado la cuenta de inicio de sesión del servicio, siga los pasos a continuación para mitigar el problema:
 
            a. Realice una desinstalación limpia del IR autohospedado actual.   
            b. Instale los bits de IR autohospedado.  
            c. Cambie la cuenta de servicio mediante el procedimiento siguiente:  

             i. Vaya a la carpeta de instalación de IR autohospedado y cambie a la carpeta *Microsoft Integration Runtime\4.0\Shared*.  
             ii. Abra la ventana Símbolo del sistema con privilegios elevados. Reemplace *\<user>* y *\<password>* por su propio nombre de usuario y contraseña y, a continuación, ejecute el comando siguiente:   
                `dmgcmd.exe -SwitchServiceAccount "<user>" "<password>"`  
             iii. Si quiere cambiar a una cuenta LocalSystem, asegúrese de usar un formato correcto para esta cuenta: `dmgcmd.exe -SwitchServiceAccount "NT Authority\System" ""`  
                *No* use este formato: `dmgcmd.exe -SwitchServiceAccount "LocalSystem" ""`     
             iv. De manera opcional, dado que el sistema local tiene más privilegios que el administrador, también puede cambiarlo directamente en "Servicios".  
             v. Puede usar un usuario local o de dominio para la cuenta de inicio de sesión del servicio de IR.            

            d. Registre el entorno de ejecución de integración.

* Si el error es "No se pudo iniciar el 'servicio de Integration Runtime" (DIAHostService). Compruebe que dispone de suficientes privilegios para iniciar servicios del sistema", haga lo siguiente:

    1. Compruebe la cuenta del servicio de inicio de sesión *DIAHostService* en el panel del servicio de Windows.
    
        ![Captura de pantalla del panel "Iniciar sesión" de la cuenta de servicio.](media/self-hosted-integration-runtime-troubleshoot-guide/logon-service-account.png)

    1. Compruebe si la cuenta de servicio de inicio de sesión tiene permiso **Iniciar sesión como servicio** para iniciar el servicio de Windows:

        ![Captura de pantalla del panel de propiedades de " Iniciar sesión como servicio".](media/self-hosted-integration-runtime-troubleshoot-guide/logon-as-service.png)

#### <a name="more-information"></a>Más información

Si ninguno de los dos patrones anteriores de resolución se aplica en su caso, intente recopilar los siguientes registros de eventos de Windows: 
- Registros de aplicaciones y servicios > Integration Runtime
- Registros de Windows > Aplicación

### <a name="cant-find-the-register-button-to-register-a-self-hosted-ir"></a>No se encuentra el botón Registrar para registrar un IR autohospedado    

#### <a name="symptoms"></a>Síntomas

Al registrar un IR autohospedado, el botón **Registrar** no se muestra en el panel de Configuration Manager.

![Captura de pantalla del panel de Configuration Manager, que muestra un mensaje que indica que el nodo de Integration Runtime no está registrado.](media/self-hosted-integration-runtime-troubleshoot-guide/no-register-button.png)

#### <a name="cause"></a>Causa

A partir del lanzamiento de Integration Runtime 3.0, se ha quitado el botón **Registrar** en un nodo de Integration Runtime existente para habilitar un entorno más limpio y seguro. Si un nodo se ha registrado en algún entorno de ejecución de integración, ya sea en línea o no, para volver a registrarlo en otro entorno de ejecución de integración, debe desinstalar el nodo anterior y, a continuación, instalar y registrar el nodo.

#### <a name="resolution"></a>Solución

1. En el Panel de control, desinstale el entorno de ejecución de integración existente.

    > [!IMPORTANT] 
    > En el proceso siguiente, seleccione **Sí**. No mantenga los datos durante el proceso de desinstalación.

    ![Captura de pantalla del botón "Sí" para eliminar todos los datos de usuario del entorno de ejecución de integración.](media/self-hosted-integration-runtime-troubleshoot-guide/delete-data.png)

1. Si no tiene el archivo MSI del instalador del entorno de ejecución de integración, vaya al [centro de descarga](https://www.microsoft.com/en-sg/download/details.aspx?id=39717) para descargar el entorno de ejecución de integración más reciente.
1. Instale el archivo MSI y registre el entorno de ejecución de integración.


### <a name="unable-to-register-the-self-hosted-ir-because-of-localhost"></a>No se puede registrar el IR autohospedado debido a localhost    

#### <a name="symptoms"></a>Síntomas

No puede registrar el IR autohospedado en una máquina nueva cuando usa get_LoopbackIpOrName.

**Depuración:** Se ha producido un error de tiempo de ejecución.
El inicializador de tipo para "Microsoft.DataTransfer.DIAgentHost.DataSourceCache" produjo una excepción.
Se produjo un error no recuperable durante una búsqueda de base de datos.
 
**Detalle de la excepción:** System.TypeInitializationException: El inicializador de tipo para "Microsoft.DataTransfer.DIAgentHost.DataSourceCache" produjo una excepción. ---> System.Net.Sockets.SocketException: Se produjo un error no recuperable durante una búsqueda de base de datos en System.Net.Dns.GetAddrInfo(nombre de la cadena).

#### <a name="cause"></a>Causa

El problema suele producirse cuando se está resolviendo el localhost.

#### <a name="resolution"></a>Solución

Use la dirección IP de localhost 127.0.0.1 para hospedar el archivo y resolver el problema.

### <a name="self-hosted-setup-failed"></a>Error al instalar IR autohospedado    

#### <a name="symptoms"></a>Síntomas

No puede desinstalar un IR existente, instalar un IR nuevo ni actualizar un IR existente a un IR nuevo.

#### <a name="cause"></a>Causa

La instalación del entorno de ejecución de integración depende del servicio de Windows Installer. Puede experimentar problemas de instalación por los siguientes motivos:
- Espacio en disco disponible insuficiente.
- Falta de permisos.
- El servicio de Windows NT está bloqueado.
- El uso de la CPU es demasiado alto.
- El archivo MSI se hospeda en una ubicación de red lenta.
- Algunos registros o archivos del sistema se modificaron de forma accidental.

### <a name="the-ir-service-account-failed-to-fetch-certificate-access"></a>La cuenta de servicio del IR no pudo capturar el acceso al certificado

#### <a name="symptoms"></a>Síntomas

Al instalar el IR autohospedado a través de Microsoft Integration Runtime Configuration Manager, se genera un certificado con una entidad de certificación (CA) de confianza. No se pudo aplicar el certificado para cifrar la comunicación entre dos nodos, y se muestra el siguiente mensaje de error: 

"No se pudo cambiar el modo de cifrado de la comunicación de intranet: No se pudo conceder acceso a la cuenta de servicio de Integration Runtime al certificado '\<*certificate name*>'. Código de error: 103".

![Captura de pantalla que muestra el mensaje de error "… No se pudo conceder acceso a la cuenta de servicio de Integration Runtime al certificado".](media/self-hosted-integration-runtime-troubleshoot-guide/integration-runtime-service-account-certificate-error.png)

#### <a name="cause"></a>Causa

El certificado usa un almacenamiento de proveedor de almacenamiento de claves (KSP), que todavía no se admite. Hasta la fecha, IR autohospedado solo admite el almacenamiento de proveedor de servicios criptográficos (CSP).

#### <a name="resolution"></a>Solución

En este caso, se recomienda usar certificados de CSP.

**Solución 1** 

Para importar el certificado, ejecute el siguiente comando:

`Certutil.exe -CSP "CSP or KSP" -ImportPFX FILENAME.pfx`

![Captura de pantalla del comando certutil para importar el certificado.](media/self-hosted-integration-runtime-troubleshoot-guide/use-certutil.png)

**Solución 2** 

Para convertir el certificado, ejecute los siguientes comandos:

`openssl pkcs12 -in .\xxxx.pfx -out .\xxxx_new.pem -password pass: <EnterPassword>`
`openssl pkcs12 -export -in .\xxxx_new.pem -out xxxx_new.pfx`

Antes y después de la conversión:

![Captura de pantalla del resultado antes de la conversión del certificado.](media/self-hosted-integration-runtime-troubleshoot-guide/before-certificate-change.png)

![Captura de pantalla del resultado después de la conversión del certificado.](media/self-hosted-integration-runtime-troubleshoot-guide/after-certificate-change.png)

### <a name="self-hosted-integration-runtime-version-5x"></a>Entorno de ejecución de integración autohospedado, versión 5.x
Para la actualización a la versión 5.x del entorno de ejecución de integración autohospedado de Azure Data Factory, se requiere el **tiempo de ejecución de .NET Framework 4.7.2** o posterior. En la página de descarga, habrá vínculos de descarga para la versión 4.x más reciente y las dos versiones 5.x más recientes. 

Para los clientes de Azure Data Factory v2:
- Si la actualización automática está activada y ya ha actualizado su entorno de tiempo de ejecución de .NET Framework a la versión 4.7.2 o posterior, el entorno de ejecución de integración autohospedado se actualizará automáticamente a la versión 5.x más reciente.
- Si la actualización automática está activada y ya ha actualizado su entorno de tiempo de ejecución de .NET Framework a la versión 4.7.2 o posterior, el entorno de ejecución de integración autohospedado se actualizará automáticamente a la versión 5.x más reciente. El entorno de ejecución de integración autohospedado permanecerá en la versión 4.x actual. Verá una advertencia para la actualización del entorno de tiempo de ejecución de .NET Framework en el portal y en el cliente del entorno de ejecución de integración autohospedado.
- Si la actualización automática está desactivada y ya ha actualizado el entorno de tiempo de ejecución de .NET Framework a la versión 4.7.2 o posterior, puede descargar manualmente la versión 5.x más reciente e instalarla en la máquina.
- Si la actualización automática está desactivada y no ha actualizado el entorno de tiempo de ejecución de .NET Framework a la versión 4.7.2 o posterior, al intentar instalar manualmente el entorno de ejecución de integración autohospedado 5. x y registrar la clave, se le pedirá que actualice primero la versión del entorno de tiempo de ejecución de .NET Framework.


Para los clientes de Azure Data Factory v1:
- El entorno de ejecución de integración autohospedado, versión 5.x, no admite Azure Data Factory v1.
- El entorno de ejecución de integración autohospedado se actualizará automáticamente a la versión 4.x más reciente. Y la versión 4.x más reciente no expirará. 
- Si intenta instalar manualmente el entorno de ejecución de integración autohospedado 5.x y registrar la clave, se le indicará que esta versión no admite Azure Data Factory v1.


## <a name="self-hosted-ir-connectivity-issues"></a>Problemas de conectividad de IR autohospedado

### <a name="self-hosted-integration-runtime-cant-connect-to-the-cloud-service"></a>El entorno de ejecución de integración autohospedado no puede conectarse al servicio en la nube

#### <a name="symptoms"></a>Síntomas

Al intentar registrar el entorno de ejecución de integración autohospedado, Configuration Manager muestra el siguiente mensaje de error:

"El nodo de Integration Runtime (autohospedado) ha encontrado un error durante el registro".

![Captura de pantalla del mensaje "El nodo de Integration Runtime (autohospedado) ha encontrado un error durante el registro".](media/self-hosted-integration-runtime-troubleshoot-guide/unable-to-connect-to-cloud-service.png)

#### <a name="cause"></a>Causa 

El IR autohospedado no puede conectarse al back-end del servicio Azure Data Factory. Este problema se debe normalmente a la configuración de red en el firewall.

#### <a name="resolution"></a>Solución

1. Compruebe si el servicio de entorno de ejecución de integración está en ejecución. En caso afirmativo, vaya al paso 2.
    
   ![Captura de pantalla que muestra que el servicio de IR autohospedado está en ejecución.](media/self-hosted-integration-runtime-troubleshoot-guide/integration-runtime-service-running-status.png)
    
1. Si no hay ningún proxy configurado en el IR autohospedado, que es la configuración predeterminada, ejecute el siguiente comando de PowerShell en la máquina donde está instalado el entorno de ejecución de integración autohospedado:

    ```powershell
    (New-Object System.Net.WebClient).DownloadString("https://wu2.frontend.clouddatahub.net/")
    ```
        
   > [!NOTE]     
   > La dirección URL del servicio puede variar en función de la ubicación de la instancia de Data Factory. Para encontrar la dirección URL del servicio, seleccione **ADF UI** > **Connections** > **Integration runtimes** > **Edit Self-hosted IR** > **Nodes** > **View Service URLs** (Interfaz de usuario de ADF > Conexiones > Entornos de ejecución de integración > Editar IR autohospedado > Nodos > Ver direcciones URL del servicio).
            
    La respuesta esperada es la siguiente:
            
    ![Captura de pantalla de la respuesta del comando de PowerShell.](media/self-hosted-integration-runtime-troubleshoot-guide/powershell-command-response.png)
            
1. Si no recibe la respuesta que esperaba, use uno de los métodos siguientes, según corresponda:
            
    * Si recibe el mensaje "No se pudo resolver el nombre remoto", hay un problema con el sistema de nombres de dominio (DNS). Póngase en contacto con su equipo de redes para solucionar el problema.
    * Si recibe el error "El certificado SSL/TLS no es de confianza", [compruebe el certificado](https://wu2.frontend.clouddatahub.net/) para ver si es de confianza en la máquina y luego instale el certificado público mediante el Administrador de certificados. Esta acción debería mitigar el problema.
    * Vaya a **Windows** > **Visor de eventos (registros)**  > **Registros de aplicaciones y servicios** > **Integration Runtime** y compruebe si hay errores causados por DNS, una regla del firewall o la configuración de red de la empresa. Si encuentra este tipo de error, fuerce el cierre de la conexión. Dado que cada compañía tiene su propia configuración de red personalizada, póngase en contacto con su equipo de red para solucionar estos problemas.

1. Si el proxy se ha configurado en el entorno de ejecución de integración autohospedado, compruebe si el servidor proxy puede acceder al punto de conexión de servicio. Para ver un comando de ejemplo, consulte [PowerShell, solicitudes web y servidores proxy](https://stackoverflow.com/questions/571429/powershell-web-requests-and-proxies).    
                
    ```powershell
    $user = $env:username
    $webproxy = (get-itemproperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet
    Settings').ProxyServer
    $pwd = Read-Host "Password?" -assecurestring
    $proxy = new-object System.Net.WebProxy
    $proxy.Address = $webproxy
    $account = new-object System.Net.NetworkCredential($user,[Runtime.InteropServices.Marshal]::PtrToStringAuto([Runtime.InteropServices.Marshal]::SecureStringToBSTR($pwd)), "")
    $proxy.credentials = $account
    $url = "https://wu2.frontend.clouddatahub.net/"
    $wc = new-object system.net.WebClient
    $wc.proxy = $proxy
    $webpage = $wc.DownloadData($url)
    $string = [System.Text.Encoding]::ASCII.GetString($webpage)
    $string
    ```

La respuesta esperada es la siguiente:
            
![Captura de pantalla de la respuesta esperada del comando de PowerShell.](media/self-hosted-integration-runtime-troubleshoot-guide/powershell-command-response.png)

> [!NOTE] 
> Consideraciones del proxy:
> * Compruebe si el servidor proxy debe colocarse en la lista de destinatarios seguros. Si es así, asegúrese de que [estos dominios](./data-movement-security-considerations.md#firewall-requirements-for-on-premisesprivate-network) estén en la lista de destinatarios seguros.
> * Compruebe si el certificado TLS/SSL de "wu2.frontend.clouddatahub.net/" es de confianza en el servidor proxy.
> * Si usa la autenticación de Active Directory en el proxy, cambie la cuenta de servicio por la cuenta de usuario que pueda acceder al proxy como "Integration Runtime Service".

### <a name="error-message-self-hosted-integration-runtime-nodelogical-self-hosted-ir-is-in-inactive-running-limited-state"></a>Mensaje de error: El nodo del entorno de ejecución de integración autohospedado/IR autohospedado lógico está en estado inactivo/"en ejecución (limitado)"

#### <a name="cause"></a>Causa 

El nodo de tiempo de ejecución integrado autohospedado podría tener un estado **Inactivo**, como se muestra en la siguiente captura de pantalla:

![Captura de pantalla del nodo del entorno de ejecución integrado autohospedado con estado inactivo.](media/self-hosted-integration-runtime-troubleshoot-guide/inactive-self-hosted-ir-node.png)

Este comportamiento se produce cuando los nodos no pueden comunicarse entre sí.

#### <a name="resolution"></a>Solución

1. Inicie sesión en la máquina virtual (VM) hospedada en el nodo. En **Registros de aplicaciones y servicios** > **Integration Runtime**, abra el Visor de eventos y filtre los registros de errores.

1. Compruebe si el registro de errores contiene el error siguiente: 
    
    ```
    System.ServiceModel.EndpointNotFoundException: Could not connect to net.tcp://xxxxxxx.bwld.com:8060/ExternalService.svc/WorkerManager. The connection attempt lasted for a time span of 00:00:00.9940994. TCP error code 10061: No connection could be made because the target machine actively refused it 10.2.4.10:8060. 
    System.Net.Sockets.SocketException: No connection could be made because the target machine actively refused it. 
    10.2.4.10:8060
    at System.Net.Sockets.Socket.DoConnect(EndPoint endPointSnapshot, SocketAddress socketAddress)
    at System.Net.Sockets.Socket.Connect(EndPoint remoteEP)
    at System.ServiceModel.Channels.SocketConnectionInitiator.Connect(Uri uri, TimeSpan timeout)
    ```
       
1. Si ve este error, ejecute el siguiente comando en una ventana del símbolo del sistema: 

   ```
   telnet 10.2.4.10 8060
   ```
   
1. Si recibe el error de la línea de comandos "Could not open connection to the host" (No se pudo abrir la conexión con el host) que se muestra en la siguiente captura de pantalla, póngase en contacto con el Departamento de TI para que le ayude a solucionar este problema. Una vez que pueda comunicarse con Telnet, póngase en contacto con Soporte técnico de Microsoft si todavía tiene problemas con el estado del nodo del entorno de ejecución de integración.
        
   ![Captura de pantalla del error de la línea de comandos "No se pudo abrir la conexión con el host".](media/self-hosted-integration-runtime-troubleshoot-guide/command-line-error.png)
        
1. Compruebe si el registro de errores contiene la entrada siguiente:

    ```
    Error log: Cannot connect to worker manager: net.tcp://xxxxxx:8060/ExternalService.svc/ No DNS entries exist for host azranlcir01r1. No such host is known Exception detail: System.ServiceModel.EndpointNotFoundException: No DNS entries exist for host xxxxx. ---> System.Net.Sockets.SocketException: No such host is known at System.Net.Dns.GetAddrInfo(String name) at System.Net.Dns.InternalGetHostByName(String hostName, Boolean includeIPv6) at System.Net.Dns.GetHostEntry(String hostNameOrAddress) at System.ServiceModel.Channels.DnsCache.Resolve(Uri uri) --- End of inner exception stack trace --- Server stack trace: at System.ServiceModel.Channels.DnsCache.Resolve(Uri uri)
    ```
    
1. Para solucionar este problema, pruebe uno de los métodos siguientes o ambos:
    - Colocar todos los nodos en el mismo dominio
    - Agregar la asignación de IP a host en todos los archivos de hosts de la máquina virtual hospedada

### <a name="connectivity-issue-between-the-self-hosted-ir-and-your-data-factory-instance-or-the-self-hosted-ir-and-the-data-source-or-sink"></a>Problema de conectividad entre el IR autohospedado y la instancia de Data Factory, o entre el IR autohospedado y el origen o receptor de datos

Para solucionar el problema de conectividad de red, debe saber cómo recopilar el seguimiento de red, entender cómo usarlo y [analizar el seguimiento de Monitor de red de Microsoft (Netmon)](#analyze-the-netmon-trace) antes de aplicar las herramientas de Netmon en casos reales desde el IR autohospedado.

#### <a name="symptoms"></a>Síntomas

En ocasiones, es posible que necesite solucionar determinados problemas de conectividad entre el IR autohospedado y la instancia de Data Factory, como se muestra en la captura de pantalla siguiente, o entre el IR autohospedado y el origen o el receptor de datos. 

![Captura de pantalla de un mensaje "Error de solicitud HTTP procesada".](media/self-hosted-integration-runtime-troubleshoot-guide/http-request-error.png)

En cualquier caso, puede que encuentre los siguientes errores:

* "Error de copia:Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=No se puede conectar a SQL Server: 'dirección IP'"

* "Se han producido uno o varios errores. Error al enviar la solicitud. Se ha cerrado la conexión subyacente: Error inesperado de recepción. No se pueden leer los datos de la conexión de transporte: El host remoto forzó el cierre de la conexión existente. El host remoto forzó el cierre de la conexión existente Id. de actividad."

#### <a name="resolution"></a>Solución

Cuando encuentre los errores anteriores, soluciónelos siguiendo las instrucciones de esta sección.

- Recopile un seguimiento de Netmon para el análisis: 

    1. Puede establecer el filtro para ver un restablecimiento del servidor en el lado cliente. En la captura de pantalla de ejemplo siguiente, puede ver que el lado servidor es el servidor de Data Factory.

        ![Captura de pantalla del servidor de Data Factory.](media/self-hosted-integration-runtime-troubleshoot-guide/data-factory-server.png)

    1. Al obtener el paquete de restablecimiento, puede encontrar la conversación si sigue el Protocolo de control de transmisión (TCP).

        ![Captura de pantalla de la conversación de TCP.](media/self-hosted-integration-runtime-troubleshoot-guide/find-conversation.png)

    1. Para obtener la conversación entre el cliente y el servidor de Data Factory, quite el filtro.

        ![Captura de pantalla de los detalles de la conversación.](media/self-hosted-integration-runtime-troubleshoot-guide/get-conversation.png)

- Un análisis del seguimiento de Netmon que ha recopilado muestra que el total del período de vida (TTL) es 64. Según los valores mencionados en el artículo de [conceptos básicos del período de vida (TTL) de IP y límite de salto](https://packetpushers.net/ip-time-to-live-and-hop-limit-basics/), como se extraen en la lista siguiente, se aprecia que es el sistema Linux el que restablece el paquete y provoca la desconexión.

    Los valores predeterminados de TTL y límite de salto varían entre distintos sistemas operativos, como se puede ver aquí:
    - Kernel de Linux 2.4 (2001 aproximadamente): 255 para TCP, Protocolo de datagramas de usuario (UDP) y Protocolo de mensajes de control de Internet (ICMP)
    - Kernel de Linux 4.10 (2015): 64 para TCP, UDP e ICMP
    - Windows XP (2001): 128 para TCP, UDP e ICMP
    - Windows 10 (2015): 128 para TCP, UDP e ICMP
    - Windows Server 2008: 128 para TCP, UDP e ICMP
    - Windows Server 2019 (2018): 128 para TCP, UDP e ICMP
    - macOS (2001): 64 para TCP, UDP e ICMP

    ![Captura de pantalla que muestra un valor de TTL de 61.](media/self-hosted-integration-runtime-troubleshoot-guide/ttl-61.png)
    
    En el ejemplo anterior, el TTL se muestra como 61 en lugar de 64, porque cuando el paquete de red alcanza su destino, debe atravesar varios saltos, como enrutadores o dispositivos de red. El número de enrutadores o dispositivos de red se deduce para generar el TTL final.
    
    En este caso, se ve que es posible que el restablecimiento se haya enviado desde el sistema Linux con TTL 64.

-  Para confirmar de dónde puede provenir el dispositivo de restablecimiento, compruebe el cuarto salto desde el IR autohospedado.
 
    *Paquete de red del sistema Linux A con TTL 64 -> B TTL 64 menos 1 = 63 -> C TTL 63 menos 1 = 62 -> TTL 62 menos 1 = 61 IR autohospedado*

- En una situación ideal, el número de saltos de TTL sería 128, lo que significa que el sistema operativo Windows está ejecutando la instancia de Data Factory. Como se muestra en el ejemplo siguiente, *128 menos 107 = 21 saltos*, es decir, se enviaron 21 saltos para el paquete desde la instancia de Data Factory al IR autohospedada durante el protocolo de enlace de TCP 3.
 
    ![Captura de pantalla que muestra un valor de TTL de 107.](media/self-hosted-integration-runtime-troubleshoot-guide/ttl-107.png)

    Por tanto, debe comunicarle al equipo de red que compruebe cuál es el cuarto salto desde el IR autohospedado. Si es el firewall, como con el sistema Linux, compruebe en los registros por qué el dispositivo restablece el paquete después del protocolo de enlace TCP 3. 
    
    Si no está seguro de dónde investigar, intente obtener el seguimiento de Netmon tanto desde el IR autohospedado como desde el firewall durante el tiempo problemático. Este enfoque le ayudará a averiguar qué dispositivo podría haber restablecido el paquete y causado la desconexión. En este caso, también debe indicarle al equipo de red que avance.

### <a name="analyze-the-netmon-trace"></a>Análisis del seguimiento de Netmon

> [!NOTE] 
> Las siguientes instrucciones se aplican al seguimiento de Netmon. Dado que el seguimiento de Netmon no recibe actualmente soporte técnico, puede usar Wireshark a tales efectos.

Al intentar conectarse mediante Telnet a **8.8.8.8 888** con el seguimiento de Netmon recopilado, debería ver el seguimiento en las siguientes capturas de pantallas:

![Captura de pantalla que muestra el mensaje de error "No se pudo abrir la conexión con el host en el puerto 888".](media/self-hosted-integration-runtime-troubleshoot-guide/netmon-trace-1.png)

![Captura de pantalla que muestra una descripción del seguimiento de Netmon.](media/self-hosted-integration-runtime-troubleshoot-guide/netmon-trace-2.png)
 

Las imágenes anteriores muestran que no pudo establecer una conexión TCP con el lado servidor **8.8.8.8** en el puerto **888**, por lo que ve dos paquetes **SynReTransmit** adicionales. Dado que el origen **SELF-HOST2** no se pudo conectar a **8.8.8.8** con el primer paquete, seguirá intentando establecer la conexión.

> [!TIP]
> Para establecer esta conexión, pruebe la siguiente solución:
> 1. Seleccione **Cargar filtro** > **Filtro estándar** > **Direcciones** > **Direcciones IPv4**.
> 1. Para aplicar el filtro, escriba **IPv4.Address == 8.8.8.8** y, después, seleccione **Aplicar**. A continuación, debería ver la comunicación desde la máquina local al destino **8.8.8.8**.

![Captura de pantalla que muestra las direcciones de filtrado](media/self-hosted-integration-runtime-troubleshoot-guide/filter-addresses-1.png)
        
![Captura de pantalla que muestra más direcciones de filtrado](media/self-hosted-integration-runtime-troubleshoot-guide/filter-addresses-2.png)

En los ejemplos siguientes se muestran los escenarios correctos: 

- Si puede conectarse mediante Telnet a **8.8.8.8 53** sin problemas, hay un protocolo de enlace de TCP 3 correcto, y la sesión finaliza con un protocolo de enlace TCP 4.

    ![Captura de pantalla que muestra un escenario de conexión correcta.](media/self-hosted-integration-runtime-troubleshoot-guide/good-scenario-1.png)
     
    ![Captura de pantalla que muestra los detalles de un escenario de conexión correcta.](media/self-hosted-integration-runtime-troubleshoot-guide/good-scenario-2.png)

- El protocolo de enlace TCP 3 anterior produce el siguiente flujo de trabajo:

    ![Diagrama del flujo de trabajo de un protocolo de enlace TCP 3.](media/self-hosted-integration-runtime-troubleshoot-guide/tcp-3-handshake-workflow.png)
 
- El protocolo de enlace TCP 4 para finalizar la sesión se muestra en los siguientes flujos de trabajo:

    ![Captura de pantalla de los detalles del protocolo de enlace TCP 4.](media/self-hosted-integration-runtime-troubleshoot-guide/tcp-4-handshake.png)

    ![Diagrama del flujo de trabajo de un protocolo de enlace TCP 4.](media/self-hosted-integration-runtime-troubleshoot-guide/tcp-4-handshake-workflow.png) 

### <a name="microsoft-email-notification-about-updating-your-network-configuration"></a>Notificación por correo electrónico de Microsoft sobre la actualización de la configuración de red

Es posible que reciba la siguiente notificación por correo electrónico, que le recomienda actualizar la configuración de red para permitir la comunicación con nuevas direcciones IP para Azure Data Factory antes del 8 de noviembre de 2020:

   ![Captura de pantalla de la notificación por correo electrónico de Microsoft que solicita la actualización de la configuración de red.](media/self-hosted-integration-runtime-troubleshoot-guide/email-notification.png)

#### <a name="determine-whether-this-notification-affects-you"></a>Determinación de si esta notificación le afecta

Esta notificación se aplica a los escenarios siguientes:

##### <a name="scenario-1-outbound-communication-from-a-self-hosted-integration-runtime-thats-running-on-premises-behind-a-corporate-firewall"></a>Escenario 1: comunicación saliente desde un entorno de ejecución de integración autohospedado que se ejecuta en el entorno local detrás de un firewall corporativo

Cómo determinar si se ve afectado:

- *No se verá afectado* si define reglas de firewall basadas en nombres de dominio completos (FQDN) que usen el enfoque descrito en [Configuraciones de firewall y configuración de direcciones IP permitidas](data-movement-security-considerations.md#firewall-configurations-and-allow-list-setting-up-for-ip-addresses).

- *Se verá afectado* si habilita de forma explícita la lista de direcciones IP de salida permitidas en el firewall corporativo.

   Si se ve afectado, adopte la medida siguiente: pida al equipo de infraestructura de red que actualice la configuración de red para usar las direcciones IP de Data Factory más recientes antes del 8 de noviembre de 2020. Para descargar las direcciones IP más recientes, vaya a [Detección de etiquetas de servicio mediante archivos JSON descargables](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files).

##### <a name="scenario-2-outbound-communication-from-a-self-hosted-integration-runtime-thats-running-on-an-azure-vm-inside-a-customer-managed-azure-virtual-network"></a>Escenario 2: comunicación saliente desde un entorno de ejecución de integración autohospedado que se ejecuta en una VM de Azure dentro de una red virtual de Azure administrada por el cliente

Cómo determinar si se ve afectado:

- Compruebe si tiene reglas de grupo de seguridad de red (NSG) de salida en una red privada que contiene el entorno de ejecución de integración autohospedado. Si no hay restricciones de salida, no se verá afectado.

- Si tiene restricciones de reglas de salida, compruebe si usa etiquetas de servicio. Si usa etiquetas de servicio, no se verá afectado. No es necesario cambiar ni agregar nada, ya que el nuevo intervalo IP está bajo las etiquetas de servicio existentes. 

  ![Captura de pantalla de una comprobación de destino que muestra DataFactory como destino.](media/self-hosted-integration-runtime-troubleshoot-guide/destination-check.png)

- *Se verá afectado* si habilita de forma explícita la lista de direcciones IP salientes permitidas en la configuración de las reglas de NSG en la red virtual de Azure.

   Si se ve afectado, adopte la medida siguiente: pida al equipo de infraestructura de red que actualice las reglas de NSG en la configuración de red virtual de Azure para usar las direcciones IP de Data Factory más recientes antes del 8 de noviembre de 2020. Para descargar las direcciones IP más recientes, vaya a [Detección de etiquetas de servicio mediante archivos JSON descargables](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files).

##### <a name="scenario-3-outbound-communication-from-ssis-integration-runtime-in-a-customer-managed-azure-virtual-network"></a>Escenario 3: comunicación saliente de SSIS Integration Runtime en una red virtual de Azure administrada por el cliente

Cómo determinar si se ve afectado:

- Compruebe si tiene reglas de NSG de salida en una red privada que contiene SQL Server Integration Services (SSIS) Integration Runtime. Si no hay restricciones de salida, no se verá afectado.

- Si tiene restricciones de reglas de salida, compruebe si usa etiquetas de servicio. Si usa etiquetas de servicio, no se verá afectado. No es necesario cambiar ni agregar nada, ya que el nuevo intervalo IP está bajo las etiquetas de servicio existentes.

- *Se verá afectado* si habilita de forma explícita la lista de direcciones IP salientes permitidas en la configuración de las reglas de NSG en la red virtual de Azure.

  Si se ve afectado, adopte la medida siguiente: pida al equipo de infraestructura de red que actualice las reglas de NSG en la configuración de red virtual de Azure para usar las direcciones IP de Data Factory más recientes antes del 8 de noviembre de 2020. Para descargar las direcciones IP más recientes, vaya a [Detección de etiquetas de servicio mediante archivos JSON descargables](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files).

### <a name="couldnt-establish-a-trust-relationship-for-the-ssltls-secure-channel"></a>No se pudo establecer una relación de confianza para el canal seguro SSL/TLS 

#### <a name="symptoms"></a>Síntomas

El IR autohospedado no puede conectarse al servicio Azure Data Factory.

Al comprobar el registro de eventos de IR autohospedado o los registros de notificación del cliente en la tabla CustomLogEvent, encontrará el siguiente mensaje de error:

"The underlying connection was closed: No se ha podido establecer la relación de confianza para el canal seguro SSL/TLS. El certificado remoto no es válido según el procedimiento de validación".

La manera más sencilla de comprobar el certificado de servidor del servicio Data Factory es abrir la dirección URL del servicio Data Factory en el explorador. Por ejemplo, abra el [vínculo para comprobar el certificado de servidor](https://eu.frontend.clouddatahub.net/) en la máquina donde está instalado el IR autohospedado y, a continuación, vea la información del certificado de servidor.

  ![Captura de pantalla del panel de comprobación del certificado de servidor del servicio Azure Data Factory.](media/self-hosted-integration-runtime-troubleshoot-guide/server-certificate.png)

  ![Captura de pantalla de la ventana para comprobar la ruta del certificación de servidor.](media/self-hosted-integration-runtime-troubleshoot-guide/certificate-path.png)

#### <a name="cause"></a>Causa

Este problema se debe a dos motivos posibles:

- Motivo 1: La CA raíz del certificado de servidor del servicio Data Factory no es de confianza en la máquina donde está instalado el IR autohospedado. 
- Motivo 2: Usa un proxy en el entorno, y el certificado de servidor del servicio Data Factory se sustituye por el proxy, y el certificado de servidor reemplazado no es de confianza para la máquina donde está instalado el IR autohospedado.

#### <a name="resolution"></a>Solución

- Para el motivo 1: Asegúrese de que el certificado de servidor de Data Factory y su cadena de certificados sean de confianza para la máquina donde está instalado el IR autohospedado.
- Para el motivo 2: Confíe en la CA raíz reemplazada en la máquina con el IR autohospedado, o bien configure el proxy para que no reemplace el certificado de servidor de Data Factory.

Para obtener más información sobre la confianza de certificados en Windows, consulte [Instalación del certificado raíz de confianza](/skype-sdk/sdn/articles/installing-the-trusted-root-certificate).

#### <a name="additional-information"></a>Información adicional
Hemos implementado un nuevo certificado SSL, que se firma desde DigiCert. Compruebe si DigiCert Global Root G2 está en la CA raíz de confianza.

  ![Captura de pantalla que muestra la carpeta DigiCert Global Root G2 en el directorio Entidades de certificación raíz de confianza.](media/self-hosted-integration-runtime-troubleshoot-guide/trusted-root-ca-check.png)

Si no está en la CA raíz de confianza, [descárguelo aquí](http://cacerts.digicert.com/DigiCertGlobalRootG2.crt ). 


## <a name="next-steps"></a>Pasos siguientes

Para obtener más ayuda para solucionar problemas, pruebe los siguientes recursos:

*  [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Página de preguntas y respuestas de Microsoft](/answers/topics/azure-data-factory.html)
*  [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)
*  [Guía de rendimiento de flujos de datos de asignación](concepts-data-flow-performance.md)
