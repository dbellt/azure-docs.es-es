---
title: Reenvío de la información de las alertas
description: Puede enviar información de alertas a los sistemas asociados mediante las reglas de reenvío.
ms.date: 12/02/2020
ms.topic: how-to
ms.openlocfilehash: 0fec86a6212cc0f4cb3df67280a0e6d39ed329ac
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109732234"
---
# <a name="forward-alert-information"></a>Reenvío de la información de las alertas

Puede enviar información de alertas a los asociados que están integrados con Azure Defender para IoT, a los servidores de syslog, a las direcciones de correo electrónico y más. Trabajar con reglas de reenvío le permite enviar rápidamente información de alertas a las partes interesadas en seguridad.  

Syslog y otras acciones de reenvío predeterminadas se entregan a través del sistema. Es posible que haya más acciones de reenvío disponibles al integrarse con proveedores asociados, como Microsoft Azure Sentinel, ServiceNow o Splunk.

:::image type="content" source="media/how-to-work-with-alerts-sensor/alert-information-screen.png" alt-text="Información de alertas.":::

Los administradores de Defender para IoT tienen permiso para usar las reglas de reenvío.

## <a name="about-forwarded-alert-information"></a>Acerca de la información de alertas reenviada

Las alertas proporcionan información acerca de una amplia gama de eventos operativos y de seguridad. Por ejemplo:

  - Fecha y hora de la alerta

  - Motor que detectó el evento

  - Título de la alerta y mensaje descriptivo

  - Gravedad de la alerta

  - Nombre y dirección IP del origen y destino

  - Tráfico sospechoso detectado

:::image type="content" source="media/how-to-work-with-alerts-sensor/address-scan-detected-screen.png" alt-text="Se detectó un análisis de direcciones.":::

La información pertinente se envía a los sistemas asociados cuando se han creado reglas de reenvío.

## <a name="create-forwarding-rules"></a>Creación de reglas de reenvío

Para crear una regla de reenvío, haga lo siguiente:

1. Seleccione **Reenvío** en el menú lateral.

   ::image type="content" source="media/how-to-work-with-alerts-sensor/create-forwarding-rule-screen.png" alt-text="Icono de creación de una regla de envío.":::

2. Seleccione **Create Forwarding Rule** (crear una regla de reenvío).

   :::image type="content" source="media/how-to-work-with-alerts-sensor/create-a-forwardong-rule.png" alt-text="Creación de una nueva regla de reenvío.":::

3. Escriba el nombre de la regla de reenvío.

### <a name="forwarding-rule-criteria"></a>Criterios de la regla de reenvío 

Defina los criterios por los que se desencadenará una regla de reenvío. El uso de criterios en las reglas de reenvío ayuda a identificar y administrar el volumen de información que se envía del sensor a los sistemas externos. Están disponibles las siguientes opciones:

**Protocolos**: la regla de reenvío solo se activa si el tráfico detectado se estaba ejecutando a través de protocolos específicos. Seleccione los protocolos necesarios en la lista desplegable o selecciónelos todos.

**Motores**: seleccione los motores necesarios o selecciónelos todos. Se enviarán las alertas de los motores seleccionados.

**Niveles de gravedad**: este es el incidente mínimo que se reenviará, en términos de nivel de gravedad. Por ejemplo, si selecciona **leve**, se reenviarán las alertas de gravedad leve y todas las alertas por encima de este nivel de gravedad. Los niveles están predefinidos.

### <a name="forwarding-rule-actions"></a>Acciones de regla de reenvío

Las acciones de regla de reenvío indican al sensor que reenvíe la información de alertas a los proveedores o servidores asociados. Puede crear varias acciones para cada regla de reenvío.

Además de las acciones de reenvío que se entregan con el sistema, pueden estar disponibles otras acciones al integrarse con los proveedores asociados. 

#### <a name="email-address-action"></a>Acción de dirección de correo electrónico

Envíe un correo que incluya la información de la alerta. Puede especificar una dirección de correo electrónico por regla.

Para definir el correo electrónico de la regla de reenvío, haga lo siguiente:

1. Escriba una sola dirección de correo electrónico. Si es necesario enviar más de un correo, cree otra acción.

2. Escriba la zona horaria para la marca de tiempo de la detección de alertas en SIEM.

3. Seleccione **Submit** (Enviar).

#### <a name="syslog-server-actions"></a>Acciones de servidor de Syslog

Se admiten los siguientes formatos:

- Mensajes de texto

- Mensajes CEF

- Mensajes LEEF

- Mensajes de objeto

:::image type="content" source="media/how-to-work-with-alerts-sensor/create-actions-rule.png" alt-text="Creación de las acciones de regla de reenvío.":::

Escriba los siguientes parámetros:

- Nombre de host y puerto de syslog.

- Protocolo TCP y UDP.

- Zona horaria para la marca de tiempo de la detección de alertas en SIEM.

- Archivo de certificado de cifrado TLS y archivo de clave para los servidores CEF (opcional).
    
:::image type="content" source="media/how-to-work-with-alerts-sensor/configure-encryption.png" alt-text="Configuración del cifrado para la regla de reenvío.":::

| Campos de salida del mensaje de texto de syslog | Descripción |
|--|--|
| Fecha y hora | Fecha y hora en que el equipo servidor de syslog recibió la información. |
| Priority | User.Alert |
| Hostname | Dirección IP del sensor |
| Protocolo | TCP o UDP |
| Message | Sensor: el nombre del sensor.<br /> Alerta: el título de la alerta.<br /> Escriba:  el tipo de la alerta. Puede ser **infracción del protocolo**, **infracción de la directiva**, **malware**, **anomalía** u **operativa**.<br /> Gravedad: Gravedad de la alerta. Puede ser **advertencia**, **leve**, **grave** o **crítica**.<br /> Origen: el nombre del dispositivo de origen.<br /> IP de origen: la dirección IP del dispositivo de origen.<br /> Destino: el nombre del dispositivo de destino.<br /> IP de destino: la dirección IP del dispositivo de destino.<br /> Mensaje: el mensaje de la alerta.<br /> Grupo de alertas: el grupo de alertas asociado a la alerta. |


| Salida del objeto syslog | Descripción |
|--|--|
| Fecha y hora |   Fecha y hora en que el equipo servidor de syslog recibió la información. |  
| Priority |    User.Alert | 
| Hostname |    Dirección IP del sensor | 
| Message | Nombre del sensor: el nombre del dispositivo. <br /> Hora de la alerta: el momento en que se detectó la alerta: puede variar en función de la hora de la máquina del servidor de syslog y depende de la configuración de zona horaria de la regla de reenvío. <br /> Título de la alerta: el título de la alerta. <br /> Mensaje de alerta: el mensaje de la alerta. <br /> Gravedad de la alerta: la gravedad de la alerta: **Advertencia**, **leve**, **grave** o **crítica**. <br /> Tipo de alerta: **Infracción del protocolo**, **infracción de la directiva**, **malware**, **anomalía** u **operativa**. <br /> Protocolo: el protocolo de la alerta.  <br /> **Source_MAC**: dirección IP, nombre, proveedor o sistema operativo del dispositivo de origen. <br /> Destination_MAC: dirección IP, nombre, proveedor o sistema operativo del destino. Si faltan datos, el valor será **N/D**. <br /> alert_group: el grupo de alertas asociado a la alerta. |


| Formato de salida CEF de syslog | Descripción |
|--|--|
| Fecha y hora | Fecha y hora en que el equipo servidor de syslog recibió la información. |
| Priority | User.Alert | 
| Hostname | Dirección IP del sensor |
| Message | CEF:0 <br />Azure Defender para IoT <br />Nombre del sensor: el nombre del dispositivo de sensor. <br />Versión del sensor <br />Título de la alerta: el título de la alerta. <br />msg: el mensaje de la alerta. <br />protocol: el protocolo de la alerta. <br />severity:  **advertencia**, **leve**, **grave** o **crítica**. <br />type:  **infracción del protocolo**, **infracción de la directiva**, **malware**, **anomalía** u **operativa**. <br /> start: el momento en que se detectó la alerta. <br />Puede variar en función de la hora de la máquina del servidor de syslog y depende de la configuración de zona horaria de la regla de reenvío. <br />src_ip: dirección IP del dispositivo de origen.  <br />dst_ip: dirección IP del dispositivo de destino.<br />cat: el grupo de alertas asociado a la alerta.  |

| Formato de salida LEEF de syslog | Descripción |
|--|--|
| Fecha y hora |   Fecha y hora en que el equipo servidor de syslog recibió la información. |  
| Priority |    User.Alert | 
| Hostname |    Dirección IP del sensor |
| Message | Nombre del sensor: el nombre del dispositivo de Azure Defender para IoT. <br />LEEF:1.0 <br />Azure Defender para IoT <br />Sensor  <br />Versión del sensor <br />Alerta de Azure Defender para IoT <br />título: el título de la alerta. <br />msg: el mensaje de la alerta. <br />protocol: el protocolo de la alerta.<br />severity:  **advertencia**, **leve**, **grave** o **crítica**. <br />type: el tipo de la alerta: **Infracción del protocolo**, **infracción de la directiva**, **malware**, **anomalía** u **operativa**. <br />start: la hora de la alerta. Tenga en cuenta que puede ser diferente de la hora en la máquina del servidor de syslog. (Esto depende de la configuración de zona horaria). <br />src_ip: dirección IP del dispositivo de origen.<br />dst_ip: dirección IP del dispositivo de destino. <br />cat: el grupo de alertas asociado a la alerta. |

Después de escribir la información, seleccione **Enviar**.

#### <a name="webhook-server-action"></a>Acción del servidor de webhooks

Envíe información de alertas a un servidor de webhooks. El trabajo con servidores de webhooks permite configurar integraciones que se suscriben a eventos de alerta con Defender para IoT. Cuando se desencadena un evento de alerta, la consola de administración envía una carga HTTP POST a la dirección URL configurada del webhook. Los webhooks se pueden usar para actualizar un sistema SIEM externo, sistemas SOAR, sistemas de administración de incidentes, etc.   

**Para definir en una acción de webhook:**

1. Seleccione la acción de webhook.

:::image type="content" source="media/how-to-work-with-alerts-sensor/webhook.png" alt-text="Defina una regla de reenvío de webhook.":::

1. Escriba la dirección del servidor en el campo **URL**.
1. En los campos **Clave** y **Valor**, personalice el encabezado HTTP con una definición de clave y valor. Las claves solo pueden contener letras, números, guiones y subrayados. Los valores solo pueden contener un espacio inicial o uno final.
1. Seleccione **Guardar**.

#### <a name="netwitness-action"></a>Acción de NetWitness

Envíe información de alertas a un servidor de NetWitness.

Para definir los parámetros de reenvío de NetWitness, haga lo siguiente:

1. Escriba la información de **nombre de host** y **puerto** de NetWitness.

2. Escriba la zona horaria para la marca de tiempo de la detección de alertas en SIEM.

   :::image type="content" source="media/how-to-work-with-alerts-sensor/add-timezone.png" alt-text="Agregue una zona horaria a la regla de reenvío.":::

3. Seleccione **Submit** (Enviar).

#### <a name="integrated-vendor-actions"></a>Acciones de proveedor integradas

Es posible que haya integrado su sistema con un proveedor de seguridad, administración de dispositivos u otro sector. Estas integraciones le permiten hacer lo siguiente:

  - Enviar la información de las alertas.

  - Enviar la información del inventario de dispositivos.

  - Comunicarse con los firewalls del proveedor.

Las integraciones ayudan a conectar soluciones de seguridad que antes estaban aisladas, mejorar la visibilidad del dispositivo y acelerar la respuesta en todo el sistema para mitigar los riesgos con mayor rapidez.

Use la sección Acciones para especificar las credenciales y otra información necesaria para comunicarse con los proveedores integrados.

Para obtener más información sobre cómo configurar las reglas de reenvío para las integraciones, consulte los artículos de integración de asociados correspondientes.

### <a name="test-forwarding-rules"></a>Prueba de reglas de reenvío

Pruebe la conexión entre el sensor y el servidor de asociado que está definido en las reglas de reenvío:

1. Seleccione la regla en el cuadro de diálogo **Regla de reenvío**.

2. Seleccione el cuadro **Más**.

3. Seleccione **Enviar mensaje de prueba**.

4. Vaya al sistema de asociado para comprobar que se ha recibido la información enviada por el sensor.

### <a name="edit-and-delete-forwarding-rules"></a>Edición y eliminación de reglas de reenvío 

Para editar una regla de reenvío, haga lo siguiente:

- En la pantalla **Regla de reenvío**, seleccione **Editar** en el menú desplegable **Más**. Realice los cambios necesarios y seleccione **Enviar**.

Para eliminar una regla de reenvío, haga lo siguiente:

- En la pantalla **Regla de reenvío**, seleccione **Eliminar** en el menú desplegable **Más**. En el cuadro de diálogo **Advertencia**, seleccione **Aceptar**.

### <a name="forwarding-rules-and-alert-exclusion-rules"></a>Reglas de reenvío y reglas de exclusión de alertas

Puede que el administrador haya definido reglas de exclusión de alertas. Dichas reglas ayudan a los administradores a lograr un control más detallado sobre la activación de las alertas, ya que indican al sensor que omita los eventos de la alerta según varios parámetros. Estos parámetros pueden incluir direcciones de dispositivo, nombres de alerta o sensores específicos.

Es decir, que las reglas de reenvío que defina podrían omitirse en función de las reglas de exclusión que haya creado el administrador. Las reglas de exclusión se definen en la consola de administración local.

## <a name="see-also"></a>Consulte también

[Aceleración de los flujos de trabajo de alertas](how-to-accelerate-alert-incident-response.md)
