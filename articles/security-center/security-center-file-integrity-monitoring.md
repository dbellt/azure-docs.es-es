---
title: Supervisión de la integridad de los archivos en Azure Security Center
description: Aprenda a configurar la supervisión de la integridad de los archivos (FIM) en Azure Security Center mediante este tutorial.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 04/25/2021
ms.author: memildin
ms.openlocfilehash: e14307207ddbe9f1b89bd05d7015dafd76b10d51
ms.sourcegitcommit: 18cd3c1c8cc47258c6a1a04e0e03d6248c52ef24
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107992501"
---
# <a name="file-integrity-monitoring-in-azure-security-center"></a>Supervisión de la integridad de los archivos en Azure Security Center
Aprenda a configurar la supervisión de la integridad de los archivos (FIM) en Azure Security Center mediante este tutorial.


## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponibilidad general (GA)|
|Precios:|Requiere [Azure Defender para servidores](defender-for-servers-introduction.md).<br>FIM carga datos en el área de trabajo de Log Analytics. Se aplican cargos de datos, según la cantidad de datos que cargue. Para más información, consulte [Precios de Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/).|
|Roles y permisos necesarios:|El **propietario del área de trabajo** puede habilitar o deshabilitar FIM (para obtener más información, consulte [Roles de Azure para Log Analytics](/services-hub/health/azure-roles#azure-roles)).<br>El **lector** puede ver los resultados.|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)<br>Solo se admite en las regiones en las que está disponible la solución de seguimiento de cambios de Azure Automation.<br>![Sí](./media/icons/yes-icon.png) Dispositivos habilitados para [Azure Arc](../azure-arc/servers/overview.md).<br>Consulte [Regiones admitidas para el área de trabajo de Log Analytics vinculada](../automation/how-to/region-mappings.md).<br>[Más información sobre el seguimiento de cambios](../automation/change-tracking/overview.md).|
|||

## <a name="what-is-fim-in-security-center"></a>¿Qué la supervisión de la integridad de los archivos (FIM) Security Center?
La supervisión de la integridad de los archivos (FIM), conocida también como supervisión de los cambios, examina los archivos del sistema operativo, los registros de Windows, el software de aplicación, los archivos del sistema Linux, etc., para comprobar la existencia de cambios que podrían indicar un ataque. 

Security Center recomienda que las entidades se supervisen con FIM. Además, puede definir sus propias directivas o entidades de FIM para supervisar. FIM le alerta sobre actividades sospechosas como:

- Eliminación y creación de archivos y clave del Registro
- Modificaciones de archivos (cambios en el tamaño de archivo, listas de control de acceso y hash del contenido)
- Modificaciones del Registro (cambios en el tamaño, listas de control de acceso, tipo y contenido)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Revisar la lista de entidades sugeridas para supervisar con FIM.
> * Definir sus propias reglas de FIM personalizadas.
> * Auditar los cambios en las entidades supervisadas.
> * Utilizar caracteres comodín para simplificar el seguimiento a través de directorios.


## <a name="how-does-fim-work"></a>¿Cómo funciona FIM?

Al comparar el estado actual de estos elementos con el estado durante el examen anterior, FIM le avisa si se han realizado modificaciones sospechosas.

FIM emplea la solución Azure Change Tracking para realizar el seguimiento de los cambios e identificarlos en su entorno. Cuando la supervisión de la integridad de los archivos está habilitada, tendrá un recurso **Change Tracking** de tipo **Solución**. Para obtener detalles sobre la frecuencia de recopilación de datos, consulte [Detalles de la recopilación de datos de Change Tracking](../automation/change-tracking/overview.md#change-tracking-and-inventory-data-collection).

> [!NOTE]
> Si quita el recurso **Change Tracking**, deshabilitará también la característica de supervisión de la integridad de los archivos en Security Center.

## <a name="which-files-should-i-monitor"></a>¿Qué archivos se deben supervisar?

Al elegir los archivos que quiere supervisar, es conveniente que piense en los archivos que son críticos para su sistema y aplicaciones. Supervise archivos que no espera que cambien sin haberlo planeado. Si elige archivos que las aplicaciones o el sistema operativo cambian con frecuencia (por ejemplo, archivos de registro y archivos de texto), se genera mucho ruido que dificulta la identificación de un ataque.

Security Center proporciona la siguiente lista de elementos cuya supervisión se recomienda según los patrones de ataque conocidos.

|Archivos de Linux|Archivos de Windows|Claves del Registro de Windows (HKLM = HKEY_LOCAL_MACHINE)|
|:----|:----|:----|
|/bin/login|C:\autoexec.bat|HKLM\SOFTWARE\Microsoft\Cryptography\OID\EncodingType 0\CryptSIPDllRemoveSignedDataMsg\{C689AAB8-8E78-11D0-8C47-00C04FC295EE}|
|/bin/passwd|C:\boot.ini|HKLM\SOFTWARE\Microsoft\Cryptography\OID\EncodingType 0\CryptSIPDllRemoveSignedDataMsg\{603BCC1F-4B59-4E08-B724-D2C6297EF351}|
|/etc/*.conf|C:\config.sys|HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\IniFileMapping\SYSTEM.ini\boot|
|/usr/bin|C:\Windows\system.ini|HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows|
|/usr/sbin|C:\Windows\win.ini|HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon|
|/bin|C:\Windows\regedit.exe|HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders|
|/sbin|C:\Windows\System32\userinit.exe|HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders|
|/boot|C:\Windows\explorer.exe|HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run|
|/usr/local/bin|C:\Archivos de programa\Microsoft Security Client\msseces.exe|HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce|
|/usr/local/sbin||HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnceEx|
|/opt/bin||HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServices|
|/opt/sbin||HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServicesOnce|
|/etc/crontab||HKLM\SOFTWARE\WOW6432Node\Microsoft\Cryptography\OID\EncodingType 0\CryptSIPDllRemoveSignedDataMsg\{C689AAB8-8E78-11D0-8C47-00C04FC295EE}|
|/etc/init.d||HKLM\SOFTWARE\WOW6432Node\Microsoft\Cryptography\OID\EncodingType 0\CryptSIPDllRemoveSignedDataMsg\{603BCC1F-4B59-4E08-B724-D2C6297EF351}|
|/etc/cron.hourly||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\IniFileMapping\system.ini\boot|
|/etc/cron.daily||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Windows|
|/etc/cron.weekly||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Winlogon|
|/etc/cron.monthly||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\RunOnce|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\RunOnceEx|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\RunServices|
|||HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce|
|||HKLM\SYSTEM\CurrentControlSet\Control\hivelist|
|||HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\KnownDLLs|
|||HKLM\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\DomainProfile|
|||HKLM\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\PublicProfile|
|||HKLM\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy\StandardProfile|


## <a name="enable-file-integrity-monitoring"></a>Habilitación de la supervisión de la integridad de los archivos 

FIM solo está disponible desde las páginas de Security Center en Azure Portal. Actualmente no hay ninguna API REST que funcione con FIM.

1. En el área **Protección avanzada** del panel de **Azure Defender**, seleccione **Supervisión de la integridad de los archivos**.

   :::image type="content" source="./media/security-center-file-integrity-monitoring/open-file-integrity-monitoring.png" alt-text="Inicio de FIM" lightbox="./media/security-center-file-integrity-monitoring/open-file-integrity-monitoring.png":::

    Se abre la página de configuración **Supervisión de la integridad de los archivos**.

    Se proporciona la siguiente información para cada área de trabajo:

    - Número total de cambios que se produjeron durante la última semana (puede ver un guion "-" si FIM no está habilitado en el área de trabajo)
    - Número total de equipos y las máquinas virtuales que informan al área de trabajo
    - Ubicación geográfica del área de trabajo
    - Suscripción de Azure en la que se encuentra el área de trabajo

1. Use esta página para:

    - Acceder y ver el estado y la configuración de cada área de trabajo.

    - ![Icono Actualizar plan][4] Actualizar el área de trabajo para usar Azure Defender. Este icono indica que el área de trabajo o la suscripción no están protegidas por Azure Defender. Para usar las características FIM, la suscripción debe estar protegida por Azure Defender. Para más información, consulte [Azure Security Center gratuito frente a Azure Defender habilitado](security-center-pricing.md).

    - ![Icono Habilitar][3] Habilitar FIM en todas las máquinas del área de trabajo y configurar las opciones de FIM. Este icono indica que FIM no está habilitada en el área de trabajo.

        :::image type="content" source="./media/security-center-file-integrity-monitoring/workspace-list-fim.png" alt-text="Habilitación de FIM para un área de trabajo específica":::


    > [!TIP]
    > Si no hay ningún botón Habilitar o Actualizar y el espacio está en blanco, significa que FIM ya está habilitado en el área de trabajo.


1. Seleccione **HABILITAR**. Se muestran los detalles del área de trabajo, incluido el número de equipos con Windows y Linux en el área de trabajo.

    :::image type="content" source="./media/security-center-file-integrity-monitoring/workspace-fim-status.png" alt-text="Página de detalles del área de trabajo de FIM":::

   También se muestra la configuración recomendada para Windows y Linux.  Expanda **Archivos de Windows**, **Registro** y **Archivos de Linux** para ver la lista completa de elementos recomendados.

1. Desactive las casillas de las entidades recomendadas que no quiere que supervise FIM.

1. Seleccione **Aplicar la supervisión de la integridad de los archivos** para habilitar FIM.

> [!NOTE]
> Puede cambiar la configuración en cualquier momento. Consulte a continuación [Edición de las entidades supervisadas](#edit-monitored-entities) para más información.



## <a name="audit-monitored-workspaces"></a>Auditoría de áreas de trabajo supervisadas 

El panel **Supervisión de la integridad de los archivos** se muestra en las áreas de trabajo en las que está habilitado FIM. El panel de FIM se abre después de habilitar FIM en un área de trabajo o al seleccionar un área de trabajo en la ventana **Supervisión de la integridad de los archivos** que ya tiene FIM habilitado.

:::image type="content" source="./media/security-center-file-integrity-monitoring/fim-dashboard.png" alt-text="Panel de FIM y sus distintos paneles informativos":::

El panel de FIM de un área de trabajo muestra los detalles siguientes:

- Número total de máquinas conectadas al área de trabajo
- Número total de cambios que se produjeron durante el período de tiempo seleccionado
- Un desglose del tipo de cambio (archivos, registro)
- Un desglose de la categoría del cambio (modificado, agregado, quitado)

Seleccione **Filtrar** en la parte superior del panel para cambiar el período de tiempo durante el que se muestran los cambios.

:::image type="content" source="./media/security-center-file-integrity-monitoring/dashboard-filter.png" alt-text="Filtro de período de tiempo del panel de FIM":::

En la pestaña **Servidores** se enumeran las máquinas que informan a esta área de trabajo. En el panel se muestra para cada máquina:

- Número total de cambios que se produjeron durante el período de tiempo seleccionado
- Un desglose de los cambios totales a medida que se producen cambios en los archivos o el registro

Al seleccionar una máquina, la consulta aparece junto con los resultados que identifican los cambios realizados durante el período de tiempo seleccionado para la máquina. Los cambios se pueden expandir para obtener más información.

:::image type="content" source="./media/security-center-file-integrity-monitoring/query-machine-changes.png" alt-text="Consulta de Log Analytics que muestra los cambios identificados por la supervisión de la integridad de los archivos de Azure Security Center" lightbox="./media/security-center-file-integrity-monitoring/query-machine-changes.png":::

La pestaña **Cambios** (se muestra a continuación) enumera todos los cambios del área de trabajo durante el período de tiempo seleccionado. En el panel se muestra para las entidades que se han cambiado:

- Máquina en la que se produjo el cambio
- Tipo de cambio (registro o archivo)
- Categoría del cambio (modificado, agregado, eliminado)
- Fecha y hora del cambio

:::image type="content" source="./media/security-center-file-integrity-monitoring/changes-tab.png" alt-text="Pestaña de cambios de la supervisión de la integridad de los archivos de Azure Security Center" lightbox="./media/security-center-file-integrity-monitoring/changes-tab.png":::

Se abre la hoja **Cambiar detalles** al escribir un cambio en el campo de búsqueda o al seleccionar una entidad enumerada en la pestaña **Cambios**.

:::image type="content" source="./media/security-center-file-integrity-monitoring/change-details.png" alt-text="Supervisión de la integridad de los archivos de Azure Security Center que muestra el panel de detalles de un cambio" lightbox="./media/security-center-file-integrity-monitoring/change-details.png":::

## <a name="edit-monitored-entities"></a>Edición de las entidades supervisadas

1. En el **panel Supervisión de la integridad de los archivos** de un área de trabajo, seleccione **Configuración** en la barra de herramientas. 

    :::image type="content" source="./media/security-center-file-integrity-monitoring/file-integrity-monitoring-dashboard-settings.png" alt-text="Acceso a la configuración de supervisión de la integridad de los archivos de un área de trabajo" lightbox="./media/security-center-file-integrity-monitoring/file-integrity-monitoring-dashboard-settings.png":::

   La **configuración del área de trabajo** se abre con pestañas para cada tipo de elemento que se puede supervisar:

      - Registro de Windows
      - Archivos de Windows
      - Archivos de Linux
      - Contenido del archivo
      - Servicios de Windows

      En cada pestaña se muestran las entidades que se pueden editar de esa categoría. En cada entidad de la lista, Security Center identifica si FIM está habilitado (true) o deshabilitado (false).  Al editar la entidad, puede habilitar o deshabilitar FIM.

    :::image type="content" source="./media/security-center-file-integrity-monitoring/file-integrity-monitoring-workspace-configuration.png" alt-text="Configuración del área de trabajo para la supervisión de la integridad de los archivos en Azure Security Center":::

1. Seleccione una entrada de una de las pestañas y edite cualquiera de los campos disponibles en el panel **Editar Change Tracking**. Las opciones son:

    - Habilitar (true) o deshabilitar (false) la supervisión de la integridad de los archivos
    - Proporcionar o cambiar el nombre de la entidad
    - Proporcionar o cambiar el valor o la ruta de acceso
    - Eliminar la entidad

1. Descarte o guarde los cambios.


## <a name="add-a-new-entity-to-monitor"></a>Agregar una nueva entidad para supervisar

1. En el **panel Supervisión de la integridad de los archivos** de un área de trabajo, seleccione **Configuración** en la barra de herramientas. 

    Se abre la hoja **Configuración del área de trabajo**.

1. Abra la **Configuración del área de trabajo**:

    1. Seleccione la pestaña correspondiente al tipo de entidad que quiere agregar: Registro de Windows, Archivos de Windows, Archivos de Linux, Contenido del archivo o Servicios de Windows. 
    1. Seleccione **Agregar**. 

        En este ejemplo, hemos seleccionado **Archivos de Linux**.

        :::image type="content" source="./media/security-center-file-integrity-monitoring/file-integrity-monitoring-add-element.png" alt-text="Adición de un elemento para supervisarlo en la supervisión de la integridad de los archivos de Azure Security Center" lightbox="./media/security-center-file-integrity-monitoring/file-integrity-monitoring-add-element.png":::

1. Seleccione **Agregar**. Se abre la hoja **Add for Change Tracking** (Agregar para Change Tracking).

1. Escriba la información necesaria y seleccione **Guardar**.

## <a name="folder-and-path-monitoring-using-wildcards"></a>Supervisión de carpetas y rutas de acceso mediante caracteres comodín.

Utilice caracteres comodín para simplificar el seguimiento a través de directorios. Las siguientes reglas se aplican cuando configura la supervisión de carpetas mediante caracteres comodín:
-   Los caracteres comodín son necesarios para realizar el seguimiento de varios archivos.
-   Los caracteres comodín solo se pueden usar en el último segmento de una ruta de acceso, como C:\folder\file o /etc/*.conf.
-   Si una variable de entorno tiene una ruta de acceso no válida, la validación será correcta, pero se producirá un error en dicha ruta cuando se ejecute el inventario.
-   Al establecer la ruta de acceso, evite las rutas de acceso generales como c:\*.*, ya que daría lugar a que se recorrieran demasiadas carpetas.

## <a name="disable-fim"></a>Deshabilitación de FIM
Puede deshabilitar FIM. FIM emplea la solución Azure Change Tracking para realizar el seguimiento de los cambios e identificarlos en su entorno. Al deshabilitar FIM, quita la solución Change Tracking del área de trabajo seleccionada.

Para deshabilitar FIM:

1. En el **panel Supervisión de la integridad de los archivos** de un área de trabajo, seleccione **Deshabilitar**.

    :::image type="content" source="./media/security-center-file-integrity-monitoring/disable-file-integrity-monitoring.png" alt-text="Deshabilitación de la supervisión de la integridad de los archivos desde la página de configuración":::

1. Seleccione **Quitar**.

## <a name="next-steps"></a>Pasos siguientes
En este artículo ha obtenido información sobre cómo usar la Supervisión de la integridad de los archivos (FIM) de Security Center. Para obtener más información sobre Security Center, vea las páginas siguientes:

* [Establecimiento de directivas de seguridad](tutorial-security-policy.md): aprenda a configurar directivas de seguridad para las suscripciones y los grupos de recursos de Azure.
* [Administración de recomendaciones de seguridad](security-center-recommendations.md): conozca una serie de recomendaciones que le ayudarán a proteger los recursos de Azure.
* [Blog de seguridad de Azure](/archive/blogs/azuresecurity/): obtenga las últimas noticias e información sobre la seguridad en Azure.

<!--Image references-->
[3]: ./media/security-center-file-integrity-monitoring/enable.png
[4]: ./media/security-center-file-integrity-monitoring/upgrade-plan.png