---
title: Novedades de Azure Defender para IoT
description: Este artículo le permite conocer las novedades de la versión más reciente de Defender para IoT.
ms.topic: overview
ms.date: 04/25/2021
ms.openlocfilehash: f7631caedcb26227abf65d6fad216fc629d41dd4
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107987100"
---
# <a name="whats-new-in-azure-defender-for-iot"></a>Novedades de Azure Defender para IoT  

En este artículo se enumeran las nuevas características y mejoras de características de Defender para IoT.

Las características mencionadas se encuentran en VERSIÓN PRELIMINAR. Los [Términos de uso complementarios para las versiones preliminares de Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) incluyen otros términos legales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado con disponibilidad general.

## <a name="versioning-and-support-for-azure-defender-for-iot"></a>Control de versiones y compatibilidad con Azure Defender para IoT 

A continuación se enumeran la compatibilidad, las directivas de cambios importantes de Defender para IoT y las versiones de Azure Defender para IoT que están disponibles actualmente. 

### <a name="servicing-information-and-timelines"></a>Información y escalas de tiempo de mantenimiento 

Microsoft planea publicar actualizaciones de Azure Defender para IoT al menos una vez por trimestre. Cada versión de disponibilidad general del sensor de Azure Defender para IoT y la consola de administración local tienen nueve meses de soporte técnico después de su publicación. Las correcciones y las nuevas funcionalidades se aplicarán a la versión de disponibilidad general actual que se encuentre actualmente con soporte técnico y no se aplicarán a las versiones anteriores de disponibilidad general.

### <a name="versions-and-support-dates"></a>Versiones y fechas de soporte técnico

| Versión | Fecha de lanzamiento | Fecha de finalización de soporte técnico |
|--|--|--|
| 10.0 | Enero de 2021 | Octubre de 2021 |
| 10,3 | 04/2021 | 02/2022 |

## <a name="april-2021"></a>Abril de 2021

### <a name="work-with-automatic-threat-intelligence-updates-public-preview"></a>Uso de actualizaciones automáticas de inteligencia sobre amenazas (versión preliminar pública)

Los nuevos paquetes de inteligencia sobre amenazas ahora se pueden insertar automáticamente en los sensores conectados a la nube a medida que Microsoft Defender para IoT los libera. A esto además se le suma la posibilidad de descargar los paquetes de inteligencia sobre amenazas y luego cargarlos en los sensores.

Trabajar con actualizaciones automáticas ayuda a reducir los esfuerzos operativos y a garantizar una mayor seguridad. Habilite la actualización automática mediante la incorporación del sensor conectado a la nube en el portal de Defender para IoT con el botón de alternancia **Automatic Threat Intelligence Updates** (Actualizaciones automáticas de inteligencia sobre amenazas) activado.

Si quiere adoptar un enfoque más conservador para actualizar los datos de inteligencia sobre amenazas, puede insertar manualmente paquetes desde el portal de Azure Defender para IoT en los sensores conectados a la nube solo cuando considere que es necesario.
Así, puede controlar cuándo se instala un paquete, sin necesidad de descargarlo y luego cargarlo en los sensores. Inserte manualmente actualizaciones en los sensores desde la página **Sites and Sensors** (Sitios y sensores) de Defender para IoT.

También puede revisar la siguiente información sobre los paquetes de inteligencia sobre amenazas:

- Versión instalada del paquete
- Modo de actualización de inteligencia sobre amenazas 
- Estado de la actualización de inteligencia sobre amenazas

### <a name="view-cloud-connected-sensor-information-public-preview"></a>Visualización de la información del sensor conectado a la nube (versión preliminar pública)

Vea información operativa importante sobre los sensores conectados a la nube en la página **Sites and Sensors** (Sitios y sensores).

- La versión del sensor instalada.
- El estado de conexión del sensor a la nube.
- La última vez que se detectó que el sensor se conectaba a la nube.

### <a name="alert-api-enhancements"></a>Mejoras de la API de alerta

Hay nuevos campos disponibles para los usuarios que trabajan con las API de alerta.

**Consola de administración local**

- Dirección de origen y destino
- Pasos para la corrección
- Nombre del sensor definido por el usuario
- Nombre de la zona asociada al sensor 
- Nombre del sitio asociado al sensor

**Sensor**

- Dirección de origen y destino
- Pasos para la corrección

Al trabajar con nuevos campos, se requiere la versión 2 de la API.

### <a name="features-delivered-as-generally-available-ga"></a>Características que se proporcionan como disponibles con carácter general (GA)

Las siguientes características estaban disponibles anteriormente para la versión preliminar pública y ahora están disponibles con carácter general (GA):

- Sensor: reglas de alerta personalizadas mejoradas.
- Consola de administración local: exportación de alertas
- Incorporación de una segunda interfaz de red a la consola de administración local
- Generador de dispositivos: nuevo microagente

## <a name="march-2021"></a>Marzo de 2021

### <a name="sensor---enhanced-custom-alert-rules-public-preview"></a>Sensor: reglas de alerta personalizadas mejoradas (versión preliminar pública)

Ahora puede crear reglas de alerta personalizadas en función del día, grupo de días o período de tiempo durante el que se detectó la actividad de red.  Trabajar con las condiciones de regla de día y hora resulta útil, por ejemplo, en los casos en los que la gravedad de la alerta se deriva según la hora en que se produce el evento de alerta. Por ejemplo, cree una regla personalizada que desencadene una alerta de gravedad alta cuando se detecte actividad de red durante un fin de semana o por la noche.

Esta característica está disponible en el sensor con el lanzamiento de la versión 10.2.

### <a name="on-premises-management-console---export-alerts-public-preview"></a>Consola de administración local: exportación de alertas (versión preliminar pública)

La información de las alertas ahora se puede exportar a un archivo .csv desde la consola de administración local. Puede exportar la información de todas las alertas detectadas o puede exportar la información en función de la vista filtrada.

Esta característica está disponible en la consola de administración local con el lanzamiento de la versión 10.2.

### <a name="add-second-network-interface-to-on-premises-management-console-public-preview"></a>Incorporación de una segunda interfaz de red a la consola de administración local (versión preliminar pública)

Ahora puede mejorar la seguridad de la implementación al agregar una segunda interfaz de red a la consola de administración local. Esta característica permite que la administración local tenga sensores conectados en una red segura, a la vez que permite a los usuarios acceder a la consola de administración local a través de una segunda interfaz de red independiente.

Esta característica está disponible en la consola de administración local con el lanzamiento de la versión 10.2.

### <a name="add-second-network-interface-to-on-premises-management-console-public-preview"></a>Agregue una segunda interfaz de red a la consola de administración local (versión preliminar pública).

Ahora puede mejorar la seguridad de la implementación al agregar una segunda interfaz de red a la consola de administración local. Esta característica permite que la administración local tenga sensores conectados en una red segura, a la vez que permite a los usuarios acceder a la consola de administración local a través de una segunda interfaz de red independiente.

Esta característica está disponible en la consola de administración local con el lanzamiento de la versión 10.2.
### <a name="device-builder---new-micro-agent-public-preview"></a>Generador de dispositivos: nuevo microagente (versión preliminar pública)

Hay disponible un nuevo módulo del generador de dispositivos. El módulo, denominado microagente, permite lo siguiente:

- **Integración con Azure IoT Hub y Azure Defender para IoT**: logre una mejor seguridad para el punto de conexión directamente en los dispositivos de IoT mediante la integración con la opción de supervisión que proporcionan Azure IoT Hub y Azure Defender para IoT.
- **Opciones de implementación flexibles compatibles con los sistemas operativos de IoT estándar**: se pueden implementar como un paquete binario o como código fuente modificable, compatibles con los sistemas operativos de IoT estándar, como Linux y Azure RTOS.
- **Requisitos mínimos de recursos sin dependencias del kernel de sistema operativo**: superficie de memoria pequeña, consumo de CPU bajo y ninguna dependencia del kernel del sistema operativo.
- **Administración de la postura de seguridad**: supervise de forma proactiva la posición de seguridad de los dispositivos IoT.
- **Detección de amenazas de IoT/OT continua y en tiempo real**: detecte amenazas como redes de robots (botnets), intentos por fuerza bruta, mineros de criptomonedas y actividad de red sospechosa.

La documentación de Defender-IoT-micro-agent se transferirá a la carpeta *Agent-based solution for device builders>Classic* (Solución basada en agentes para generadores de dispositivos > Clásica).

Este conjunto de características está disponible con la versión preliminar pública en la nube actual.

## <a name="january-2021"></a>Enero de 2021

- [Seguridad](#security)
- [Incorporación](#onboarding)
- [Facilidad de uso](#usability)
- [Otras actualizaciones](#other-updates)
### <a name="security"></a>Seguridad

En esta versión, se han realizado mejoras en la recuperación de certificados y contraseñas.

#### <a name="certificates"></a>Certificados
  
Esta versión le permite:

- Cargar certificados SSL directamente en los sensores y en las consolas de administración locales.
- Realizar la validación entre la consola de administración y los sensores conectados, y entre una consola de administración y una consola de administración de alta disponibilidad. La validación se basa en las fechas de expiración, la autenticidad de la entidad de certificación raíz y las listas de revocación de certificados.  Si se produce un error en la validación, la sesión no continuará.

Para actualizaciones:

- Durante la actualización, no se produce ningún cambio en la funcionalidad de validación o del certificado SSL.
- Después de la actualización, los usuarios administrativos del sensor y de la consola de administración local pueden reemplazar los certificados SSL o activar la validación de certificados SSL desde la ventana Configuración del sistema, Certificado SSL.  

En el caso de instalaciones nuevas:

- Durante el primer inicio de sesión, los usuarios deben usar un certificado SSL (recomendado) o un certificado autofirmado generado localmente (no recomendado).
- La validación de certificados está activada de forma predeterminada con las instalaciones nuevas.

#### <a name="password-recovery"></a>Recuperación de la contraseña
  
Los usuarios administrativos del sensor y de la consola de administración local ahora pueden recuperar contraseñas desde el portal de Azure Defender para IoT. Antes, esta tarea requería la intervención del equipo de soporte técnico.

### <a name="onboarding"></a>Incorporación

#### <a name="on-premises-management-console---committed-devices"></a>Consola de administración local: dispositivos confirmados

Después del inicio de sesión inicial en la consola de administración local, ahora es necesario que los usuarios carguen un archivo de activación. El archivo contiene el número agregado de dispositivos que se van a supervisar en la red de la organización. Este número se conoce como el número de dispositivos confirmados.
Los dispositivos confirmados se definen durante el proceso de incorporación en el portal de Azure Defender para IoT, donde se genera el archivo de activación.
Tanto los usuarios nuevos como los que actualizan deben cargar el archivo de activación.
Después de la activación inicial, el número de dispositivos detectados en la red puede superar el número de dispositivos confirmados. Este evento puede ocurrir, por ejemplo, si conecta más sensores a la consola de administración. Si hay una discrepancia entre el número de dispositivos detectados y el número de dispositivos confirmados, aparece una advertencia en la consola de administración. Si se produce este evento, debe cargar un nuevo archivo de activación.

#### <a name="pricing-page-options"></a>Opciones de la página de precios

La página de precios le permite incorporar nuevas suscripciones a Azure Defender para IoT y definir los dispositivos confirmados en la red.  
Además, la página de precios ahora permite administrar las suscripciones existentes asociadas a un sensor y actualizar la confirmación de los dispositivos.

#### <a name="view-and-manage-onboarded-sensors"></a>Visualización y administración de sensores incorporados

Una nueva página del portal, Site and Sensors (Sitios y sensores), le permite:

- Agregar información descriptiva sobre el sensor. Por ejemplo, una zona asociada al sensor o etiquetas de texto libre.
- Ver y filtrar la información del sensor. Por ejemplo, ver los detalles sobre los sensores que están conectados a la nube o administrados de forma local o ver información sobre los sensores de una zona específica.  

### <a name="usability"></a>Facilidad de uso

#### <a name="azure-sentinel-new-connector-page"></a>Nueva página del conector de Azure Sentinel

La página del conector de datos de Azure Defender para IoT se ha rediseñado. El conector de datos ahora se basa en suscripciones en lugar de en centros de IoT, lo que permite a los clientes administrar mejor su conexión de configuración a Azure Sentinel.

#### <a name="azure-portal-permission-updates"></a>Actualizaciones de permisos de Azure Portal  

Se ha agregado compatibilidad con el lector de seguridad y el administrador de seguridad.

### <a name="other-updates"></a>Otras actualizaciones

#### <a name="access-group---zone-permissions"></a>Grupo de acceso: permisos de zona
  
Las reglas del grupo de acceso de la consola de administración local no incluirán la opción de conceder acceso a una zona específica. No hay ningún cambio en la definición de reglas que usan sitios, regiones y unidades de negocio.   Tras la actualización, los grupos de acceso que contengan reglas que permitan el acceso a zonas específicas se modificarán para permitir el acceso a su sitio principal, incluidas todas sus zonas.

#### <a name="terminology-changes"></a>Cambios de terminología

Se ha cambiado el nombre del término "recurso" y ha pasado a denominarse "dispositivo" en el sensor y en la consola de administración local, así como en los informes y otras interfaces de la solución.
En las alertas del sensor y de la consola de administración local, el término "administrar este evento" se denomina ahora "pasos de corrección".

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Defender para IoT](getting-started.md)
