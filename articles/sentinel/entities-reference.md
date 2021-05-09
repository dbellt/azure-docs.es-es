---
title: Referencia de los tipos de entidad de Azure Sentinel | Microsoft Docs
description: En este artículo se muestran los tipos de entidad de Azure Sentinel y sus identificadores necesarios.
services: sentinel
cloud: na
documentationcenter: na
author: yelevin
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 02/10/2021
ms.author: yelevin
ms.openlocfilehash: acae7a6121587a1a58c6f19b0b09287b3d93cc84
ms.sourcegitcommit: 18cd3c1c8cc47258c6a1a04e0e03d6248c52ef24
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107992168"
---
# <a name="azure-sentinel-entity-types-reference"></a>Referencia de los tipos de entidad de Azure Sentinel

## <a name="entity-types-and-identifiers"></a>Tipos de entidad e identificadores

En la tabla siguiente se muestran los **tipos de entidad** disponibles actualmente para la asignación en Azure Sentinel y los **atributos** disponibles como **identificadores** para cada tipo de entidad, que aparecen en la lista desplegable **Identificadores** en la sección [Asignación de entidades](map-data-fields-to-entities.md) del [Asistente para reglas de análisis](tutorial-detect-threats-custom.md).

Cada uno de los identificadores de la columna **Identificadores necesarios** es mínimamente necesario para identificar su entidad. Sin embargo, es posible que un identificador necesario, por sí solo, no sea suficiente para proporcionar una identificación *única*. Cuanto más identificadores se usan, mayor es la probabilidad de una identificación única. Puede usar hasta tres identificadores para una asignación de entidad única.

Para obtener los mejores resultados (una identificación única garantizada) debe usar identificadores de la columna **Identificadores más seguros** siempre que sea posible. El uso de varios identificadores seguros permite la correlación entre identificadores seguros de distintos esquemas y orígenes de datos. Esto, a su vez, permite que Azure Sentinel proporcione conclusiones más completas para una entidad determinada.

| Tipo de entidad | Identificadores | Identificadores necesarios | Identificadores más seguros |
| - | - | - | - |
| [**Cuenta de usuario**](#user-account)<br>*(Cuenta)* | Nombre<br>FullName<br>NTDomain<br>DnsDomain<br>UPNSuffix<br>Sid<br>AadTenantId<br>AadUserId<br>PUID<br>IsDomainJoined<br>DisplayName<br>ObjectGuid | FullName<br>Sid<br>Nombre<br>AadUserId<br>PUID<br>ObjectGuid | Name + NTDomain<br>Name + UPNSuffix<br>AadUserId<br>Sid |
| [**Host**](#host) | DnsDomain<br>NTDomain<br>HostName<br>FullName<br>NetBiosName<br>AzureID<br>OMSAgentID<br>OSFamily<br>OSVersion<br>IsDomainJoined | FullName<br>HostName<br>NetBiosName<br>AzureID<br>OMSAgentID | HostName + NTDomain<br>HostName + DnsDomain<br>NetBiosName + NTDomain<br>NetBiosName + DnsDomain<br>AzureID<br>OMSAgentID |
| [**Dirección IP**](#ip-address)<br>*(IP)* | Dirección | Dirección | |
| [**Malware**](#malware) | Nombre<br>Category | Nombre | |
| [**Archivo**](#file) | Directorio<br>Nombre | Nombre | |
| [**Proceso**](#process) | ProcessId<br>CommandLine<br>ElevationToken<br>CreationTimeUtc | CommandLine<br>ProcessId | |
| [**Aplicación en la nube**](#cloud-application)<br>*(CloudApplication)* | AppId<br>Nombre<br>InstanceName | AppId<br>Nombre | |
| [**Nombre de dominio**](#domain-name)<br>*(DNS)* | DomainName | DomainName | |
| [**Recurso de Azure**](#azure-resource) | ResourceId | ResourceId | |
| [**Hash de archivo**](#file-hash)<br>*(FileHash)* | Algoritmo<br>Valor | Algoritmo + valor | |
| [**Clave del Registro**](#registry-key) | Hive<br>Clave | Hive<br>Clave | Hive + clave |
| [**Valor del Registro**](#registry-value) | Nombre<br>Valor<br>ValueType | Nombre | |
| [**Grupo de seguridad**](#security-group) | DistinguishedName<br>SID<br>ObjectGuid | DistinguishedName<br>SID<br>ObjectGuid | |
| [**URL**](#url) | Url | Url | |
| [**Dispositivo IoT**](#iot-device) | IoTHub<br>DeviceId<br>DeviceName<br>IoTSecurityAgentId<br>DeviceType<br>Source<br>SourceRef<br>Fabricante<br>Modelado<br>OperatingSystem<br>IpAddress<br>MacAddress<br>Protocolos<br>SerialNumber | IoTHub<br>deviceId | IoTHub + DeviceId |
| [**Mailbox**](#mailbox) | MailboxPrimaryAddress<br>DisplayName<br>Upn<br>ExternalDirectoryObjectId<br>RiskLevel | MailboxPrimaryAddress | |
| [**Clúster de correo**](#mail-cluster) | NetworkMessageIds<br>CountByDeliveryStatus<br>CountByThreatType<br>CountByProtectionStatus<br>Amenazas<br>Consultar<br>QueryTime<br>MailCount<br>IsVolumeAnomaly<br>Source<br>ClusterSourceIdentifier<br>ClusterSourceType<br>ClusterQueryStartTime<br>ClusterQueryEndTime<br>ClusterGroup | Consultar<br>Source | Query + Source |
| [**Mensaje de correo**](#mail-message) | Recipient<br>Direcciones URL<br>Amenazas<br>Remitente<br>P1Sender<br>P1SenderDisplayName<br>P1SenderDomain<br>SenderIP<br>P2Sender<br>P2SenderDisplayName<br>P2SenderDomain<br>ReceivedDate<br>NetworkMessageId<br>InternetMessageId<br>Asunto<br>BodyFingerprintBin1<br>BodyFingerprintBin2<br>BodyFingerprintBin3<br>BodyFingerprintBin4<br>BodyFingerprintBin5<br>AntispamDirection<br>DeliveryAction<br>DeliveryLocation<br>Lenguaje<br>ThreatDetectionMethods | NetworkMessageId<br>Recipient | NetworkMessageId + Recipient |
| [**Correo de envío**](#submission-mail) | SubmissionId<br>SubmissionDate<br>Remitente<br>NetworkMessageId<br>Timestamp<br>Recipient<br>Remitente<br>SenderIp<br>Asunto<br>ReportType | SubmissionId<br>NetworkMessageId<br>Recipient<br>Remitente |  |
|

## <a name="entity-type-schemas"></a>Esquemas de tipo de entidad

A continuación se ofrece una vista más detallada de los esquemas completos de cada tipo de entidad. Observará que muchos de estos esquemas incluyen vínculos a otros tipos de entidad; por ejemplo, el esquema de la cuenta de usuario incluye un vínculo al tipo de entidad host, ya que un atributo de una cuenta de usuario es el host en el que está definido. Estas entidades vinculadas externamente no se pueden usar como identificadores para la asignación de entidades, pero son muy útiles para ofrecer una imagen completa de las entidades en las páginas de entidades y en el gráfico de investigación.

> [!NOTE]
> Un signo de interrogación después del valor de la columna **Tipo** indica que el campo admite un valor NULL.

## <a name="user-account"></a>Cuenta de usuario

*Nombre de entidad: Cuenta*

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | ‘account’ |
| Nombre | String | Nombre de la cuenta. Este campo solo debe contener el nombre sin ningún dominio agregado. |
| *FullName* | *N/D* | *No forma parte del esquema, incluido para la compatibilidad con versiones anteriores de la asignación de entidades.*
| NTDomain | String | Nombre de dominio NETBIOS tal como aparece en el formato de alerta: dominio\nombreDeUsuario. Ejemplos: Finanzas, NT AUTHORITY |
| DnsDomain | String | Nombre DNS del dominio completo. Ejemplos: finance.contoso.com |
| UPNSuffix | String | Sufijo de nombre principal de usuario para la cuenta. En algunos casos, también es el nombre de dominio. Ejemplos: contoso.com |
| Host | Entidad | Host que contiene la cuenta, si es una cuenta local. |
| Sid | String | Identificador de seguridad de la cuenta, como S-1-5-18. |
| AadTenantId | ¿GUID? | Id. de inquilino de Azure AD, si se conoce. |
| AadUserId | ¿GUID? | Id. de objeto de la cuenta de Azure AD, si se conoce. |
| PUID | ¿GUID? | Id. de usuario de pasaporte de Azure AD, si se conoce. |
| IsDomainJoined | ¿Valor booleano? | Determina si se trata de una cuenta de dominio. |
| DisplayName | String | Nombre para mostrar de la cuenta. |
| ObjectGuid | ¿GUID? | El atributo objectGUID es un atributo de un solo valor y es el identificador único del objeto, asignado por Active Directory. |
|

Identificadores seguros de una entidad Cuenta:

- Name + UPNSuffix
- AadUserId
- SID + Host (necesario para los SID de cuentas integradas)
- SID (excepto los SID de las cuentas integradas)
- Name + NTDomain (a menos que NTDomain sea un dominio integrado, como "Workgroup")
- Name + Host (si NTDomain es un dominio integrado, como "Workgroup")
- Name + DnsDomain
- PUID
- ObjectGuid

Identificadores no seguros de una entidad Cuenta:

- Nombre

## <a name="host"></a>Host

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | ‘host’ |
| DnsDomain | String | Dominio DNS al que pertenece este host. Debe contener el sufijo DNS completo del dominio, si se conoce. |
| NTDomain | String | Dominio NT al que pertenece este host. |
| HostName | String | Nombre del host sin el sufijo de dominio. |
| *FullName* | *N/D* | *No forma parte del esquema, incluido para la compatibilidad con versiones anteriores de la asignación de entidades.*
| NetBiosName | String | Nombre del host (antes de Windows 2000). |
| IoTDevice | Entidad | Entidad del dispositivo IoT (si este host representa un dispositivo IoT). |
| AzureID | String | Id. del recurso de Azure de la VM, si se conoce. |
| OMSAgentID | String | Id. del agente de OMS, si el host tiene instalado el agente de OMS. |
| OSFamily | ¿Enumeración? | Uno de los siguientes valores: <li>Linux<li>Windows<li>Android<li>IOS |
| OSVersion | String | Representación de texto libre del sistema operativo.<br>Este campo está pensado para contener versiones específicas que son más detalladas que OSFamily o valores futuros que la enumeración OSFamily no admite. |
| IsDomainJoined | Bool | Determina si este host pertenece a un dominio. |
|

Identificadores seguros de una entidad de host:
- HostName + NTDomain
- HostName + DnsDomain
- NetBiosName + NTDomain
- NetBiosName + DnsDomain
- AzureID
- OMSAgentID
- IoTDevice (no compatible con la asignación de entidades)

Identificadores no seguros de una entidad de host:
- HostName
- NetBiosName

## <a name="ip-address"></a>Dirección IP

*Nombre de entidad: IP*

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | ‘ip’ |
| Dirección | String | Dirección IP en forma de cadena, como 127.0.0.1 (ya sea IPv4 o IPv6). |
| Ubicación | GeoLocation | Contexto de ubicación geográfica asociado a la entidad IP. |
|

Identificadores seguros de una entidad IP:
- Dirección

## <a name="malware"></a>Malware

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | ‘malware’ |
| Nombre | String | Nombre del malware proporcionado por el proveedor, como `Win32/Toga!rfn`. |
| Category | String | Categoría de malware proporcionada por el proveedor, como Troyano. |
| Archivos | List\<Entity> | Lista de entidades de archivo vinculadas en las que se encontró el malware. Puede contener las entidades Archivo en línea o como referencia.<br>Consulte la entidad Archivo para obtener más detalles sobre la estructura. |
| Procesos | List\<Entity> | Lista de entidades Proceso vinculadas en las que se encontró el malware. Se suele usar cuando la alerta se desencadena en una actividad sin archivos.<br>Consulte la entidad [Proceso](#process) para obtener más detalles sobre la estructura. |
|

Identificadores seguros de una entidad Malware:

- Nombre + categoría

## <a name="file"></a>Archivo

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | ‘file’ |
| Directorio | String | Ruta de acceso completa al archivo. |
| Nombre | String | Nombre de archivo sin la ruta de acceso (es posible que algunas alertas no incluyan la ruta de acceso). |
| Host | Entidad | Host en el que se almacenó el archivo. |
| FileHashes | Lista&lt;entidad&gt; | Hash de archivo asociados a este archivo. |
|

Identificadores seguros de una entidad Archivo:
- Name + Directory
- Name + FileHash
- Name + Directory + FileHash

## <a name="process"></a>Proceso

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | ‘process’ |
| ProcessId | String | El id. de proceso. |
| CommandLine | String | Línea de comandos utilizada para crear el proceso. |
| ElevationToken | ¿Enumeración? | Token de elevación asociado con el proceso.<br>Valores posibles:<li>TokenElevationTypeDefault<li>TokenElevationTypeFull<li>TokenElevationTypeLimited |
| CreationTimeUtc | DateTime? | Hora en que se empezó a ejecutar el proceso. |
| ImageFile | Entidad (archivo) | Puede contener la entidad de archivo en línea o como referencia.<br>Consulte la entidad de archivo para obtener más detalles sobre la estructura. |
| Cuenta | Entidad | Cuenta que ejecuta los procesos.<br>Puede contener la entidad [Cuenta](#user-account) en línea o como referencia.<br>Consulte la entidad [Cuenta](#user-account) para obtener más detalles sobre la estructura. |
| ParentProcess | Entidad (proceso) | Entidad del proceso principal. <br>Puede contener datos parciales, es decir, solo el PID. |
| Host | Entidad | Host en el que se estaba ejecutando el proceso. |
| LogonSession | Entidad (HostLogonSession) | Sesión en la que se estaba ejecutando el proceso. |
|

Identificadores seguros de una entidad Proceso:

- Host + ProcessId + CreationTimeUtc
- Host + ParentProcessId + CreationTimeUtc + CommandLine
- Host + ProcessId + CreationTimeUtc + ImageFile
- Host + ProcessId + CreationTimeUtc + ImageFile.FileHash

Identificadores no seguros de una entidad Proceso:

- ProcessId + CreationTimeUtc + CommandLine (y ningún host)
- ProcessId + CreationTimeUtc + ImageFile (y ningún host)

## <a name="cloud-application"></a>Aplicación en la nube

*Nombre de entidad: CloudApplication*

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | ‘cloud-application’ |
| AppId | Int | Identificador técnico de la aplicación. Debe ser uno de los valores definidos en la lista de [identificadores de aplicación en la nube](#cloud-application-identifiers). El valor del campo AppId es opcional. |
| Nombre | String | Nombre de la aplicación en la nube relacionada. El valor del nombre de la aplicación es opcional. |
| InstanceName | String | Nombre de instancia definido por el usuario de la aplicación en la nube. A menudo se usa para distinguir entre varias aplicaciones del mismo tipo que tiene un cliente. |
|

Identificadores seguros de una entidad Aplicación en la nube:
 - AppId (sin InstanceName)
 - Name (sin InstanceName)
 - AppId + InstanceName
 - Name + InstanceName

## <a name="domain-name"></a>Nombre de dominio

*Nombre de entidad: DNS*

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | ‘dns’ |
| DomainName | String | Nombre del registro DNS asociado a la alerta. |
| IpAddress | Lista&lt;entidad (IP)&gt; | Entidades correspondientes a las direcciones IP resueltas. |
| DnsServerIp | Entidad (IP) | Entidad que representa el servidor DNS que resuelve la solicitud. |
| HostIpAddress | Entidad (IP) | Entidad que representa el cliente de la solicitud DNS. |
|

Identificadores seguros de una entidad DNS:
- DomainName + DnsServerIp + HostIpAddress

Identificadores no seguros de una entidad DNS:
- DomainName + HostIpAddress

## <a name="azure-resource"></a>Recurso de Azure

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | 'azure-resource' |
| ResourceId | String | Id. de recurso de Azure del recurso. |
| SubscriptionId | String | Identificador de suscripción del recurso. |
| TryGetResourceGroup | Bool | Valor del grupo de recursos si existe. |
| TryGetProvider | Bool | Valor del proveedor si existe. |
| TryGetName | Bool | Valor del nombre si existe. |
|

Identificadores seguros de una entidad de recurso de Azure:
- ResourceId

## <a name="file-hash"></a>Hash de archivo

*Nombre de entidad: FileHash*

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | 'filehash' |
| Algoritmo | Enumeración | Tipo de algoritmo hash. Valores posibles:<li>Desconocido<li>MD5<li>SHA1<li>SHA256<li>SHA256AC |
| Valor | String | El valor hash. |
|

Identificadores seguros de una entidad Hash de archivo:
- Algoritmo + valor

## <a name="registry-key"></a>Clave del Registro

*Nombre de entidad: RegistryKey*

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | ‘registry-key’ |
| Hive | ¿Enumeración? | Uno de los siguientes valores:<li>HKEY_LOCAL_MACHINE<li>HKEY_CLASSES_ROOT<li>HKEY_CURRENT_CONFIG<li>HKEY_USERS<li>HKEY_CURRENT_USER_LOCAL_SETTINGS<li>HKEY_PERFORMANCE_DATA<li>HKEY_PERFORMANCE_NLSTEXT<li>HKEY_PERFORMANCE_TEXT<li>HKEY_A<li>HKEY_CURRENT_USER |
| Clave | String | Ruta de acceso a la clave del Registro. |
|

Identificadores seguros de una entidad de clave del Registro:
- Hive + clave

## <a name="registry-value"></a>Valor del Registro

*Nombre de entidad: RegistryValue*

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | ‘registry-value’ |
| Clave | Entidad (RegistryKey) | Entidad de clave del Registro. |
| Nombre | String | Nombre del valor del Registro. |
| Valor | String | Representación con formato de cadena de los datos del valor. |
| ValueType | ¿Enumeración? | Uno de los siguientes valores:<li>String<li>Binary<li>DWord<li>Qword<li>MultiString<li>ExpandString<li>Ninguno<li>Desconocido<br>Los valores deben ajustarse a la enumeración Microsoft.Win32.RegistryValueKind. |
|

Identificadores seguros de una entidad de valor del Registro:
- Key + Name

Identificadores no seguros de una entidad de valor del Registro:
- Name (sin Key)

## <a name="security-group"></a>Grupo de seguridad

*Nombre de entidad: SecurityGroup*

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | 'security-group' |
| DistinguishedName | String | Nombre distintivo del grupo. |
| SID | String | El atributo SID es un atributo de un solo valor que especifica el identificador de seguridad (SID) del grupo. |
| ObjectGuid | ¿GUID? | El atributo objectGUID es un atributo de un solo valor y es el identificador único del objeto, asignado por Active Directory. |
|

Identificadores seguros de una entidad de grupo de seguridad:
 - DistinguishedName
 - SID
 - ObjectGuid

## <a name="url"></a>URL

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | 'url' |
| Url | Identificador URI | Dirección URL completa a la que apunta la entidad. |
|

Identificadores seguros de una entidad URL:
- URL (si es una dirección URL absoluta)

Identificadores no seguros de una entidad URL:
- URL (si es una dirección URL relativa)

## <a name="iot-device"></a>Dispositivo IoT

*Nombre de entidad: IoTDevice*

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | 'iotdevice' |
| IoTHub | Entidad (AzureResource) | Entidad AzureResource que representa la instancia de IoT Hub a la que pertenece el dispositivo. |
| deviceId | String | Id. del dispositivo en el contexto de la instancia de IoT Hub. |
| DeviceName | String | Nombre descriptivo del dispositivo. |
| IoTSecurityAgentId | ¿GUID? | Id. del agente de *Defender para IOT* que se ejecuta en el dispositivo. |
| DeviceType | String | Tipo de dispositivo ("sensor de temperatura", "congelador", "aerogenerador", etc.). |
| Source | String | Origen (Microsoft/proveedor) de la entidad de dispositivo. |
| SourceRef | Entidad (URL) | Referencia de URL al elemento de origen donde se administra el dispositivo. |
| Fabricante | String | Fabricante del dispositivo. |
| Modelado | String | Modelo del dispositivo. |
| OperatingSystem | String | Sistema operativo que ejecuta el dispositivo. |
| IpAddress | Entidad (IP) | Dirección IP actual del dispositivo. |
| MacAddress | String | La dirección MAC del dispositivo. |
| Protocolos | Lista&lt;cadena&gt; | Lista de los protocolos que admite el dispositivo. |
| SerialNumber | String | El número de serie del dispositivo. |
|

Identificadores seguros de una entidad de dispositivo IoT:
- IoTHub + DeviceId

Identificadores no seguros de una entidad de dispositivo IoT:
- DeviceId (sin IoTHub)

## <a name="mailbox"></a>Mailbox

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | 'mailbox' |
| MailboxPrimaryAddress | String | Dirección principal del buzón. |
| DisplayName | String | Nombre para mostrar del buzón. |
| Upn | String | UPN del buzón. |
| RiskLevel | ¿Enumeración? | Nivel de riesgo del buzón. Valores posibles:<li>Ninguno<li>Bajo<li>Media<li>Alto |
| ExternalDirectoryObjectId | ¿GUID? | Identificador de AzureAD del buzón. Similar a AadUserId en la entidad Cuenta, pero esta propiedad es específica del objeto de buzón en Office. |
|

Identificadores seguros de una entidad de buzón:
- MailboxPrimaryAddress

## <a name="mail-cluster"></a>Clúster de correo

*Nombre de entidad: MailCluster*

> [!NOTE]
> **Microsoft Defender para Office 365** se conocía anteriormente como Protección contra amenazas avanzada de Office 365 (ATP de O365).

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | 'mail-cluster' |
| NetworkMessageIds | IList&lt;cadena&gt; | Identificadores de mensaje de correo que forman parte del clúster de correo. |
| CountByDeliveryStatus | IDictionary&lt;cadena,entero&gt; | Recuento de mensajes de correo por representación de cadena DeliveryStatus. |
| CountByThreatType | IDictionary&lt;cadena,entero&gt; | Recuento de mensajes de correo por representación de cadena ThreatType. |
| CountByProtectionStatus | IDictionary&lt;cadena,larga&gt; | Recuento de mensajes de correo por estado de protección contra amenazas. |
| Amenazas | IList&lt;cadena&gt; | Amenazas de los mensajes de correo que forman parte del clúster de correo. |
| Consultar | String | Consulta que se usó para identificar los mensajes del clúster de correo. |
| QueryTime | DateTime? | Tiempo de la consulta. |
| MailCount | ¿Entero? | Número de mensajes de correo que forman parte del clúster de correo. |
| IsVolumeAnomaly | ¿Valor booleano? | Determina si se trata de un clúster de correo de anomalías de volumen. |
| Source | String | Origen del clúster de correo (el valor predeterminado es "O365 ATP"). |
| ClusterSourceIdentifier | String | Id. del mensaje de red del correo que es el origen de este clúster de correo. |
| ClusterSourceType | String | Tipo de origen del clúster de correo. Esto se asigna a la configuración MailClusterSourceType de Microsoft Defender para Office 365 (consulte la nota anterior). |
| ClusterQueryStartTime | DateTime? | Hora de inicio del clúster: se usa como hora de inicio para la consulta de recuentos de clúster. Normalmente, se establece en la hora de finalización menos el valor de DaysToLookBack de Microsoft Defender para Office 365 (vea la nota anterior). |
| ClusterQueryEndTime | DateTime? | Hora de finalización del clúster: se usa como hora de finalización para la consulta de recuentos de clúster. Normalmente, hora de recepción de los datos de correo. |
| ClusterGroup | String | Corresponde a la clave de consulta de Kusto que se usa en Microsoft Defender para Office 365 (consulte la nota anterior). |
|

Identificadores seguros de una entidad de clúster de correo:
- Query + Source

## <a name="mail-message"></a>Mensaje de correo

*Nombre de entidad: MailMessage*

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | 'mail-message' |
| Archivos | IList&lt;archivo&gt; | Entidades Archivo de los datos adjuntos de este mensaje de correo. |
| Recipient | String | Destinatario de este mensaje de correo. En el caso de varios destinatarios, el mensaje de correo se copia y cada copia tiene un destinatario. |
| Direcciones URL | IList&lt;cadena&gt; | Direcciones URL includas en este mensaje de correo. |
| Amenazas | IList&lt;cadena&gt; | Amenazas incluidas en este mensaje de correo. |
| Remitente | String | La dirección de correo electrónico del remitente. |
| P1Sender | String | Id. de correo electrónico del usuario (delegado) que envió este correo "en nombre del usuario P2 (principal)". Si el correo electrónico no lo envió el delegado, este valor es igual a P2Sender. |
| P1SenderDisplayName | String | Nombre para mostrar del usuario (delegado) que envió este correo "en nombre del usario P2 (principal)". Se representa en el encabezado del correo electrónico mediante la propiedad "OnbehalfofSenderDisplayName". |
| P1SenderDomain | String | Dominio de correo electrónico del usuario (delegado) que envió este correo "en nombre del usuario P2 (principal)". Si el correo electrónico no lo envió el delegado, este valor es igual a P2SenderDomain. |
| P2Sender | String | Correo electrónico del usuario (principal) en nombre del que se envió este correo electrónico. |
| P2SenderDisplayName | String | Nombre para mostrar del usuario (principal) en nombre del que se envió este correo electrónico. Si el correo electrónico no lo envío el delegado, representa el nombre para mostrar del remitente. |
| P2SenderDomain | String | Dominio de correo electrónico del usuario (principal) en nombre del que se envió este correo electrónico. Si el correo electrónico no lo envió el delegado, representa el dominio del remitente. |
| SenderIP | String | Dirección IP del remitente. |
| ReceivedDate | DateTime | Fecha de recepción de este mensaje. |
| NetworkMessageId | ¿GUID? | Id. de mensaje de red de este mensaje de correo. |
| InternetMessageId | String | Id. de mensaje de Internet de este mensaje de correo. |
| Asunto | String | Asunto del mensaje de correo. |
| BodyFingerprintBin1<br>BodyFingerprintBin2<br>BodyFingerprintBin3<br>BodyFingerprintBin4<br>BodyFingerprintBin5 | ¿UInt? | Microsoft Defender para Office 365 lo usa para buscar mensajes de correo coincidentes o similares. |
| AntispamDirection | ¿Enumeración? | Direccionalidad de este mensaje de correo. Valores posibles:<li>Desconocido<li>Entrada<li>Salida<li>Intraorg (interno) |
| DeliveryAction | ¿Enumeración? | Acción de entrega de este mensaje de correo. Valores posibles:<li>Desconocido<li>DeliveredAsSpam<li>Delivered (Entregado)<li>Bloqueado<li>Reemplazados |
| DeliveryLocation | ¿Enumeración? | Ubicación de entrega de este mensaje de correo. Valores posibles:<li>Desconocido<li>Bandeja de entrada<li>JunkFolder<li>DeletedFolder<li>Cuarentena<li>Externo<li>Con error<li>Dropped<li>Forwarded |
| Idioma | String | Idioma en que está escrito el contenido del mensaje de correo. |
| ThreatDetectionMethods | IList&lt;cadena&gt; | Lista de métodos de detección de amenazas aplicados en este correo. |
|

Identificadores seguros de una entidad de mensaje de correo:
- NetworkMessageId + Recipient

## <a name="submission-mail"></a>Correo de envío

*Nombre de entidad: SubmissionMail*

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| Tipo | String | 'SubmissionMail' |
| SubmissionId | ¿GUID? | Id. de envío. |
| SubmissionDate | DateTime? | Fecha y hora de notificación de este envío. |
| Remitente | String | Dirección de correo electrónico del remitente. |
| NetworkMessageId | ¿GUID? | Id. del mensaje de red del correo electrónico al que pertenece el envío. |
| Timestamp | DateTime? | Marca de tiempo de recepción del mensaje (correo). |
| Recipient | String | Destinatario del correo. |
| Remitente | String | Remitente del correo. |
| SenderIp | String | Dirección IP del remitente. |
| Asunto | String | Asunto del correo de envío. |
| ReportType | String | Tipo de envío de la instancia especificada. Se asigna a Correo no deseado, Cebo, Malware o Correo deseado. |
|

Identificadores seguros de una entidad SubmissionMail:
- SubmissionId, Submitter, NetworkMessageId, Recipient

## <a name="cloud-application-identifiers"></a>Identificadores de aplicación en la nube

En la lista siguiente se definen los identificadores de las aplicaciones en la nube conocidas. El valor de identificador de aplicación se usa como un identificador de entidad de [aplicación en la nube](#cloud-application).

|Identificador de la aplicación|Nombre|
|------|----|
|10026|DocuSign|
|10395|Anaplan|
|10489|Box|
|10549|Cisco Webex|
|10618|Atlassian|
|10915|cornerstone ondemand|
|10921|Zendesk|
|10980|Okta|
|11042|Jive Software|
|11114|Salesforce|
|11161|Office 365|
|11162|Microsoft OneNote Online|
|11394|Microsoft Online Services|
|11522|Yammer|
|11599|Amazon Web Services|
|11627|Dropbox|
|11 713|Expensify|
|11770|G Suite|
|12005|SuccessFactors|
|12260|Microsoft Azure|
|12275|Workday|
|13843|LivePerson|
|13979|Concur|
|14509|ServiceNow|
|15570|Tableau|
|15600|Microsoft OneDrive para la Empresa|
|15782|Citrix ShareFile|
|17152|Amazon|
|17865|Ariba Inc|
|18432|Zscaler|
|19688|Xactly|
|20595|Microsoft Cloud App Security|
|20892|Microsoft SharePoint Online|
|20893|Microsoft Exchange Online|
|20940|Active Directory|
|20941|Adallom CPanel|
|22110|Google Cloud Platform|
|22930|Gmail|
|23004|Autodesk Fusion Lifecycle|
|23043|Slack|
|23233|Microsoft Office Online|
|25275|Microsoft Skype Empresarial|
|25988|Google Docs|
|26055|Centro de administración de Microsoft Office 365.|
|26060|OPSWAT Gears|
|26061|Microsoft Word Online|
|26062|Microsoft PowerPoint Online|
|26063|Microsoft Excel Online|
|26069|Google Drive|
|26206|Workiva|
|26311|Microsoft Dynamics|
|26318|Microsoft Azure AD|
|26320|Microsoft Office Sway|
|26321|Microsoft Delve|
|26324|Microsoft Power BI|
|27548|Microsoft Forms|
|27592|Microsoft Flow|
|27593|Microsoft PowerApps|
|28353|Workplace by Facebook|
|28373|Emulador de proxy de CAS|
|28375|Microsoft Teams|
|32780|Microsoft Dynamics 365|
|33626|Google|
|34127|Microsoft AppSource|
|34667|HighQ|
|35395|Microsoft Dynamics Talent|
|

## <a name="next-steps"></a>Pasos siguientes

En este documento ha obtenido información acerca de la estructura de la entidad, los identificadores y el esquema de Azure Sentinel.

Obtenga más información sobre las [entidades](entities-in-azure-sentinel.md) y [la asignación de entidades](map-data-fields-to-entities.md). 
