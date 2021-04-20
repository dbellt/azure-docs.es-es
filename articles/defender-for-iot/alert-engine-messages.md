---
title: Tipos y descripciones de alertas
description: Revise las descripciones de alertas de Defender para IoT.
author: shhazam-ms
manager: rkarlin
ms.author: shhazam
ms.date: 4/8/2021
ms.topic: how-to
ms.service: azure
ms.openlocfilehash: 9ef7aa388d0f25adcafec1cb4a5b38dcfb8597a1
ms.sourcegitcommit: c3739cb161a6f39a9c3d1666ba5ee946e62a7ac3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107210472"
---
# <a name="alert-types-and-descriptions"></a>Tipos y descripciones de alertas

En este artículo se describen todos los tipos de alerta que pueden generarse a partir de los motores de Defender para IoT. Las alertas aparecen en la ventana Alertas, donde puede administrar el evento de alerta. 

## <a name="policy-engine-alerts"></a>Alertas del motor de directivas

Las alertas del motor de directivas describen las desviaciones del comportamiento de red aprendido de la línea base.

| Título  | Descripción | severity |
|--|--|--|
| Uso anómalo de direcciones MAC | Se ha detectado un nuevo dispositivo de origen en la red, pero no se ha autorizado. | Minor |
| Se ha cambiado el software Beckhoff | El firmware se actualizó en un dispositivo de origen. Puede tratarse de actividades autorizadas, por ejemplo, un procedimiento de mantenimiento planeado. | Principal |
| Error de inicio de sesión en la base de datos | Se detectó un error de inicio de sesión desde un dispositivo de origen a un servidor de destino. Esto podría ser resultado de un error humano, pero también podría indicar un intento malintencionado de poner en peligro el servidor o los datos que hay en él. | Principal |
| Se ha modificado la versión de firmware de Emerson ROC | El firmware se actualizó en un dispositivo de origen. Puede tratarse de actividades autorizadas, por ejemplo, un procedimiento de mantenimiento planeado. | Principal |
| Una dirección externa de la red se ha comunicado con Internet | Un dispositivo de origen definido como parte de la red se está comunicando con direcciones de Internet. El origen no está autorizado para comunicarse con direcciones de Internet. | Crítico |
| Dispositivo de campo detectado de forma inesperada | Se ha detectado un nuevo dispositivo de origen en la red, pero no se ha autorizado. | Principal |
| Cambio de firmware detectado | El firmware se actualizó en un dispositivo de origen. Puede tratarse de actividades autorizadas, por ejemplo, un procedimiento de mantenimiento planeado. | Principal |
| Se ha cambiado la versión de firmware | El firmware se actualizó en un dispositivo de origen. Puede tratarse de actividades autorizadas, por ejemplo, un procedimiento de mantenimiento planeado. | Principal |
| Operación no autorizada de Foxboro I/A | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Error de inicio de sesión de FTP | Se detectó un error de inicio de sesión desde un dispositivo de origen a un servidor de destino. Esto podría ser resultado de un error humano, pero también podría indicar un intento malintencionado de poner en peligro el servidor o los datos que hay en él. | Principal |
| El código de función generó una excepción no autorizada | Un dispositivo de origen (subordinado) ha devuelto una excepción a un dispositivo de destino (maestro). | Principal |
| Configuración del tipo de mensaje de GOOSE | Se ha modificado la configuración del mensaje (identificado por el id. de protocolo) en un dispositivo de origen. | Advertencia |
| La versión del firmware de Honeywell ha cambiado. | El firmware se actualizó en un dispositivo de origen. Puede tratarse de actividades autorizadas, por ejemplo, un procedimiento de mantenimiento planeado. | Principal |
| Comunicación HTTP ilícita. | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Se ha detectado un acceso a Internet | Un dispositivo de origen definido como parte de la red se está comunicando con direcciones de Internet. El origen no está autorizado para comunicarse con direcciones de Internet. | Principal |
| La versión del firmware de Mitsubishi ha cambiado. | El firmware se actualizó en un dispositivo de origen. Puede tratarse de actividades autorizadas, por ejemplo, un procedimiento de mantenimiento planeado. | Principal |
| Infracción del intervalo de direcciones de Modbus | Un dispositivo maestro ha solicitado acceso a una nueva dirección de memoria subordinada. | Principal |
| La versión del firmware de Modbus ha cambiado. | El firmware se actualizó en un dispositivo de origen. Puede tratarse de actividades autorizadas, por ejemplo, un procedimiento de mantenimiento planeado. | Principal |
| Nueva actividad detectada: clase CIP | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: servicio de clase CIP | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: comando CIP PCCC | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: símbolo CIP | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: conexión de E/S de EtherNet/IP | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: comando de protocolo de EtherNet/IP | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: código de mensaje de GSM | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: códigos de comando de LonTalk | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva detección de puertos | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Advertencia |
| Nueva actividad detectada: variable de red de LonTalk | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: solicitud de datos de Ovation | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: comando de lectura/escritura (grupo de índices de AMS) | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: comando de lectura/escritura (desplazamiento de índices de AMS) | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: tipo de mensaje de DeltaV no autorizado | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: operación de DeltaV ROC no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: tipo de mensaje de RPC no autorizado | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: invocación de procedimiento RPC no autorizado | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: uso del comando de protocolo de AMS | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: uso de comandos de Siemens SICAM | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: uso del comando de protocolo de Suitelink | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: uso de sesiones de protocolo de Suitelink | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Nueva actividad detectada: uso del comando de Yokogawa VNetIP | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Se ha detectado un nuevo recurso | Se ha detectado un nuevo dispositivo de origen en la red, pero no se ha autorizado. | Principal |
| Nueva configuración del dispositivo LLDP | Se ha detectado un nuevo dispositivo de origen en la red, pero no se ha autorizado. | Principal |
| Nueva detección de puertos | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Advertencia |
| Comando no autorizado de Omron FINS | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Se ha modificado el firmware de S7 Plus PLC | El firmware se actualizó en un dispositivo de origen. Puede tratarse de actividades autorizadas, por ejemplo, un procedimiento de mantenimiento planeado. | Principal |
| Configuración del tipo de mensaje de valores de ejemplo | Se ha modificado la configuración del mensaje (identificado por el id. de protocolo) en un dispositivo de origen. | Advertencia |
| Sospecha de un examen de integridad no válido | Se ha detectado un análisis en un dispositivo de origen de DNP3 (estación remota). Este examen no se ha autorizado como tráfico aprendido de la red. | Principal |
| Comando no autorizado de Toshiba Computer Link | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Minor |
| Operación de archivo ABB Totalflow no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Operación de registro de ABB Totalflow no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Acceso no autorizado al bloque de datos de Siemens S7 | Un dispositivo de origen intentó acceder a un recurso en otro dispositivo. Un intento de acceso a este recurso entre estos dos dispositivos no se ha autorizado como tráfico aprendido en la red. | Advertencia |
| Acceso no autorizado al objeto de Siemens S7 Plus | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Acceso no autorizado a la etiqueta Wonderware | Un dispositivo de origen intentó acceder a un recurso en otro dispositivo. Un intento de acceso a este recurso entre estos dos dispositivos no se ha autorizado como tráfico aprendido en la red. | Principal |
| Acceso no autorizado a objetos BACNet | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Ruta BACNet no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Inicio de sesión a la base de datos no autorizado | Se ha detectado un intento de inicio de sesión entre un cliente de origen y un servidor de destino. La comunicación entre estos dispositivos no se ha autorizado como tráfico aprendido en la red. | Principal |
| Operación de base de datos no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Operación de Emerson ROC no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Acceso no autorizado a archivos de GE SRTP | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Comando de protocolo de GE SRTP no autorizado | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Operación de memoria del sistema de GE SRTP no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Actividad HTTP no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Servidor HTTP no autorizado | Se ha detectado una aplicación no autorizada en un dispositivo de origen. La aplicación no se ha autorizado como aplicación aprendida en la red. | Principal |
| Acción SOAP HTTP no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Agente de usuario HTTP no autorizado | Se ha detectado una aplicación no autorizada en un dispositivo de origen. La aplicación no se ha autorizado como aplicación aprendida en la red. | Principal |
| Se ha detectado una conexión a Internet no autorizada. | Un dispositivo de origen definido como parte de la red se está comunicando con direcciones de Internet. El origen no está autorizado para comunicarse con direcciones de Internet. | Crítico |
| Comando MELSEC de Mitsubishi no autorizado | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Acceso no autorizado a programas MMS | Un dispositivo de origen intentó acceder a un recurso en otro dispositivo. Un intento de acceso a este recurso entre estos dos dispositivos no se ha autorizado como tráfico aprendido en la red. | Principal |
| Servicio MMS no autorizado | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Conexión de multidifusión/difusión no autorizada | Se ha detectado una conexión de multidifusión/difusión entre un dispositivo de origen y otros dispositivos. La comunicación multidifusión/difusión no está autorizada. | Crítico |
| Consulta de nombre no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Actividad de OPC UA no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Solicitud o respuesta de OPC UA no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Una regla definida por el usuario ha detectado una operación no autorizada | Se ha detectado tráfico entre dos dispositivos. Esta actividad no se autoriza en función de una regla de alerta personalizada definida por un usuario. | Principal |
| Lectura no autorizada de la configuración de PLC | El dispositivo de origen no se ha definido como un dispositivo de programación pero realizó una operación de lectura y escritura en un controlador de destino. Los dispositivos de programación son los únicos que deben realizar cambios de programación. Es posible que se haya instalado una aplicación de programación en este dispositivo. | Advertencia |
| Escritura no autorizada de la configuración de PLC | El dispositivo de origen envió un comando para leer el programa de un controlador de destino o escribir en él. Esta actividad no se ha mostrado anteriormente. | Principal |
| Carga de programa PLC no autorizada | El dispositivo de origen envió un comando para leer el programa de un controlador de destino o escribir en él. Esta actividad no se ha mostrado anteriormente. | Principal |
| Programación de PLC no autorizada | El dispositivo de origen no se ha definido como un dispositivo de programación pero realizó una operación de lectura y escritura en un controlador de destino. Los dispositivos de programación son los únicos que deben realizar cambios de programación. Es posible que se haya instalado una aplicación de programación en este dispositivo. | Crítico |
| Tipo de marco Profinet no autorizado | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Comando SAIA S-Bus no autorizado | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Ejecución no autorizada de la función de control de Siemens S7 | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Ejecución no autorizada de la función definida por el usuario de Siemens S7 | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Acceso no autorizado al bloque de Siemens S7 Plus | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Operación no autorizada de Siemens S7 Plus | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Inicio de sesión de SMB no autorizado | Se ha detectado un intento de inicio de sesión entre un cliente de origen y un servidor de destino. La comunicación entre estos dispositivos no se ha autorizado como tráfico aprendido en la red. | Principal |
| Operación SNMP no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Acceso a SSH no autorizado | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Proceso de Windows no autorizado | Se ha detectado una aplicación no autorizada en un dispositivo de origen. La aplicación no se ha autorizado como aplicación aprendida en la red. | Principal |
| Servicio de Windows no autorizado | Se ha detectado una aplicación no autorizada en un dispositivo de origen. La aplicación no se ha autorizado como aplicación aprendida en la red. | Principal |
| Una regla definida por el usuario ha detectado una operación no autorizada | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros infringe una regla definida por el usuario | Principal |
| Extensión de Modbus Schneider Electric no permitida | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Uso no permitido de tipos ASDU | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Uso no permitido del código de la función DNP3 | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |
| Uso no permitido de la indicación interna (IIN) | Un dispositivo de origen de DNP3 (estación remota) ha comunicado una indicación interna (IIN) que no ha autorizado como tráfico aprendido en la red. | Principal |
| Uso no permitido del código de la función de Modbus | Se han detectado nuevos parámetros de tráfico. Esta combinación de parámetros no se ha autorizado como tráfico aprendido en la red. La siguiente combinación no está autorizada. | Principal |

## <a name="anomaly-engine-alerts"></a>Alertas del motor de anomalías

| Título | Descripción | severity |
|--|--|--|
| Patrón de excepción anómalo en dispositivo secundario | Se ha detectado un número excesivo de errores en un dispositivo de origen. Esto puede ser resultado de un problema operativo. | Minor |
| Longitud de encabezado HTTP anómala | El dispositivo de origen ha enviado un mensaje anómalo. Esto puede indicar un intento de atacar el dispositivo de destino. | Crítico |
| Número anómalo de parámetros en el encabezado HTTP | El dispositivo de origen ha enviado un mensaje anómalo. Esto puede indicar un intento de atacar el dispositivo de destino. | Crítico |
| Comportamiento periódico anómalo en el canal de comunicación | Se ha detectado un cambio en la frecuencia de comunicación entre el dispositivo de origen y el de destino. | Minor |
| Finalización anómala de aplicaciones | Se ha detectado un número excesivo de comandos de detención en un dispositivo de origen. Esto puede ser el resultado de un problema operativo o un intento de manipular el dispositivo. | Principal |
| Ancho de banda de tráfico anómalo | Se ha detectado un ancho de banda anómalo en un canal. El ancho de banda parece ser significativamente inferior o superior al detectado previamente. Para más información, trabaje con el widget de ancho de banda total. | Advertencia |
| Ancho de banda de tráfico anómalo entre dispositivos | Se ha detectado un ancho de banda anómalo en un canal. El ancho de banda parece ser significativamente inferior o superior al detectado previamente. Para más información, trabaje con el widget de ancho de banda total. | Advertencia |
| Se detectó un análisis de direcciones | Se ha detectado un dispositivo de origen en el análisis de los dispositivos de red. Este dispositivo no se ha autorizado como un dispositivo de análisis de redes. | Crítico |
| Examen de direcciones ARP detectado | Se ha detectado un dispositivo de origen en el análisis de los dispositivos de red mediante el protocolo de resolución de direcciones (ARP). Esta dirección de dispositivo no se ha autorizado como dirección de análisis de ARP válida. | Crítico |
| Examen de direcciones ARP detectado | Se ha detectado un dispositivo de origen en el análisis de los dispositivos de red mediante el protocolo de resolución de direcciones (ARP). Esta dirección de dispositivo no se ha autorizado como dirección de análisis de ARP válida. | Crítico |
| Suplantación de identidad de ARP | Se ha detectado una cantidad anómala de paquetes en la red. Esto podría indicar un ataque, por ejemplo, un ataque de suplantación de identidad de ARP o un ataque "flood" de congestión del servidor de ICMP. | Advertencia |
| Intentos de inicio de sesión excesivos | Se ha detectado un dispositivo de origen que realiza demasiados intentos de inicio de sesión en un servidor de destino. Puede tratarse de un ataque por fuerza bruta. Puede que un actor malintencionado haya puesto en peligro el servidor. | Crítico |
| Número de sesiones excesivo | Se ha detectado un dispositivo de origen que realiza demasiados intentos de inicio de sesión en un servidor de destino. Puede tratarse de un ataque por fuerza bruta. Puede que un actor malintencionado haya puesto en peligro el servidor. | Crítico |
| Velocidad de reinicio excesiva de una estación remota | Se ha detectado un número excesivo de comandos de reinicio en un dispositivo de origen. Esto puede ser el resultado de un problema operativo o un intento de manipular el dispositivo. | Principal |
| Intentos excesivos de inicio de sesión en SMB | Se ha detectado un dispositivo de origen que realiza demasiados intentos de inicio de sesión en un servidor de destino. Puede tratarse de un ataque por fuerza bruta. Puede que un actor malintencionado haya puesto en peligro el servidor. | Crítico |
| Ataque "flood" de congestión del servidor de ICMP. | Se ha detectado una cantidad anómala de paquetes en la red. Esto podría indicar un ataque, por ejemplo, un ataque de suplantación de identidad de ARP o un ataque "flood" de congestión del servidor de ICMP. | Advertencia |
| Contenido de encabezado HTTP no válido | El dispositivo de origen ha iniciado una solicitud no válida. | Crítico |
| Canal de comunicación inactivo | Un canal de comunicación entre dos dispositivos estuvo inactivo durante un período en el que normalmente se detecta actividad. Esto podría indicar que se ha cambiado el programa que genera este tráfico o que el programa no está disponible. Se recomienda revisar la configuración del programa instalado y comprobar que está configurado correctamente. | Advertencia |
| Se ha detectado un examen de direcciones de larga duración | Se ha detectado un dispositivo de origen en el análisis de los dispositivos de red. Este dispositivo no se ha autorizado como un dispositivo de análisis de redes. | Crítico |
| Intento de averiguación de contraseña detectado | Se ha detectado un dispositivo de origen que realiza demasiados intentos de inicio de sesión en un servidor de destino. Puede tratarse de un ataque por fuerza bruta. Puede que un actor malintencionado haya puesto en peligro el servidor. | Crítico |
| Se ha detectado un examen de PLC | Se ha detectado un dispositivo de origen en el análisis de los dispositivos de red. Este dispositivo no se ha autorizado como un dispositivo de análisis de redes. | Crítico |
| Se ha detectado un examen de puertos | Se ha detectado un dispositivo de origen en el análisis de los dispositivos de red. Este dispositivo no se ha autorizado como un dispositivo de análisis de redes. | Crítico |
| Longitud de mensaje inesperada | El dispositivo de origen ha enviado un mensaje anómalo. Esto puede indicar un intento de atacar el dispositivo de destino. | Crítico |
| Tráfico inesperado para el puerto estándar | Se ha detectado tráfico en un dispositivo mediante un puerto reservado para otro protocolo. | Principal |

## <a name="protocol-violation-engine-alerts"></a>Alertas del motor de infracciones de protocolo

| Título | Descripción | severity |
|--|--|--|
| Demasiados paquetes con formato incorrecto en una única sesión | Se ha enviado un número anómalo de paquetes con formato incorrecto desde el dispositivo de origen al dispositivo de destino. Esto puede indicar comunicaciones erróneas o un intento de manipular el dispositivo de destino. | Principal |
| Actualización de firmware | Un dispositivo de origen envió un comando para actualizar el firmware en un dispositivo de destino. Compruebe que la programación, la configuración y las actualizaciones de firmware recientes realizadas en el dispositivo de destino sean válidas. | Advertencia |
| El código de función no es compatible con la estación remota | El dispositivo de destino ha recibido una solicitud no válida. | Principal |
| Mensaje de BACNet no válido | El dispositivo de origen ha iniciado una solicitud no válida. | Principal |
| Intento de conexión no válida en el puerto 0 | Un dispositivo de origen intentó conectarse al dispositivo de destino en el número de puerto cero (0). En TCP, el puerto 0 está reservado y no se puede usar. En el caso de UDP, el puerto es opcional y el valor 0 significa que no hay ningún puerto. Normalmente no hay ningún servicio en un sistema que escucha en el puerto 0. Este evento puede indicar un intento de atacar el dispositivo de destino o indicar que una aplicación se programó de forma incorrecta. | Minor |
| Operación de DNP3 no válida | El dispositivo de origen ha iniciado una solicitud no válida. | Principal |
| Operación MODBUS no válida (excepción generada por el maestro) | El dispositivo de origen ha iniciado una solicitud no válida. | Principal |
| Operación MODBUS no válida (código de función cero) | El dispositivo de origen ha iniciado una solicitud no válida. | Principal |
| Versión de protocolo no válida | El dispositivo de origen ha iniciado una solicitud no válida. | Principal |
| Se ha enviado un parámetro incorrecto a una estación remota | El dispositivo de destino ha recibido una solicitud no válida. | Principal |
| Inicio de un código de función obsoleto (inicializar datos) | El dispositivo de origen ha iniciado una solicitud no válida. | Minor |
| Inicio de un código de función obsoleto (guardar configuración) | El dispositivo de origen ha iniciado una solicitud no válida. | Minor |
| El dispositivo maestro solicitó una confirmación en el nivel de aplicación | El dispositivo de origen ha iniciado una solicitud no válida. | Advertencia |
| Excepción Modbus | Un dispositivo de origen (subordinado) ha devuelto una excepción a un dispositivo de destino (maestro). | Principal |
| El dispositivo esclavo recibió un tipo de ASDU no válido | El dispositivo de destino ha recibido una solicitud no válida. | Principal |
| El dispositivo esclavo recibió una causa de comando de transmisión no válida | El dispositivo de destino ha recibido una solicitud no válida. | Principal |
| Un dispositivo esclavo recibió una dirección común no válida | El dispositivo de destino ha recibido una solicitud no válida. | Principal |
| Un dispositivo esclavo recibió un parámetro de dirección de datos no válido | El dispositivo de destino ha recibido una solicitud no válida. | Principal |
| Un dispositivo esclavo recibió un parámetro de valor de datos no válido | El dispositivo de destino ha recibido una solicitud no válida. | Principal |
| El dispositivo subordinado ha recibido un código de función no válido | El dispositivo de destino ha recibido una solicitud no válida. | Principal |
| El dispositivo esclavo recibió una dirección de objeto de información no válida | El dispositivo de destino ha recibido una solicitud no válida. | Principal |
| Se ha enviado un objeto desconocido a la estación remota | El dispositivo de destino ha recibido una solicitud no válida. | Principal |
| Uso de un código de función reservado | El dispositivo de origen ha iniciado una solicitud no válida. | Principal |
| Uso de un formato incorrecto por parte de la estación remota | El dispositivo de origen ha iniciado una solicitud no válida. | Advertencia |
| Uso de marcas de estado reservadas (IIN) | Un dispositivo de origen de DNP3 (estación remota) usó el indicador interno reservado 2.6. Se recomienda comprobar la configuración del dispositivo. | Advertencia |

## <a name="malware-engine-alerts"></a>Alertas del motor de malware

| Título | Descripción| severity |
|--|--|--|
| Intento de conexión a una dirección IP malintencionada conocida | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Principal |
| Mensaje SMB no válido (implante de puerta trasera de DoublePulsar) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Solicitud de nombre de dominio malintencionado | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Principal |
| Archivo de prueba de malware detectado: el antivirus EICAR ha funcionado correctamente | Se ha detectado un archivo de prueba del antivirus EICAR en el tráfico entre dos dispositivos. El archivo no es malware. Se utiliza para confirmar que el software antivirus está instalado correctamente; muestra lo que sucede cuando se detecta un virus y comprueba los procedimientos internos y las reacciones cuando se detecta un virus. El software antivirus debe detectar EICAR como si fuera un virus real. | Principal |
| Sospecha de malware de Conficker | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Principal |
| Sospecha de ataque de denegación de servicio | Un dispositivo de origen intentó iniciar un número excesivo de conexiones nuevas con un dispositivo de destino. Podría tratarse de un ataque de denegación de servicio (DOS) contra el dispositivo de destino y podría interrumpir la funcionalidad del dispositivo, afectar al rendimiento y la disponibilidad del servicio o causar errores irrecuperables. | Crítico |
| Sospecha de actividad malintencionada | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Principal |
| Sospecha de actividad malintencionada (BlackEnergy) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de actividad malintencionada (DarkComet) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de actividad malintencionada (Duqu) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de actividad malintencionada (Flame) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de actividad malintencionada (Havex) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de actividad malintencionada (Karagany) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de actividad malintencionada (LightsOut) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de actividad malintencionada (consultas de nombre) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Principal |
| Sospecha de actividad malintencionada (Poison Ivy) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de actividad malintencionada (Regin) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de actividad malintencionada (Stuxnet) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de actividad malintencionada (WannaCry) | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Principal |
| Sospecha de malware de NotPetya: se han detectado parámetros SMB no válidos | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de malware de NotPetya: se ha detectado una transacción de SMB no válida | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |
| Sospecha de ejecución remota de código con PsExec | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Principal |
| Sospecha de administración remota de servicios de Windows | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Principal |
| Archivo ejecutable sospechoso detectado en un punto de conexión | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Principal |
| Tráfico sospechoso detectado | Se ha detectado actividad sospechosa en la red. Esta actividad puede estar asociada con un ataque que aproveche un método usado por malware conocido. | Crítico |

## <a name="operational-engine-alerts"></a>Alertas del motor operativo

| Título | Descripción | severity |
|--|--|--|
| Se ha enviado un comando de detención de PLC de S7 | El dispositivo de origen ha enviado un comando de detención a un controlador de destino. El controlador dejará de funcionar hasta que se envíe un comando de inicio. | Advertencia |
| Error en la operación BACNet | Un servidor ha devuelto un código de error. Esto indica un error del servidor o una solicitud no válida de un cliente. | Principal |
| Estado incorrecto del dispositivo MMS | Un dispositivo de fabricación virtual MMS (VMD) ha enviado un mensaje de estado. El mensaje indica que es posible que el servidor no esté configurado correctamente, sea parcialmente operativo o no esté operativo en absoluto. | Principal |
| Cambio en la configuración del dispositivo | Se ha detectado un cambio de configuración en un dispositivo de origen. | Minor |
| Desbordamiento continuo del búfer de eventos en una estación remota | Se ha detectado un evento de desbordamiento de búfer en un dispositivo de origen. El evento puede producir daños en los datos, bloqueos del programa o ejecución de código malintencionado. | Principal |
| Restablecimiento del controlador | Un dispositivo de origen ha enviado un comando de restablecimiento a un controlador de destino. El controlador ha dejado de funcionar temporalmente y se ha iniciado de nuevo automáticamente. | Advertencia |
| Detención del controlador | El dispositivo de origen ha enviado un comando de detención a un controlador de destino. El controlador dejará de funcionar hasta que se envíe un comando de inicio. | Advertencia |
| El dispositivo no pudo recibir una dirección IP dinámica | El dispositivo de origen está configurado para recibir una dirección IP dinámica de un servidor DHCP, pero no recibió una dirección. Esto indica un error de configuración en el dispositivo o un error operativo en el servidor DHCP. Se recomienda notificar al administrador de red el incidente | Principal |
| Se sospecha que el dispositivo está desconectado (no responde) | Un dispositivo de origen no respondió a un comando que se le ha enviado. Es posible que se haya desconectado al enviar el comando. | Principal |
| Error en la solicitud de servicio CIP de EtherNet/IP | Un servidor ha devuelto un código de error. Esto indica un error del servidor o una solicitud no válida de un cliente. | Principal |
| Error del comando de protocolo de encapsulación de EtherNet/IP | Un servidor ha devuelto un código de error. Esto indica un error del servidor o una solicitud no válida de un cliente. | Principal |
| Desbordamiento de búfer de eventos en estación remota | Se ha detectado un evento de desbordamiento de búfer en un dispositivo de origen. El evento puede producir daños en los datos, bloqueos del programa o ejecución de código malintencionado. | Principal |
| No se ha producido la operación de copia de seguridad esperada | No se ha producido la actividad de copia de seguridad o transferencia de archivos esperada entre dos dispositivos. Esto puede indicar errores en el proceso de copia de seguridad o de transferencia de archivos. | Principal |
| Error de comando de GE SRTP | Un servidor ha devuelto un código de error. Esto indica un error del servidor o una solicitud no válida de un cliente. | Principal |
| Se ha enviado el comando de detención de PLC GE SRTP | El dispositivo de origen ha enviado un comando de detención a un controlador de destino. El controlador dejará de funcionar hasta que se envíe un comando de inicio. | Advertencia |
| El bloque de control GOOSE requiere una configuración adicional | Un dispositivo de origen ha enviado un mensaje GOOSE que indica que el dispositivo necesita una puesta en marcha. Esto significa que el bloque de control GOOSE requiere una configuración adicional y que los mensajes GOOSE no son operativos de forma parcial o total. | Principal |
| Se ha cambiado la configuración del conjunto de datos de GOOSE | Se ha cambiado un conjunto de datos de mensaje (identificado por el id. de protocolo) en un dispositivo de origen. Esto significa que el dispositivo informará de un conjunto de datos diferente para este mensaje. | Advertencia |
| Estado inesperado del controlador Honeywell | Un controlador Honeywell ha enviado un mensaje de diagnóstico inesperado que indica un cambio de estado. | Advertencia |
| Errores del cliente HTTP | El dispositivo de origen ha iniciado una solicitud no válida. | Advertencia |
| Dirección IP no válida | El sistema ha detectado tráfico entre un dispositivo de origen y una dirección IP que no es una dirección válida. Esto puede indicar una configuración incorrecta o un intento de generar tráfico ilegal. | Minor |
| Error de autenticación maestro-subordinado | Error en el proceso de autenticación entre un dispositivo de origen DNP3 (maestro) y un dispositivo de destino (estación remota). | Minor |
| Error en la solicitud de servicio de MMS | Un servidor ha devuelto un código de error. Esto indica un error del servidor o una solicitud no válida de un cliente. | Principal |
| No se ha detectado ningún tráfico en la interfaz del sensor | Un sensor ha dejado de detectar tráfico de red en una interfaz de red. | Crítico |
| El servidor de OPC UA ha generado un evento que requiere la atención del usuario | Un servidor de OPC UA ha enviado una notificación de eventos a un cliente. Este tipo de evento requiere la atención del usuario | Principal |
| Error en la solicitud de servicio de OPC UA | Un servidor ha devuelto un código de error. Esto indica un error del servidor o una solicitud no válida de un cliente. | Principal |
| La estación remota se ha reiniciado | Se ha detectado un reinicio en frío en un dispositivo de origen. Esto significa que el dispositivo se apagó físicamente y que ha vuelto a encenderse. | Advertencia |
| La estación remota se reinicia con frecuencia | Se ha detectado un número excesivo de reinicios en frío en un dispositivo de origen. Esto significa que el dispositivo se apagó físicamente y que vuelve encenderse un número excesivo de veces. | Minor |
| Se ha cambiado la configuración de la estación remota | Se ha detectado un cambio de configuración en un dispositivo de origen. | Principal |
| Se ha detectado una configuración dañada de la estación remota | Este dispositivo de origen DNP3 (estación remota) ha informado de una configuración dañada. | Principal |
| Error del comando DCP de Profinet | Un servidor ha devuelto un código de error. Esto indica un error del servidor o una solicitud no válida de un cliente. | Principal |
| Restablecimiento de fábrica de dispositivos Profinet | Un dispositivo de origen ha enviado un comando de restablecimiento de fábrica a un dispositivo de destino Profinet. El comando de restablecimiento borra la configuración del dispositivo Profinet y detiene su funcionamiento. | Advertencia |
| Error en la operación de RPC | Un servidor ha devuelto un código de error. Esto indica un error del servidor o una solicitud no válida de un cliente. | Principal |
| Se ha modificado la configuración del conjunto de datos del mensaje de valores de ejemplo | Se ha cambiado un conjunto de datos de mensaje (identificado por el id. de protocolo) en un dispositivo de origen. Esto significa que el dispositivo informará de un conjunto de datos diferente para este mensaje. | Advertencia |
| Error irrecuperable del dispositivo subordinado | Se ha detectado un error de condición irrecuperable en un dispositivo de origen. Este tipo de error suele indicar un error de hardware o un error al ejecutar un comando específico. | Principal |
| Sospecha de problemas de hardware en la estación remota | Se ha detectado un error de condición irrecuperable en un dispositivo de origen. Este tipo de error suele indicar un error de hardware o un error al ejecutar un comando específico. | Principal |
| Sospecha de dispositivo MODBUS sin respuesta | Un dispositivo de origen no respondió a un comando que se le ha enviado. Es posible que se haya desconectado al enviar el comando. | Minor |
| Se ha detectado tráfico en la interfaz del sensor | Un sensor ha reanudado la detección de tráfico de red en una interfaz de red. | Advertencia |

## <a name="next-steps"></a>Pasos siguientes

Puede [administrar los eventos de alerta](how-to-manage-the-alert-event.md).
Más información sobre cómo [reenviar información de alertas](how-to-forward-alert-information-to-partners.md).
