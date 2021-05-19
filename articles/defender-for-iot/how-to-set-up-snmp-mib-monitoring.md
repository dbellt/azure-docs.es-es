---
title: Configuración de la supervisión de MIB del protocolo simple de administración de redes
description: Puede realizar la supervisión del estado del sensor mediante SNMP. El sensor responde a las consultas SNMP enviadas desde un servidor de supervisión autorizado.
ms.date: 12/14/2020
ms.topic: how-to
ms.openlocfilehash: 14803fd2f9c088fb4454f97ff1524e8d651ccd05
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109654839"
---
# <a name="set-up-snmp-mib-monitoring"></a>Configuración de la supervisión de MIB del protocolo simple de administración de redes

Puede realizar la supervisión del estado del sensor mediante el Protocolo simple de administración de redes (SNMP). El sensor responde a las consultas SNMP enviadas desde un servidor de supervisión autorizado. El monitor SNMP sondea los OID de sensor periódicamente (hasta 50 veces por segundo).

Las versiones admitidas de SNMP son SNMP v2 o SNMP v3. SNMP usa UDP como protocolo de transporte con el puerto 161 (SNMP).

Antes de empezar a configurar la supervisión SNMP, debe abrir el puerto UDP 161 en el firewall.

## <a name="oids"></a>OID

| Consola de administración y sensor | OID | Formato | Descripción |
|--|--|--|--|
| Nombre del dispositivo | 1.3.6.1.2.1.1.5.0 | STRING | Nombre del dispositivo de la consola de administración local |
| Vendor | 1.3.6.1.2.1.1.4.0 | STRING | Soporte técnico de Microsoft (support.microsoft.com) |
| Plataforma | 1.3.6.1.2.1.1.1.0 | STRING | Sensor o consola de administración local |
| Número de serie | 1.3.6.1.4.1.53313.1 |STRING | Cadena que la licencia usa |
| Versión del software | 1.3.6.1.4.1.53313.2 | STRING | Cadena de versión completa de Xsense y cadena de versión completa de administración |
| Uso de CPU | 1.3.6.1.4.1.53313.3.1 | GAUGE32 | Indicación de 0 a 100 |
| Temperatura de la CPU | 1.3.6.1.4.1.53313.3.2 | STRING | Indicación Celsius de 0 a 100 basada en la entrada de Linux. Se devolverá el mensaje "No se encontraron sensores" desde cualquier máquina que no tenga ningún sensor de temperatura física real (por ejemplo, máquinas virtuales).|
| Uso de la memoria | 1.3.6.1.4.1.53313.3.3 | GAUGE32 | Indicación de 0 a 100 |
| Uso de disco | 1.3.6.1.4.1.53313.3.4 | GAUGE32 | Indicación de 0 a 100 |
| Estado del servicio | 1.3.6.1.4.1.53313.5  |STRING | En línea o sin conexión si uno de los cuatro componentes esenciales está fuera de servicio |
| Estado del servicio | 1.3.6.1.4.1.53313.5  |STRING | En línea o sin conexión si uno de los cuatro componentes esenciales está fuera de servicio |
| Conexión local o en la nube | 1.3.6.1.4.1.53313.6   |STRING | Indica si el sensor está conectado al portal de Defender para IoT o si se administra solo en el entorno local |
| Estado de licencia | 1.3.6.1.4.1.53313.5  |STRING | Indica si el archivo de activación ha expirado o no |

   - Las claves no existentes responden con NULL, HTTP 200, basándose en [Stack Overflow](https://stackoverflow.com/questions/51419026/querying-for-non-existing-record-returns-null-with-http-200).
    
   - Las MIB relacionadas con el hardware (uso de CPU, temperatura de CPU, uso de memoria, uso de disco) deben probarse en todas las arquitecturas y sensores físicos. Se espera que la temperatura de la CPU en máquinas virtuales no sea aplicable.

Puede descargar el registro que contiene todas las consultas SNMP que recibe el sensor, incluidos los datos de conexión y los datos sin procesar del paquete.

Para definir la supervisión del estado de SNMP v2:

1. En el menú lateral, seleccione **Configuración del sistema**.

2. En el panel **Active Discovery** (Detección activa), seleccione **SNMP MIB Monitoring** (Supervisión de MIB de SNMP) :::image type="icon" source="media/how-to-set-up-snmp-mib-monitoring/snmp-icon.png" border="false":::.

    :::image type="content" source="media/how-to-set-up-snmp-mib-monitoring/edit-snmp.png" alt-text="Edite la ventana de SNMP.":::

3. En la sección **Allowed Hosts** (Hosts permitidos), seleccione **Agregar host** y escriba la dirección IP del servidor que realiza la supervisión del estado del sistema.

    :::image type="content" source="media/how-to-set-up-snmp-mib-monitoring/snmp-allowed-ip-addess.png" alt-text="Escriba la dirección IP de los hosts permitidos.":::

4. En la sección **Autenticación**, en el cuadro **SNMP v2 Community String** (Cadena de comunidad SNMP v2), escriba la cadena. La cadena de comunidad SNMP puede contener hasta 32 caracteres e incluir cualquier combinación de caracteres alfanuméricos (letras mayúsculas, minúsculas y números). No se permiten espacios.

5. Seleccione **Guardar**.

Para definir la supervisión del estado de SNMP v3:

1. En el menú lateral, seleccione **Configuración del sistema**.

2. En el panel **Active Discovery** (Detección activa), seleccione **SNMP MIB Monitoring** (Supervisión de MIB de SNMP) :::image type="icon" source="media/how-to-set-up-snmp-mib-monitoring/snmp-icon.png" border="false":::.

    :::image type="content" source="media/how-to-set-up-snmp-mib-monitoring/edit-snmp.png" alt-text="Edite la ventana de SNMP.":::

3. En la sección **Allowed Hosts** (Hosts permitidos), seleccione **Agregar host** y escriba la dirección IP del servidor que realiza la supervisión del estado del sistema.

    :::image type="content" source="media/how-to-set-up-snmp-mib-monitoring/snmp-allowed-ip-addess.png" alt-text="Escriba la dirección IP de los hosts permitidos.":::

4. En la sección **Autenticación**, establezca los parámetros siguientes:

    | Parámetro | Descripción |
    |--|--|
    | **Nombre de usuario** | El nombre de usuario SNMP puede contener hasta 32 caracteres e incluir cualquier combinación de caracteres alfanuméricos (letras mayúsculas, minúsculas y números). No se permiten espacios. <br /> <br />El nombre de usuario de la autenticación SNMP v3 debe estar configurado en el sistema y en el servidor SNMP. |
    | **Contraseña** | Escriba una contraseña de autenticación que distinga entre mayúsculas y minúsculas. La contraseña de autenticación puede contener de 8 a 12 caracteres e incluir cualquier combinación de caracteres alfanuméricos (letras mayúsculas, minúsculas y números). <br /> <br/>El nombre de usuario de la autenticación SNMP v3 debe estar configurado en el sistema y en el servidor SNMP. |
    | **Tipo de autenticación** | Seleccione MD5 o SHA. |
    | **Cifrado** | Seleccione DES o AES. |
    | **Clave secreta** | La clave debe contener exactamente ocho caracteres e incluir cualquier combinación de caracteres alfanuméricos (letras mayúsculas, minúsculas y números). |

5. Seleccione **Guardar**.

## <a name="see-also"></a>Consulte también

[Exportación de registros de la solución de problemas](how-to-troubleshoot-the-sensor-and-on-premises-management-console.md)
