---
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.topic: include
ms.date: 04/28/2019
ms.author: ramamill
ms.openlocfilehash: bff9457220efbb6a6b318550a5b2dd9af4ae230f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96019209"
---
Siga los pasos para sus circunstancias específicas.

### <a name="unregister-a-connected-process-server"></a>Anulación del registro de un servidor de procesos conectado

1. Establezca una conexión remota al servidor de procesos como administrador.
2. En el **Panel de Control**, abra **Programas > Desinstalar un programa**.
3. Desinstale el programa **Microsoft Azure Site Recovery Mobility Service/Servidor de destino maestro**.
4. Desinstale el programa **Configuración de Microsoft Azure Site Recovery/Servidor de procesos**.
5. Después de desinstalan los programas en los pasos 3 y 4, desinstale **Configuración de Microsoft Azure Site Recovery/Dependencias del servidor de procesos**.

### <a name="unregister-a-disconnected-process-server"></a>Anulación del registro de un servidor de procesos desconectado

Use solamente estos pasos si no hay ninguna manera de reactivar la máquina donde está instalado el servidor de procesos.

1. Inicie sesión como administrador en el servidor de configuración.
2. Abra un símbolo del sistema administrativo y vaya a `%ProgramData%\ASR\home\svsystems\bin`.
3. Ejecute este comando para obtener una lista de uno o más servidores de procesos.

    ```
    perl Unregister-ASRComponent.pl -IPAddress <IP_of_Process_Server> -Component PS
    ```
    - S. No: el número de serie del servidor de procesos.
    - IP/Nombre: la dirección IP y el nombre de la máquina que ejecuta el servidor de procesos.
    - Heartbeat: último latido de la máquina del servidor de procesos.
    ![En la captura de pantalla se muestra una presentación de texto sin formato con información sobre los servidores de procesos y el texto Please choose one of the above servers to un-register (Elija uno de los servidores anteriores para anular el registro). (media/site-recovery-vmware-unregister-process-server/Unregister-cmd.PNG)

4. Especifique el número de serie del servidor de procesos cuyo registro quiera anular.
5. Al anular el registro de un servidor de procesos, se quitan todos sus detalles del sistema y se muestra el mensaje: **Se eliminó a <nombre del servidor> del registro correctamente (dirección IP del servidor)**

