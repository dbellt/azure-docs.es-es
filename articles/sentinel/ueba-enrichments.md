---
title: Referencia de características enriquecidas de UEBA de Azure Sentinel | Microsoft Docs
description: En este artículo se muestran las características enriquecidas generadas por el análisis de comportamiento de entidades de Azure Sentinel.
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
ms.date: 01/04/2021
ms.author: yelevin
ms.openlocfilehash: d393b325d30e2136ac08741c4b5010130535300a
ms.sourcegitcommit: 19dfdfa85e92c6a34933bdd54a7c94e8b00eacfd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109664625"
---
# <a name="azure-sentinel-ueba-enrichments-reference"></a>Referencia de características enriquecidas de UEBA de Azure Sentinel

En este artículo se describe la tabla **Análisis de comportamiento** que se encuentra en las [páginas de detalles de la entidad](identify-threats-with-entity-behavior-analytics.md#how-to-use-entity-pages), así como otras características enriquecidas de entidades que puede usar para centrar y mejorar las investigaciones de incidentes de seguridad.

Las tablas [Información de usuarios](#user-insights-table) e [Información de dispositivos](#device-insights-table) contienen información de entidades de Active Directory/Azure AD y orígenes de inteligencia sobre amenazas de Microsoft.

Otras tablas, que se describen en [tablas de información de actividades](#activity-insights-tables), contienen información de las entidades basada en los perfiles de comportamiento creados por el análisis de comportamiento de entidades de Azure Sentinel. 

<a name="baseline-explained"></a>Las actividades del usuario se analizan con respecto a una base de referencia que se compila dinámicamente cada vez que se usa. Cada actividad tiene definido un período de retrospectiva a partir del cual se deriva la base de referencia dinámica. Este período de retrospectiva se especifica en la columna [**Base de referencia**](#activity-insights-tables) de esta tabla.

> [!NOTE] 
> El campo **Nombre de la característica enriquecida** de las tablas [Información de usuarios](#user-insights-table), [Información de dispositivos](#device-insights-table) e [Información de actividades](#activity-insights-tables) muestra dos filas de información. 
> 
> La primera, en **negrita**, está el "nombre descriptivo" de la característica enriquecida. La segunda *(en cursiva y entre paréntesis)* es el nombre del campo de la característica enriquecida tal y como se almacena en la [**tabla de análisis de comportamiento**](#behavior-analytics-table).

## <a name="behavior-analytics-table"></a>Tabla de análisis de comportamiento

En la tabla siguiente se describen los datos de análisis de comportamiento que se muestran en cada [página de detalles de entidad](identify-threats-with-entity-behavior-analytics.md#how-to-use-entity-pages) de Azure Sentinel.

| Campo                     | Descripción                                                         |
|---------------------------|---------------------------------------------------------------------|
| **TenantId**                  | Número de id. único del inquilino                                      |
| **SourceRecordId**            | Número de id. único del evento de EBA                                   |
| **TimeGenerated**             | Marca de tiempo de la repetición de la actividad                              |
| **TimeProcessed**             | Marca de tiempo del procesamiento de la actividad por parte del motor de EBA            |
| **ActivityType**              | Categoría de alto nivel de la actividad                                 |
| **ActionType**                | Nombre normalizado de la actividad                                     |
| **UserName**                  | Nombre de usuario del usuario que inició la actividad                    |
| **UserPrincipalName**         | Nombre de usuario completo del usuario que inició la actividad               |
| **EventSource**               | Origen de datos que proporcionó el evento original                        |
| **SourceIPAddress**           | Dirección IP desde la que se inició la actividad                        |
| **SourceIPLocation**          | País desde el que se inició la actividad, enriquecido a partir de la dirección IP |
| **SourceDevice**              | Nombre de host del dispositivo que inició la actividad                  |
| **DestinationIPAddress**      | Dirección IP del destino de la actividad                            |
| **DestinationIPLocation**     | País del destino de la actividad, enriquecido a partir de la dirección IP     |
| **DestinationDevice**         | Nombre del dispositivo de destino                                           |
| **UsersInsights**         | Enriquecimientos contextuales de usuarios implicados                            |
| **DevicesInsights**       | Enriquecimientos contextuales de dispositivos implicados                          |
| **ActivityInsights**      | Análisis contextual de la actividad basada en la generación de perfiles              |
| **InvestigationPriority** | Puntuación de anomalías, entre 0 y 10 (0=benigno, 10=muy anómalo)         |
|

## <a name="user-insights-table"></a>Tabla de información de usuario

En la tabla siguiente se describen los elementos <?> en la tabla **Información de usuarios** de Azure Sentinel

| Nombre de la característica enriquecida | Descripción | Valor de ejemplo |
| --- | --- | --- | --- |
| **Nombre para mostrar de la cuenta**<br>*(AccountDisplayName)* | Nombre para mostrar de la cuenta del usuario. | Admin, Hayden Cook |
| **Dominio de cuenta**<br>*(AccountDomain)* | Nombre de dominio de la cuenta del usuario. |  |
| **Identificador del objeto de la cuenta**<br>*(AccountObjectID)* | El identificador del objeto de la cuenta del usuario. | a58df659-5cab-446c-9dd0-5a3af20ce1c2 |
| **Radio de explosión**<br>*(BlastRadius)* | El radio de explosión se calcula en función de varios factores: la posición del usuario en el árbol de la organización y los roles y permisos del usuario de Azure Active Directory. | Bajo, medio, alto |
| **Cuenta inactiva**<br>*(IsDormantAccount)* | La cuenta no se ha usado durante los últimos 180 días. | True, False |
| **Administrador local**<br>*(IsLocalAdmin)* | La cuenta tiene privilegios de administrador local. | True, False |
| **Cuenta nueva**<br>*(IsNewAccount)* | La cuenta se creó en los últimos 30 días. | True, False |
| **SID local**<br>*(OnPremisesSID)* | El SID local del usuario relacionado con la acción. | S-1-5-21-1112946627-1321165628-2437342228-1103 |
|

## <a name="device-insights-table"></a>Tabla de información de dispositivos

| Nombre de la característica enriquecida | Descripción | Valor de ejemplo |
| --- | --- | --- | --- |
| **Browser**<br>*(Explorador)* | Explorador usado en la acción. | Edge, Chrome |
| **Familia de dispositivos**<br>*(DeviceFamily)* | Familia de dispositivos usada en la acción. | Windows |
| **Tipo de dispositivo**<br>*(DeviceType)* | Tipo de dispositivo de cliente usado en la acción. | Escritorio |
| **ISP**<br>*(ISP)* | Proveedor de servicios de Internet usado en la acción. |  |
| **Sistema operativo**<br>*(OperatingSystem)* | Sistema operativo usado en la acción. | Windows 10 |
| **Descripción del indicador de información sobre amenazas**<br>*(ThreatIntelIndicatorDescription)* | Descripción del indicador de amenazas observado resuelto a partir de la dirección IP usada en la acción. | Host is member of botnet: azorult |
| **Tipo de indicador de información sobre amenazas**<br>*(ThreatIntelIndicatorType)* | El tipo de indicador de amenazas resuelto a partir de la dirección IP usada en la acción. | Botnet, C2, CryptoMining, Darknet, Ddos, MaliciousUrl, Malware, Phishing, Proxy, PUA, Watchlist |
| **Agente de usuario**<br>*(UserAgent)* | Agente de usuario usaso en la acción. | Biblioteca de cliente de Microsoft Azure Graph 1.0,<br>Swagger-Codegen/1.4.0.0/csharp,<br>EvoSTS |
| **Familia de agentes de usuario**<br>*(UserAgentFamily)* | Familia de agentes de usuario usada en la acción. | Chrome, Edge, Firefox |
|

## <a name="activity-insights-tables"></a>Tablas de información de actividades

### <a name="action-performed"></a>Acción realizada

| Nombre de la característica enriquecida | [Base de referencia](#baseline-explained) (días) | Descripción | Valor de ejemplo |
| --- | --- | --- | --- |
| **Primera vez que el usuario realizó la acción**<br>*(FirstTimeUserPerformedAction)* | 180 | El usuario realizó la acción por primera vez. | True, False |
| **Acción infrecuente realizada por el usuario**<br>*(ActionUncommonlyPerformedByUser)* | 10 | Acción que el usuario no realiza normalmente. | True, False |
| **Acción infrecuente realizada entre elementos del mismo nivel**<br>*(ActionUncommonlyPerformedAmongPeers)* | 180 | La acción no se realiza normalmente entre elementos del mismo nivel del usuario. | True, False |
| **Primera vez que se realiza la acción en el inquilino**<br>*(FirstTimeActionPerformedInTenant)* | 180 | Cualquier persona de la organización realizó la acción por primera vez. | True, False |
| **Acción infrecuente realizada en el inquilino**<br>*(ActionUncommonlyPerformedInTenant)* | 180 | La acción no se realiza normalmente en la organización. | True, False |
|

### <a name="app-used"></a>Aplicación usada

| Nombre de la característica enriquecida | [Base de referencia](#baseline-explained) (días) | Descripción | Valor de ejemplo |
| --- | --- | --- | --- |
| **Primera vez que el usuario usó la aplicación**<br>*(FirstTimeUserUsedApp)* | 180 | El usuario usó la aplicación por primera vez. | True, False |
| **Aplicación poco utilizada por el usuario**<br>*(AppUncommonlyUsedByUser)* | 10 | El usuario no suele usar la aplicación. | True, False |
| **Aplicación poco usada entre elementos del mismo nivel**<br>*(AppUncommonlyUsedAmongPeers)* | 180 | La aplicación no se usa normalmente entre los elementos del mismo nivel del usuario. | True, False |
| **Primera vez que se observa la aplicación en el inquilino**<br>*(FirstTimeAppObservedInTenant)* | 180 | La aplicación se observó por primera vez en la organización. | True, False |
| **Aplicación poco usada en el inquilino**<br>*(AppUncommonlyUsedInTenant)* | 180 | La aplicación no se usa habitualmente en la organización. | True, False |
| 

### <a name="browser-used"></a>Explorador usado

| Nombre de la característica enriquecida | [Base de referencia](#baseline-explained) (días) | Descripción | Valor de ejemplo |
| --- | --- | --- | --- |
| **Primera vez que el usuario se conecta a través del explorador**<br>*(FirstTimeUserConnectedViaBrowser)* | 30 | La primera vez que el usuario observó el explorador. | True, False |
| **Explorador poco utilizado por el usuario**<br>*(BrowserUncommonlyUsedByUser)* | 10 | El usuario no suele usar el explorador. | True, False |
| **Explorador poco usado entre elementos del mismo nivel**<br>*(BrowserUncommonlyUsedAmongPeers)* | 30 | El explorador no se usa normalmente entre los elementos del mismo nivel del usuario. | True, False |
| **Primera vez que se observa el explorador en el inquilino**<br>*(FirstTimeBrowserObservedInTenant)* | 30 | El explorador se observó por primera vez en la organización. | True, False |
| **Explorador poco usado en el inquilino**<br>*(BrowserUncommonlyUsedInTenant)* | 30 | El explorador no se usa habitualmente en la organización. | True, False |
| 

### <a name="country-connected-from"></a>País de origen de la conexión

| Nombre de la característica enriquecida | [Base de referencia](#baseline-explained) (días) | Descripción | Valor de ejemplo |
| --- | --- | --- | --- |
| **Primera vez que el usuario se conecta desde el país**<br>*(FirstTimeUserConnectedFromCountry)* | 90 | El usuario conectó por primera vez la ubicación geográfica, tal y como se resolvió a partir de la dirección IP. | True, False |
| **País con conexión infrecuente del usuario**<br>*(CountryUncommonlyConnectedFromByUser)* | 10 | El usuario no suele conectar la ubicación geográfica, tal y como se resolvió a partir de la dirección IP. | True, False |
| **País de conexión infrecuente entre elementos del mismo nivel**<br>*(CountryUncommonlyConnectedFromAmongPeers)* | 90 | La ubicación geográfica, tal y como se resolvió desde la dirección IP, no se conecta normalmente entre los pares del usuario. | True, False |
| **Primera conexión con origen en el país observada en el inquilino**<br>*(FirstTimeConnectionFromCountryObservedInTenant)* | 90 | Primera conexión con origen en el país por cualquier persona de la organización. | True, False |
| **Conexión con origen en el país infrecuente desde en el inquilino**<br>*(CountryUncommonlyConnectedFromInTenant)* | 90 | La organización no suele conectar la ubicación geográfica, tal y como se resolvió a partir de la dirección IP. | True, False |
| 

### <a name="device-used-to-connect"></a>Dispositivo usado para conectarse

| Nombre de la característica enriquecida | [Base de referencia](#baseline-explained) (días) | Descripción | Valor de ejemplo |
| --- | --- | --- | --- |
| **Primera vez que el usuario se conecta desde el dispositivo**<br>*(FirstTimeUserConnectedFromDevice)* | 30 | La primera vez que el usuario conectó el dispositivo de origen. | True, False |
| **Dispositivo poco usado por el usuario**<br>*(DeviceUncommonlyUsedByUser)* | 10 | El usuario no suele usar el dispositivo. | True, False |
| **Dispositivo poco usado entre elementos del mismo nivel**<br>*(DeviceUncommonlyUsedAmongPeers)* | 180 | El dispositivo no se usa normalmente entre los elementos del mismo nivel del usuario. | True, False |
| **Dispositivo observado por primera vez en el inquilino**<br>*(FirstTimeDeviceObservedInTenant)* | 30 | El dispositivo se observó por primera vez en la organización. | True, False |
| **Dispositivo poco usado en el inquilino**<br>*(DeviceUncommonlyUsedInTenant)* | 180 | El dispositivo no se usa habitualmente en la organización. | True, False |
| 

### <a name="other-device-related"></a>Otros relacionados con el dispositivo

| Nombre de la característica enriquecida | [Base de referencia](#baseline-explained) (días) | Descripción | Valor de ejemplo |
| --- | --- | --- | --- |
| **Primera vez que el usuario inició sesión en el dispositivo**<br>*(FirstTimeUserLoggedOnToDevice)* | 180 | El usuario conectó el dispositivo de destino por primera vez. | True, False |
| **Familia de dispositivos poco usada en el inquilino**<br>*(DeviceFamilyUncommonlyUsedInTenant)* | 30 | La familia de dispositivos no se usa habitualmente en la organización. | True, False |
| 

### <a name="internet-service-provider-used-to-connect"></a>Proveedor de servicios de Internet que se usa para conectarse

| Nombre de la característica enriquecida | [Base de referencia](#baseline-explained) (días) | Descripción | Valor de ejemplo |
| --- | --- | --- | --- |
| **Primera vez que el usuario se conecta a través de ISP**<br>*(FirstTimeUserConnectedViaISP)* | 30 | La primera vez que el usuario observó el ISP. | True, False |
| **ISP poco usado por el usuario**<br>*(ISPUncommonlyUsedByUser)* | 10 | El usuario no suele usar el ISP. | True, False |
| **ISP poco usado entre elementos del mismo nivel**<br>*(ISPUncommonlyUsedAmongPeers)* | 30 | El ISP no se usa normalmente entre los elementos del mismo nivel del usuario. | True, False |
| **Primera conexión a través de ISP en el inquilino**<br>*(FirstTimeConnectionViaISPInTenant)* | 30 | El ISP se observó por primera vez en la organización. | True, False |
| **ISP poco usado en el inquilino**<br>*(ISPUncommonlyUsedInTenant)* | 30 | El ISP no se utiliza normalmente en la organización. | True, False |
| 

### <a name="resource-accessed"></a>Recurso al que se accede

| Nombre de la característica enriquecida | [Base de referencia](#baseline-explained) (días) | Descripción | Valor de ejemplo |
| --- | --- | --- | --- |
| **Recurso al que se accede por primera vez**<br>*(FirstTimeUserAccessedResource)* | 180 | El usuario ha tenido acceso al recurso por primera vez. | True, False |
| **Recurso al que accede poco el usuario**<br>*(ResourceUncommonlyAccessedByUser)* | 10 | El usuario no suele tener acceso al recurso. | True, False |
| **Recurso al que tienen poco acceso los elementos del mismo nivel**<br>*(ResourceUncommonlyAccessedAmongPeers)* | 180 | Normalmente no se accede al recurso entre los elementos del mismo nivel del usuario. | True, False |
| **Primera vez que se tiene acceso al recurso en el inquilino**<br>*(FirstTimeResourceAccessedInTenant)* | 180 | Cualquier persona de la organización ha tenido acceso al recurso por primera vez. | True, False |
| **Recurso al que accede poco el inquilino**<br>*(ResourceUncommonlyAccessedInTenant)* | 180 | Normalmente no se tiene acceso al recurso en la organización. | True, False |
| 

### <a name="miscellaneous"></a>Varios

| Nombre de la característica enriquecida | [Base de referencia](#baseline-explained) (días) | Descripción | Valor de ejemplo |
| --- | --- | --- | --- |
| **Última vez que el usuario realizó la acción**<br>*(LastTimeUserPerformedAction)* | 180 | Última vez que el usuario realizó la misma acción. | <Timestamp> |
| **No se ha realizado una acción similar en el pasado**<br>*(SimilarActionWasn'tPerformedInThePast)* | 30 | El usuario no realizó ninguna acción en el mismo proveedor de recursos. | True, False |
| **Ubicación de IP de origen**<br>*(SourceIPLocation)* | *N/D* | El país resuelto a partir de la IP de origen de la acción. | [Surrey, England] |
| **Gran volumen de operaciones poco frecuentes**<br>*(UncommonHighVolumeOfOperations)* | 7 | Un usuario realizó una serie de operaciones similares en el mismo proveedor | True, False |
| **Número inusual de errores de acceso condicional de Azure AD**<br>*(UnusualNumberOfAADConditionalAccessFailures)* | 5 | No se pudo autenticar un número inusual de usuarios debido a un acceso condicional | True, False |
| **Número inusual de dispositivos agregados**<br>*(UnusualNumberOfDevicesAdded)* | 5 | Un usuario agregó un número de dispositivos inusual. | True, False |
| **Número inusual de dispositivos eliminados**<br>*(UnusualNumberOfDevicesDeleted)* | 5 | Un usuario eliminó un número de dispositivos inusual. | True, False |
| **Número inusual de usuarios agregados al grupo**<br>*(UnusualNumberOfUsersAddedToGroup)* | 5 | Un usuario agregó un número inusual de usuarios a un grupo. | True, False |
|