---
title: Activación y configuración del sensor
description: En este artículo se describe cómo iniciar sesión en una consola de sensor y cómo activarla.
ms.date: 04/29/2021
ms.topic: how-to
ms.openlocfilehash: 5e1e710f4f4eae844bbe18887bd3c3a1ab9f1da3
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109656963"
---
# <a name="activate-and-set-up-your-sensor"></a>Activación y configuración del sensor

En este artículo se describe cómo activar un sensor y realizar su configuración inicial.

Los usuarios administradores llevan a cabo la activación al iniciar sesión por primera vez, así como cuando se requiera la administración de la activación. El programa de configuración garantiza que el sensor está configurado para detectar y enviar alertas de manera óptima.

Los analistas de seguridad y los usuarios de solo lectura no pueden activar un sensor ni generar una contraseña nueva.

## <a name="sign-in-and-activation-for-administrator-users"></a>Inicio de sesión y activación para usuarios administradores

Los administradores que inician sesión por primera vez deben comprobar que cuentan con acceso a los archivos de recuperación de contraseña y activación que se descargaron durante la incorporación del sensor. En caso contrario, necesitarán permisos de administrador de seguridad, de colaborador de la suscripción o de propietario de la suscripción de Azure para generar dichos archivos en el portal de Azure Defender para IoT.

### <a name="first-time-sign-in-and-activation-checklist"></a>Lista de comprobación del primer inicio de sesión y activación

Antes de iniciar sesión en la consola del sensor, los usuarios administradores deben tener acceso a lo siguiente:

- La dirección IP del sensor que se definió durante la instalación.

- Credenciales de inicio de sesión de usuario para el sensor. Si descargó una imagen ISO para el sensor, use las credenciales predeterminadas que recibió durante la instalación. Se recomienda crear un nuevo usuario *administrador* después de la activación.

- Una contraseña inicial. Si compró un sensor preconfigurado en Arrow, debe generar una contraseña al iniciar sesión por primera vez.

- El archivo de activación asociado a este sensor. El archivo se generó y descargó durante la incorporación del sensor en el portal de Defender para IoT.

- Un certificado firmado por una entidad de certificación SSL/TLS que su empresa requiera.

### <a name="about-activation-files"></a>Acerca de los archivos de activación

El sensor se incorporó a Azure Defender para IoT en un modo de administración concreto:

| Tipo de modo | Descripción |
|--|--|
| **Modo conectado a la nube** | La información que detecta el sensor se muestra en la consola del sensor. Además, se proporciona información de alertas mediante el centro de IoT, que puede compartirse con otros servicios de Azure, como Azure Sentinel. También puede habilitar las actualizaciones automáticas de inteligencia sobre amenazas. |
| **Modo conectado de manera local** | La información que detecta el sensor se muestra en la consola del sensor. La información de detección también se comparte con la consola de administración local si el sensor está conectado a ella. |

Se generó y descargó un archivo de activación conectado de manera local o conectado a la nube para este sensor durante la incorporación. El archivo de activación contiene instrucciones para el modo de administración del sensor. *Se debe cargar un archivo de activación único para cada sensor que implemente.*  La primera vez que inicie sesión, deberá cargar el archivo de activación correspondiente a este sensor.

:::image type="content" source="media/how-to-activate-and-set-up-your-sensor/azure-defender-for-iot-activation-file-download-button.png" alt-text="Incorporación de un sensor en el portal de Azure Defender para IoT.":::

### <a name="about-certificates"></a>Acerca de los certificados

Tras la instalación del sensor, se genera un certificado autofirmado local que se usa para acceder a la consola del sensor. Cuando un administrador inicia sesión en la consola por primera vez, se le solicita que incorpore un certificado SSL/TLS.

Hay dos niveles de seguridad disponibles:

- Cumplir con los requisitos de cifrado y certificado específicos solicitados por la organización al cargar el certificado firmado por la entidad de certificación.
- Permitir la validación entre la consola de administración y los sensores conectados. La validación se evalúa con una lista de revocación de certificados y la fecha de expiración del certificado. *Si se produce un error en la validación, la comunicación entre la consola de administración y el sensor se detiene y aparece un error de validación en la consola.* Esta opción está habilitada de manera predeterminada después de la instalación.  

La consola admite los siguientes tipos de certificado:

- Infraestructura de clave empresarial y privada (PKI privada)

- Infraestructura de clave pública (PKI pública)

- Generado localmente en el dispositivo (autofirmado localmente) 

  > [!IMPORTANT]
  > No se recomienda usar el certificados autofirmado predeterminado. Este certificado no es seguro y debe usarse solo para entornos de prueba. No se puede validar el propietario del certificado y no se puede mantener la seguridad del sistema. Esta opción nunca debe usarse para redes de producción.

### <a name="sign-in-and-activate-the-sensor"></a>Inicio de sesión y activación del sensor

**Para iniciar sesión y realizar la activación, haga lo siguiente:**

1. Vaya a la consola del sensor desde el explorador web mediante la dirección IP definida durante la instalación. Se abre el cuadro de diálogo de inicio de sesión.

    :::image type="content" source="media/how-to-activate-and-set-up-your-sensor/azure-defender-for-iot-sensor-log-in-screen.png" alt-text="Sensor de Azure Defender para IoT.":::

1. Escriba las credenciales que se definieron al instalar el sensor o seleccione la opción **Password recovery** (Recuperación de contraseña). Si compró un sensor preconfigurado en Arrow, genere primero una contraseña. Para obtener más información sobre la recuperación de contraseñas, consulte [Investigar el error de contraseña en el inicio de sesión inicial](how-to-troubleshoot-the-sensor-and-on-premises-management-console.md#investigate-password-failure-at-initial-sign-in).

1. Después de iniciar sesión, se abrirá el cuadro de diálogo **Activar**. Seleccione **Cargar** y diríjase al archivo de activación que descargó durante la incorporación del sensor.

   :::image type="content" source="media/how-to-activate-and-set-up-your-sensor/activation-upload-screen-with-upload-button.png" alt-text="Seleccione Cargar y diríjase al archivo de activación.":::

1. Seleccione el vínculo **Sensor Network Configuration** (configuración de red del sensor) si quiere cambiar la configuración de red del sensor antes de la activación. Consulte [Actualización de la configuración de red del sensor antes de la activación](#update-sensor-network-configuration-before-activation).

1. Acepte los términos y condiciones.

1. Seleccione **Activar**. Se abre el cuadro de diálogo de certificado SSL/TLS.

1. Defina un nombre de certificado.
1. Cargue los archivos de clave y CRT.
1. Escriba una frase de contraseña y cargue un archivo PEM, si es necesario.
1. Seleccione **Next** (Siguiente). Se abre la pantalla de validación. De manera predeterminada, la validación entre la consola de administración y los sensores conectados está habilitada.
1. Desactive el botón de alternancia **Habilitar la validación en todo el sistema** para deshabilitar la validación. Se recomienda que esté habilitada.
1. Seleccione **Guardar**.  

Es posible que tenga que actualizar la pantalla después de cargar el certificado firmado por la entidad de certificación.

Para obtener información sobre cómo cargar un nuevo certificado, los parámetros de certificado admitidos y cómo trabajar con los comandos de certificados de la CLI, consulte [Administración de sensores individuales](how-to-manage-individual-sensors.md).

#### <a name="update-sensor-network-configuration-before-activation"></a>Actualización de la configuración de red del sensor antes de la activación  

Los parámetros de configuración de red del sensor se definieron durante la instalación del software o cuando se compró un sensor configurado previamente. Se definieron los siguientes parámetros:

- Dirección IP
- DNS
- Puerta de enlace predeterminada
- Máscara de subred
- Nombre de host

Puede que quiere actualizar esta información antes de activar el sensor. Por ejemplo, quizá deba cambiar los parámetros preconfigurados que ha definido Arrow. También puede definir la configuración del proxy antes de activar el sensor.

**Para actualizar los parámetros de configuración de la red del sensor, haga lo siguiente:**

1. Seleccione el vínculo **Sensor Network Configuration** (Configuración de red del sensor) en el cuadro de diálogo **Activación**.

   :::image type="content" source="media/how-to-activate-and-set-up-your-sensor/editable-network-configuration-screen-v2.png" alt-text="Configuración de red del sensor.":::

2. Se muestran los parámetros definidos durante la instalación. También está disponible la opción para definir el proxy. Actualice todos los valores necesarios y seleccione **Guardar**.

### <a name="activate-an-expired-license-versions-under-100"></a>Activación de una licencia expirada (versiones inferiores a la 10.0)

Para los usuarios con versiones anteriores a la 10.0, la licencia puede expirar y se mostrará la siguiente alerta. 

:::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/activation-popup.png" alt-text="Cuando expire la licencia, deberá actualizarla a través del archivo de activación.":::

**Para activar la licencia:**

1. Abra un caso para el equipo de [soporte técnico](https://ms.portal.azure.com/?passwordRecovery=true&Microsoft_Azure_IoT_Defender=canary#create/Microsoft.Support).

1. Proporcione al equipo de soporte técnico el número del identificador de activación.

1. El equipo de soporte técnico le suministrará una nueva información de licencia en forma de una cadena de letras.

1. Lea los términos y condiciones, y seleccione la casilla de aprobación.

1. Pegue la cadena en el espacio proporcionado.

    :::image type="content" source="media/how-to-activate-and-set-up-your-on-premises-management-console/add-license.png" alt-text="Pegue la cadena en el campo proporcionado.":::

1. Seleccione **Activar**.

### <a name="subsequent-sign-ins"></a>Inicios de sesión posteriores

Después de la activación del primer uso, se abre la consola del sensor de Azure Defender para IoT después de iniciar sesión sin necesidad de un archivo de activación. Solo necesitará las credenciales de inicio de sesión.

Después de iniciar sesión, se abre la consola de Azure Defender para IoT.

:::image type="content" source="media/how-to-activate-and-set-up-your-sensor/azure-defender-for-iot-log-in-screen-dashboard-v2.png" alt-text="Consola de Azure Defender para IoT.":::

## <a name="initial-setup-and-learning-for-administrators"></a>Configuración y aprendizaje iniciales (para administradores)

Después del primer inicio de sesión, el sensor de Azure Defender para IoT comienza a supervisar la red automáticamente. Los dispositivos de red aparecerán en las secciones Mapa de dispositivos e Inventario de dispositivos. Azure Defender para IoT comenzará a detectar y enviará alertas sobre todos los incidentes operativos y de seguridad que se produzcan en la red. Después, podrá crear informes y consultas en función de la información detectada.

Inicialmente, esta actividad se realiza en el modo de aprendizaje, en el que se indica al sensor que aprenda la actividad habitual de la red. Por ejemplo, el sensor aprende sobre los dispositivos detectados en la red, los protocolos detectados en la red y las transferencias de archivos que se producen entre dispositivos específicos. Esta actividad se convierte en la actividad de base de referencia de la red.

### <a name="review-and-update-basic-system-settings"></a>Revisión y actualización de la configuración básica del sistema

Revise la configuración del sistema del sensor para asegurarse de que el sensor esté configurado para detectar y enviar alertas de manera óptima.

Defina la configuración del sistema del sensor. Por ejemplo:

- Defina ICS (o IoT) y las subredes segregadas.

- Defina los alias de puerto para los protocolos específicos de un sitio.

- Defina las redes VLAN y los nombres que se usarán.

- Si se está usando DHCP, defina rangos DHCP legítimos.

- Defina la integración con Active Directory y servidores de correo de la forma mas apropiada.

### <a name="disable-learning-mode"></a>Deshabilitación del modo de aprendizaje

Después de ajustar la configuración del sistema, puede permitir que el sensor de Azure Defender para IoT se ejecute en modo de aprendizaje hasta que crea que las detecciones del sistema reflejan con precisión la actividad de la red.

El modo de aprendizaje debe ejecutarse durante aproximadamente dos a seis semanas, según el tamaño de la red y su complejidad. Después de deshabilitar el modo de aprendizaje, toda la actividad que difiera de la actividad de base de referencia desencadenará una alerta.

**Para deshabilitar el modo de aprendizaje, haga lo siguiente:**

- Seleccione **Configuración del sistema** y desactive la opción **Aprendizaje**.

## <a name="first-time-sign-in-for-security-analysts-and-read-only-users"></a>Inicio de sesión por primera vez para analistas de seguridad y usuarios de solo lectura

Antes de iniciar sesión, compruebe que dispone de lo siguiente:

- La dirección IP del sensor.
- Credenciales de inicio de sesión proporcionadas por el administrador.

## <a name="console-tools-overview"></a>Herramientas de la consola: Información general

Se accede a las herramientas de la consola desde el menú lateral.

**Navegación** 

| Periodo | Icono | Descripción |
| -----------|--|--|
| Panel | :::image type="icon" source="media/concept-sensor-console-overview/dashboard-icon-azure.png" border="false"::: | Vea una instantánea intuitiva del estado de seguridad de la red. |
| Mapa de dispositivos | :::image type="icon" source="media/concept-sensor-console-overview/asset-map-icon-azure.png" border="false"::: | Vea los dispositivos de red, las conexiones de dispositivos y las propiedades de los dispositivos en un mapa. Hay disponibles varias opciones de acercamiento, resaltado y filtro para mostrar la red. |
| Inventario de dispositivos | :::image type="icon" source="media/concept-sensor-console-overview/asset-inventory-icon-azure.png" border="false":::  | El inventario de dispositivos muestra una lista de atributos de dispositivo que detecta el sensor. Hay opciones disponibles para: <br /> - Ordenar o filtrar la información en función de los campos de la tabla y mostrar la información filtrada. <br /> - Exportar información a un archivo CSV. <br /> - Importar los detalles del registro de Windows.|
| Alertas | :::image type="icon" source="media/concept-sensor-console-overview/alerts-icon-azure.png" border="false"::: | Muestra alertas cuando se producen infracciones de las directivas, se producen desviaciones del comportamiento de base de referencia o se detecta cualquier tipo de actividad sospechosa en la red. |
| Informes | :::image type="icon" source="media/concept-sensor-console-overview/reports-icon-azure.png" border="false"::: | Vea informes basados en consultas de minería de datos. |

**Análisis**

| Periodo| Icono | Descripción |
|---|---|---|
| Escala de tiempo de eventos | :::image type="icon" source="media/concept-sensor-console-overview/event-timeline-icon-azure.png" border="false"::: | Vea una escala de tiempo con información sobre las alertas, eventos de red (informativos) y operaciones de usuario, como los inicios de sesión de usuario y las eliminaciones de usuarios.|

**Navegación**

| Periodo | Icono | Descripción |
|---|---|---|
| Minería de datos | :::image type="icon" source="media/concept-sensor-console-overview/data-mining-icon-azure.png" border="false"::: | Genere información exhaustiva y detallada sobres los dispositivos de la red en varias capas. |
| Investigación | :::image type="icon" source="media/concept-sensor-console-overview/trends-and-statistics-icon-azure.jpg" border="false"::: | Vea tendencias y estadísticas en una amplia gama de widgets. |
| Evaluación de riesgos | :::image type="icon" source="media/concept-sensor-console-overview/vulnerabilities-icon-azure.png" border="false"::: | Muestre la ventana **Vulnerabilidades**. |

**Administrador**

| Periodo | Icono | Descripción |
|---|---|---|
| Usuarios | :::image type="icon" source="media/concept-sensor-console-overview/users-icon-azure.png" border="false"::: | Defina usuarios y roles con varios niveles de acceso. |
| Reenvío | :::image type="icon" source="media/concept-sensor-console-overview/forwarding-icon-azure.png" border="false"::: | Reenvíe información de alertas a los asociados y a los orígenes internos (por ejemplo, Azure Sentinel) que se integran con Defender para IoT, a las direcciones de correo electrónico y a los servidores de webhook, entre otros. <br /> Consulte [Reenvío de la información de las alertas](how-to-forward-alert-information-to-partners.md) para más información. |
| Configuración del sistema | :::image type="icon" source="media/concept-sensor-console-overview/system-settings-icon-azure.png" border="false"::: | Configure las opciones del sistema. Por ejemplo, defina la configuración de DHCP, proporcione los detalles del servidor de correo o cree un alias de puerto. |
| Importar configuración | :::image type="icon" source="media/concept-sensor-console-overview/import-settings-icon-azure.png" border="false"::: | Muestre la ventana **Importar configuración**. Puede realizar cambios manuales en la información de un dispositivo.<br /> Consulte [Importación de información del dispositivo](how-to-import-device-information.md) para más información. |

**Soporte técnico**

| Periodo| Icono | Descripción |
|----|---|---|
| Soporte técnico | :::image type="icon" source="media/concept-sensor-console-overview/support-icon-azure.png" border="false"::: | Póngase en contacto con el [servicio de soporte técnico de Microsoft](https://support.microsoft.com/) para obtener ayuda. |

## <a name="see-also"></a>Consulte también

[Investigación y paquetes de inteligencia sobre amenazas](how-to-work-with-threat-intelligence-packages.md)

[Incorporación de sensores](getting-started.md#onboard-a-sensor)

[Administración de archivos de activación del sensor](how-to-manage-individual-sensors.md#manage-sensor-activation-files)

[Control del tráfico que se supervisa](how-to-control-what-traffic-is-monitored.md)
