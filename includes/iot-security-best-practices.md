---
title: archivo de inclusión
description: archivo de inclusión
services: iot-fundamentals
author: robinsh
ms.service: iot-fundamentals
ms.topic: include
ms.date: 08/07/2018
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 4fdb891d668d99644d8a9ed9c15d158e65d53ba5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "72793117"
---
La protección de una infraestructura de Internet de las cosas precisa de una estrategia de seguridad rigurosa y detallada. La eficacia de esta estrategia se basa en la protección de los datos en la nube, la protección de la integridad de los datos en su tránsito por la red pública de Internet y el aprovisionamiento de dispositivos de forma segura. Cada capa genera mayor garantía de seguridad en toda la infraestructura.

## <a name="secure-an-iot-infrastructure"></a>Protección de una infraestructura de IoT

Esta estrategia de seguridad detallada puede desarrollarse y ejecutarse con la participación activa de diversas personas implicadas en la fabricación, el desarrollo y la implementación de infraestructura y dispositivos IoT. A continuación verá una descripción de alto nivel de estos participantes.

* **Integrador o fabricante de hardware de IoT**: habitualmente, estos actores son los fabricantes del hardware de IoT implementado, integradores que ensamblan hardware de diversos fabricantes o proveedores que proporcionan hardware para una implementación de IoT que fabrican o integran otros proveedores.

* **Desarrollador de soluciones de IoT**: el desarrollo de soluciones de IoT suelen realizarlo los desarrolladores de soluciones. Este desarrollador puede formar parte de un equipo interno o ejercer como integrador de sistemas especializado en esta actividad. El desarrollador de soluciones de IoT puede desarrollar diversos componentes de la solución de IoT desde cero, integrar varios componentes de serie o de código abierto o adoptar aceleradores de soluciones mediante adaptaciones menores.

* **Implementador de soluciones de IoT**: una vez que se desarrolla una solución de IoT, es preciso implementarla sobre el terreno. Este proceso implica la implementación de hardware, la interconexión de dispositivos y la implementación de soluciones en dispositivos de hardware o en la nube.

* **Operador de soluciones de IoT**: una vez que la solución de IoT se implementa, requiere operaciones, supervisión, actualizaciones y mantenimiento a largo plazo. De estas tareas puede encargarse un equipo interno formado por especialistas en tecnología de la información, equipos de operaciones de hardware y mantenimiento y especialistas en dominios que supervisan que la infraestructura IoT funciona de manera correcta globalmente.

En las secciones que siguen se proporcionan los procedimientos recomendadas para cada uno de los implicados a fin de desarrollar, implementar y poner en funcionamiento una infraestructura de IoT segura.

## <a name="iot-hardware-manufacturerintegrator"></a>Integrador/fabricante de hardware IoT

Estos son los procedimientos recomendados para los fabricantes de hardware de IoT e integradores de hardware.

* **Acotar el hardware a unos requisitos mínimos**: el diseño del hardware debe incluir únicamente las características mínimas necesarias para su funcionamiento. Un ejemplo es la inclusión de puertos USB únicamente si es necesario para el funcionamiento del dispositivo. Estas características adicionales abren el dispositivo para vectores de ataques no deseados, los cuales deben evitarse.

* **Crear hardware a prueba de manipulaciones**: mecanismos integrados para detectar la alteración física del hardware; por ejemplo, abrir la cubierta del dispositivo o quitar una parte de este. Estas señales de alteración pueden formar parte de la transmisión de datos que se carga en la nube y que podría alertar a los operadores sobre estos eventos.

* **Basarse en hardware seguro**: si COGS lo permite, cree características de seguridad como el almacenamiento seguro y cifrado, o la funcionalidad de arranque basada en el Módulo de plataforma segura (TPM). Estas características crean dispositivos más seguros y ayudan a proteger la infraestructura de IoT general.

* **Hacer que las actualizaciones sean seguras**: las actualizaciones del firmware son inevitables durante la vigencia del dispositivo. La creación de dispositivos con rutas de acceso seguras para las actualizaciones y la garantía criptográfica de la versión del firmware permitirá la protección del dispositivo durante y después de las actualizaciones.

## <a name="iot-solution-developer"></a>Desarrollador de soluciones de IoT

Estos son los procedimientos recomendados para los desarrolladores de soluciones de IoT:

* **Seguir una metodología de desarrollo de software seguro**: el desarrollo de software seguro requiere pensar bien la seguridad, desde la concepción del proyecto hasta su implementación, prueba y desarrollo. Esta metodología influye en la elección de plataformas, idiomas y herramientas. El Ciclo de vida de desarrollo de seguridad de Microsoft proporciona un enfoque paso a paso para la creación de software seguro.

* **Elegir software de código abierto con cuidado**: el software de código abierto ofrece una oportunidad para desarrollar soluciones con rapidez. Al elegir software de código abierto, tenga en cuenta el nivel de actividad de la comunidad para cada componente de código abierto. Una comunidad activa garantiza la compatibilidad con el software; se detectarán problemas, a los que se hará frente. La otra opción es que no se admita un proyecto de software de código abierto complejo e inactivo, y así es probable es que no se detecten problemas.

* **Integrar con cuidado**: muchas de las brechas de seguridad del software aparecen en el límite de las bibliotecas y las API. Es posible que la funcionalidad que puede no requerirse para la implementación actual siga estando disponible a través de un nivel de API. Para garantizar la seguridad general, asegúrese de comprobar que no existen errores de seguridad en todas las interfaces de los componentes que se están integrando.

## <a name="iot-solution-deployer"></a>Implementador de soluciones de IoT

A continuación se muestran los procedimientos recomendados para los implementadores de soluciones de IoT:

* **Implementar hardware de forma segura**: las implementaciones de IoT pueden requerir la implementación de hardware en ubicaciones no seguras; por ejemplo, en espacios públicos o configuraciones regionales no supervisadas. En este tipo de situaciones, asegúrese de que la implementación de hardware sea lo más resistente posible a las manipulaciones. Si hay puertos USB o de otro tipo disponibles en el hardware, asegúrese de que estos se tratan de forma segura. Muchos vectores de ataques pueden usarlos como punto de entrada.

* **Mantener seguras las claves de autenticación**: durante la implementación, cada dispositivo requiere identificadores de dispositivo y claves de autenticación asociadas generadas por el servicio en la nube. Mantenga seguras estas claves físicamente incluso después de la implementación. Un dispositivo malintencionado puede usar cualquier clave en peligro para su enmascaramiento como dispositivo existente.

## <a name="iot-solution-operator"></a>Operador de soluciones de IoT

Estos son los procedimientos recomendados para los operadores de soluciones de IoT:

* **Mantener el sistema actualizado**: asegúrese de que los sistemas operativos y todos los controladores de los dispositivos están actualizados con las versiones más recientes. Si activa las actualizaciones automáticas en Windows 10 (IoT u otras SKU), Microsoft mantiene el equipo actualizado, ofreciendo a los dispositivos de IoT un sistema operativo seguro. En el caso de otros sistemas operativos (como Linux), el hecho de mantenerlos actualizados garantiza también su protección contra ataques malintencionados.

* **Proteger contra la actividad malintencionada**: si el sistema operativo lo permite, instale las funcionalidades antivirus y antimalware más recientes en el sistema operativo de todos y cada uno de los dispositivos. Esta práctica puede ayudar a mitigar las amenazas más externas. Puede proteger los sistemas operativos más modernos contra amenazas realizando los pasos adecuados.

* **Auditar con frecuencia**: la auditoría de los problemas relacionados con la seguridad de la infraestructura de IoT es clave a la hora de responder a incidentes de seguridad. La mayoría de los sistemas operativos proporciona un registro de eventos integrado que se debe revisar con frecuencia para asegurarse de que no se haya producido ninguna infracción de seguridad. La información de la auditoría se puede enviar como transmisión independiente de telemetría al servicio en la nube, donde puede analizarse.

* **Proteger físicamente la infraestructura de IoT**: los peores ataques contra la seguridad de la infraestructura de IoT se inician mediante el acceso físico a los dispositivos. La protección contra el uso malintencionado de puertos USB y otros accesos físicos es una importante práctica de seguridad. Una clave para descubrir las infracciones que pudieran haberse producido es registrar el acceso físico, como el uso del puerto USB. Una vez más, Windows 10 (IoT y el resto de SKU) habilita el registro detallado de estos eventos.

* **Proteger las credenciales en la nube**: es posible que las credenciales de autenticación en la nube usadas para configurar y manejar una implementación de IoT sean la forma más fácil de obtener acceso a un sistema IoT y ponerlo en peligro. Proteja las credenciales cambiando la contraseña con frecuencia y no usándolas en equipos públicos.

Las funcionalidades de los distintos dispositivos IoT varían. Algunos dispositivos pueden ser equipos que ejecutan sistemas operativos de escritorio comunes, y otros pueden estar ejecutando sistemas operativos muy ligeros. Los procedimientos recomendados de seguridad descritos anteriormente pueden ser aplicables a estos dispositivos en diverso grado. En caso de proporcionarse, deben seguirse las prácticas recomendadas de seguridad e implementación facilitadas por el fabricante de estos dispositivos.

Puede que algunos dispositivos heredados y limitados no se hayan diseñado de forma específica para la implementación de IoT. Estos dispositivos pueden carecer de la funcionalidad para cifrar datos, conectarse a Internet o proporcionar auditoría avanzada. En estos casos, una puerta de enlace de campo moderna y segura puede agregar datos desde dispositivos heredados y proporcionar la seguridad necesaria para la conexión a Internet de estos dispositivos. Las puertas de enlace de campo proporcionan una autenticación segura, una negociación de sesiones cifradas, la recepción de comandos desde la nube y muchas otras características de seguridad.