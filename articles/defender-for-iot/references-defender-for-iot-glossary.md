---
title: Glosario de Defender para IoT
description: En este glosario se proporciona una breve descripción de los términos y conceptos más importantes de la plataforma de Defender para IoT.
ms.date: 12/09/2020
ms.topic: article
ms.openlocfilehash: 829469a523ffd259b0108fcfc189715ba99cce74
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108203692"
---
# <a name="defender-for-iot-glossary"></a>Glosario de Defender para IoT

En este glosario se proporciona una breve descripción de los términos y conceptos más importantes de la plataforma de Azure Defender para IoT. Seleccione los vínculos de **Más información** para ir a los términos relacionados en el glosario. Esto le ayudará a aprender y usar las herramientas del producto con mayor rapidez.

> [!Note]
> Cualquier término con que tenga `(DB)` en su nombre es un término de generador de dispositivos basado en agente. 

<a name="glossary-a"></a>

## <a name="a"></a>A

| Término | Descripción | Más información |
|--|--|--|
| **Grupo de acceso** | Admitir los requisitos de acceso de usuario para organizaciones de gran tamaño mediante la creación de reglas del grupo de acceso.<br /><br />Las reglas permiten controlar el acceso de vista y configuración a la consola de administración local de Defender para IoT para roles de usuario específicos en las unidades de negocio, las regiones, los sitios y las zonas pertinentes.<br /><br />Por ejemplo, permitir a los analistas de seguridad de un grupo de Active Directory acceder a los datos de la industria automovilística de Europa occidental, pero evitar el acceso a los datos de África. | **[Consola de administración local](#o)** <br /><br />**[Unidad de negocio](#b)** |
| **Tokens de acceso** | Generar tokens de acceso para acceder a la API REST de Defender para IoT. | **[API](#glossary-a)** |
| **Confirmar evento de alertas** | Indicar a Defender para que IoT oculte la alerta una vez para el evento detectado. La alerta se volverá a desencadenar si el evento se vuelve a detectar. | **[Alerta](#glossary-a)<br /><br />[Aprender sobre eventos de alertas](#l)<br /><br />[Silenciar evento de alertas](#m)** |
| **Alerta** | Mensaje que desencadena el motor de Defender para IoT con respecto a las desviaciones del comportamiento de red autorizado, las anomalías de red o la actividad y el tráfico de red sospechosos. | **[Regla de reenvío](#f)<br /><br />[Regla de exclusión](#e)<br /><br />[Notificaciones del sistema](#s)** |
| **Comentario de alerta** | Comentarios que realizan los analistas de seguridad y los administradores en los mensajes de alertas. Por ejemplo, un comentario de alerta pueden ser las instrucciones sobre las acciones de mitigación que se deben realizar o los nombres de los usuarios con los que ponerse en contacto con respecto al evento.<br /><br />Los usuarios que revisan las alertas pueden elegir el comentario o los comentarios que mejor reflejen el estado del evento o los pasos que se han tomado para investigar la alerta. | **[Alerta](#glossary-a)** |
| **Motor de anomalías** | El motor de Defender for IoT que detecta las comunicaciones o los comportamientos inusuales de máquina a máquina (M2M). Por ejemplo, el motor podría detectar demasiados intentos de inicio de sesión de SMB. Las alertas de anomalías se desencadenan cuando se detectan estos eventos. | **[motores de Defender para IoT](#d)** |
| **API** | Permitir a sistemas externos acceder a los datos detectados por Defender para IoT y realizar acciones con la API REST externa mediante conexiones SSL. | **[Tokens de acceso](#glossary-a)** |
| **Informe de vectores de ataque** | Representación gráfica en tiempo real de las cadenas de vulnerabilidades de los puntos de conexión con vulnerabilidades de seguridad.<br /><br />Los informes le permiten evaluar el efecto de las actividades de mitigación en la secuencia de ataque para determinar. Por ejemplo, puede evaluar si una actualización del sistema altera la ruta de acceso del atacante interrumpiendo la cadena de ataque o si permanece una ruta de acceso de ataque alternativa. Esto prioriza las actividades de corrección y mitigación. | **[Informe de evaluación de riesgos](#r)** |

## <a name="b"></a>B

| Término | Descripción | Más información |
|--|--|--|
| **Unidad de negocio** | Organización lógica de la empresa en función de sectores específicos.<br /><br />Por ejemplo, una empresa global que tiene fábricas de vidrio y de plástico se puede administrar como dos unidades de negocio diferentes. Puede controlar el acceso de los usuarios de Defender para IoT a unidades de negocio específicas. | **[Consola de administración local](#o)<br /><br />[Grupo de acceso](#glossary-a)<br /><br />[Sitio](#s)<br /><br />[Zona](#z)** |
| **Línea de base** | Tráfico, protocolos, comandos y dispositivos de red aprobados. Defender para IoT identifica las desviaciones de la línea de base de red. Permite ver el tráfico de línea de base aprobado mediante la generación de informes de minería de datos. | **[Minería de datos](#d)<br /><br />[Modo aprendizaje](#l)** |

## <a name="c"></a>C

| Término | Descripción | Más información |
|--|--|--|
| **Comandos de la CLI** | Opciones de la interfaz de la línea de comandos (CLI) para los usuarios administradores de Defender para IoT. Los comandos de la CLI están disponibles para las características a las que no se puede acceder desde las consolas de Defender para IoT. | - |


## <a name="d"></a>D

| Término | Descripción | Más información |
|--|--|--|
| **Minería de datos** | Generar informes completos y pormenorizados sobre los dispositivos de red:<br /><br />- **respuesta a los incidentes SOC**: informes en tiempo real para ayudar a gestionar la respuesta inmediata a los incidentes. Por ejemplo, un informe puede mostrar los dispositivos que pueden necesitar la aplicación de revisiones.<br /><br />- **Análisis forense**: informes basados en datos históricos de informes de investigación.<br /><br />- **integridad de la red de TI**: informes que ayudan a mejorar la seguridad general de la red. Por ejemplo, un informe puede enumerar los dispositivos con credenciales de autenticación poco seguras.<br /><br />- **visibilidad**: informes que abarcan todos los elementos de consulta para ver todos los parámetros de línea de base de la red.<br /><br />Guarde los informes de minería de datos para que los usuarios de solo lectura puedan verlos. | **[Línea de base](#b)<br /><br />[Informes](#r)** |
| **plataforma de Defender para IoT** | La solución de Defender para IoT instalada en los sensores de Defender para IoT y la consola de administración local. | **[Sensor](#s)<br /><br />[Consola de administración local](#o)** |
| **Mapa de dispositivos** | Representación gráfica de los dispositivos de red que detecta Defender para IoT. Muestra las conexiones entre los dispositivos e información sobre cada dispositivo. Use el mapa para:<br /><br />- recuperar y controlar la información crítica del dispositivo.<br /><br />- analizar segmentos de red.<br /><br />- exportar detalles y resúmenes del dispositivo. | **[grupo de capa de Purdue](#p)** |
| **Inventario de dispositivos: sensor** | El inventario de dispositivos muestra una amplia gama de atributos de dispositivo detectados por Defender para IoT. Hay opciones disponibles para:<br /><br />- filtrar la información mostrada.<br /><br />- exportar esta información a un archivo CSV.<br /><br />- importar los detalles del registro de Windows. | **[Group (Grupo)](#g)** <br /><br />**[Inventario de dispositivos: consola de administración local](#d)** |
| **Inventario de dispositivos: consola de administración local** | La información del dispositivo de los sensores conectados se puede ver desde la consola de administración local en el inventario de dispositivos. Esto proporciona a los usuarios de la consola de administración local una vista completa de toda la información de la red. | **[Inventario de dispositivos: sensor](#d)<br /><br />[Inventario de dispositivos: integrador de datos](#d)** |
| **Inventario de dispositivos: integrador de datos** | Las funcionalidades de integración de datos de la consola de administración local permiten mejorar los datos del inventario de dispositivos con información de otros recursos de la empresa. Algunos recursos de ejemplo son las CMDB, DNS, los firewalls y las API web. | **[Inventario de dispositivos: consola de administración local](#d)** |
| **Dispositivos gemelos** `(DB)` | Los dispositivos gemelos son documentos JSON que almacenan información acerca del estado del dispositivo, incluidos metadatos, configuraciones y condiciones. | [Módulo gemelo](#m) <br /> <br />[Módulo gemelo de Defender-IoT-micro-agent](#s) |

## <a name="e"></a>E

| Término | Descripción | Más información |
|--|--|--|
| **Engines** (Motores) | Los motores de análisis de autoaprendizaje de Defender para IoT eliminan la necesidad de actualizar firmas o definir reglas. Los motores emplean el análisis de comportamiento específico de ICS y la ciencia de datos para analizar de forma continua el tráfico de red de OT en busca de anomalías, malware, problemas operativos, infracciones de protocolo y desviaciones de la actividad de red de línea de base.<br /><br />Cuando un motor detecta una desviación, se desencadena una alerta. Las alertas se pueden ver y administrar desde la pantalla **Alertas** o desde un SIEM. | **[Alerta](#glossary-a)** |
| **Vista empresarial** | Mapa global que presenta las unidades de negocio, los sitios y las zonas en las que se instalan los sensores de Defender para IoT. Ver las ubicaciones geográficas de las alertas malintencionadas, las alertas operativas y mucho más. | **[Unidad de negocio](#b)<br /><br />[Sitio](#s)<br /><br />[Zona](#z)** |
| **Escala de tiempo de eventos** | Escala de tiempo de la actividad detectada en la red, incluidos:<br /><br />- alertas desencadenadas.<br /><br />- eventos de red (informativo).<br /><br />- operaciones de usuario, como el inicio de sesión, la eliminación de usuarios o la creación de usuarios, y operaciones de administración de alertas, como silenciar, aprender y confirmar. Disponible en las consolas de sensores. | - |
| **Regla de exclusión** | Indicar a Defender for IoT que ignore los desencadenadores de alertas en función del período de tiempo, la dirección del dispositivo, el nombre de la alerta o un sensor específico.<br /><br />Por ejemplo, si sabe que todos los dispositivos OT supervisados por un sensor específico van a pasar por un procedimiento de mantenimiento entre las 6:30 y las 10:15 de la mañana, puede establecer una regla de exclusión que indique que este sensor no debe enviar ninguna alerta en el período predefinido. | **[Alerta](#glossary-a)<br /><br />[Silenciar evento de alertas](#m)** |

## <a name="f"></a>F

| Término | Descripción | Más información |
|--|--|--|
| **Regla de reenvío** | Las reglas de reenvío indican a Defender para IoT que envíe información de alertas a proveedores o sistemas asociados.<br /><br />Por ejemplo, enviar información de alertas a un servidor de Splunk o a un servidor de syslog. | **[Alerta](#glossary-a)** |

## <a name="g"></a>G

| Término | Descripción | Más información |
|--|--|--|
| **Grupo** | Grupos predefinidos o personalizados de dispositivos que contienen atributos específicos, como los dispositivos que llevan a cabo actividades de programación o dispositivos que se encuentran en una subred específica. Use los grupos para ayudarle a ver y analizar dispositivos en Defender para IoT.<br /><br />Los grupos se pueden ver y crear desde el mapa de dispositivos y el inventario de dispositivos. | **[Mapa de dispositivos](#d)<br /><br />[Inventario de dispositivos](#d)** |

## <a name="h"></a>H

| Término | Descripción | Más información |
|--|--|--|
| **Entorno de desarrollo abierto Horizon** | Proteger los dispositivos IoT e ICS que ejecutan protocolos propietarios y personalizados, o protocolos que se desvían de cualquier estándar. Use el SDK del entorno de desarrollo abierto (ODE) de Horizon para desarrollar complementos de disección que descodifican el tráfico de red basado en protocolos definidos. Los servicios de Defender para IoT analizan el tráfico para ofrecer supervisión, alertas e informes completos.<br /><br />Use Horizon para:<br /><br />- **Expandir** la visibilidad y el control sin necesidad de actualizar las versiones de la plataforma de Defender para IoT.<br /><br />- **Proteger** la información propietaria mediante el desarrollo local como complemento externo.<br /><br />- **Localizar** el texto de las alertas, eventos y parámetros de protocolos.<br /><br />Póngase en contacto con el representante de casos de éxito de clientes para obtener información detallada. | **[Compatibilidad de protocolos](#p)<br /><br />[Localización](#l)** |
| **alerta personalizada de Horizon** | Mejorar la administración de alertas en la empresa mediante el desencadenamiento de alertas personalizadas para cualquier protocolo (basado en la disección del tráfico del marco de Horizon).<br /><br />Estas alertas se pueden usar para comunicar información:<br /><br />- sobre las detecciones de tráfico basadas en protocolos y protocolos subyacentes en un complemento propietario de Horizon.<br /><br />- sobre una combinación de campos de protocolo de todas las capas de protocolos. | **[Compatibilidad de protocolos](#p)** |

## <a name="i"></a>I

| Término | Descripción | Más información |
|--|--|--|
| **IoT Hub** `(DB)` | Servicio administrado, hospedado en la nube, que actúa como centro de mensajes para comunicaciones bidireccionales entre la aplicación de IoT y los dispositivos que administra.  |   |
| **Integraciones** | Expandir las funcionalidades de Defender para IoT compartiendo la información del dispositivo con los sistemas asociados. Las organizaciones pueden conectar las soluciones de seguridad, NAC, administración de incidentes y administración de dispositivos puestas previamente en silos para acelerar las respuestas en todo el sistema y mitigar los riesgos con mayor rapidez. | **[Regla de reenvío](#f)** |
| **Subred interna** | Configuraciones de subred definidas por Defender para IoT. En algunos casos, como en los entornos donde se usan intervalos públicos como intervalos internos, puede indicar a Defender para IoT que resuelva todas las subredes como subredes internas. Las subredes se muestran en el mapa y en varios informes de Defender para IoT. | **[Subredes](#s)** |

## <a name="l"></a>L

| Término | Descripción | Más información |
|--|--|--|
| **Aprender evento de alertas** | Indicar a defender para IoT que autorice el tráfico detectado en un evento de alertas. | **[Alerta](#glossary-a)<br /><br />[Confirmar evento de alertas](#glossary-a)<br /><br />[Silenciar evento de alertas](#m)** |
| **Modo aprendizaje** | Es el modo que se usa cuando Defender para IoT aprende la actividad de red. Esta actividad se convierte en la línea de base de la red. Defender para IoT permanece en este modo durante un período predefinido después de la instalación. La actividad que se desvía de la actividad aprendida después de este período desencadenará alertas de Defender para IoT. | **[Aprendizaje inteligente de TI](#s)<br /><br />[Línea de base](#b)** |
| **Localización** | Localizar texto para alertas, eventos y parámetros de protocolo para los complementos de disección desarrollados por Horizon. | **[Entorno de desarrollo abierto Horizon](#h)** |

## <a name="m"></a>M


| Término | Descripción | Más información |
|--|--|--|
| **Microagente** `(DB)` | Proporciona funcionalidades de seguridad de profundidad para dispositivos IoT, como la seguridad y la detección de amenazas. | |
| **Módulo gemelo** `(DB)` | Los módulos gemelos son documentos JSON que almacenan información acerca del estado del módulo, incluidos metadatos, configuraciones y condiciones. | [Dispositivo gemelo](#d) <br /> <br />[Módulo gemelo de Defender-IoT-micro-agent](#s) |
| **Silenciar evento de alertas** | Indicar a Defender para IoT que omita continuamente la actividad con dispositivos idénticos y tráficos comparables. | **[Alerta](#glossary-a)<br /><br />[Regla de exclusión](#e)<br /><br />[Confirmar evento de alertas](#glossary-a)<br /><br />[Aprender evento de alertas](#l)** |

## <a name="n"></a>N

| Término | Descripción | Más información |
|--|--|--|
| **Notificaciones** | Información acerca de los cambios de red o las propiedades de dispositivo sin resolver. Existen opciones disponibles para actualizar la información del dispositivo y de la red con los nuevos datos detectados. La respuesta a las notificaciones enriquece el inventario de dispositivos, el mapa y varios informes. Disponible en las consolas de sensores. | **[Alerta](#glossary-a)<br /><br />[Notificaciones del sistema](#s)** |

## <a name="o"></a>O

| Término | Descripción | Más información |
|--|--|--|
| **Consola de administración local** | La consola de administración local proporciona una vista y administración centralizadas de los dispositivos y las amenazas que detectan las implementaciones de sensores de Defender para IoT en la organización. | **[Plataforma de Defender para IoT](#d)<br /><br />[Sensor](#s)** |
| **Alerta operativa** | Alertas que abordan los problemas operativos de la red, como un dispositivo sospechoso de estar desconectado de la red. | **[Alerta](#glossary-a)<br /><br />[Alerta de seguridad](#s)** |

## <a name="p"></a>P

| Término | Descripción | Más información |
|--|--|--|
| **capa de Purdue** | Muestra las interconexiones e interdependencias de los componentes principales de un ICS típico en el mapa. |  |
| **Compatibilidad con protocolos** | Además de la compatibilidad con protocolos insertados, puede proteger los dispositivos IoT e ICS que ejecutan protocolos propietarios y personalizados, o protocolos que se desvían de cualquier estándar, mediante el uso del SDK del entorno de desarrollo abierto de Horizon. | **[Entorno de desarrollo abierto Horizon](#h)** |

## <a name="r"></a>R

| Término | Descripción | Más información |
|--|--|--|
| **Región** | División lógica de una organización global en regiones geográficas. Algunos ejemplos son Norteamérica, Europa occidental y Europa Oriental.<br /><br />Norteamérica podría tener fábricas de varias unidades de negocio. | **[Grupo de acceso](#glossary-a)<br /><br />[Unidad de negocio](#b)<br /><br />[Consola de administración local](#o)<br /><br />[Sitio](#s)<br /><br />[Zona](#z)** |
| **Informes** | Los informes reflejan la información generada por los resultados de la consulta de minería de datos. Esto incluye los resultados de minería de datos predeterminados, que están disponibles en la vista **Informes**. Los administradores y los analistas de seguridad también pueden generar consultas de minería de datos personalizadas y guardarlas como informes. Estos informes también estarán disponibles para los usuarios de solo lectura. | **[Minería de datos](#d)** |
| **Informe de evaluación de riesgos** | Los informes de evaluación de riesgos permiten generar una puntuación de seguridad para cada dispositivo de red, junto con una puntuación general de seguridad de red. La puntuación total representa el porcentaje del 100 % de seguridad. El informe proporciona recomendaciones de mitigación que le ayudarán a mejorar la puntuación de seguridad actual. | - |

## <a name="s"></a>S

| Término | Descripción | Más información |
|--|--|--|
| **Alerta de seguridad** | Alertas que abordan problemas de seguridad, como intentos excesivos de inicio de sesión de SMB o detecciones de malware. | **[Alerta](#glossary-a)<br /><br />[Alerta operativa](#o)** |
| **Módulo gemelo de Defender-IoT-micro-agent** `(DB)` | Este módulo gemelo contiene toda la información pertinente para la seguridad de cada dispositivo específico de la solución. | [Dispositivo gemelo](#d) <br /> <br />[Módulo gemelo](#m)  |
| **Sondeo selectivo** | Defender para IoT inspecciona de forma pasiva el tráfico de TI y OT y detecta información pertinente sobre los dispositivos, sus atributos, su comportamiento, etc. En algunos casos, es posible que alguna información no sea visible en los análisis pasivos de la red.<br /><br />Cuando esto ocurre, puede usar las herramientas de sondeo detallado seguras de Defender para IoT con el fin de detectar información importante sobre los dispositivos a los que anteriormente no se podía acceder. | - |
| **Sensor** | Máquina física o virtual en la que está instalada la plataforma de Defender para IoT. | **[Consola de administración local](#o)** |
| **Sitio** | Ubicación de una fábrica u otra entidad. El sitio debe contener una o varias zonas en las que esté instalado un sensor. | **[Zona](#z)** |
| **Administración de sitios** | Opción de la consola de administración local que le permite administrar los sensores de la empresa. | - |
| **Aprendizaje inteligente de TI** | Después de que finaliza el período de aprendizaje y se deshabilita el modo aprendizaje, Defender para IoT podría detectar un nivel inusualmente alto de cambios en la línea de base que son el resultado de la actividad de TI normal, como las solicitudes DNS y HTTP. Este tráfico podría desencadenar alertas de infracción de directivas y notificaciones del sistema innecesarias. Para reducir estas alertas y notificaciones, puede habilitar el Aprendizaje de TI inteligente. | **[Modo aprendizaje](#l)<br /><br />[Línea de base](#b)** |
| **Subredes** | Para permitir centrarse en los dispositivos de OT, la subred agrega automáticamente los dispositivos de TI en el mapa de dispositivos. Cada subred se presenta como una única entidad en el mapa, que incluye una funcionalidad interactiva de contracción y expansión para centrarse en una subred de TI y volver atrás. | **[Mapa de dispositivos](#d)** |
| **Notificaciones del sistema** | Notificaciones de la consola de administración local correspondientes a:<br /><br />- Estado de la conexión del sensor.<br /><br />- Errores de la copia de seguridad remota. | **[Notificaciones](#n)<br /><br />[Alerta](#glossary-a)** |

## <a name="z"></a>Z

| Término | Descripción | Más información |
|--|--|--|
| **Zona** | Área dentro de un sitio en el que se instalan uno o varios sensores. | **[Sitio](#s)<br /><br />[Unidad de negocio](#b)<br /><br />[Región](#r)** |
