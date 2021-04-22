---
title: Asignación de campos de clave y de campo CommonSecurityLog de Common Event Format (CEF)
description: En este artículo se asignan las claves de CEF a los nombres de campo correspondientes en CommonSecurityLog en Azure Sentinel.
services: sentinel
author: batamig
ms.author: bagol
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: reference
ms.date: 04/12/2021
ms.openlocfilehash: 1670d1bb291e30295018146f2a24c5282feac6e7
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107311658"
---
# <a name="cef-and-commonsecuritylog-field-mapping"></a>Asignación de campos de CEF y CommonSecurityLog

En las tablas siguientes se asignan nombres de campo de Common Event Format (CEF) a los nombres que usan en el CommonSecurityLog de Azure Sentinel y pueden resultar útiles cuando se trabaja con un origen de datos de CEF en Azure Sentinel.

Para obtener más información, consulte [Conexión de la solución externa con el formato de evento común](connect-common-event-format.md).

## <a name="a---c"></a>A - C

|Nombre la de clave CEF  |Nombre del campo CommonSecurityLog  |Descripción  |
|---------|---------|---------|
| act    |    <a name="deviceaction"></a> DeviceAction     |  La acción mencionada en el evento.       |
|   app  |    ApplicationProtocol     |  Protocolo usado en la aplicación, como HTTP, HTTPS, SSHv2, Telnet, POP, IMPA, IMAPS, etc.   |
| cnt    |    EventCount     |  Un recuento asociado al evento, que muestra el número de veces que se observó el mismo evento.       |
| | | |

## <a name="d"></a>D

|Nombre la de clave CEF  |Nombre CommonSecurityLog  |Descripción  |
|---------|---------|---------|
|Proveedor del dispositivo     |  DeviceVendor       | Cadena que, junto con las definiciones de versión y producto del dispositivo, identifica de forma única el tipo de dispositivo de envío.       |
|Producto del dispositivo     |   DeviceProduct      |   Cadena que, junto con las definiciones de versión y proveedor del dispositivo, identifica de forma única el tipo de dispositivo de envío.        |
|Versión del dispositivo     |   DeviceVersion      |      Cadena que, junto con las definiciones de proveedor y producto del dispositivo, identifica de forma única el tipo de dispositivo de envío.     |
| destinationDnsDomain    | DestinationDnsDomain        |   Elemento DNS del nombre de dominio completo (FQDN).      |
| destinationServiceName | DestinationServiceName | Servicio que la etiqueta tiene como evento. Por ejemplo, `sshd`.|
| destinationTranslatedAddress | DestinationTranslatedAddress | Identifica el destino traducido al que hace referencia el evento en una red IP, como una dirección IP IPv4. |
| destinationTranslatedPort | DestinationTranslatedPort | Puerto, después de la traducción, como un firewall. <br>Números de Puerto válidos: `0` - `65535` |
| deviceDirection | <a name="communicationdirection"></a> CommunicationDirection | Cualquier información sobre la dirección que ha tomado la comunicación observada. Valores válidos: <br>- `0` = Entrante <br>- `1` = Salida |
| deviceDnsDomain | DeviceDnsDomain | Elemento del dominio DNS del nombre de dominio completo (FQDN) |
|DeviceEventClassID     |   DeviceEventClassID     |   Cadena o entero que actúa como identificador único por tipo de evento.      |
| deviceExternalID | DeviceExternalID | Nombre que identifica de forma única el dispositivo que genera el evento. |
| deviceFacility | DeviceFacility | Dispositivo que genera el evento.|
| deviceInboundInterface | DeviceInboundInterface |Interfaz en la que el paquete o los datos entran en el dispositivo.  |
| deviceNtDomain | DeviceNtDomain | El dominio de Windows de la dirección del dispositivo |
| deviceOutboundInterface | DeviceOutboundInterface |Interfaz en la que el paquete o los datos entran en el dispositivo. |
| devicePayloadId |DevicePayloadId |Identificador único de la carga asociada al evento. |
| deviceProcessName | ProcessName | Nombre del proceso asociado al evento. <br><br>Por ejemplo, en UNIX, proceso que genera la entrada de syslog. |
| deviceTranslatedAddress | DeviceTranslatedAddress | Identifica la dirección de dispositivo traducida a la que hace referencia el evento, en una red IP. <br><br>El formato es una dirección IPv4. |
| dhost |DestinationHostName | Destino al que hace referencia el evento en una red IP.  <br>El formato debe ser un FQDN asociado al nodo de destino, cuando un nodo está disponible. Por ejemplo, `host.domain.com` o `host`. |
| dmac | DestinationMacAddress | Dirección MAC de destino (FQDN) |
| dntdom | DestinationNTDomain | Nombre de dominio de Windows de la dirección de destino.|
| dpid | DestinationProcessId |Id. proceso de destino asociado al evento.|
| dpriv | DestinationUserPrivileges | Define los privilegios del uso de destino. <br>Valores válidos: `Admninistrator`, `User` y `Guest` |
| dproc | DestinationProcessName | Nombre del proceso de destino del evento, como `telnetd` o `sshd.` |
| dpt | DestinationPort | Puerto de destino. <br>Valores válidos: `*0` - `65535` |
| dst | DestinationIP | Dirección IpV4 de destino a la que hace referencia el evento en una red IP. |
| dtz | DeviceTimeZon | Zona horaria del dispositivo que genera el evento |
| duid |DestinationUserId | Identifica el usuario de destino por id. |
| duser | DestinationUserName |Identifica el usuario de destino por nombre.|
| dvc | DeviceAddress | Dirección IPv4 del dispositivo que genera el evento. |
| dvchost | DeviceName | El FQDN asociado al nodo del dispositivo, cuando un nodo está disponible. Por ejemplo, `host.domain.com` o `host`.| 
| dvcmac | DeviceMacAddress | Dirección MAC del dispositivo que genera el evento. |
| dvcpid | Identificador del proceso | Define el identificador del proceso en el dispositivo que genera el evento. |

## <a name="e---i"></a>E-I

|Nombre la de clave CEF  |Nombre CommonSecurityLog  |Descripción  |
|---------|---------|---------|
|end     |  EndTime       | La hora a la que finalizó la actividad relacionada con el evento.        |
|externalId    |   ExternalID      | Un id. usado por el dispositivo de origen. Normalmente, estos valores tienen valores crecientes que están asociados a un evento.        |
|fileCreateTime     |  FileCreateTime      | Hora a la que se creó el archivo.        |
|fileHash     |   FileHash      |   Hash de un archivo.      |
|fileId     |   FileID      |Un id. asociado a un archivo, como el inode.         |
| fileModificationTime | FileModificationTime |Hora a la que se modificó el archivo por última vez. |
| filePath | FilePath | Ruta de acceso completa del archivo, incluido el nombre de archivo. Por ejemplo, `C:\ProgramFiles\WindowsNT\Accessories\wordpad.exe` o `/usr/bin/zip`.|
| filePermission |FilePermission |Permisos del archivo. |
| fileType | FileType | Tipo de archivo, como canalización, socket, etc.|
|fname | FileName| Nombre del archivo, sin la ruta de acceso. |
| fsize | FileSize | Tamaño del archivo. |
|Host    |  Computer       | Host, desde Syslog        |
|in     |  ReceivedBytes      |Número de bytes transferidos de entrada.         |
| | | |

## <a name="m---p"></a>M-P

|Nombre la de clave CEF  |Nombre CommonSecurityLog  |Descripción  |
|---------|---------|---------|
|msg   |  Message       | Un mensaje que proporciona más detalles sobre el evento.        |
|Nombre     |  Actividad       |   Cadena que representa una descripción inteligible y comprensible del evento.     |
|oldFileCreateTime     |  OldFileCreateTime       | Hora a la que se creó el archivo anterior.        |
|oldFileHash     |   OldFileHash      |   Hash del archivo anterior.      |
|oldFileId     |   OldFileId     |   Y el identificador asociado al archivo anterior, como el inode.      |
| oldFileModificationTime | OldFileModificationTime |Hora a la que se modificó el archivo anterior por última vez. |
| oldFileName |  OldFileName |Nombre del archivo anterior. |
| oldFilePath | OldFilePath | Ruta de acceso completa del archivo anterior, incluido el nombre de archivo. <br>Por ejemplo, `C:\ProgramFiles\WindowsNT\Accessories\wordpad.exe` o `/usr/bin/zip`.|
| oldFilePermission | OldFilePermission |Permisos del archivo anterior. |
|oldFileSize | OldFileSize | Tamaño del archivo anterior.|
| oldFileType | OldFileType | Tipo de archivo del archivo anterior, como una canalización, un socket, etc.|
| out | SentBytes | Número de bytes transferidos de salida. |
| Resultado | Resultado | Resultado del evento, como `success` o `failure`.|
|proto    |  Protocolo       | Protocolo de transporte que identifica el protocolo de nivel 4 utilizado. <br><br>Los valores posibles incluyen nombres de protocolo, como `TCP` o `UDP`.        |
| | | |

## <a name="r---t"></a>R-T

|Nombre la de clave CEF  |Nombre CommonSecurityLog  |Descripción  |
|---------|---------|---------|
|Razón     |  Motivo      |Motivo por el que se generó un evento de auditoría. <br><br>Por ejemplo, `Bad password` o `Unknown user`.         |
|Request     |   RequestURL      | Dirección URL a la que se tiene acceso para una solicitud HTTP, incluido el protocolo. Por ejemplo: `http://www/secure.com`        |
|requestClientApplication     |   RequestClientApplication      |   Agente de usuario asociado a la solicitud.      |
| requestContext | RequestContext | Describe el contenido desde el que se originó la solicitud, como el origen de referencia HTTP. |
| requestCookies | RequestCookies |Cookies asociadas a la solicitud. |
| requestMethod | RequestMethod | Método utilizado para tener acceso a una dirección URL. <br><br>Los valores válidos incluyen métodos como `POST`, `GET`, etc. |
| rt | ReceiptTime | La hora a la que finalizó el evento relacionado con la actividad que se recibió. |
|Gravedad     |  <a name="logseverity"></a> LogSeverity       |  Cadena o entero que describe la importancia del evento.<br><br> Valores de cadena válidos: `Unknown` , `Low`, `Medium`, `High` y `Very-High` <br><br>Los valores de entero válidos son:<br> - `0`-`3` = Bajo <br>- `4`-`6` = Medio<br>- `7`-`8` = Alto<br>- `9`-`10` = Muy alto |
| shost    | SourceHostName        |Identifica el origen al que hace referencia el evento en una red IP. El formato debe ser un nombre de dominio completo (DQDN) asociado al nodo de origen, cuando un nodo está disponible. Por ejemplo, `host` o `host.domain.com`. |
| smac | SourceMacAddress | Dirección MAC de origen. |
| sntdom | SourceNTDomain | Nombre de dominio de Windows para la dirección de origen. |
| sourceDnsDomain | SourceDnsDomain | Elemento del dominio DNS del FQDN completo. |
| sourceServiceName | SourceServiceName | Servicio responsable de generar el evento. |
| sourceTranslatedAddress | SourceTranslatedAddress | Identifica el origen traducido al que hace referencia el evento, en una red IP. |
| sourceTranslatedPort | SourceTranslatedPort | Puerto de origen después de la traducción, como un firewall. <br>Los números de puerto válidos son `0` - `65535`. |
| spid | SourceProcessId | Id. del proceso de origen asociado al evento.|
| spriv | SourceUserPrivileges | Privilegios del usuario de origen. <br><br>Los valores válidos incluyen: `Administrator`, `User` y `Guest` |
| sproc | SourceProcessName | Nombre del proceso de origen del evento.|
| spt | SourcePort | Número de puerto de origen. <br>Los números de puerto válidos son `0` - `65535`. |
| src | SourceIP |Origen al que hace referencia un evento en una red IP, como una dirección IPv4. |
| start | StartTime | Hora a la que se inicia la actividad a la que hace referencia el evento. |
| suid | SourceUserID | Identifica el usuario de origen por identificador. |
| suser | SourceUserName | Identifica el usuario de origen por nombre. |
| type | EventType | Tipo de evento. Los valores de valor incluyen: <br>- `0`: evento base <br>- `1`: agregado <br>- `2`: eventos de correlación <br>- `3`: evento de acción <br><br>**Nota**: este evento se puede omitir para los eventos base. |
| | | |

## <a name="custom-fields"></a>Custom Fields

En las tablas siguientes se asignan los nombres de las claves CEF y los campos CommonSecurityLog que están disponibles para que los clientes los usen para los datos que no se aplican a ninguno de los campos integrados.

### <a name="custom-ipv6-address-fields"></a>Campos de dirección IPv6 personalizados

En la tabla siguiente se asigna la clave CEF y los nombres de CommonSecurityLog para los campos de dirección *IPv6* disponibles para los datos personalizados.

|Nombre la de clave CEF  |Nombre CommonSecurityLog  |
|---------|---------|
|     c6a1    |     DeviceCustomIPv6Address1       |
|     c6a1Label    |     DeviceCustomIPv6Address1Label    |
|     c6a2    |     DeviceCustomIPv6Address2    |
|     c6a2Label    |     DeviceCustomIPv6Address2Label    |
|     c6a3    |     DeviceCustomIPv6Address3    |
|     c6a3Label    |     DeviceCustomIPv6Address3Label    |
|     c6a4    |     DeviceCustomIPv6Address4    |
|     c6a4Label    |     DeviceCustomIPv6Address4Label    |
|     cfp1    |     DeviceCustomFloatingPoint1    |
|     cfp1Label    |     deviceCustomFloatingPoint1Label    |
|     cfp2    |     DeviceCustomFloatingPoint2    |
|     cfp2Label    |     deviceCustomFloatingPoint2Label    |
|     cfp3    |     DeviceCustomFloatingPoint3    |
|     cfp3Label    |     deviceCustomFloatingPoint3Label    |
|     cfp4    |     DeviceCustomFloatingPoint4    |
|     cfp4Label    |     deviceCustomFloatingPoint4Label    |
| | |

### <a name="custom-number-fields"></a>Campos de número personalizados

En la tabla siguiente se asigna la clave CEF y los nombres de CommonSecurityLog para los campos de *número* disponibles para los datos personalizados.

|Nombre la de clave CEF  |Nombre CommonSecurityLog  |
|---------|---------|
|     cn1    |     DeviceCustomNumber1       |
|     cn1Label    |     DeviceCustomNumber1Label       |
|     cn2    |     DeviceCustomNumber2       |
|     cn2Label    |     DeviceCustomNumber2Label       |
|     cn3    |     DeviceCustomNumber3       |
|     cn3Label    |     DeviceCustomNumber3Label       |
| | |

### <a name="custom-string-fields"></a>Campos de cadena personalizados

En la tabla siguiente se asigna la clave CEF y los nombres de CommonSecurityLog para los campos de *cadena* disponibles para los datos personalizados.

|Nombre la de clave CEF  |Nombre CommonSecurityLog  |
|---------|---------|
|     cs1    |     DeviceCustomString1 <sup>[1](#use-sparingly)</sup>    |
|     cs1Label    |     DeviceCustomString1Label <sup>[1](#use-sparingly)</sup>    |
|     cs2    |     DeviceCustomString2 <sup>[1](#use-sparingly)</sup>   |
|     cs2Label    |     DeviceCustomString2Label   <sup>[1](#use-sparingly)</sup> |
|     cs3    |     DeviceCustomString3  <sup>[1](#use-sparingly)</sup>  |
|     cs3Label    |     DeviceCustomString3Label   <sup>[1](#use-sparingly)</sup> |
|     cs4    |     DeviceCustomString4   <sup>[1](#use-sparingly)</sup> |
|     cs4Label    |     DeviceCustomString4Label  <sup>[1](#use-sparingly)</sup>  |
|     cs5    |     DeviceCustomString5 <sup>[1](#use-sparingly)</sup>   |
|     cs5Label    |     DeviceCustomString5Label <sup>[1](#use-sparingly)</sup>    |
|     cs6    |     DeviceCustomString6   <sup>[1](#use-sparingly)</sup> |
|     cs6Label    |     DeviceCustomString6Label   <sup>[1](#use-sparingly)</sup> |
|     flexString1    |     FlexString1    |
|     flexString1Label    |     FlexString1Label    |
|     flexString2    |     FlexString2    |
|     flexString2Label    |     FlexString2Label    |
| | |

> [!TIP]
> <a name="use-sparingly"></a><sup>1</sup> Se recomienda usar los campos **DeviceCustomString** con moderación y usar campos integrados más específicos cuando sea posible.
> 

### <a name="custom-timestamp-fields"></a>Campos de marca de tiempo personalizados

En la tabla siguiente se asigna la clave CEF y los nombres de CommonSecurityLog para los campos de *marca de tiempo* disponibles para los datos personalizados.

|Nombre la de clave CEF  |Nombre CommonSecurityLog  |
|---------|---------|
|     deviceCustomDate1    |     DeviceCustomDate1    |
|     deviceCustomDate1Label    |     DeviceCustomDate1Label    |
|     deviceCustomDate2       |     DeviceCustomDate2    |
|     deviceCustomDate2Label    |     DeviceCustomDate2Label    |
|     flexDate1    |     FlexDate1    |
|     flexDate1Label    |     FlexDate1Label    |
| | |

### <a name="custom-integer-data-fields"></a>Campos de datos enteros personalizados

En la tabla siguiente se asigna la clave CEF y los nombres de CommonSecurityLog para los campos de *enteros* disponibles para los datos personalizados.

|Nombre la de clave CEF  |Nombre CommonSecurityLog  |
|---------|---------|
|     flexNumber1    |     FlexNumber1    |
|     flexNumber1Label    |     FlexNumber1Label    |
|     flexNumber2    |     FlexNumber2    |
|     flexNumber2Label    |     FlexNumber2Label    |
| | |

## <a name="enrichment-fields"></a>Campos de enriquecimiento

Los siguientes campos **commonSecurityLog** se agregan mediante Azure Sentinel para enriquecer los eventos originales recibidos de los dispositivos de origen, y no tienen asignaciones en las claves CEF:

### <a name="threat-intelligence-fields"></a>Campos de inteligencia sobre amenazas

|Nombre del campo CommonSecurityLog  |Descripción  |
|---------|---------|
|   **IndicatorThreatType**  |  El tipo de amenaza de [MaliciousIP](#MaliciousIP), según la fuente de inteligencia sobre amenazas.       |
| <a name="MaliciousIP"></a>**MaliciousIP** | Enumera las direcciones IP del mensaje que se correlacionan con la fuente de inteligencia sobre amenazas actual. |
|  **MaliciousIPCountry**   | El país de [MaliciousIP](#MaliciousIP), según la información geográfica en el momento de la ingesta de registros.        |
| **MaliciousIPLatitude**    |   La longitud de [MaliciousIP](#MaliciousIP), según la información geográfica en el momento de la ingesta de registros.      |
| **MaliciousIPLongitude**    |  La longitud de [MaliciousIP](#MaliciousIP), según la información geográfica en el momento de la ingesta de registros.       |
| **ReportReferenceLink**    |    Vínculo al informe de inteligencia sobre amenazas.     |
|  **ThreatConfidence**   |   La confianza de la amenaza [MaliciousIP](#MaliciousIP), según la fuente de inteligencia sobre amenazas.      |
| **ThreatDescription**    |   La descripción de [MaliciousIP](#MaliciousIP), según la fuente de inteligencia sobre amenazas.      |
| **ThreatSeverity** | La gravedad de la amenaza para [MaliciousIP](#MaliciousIP), según la fuente de inteligencia sobre amenazas en el momento de la ingesta de registros. |
|     |         |

### <a name="additional-enrichment-fields"></a>Campos de enriquecimiento adicionales

|Nombre del campo CommonSecurityLog  |Descripción  |
|---------|---------|
|**OriginalLogSeverity**     |  Siempre está vacío, compatible para la integración con CiscoASA. <br>Para obtener más información sobre los valores de gravedad del registro, vea el campo [LogSeverity](#logseverity).       |
|**RemoteIP**     |     Dirección IP remota. <br>Este valor se basa en el campo [CommunicationDirection](#communicationdirection), si es posible.     |
|**RemotePort**     |   Puerto remoto. <br>Este valor se basa en el campo [CommunicationDirection](#communicationdirection), si es posible.      |
|**SimplifiedDeviceAction**     |   Simplifica el valor de [DeviceAction](#deviceaction) a un conjunto estático de valores, manteniendo el valor original en el campo [DeviceAction](#deviceaction). <br>Por ejemplo: `Denied` > `Deny`.      |
|**SourceSystem**     | Siempre se define como **OpsManager**.        |
|     |         |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte [Conexión de la solución externa con el formato de evento común](connect-common-event-format.md).
