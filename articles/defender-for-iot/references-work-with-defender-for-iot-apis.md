---
title: Trabajo con las API de Defender para IoT
description: Use una API REST externa para acceder a los datos que han detectado los sensores y las consolas de administración y realizar acciones con esos datos.
ms.date: 12/14/2020
ms.topic: reference
ms.openlocfilehash: e7833a20d4f708ecb5b80394fae2c56fc07c9489
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107752739"
---
# <a name="defender-for-iot-sensor-and-management-console-apis"></a>API del sensor y la consola de administración de Defender para IoT

Use una API REST externa para acceder a los datos que han detectado los sensores y las consolas de administración y realizar acciones con esos datos.

Las conexiones están protegidas mediante SSL.

## <a name="getting-started"></a>Introducción

En general, cuando se usa una API externa en el sensor o la consola de administración local de Azure Defender para IoT, es necesario generar un token de acceso. No se requieren tokens para las API de autenticación que se usan en el sensor y en la consola de administración local.

Para generar un token, haga lo siguiente:

1. En la ventana **Configuración del sistema**, seleccione **Tokens de acceso**.
  
   :::image type="content" source="media/references-work-with-defender-for-iot-apis/access-tokens.png" alt-text="Captura de pantalla de la ventana Configuración del sistema con el botón Tokens de acceso resaltado.":::

2. Seleccione **Generar nuevo token**.
   
   :::image type="content" source="media/references-work-with-defender-for-iot-apis/new-token.png" alt-text="Seleccione el botón para generar un nuevo token.":::

3. Describa el objetivo del nuevo token y seleccione **Siguiente**.
   
   :::image type="content" source="media/references-work-with-defender-for-iot-apis/token-name.png" alt-text="Genere un token nuevo y escriba el nombre de la integración asociada al mismo.":::

4. Se muestra el token de acceso. Cópielo, ya que no se volverá a mostrar.
   
   :::image type="content" source="media/references-work-with-defender-for-iot-apis/token-code.png" alt-text="Copie el token de acceso para la integración.":::

5. Seleccione **Finalizar**. Los tokens que cree aparecerán en el cuadro de diálogo **Tokens de acceso**.
   
   :::image type="content" source="media/references-work-with-defender-for-iot-apis/access-token-window.png" alt-text="Captura de pantalla del cuadro de diálogo Tokens de dispositivo con tokens rellenados":::

   **Usado** indica la última vez que se recibió una llamada externa con este token.

   Si se muestra el valor **N/A** en el campo **Usado** de este token, la conexión entre el sensor y el servidor conectado no funciona.

6. Agregue un encabezado HTTP titulado **Authorization** a su solicitud y establezca su valor en el token que ha generado.

## <a name="sensor-api-specifications"></a>Especificaciones de la API de sensor

En esta sección se describen las siguientes API de sensor:

- [Recuperación de información del dispositivo: /api/v1/devices](#retrieve-device-information---apiv1devices)

- [Recuperación de información de conexión del dispositivo: /api/v1/devices/connections](#retrieve-device-connection-information---apiv1devicesconnections)

- [Recuperación de información en CVE: /api/v1/devices/cves](#retrieve-information-on-cves---apiv1devicescves)

- [Recuperación de información de alertas: /api/v1/alerts](#retrieve-alert-information---apiv1alerts)

- [Recuperación de los eventos de escala de tiempo: /api/v1/events](#retrieve-timeline-events---apiv1events)

- [Recuperación de información de vulnerabilidad: /api/v1/reports/vulnerabilities/devices](#retrieve-vulnerability-information---apiv1reportsvulnerabilitiesdevices)

- [Recuperación de vulnerabilidades de seguridad: /api/v1/reports/vulnerabilities/security](#retrieve-security-vulnerabilities---apiv1reportsvulnerabilitiessecurity)

- [Recuperación de vulnerabilidades operativas: /api/v1/reports/vulnerabilities/operational](#retrieve-operational-vulnerabilities---apiv1reportsvulnerabilitiesoperational)

- [Validación de credenciales de usuario: /api/external/authentication/validation](#validate-user-credentials---apiexternalauthenticationvalidation)

- [Cambio de contraseña: /external/authentication/set_password](#change-password---externalauthenticationset_password)

- [Actualización de la contraseña de usuario por el administrador del sistema: /external/authentication/set_password_by_admin](#user-password-update-by-system-admin---externalauthenticationset_password_by_admin)

### <a name="retrieve-device-information---apiv1devices"></a>Recuperación de información del dispositivo: /api/v1/devices

Use esta API para solicitar una lista de todos los dispositivos que ha detectado un sensor de Defender para IoT.

#### <a name="method"></a>Método

**GET**

Solicita una lista de todos los dispositivos que ha detectado el sensor de Defender para IoT.

#### <a name="query-parameters"></a>Parámetros de consulta

- **authorized**: para filtrar solo los dispositivos autorizados y no autorizados.

  **Ejemplos**:

  `/api/v1/devices?authorized=true`

  `/api/v1/devices?authorized=false`

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Matriz de objetos JSON que representan a dispositivos.

#### <a name="device-fields"></a>Campos de dispositivo

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **id** | Numeric | No | - |
| **ipAddresses** | Matriz JSON | Sí | Direcciones IP (puede ser más de una dirección en el caso de direcciones de Internet o de un dispositivo con dos adaptadores de red) |
| **name** | String | No | - |
| **type** | String | No | Desconocido, estación de ingeniería, PLC, HMI, analista, controlador de dominio, servidor de base de servidores, punto de acceso inalámbrico, enrutador, conmutador, servidor, estación de trabajo, cámara IP, impresora, firewall, estación de la terminal, VPN Gateway, Internet o multidifusión y difusión |
| **macAddresses** | Matriz JSON | Sí | Direcciones MAC (puede ser más de una dirección en el caso de un dispositivo con dos adaptadores de red) |
| **operatingSystem** | String | Sí | - |
| **engineeringStation** | Boolean | No | Verdadero o falso |
| **scanner** | Boolean | No | Verdadero o falso |
| **autorizado** | Boolean | No | Verdadero o falso |
| **vendor** | String | Sí | - |
| **protocols** | Matriz JSON | Sí | Objeto Protocol |
| **firmware** | Matriz JSON | Sí | Objeto Firmware |

#### <a name="protocol-fields"></a>Campos del protocolo

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **Nombre** | String | No |  |
| **Direcciones** | Matriz JSON | Sí | Patrón o valores numéricos |

#### <a name="firmware-fields"></a>Campos de firmware

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **serial** | String | No | N/D o el valor real |
| **model** | String | No | N/D o el valor real |
| **firmwareVersion** | Double | No | N/D o el valor real |
| **additionalData** | String | No | N/D o el valor real |
| **moduleAddress** | String | No | N/D o el valor real |
| **rack** | String | No | N/D o el valor real |
| **slot** | String | No | N/D o el valor real |
| **address** | String | No | N/D o el valor real |

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
[

    {
    
    "vendor": null,
    
    "name": "10.4.14.102",
    
    "firmware": [
    
        {
        
            "slot": "N/A",
            
            "additionalData": "N/A",
            
            "moduleAddress": "Network: Local network (0), Node: 0, Unit: CPU (0x0)",
            
            "rack": "N/A",
            
            "address": "10.4.14.102",
            
            "model": "AAAAAAAAAA",
            
            "serial": "N/A",
            
            "firmwareVersion": "20.55"
        
        },
    
        {
        
            "slot": "N/A",
            
            "additionalData": "N/A",
            
            "moduleAddress": "Network: Local network (0), Node: 0, Unit: Unknown (0x3)",
            
            "rack": "N/A",
            
            "address": "10.4.14.102",
            
            "model": "AAAAAAAAAAAAAAAAAAAA",
            
            "serial": "N/A",
            
            "firmwareVersion": "20.55"
        
        },
    
        {
        
            "slot": "N/A",
            
            "additionalData": "N/A",
            
            "moduleAddress": "Network: Local network (0), Node: 3, Unit: CPU (0x0)",
            
            "rack": "N/A",
            
            "address": "10.4.14.102",
            
            "model": "AAAAAAAAAAAAAAAAAAAA",
            
            "serial": "N/A",
            
            "firmwareVersion": "20.55"
        
        },
    
        {
        
            "slot": "N/A",
            
            "additionalData": "N/A",
            
            "moduleAddress": "Network: 3, Node: 0, Unit: CPU (0x0)",
            
            "rack": "N/A",
            
            "address": "10.4.14.102",
            
            "model": "AAAAAAAAAAAAAAAAAAAA",
            
            "serial": "N/A",
            
            "firmwareVersion": "20.55"
        
        }
    
    ],
    
    "id": 79,
    
    "macAddresses": null,
    
    "authorized": true,
    
    "ipAddresses": [
    
        "10.4.14.102"
    
    ],
    
    "engineeringStation": false,
    
    "type": "PLC",
    
    "operatingSystem": null,
    
    "protocols": [
    
        {
        
            "addresses": [],
            
            "id": 62,
            
            "name": "Omron FINS"
        
        }
    
    ],
    
    "scanner": false
    
}

]
```

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| GET | curl -k -H "Authorization: <TOKEN_AUT>" https://<DIRECCIÓN_IP>/api/v1/devices | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:<span>//127<span>.0.0.1/api/v1/devices?authorized=true |

### <a name="retrieve-device-connection-information---apiv1devicesconnections"></a>Recuperación de información de conexión del dispositivo: /api/v1/devices/connections

Use esta API para solicitar una lista de todas las conexiones por cada dispositivo.

#### <a name="method"></a>Método

**GET**

#### <a name="query-parameters"></a>Parámetros de consulta

Si no establece los parámetros de la consulta, se devuelven todas las conexiones del dispositivo.

**Ejemplo**:

`/api/v1/devices/connections`

- **deviceId**: filtre según el id. de un dispositivo específico para ver sus conexiones.

  **Ejemplo**:

  `/api/v1/devices/<deviceId>/connections`

- **lastActiveInMinutes**: periodo de tiempo a partir de este momento hacia atrás, por minuto, durante el que las conexiones estuvieron activas.

  **Ejemplo**:

  `/api/v1/devices/2/connections?lastActiveInMinutes=20`

- **discoveredBefore**: filtre solo las conexiones que se detectaron antes de una hora específica (en milisegundos, hora UTC).

  **Ejemplo**:

  `/api/v1/devices/2/connections?discoveredBefore=<epoch>`

- **discoveredAfter**: filtre solo las conexiones que se detectaron después de una hora específica (en milisegundos, hora UTC).

  **Ejemplo**:

  `/api/v1/devices/2/connections?discoveredAfter=<epoch>`

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Matriz de objetos JSON que representan a conexiones de dispositivo.

#### <a name="fields"></a>Campos

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **firstDeviceId** | Numeric | No | - |
| **secondDeviceId** | Numeric | No | - |
| **lastSeen** | Numeric | No | Tiempo (hora UTC) |
| **discovered** | Numeric | No | Tiempo (hora UTC) |
| **ports** | Matriz de números | No | - |
| **protocols** | Matriz JSON | No | Campo del protocolo |

#### <a name="protocol-field"></a>Campo del protocolo

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **name** | String | No | - |
| **commands** | Matriz de cadena | No | - |

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
[

    {
    
        "firstDeviceId": 171,
        
        "secondDeviceId": 22,
        
        "lastSeen": 1511281457933,
        
        "discovered": 1511872830000,
        
        "ports": [
        
            502
        
        ],
    
        "protocols": [
        
        {
        
            name: "modbus",
            
            commands: [
            
                "Read Coils"
        
            ]
        
        },
    
        {
        
            name: "ams",
            
            commands: [
            
                "AMS Write"
        
            ]
        
        },
    
        {
        
            name: "http",
            
            commands: [
        
            ]
        
        }
    
    ]
    
    },
    
    {
    
        "firstDeviceId": 171,
        
        "secondDeviceId": 23,
        
        "lastSeen": 1511281457933,
        
        "discovered": 1511872830000,
        
        "ports": [
        
            502
        
        ],
        
        "protocols": [
        
            {
            
                name: "s7comm",
                
                commands: [
                
                    "Download block",
                    
                    "Upload"
            
                ]
            
            }
        
        ]
    
    }

]
```

#### <a name="curl-command"></a>Comando Curl

> [!div class="mx-tdBreakAll"]
> | Tipo | API existentes | Ejemplo |
> |--|--|--|
> | GET | curl -k -H "Authorization: <TOKEN_AUT>" https://<DIRECCIÓN_IP>/api/v1/devices/connections | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/<span>/127.0.0.1/api/v1/devices/connections |
> | GET | curl -k -H "Authorization: <TOKEN_AUT>" 'https://<DIRECCIÓN_IP>/api/v1/devices/<deviceId>/connections?lastActiveInMinutes=&discoveredBefore=&discoveredAfter=' | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" 'https:/<span>/127.0.0.1/api/v1/devices/2/connections?lastActiveInMinutes=20&discoveredBefore=1594550986000&discoveredAfter=1594550986000' |

### <a name="retrieve-information-on-cves---apiv1devicescves"></a>Recuperación de información en CVE: /api/v1/devices/cves

Use esta API para solicitar una lista de todos los CVE conocidos que se detectaron en los dispositivos de la red.

#### <a name="method"></a>Método

**GET**

#### <a name="query-parameters"></a>Parámetros de consulta

De manera predeterminada, esta API proporciona la lista de todas las direcciones IP de dispositivos con CVE, hasta 100 CVE de puntuación superior para cada dirección IP.

**Ejemplo**:

`/api/v1/devices/cves`

- **deviceId**: filtre según la dirección IP de un dispositivo específica para obtener hasta 100 CVE de puntuación superior identificados en ese dispositivo específico.

  **Ejemplo**:

  `/api/v1/devices/<ipAddress>/cves`

- **top**: cuántos CVE de puntuación superior se deben recuperar para cada dirección IP del dispositivo.

  **Ejemplo**:

  `/api/v1/devices/cves?top=50`

  `/api/v1/devices/<ipAddress>/cves?top=50`

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Matriz de objetos JSON que representan a los CVE identificados en las direcciones IP.

#### <a name="fields"></a>Campos

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **cveId** | String | No | - |
| **ipAddress** | String | No | Dirección IP |
| **score** | String | No | 0,0 a 10,0 |
| **attackVector** | String | No | Red, red adyacente, local o física |
| **description** | String | No | - |

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
[

    {
    
        "cveId": "CVE-2007-0099",
        
        "score": "9.3",
        
        "ipAddress": "10.35.1.51",
        
        "attackVector": "NETWORK",
        
        "description": "Race condition in the msxml3 module in Microsoft XML Core
        
        Services 3.0, as used in Internet Explorer 6 and other
        
        applications, allows remote attackers to execute arbitrary
        
        code or cause a denial of service (application crash) via many
        
        nested tags in an XML document in an IFRAME, when synchronous
        
        document rendering is frequently disrupted with asynchronous
        
        events, as demonstrated using a JavaScript timer, which can
        
        trigger NULL pointer dereferences or memory corruption, aka
        
        \"MSXML Memory Corruption Vulnerability.\""
    
    },
    
    {
    
        "cveId": "CVE-2009-1547",
        
        "score": "9.3",
        
        "ipAddress": "10.35.1.51",
        
        "attackVector": "NETWORK",
        
        "description": "Unspecified vulnerability in Microsoft Internet Explorer 5.01
        
        SP4, 6, 6 SP1, and 7 allows remote attackers to execute
        
        arbitrary code via a crafted data stream header that triggers
        
        memory corruption, aka \"Data Stream Header Corruption
        
        Vulnerability.\""
    
    }

]
```

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| GET | curl -k -H "Authorization: <TOKEN_AUT>" https://<DIRECCIÓN_IP>/api/v1/devices/cves | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/<span>/127.0.0.1/api/v1/devices/cves |
| GET | curl -k -H "Authorization: <TOKEN_AUT>" https://<DIRECCIÓN_IP>/api/v1/devices/<deviceIpAddress>/cves?top= | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/<span>/127.0.0.1/api/v1/devices/10.10.10.15/cves?top=50 |

### <a name="retrieve-alert-information---apiv1alerts"></a>Recuperación de información de alertas: /api/v1/alerts

Use esta API para solicitar una lista de todas las alertas que ha detectado el sensor de Defender para IoT.

#### <a name="method"></a>Método

**GET**

#### <a name="query-parameters"></a>Parámetros de consulta

- **state**: para filtrar solo las alertas controladas o no controladas.

  **Ejemplo**:

  `/api/v1/alerts?state=handled`

- **fromTime**: para filtrar las alertas creadas a partir de una hora específica (en milisegundos, hora UTC).

  **Ejemplo**:

  `/api/v1/alerts?fromTime=<epoch>`

- **toTime**: para filtrar solo las alertas creadas antes de una hora específica (en milisegundos, hora UTC).

  **Ejemplo**:

  `/api/v1/alerts?toTime=<epoch>`

- **type**: para filtrar las alertas por un tipo específico. Tipos existentes para filtrar: nuevos dispositivos inesperados, desconexiones.

  **Ejemplo**:

  `/api/v1/alerts?type=disconnections`

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Matriz de objetos JSON que representan alertas.

#### <a name="alert-fields"></a>Campos de alerta

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **Id** | Numeric | No | - |
| **time** | Numeric | No | Tiempo (hora UTC) |
| **title** | String | No | - |
| **message** | String | No | - |
| **severity** | String | No | Advertencia, leve, grave o crítica |
| **engine** | String | No | Infracción del protocolo, infracción de la directiva, malware, anomalía u operativa |
| **sourceDevice** | Numeric | Sí | Id. de dispositivo |
| **destinationDevice** | Numeric | Sí | Id. de dispositivo |
| **sourceDeviceAddress** | Numeric | Sí | IP, MAC, Null |
| **destinationDeviceAddress** | Numeric | Sí | IP, MAC, Null |
| **remediationSteps** | String | Sí | Pasos para la corrección descritos en la alerta |
| **additionalInformation** | Objeto Additional information | Sí | - |

Tenga en cuenta que se requiere /api/v2/ para la información siguiente:

- sourceDeviceAddress 
- destinationDeviceAddress
- remediationSteps

#### <a name="additional-information-fields"></a>Campos de información adicional

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **description** | String | No | - |
| **information** | Matriz JSON | No | String |

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
[

    {
    
        "engine": "Policy Violation",
        
        "severity": "Major",
        
        "title": "Internet Access Detected",
        
        "additionalinformation": {
        
            "information": [
            
                "170.60.50.201 over port BACnet (47808)"
            
            ],
            
            "description": "External Addresses"
        
        },
    
        "sourceDevice": null,
        
        "destinationDevice": null,
        
        "time": 1509881077000,
        
        "message": "Device 192.168.0.13 tried to access an external IP address which is an address in the Internet and is not allowed by policy. It is recommended to notify the security officer of the incident.",
        
        "id": 1
    
    },
    
    {
    
        "engine": "Protocol Violation",
        
        "severity": "Major",
        
        "title": "Illegal MODBUS Operation (Exception Raised by Master)",
        
        "sourceDevice": 3,
        
        "destinationDevice": 4,
        
        "time": 1505651605000,
        
        "message": "A MODBUS master 192.168.110.131 attempted to initiate an illegal operation.\nThe operation is considered to be illegal since it incorporated function code \#129 which should not be used by a master.\nIt is recommended to notify the security officer of the incident.",
        
        "id": 2,
        
        "additionalInformation": null,
    
    }

]

```

#### <a name="curl-command"></a>Comando Curl

> [!div class="mx-tdBreakAll"]
> | Tipo | API existentes | Ejemplo |
> |--|--|--|
> | GET | curl -k -H "Authorization: <TOKEN_AUT>" 'https://<DIRECCIÓN_IP>/api/v1/alerts?state=&fromTime=&toTime=&type=' | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" 'https:/<span>/127.0.0.1/api/v1/alerts?state=unhandled&fromTime=1594550986000&toTime=1594550986001&type=disconnections' |

### <a name="retrieve-timeline-events---apiv1events"></a>Recuperación de los eventos de escala de tiempo: /api/v1/events

Use esta API para solicitar una lista de los eventos informados a la escala de tiempo del evento.

#### <a name="method"></a>Método

**GET**

#### <a name="query-parameters"></a>Parámetros de consulta

- **minutesTimeFrame**: período de tiempo de este momento hacia atrás, por minuto, en el que se informó de los eventos.

  **Ejemplo**:

  `/api/v1/events?minutesTimeFrame=20`

- **type**: para filtrar la lista de eventos por un tipo específico.

  **Ejemplos**:

  `/api/v1/events?type=DEVICE_CONNECTION_CREATED`

  `/api/v1/events?type=REMOTE_ACCESS&minutesTimeFrame`

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Matriz de objetos JSON que representan alertas.

#### <a name="event-fields"></a>Campos del evento

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|--|
| **timestamp** | Numeric | No | Tiempo (hora UTC) |
| **title** | String | No | - |
| **severity** | String | No | INFORMACIÓN, AVISO o ALERTA |
| **owner** | String | Sí | Si el evento se creó manualmente, este campo incluirá el nombre del usuario que creó el evento. |
| **content** | String | No | - |

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
[

    {
    
        "severity": "INFO",
        
        "title": "Back to Normal",
        
        "timestamp": 1504097077000,
        
        "content": "Device 10.2.1.15 was found responsive, after being suspected as disconnected",
        
        "owner": null,
        
        "type": "BACK_TO_NORMAL"
    
    },
    
    {
    
        "severity": "ALERT",
        
        "title": "Alert Detected",
        
        "timestamp": 1504096909000,
        
        "content": "Device 10.2.1.15 is suspected to be disconnected (unresponsive).",
        
        "owner": null,
        
        "type": "ALERT_REPORTED"
    
    },
    
    {
    
        "severity": "ALERT",
        
        "title": "Alert Detected",
        
        "timestamp": 1504094446000,
        
        "content": "A DNP3 Master 10.2.1.14 attempted to initiate a request which is not allowed by policy.\nThe policy indicates the allowed function codes, address ranges, point indexes and time intervals.\nIt is recommended to notify the security officer of the incident.",
        
        "owner": null,
        
        "type": "ALERT_REPORTED"
    
    },
    
    {
    
        "severity": "NOTICE",
        
        "title": "PLC Program Update",
        
        "timestamp": 1504094344000,
        
        "content": "Program update detected, sent from 10.2.1.25 to 10.2.1.14",
        
        "owner": null,
        
        "type": "PROGRAM_DEVICE"
    
    }

]

```

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| GET | curl -k -H "Authorization: <TOKEN_AUT>" 'https://<DIRECCIÓN_IP>/api/v1/events?minutesTimeFrame=&type=' | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" 'https:/<span>/127.0.0.1/api/v1/events?minutesTimeFrame=20&type=DEVICE_CONNECTION_CREATED' |

### <a name="retrieve-vulnerability-information---apiv1reportsvulnerabilitiesdevices"></a>Recuperación de información de vulnerabilidad: /api/v1/reports/vulnerabilities/devices

Use esta API para solicitar los resultados de la evaluación de vulnerabilidades de cada dispositivo.

#### <a name="method"></a>Método

**GET**

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Matriz de objetos JSON que representan a dispositivos evaluados.

El objeto Device contiene:

- Datos generales

- Puntuación de evaluación

- Puntos vulnerables

#### <a name="device-fields"></a>Campos de dispositivo

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **name** | String | No | - |
| **ipAddresses** | Matriz JSON | No | - |
| **securityScore** | Numeric | No | - |
| **vendor** | String | Sí |  |
| **firmwareVersion** | String | Sí | - |
| **model** | String | Sí | - |
| **isWirelessAccessPoint** | Boolean | No | Verdadero o falso |
| **operatingSystem** | Objeto Operating system | Sí | - |
| **vulnerabilities** | Objeto Vulnerabilities | Sí | - |

#### <a name="operating-system-fields"></a>Campos del sistema operativo

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **Nombre** | String | Sí | - |
| **Tipo** | String | Sí | - |
| **Versión** | String | Sí | - |
| **latestVersion** | String | Sí | - |

#### <a name="vulnerabilities-fields"></a>Campos de vulnerabilidades
 
| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **antiViruses** | Matriz JSON | Sí | Nombres de antivirus |
| **plainTextPasswords** | Matriz JSON | Sí | Objetos Password |
| **remoteAccess** | Matriz JSON | Sí | Objetos Remote access |
| **isBackupServer** | Boolean | No | Verdadero o falso |
| **openedPorts** | Matriz JSON | Sí | Objetos Opened port |
| **isEngineeringStation** | Boolean | No | Verdadero o falso |
| **isKnownScanner** | Boolean | No | Verdadero o falso |
| **cves** | Matriz JSON | Sí | Objetos CVE |
| **isUnauthorized** | Boolean | No | Verdadero o falso |
| **malwareIndicationsDetected** | Boolean | No | Verdadero o falso |
| **weakAuthentication** | Matriz JSON | Sí | Aplicaciones detectadas que usan una autenticación débil |

#### <a name="password-fields"></a>Campos de contraseña

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **password** | String | No | - |
| **protocolo** | String | No | - |
| **strength** | String | No | Muy débil, débil, media o segura. |

#### <a name="remote-access-fields"></a>Campos de acceso remoto

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **port** | Numeric | No | - |
| **transport** | String | No | TCP o UDP |
| **client** | String | No | Dirección IP |
| **clientSoftware** | String | No | SSH, VNC, escritorio remoto o TeamViewer |

#### <a name="open-port-fields"></a>Campos de puerto abierto

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **port** | Numeric | No | - |
| **transport** | String | No | TCP o UDP |
| **protocolo** | String | Sí | - |
| **isConflictingWithFirewall** | Boolean | No | Verdadero o falso |

#### <a name="cve-fields"></a>Campos de CVE

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **Id** | String | No | - |
| **score** | Numeric | No | Double |
| **description** | String | No | - |

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
[

    {
    
        "name": "IED \#10",
        
        "ipAddresses": ["10.2.1.10"],
        
        "securityScore": 100,
        
        "vendor": "ABB Switzerland Ltd, Power Systems",
        
        "firmwareVersion": null,
        
        "model": null,
        
        "operatingSystem": {
        
            "name": "ABB Switzerland Ltd, Power Systems",
            
            "type": "abb",
            
            "version": null,
            
            "latestVersion": null
        
        },
        
        "vulnerabilities": {
            
        "antiViruses": [
            
        "McAfee"
            
        ],
            
        "plainTextPasswords": [
            
            {
            
                "password": "123456",
                
                "protocol": "HTTP",
                
                "lastSeen": 1462726930471,
                
                "strength": "Very Weak"
            
            }
            
        ],
            
        "remoteAccess": [
            
            {
            
                "port": 5900,
                
                "transport": "TCP",
                
                "clientSoftware": "VNC",
                
                "client": "10.2.1.20"
            
            }
            
        ],
            
        "isBackupServer": true,
            
        "openedPorts": [
            
            {
            
                "port": 445,
                
                "transport": "TCP",
                
                "protocol": "SMP Over IP",
                
                "isConflictingWithFirewall": false
            
            },
            
            {
            
                "port": 80,
                
                "transport": "TCP",
                
                "protocol": "HTTP",
                
                "isConflictingWithFirewall": false
            
            }
            
        ],
            
        "isEngineeringStation": false,
            
        "isKnownScanner": false,
            
        "cves": [
            
            {
            
                "id": "CVE-2015-6490",
                
                "score": 10,
                
                "description": "Frosty URL - Stack-based buffer overflow on Allen-Bradley MicroLogix 1100 devices before B FRN 15.000 and 1400 devices through B FRN 15.003 allows remote attackers to execute arbitrary code via unspecified vectors"
            
            },
            
            {
            
                "id": "CVE-2012-6437",
                
                "score": 10,
                
                "description": "MicroLogix 1100 and 1400 do not properly perform authentication for Ethernet firmware updates, which allows remote attackers to execute arbitrary code via a Trojan horse update image"
            
            },
            
            {
            
                "id": "CVE-2012-6440",
                
                "score": 9.3,
                
                "description": "MicroLogix 1100 and 1400 allows man-in-the-middle attackers to conduct replay attacks via HTTP traffic."
        
            }
        
        ],
        
        "isUnauthorized": false,
        
        "malwareIndicationsDetected": true
        
        }
    
    }

]

```

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| GET | curl -k -H "Authorization: <TOKEN_AUT>" https://<DIRECCIÓN_IP>/api/v1/reports/vulnerabilities/devices | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/<span>/127.0.0.1/api/v1/reports/vulnerabilities/devices |

### <a name="retrieve-security-vulnerabilities---apiv1reportsvulnerabilitiessecurity"></a>Recuperación de vulnerabilidades de seguridad: /api/v1/reports/vulnerabilities/security

Use esta API para solicitar los resultados de una evaluación de vulnerabilidades general. Esta evaluación proporciona información sobre el nivel de seguridad del sistema.

Esta evaluación se basa en la información general del sistema y de la red, y no en la valoración de un dispositivo específico.

#### <a name="method"></a>Método

**GET**

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Objeto JSON que representa a los resultados evaluados. Cada clave puede admitir valores NULL. De lo contrario, contendrá un objeto JSON con claves que no aceptan valores NULL.

### <a name="result-fields"></a>Campos de resultado

**Claves**

**unauthorizedDevices**

| Nombre del campo | Tipo | Lista de valores |
| ---------- | ---- | -------------- |
| **address** | String | Dirección IP |
| **name** | String | - |
| **firstDetectionTime** | Numeric | Tiempo (hora UTC) |
| lastSeen | Numeric | Tiempo (hora UTC) |

**illegalTrafficByFirewallRules**

| Nombre del campo | Tipo | Lista de valores |
| ---------- | ---- | -------------- |
| **servidor** | String | Dirección IP |
| **client** | String | Dirección IP |
| **port** | Numeric | - |
| **transport** | String | TCP, UDP o ICMP |

**weakFirewallRules**

| Nombre del campo | Tipo | Lista de valores |
| ---------- | ---- | -------------- |
| **sources** | Matriz JSON de fuentes. Cada origen puede estar en cualquiera de los siguientes cuatro formatos. | "Cualquiera", "dirección IP (host)", "de IP a IP (INTERVALO)", "dirección IP, máscara de subred (RED)" |
| **destinations** | Matriz JSON de destinos. Cada destino puede estar en cualquiera de los siguientes cuatro formatos. | "Cualquiera", "dirección IP (host)", "de IP a IP (INTERVALO)", "dirección IP, máscara de subred (RED)" |
| **ports** | Matriz JSON de puertos en cualquiera de los siguientes tres formatos | "Cualquiera", "puerto (protocolo, si se detecta)", "de puerto a puerto (protocolo, si se detecta)" |

**accessPoints**

| Nombre del campo | Tipo | Lista de valores |
| ---------- | ---- | -------------- |
| **macAddress** | String | Dirección MAC |
| **vendor** | String | Nombre del proveedor |
| **ipAddress** | String | Dirección IP o N/D |
| **name** | String | Nombre de dispositivo o N/D |
| **wireless** | String | No, posible o sí |

**connectionsBetweenSubnets**

| Nombre del campo | Tipo | Lista de valores |
| ---------- | ---- | -------------- |
| **servidor** | String | Dirección IP |
| **client** | String | Dirección IP |

**industrialMalwareIndicators**

| Nombre del campo | Tipo | Lista de valores |
| ---------- | ---- | -------------- |
| **detectionTime** | Numeric | Tiempo (hora UTC) |
| **alertMessage** | String | - |
| **description** | String | - |
| **devices** | Matriz JSON | Nombres de dispositivo | 

**internetConnections**

| Nombre del campo | Tipo | Lista de valores |
| ---------- | ---- | -------------- |
| **internalAddress** | String | Dirección IP |
| **autorizado** | Boolean | Sí o no | 
| **externalAddresses** | Matriz JSON | Dirección IP |

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
{

    "unauthorizedDevices": [
    
        {
        
            "address": "10.2.1.14",
            
            "name": "PLC \#14",
            
            "firstDetectionTime": 1462645483000,
            
            "lastSeen": 1462645495000,
        
        }
    
    ],
    
    "redundantFirewallRules": [
    
        {
        
            "sources": "170.39.3.0/255.255.255.0",
            
            "destinations": "Any",
            
            "ports": "102"
        
        }
    
    ],
    
    "connectionsBetweenSubnets": [
    
        {
        
            "server": "10.2.1.22",
            
            "client": "170.39.2.0"
        
        }
    
    ],
    
    "industrialMalwareIndications": [
    
        {
        
            "detectionTime": 1462645483000,
            
            "alertMessage": "Suspicion of Malicious Activity (Regin)",
            
            "description": "Suspicious network activity was detected. Such behavior might be attributed to the Regin malware.",
            
            "addresses": [
            
                "10.2.1.4",
                
                "10.2.1.5"
            
            ]
        
        }
    
    ],
    
    "illegalTrafficByFirewallRules": [
    
        {
        
            "server": "10.2.1.7",
            
            "port": "20000",
            
            "client": "10.2.1.4",
            
            "transport": "TCP"
        
        },
    
        {
        
            "server": "10.2.1.8",
            
            "port": "20000",
            
            "client": "10.2.1.4",
            
            "transport": "TCP"
        
        },
    
        {
        
            "server": "10.2.1.9",
            
            "port": "20000",
            
            "client": "10.2.1.4",
            
            "transport": "TCP"
        
        }
    
    ],
    
    "internetConnections": [
    
        {
        
            "internalAddress": "10.2.1.1",
            
            "authorized": "Yes",
            
            "externalAddresses": ["10.2.1.2",”10.2.1.3”]
        
        }
    
    ],
    
    "accessPoints": [
    
        {
        
            "macAddress": "ec:08:6b:0f:1e:22",
            
            "vendor": "TP-LINK TECHNOLOGIES",
            
            "ipAddress": "173.194.112.22",
            
            "name": "Enterprise AP",
            
            "wireless": "Yes"
        
        }
    
    ],
    
    "weakFirewallRules": [
    
        {
        
            "sources": "170.39.3.0/255.255.255.0",
            
            "destinations": "Any",
            
            "ports": "102"
        
        }
    
    ]

}

```

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| GET | curl -k -H "Authorization: <TOKEN_AUT>" https://<DIRECCIÓN_IP>/api/v1/reports/vulnerabilities/security | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/<span>/127.0.0.1/api/v1/reports/vulnerabilities/security |

### <a name="retrieve-operational-vulnerabilities---apiv1reportsvulnerabilitiesoperational"></a>Recuperación de vulnerabilidades operativas: /api/v1/reports/vulnerabilities/operational

Use esta API para solicitar los resultados de una evaluación de vulnerabilidades general. Esta evaluación ofrece conclusiones sobre el estado operativo de la red. Se basa en la información general del sistema y de la red, y no en la valoración de un dispositivo específico.

#### <a name="method"></a>Método

**GET**

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Objeto JSON que representa a los resultados evaluados. Cada clave contiene una matriz JSON de resultados.

#### <a name="result-fields"></a>Campos de resultado

**Claves**

**backupServer**

| Nombre del campo | Tipo | Lista de valores |
|--|--|--|
| **de origen** | String | Dirección IP |
| **destination** | String | Dirección IP |
| **port** | Numeric | - |
| **transport** | String | TCP o UDP |
| **backupMaximalInterval** | String | - |
| **lastSeenBackup** | Numeric | Tiempo (hora UTC) |

**ipNetworks**

| Nombre del campo | Tipo | Lista de valores |
|--|--|--|
| **addresses** | Numeric | - |
| **network** | String | Dirección IP |
| **mask** | String | Máscara de subred |

**protocolProblems**

| Nombre del campo | Tipo | Lista de valores |
|--|--|--|
| **protocolo** | String | - |
| **addresses** | Matriz JSON | Direcciones IP |
| **alert** | String | - |
| **reportTime** | Numeric | Tiempo (hora UTC) |

**protocolDataVolumes**

| Nombre del campo | Tipo | Lista de valores |
|--|--|--|
| protocol | String | - |
| volumen | String | "MB de número de volumen" |

**disconnections**

| Nombre del campo | Tipo | Lista de valores |
|--|--|--|
| **assetAddress** | String | Dirección IP |
| **assetName** | String | - |
| **lastDetectionTime** | Numeric | Tiempo (hora UTC) |
| **backToNormalTime** | Numeric | Tiempo (hora UTC) |     

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
{

    "backupServer": [
    
        {
        
            "backupMaximalInterval": "1 Hour, 29 Minutes",
            
            "source": "10.2.1.22",
            
            "destination": "170.39.2.14",
            
            "port": 10000,
            
            "transport": "TCP",
            
            "lastSeenBackup": 1462645483000
        
        }
    
    ],

    "ipNetworks": [
    
        {
        
            "addresses": "21",
            
            "network": "10.2.1.0",
            
            "mask": "255.255.255.0"
        
        },
        
        {
        
            "addresses": "3",
            
            "network": "170.39.2.0",
            
            "mask": "255.255.255.0"
        
        }
    
    ],

    "protocolProblems": [
    
        {
        
            "protocol": "DNP3",
            
            "addresses": [
            
                "10.2.1.7",
                
                "10.2.1.8"
            
            ],
            
            "alert": "Illegal DNP3 Operation",
            
            "reportTime": 1462645483000
        
        },
        
        {
        
            "protocol": "DNP3",
            
            "addresses": [
            
                "10.2.1.15"
            
            ],
            
            "alert": "Master Requested an Application Layer Confirmation",
            
            "reportTime": 1462645483000
        
        }
        
    ],

    "protocolDataVolumes": [
    
        {
        
            "protocol": "MODBUS (502)",
            
            "volume": "21.07 MB"
        
        },
        
        {
        
            "protocol": "SSH (22)",
            
            "volumn": "0.001 MB"
        
        }
    
    ],
    
    "disconnections": [
    
        {
        
            "assetAddress": "10.2.1.3",
            
            "assetName": "PLC \#3",
            
            "lastDetectionTime": 1462645483000,
            
            "backToNormalTime": 1462645484000
        
        }
    
    ]

}

```

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| GET | curl -k -H "Authorization: <TOKEN_AUT>" https://<DIRECCIÓN_IP>/api/v1/reports/vulnerabilities/operational | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/<span>/127.0.0.1/api/v1/reports/vulnerabilities/operational |

### <a name="validate-user-credentials---apiexternalauthenticationvalidation"></a>Validación de credenciales de usuario: /api/external/authentication/validation

Use esta API para validar el nombre de usuario y contraseña de Defender para IoT. Todos los roles de usuario de Defender para IoT pueden funcionar con la API.

No necesita un token de acceso de Defender para IoT para usar esta API.

#### <a name="method"></a>Método

**POST**

#### <a name="request-type"></a>Tipo de solicitud

**JSON**

#### <a name="query-parameters"></a>Parámetros de consulta

| **Nombre** | **Tipo** | **Admisión de valores NULL** |
|--|--|--|
| **username** | String | No |
| **password** | String | No |

#### <a name="request-example"></a>Ejemplo de solicitud

```rest
request:

{

    "username": "test",
    
    "password": "Test12345\!"

}

```

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Cadena de mensaje con los detalles del estado de la operación:

- **Correcta - msg**: autenticación correcta

- **Error - error**: error de validación de credenciales

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
response:

{

    "msg": "Authentication succeeded."

}

```

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| GET | curl -k -H "Authorization: <TOKEN_AUT>" https://<DIRECCIÓN_IP>/api/external/authentication/validation | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/<span>/127.0.0.1/api/external/authentication/validation |

### <a name="change-password---externalauthenticationset_password"></a>Cambio de contraseña: /external/authentication/set_password

Use esta API para que los usuarios puedan cambiar sus contraseñas. Todos los roles de usuario de Defender para IoT pueden funcionar con la API. No necesita un token de acceso de Defender para IoT para usar esta API.

#### <a name="method"></a>Método

**POST**

#### <a name="request-type"></a>Tipo de solicitud

**JSON**

#### <a name="request-example"></a>Ejemplo de solicitud

```rest
request:

{

    "username": "test",
    
    "password": "Test12345\!",
    
    "new_password": "Test54321\!"

}

```

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Cadena de mensaje con los detalles del estado de la operación:

- **Correcta – msg**: se ha reemplazado la contraseña

- **Error - error**: error de autenticación del usuario

- **Error - error**: la contraseña no coincide con la directiva de seguridad

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
response:

{

    "error": {
    
        "userDisplayErrorMessage": "User authentication failure"
    
    }

}

```

#### <a name="device-fields"></a>Campos de dispositivo

| **Nombre** | **Tipo** | **Admisión de valores NULL** |
|--|--|--|
| **username** | String | No |
| **password** | String | No |
| **new_password** | String | No |

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| POST | curl -k -d '{"username": "<NOMBRE_USUARIO>","password": "<CONTRASEÑA_ACTUAL>","new_password": "<NUEVA_CONTRASEÑA>"}' -H 'Content-Type: application/json'  https://<DIRECCIÓN_IP>/api/external/authentication/set_password | curl -k -d '{"username": "myUser","password": "1234@abcd","new_password": "abcd@1234"}' -H 'Content-Type: application/json'  https:/<span>/127.0.0.1/api/external/authentication/set_password |

### <a name="user-password-update-by-system-admin---externalauthenticationset_password_by_admin"></a>Actualización de la contraseña de usuario por el administrador del sistema: /external/authentication/set_password_by_admin

Use esta API para que los administradores del sistema puedan cambiar las contraseñas de usuarios especificados. Los roles de usuario administrador de Defender para IoT pueden funcionar con la API. No necesita un token de acceso de Defender para IoT para usar esta API.

#### <a name="method"></a>Método

**POST**

#### <a name="request-type"></a>Tipo de solicitud

**JSON**

#### <a name="request-example"></a>Ejemplo de solicitud

```rest
request:

{

    "username": "test",
    
    "password": "Test12345\!",
    
    "new_password": "Test54321\!"

}
```

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Cadena de mensaje con los detalles del estado de la operación:

- **Correcta – msg**: se ha reemplazado la contraseña

- **Error - error**: error de autenticación del usuario

- **Error - error**: el usuario no existe

- **Error - error**: la contraseña no coincide con la directiva de seguridad

- **Error - error**: el usuario no tiene los permisos para cambiar la contraseña

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
response:

{

    "error": {
    
        "userDisplayErrorMessage": "The user 'test_user' doesn't exist",
        
        "internalSystemErrorMessage": "The user 'yoavfe' doesn't exist"
    
    }

}

```

#### <a name="device-fields"></a>Campos de dispositivo

| **Nombre** | **Tipo** | **Admisión de valores NULL** |
|--|--|--|
| **admin_username** | String | No |
| **admin_password** | String | No |
| **username** | String | No |
| **new_password** | String | No |

#### <a name="curl-command"></a>Comando Curl

> [!div class="mx-tdBreakAll"]
> | Tipo | API existentes | Ejemplo |
> |--|--|--|
> | POST | curl -k -d '{"admin_username":"<NOMBRE_USUARIO_ADMINISTRADOR>","admin_password":"<CONTRASEÑA_ADMINISTRADOS>","username": "<NOMBRE_USUARIO>","new_password": "<NUEVA_CONTRASEÑA>"}' -H 'Content-Type: application/json'  https://<DIRECCIÓN_IP>/api/external/authentication/set_password_by_admin | curl -k -d '{"admin_user":"adminUser","admin_password": "1234@abcd","username": "myUser","new_password": "abcd@1234"}' -H 'Content-Type: application/json'  https:/<span>/127.0.0.1/api/external/authentication/set_password_by_admin |

## <a name="on-premises-management-console-api-specifications"></a>Especificaciones de la API de la consola de administración local ##

En esta sección se describen las API de la consola de administración local para:
- Exclusiones de alertas
- Información del dispositivo
- Información de alertas

### <a name="alert-exclusions"></a>Exclusiones de alertas ###

Defina las condiciones en las que no se enviarán alertas. Por ejemplo, defina y actualice las horas de inicio y detención, los dispositivos o subredes que se deben excluir al desencadenar las alertas o los motores de Defender para IoT que deben excluirse. Por ejemplo, durante una ventana de mantenimiento, quizá quiera detener la entrega de todas las alertas, salvo las alertas de malware en dispositivos críticos. Los elementos que defina aquí aparecen en la ventana **Exclusiones de alertas** de la consola de administración local como reglas de exclusión de solo lectura.

#### <a name="externalv1maintenancewindow"></a>/external/v1/maintenanceWindow

- **/external/authentication/validation**

- **Ejemplo de respuesta**

- **respuesta:**

```rest
{
    "msg": "Authentication succeeded."
}

```

#### <a name="change-password---externalauthenticationset_password"></a>Cambio de contraseña: /external/authentication/set_password 

Use esta API para que los usuarios puedan cambiar sus contraseñas. Todos los roles de usuario de Defender para IoT pueden funcionar con la API. No necesita un token de acceso de Defender para IoT para usar esta API.

#### <a name="user-password-update-by-system-admin---externalauthenticationset_password_by_admin"></a>Actualización de la contraseña de usuario por el administrador del sistema: /external/authentication/set_password_by_admin 

Use esta API para que los administradores del sistema puedan cambiar las contraseñas de usuarios específicos. Los roles de usuario administrador de Defender para IoT pueden funcionar con la API. No necesita un token de acceso de Defender para IoT para usar esta API.

### <a name="retrieve-device-information---externalv1devices"></a>Recuperación de información del dispositivo: /external/v1/devices ###

Esta API solicita una lista de todos los dispositivos detectados por los sensores de Defender para IoT que están conectados a una consola de administración local.

#### <a name="method"></a>Método

**GET**

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Matriz de objetos JSON que representan a dispositivos.

#### <a name="query-parameters"></a>Parámetros de consulta

- **authorized**: para filtrar solo los dispositivos autorizados y no autorizados.

- **siteId**: para filtrar solo los dispositivos relacionados con sitios específicos.

- **zoneId**: para filtrar solo los dispositivos relacionados con zonas específicas. [1](#1)

- **sensorId**: para filtrar solo los dispositivos detectados por sensores específicos. [1](#1)

###### <a name="you-might-not-have-the-site-and-zone-id-if-this-is-the-case-query-all-devices-to-retrieve-the-site-and-zone-id"></a><a id="1">1</a> *Es posible que no tenga el id. de zona y de sitio. En este caso, consulte a todos los dispositivos para recuperar el id. de sitio y de zona.*

#### <a name="query-parameters-example"></a>Ejemplo de parámetros de consulta

`/external/v1/devices?authorized=true`

`/external/v1/devices?authorized=false`

`/external/v1/devices?siteId=1,2`

`/external/v1/devices?zoneId=5,6`

`/external/v1/devices?sensorId=8`

#### <a name="device-fields"></a>Campos de dispositivo

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **sensorId** | Numeric | No | - |
| **zoneId** | Numeric | Sí | - |
| **siteId** | Numeric | Sí | - |
| **ipAddresses** | Matriz JSON | Sí | Direcciones IP (puede ser más de una dirección en el caso de direcciones de Internet o de un dispositivo con dos adaptadores de red) |
| **name** | String | No | - |
| **type** | String | No | Desconocido, estación de ingeniería, PLC, HMI, analista, controlador de dominio, servidor de base de servidores, punto de acceso inalámbrico, enrutador, conmutador, servidor, estación de trabajo, cámara IP, impresora, firewall, estación de la terminal, VPN Gateway, Internet o multidifusión y difusión |
| **macAddresses** | Matriz JSON | Sí | Direcciones MAC (puede ser más de una dirección en el caso de un dispositivo con dos adaptadores de red) |
| **operatingSystem** | String | Sí | - |
| **engineeringStation** | Boolean | No | Verdadero o falso |
| **scanner** | Boolean | No | Verdadero o falso |
| **autorizado** | Boolean | No | Verdadero o falso |
| **vendor** | String | Sí | - |
| **Protocolos** | Matriz JSON | Sí | Objeto Protocol |
| **firmware** | Matriz JSON | Sí | Objeto Firmware |

#### <a name="protocol-fields"></a>Campos del protocolo

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| Nombre | String | No | - |
| Direcciones | Matriz JSON | Sí | Patrón o valores numéricos |

#### <a name="firmware-fields"></a>Campos de firmware

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **serial** | String | No | N/D o el valor real |
| **model** | String | No | N/D o el valor real |
| **firmwareVersion** | Double | No | N/D o el valor real |
| **additionalData** | String | No | N/D o el valor real |
| **moduleAddress** | String | No | N/D o el valor real |
| **rack** | String | No | N/D o el valor real |
| **slot** | String | No | N/D o el valor real |
| **address** | String | No | N/D o el valor real |

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
[

    {
    
    "sensorId": 7,
    
    "zoneId": 1,
    
    "siteId": 1,
    
    "vendor": null,
    
    "name": "10.4.14.102",
    
    "firmware": [
    
    {
    
        "slot": "N/A",
        
        "additionalData": "N/A",
        
        "moduleAddress": "Network: Local network (0), Node: 0, Unit: CPU (0x0)",
        
        "rack": "N/A",
        
        "address": "10.4.14.102",
        
        "model": "AAAAAAAAAA",
        
        "serial": "N/A",
        
        "firmwareVersion": "20.55"
    
    },
    
    {
    
        "slot": "N/A",
        
        "additionalData": "N/A",
        
        "moduleAddress": "Network: Local network (0), Node: 0, Unit: Unknown (0x3)",
        
        "rack": "N/A",
        
        "address": "10.4.14.102",
        
        "model": "AAAAAAAAAAAAAAAAAAAA",
        
        "serial": "N/A",
        
        "firmwareVersion": "20.55"
    
    },
    
    {
    
        "slot": "N/A",
        
        "additionalData": "N/A",
        
        "moduleAddress": "Network: Local network (0), Node: 3, Unit: CPU (0x0)",
        
        "rack": "N/A",
        
        "address": "10.4.14.102",
        
        "model": "AAAAAAAAAAAAAAAAAAAA",
        
        "serial": "N/A",
        
        "firmwareVersion": "20.55"
    
    },
    
    {
    
        "slot": "N/A",
        
        "additionalData": "N/A",
        
        "moduleAddress": "Network: 3, Node: 0, Unit: CPU (0x0)",
        
        "rack": "N/A",
        
        "address": "10.4.14.102",
        
        "model": "AAAAAAAAAAAAAAAAAAAA",
        
        "serial": "N/A",
        
        "firmwareVersion": "20.55"
    
    }

],

"id": 79,

"macAddresses": null,

"authorized": true,

"ipAddresses": [

    "10.4.14.102"

],

"engineeringStation": false,

"type": "PLC",

"operatingSystem": null,

"protocols": [

    {
    
        "addresses": [],
        
        "id": 62,
        
        "name": "Omron FINS"
    
    }

],

"scanner": false

}

]
```

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| GET | curl -k -H "Authorization: <TOKEN_AUT>" 'https://<>DIRECCIÓN_IP>/external/v1/devices?siteId=&zoneId=&sensorId=&authorized=' | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" 'https:/<span>/127.0.0.1/external/v1/devices?siteId=1&zoneId=2&sensorId=5&authorized=true' |

### <a name="retrieve-alert-information---externalv1alerts"></a>Recuperación de información de alertas: /external/v1/alerts

Use esta API para recuperar todas las alertas o las alertas filtradas de una consola de administración local.

#### <a name="method"></a>Método

**GET**

#### <a name="query-parameters"></a>Parámetros de consulta

- **state**: para filtrar solo las alertas controladas y no controladas.

  **Ejemplo**:

  `/api/v1/alerts?state=handled`

- **fromTime**: para filtrar las alertas creadas a partir de una hora específica (en milisegundos, hora UTC).

  **Ejemplo**:

  `/api/v1/alerts?fromTime=<epoch>`

- **toTime**: para filtrar solo las alertas creadas antes de una hora específica (en milisegundos, hora UTC).

  **Ejemplo**:

  `/api/v1/alerts?toTime=<epoch>`

- **siteId**: el sitio en el que se detectó la alerta.
- **zoneId**: la zona en la que se detectó la alerta.
- **sensor**: el sensor en el que se detectó la alerta.

*Es posible que no tenga el identificador de zona y de sitio. En este caso, consulte todos los dispositivos para recuperarlo.*

#### <a name="alert-fields"></a>Campos de alerta 

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **Id** | Numeric | No | - |
| **time** | Numeric | No | Tiempo (hora UTC) |
| **title** | String | No | - |
| **message** | String | No | - |
| **severity** | String | No | Advertencia, leve, grave o crítica |
| **engine** | String | No | Infracción del protocolo, infracción de la directiva, malware, anomalía u operativa |
| **sourceDevice** | Numeric | Sí | Id. de dispositivo |
| **destinationDevice** | Numeric | Sí | Id. de dispositivo |
| **sourceDeviceAddress** | Numeric | Sí | IP, MAC, Null |
| **destinationDeviceAddress** | Numeric | Sí | IP, MAC, Null |
| **remediationSteps** | String | Sí | Pasos para la corrección que se muestran en la alerta|
| **sensorName** | String | Sí | Nombre del sensor que el usuario define en la consola|
|**zoneName** | String | Sí | Nombre de la zona asociada al sensor en la consola|
| **siteName** | String | Sí | Nombre del sitio asociado al sensor en la consola |
| **additionalInformation** | Objeto Additional information | Sí | - |

Tenga en cuenta que se requiere /api/v2/ para la información siguiente:

- sourceDeviceAddress 
- destinationDeviceAddress
- remediationSteps
- sensorName
- zoneName
- siteName

#### <a name="additional-information-fields"></a>Campos de información adicional

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **description** | String | No | - |
| **information** | Matriz JSON | No | String |

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
[

    {
    
        "engine": "Operational",
        
        "handled": false,
        
        "title": "Traffic Detected on sensor Interface",
        
        "additionalInformation": null,
        
        "sourceDevice": 0,
        
        "zoneId": 1,
        
        "siteId": 1,
        
        "time": 1594808245000,
        
        "sensorId": 1,
        
        "message": "The sensor resumed detecting network traffic on ens224.",
        
        "destinationDevice": 0,
        
        "id": 1,
        
        "severity": "Warning"
    
    },
    
    {
    
        "engine": "Anomaly",
        
        "handled": false,
        
        "title": "Address Scan Detected",
        
        "additionalInformation": null,
        
        "sourceDevice": 4,
        
        "zoneId": 1,
        
        "siteId": 1,
        
        "time": 1594808260000,
        
        "sensorId": 1,
        
        "message": "Address scan detected.\nScanning address: 10.10.10.22\nScanned subnet: 10.11.0.0/16\nScanned addresses: 10.11.1.1, 10.11.20.1, 10.11.20.10, 10.11.20.100, 10.11.20.2, 10.11.20.3, 10.11.20.4, 10.11.20.5, 10.11.20.6, 10.11.20.7...\nIt is recommended to notify the security officer of the incident.",
        
        "destinationDevice": 0,
        
        "id": 2,
        
        "severity": "Critical"
    
    },
    
    {
    
        "engine": "Operational",
        
        "handled": false,
        
        "title": "Suspicion of Unresponsive MODBUS Device",
        
        "additionalInformation": null,
        
        "sourceDevice": 194,
        
        "zoneId": 1,
        
        "siteId": 1,
        
        "time": 1594808285000,
        
        "sensorId": 1,
        
        "message": "Outstation device 10.13.10.1 (Protocol Address 53) seems to be unresponsive to MODBUS requests.",
        
        "destinationDevice": 0,
        
        "id": 3,
        
        "severity": "Minor"
    
    }
  
]
```

#### <a name="curl-command"></a>Comando Curl

> [!div class="mx-tdBreakAll"]
> | Tipo | API existentes | Ejemplo |
> |--|--|--|
> | GET | curl -k -H "Authorization: <TOKEN_AUT>" 'https://<>DIRECCIÓN_IP>/external/v1/alerts?state=&zoneId=&fromTime=&toTime=&siteId=&sensor=' | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" 'https:/<span>/127.0.0.1/external/v1/alerts?state=unhandled&zoneId=1&fromTime=0&toTime=1594551777000&siteId=1&sensor=1' |

### <a name="qradar-alerts"></a>Alertas de QRadar

La integración de QRadar con Defender para IoT le ayuda a identificar las alertas generadas por Defender para IoT y a realizar acciones con estas alertas. QRadar recibe los datos de Defender para IoT y luego se pone en contacto con el componente de la consola de administración local de la API pública.

Para enviar los datos detectados por Defender para IoT a QRadar, defina una regla de reenvío en el sistema de Defender para IoT y seleccione la opción **Remote support alert handling** (Control de alertas de soporte remoto).

:::image type="content" source="media/references-work-with-defender-for-iot-apis/edit-forwarding-rules.png" alt-text="Edite las reglas de reenvío para que adaptarlas a sus necesidades.":::

Al seleccionar esta opción durante el proceso de configuración de las reglas de reenvío, se muestran los siguientes campos adicionales en QRadar:

- **UUID**: identificador único de la alerta, como 1-1555245116250.

- **Sitio**: el sitio en el que se detectó la alerta.

- **Zone**: la zona en la que se detectó la alerta.

Ejemplo de la carga útil que se envía a QRadar:

```
<9>May 5 12:29:23 sensor_Agent LEEF:1.0|CyberX|CyberX platform|2.5.0|CyberX platform Alert|devTime=May 05 2019 15:28:54 devTimeFormat=MMM dd yyyy HH:mm:ss sev=2 cat=XSense Alerts title=Device is Suspected to be Disconnected (Unresponsive) score=81 reporter=192.168.219.50 rta=0 alertId=6 engine=Operational senderName=sensor Agent UUID=5-1557059334000 site=Site zone=Zone actions=handle dst=192.168.2.2 dstName=192.168.2.2 msg=Device 192.168.2.2 is suspected to be disconnected (unresponsive).
```

#### <a name="externalv1alertsltuuidgt"></a>/external/v1/alerts/&lt;UUID&gt;

#### <a name="method"></a>Método

**PUT**

#### <a name="request-type"></a>Tipo de solicitud

**JSON**

#### <a name="request-content"></a>Contenido de la solicitud

Objeto JSON que representa a la acción que se realizará en la alerta que contiene el UUID.

#### <a name="action-fields"></a>Campos de acción

| Nombre | Tipo | Nullable | Lista de valores |
|--|--|--|--|
| **action** | String | No | handle o handleAndLearn |

#### <a name="request-example"></a>Ejemplo de solicitud

```rest
{
    "action": "handle"
}

```

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Matriz de objetos JSON que representan a dispositivos.

#### <a name="response-fields"></a>Campos de respuesta


| Nombre | Tipo | Nullable | Descripción |
|--|--|--|--|
| **content / error** | String | No | Si la solicitud se realiza correctamente, aparece la propiedad content. De lo contrario, aparece la propiedad error. |

#### <a name="possible-content-values"></a>Posibles valores de content

| status code | Valor de content | Descripción |
|--|--|--|
| 200 | La solicitud de actualización de alertas finalizó correctamente. | La solicitud de actualización finalizó correctamente. Sin comentarios. |
| 200 | Ya se había controlado la alerta (**handle**). | La alerta ya se había controlado cuando se recibió una solicitud de control para la alerta.<br />La alerta sigue **controlada**. |
| 200 | La alerta ya se había controlado y aprendido (**handleAndLearn**). | La alerta ya se había controlado y aprendido cuando se recibió una solicitud para **handleAndLearn**.<br />La alerta permanece en el estado **handledAndLearn**. |
| 200 | Ya se había controlado la alerta (**controlada**).<br />Se realizó el control y aprendizaje (**handleAndLearn**) en la alerta. | La alerta ya se había controlado cuando se recibió una solicitud para **handleAndLearn**.<br />La alerta se convierte en **handleAndLearn**. |
| 200 | La alerta ya se había controlado y aprendido (**handleAndLearn**). Solicitud de control omitida. | La alerta ya se encontraba en el estado **handleAndLearn** cuando se recibió una solicitud para controlarla. La alerta sigue en el estado **handleAndLearn**. |
| 500 | Acción no válida. | La acción enviada no es una acción válida que pueda realizarse en la alerta. |
| 500 | Se ha producido un error inesperado. | Se ha producido un error inesperado. Para resolver el problema, póngase en contacto con el soporte técnico. |
| 500 | No se pudo ejecutar la solicitud porque no se encontró ninguna alerta con este UUID. | No se encontró el UUID de alerta especificado en el sistema. |

#### <a name="response-example"></a>Ejemplo de respuesta

**Correcto**

```rest
{
    "content": "Alert update request finished successfully"
}
```

**Incorrecto**

```rest
{
    "error": "Invalid action"
}
```

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| PUT | curl -k -X PUT -d '{"action": "<ACTION>"}' -H "Authorization: <TOKEN_AUT>" https://<DIRECCIÓN_IP>/external/v1/alerts/<UUID> | curl -k -X PUT -d '{"action": "handle"}' -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/<span>/127.0.0.1/external/v1/alerts/1-1594550943000 |

### <a name="alert-exclusions-maintenance-window---externalv1maintenancewindow"></a>Exclusiones de alerta (ventana de mantenimiento): /external/v1/maintenanceWindow

Defina las condiciones en las que no se enviarán alertas. Por ejemplo, defina y actualice las horas de inicio y detención, los dispositivos o subredes que se deben excluir al desencadenar las alertas o los motores de Defender para IoT que deben excluirse. Por ejemplo, durante una ventana de mantenimiento, quizá quiera detener la entrega de alertas para todas las alertas, salvo las alertas de malware en dispositivos críticos.

Las API que defina aquí aparecen en la ventana **Exclusiones de alerta** de la consola de administración local como una regla de exclusión de solo lectura.

:::image type="content" source="media/references-work-with-defender-for-iot-apis/alert-exclusion-window.png" alt-text="La ventana Exclusiones de alerta, que muestra una lista de todas las reglas de exclusión. ":::

#### <a name="method---post"></a>Método - POST

#### <a name="query-parameters"></a>Parámetros de consulta

- **ticketId**: define el id. del vale de mantenimiento en los sistemas del usuario.

- **ttl**: define el TTL (período de vida), que es la duración de la ventana de mantenimiento en minutos. Una vez transcurrido el período de tiempo que define este parámetro, el sistema inicia automáticamente el envío de alertas.

- **engines**: define en qué motor de seguridad se van a suprimir las alertas durante el proceso de mantenimiento:

   - ANOMALY

   - MALWARE

   - OPERATIONAL

   - POLICY_VIOLATION

   - PROTOCOL_VIOLATION

- **sensorIds**: define en qué sensor de Defender para IoT se van a suprimir las alertas durante el proceso de mantenimiento. Es el mismo id. recuperado de /api/v1/appliances (GET).

- **subnets**: define en qué subred se van a suprimir las alertas durante el proceso de mantenimiento. La subred se envía en el siguiente formato: 192.168.0.0/16.

#### <a name="error-codes"></a>Códigos de error

- **201 (creado)** : la acción se ha completado correctamente.

- **400 (solicitud errónea)** : aparece en los siguientes casos:

   - El parámetro **ttl** no es un número o no es positivo.

   - El parámetro **subnets** se definió con un formato incorrecto.

   - Falta el parámetro **ticketId**.

   - El parámetro **engine** no coincide con los motores de seguridad existentes.

- **404 (no encontrado)** : uno de los sensores no existe.

- **409 (conflicto)** : el id. de vale está vinculado a otra ventana de mantenimiento abierta.

- **500 (error interno del servidor)** : cualquier otro error inesperado.

> [!NOTE]
> Asegúrese de que el id. de vale no está vinculado a una ventana abierta existente. Se genera la siguiente regla de exclusión: Maintenance-{nombre de token}-{id. de vale}.

#### <a name="method---put"></a>Método - PUT

Permite actualizar la duración de la ventana de mantenimiento después de iniciar el proceso de mantenimiento al cambiar el parámetro **ttl**. La nueva definición de duración invalida a la anterior.

Este método resulta útil cuando se quiere establecer una duración más larga que la duración configurada actualmente.

#### <a name="query-parameters"></a>Parámetros de consulta

- **ticketId**: define el id. del vale de mantenimiento en los sistemas del usuario.

- **ttl**: Define la duración de la ventana en minutos.

#### <a name="error-code"></a>Código de error

- **200 (correcto)** : la acción se ha completado correctamente.

- **400 (solicitud errónea)** : aparece en los siguientes casos:

   - El parámetro **ttl** no es un número o no es positivo.

   - Falta el parámetro **ticketId**.

   - Falta el parámetro **ttl**.

- **404 (no encontrado)** : el id. de vale no está vinculado a una ventana de mantenimiento abierta.

- **500 (error interno del servidor)** : cualquier otro error inesperado.

> [!NOTE]
> Asegúrese de que el id. de vale esté vinculado a una ventana abierta existente.

#### <a name="method---delete"></a>Método - DELETE

Cierra una ventana de mantenimiento existente.

#### <a name="query-parameters"></a>Parámetros de consulta

- **ticketId**: registra el id. del vale de mantenimiento en los sistemas del usuario.

#### <a name="error-code"></a>Código de error

- **200 (correcto)** : la acción se ha completado correctamente.

- **400 (solicitud errónea)** : falta el parámetro **ticketId**.

- **404 (no encontrado)** : el id. de vale no está vinculado a una ventana de mantenimiento abierta.

- **500 (error interno del servidor)** : cualquier otro error inesperado.

> [!NOTE]
> Asegúrese de que el id. de vale esté vinculado a una ventana abierta existente.

#### <a name="method---get"></a>Método - GET

Recupere un registro de todas las acciones de apertura, cierre y actualización que se realizaron en el sistema durante el mantenimiento. Puede recuperar un registro solo de las ventanas de mantenimiento que estuvieron activas en el pasado y que se han cerrado.

#### <a name="query-parameters"></a>Parámetros de consulta

- **fromDate**: filtra los registros de la fecha predefinida y posteriores. El formato es 2019-12-30.

- **toDate**: filtra los registros hasta la fecha predefinida. El formato es 2019-12-30.

- **ticketId**: filtra los registros relacionados con un id. de vale específico.

- **tokenName**: filtra los registros relacionados con un token específico.

#### <a name="error-code"></a>Código de error 

- **200 (correcto)** : la acción se ha completado correctamente.

- **400 (solicitud errónea)** : El formato de fecha es incorrecto.

- **204 (sin contenido)** : no hay ningún dato que mostrar.

- **500 (error interno del servidor)** : cualquier otro error inesperado.

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Matriz de objetos JSON que representan a las operaciones de la ventana de mantenimiento.

#### <a name="response-structure"></a>Estructura de la respuesta

| Nombre | Tipo | Comentario | Nullable |
|--|--|--|--|
| **dateTime** | String | Ejemplo: "2012-04-23T18:25:43.511Z" | no |
| **ticketId** | String | Ejemplo: "9a5fe99c-d914-4bda-9332-307384fe40bf" | no |
| **tokenName** | String | - | no |
| **engines** | Matriz de cadena | - | sí |
| **sensorIds** | Matriz de cadena | - | sí |
| **subredes** | Matriz de cadena | - | sí |
| **ttl** | Numeric | - | sí |
| **operationType** | String | Los valores son "OPEN", "UPDATE" y "CLOSE". | no |

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| POST | curl -k -X POST -d '{"ticketId": "<ID_VALE>",ttl": <TIME_TO_LIVE>,"engines": [<MOTOR1, MOTOR2...MOTORn>],"sensorIds": [<ID_SENSOR1, ID_SENSOR2...ID_SENSORn>],"subnets": [<SUBRED1, SUBRED2...SUBREDn>]}' -H "Authorization: <AUTH_TOKEN>" https:/<span>/127.0.0.1/external/v1/maintenanceWindow | curl -k -X POST -d '{"ticketId": "a5fe99c-d914-4bda-9332-307384fe40bf","ttl": "20","engines": ["ANOMALY"],"sensorIds": ["5","3"],"subnets": ["10.0.0.3"]}' -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/<span>/127.0.0.1/external/v1/maintenanceWindow |
| PUT | curl -k -X PUT -d '{"ticketId": "<ID_VALE>",ttl": "<PERÍODO_DE_VIDA>"}' -H "Authorization: <TOKEN_AUT>" https:/<span>/127.0.0.1/external/v1/maintenanceWindow | curl -k -X PUT -d '{"ticketId": "a5fe99c-d914-4bda-9332-307384fe40bf","ttl": "20"}' -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/<span>/127.0.0.1/external/v1/maintenanceWindow |
| DELETE | curl -k -X DELETE -d '{"ticketId": "<ID_VALE>"}' -H "Authorization: <TOKEN_AUT>" https:/<span>/127.0.0.1/external/v1/maintenanceWindow | curl -k -X DELETE -d '{"ticketId": "a5fe99c-d914-4bda-9332-307384fe40bf"}' -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" https:/<span>/127.0.0.1/external/v1/maintenanceWindow |
| GET | curl -k -H "Authorization: <TOKEN_AUT>" 'https://<DIRECCIÓN_IP>/external/v1/maintenanceWindow?fromDate=&toDate=&ticketId=&tokenName=' | curl -k -H "Authorization: 1234b734a9244d54ab8d40aedddcabcd" 'https:/<span>/127.0.0.1/external/v1/maintenanceWindow?fromDate=2020-01-01&toDate=2020-07-14&ticketId=a5fe99c-d914-4bda-9332-307384fe40bf&tokenName=a' |

### <a name="authenticate-user-credentials---externalauthenticationvalidation"></a>Autenticación de credenciales de usuario: /external/authentication/validation

Use esta API para validar las credenciales de usuario. Todos los roles de usuario de Defender para IoT pueden funcionar con la API. No necesita un token de acceso de Defender para IoT para usar esta API.

#### <a name="method"></a>Método

**POST**

#### <a name="request-type"></a>Tipo de solicitud

**JSON**

#### <a name="request-example"></a>Ejemplo de solicitud

```rest
request:

{

    "username": "test",

    "password": "Test12345\!"

}
```

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Cadena de mensaje con los detalles del estado de la operación:

- **Correcta – msg**: autenticación correcta

- **Error - error**: error de validación de credenciales

#### <a name="device-fields"></a>Campos de dispositivo

| **Nombre** | **Tipo** | **Admisión de valores NULL** |
|--|--|--|
| **username** | String | No |
| **password** | String | No |

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
response:

{

    "msg": "Authentication succeeded."

}
```

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| POST | curl -k -d '{"username":"<NOMBRE_USUARIO>","password":"CONTRASEÑA"}' 'https://<DIRECCIÓN_IP>/external/authentication/validation' | curl -k -d '{"username":"myUser","password":"1234@abcd"}' 'https:/<span>/127.0.0.1/external/authentication/validation' |

### <a name="change-password---externalauthenticationset_password"></a>Cambio de contraseña: /external/authentication/set_password

Use esta API para que los usuarios puedan cambiar sus contraseñas. Todos los roles de usuario de Defender para IoT pueden funcionar con la API. No necesita un token de acceso de Defender para IoT para usar esta API.

#### <a name="method"></a>Método

**POST**

#### <a name="request-type"></a>Tipo de solicitud

**JSON**

#### <a name="request-example"></a>Ejemplo de solicitud

```rest
request:

{

    "username": "test",
    
    "password": "Test12345\!",
    
    "new_password": "Test54321\!"

}

```

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Cadena de mensaje con los detalles del estado de la operación:

- **Correcta – msg**: se ha reemplazado la contraseña

- **Error - error**: error de autenticación del usuario

- **Error - error**: la contraseña no coincide con la directiva de seguridad

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
response:

{

    "error": {
    
        "userDisplayErrorMessage": "User authentication failure"
    
    }

}

```

#### <a name="device-fields"></a>Campos de dispositivo

| **Nombre** | **Tipo** | **Admisión de valores NULL** |
|--|--|--|
| **username** | String | No |
| **password** | String | No |
| **new_password** | String | No |

#### <a name="curl-command"></a>Comando Curl

| Tipo | API existentes | Ejemplo |
|--|--|--|
| POST | curl -k -d '{"username": "<NOMBRE_USUARIO>","password": "<CONTRASEÑA_ACTUAL>","new_password": "<NUEVA_CONTRASEÑA>"}' -H 'Content-Type: application/json'  https://<DIRECCIÓN_IP>/external/authentication/set_password | curl -k -d '{"username": "myUser","password": "1234@abcd","new_password": "abcd@1234"}' -H 'Content-Type: application/json'  https:/<span>/127.0.0.1/external/authentication/set_password |

### <a name="user-password-update-by-system-admin---externalauthenticationset_password_by_admin"></a>Actualización de la contraseña de usuario por el administrador del sistema: /external/authentication/set_password_by_admin

Use esta API para que los administradores del sistema puedan cambiar las contraseñas de usuarios especificados. Los roles de usuario administrador de Defender para IoT pueden funcionar con la API. No necesita un token de acceso de Defender para IoT para usar esta API.

#### <a name="method"></a>Método

**POST**

#### <a name="request-type"></a>Tipo de solicitud

**JSON**

#### <a name="request-example"></a>Ejemplo de solicitud

```rest
request:

{

    "username": "test",
    
    "password": "Test12345\!",
    
    "new_password": "Test54321\!"

}
```

#### <a name="response-type"></a>Tipo de respuesta

**JSON**

#### <a name="response-content"></a>Contenido de la respuesta

Cadena de mensaje con los detalles del estado de la operación:

- **Correcta – msg**: se ha reemplazado la contraseña

- **Error - error**: error de autenticación del usuario

- **Error - error**: el usuario no existe

- **Error - error**: la contraseña no coincide con la directiva de seguridad

- **Error - error**: el usuario no tiene los permisos para cambiar la contraseña

#### <a name="response-example"></a>Ejemplo de respuesta

```rest
response:

{

    "error": {
    
        "userDisplayErrorMessage": "The user 'test_user' doesn't exist",
        
        "internalSystemErrorMessage": "The user 'yoavfe' doesn't exist"
    
    }

}

```

#### <a name="device-fields"></a>Campos de dispositivo

| **Nombre** | **Tipo** | **Admisión de valores NULL** |
|--|--|--|
| **admin_username** | String | No |
| **admin_password** | String | No |
| **username** | String | No |
| **new_password** | String | No |

#### <a name="curl-command"></a>Comando Curl

> [!div class="mx-tdBreakAll"]
> | Tipo | API existentes | Ejemplo |
> |--|--|--|
> | POST | curl -k -d '{"admin_username":"<NOMBRE_USUARIO_ADMINISTRADOR>","admin_password":"<CONTRASEÑA_ADMINISTRADOS>","username": "<NOMBRE_USUARIO>","new_password": "<NUEVA_CONTRASEÑA>"}' -H 'Content-Type: application/json'  https://<DIRECCIÓN_IP>/external/authentication/set_password_by_admin | curl -k -d '{"admin_user":"adminUser","admin_password": "1234@abcd","username": "myUser","new_password": "abcd@1234"}' -H 'Content-Type: application/json'  https:/<span>/127.0.0.1/external/authentication/set_password_by_admin |

## <a name="next-steps"></a>Pasos siguientes

- [Investigación de las detecciones de sensores en un inventario de dispositivos](how-to-investigate-sensor-detections-in-a-device-inventory.md)

- [Investigación de todas las detecciones de los sensores de empresa en un inventario de dispositivos](how-to-investigate-all-enterprise-sensor-detections-in-a-device-inventory.md)
