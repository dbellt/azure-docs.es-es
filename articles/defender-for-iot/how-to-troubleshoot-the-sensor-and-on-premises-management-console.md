---
title: Solución de problemas del sensor y de la consola de administración local
description: Solucione los problemas de su sensor y la consola de administración local para eliminar los problemas que pueda tener.
ms.date: 04/22/2021
ms.topic: article
ms.openlocfilehash: aac8fb9d3b6c102d72cf427cae46b8530b94cc0f
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108734640"
---
# <a name="troubleshoot-the-sensor-and-on-premises-management-console"></a>Solución de problemas del sensor y de la consola de administración local

En este artículo se describen las herramientas básicas de solución de problemas para el sensor y la consola de administración local. Además de los elementos que se describen aquí, puede comprobar el estado del sistema de las siguientes maneras:

**Alertas**: Cuando la interfaz de sensor que supervisa el tráfico está inactiva, se crea una alerta. 

**SNMP**: El estado del sensor se supervisa mediante SNMP. Azure Defender para IoT responde a las consultas SNMP enviadas desde un servidor de supervisión autorizado. 

**Notificaciones del sistema**: Cuando una consola de administración controla el sensor, puede reenviar alertas sobre las copias de seguridad del sensor con errores y los sensores desconectados.

## <a name="sensor-troubleshooting-tools"></a>Herramienta para la solución de problemas del sensor

### <a name="investigate-password-failure-at-initial-sign-in"></a>Investigar el error de contraseña en el inicio de sesión inicial

Al iniciar sesión en un sensor Arrow preconfigurado por primera vez, deberá realizar la recuperación de contraseña.

Para recuperar la contraseña:

1. En la pantalla de inicio de sesión de Defender para IoT, seleccione la opción **Recuperación de contraseña**. Se abre la pantalla **Recuperación de contraseña**.

1. Seleccione **CyberX** o **Soporte técnico** y copie el identificador único.

1. Vaya a Azure Portal y seleccione **Sitios y sensores**.  

1. Seleccione la pestaña **Recuperar la contraseña de la consola de administración local**.

   :::image type="content" source="media/password-recovery-images/recover-button.png" alt-text="Selección del botón de recuperación de administración local para descargar el archivo de recuperación.":::

1. Escriba el identificador único que recibió en la pantalla **Recuperación de contraseña** y seleccione **Recuperar**. Se descarga el archivo `password_recovery.zip`.

    > [!NOTE]
    > No modifique el archivo de recuperación de contraseña. Es un archivo firmado y no funcionará si se altera.

1. En la pantalla **Recuperación de contraseña**, seleccione **Cargar**. Se abrirá la ventana **Cargar archivo de recuperación de contraseña**.

1. Seleccione **Examinar** para buscar el archivo `password_recovery.zip`, o arrástrelo a la ventana.

1. Seleccione **Siguiente**; aparecerá el usuario y la contraseña generada por el sistema para la consola de administración.

    > [!NOTE]
    > Cuando inicie sesión en un sensor o en una consola de administración local por primera vez, se vinculará a la suscripción a la que lo conectó. Si necesita restablecer la contraseña para el usuario de CyberX o soporte técnico, deberá seleccionar esa suscripción. Para obtener más información sobre cómo recuperar una contraseña de usuario de CyberX o de soporte técnico, consulte [Restablecimiento de contraseñas](how-to-create-and-manage-users.md#resetting-passwords).

### <a name="investigate-a-lack-of-traffic"></a>Investigar la falta de tráfico

Cuando el sensor reconoce que no hay tráfico en uno de los puertos configurados, aparece un indicador en la parte superior de la consola. Este indicador es visible para todos los usuarios.

:::image type="content" source="media/how-to-troubleshoot-the-sensor-and-on-premises-management-console/no-traffic-detected.png" alt-text="Captura de pantalla de la alerta de que no se ha detectado tráfico.":::
 
Cuando aparezca este mensaje, puede investigar dónde no hay tráfico. Asegúrese de que el cable del intervalo está conectado y de que no se ha producido ningún cambio en la arquitectura del intervalo.  

Para obtener información de soporte técnico y solución de problemas, póngase en contacto con el [Soporte técnico de Microsoft](https://support.serviceshub.microsoft.com/supportforbusiness/create?sapId=82c88f35-1b8e-f274-ec11-c6efdd6dd099).

### <a name="check-system-performance"></a>Compruebe el rendimiento del sistema. 

Cuando se implementa un nuevo sensor o, por ejemplo, el sensor funciona lentamente o no muestra ninguna alerta, puede comprobar el rendimiento del sistema.

Para comprobar el rendimiento del sistema:

1. En el panel de información, asegúrese de que `PPS > 0`.

   :::image type="content" source="media/how-to-troubleshoot-the-sensor-and-on-premises-management-console/dashboard-view-v2.png" alt-text="Captura de pantalla de un panel de información de ejemplo."::: 

1. Seleccione **Dispositivos** en el menú lateral.

1. En la ventana **Dispositivos**, asegúrese de que se detectan los dispositivos.

    :::image type="content" source="media/how-to-troubleshoot-the-sensor-and-on-premises-management-console/discovered-devices.png" alt-text="Asegúrese de que se detectan los dispositivos.":::

1. Seleccione **Minería de datos** en el menú lateral.

1. En la ventana **Minería de datos**, seleccione **TODOS** y genere un informe.

    :::image type="content" source="media/how-to-troubleshoot-the-sensor-and-on-premises-management-console/new-report-generated.png" alt-text="Genere un nuevo informe mediante minería de datos.":::

1. Asegúrese de que el informe contiene datos.

    :::image type="content" source="media/how-to-troubleshoot-the-sensor-and-on-premises-management-console/new-report-generated.png" alt-text="Asegúrese de que el informe contiene datos.":::

1. En el menú lateral, seleccione **Tendencias y estadísticas**.

1. En la ventana **Tendencias y estadísticas**, seleccione **Agregar widget**.

    :::image type="content" source="media/how-to-troubleshoot-the-sensor-and-on-premises-management-console/add-widget.png" alt-text="Seleccione un widget para agregarlo.":::

1. Agregue un widget y asegúrese de que muestre datos.

    :::image type="content" source="media/how-to-troubleshoot-the-sensor-and-on-premises-management-console/widget-data.png" alt-text="Asegúrese de que el widget muestre datos.":::

1. Seleccione **Alertas** en el menú lateral. Aparecerá la ventana **Alertas**.

1. Asegúrese de que se han creado las alertas.

    :::image type="content" source="media/how-to-troubleshoot-the-sensor-and-on-premises-management-console/alerts-created.png" alt-text="Asegúrese de que se han creado las alertas.":::


### <a name="investigate-a-lack-of-expected-alerts"></a>Investigar una falta de alertas previstas

Si la ventana **Alertas** no muestra una alerta esperada, compruebe lo siguiente:

- Compruebe si la misma alerta ya aparece en la ventana **Alertas** como reacción a una instancia de seguridad diferente. En caso afirmativo, y si esta alerta no se ha controlado todavía, la consola del sensor no muestra ninguna alerta nueva.

- Asegúrese de que no ha excluido esta alerta mediante el uso de las reglas de **Exclusión de alertas** en la consola de administración. 

### <a name="investigate-widgets-that-show-no-data"></a>Investigar widgets que no muestran datos

Cuando los widgets de la ventana **Tendencias y estadísticas** no muestran ningún dato, haga lo siguiente:

- [Compruebe el rendimiento del sistema](#check-system-performance).

- Asegúrese de que la configuración de hora y región esté correctamente configurada y no esté establecida en el futuro. 

### <a name="investigate-a-device-map-that-shows-only-broadcasting-devices"></a>Investigar un mapa de dispositivos que muestra solo dispositivos de difusión

Cuando los dispositivos mostrados en el mapa no aparecen conectados entre sí, es posible que se haya producido un error en la configuración del puerto de intervalo. Es decir, es posible que solo vea dispositivos de difusión y ningún tráfico de unidifusión.

:::image type="content" source="media/how-to-troubleshoot-the-sensor-and-on-premises-management-console/broadcasting-devices.png" alt-text="Vea los dispositivos de difusión.":::

En tal caso, debe comprobar que solo puede ver el tráfico de difusión. A continuación, pida al ingeniero de red que corrija la configuración del puerto de intervalo para que también pueda ver el tráfico de unidifusión.

Para comprobar que solo está viendo el tráfico de difusión:

- En la pantalla **Minería de datos**, cree un informe mediante la opción **Todos**. A continuación, vea si en el informe solo aparece el tráfico de difusión y multidifusión (y no hay tráfico de unidifusión).

O:

- Registre un PCAP directamente desde el conmutador, o bien conecte un portátil mediante Wireshark.

### <a name="connect-the-sensor-to-ntp"></a>Conectar el sensor a NTP

Puede configurar un sensor independiente y una consola de administración, con los sensores relacionados, para conectarse a NTP.

Para conectar un sensor independiente a NTP:

- [Póngase en contacto con el equipo de soporte técnico para obtener asistencia](https://support.microsoft.com/en-us/supportforbusiness/productselection?sapId=82c88f35-1b8e-f274-ec11-c6efdd6dd099).

Para conectar un sensor controlado por la consola de administración a NTP:

- La conexión a NTP está configurada en la consola de administración. Todos los sensores que controla la consola de administración obtienen la conexión NTP automáticamente.

### <a name="investigate-when-devices-arent-shown-on-the-map-or-you-have-multiple-internet-related-alerts"></a>Investigar cuando los dispositivos no se muestran en el mapa o si se reciben varias alertas relacionadas con Internet

A veces, los dispositivos ICS están configurados con direcciones IP externas. Estos dispositivos ICS no se muestran en el mapa. En lugar de los dispositivos, aparece una nube de Internet en el mapa. Las direcciones IP de estos dispositivos se incluyen en la imagen en la nube.

Otra indicación del mismo problema es cuando aparecen varias alertas relacionadas con Internet.

:::image type="content" source="media/how-to-troubleshoot-the-sensor-and-on-premises-management-console/alert-problems.png" alt-text="Varias alertas relacionadas con Internet.":::

Para corregir la configuración:

1. Haga clic con el botón derecho en el icono de la nube en el mapa del dispositivo y seleccione **Exportar direcciones IP**. Copie los intervalos públicos que son privados y agréguelos a la lista de subredes. Para obtener más información, consulte [Configurar subredes](how-to-control-what-traffic-is-monitored.md#configure-subnets).

1. Generar un nuevo informe de minería de datos para las conexiones a Internet.

1. En el informe de minería de datos, seleccione :::image type="icon" source="media/how-to-troubleshoot-the-sensor-and-on-premises-management-console/administrator-mode.png" border="false"::: para entrar en el modo de administrador y eliminar las direcciones IP de los dispositivos ICS.

### <a name="tweak-the-sensors-quality-of-service-qos"></a>Ajustar la Calidad de servicio (QoS) del sensor

Para guardar los recursos de red, puede limitar el ancho de banda de la interfaz que usa el sensor para los procedimientos cotidianos.

Para limitar el ancho de banda de la interfaz, use la herramienta CLI `cyberx-xsense-limit-interface` que se debe ejecutar con permisos sudo. La herramienta acepta los argumentos siguientes:

  - `* -i`: interfaces (ejemplo: eth0).

  - `* -l`: límite (ejemplo: 30 kbit/1 Mbit). Puede usar las siguientes unidades de ancho de banda: kbps, Mbps, kbit, Mbit o bps.

  - `* -c`: borrar (para borrar el límite de ancho de banda de la interfaz).

Para ajustar la calidad del servicio:

1. Inicie sesión en la CLI del sensor como un usuario de Defender para IoT y escriba `sudo cyberx-xsense-limit-interface-I eth0 -l value`.

   Por ejemplo: `sudo cyberx-xsense-limit-interface -i eth0 -l 30kbit`

   > [!NOTE]
   > Para un dispositivo físico, use la interfaz em1.

1. Para borrar la limitación de la interfaz, escriba `sudo cyberx-xsense-limit-interface -i eth0 -l 1mbps -c`.

## <a name="on-premises-management-console-troubleshooting-tools"></a>Herramientas de solución de problemas de la consola de administración local

### <a name="investigate-a-lack-of-expected-alerts"></a>Investigar una falta de alertas previstas

Si no se muestra una alerta esperada en la ventana **Alertas**, compruebe lo siguiente:

- Compruebe si la misma alerta ya aparece en la ventana **Alertas** como reacción a una instancia de seguridad diferente. En caso afirmativo, y si esta alerta no se ha controlado todavía, no se mostrará una nueva alerta.

- Asegúrese de que no ha excluido esta alerta mediante el uso de las reglas de **Exclusión de alertas** en la consola de administración local.  

### <a name="tweak-the-quality-of-service-qos"></a>Ajustar la Calidad del servicio (QoS)

Para ahorrar recursos de red, puede limitar el número de alertas enviadas a sistemas externos (como correos electrónicos o SIEM) en una operación de sincronización entre un dispositivo y la consola de administración local.

El valor predeterminado es 50. Esto significa que en una sesión de comunicación entre un dispositivo y la consola de administración local, no habrá más de 50 alertas para sistemas externos. 

Para limitar el número de alertas, use la propiedad `notifications.max_number_to_report` disponible en `/var/cyberx/properties/management.properties`. No es necesario reiniciar después de cambiar esta propiedad.

Para ajustar la calidad del servicio:

1. Inicie sesión como un usuario de Defender para IoT. 

1. Compruebe los valores predeterminados:

   ```bash
   grep \"notifications\" /var/cyberx/properties/management.properties
   ```

   Aparecerán los siguientes valor predeterminado:

   ```bash
   notifications.max_number_to_report=50
   notifications.max_time_to_report=10 (seconds)
   ```

1. Edite la configuración predeterminada:

   ```bash
   sudo nano /var/cyberx/properties/management.properties
   ```

1. Edite la configuración de las siguientes líneas:

   ```bash
   notifications.max_number_to_report=50
   notifications.max_time_to_report=10 (seconds)
   ```

1. Guarde los cambios. No es necesario reiniciar.

## <a name="export-information-for-troubleshooting"></a>Exportar información para solucionar problemas

Además de las herramientas para la supervisión y el análisis de la red, puede enviar información al equipo de soporte técnico para realizar una investigación más detallada. Al exportar registros, el sensor generará automáticamente una contraseña de un solo uso (OTP), única para los registros exportados, en un archivo de texto independiente. 

Para exportar registros:

1. En el panel izquierdo, seleccione **Configuración del sistema**.

1. Seleccione **Exportar registros**.

    :::image type="content" source="media/how-to-export-information-for-troubleshooting/export-a-log.png" alt-text="Exporte un registro al soporte del sistema.":::

1. En el cuadro **Nombre de archivo**, escriba el nombre de archivo que desea utilizar para la exportación del registro. La fecha predeterminada es la actual.

1. Para definir los datos que desea exportar, seleccione las categorías de datos:  

    | Exportar categoría | Descripción |
    |--|--|
    | **Registros del sistema operativo** | Seleccione esta opción para obtener información acerca del estado del sistema operativo. |
    | **Registros de instalación/actualización** | Seleccione esta opción para investigar los parámetros de configuración de la instalación y la actualización. |
    | **Salida de integridad del sistema** | Seleccione esta opción para comprobar el rendimiento del sistema. |
    | **Registros de disección** | Seleccione esta opción para permitir la inspección avanzada de la disección de protocolo. |
    | **Volcados de kernel del sistema operativo** | Seleccione esta opción para exportar el volcado de memoria del kernel. Un volcado de memoria del kernel contiene toda la memoria que está usando el kernel en el momento del problema que se produjo en este kernel. El tamaño del archivo de volcado es menor que el volcado de memoria completo. Normalmente, el archivo de volcado ocupa alrededor de un tercio respecto al tamaño de la memoria física del sistema. |
    | **Registros de reenvío** | Seleccione esta opción para investigar las reglas de reenvío. |
    | **Registros de SNMP** | Seleccione esta opción para recibir información de comprobación de la integridad de SNMP. |
    | **Registros de aplicación principal** | Seleccione esta opción para exportar los datos sobre la operación y la configuración de la aplicación principal. |
    | **Registros de comunicación con CM** | Seleccione esta opción si hay problemas continuos o interrupciones de conexión con la consola de administración. |
    | **Registros de aplicación web** | Seleccione esta opción para obtener información sobre todas las solicitudes enviadas desde la interfaz web de la aplicación. |
    | **Copia de seguridad del sistema** | Seleccione esta opción para exportar una copia de seguridad de todos los datos del sistema para investigar el estado exacto del sistema. |
    | **Estadísticas de disección** | Seleccione esta opción para permitir la inspección avanzada de las estadísticas de protocolo. |
    | **Registros de base de datos** | Seleccione esta opción para exportar los registros de la base de datos del sistema. La investigación de los registros del sistema ayuda a identificar los problemas del sistema. |
    | **Configuración** | Seleccione esta opción para exportar información acerca de todos los parámetros configurables para asegurarse de que todo se configuró correctamente. |

1. Para seleccionar todas las opciones, seleccione **Seleccionar todas** junto a **Elegir categorías**.

1. Seleccione **Exportar registros**.

Los registros exportados se agregan a la lista **Registros archivados**. Envíe la OTP al equipo de soporte técnico en un mensaje independiente y en medio de los registros exportados. El equipo de soporte técnico podrá extraer los registros exportados solo mediante la OTP única que se usa para cifrar los registros.

La lista de registros archivados puede contener hasta cinco elementos. Si el número de elementos de la lista va más allá de ese número, se eliminará el elemento más antiguo.

## <a name="see-also"></a>Consulte también

- [Visualización de alertas](how-to-view-alerts.md)

- [Configuración de la supervisión de MIB del SNMP](how-to-set-up-snmp-mib-monitoring.md)

- [Descripción de los eventos de desconexión de sensores](how-to-manage-sensors-from-the-on-premises-management-console.md#understand-sensor-disconnection-events)
