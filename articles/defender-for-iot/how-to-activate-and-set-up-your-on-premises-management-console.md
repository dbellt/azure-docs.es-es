---
title: Activación y configuración de la consola de administración local
description: La activación de la consola de administración garantiza que los sensores se registren en Azure y envían información a la consola de administración local, y que la consola de administración local lleva a cabo tareas de administración en sensores conectados.
ms.date: 4/6/2021
ms.topic: how-to
ms.openlocfilehash: db0d2a84feeb5bf52932842badda8c126994c05d
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106492161"
---
# <a name="activate-and-set-up-your-on-premises-management-console"></a>Activación y configuración de la consola de administración local 

La activación y configuración de la consola de administración local garantiza lo siguiente:

- Los dispositivos de red que se están supervisando a través de sensores conectados se registran con una cuenta de Azure.

- Los sensores envían información a la consola de administración local.

- La consola de administración local lleva a cabo tareas de administración en sensores conectados.

- Ha instalado un certificado SSL.

## <a name="sign-in-for-the-first-time"></a>Iniciar sesión por primera vez

Para iniciar sesión en la consola de administración:

1. Vaya a la dirección IP y la contraseña que ha recibido para la consola de administración local durante la instalación del sistema.
 
1. Introduzca el nombre de usuario y la contraseña que ha recibido para la consola de administración local durante la instalación del sistema. 


Si olvidó la contraseña, seleccione la opción **Recuperar contraseña** y consulte recuperación de [contraseña](how-to-manage-the-on-premises-management-console.md#password-recovery) para obtener instrucciones sobre cómo recuperar la contraseña.

## <a name="activate-the-on-premises-management-console"></a>Activación de la consola de administración local

Una vez iniciada la sesión por primera vez, tendrá que activar la consola de administración local obteniendo y cargando un archivo de activación. 

Para activar la consola de administración local, haga lo siguiente:

1. Inicie sesión en la consola de administración local.

1. En la notificación de alerta de la parte superior de la pantalla, seleccione el vínculo **Realizar acción**.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/take-action.png" alt-text="Seleccione el vínculo Realizar acción de la alerta en la parte superior de la pantalla.":::

1. En la pantalla emergente de activación, seleccione el vínculo **Azure Portal**.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/azure-portal.png" alt-text="Seleccione el vínculo Azure Portal del mensaje emergente.":::
 
1. Seleccione una suscripción para asociar a ella la consola de administración local y, a continuación, seleccione el botón **Download on-premises management console activation file** (Descargar el archivo de activación de la consola de administración local). Se descarga el archivo de activación.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/cloud_download_opm_activation_file.png" alt-text="Descargue el archivo de activación.":::

   Si aún no ha incorporado una suscripción, [incorpore una](how-to-manage-subscriptions.md#onboard-a-subscription).

1. Vuelva a la pantalla emergente de **Activación** y seleccione **Elegir archivo**.

1. Seleccione el archivo que ha descargado.

Después de la activación inicial, el número de dispositivos supervisados puede superar el número de dispositivos confirmados definidos durante la incorporación. Esto ocurre si conecta más sensores a la consola de administración. Si hay una discrepancia entre el número de dispositivos supervisados y el número de dispositivos confirmados, aparecerá una advertencia en la consola de administración. Si esto ocurre, cargue un nuevo archivo de activación.

## <a name="set-up-a-certificate"></a>Configurar un certificado

Después de instalar la consola de administración, se genera un certificado autofirmado local. Este certificado se usa para acceder a la consola. Cuando un administrador inicia sesión en la consola de administración por primera vez, se le solicita que incorpore un certificado SSL/TLS. 

Hay dos niveles de seguridad disponibles:

- Cumple con los requisitos de cifrado y certificado específicos solicitados de la organización, cargando el certificado firmado por la entidad de certificación.
- Permite la validación entre la consola de administración y los sensores conectados. La validación se evalúa con una lista de revocación de certificados y la fecha de expiración del certificado. *Si se produce un error en la validación, la comunicación entre la consola de administración y el sensor se detiene y aparece un error de validación en la consola.* Esta opción está habilitada de forma predeterminada después de la instalación.  

La consola admite los siguientes tipos de certificados:

- Infraestructura de clave empresarial y privada (PKI privada)

- Infraestructura de clave pública (PKI pública)

- Generado localmente en el dispositivo (autofirmado localmente) 

  > [!IMPORTANT]
  > No se recomienda usar certificados autofirmados. Este certificado no es seguro y debe usarse solo para entornos de prueba. No se puede validar el propietario del certificado y no se puede mantener la seguridad del sistema. Esta opción nunca debe usarse para redes de producción.

Para cargar un certificado:

1. Defina un nombre de certificado cuando se le pida después de iniciar sesión.

1. Cargue los archivos de clave y CRT.

1. Escriba una frase de contraseña y cargue un archivo PEM, si es necesario.

Es posible que deba actualizar la pantalla después de cargar el certificado firmado por la entidad de certificación.

Para desactivar la validación entre la consola de administración y los sensores conectados:

1. Seleccione **Next** (Siguiente).

1. Desactive el botón de alternancia **Habilitar la validación en todo el sistema**.

Para obtener información sobre cómo cargar un certificado nuevo, archivos de certificado compatibles y elementos relacionados, consulte [Administración de la consola de administración local](how-to-manage-the-on-premises-management-console.md).

## <a name="connect-sensors-to-the-on-premises-management-console"></a>Conectar los sensores a una consola de administración local

Debe asegurarse de que los sensores envían información a la consola de administración local y de que la consola de administración local puede realizar copias de seguridad, administrar alertas y realizar otras actividades en los sensores. Para ello, use los procedimientos siguientes para comprobar que realiza una conexión inicial entre los sensores y la consola de administración local.

Hay dos opciones disponibles para conectar sensores de Azure Defender para IoT a la consola de administración local:

- Conexión desde la consola del sensor

- Conexión mediante tunelización

Después de conectarse, debe configurar un sitio con estos sensores.

### <a name="connect-sensors-to-the-on-premises-management-console-from-the-sensor-console"></a>Conexión de sensores a la consola de administración local desde la consola del sensor

Puede conectar sensores a la consola de administración local desde la consola del sensor:

1. En la consola de administración local, seleccione **Configuración del sistema**.

1. Copie la **Cadena de conexión**.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/connection-string.png" alt-text="Copie la cadena de conexión del sensor.":::

1. En el sensor, vaya a **Configuración del sistema** y seleccione **Connection to Management Console** (Conexión a la consola de administración). :::image type="icon" source="media/how-to-manage-sensors-from-the-on-premises-management-console/connection-to-management-console.png" border="false":::

1. Pegue la cadena de conexión que copió de la consola de administración local en el campo **Cadena de conexión**.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/paste-connection-string.png" alt-text="Pegue la cadena de conexión que copió en el campo Cadena de conexión.":::

1. Seleccione **Conectar**.

### <a name="connect-sensors-by-using-tunneling"></a>Conexión de sensores mediante tunelización

Habilitar una conexión de túnel segura entre los sensores de la organización y la consola de administración local. Este programa de instalación evita la interacción con el firewall de la organización y, como resultado, reduce la superficie expuesta a ataques.

El uso de la tunelización le permite conectarse a la consola de administración local desde su dirección IP y un puerto único (es decir, 9000) a cualquier sensor.

Para configurar la tunelización en la consola de administración local:

- Abra la consola de administración local y ejecute los siguientes comandos:

  ```bash
  cyberx-management-tunnel-enable
  service apache2 reload
  sudo cyberx-management-tunnel-add-xsense --xsenseuid <sensorIPAddress> --xsenseport 9000
  service apache2 reload
  ```

Para configurar la tunelización en el sensor:

1. Abra el puerto TCP 9000 en el sensor (network.properties) manualmente. Si el puerto no está abierto, el sensor rechazará la conexión de la consola de administración local.

2. Inicie sesión con cada sensor y ejecute los comandos siguientes:

   ```bash
   sudo cyberx-xsense-management-connect -ip <centralmanagerIPAddress>
   sudo cyberx-xsense-management-tunnel
   sudo vi /var/cyberx/properties/network.properties
   opened_tcp_incoming_ports=22,80,443,102,9000
   sudo cyberx-xsense-network-validation
   sudo /etc/network/if-up.d/iptables-recover
   sudo iptables -nvL
   ```

## <a name="set-up-a-site"></a>Configurar un sitio

El mapa de empresa predeterminado proporciona una vista general de los dispositivos en función de varios niveles de ubicaciones geográficas.

La vista de los dispositivos podría ser necesaria si la estructura de la organización y los permisos de usuario son complejos. En estos casos, la configuración del sitio puede estar determinada por una estructura organizativa global, además de la estructura de la zona o del sitio estándar.

Para admitir este entorno, debe crear una topología empresarial global basada en las unidades de negocio, las regiones, los sitios y las zonas de su organización. También debe definir los permisos de acceso de usuario en torno a estas entidades mediante el uso de grupos de acceso.

Los grupos de acceso permiten un mejor control sobre el lugar en el que los usuarios administran y analizan los dispositivos en la plataforma Defender para IoT.

### <a name="how-it-works"></a>Funcionamiento

Puede definir una unidad de negocio y una región para cada sitio de la organización. Después, puede agregar zonas, que son entidades lógicas que existen en la red. 

Debe asignar al menos un sensor para cada zona. El modelo de cinco niveles proporciona la flexibilidad y la granularidad necesarias para ofrecer el sistema de protección que refleja la estructura de la organización.

:::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/diagram-of-sensor-showing-relationships.png" alt-text="Diagrama que muestra los sensores y la relación regional.":::

Con la vista empresarial, puede editar los sitios directamente. Cuando seleccione un sitio en la vista de empresa, el número de alertas abiertas aparecerá junto a cada zona.

:::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/console-map-with-data-overlay-v2.png" alt-text="Captura de pantalla de un mapa de la consola de administración local con la superposición de datos de Berlín.":::

Para configurar un sitio:

1. Agregue nuevas unidades de negocio para reflejar la estructura lógica de su organización.

   1. En la vista Enterprise, seleccione **Todos los sitios** > **Administrar unidades de negocio**.

      :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/manage-business-unit.png" alt-text="Seleccione Administrar unidades de negocio en el menú desplegable Todos los sitios de la pantalla de vista empresarial.":::

   1. Escriba el nombre de la nueva unidad de negocio y seleccione **AGREGAR**.

1. Agregue nuevas regiones para reflejar las regiones de su organización.

   1. En la vista empresarial, seleccione **Todas las regiones** > **Administrar regiones**.

   :::image type="content" source="media/how-to-manage-sensors-from-the-on-premises-management-console/manage-regions.png" alt-text="Seleccione Todas las regiones y, a continuación, Administrar regiones para administrar las regiones de su empresa.":::

   1. Escriba el nuevo nombre de la región y seleccione **AGREGAR**.

1. Agregue un sitio.

   1. En la vista Enterprise, seleccione :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/new-site-icon.png" border="false"::: en la barra superior. El cursor aparece como el signo más ( **+** ).

   1. Coloque el **+** en la ubicación del nuevo sitio y selecciónelo. Se abrirá el cuadro de diálogo **Crear nuevo sitio**.

      Captura de la vista :::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/create-new-site-screen.png" alt-text="Crear nuevo sitio.":::

   1. Defina el nombre y la dirección física del nuevo sitio y seleccione **GUARDAR**. El nuevo sitio aparece en el mapa del sitio.

4. [Agregue zonas a un sitio](#create-enterprise-zones).

5. [Conecte los sensores](how-to-manage-individual-sensors.md#connect-a-sensor-to-the-management-console).

6. [Asigne un sensor a las zonas de sitios](#assign-sensors-to-zones).

### <a name="delete-a-site"></a>Eliminación de un sitio

Si ya no necesita un sitio, puede eliminarlo de la consola de administración local.

Para eliminar un sitio:

1. En la ventana **Administración de sitio**, seleccione :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/expand-view-icon.png" border="false"::: en la barra que contiene el nombre del sitio y, luego, seleccione **Eliminar sitio**. Aparecerá el cuadro de confirmación, en el que se comprobará que desea eliminar el sitio.

2. En la ventana de confirmación, seleccione **CONFIRMAR**.

## <a name="create-enterprise-zones"></a>Crear zonas de empresa

Las zonas son entidades lógicas que le permiten dividir los dispositivos de un sitio en grupos según varias características. Por ejemplo, puede crear grupos para líneas de producción, subestaciones, áreas de sitio o tipos de dispositivos. Puede definir zonas en función de las características que sean adecuadas para su organización.

Las zonas se configuran como parte del proceso de configuración del sitio.

:::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/site-management-zones-screen-v2.png" alt-text="Captura de pantalla de la vista Zonas de administración de sitio.":::

En la siguiente tabla se describen los parámetros en la ventana **Administración de sitio**.

| Parámetro | Descripción |
|--|--|
| Nombre | El nombre del sensor. Este nombre solo se puede cambiar desde el sensor. Para más información, consulte la guía de usuario de Defender para IoT. |
| IP | La dirección IP del sensor. |
| Versión | La versión del sensor. |
| Conectividad | El estado de conectividad del servidor. El estado puede ser **Conectado** o **Desconectado**. |
| Última actualización | La fecha y hora de la última actualización. |
| Progreso de actualización | La barra de progreso indica el estado del proceso de actualización, como se indica a continuación:<br />- Cargando el paquete<br />- Preparando la instalación<br />- Deteniendo procesos<br />- Realizando una copia de seguridad de los datos<br />- Tomando instantánea<br />- Actualizando la configuración<br />- Actualizando dependencias<br />- Actualizando bibliotecas<br />- Aplicando revisiones a las bases de datos<br />- Iniciando procesos<br />- Comprobando la integridad del sistema<br />- Validación correcta<br />- Correcto<br />- Error<br />- Se ha iniciado la actualización<br />- Iniciando la instalaciónogress bar shows the status of the upgrade process, as follows:<br />- Uploading package<br />- Preparing to install<br />- Stopping processes<br />- Backing up data<br />- Taking snapshot<br />- Updating configuration<br />- Updating dependencies<br />- Updating libraries<br />- Patching databases<br />- Starting processes<br />- Validating system sanity<br />- Validation succeeded<br />- Success<br />- Failure<br />- Upgrade started<br />- Starting installation<br /></br >Para obtener más información acerca de la actualización, consulte el [Soporte técnico de Microsoft](https://support.microsoft.com/) para obtener ayuda. |
| Dispositivos | El número de dispositivos OT que supervisa el sensor. |
| Alertas | El número de alertas en el sensor. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/assign-icon.png" border="false"::: | Permite asignar un sensor a las zonas. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/delete-icon.png" border="false":::| Permite eliminar un sensor desconectado del sitio. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/sensor-icon.png" border="false"::: | Indica cuántos sensores están conectados actualmente a la zona. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/ot-assets-icon.png" border="false"::: | Indica cuántos sensores OT están conectados actualmente a la zona. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/number-of-alerts-icon.png" border="false"::: | Indica el número de alertas enviadas por los sensores asignados a la zona. |
| :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/unassign-sensor-icon.png" border="false"::: | Anula la asignación de sensores de las zonas. |

Para agregar una zona a un sitio:

1. En la ventana **Administración de sitio**, seleccione :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/expand-view-icon.png" border="false"::: en la barra que contiene el nombre del sitio y, luego, seleccione **Añadir zona**. Aparecerá el cuadro de diálogo **Crear nueva zona**.

    :::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/create-new-zone-screen.png" alt-text="Captura de pantalla de la vista Crear nueva zona.":::

1. Escriba el nombre de la zona.

1. Escriba una descripción de la nueva zona que indique claramente las características que se han usado para dividir el sitio en zonas.

1. Seleccione **SAVE** (GUARDAR). La nueva zona aparece en la ventana **Administración de sitio** en el sitio al que pertenece esta zona.

Para editar una zona:

1. En la ventana **Administración de sitio**, seleccione :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/expand-view-icon.png" border="false"::: en la barra que contiene el nombre de la zona y, luego, seleccione **Editar zona**. Aparece el cuadro de diálogo **Editar zona**.

   :::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/zone-edit-screen.png" alt-text="Captura de pantalla que muestra el cuadro de diálogo Editar zona.":::

1. Edite los parámetros de la zona y seleccione **GUARDAR**.

Para eliminar una zona:

1. En la ventana **Administración de sitio**, seleccione :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/expand-view-icon.png" border="false"::: en la barra que contiene el nombre de la zona y, luego, seleccione **Eliminar zona**.

1. En la ventana de confirmación, seleccione **SÍ**.

Para filtrar según el estado de conectividad:

- En la esquina superior izquierda, seleccione :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/down-pointing-icon.png" border="false"::: junto a **Conectividad** y, luego, seleccione una de las siguientes opciones:

  - **Todos**: Muestra todos los sensores que informan a esta consola de administración local.

  - **Conectado**: Solo muestra los sensores conectados.

  - **Desconectado**: Solo muestra los sensores desconectados.

Para filtrar según el estado de actualización:

- En la esquina superior izquierda, seleccione :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/down-pointing-icon.png" border="false"::: junto a **Estado de actualización** y, luego, seleccione una de las siguientes opciones:

  - **Todos**: Muestra todos los sensores que informan a esta consola de administración local.

  - **Válido**: Muestra los sensores con un estado de actualización válido.

  - **En curso**: Muestra los sensores que están en proceso de actualización.

  - **Error**: Muestra los sensores en los que se ha producido un error en el proceso de actualización.

## <a name="assign-sensors-to-zones"></a>Asignar sensores a las zonas

Para cada zona, debe asignar sensores que realicen análisis y alertas de tráfico local. Solo puede asignar los sensores que están conectados a la consola de administración local.

Para asignar un sensor:

1. Seleccione **Administración de sitios**. Los sensores sin asignar aparecen en la esquina superior izquierda del cuadro de diálogo.

   :::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/unassigned-sensors-view.png" alt-text="Captura de pantalla de la vista Sensores sin asignar.":::

1. Consulte el estado de **Conectividad** para comprobar que esté conectado. Si no es así, consulte [Conectar los sensores a una consola de administración local](#connect-sensors-to-the-on-premises-management-console) para obtener más información acerca de la conexión. 

1. Seleccione :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/assign-icon.png" border="false"::: para el sensor que desea asignar.

1. En el cuadro de diálogo **Asignar sensor**, seleccione la unidad de negocio, la región, el sitio y la zona que desea asignar.

   :::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/assign-sensor-screen.png" alt-text="Captura de pantalla de la vista Asignar sensor.":::

1. Seleccione **ASIGNAR**.

Para anular la asignación y eliminar un sensor:

1. Desconecte el sensor de la consola de administración local. Consulte [Conectar los sensores a una consola de administración local](#connect-sensors-to-the-on-premises-management-console) para obtener más información.

1. En la ventana **Administración de sitio**, seleccione el sensor y seleccione :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/unassign-sensor-icon.png" border="false":::. El sensor aparece en la lista de sensores sin asignar transcurridos unos instantes.

1. Para eliminar el sensor sin asignar del sitio, seleccione el sensor en la lista de sensores sin asignar y seleccione :::image type="icon" source="media/how-to-activate-and-set-up-your-on-premises-management-console/delete-icon.png" border="false":::.

## <a name="see-also"></a>Consulte también

[Solución de problemas del sensor y de la consola de administración local](how-to-troubleshoot-the-sensor-and-on-premises-management-console.md)
