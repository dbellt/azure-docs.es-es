---
author: memildin
ms.service: security-center
ms.topic: include
ms.date: 05/05/2021
ms.author: memildin
ms.custom: generated
ms.openlocfilehash: 285aa520f22874cc9ad93ca253086617a7363943
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108766531"
---
Hay **12** recomendaciones en esta categoría.

|Recomendación |Descripción |severity |
|---|---|---|
| La directiva del filtro de IP predeterminada debe ser Denegar. |La configuración de filtro IP necesita tener reglas definidas para tráfico permitido y denegar de forma predeterminada el resto del tráfico.<br />(Ninguna directiva relacionada) |Media |
|Los registros de diagnóstico de IoT Hub deben estar habilitados |Habilite los registros y consérvelos hasta un año. Esto le permite volver a crear seguimientos de actividad con fines de investigación cuando se produce un incidente de seguridad o se pone en peligro la red.<br />(Directiva relacionada: [los registros de diagnóstico de IoT Hub deben estar habilitados](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f383856f8-de7f-44a2-81fc-e5135b5c2aa4)) |Bajo |
|Credenciales de autenticación idénticas |Credenciales de autenticación idénticas para la instancia de IoT Hub utilizada por varios dispositivos. Esto puede indicar que hay un dispositivo ilegítimo que suplanta un dispositivo legítimo. También expone el riesgo de suplantación del dispositivo por parte de un atacante<br />(Ninguna directiva relacionada) |Alto |
|Dispositivos IoT: mensajes infrautilizados de envío del agente. |La capacidad del tamaño del mensaje del agente de IoT está actualmente infrautilizada, lo que provoca un aumento en el número de mensajes enviados. Ajuste de los intervalos de mensajes para una mejor utilización<br />(Ninguna directiva relacionada) |Bajo |
| Dispositivos IoT: el proceso auditado dejó de enviar eventos. |Este dispositivo ya no envía eventos de seguridad originados por un proceso Auditd.<br />(Ninguna directiva relacionada) |Alto |
|Dispositivos IoT: puertos abiertos en el dispositivo. |Se encontró un punto de conexión de escucha en el dispositivo.<br />(Ninguna directiva relacionada) |Media |
| Dispositivos IoT: error de validación de línea base del sistema operativo |Problemas de configuración del sistema identificados relacionados con la seguridad<br />(Ninguna directiva relacionada) |Media |
|Dispositivos IoT: se encontró una directiva de firewall permisiva en una de las cadenas. |Se encontró una directiva de firewall permitida en las cadenas de firewall principales (ENTRADA/SALIDA). La directiva debe denegar todo el tráfico de manera predeterminada y definir reglas para permitir la comunicación necesaria hacia y desde el dispositivo.<br />(Ninguna directiva relacionada) |Media |
| Dispositivos IoT: se encontró una regla de firewall permisiva en la cadena de entrada. |Se ha encontrado una regla en el firewall que contiene un patrón de permisos para una amplia gama de puertos o direcciones IP.<br />(Ninguna directiva relacionada) |Media |
| Dispositivos IoT: se encontró una regla de firewall permisiva en la cadena de salida. |Se encontró una regla en el firewall que contiene un patrón permisivo para una amplia gama de direcciones IP o puertos.<br />(Ninguna directiva relacionada) |Media |
| Dispositivos IoT: es preciso actualizar el conjunto de cifrado de TLS. |Se han detectado configuraciones de TLS no seguras. Se recomienda actualizar el conjunto de cifrado de TLS inmediatamente.<br />(Ninguna directiva relacionada) |Media |
|Intervalo IP amplio de la regla del filtro de IP. |Un intervalo IP de origen de la regla de filtro IP permitido es demasiado grande. Las reglas excesivamente permisivas podrían exponer su instancia de IoT Hub a agentes malintencionados.<br />(Ninguna directiva relacionada) |Media |
|||
