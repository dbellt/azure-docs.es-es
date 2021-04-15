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
ms.openlocfilehash: 28609ad27330ae4ea5ea7c0d02d5a61181fbe0df
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95560966"
---
Cuando se diseña un sistema, es importante conocer las posibles amenazas a las que puede estar expuesto y agregar las defensas adecuadas según corresponda durante su diseño y arquitectura. Es importante diseñar el producto desde el principio teniendo en cuenta la seguridad, ya que conocer la forma en que un atacante podría poner en peligro un sistema ayuda a tomar las medidas pertinentes desde el principio.

## <a name="security-starts-with-a-threat-model"></a>La seguridad comienza con un modelo de riesgos

Microsoft leva mucho tiempo usando modelos de riesgos para sus productos y ha puesto a disposición del público el proceso de modelado de riesgos de la compañía. La experiencia de la compañía muestra que el modelado tiene ventajas inesperadas que van más allá del conocimiento inmediato de qué amenazas son las más preocupantes. Por ejemplo, también facilita el debate abierto con usuarios que no pertenecen al equipo de desarrollo, lo que puede aportar nuevas ideas y mejoras al producto.

El objetivo del modelado de riesgos es conocer la forma en que un atacante puede poner en peligro un sistema y, luego, asegúrese de que se toman las medidas adecuadas para evitarlo. El modelado de riesgos obligan al equipo de diseño a considerar las mitigaciones durante el diseño del sistema, en lugar de después de que este se implementa. Este hecho es extremadamente importante, ya que la actualización de las defensas de la seguridad de un gran número de dispositivos en el campo es inviable, propensa a errores y pone en peligro a los clientes.

Muchos equipos de desarrollo realizan un excelente trabajo, ya que capturan los requisitos funcionales del sistema que benefician a los clientes. Sin embargo, la identificación de formas no obvias en que alguien podría hacer un uso incorrecto del sistema es un reto mayor. El modelado de riesgos puede ayudar a los equipos de desarrollo a conocer lo que podría hacer un atacante, y por qué. El modelado de riesgos es un proceso estructurado que crea una discusión acerca de las decisiones que se toman en el diseño de la seguridad del sistema, así como acerca de los cambios que se realizan en el diseño que afectan a la seguridad. Aunque un modelo de riesgos no es más que un documento, esta documentación también representa una forma ideal de garantizar la continuidad del conocimiento y la retención de lecciones aprendidas, y ayuda al nuevo equipo a incorporarse rápidamente. Por último, una de las consecuencias del modelado de riesgos es que permite considerar otros aspectos de la seguridad, como los compromisos relativos a la seguridad que desea proporcionar a sus clientes. Estos compromisos, junto con el modelado de riesgos, controlan su solución de Internet de las cosas e informan sobre la misma.

### <a name="when-to-do-threat-modeling"></a>Cuándo hacer un modelo de amenazas

El [modelado de riesgos](https://www.microsoft.com/en-us/sdl/adopt/threatmodeling.aspx) ofrece el máximo valor cuando lo incorpora en la fase de diseño. Cuando realice el diseño, tendrá la máxima flexibilidad para realizar los cambios necesarios para eliminar las amenazas. La eliminación de amenazas a través del diseño es el resultado deseado. Este método es mucho más fácil que agregar mitigaciones, probarlas y asegurarse de que siempre están actualizadas; además, su eliminación no siempre es posible. A medida que un producto alcanza su madurez, resulta más difícil eliminar las amenazas y, en última instancia, requiere más trabajo y equilibrios mucho más duros que si el modelado de riesgos se realiza en las primeras fases del desarrollo.

### <a name="what-to-consider-for-threat-modeling"></a>Lo que debe tener en cuenta para el modelado de amenazas

Debe mirar la solución en conjunto, pero también centrarse en las áreas siguientes:

* Las características de privacidad y seguridad
* Las características cuyos errores son pertinentes para la seguridad
* Las características que tocan un límite de confianza

### <a name="who-performs-threat-modeling"></a>Quién realiza el modelado de amenazas

El modelado de riesgos es un proceso como cualquier otro. Se recomienda tratar el documento del modelo de riesgos como cualquier otro componente de la solución y validarlo. Muchos equipos de desarrollo realizan un excelente trabajo, ya que capturan los requisitos funcionales del sistema que benefician a los clientes. Sin embargo, la identificación de formas no obvias en que alguien podría hacer un uso incorrecto del sistema es un reto mayor. El modelado de riesgos puede ayudar a los equipos de desarrollo a conocer lo que podría hacer un atacante, y por qué.

### <a name="how-to-perform-threat-modeling"></a>Cómo realizar el modelado de amenazas

El proceso del modelado de riesgos consta de cuatro pasos, que son:

* Modelar la aplicación
* Enumerar las amenazas
* Mitigar las amenazas
* Validar las mitigaciones

#### <a name="the-process-steps"></a>Los pasos del proceso

Cuando se crea un modelo de riesgos es preciso tener en cuenta tres reglas generales:

1. Cree un diagrama de la arquitectura de referencia.

2. Inicie la búsqueda en anchura. Obtenga información general y entienda el sistema como un todo, antes de realizar la búsqueda en profundidad. Con este enfoque se asegurará de que realiza la búsqueda en profundidad en los lugares correctos.

3. Maneje el proceso, no permita que el proceso le maneje a usted. Si encuentra algún problema en la fase de modelado y desea explorarlo, hágalo. No es preciso que siga estos pasos ciegamente.

#### <a name="threats"></a>Amenazas

Los cuatro elementos principales de un modelo de riesgos son:

* Procesos como servicios web, servicios Win32 y demonios *nix. Algunas entidades complejas (por ejemplo, sensores y puertas de enlace de campo) se pueden resumir como procesos cuando no es posible explorar estas áreas en profundidad.

* Almacenes de datos (todos los lugares donde se almacenan los datos, como un archivo de configuración o una base de datos)

* Flujo de datos (donde los datos se mueven entre otros elementos de la aplicación)

* Entidades externas (todo lo que interactúa con el sistema, pero que no está bajo el control de la aplicación; algunos ejemplos incluyen los usuarios y las fuentes de satélite)

Todos los elementos del diagrama arquitectónico están sujetos a diversas amenazas. En este artículo se usará el mnemotécnico STRIDE. Para más información sobre los elementos de STRIDE, consulte el artículo [Threat Modeling Again, STRIDE](/archive/blogs/larryosterman/threat-modeling-again-stride) (Modelado de riesgos otra vez, STRIDE).

Los distintos elementos del diagrama de las aplicaciones están sujetos a determinadas amenazas de STRIDE:

* Los procesos están sujetos a STRIDE
* Los flujos de datos están sujetos a TID
* Los almacenes de datos están sujetos a TID y, a veces a R, cuando los almacenes de datos son archivos de registro.
* Las entidades externas están sujetas a SRD

## <a name="security-in-iot"></a>Seguridad de IoT

Los dispositivos conectados de uso especial tienen un importante número de posibles áreas de superficie de interacción y patrones de interacción, y todos ellos deben considerarse a la hora de proporcionar un marco para proteger el acceso digital a los dispositivos. El término "acceso digital" se usa aquí para establecer la diferencia con las operaciones que se llevan a cabo a través de la interacción directa de dispositivos en las que la seguridad del acceso se proporciona a través del control físico del acceso. Por ejemplo, colocar el dispositivo en una sala con una cerradura en la puerta. Aunque el acceso físico no se puede denegar mediante software y hardware, se pueden tomar medidas para evitar que el acceso físico provoque interferencia en el sistema.

Al explorar patrones de interacción, examine el "control del dispositivo" y los "datos del dispositivo" con el mismo nivel de atención. El "control del dispositivo" se puede clasificar como toda la información que cualquiera de las partes proporcione a un dispositivo con el objetivo de cambiar o influir en el comportamiento, en relación con su estado o el estado de su entorno. Los "datos del dispositivo" se pueden clasificar como toda la información que emite un dispositivo a cualquier otra parte acerca de su estado y el estado que se percibe de su entorno.

Para optimizar los procedimientos de seguridad recomendados, se recomienda dividir las arquitecturas de IoT típicas en varios componentes o zonas como parte del ejercicio de modelado de riesgos. Dichas zonas se describen con detalle en esta sección e incluyen:

* Dispositivo,
* Puerta de enlace de campo,
* Puertas de enlace en la nube y
* Servicios.

Las zonas son formas amplias de segmentar una solución; a menudo, cada una de las zonas tiene sus propios datos y requisitos de autenticación y autorización. Las zonas también se pueden usar para aislar los daños y restringir el impacto de las zonas de confianza baja en las zonas de confianza superior.

Cada zona se separa mediante un límite de confianza, que se indica con la línea de puntos roja en el diagrama siguiente. Representa una transición de datos o información de un origen a otro. Durante esta transición, los datos o la información podrían estar sujetos a suplantación de identidad, manipulación, rechazo, revelación de información, denegación de servicio y elevación de privilegios (STRIDE).

![Zonas de seguridad de IoT](media/iot-security-architecture/iot-security-architecture-fig1.png) 

Los componentes que se representan dentro de cada límite también se someten a STRIDE, lo que permite una vista completa de 360 grados del modelado de riesgos de la solución. En las secciones siguientes se desarrollan tanto los componentes como problemas de seguridad específicos y las soluciones que se deben aplicar.

En las secciones siguientes se tratan componentes estándar que suelen encontrarse en estas zonas.

### <a name="the-device-zone"></a>La zona del dispositivo

El entorno del dispositivo es el espacio físico más próximo que rodea al dispositivo, donde el acceso físico o el acceso digital de punto a punto de la “red local” al dispositivo. Se asume que una "red local" es una red que es única y está aislada de Internet (aunque se puede conectar) e incluye cualquier tecnología de radio inalámbrica de corto alcance que permite la comunicación punto a punto de dispositivos. *No* incluye tecnologías de virtualización de redes que creen la ilusión de red local ni tampoco redes de operadores públicos que requieran que dos dispositivos cualesquiera se comuniquen a través de un espacio de la red pública si van a establecer una relación de comunicación punto a punto.

### <a name="the-field-gateway-zone"></a>La zona de puerta de enlace de campo

Una puerta de enlace de campo es un dispositivo (o aparato) o un software de equipo servidor de uso general que actúa como habilitador de comunicaciones y, potencialmente, como sistema de control de dispositivos y centro de procesamiento de datos de dispositivos. La zona de puerta de enlace de campo incluye la propia puerta de enlace de campo y todos los dispositivos conectados a ella. Como su nombre implica, las puertas de enlace de campo actúan fuera de las instalaciones dedicadas al procesamiento de datos, suelen estar enlazadas a una ubicación, pueden estar sujetas a intrusiones físicas y tienen una redundancia operativa limitada. En resumen, una puerta de enlace de campo suele se algo que cualquier persona puede tocar, e incluso sabotear si sabe cuál es su función.

Una puerta de enlace de campo se diferencia de un enrutador de tráfico en que tiene un rol activo en la administración del acceso y del flujo de información, lo que significa que es una entidad y terminal de sesión o de conexión de red dirigidos a aplicaciones. Por el contrario, los dispositivos NAT o los firewalls no son aptos como puertas de enlace de campo. Esto se debe a que no son terminales de sesión ni de conexión explícitos, sino que enrutan (o bloquean) las conexiones o sesiones que se realizan a través de ellos. La puerta de enlace de campo tiene dos áreas expuestas claramente diferenciadas. Una de ellas «mira» a los dispositivos que se conectan a ella y representa el interior de la zona, mientras que el otro «mira» a todas las partes externas y es el borde de la zona.

### <a name="the-cloud-gateway-zone"></a>La zona de la puerta de enlace en la nube

Una puerta de enlace en la nube es un sistema que permite la comunicación remota desde y hacia los dispositivos o puertas de enlace de campo de varios sitios a través de un espacio de la red pública, normalmente hacia un sistema de análisis de datos y de control basado en la nube, una federación de dichos sistemas. En algunos casos, una puerta de enlace en la nube puede facilitar inmediatamente el acceso a los dispositivos especiales desde terminales como teléfonos o tabletas. En el contexto que se trata aquí, "nube" hace referencia a un sistema dedicado al procesamiento de datos que no está enlazado con el mismo sitio que los dispositivos o puertas de enlace de campo conectados. Además, en una zona de nube, las medidas operativas también impiden el acceso físico dirigido y no necesariamente se exponen a una infraestructura de "nube pública".  

Una puerta de enlace en la nube puede asignarse a una superposición de virtualización de redes para aislar la puerta de enlace en la nube y todos sus dispositivos o puertas de enlace de campo conectados del resto del tráfico de red. La puerta de enlace en la nube no es un sistema de control de dispositivos ni unas instalaciones de almacenamiento o procesamiento de los datos de los dispositivos; dichas instalaciones hacen de interfaz con la puerta de enlace en la nube. La zona de la puerta de enlace en la nube incluye la propia puerta de enlace en la nube, junto con todas las puertas de enlace de campo y los dispositivos conectados directa o indirectamente a ella. El borde de la zona es un área expuesta distinta a través de la que se comunican todas las partes externas.

### <a name="the-services-zone"></a>La zona de servicios

En este contexto, un "servicio" se define como cualquier componente de software o módulo que interactúa con dispositivos a través de una puerta de enlace de campo o en la nube para la recopilación y análisis de datos, así como para el comando y control. Los servicios son mediadores. Actúan bajo su identidad de cara a las puertas de enlace y otros subsistemas, almacenan y analizan datos, emiten comandos de forma autónoma a los dispositivos basándose en las programaciones o perspectivas sobre los datos, y exponen información y capacidades de control a los usuarios finales autorizados.

### <a name="information-devices-versus-special-purpose-devices"></a>Dispositivos de información frente a dispositivos especiales

Los equipos, los teléfonos y las tabletas son principalmente dispositivos de información interactivos. Los teléfonos y las tabletas están optimizados explícitamente para maximizar la duración de la batería. Preferiblemente se apagan parcialmente cuando no interactúan inmediatamente con una persona o cuando no proporcionan servicios, como reproducir música o guiar a su propietario hasta una ubicación determinada. Desde una perspectiva de los sistemas, estos dispositivos de tecnología de la información actúan principalmente como servidores proxy hacia las personas. Son “accionadores de personas” que sugieren acciones y “sensores de personas” que recopilan datos de entrada.

Los dispositivos especiales, desde los simples sensores de temperatura hasta las complejas las líneas de producción de una fábrica con miles de componentes dentro de ellos, son diferentes. La finalidad de estos dispositivos está mucho más limitada y aunque proporcionan una interfaz de usuario, principalmente se limitan a interactuar con recursos del mundo físico, o a integrarse en ellos. Miden e informan acerca de las circunstancias medioambientales de informes, giran válvulas, controlar servodirecciones, hacen que suenen alarmas y encienden luces, entre otras muchas tareas. Ayudan a realizar aquellos trabajos para los que un dispositivo de información es demasiado genérico, demasiado caro, demasiado grande o demasiado complicado. El propósito concreto dicta inmediatamente su diseño técnico, así como el presupuesto disponible para su producción y la operación de duración programada. La combinación de estos dos factores clave restringe el presupuesto energético operativo disponible, la superficie física y, por consiguiente, las funcionalidades de almacenamiento, proceso y seguridad disponibles.

Si surge "algún problema" con dispositivos automáticos o que se controlen de forma remota, por ejemplo, defectos físicas o defectos de la lógica de control que permite la manipulación e intrusión no autorizadas deliberadas. Se pueden destruir los lotes de producción, los edificios se pueden saquear o quedar reducidos a cenizas, y las personas pueden resultar heridas, o incluso morir. Este tipo de daño es totalmente distinto del que produce alguien que agota el límite de una tarjeta de crédito robada. El nivel de seguridad de los dispositivos que dotan de movimiento a otros elementos y también los datos de los sensores que, en último término, generan comandos que provocan que las cosas se muevan debe ser mayor que el de los sectores bancario o de comercio electrónico.

### <a name="device-control-and-device-data-interactions"></a>Control de dispositivos e interacciones con datos de dispositivos

Los dispositivos conectados de uso especial tienen un importante número de posibles áreas de superficie de interacción y patrones de interacción, y todos ellos deben considerarse a la hora de proporcionar un marco para proteger el acceso digital a los dispositivos. El término "acceso digital" se usa aquí para establecer la diferencia con las operaciones que se llevan a cabo a través de la interacción directa de dispositivos en las que la seguridad del acceso se proporciona a través del control físico del acceso. Por ejemplo, colocar el dispositivo en una sala con una cerradura en la puerta. Aunque el acceso físico no se puede denegar mediante software y hardware, se pueden tomar medidas para evitar que el acceso físico provoque interferencia en el sistema.

Al explorar patrones de interacción, examine el "control del dispositivo" y los "datos del dispositivo" con el mismo nivel de atención que los modelos de riesgos. El "control del dispositivo" se puede clasificar como toda la información que cualquiera de las partes proporcione a un dispositivo con el objetivo de cambiar o influir en el comportamiento, en relación con su estado o el estado de su entorno. Los "datos del dispositivo" se pueden clasificar como toda la información que emite un dispositivo a cualquier otra parte acerca de su estado y el estado que se percibe de su entorno.

## <a name="performing-threat-modeling-for-the-azure-iot-reference-architecture"></a>Modelado de riesgos de la arquitectura de referencia de Azure IoT

Microsoft usa el marco esbozado anteriormente para realizar el modelado de riesgos para IoT de Azure. En la sección siguiente se usa el ejemplo concreto de la arquitectura de referencia de IoT de Azure para mostrar cómo reflexionar sobre el modelado de riesgos para IoT y cómo tratar las amenazas identificadas. En este ejemplo se identifican cuatro áreas principales de atención:

* Dispositivos y orígenes de datos,
* Transporte de datos,
* Dispositivo y procesamiento de eventos
* Presentación

![Modelado de riesgos para IoT de Azure](media/iot-security-architecture/iot-security-architecture-fig2.png)

En el diagrama siguiente se proporciona una vista simplificada de la arquitectura de IoT de Microsoft mediante un modelo de diagrama de flujo de datos que usa Microsoft Threat Modeling Tool:

![Modelado de riesgos para IoT de Azure mediante la herramienta de modelado de riesgos de MS](media/iot-security-architecture/iot-security-architecture-fig3.png)

Es importante tener en cuenta que la arquitectura separa las funcionalidades del dispositivo y de la puerta de enlace. Este enfoque permite al usuario sacar provecho de los dispositivos de puerta de enlace que sean más seguros: son capaces de comunicarse con la puerta de enlace en la nube mediante protocolos seguros, lo que normalmente requiere mayor sobrecarga de procesamiento de la que un dispositivo nativo (como un termostato) podría proporcionar por sí mismo. En la zona de servicios de Azure, suponga que el servicio Azure IoT Hub representa la puerta de enlace en la nube.

### <a name="device-and-data-sourcesdata-transport"></a>Dispositivo y orígenes de datos o transporte de datos

En esta sección se explora la arquitectura esbozada anteriormente a través de la lente del modelado de riesgos y se ofrece información general de la forma en que se tratan algunos de los problemas inherentes. Este ejemplo se centra en los elementos principales de un modelo de riesgos:

* Procesos (los que están bajo su control y los elementos externos)
* Comunicación (denominado también flujos de datos)
* Almacenamiento (denominado también almacenes de datos)

#### <a name="processes"></a>Procesos

En cada una de las categorías esbozadas en la arquitectura de IoT de Azure, en este ejemplo se intentan mitigar varias amenazas diferentes en las distintas fases en que existen información o datos: proceso, comunicación y almacenamiento. A continuación, se ofrece información general de las más comunes para la categoría de "proceso", seguida de información general de cuál sería la mejor forma de mitigar estas amenazas:

**Suplantación de identidad (S)** : un atacante puede extraer material de la clave criptográfica de un dispositivo, en el nivel de hardware o en el de software y posteriormente acceder al sistema con un dispositivo físico o virtual diferente bajo la identidad del dispositivo del que se ha tomado el material de la clave. Un buen ejemplo son los mandos a distancia que pueden encender cualquier televisor y que son herramientas populares para gastar bromas.

**Denegación de servicio (D)** : un dispositivo se puede representar como incapaz de funcionar o comunicarse al interferir con frecuencias de radio o cortar hilos. Por ejemplo, una cámara de vigilancia cuya alimentación o conexión de red se ha interrumpido intencionadamente no puede notificar datos.

**Manipulación (T)** : un atacante puede reemplazar total o parcialmente el software que se ejecuta en el dispositivo, lo que potencialmente permite que el software reemplazado use la identidad genuina del dispositivo si el material clave o las instalaciones criptográficas que contienen materiales clave estaban disponibles para el programa ilícito. Por ejemplo, un atacante puede sacar provecho del material clave extraído para interceptar y suprimir datos del dispositivo en la ruta de acceso de comunicación y reemplazarlos por datos falsos que se han autenticado con el material clave robado.

**Divulgación de información (I)** : si el dispositivo ejecuta software manipulado, dicho software puede proporcionar datos a partes no autorizadas. Por ejemplo, un atacante puede sacar provecho del material clave extraído para inyectarse a sí mismo en la ruta de acceso de comunicación entre el dispositivo y un controlador o la puerta de enlace del campo o la puerta de enlace en la nube para desviar información.

**Elevación de privilegios (E)** : se puede forzar a un dispositivo que realiza una función específica a realizar otra función. Por ejemplo, se puede obligar a una válvula programada para abrirse a la mitad a abrirse completamente.

| **Componente** | **Amenaza** | **Mitigación** | **Riesgo** | **Implementación** |
| --- | --- | --- | --- | --- |
| Dispositivo |S |Asignación de identidad para el dispositivo y autenticación del dispositivo |Reemplazo de dispositivo, o parte del dispositivo, por otro dispositivo. ¿Cómo se sabe que se habla al dispositivo correcto? |Autenticación del dispositivo mediante Seguridad de capa de transporte (TLS) o IPSec. La infraestructura debe admitir el uso de una clave precompartida (PSK) en los dispositivos que no pueden controlar la criptografía asimétrica completa. Aprovechamiento de Azure AD, [OAuth](https://www.rfc-editor.org/pdfrfc/rfc6755.txt.pdf) |
|| TRID |Aplicar mecanismos a prueba de manipulaciones al dispositivo, por ejemplo, haciendo que sea casi imposible extraer claves y otro material criptográfico del dispositivo. |El riesgo es que alguien altere el dispositivo (interferencias físicas). ¿Cómo se tiene la certeza de que el dispositivo no se ha alterado? |La mitigación más eficaz es una funcionalidad de Módulo de plataforma segura (TPM) que permita almacenar claves en el conjunto de circuitos del procesador desde el que no se pueden leer las claves, solo se pueden usar para las operaciones criptográficas que utilizan la clave, pero nunca la revelan. Cifrado de la memoria del dispositivo. Administración de claves para el dispositivo. Firma del código. |
|| E |Tener el control de acceso del dispositivo. Esquema de autorización. |Si el dispositivo permite que se realicen acciones individuales con los comandos de un origen externo o incluso sensores en peligro, permite que el ataque realice operaciones a las que no podría acceder de otra forma. |Tener el esquema de autorización del dispositivo |
| Puerta de enlace de campo |S |Autenticación de la puerta de enlace de campo en la puerta de enlace en la nube (como basada en certificado, PSK o basada en notificación). |Si alguien puede suplantar la identidad de la puerta de enlace de campo, puede presentarse como cualquier dispositivo. |TLS RSA/PSK, IPSec, [RFC 4279](https://tools.ietf.org/html/rfc4279). Los mismos problemas de almacenamiento de claves y atestación de los dispositivos en general (lo mejor es usar TPM). Extensión de 6LowPAN para IPSec para admitir redes de sensores inalámbricas (WSN). |
|| TRID |Proteger la puerta de enlace de campo contra la manipulación (¿TPM?) |Los ataques de suplantación de identidad que hacen creer a la puerta de enlace en la nube que habla con la puerta de enlace de campo pueden provocar la divulgación de información y la manipulación de datos |Cifrado de memoria, TPM y autenticación. |
|| E |Mecanismo de control de acceso de una puerta de enlace de campo | | |

Estos son algunos ejemplos de las amenazas de esta categoría:

**Suplantación de identidad**: un atacante puede extraer material de la clave criptográfica de un dispositivo, en el nivel de hardware o en el de software y posteriormente acceder al sistema con un dispositivo físico o virtual diferente bajo la identidad del dispositivo del que se ha tomado el material de la clave.

**Denegación de servicio**: un dispositivo se puede representar como incapaz de funcionar o comunicarse al interferir con frecuencias de radio o cortar hilos. Por ejemplo, una cámara de vigilancia cuya alimentación o conexión de red se ha interrumpido intencionadamente no puede notificar datos.

**Manipulación**: un atacante puede reemplazar total o parcialmente el software que se ejecuta en el dispositivo, lo que potencialmente permite que el software reemplazado use la identidad genuina del dispositivo si el material clave o las instalaciones criptográficas que contienen materiales clave estaban disponibles para el programa ilícito.

**Manipulación**: una cámara de vigilancia que muestra una imagen del espectro visible de un pasillo vacío puede dirigirse a una fotografía de dicho pasillo. Un detector de humo o de incendios puede saltar si hay alguien debajo con un mechero encendido. En cualquier caso, técnicamente el dispositivo puede ser de total confianza hacia el sistema, pero presenta información manipulada.

**Manipulación**: un atacante puede usar el material clave extraído para interceptar y suprimir datos del dispositivo en la ruta de acceso de comunicación y reemplazarlos por datos falsos que se han autenticado con el material clave robado.

**Manipulación**: un atacante puede reemplazar total o parcialmente el software que se ejecuta en el dispositivo, lo que potencialmente permite que el software reemplazado use la identidad genuina del dispositivo si el material clave o las instalaciones criptográficas que contienen materiales clave estaban disponibles para el programa ilícito.

**Divulgación de información**: si el dispositivo ejecuta software manipulado, dicho software puede proporcionar datos a partes no autorizadas.

**Divulgación de información**: un atacante puede sacar provecho del material clave extraído para inyectarse a sí mismo en la ruta de acceso de comunicación entre el dispositivo y un controlador o la puerta de enlace del campo o la puerta de enlace en la nube para desviar información.

**Denegación de servicio**: el dispositivo se puede desactivar o activar en un modo en el que la comunicación no sea posible (lo que es intencionado en muchas máquinas industriales).

**Manipulación**: el dispositivo se puede reconfigurar para que opere en un estado desconocido para el sistema de control (fuera de los parámetros de calibrado conocidos) y así proporcionar datos que se puedan interpretar erróneamente.

**Elevación de privilegios**: se puede forzar a un dispositivo que realiza una función específica a realizar otra función. Por ejemplo, se puede obligar a una válvula programada para abrirse a la mitad a abrirse completamente.

**Denegación de servicio**: el dispositivo se puede pasar a un estado en el que no sea posible la comunicación.

**Manipulación**: el dispositivo se puede reconfigurar para que opere en un estado desconocido para el sistema de control (fuera de los parámetros de calibrado conocidos) y así proporcionar datos que se puedan interpretar erróneamente.

**Suplantación de identidad, manipulación o rechazo**: si no está protegido (que casi nunca es el caso de los mandos a distancia de los consumidores), un atacante puede manipular el estado de un dispositivo de forma anónima. Un buen ejemplo son los mandos a distancia que pueden encender cualquier televisor y que son herramientas populares para gastar bromas.

#### <a name="communication"></a>Comunicación

Las amenazas existentes en la ruta de acceso de comunicación entre los dispositivos, entre los dispositivos y las puertas de enlace de campo, y entre los dispositivos y la puerta de enlace en la nube. En la tabla siguiente se proporcionan instrucciones sobre los sockets abiertos del dispositivo/VPN:

| **Componente** | **Amenaza** | **Mitigación** | **Riesgo** | **Implementación** |
| --- | --- | --- | --- | --- |
| IoT Hub de dispositivo |TID |(D)TLS (PSK/RSA) para cifrar el tráfico |La interceptación o interferencia de la comunicación entre el dispositivo y la puerta de enlace |Seguridad en el nivel de protocolo. En el caso de los protocolos personalizados, debe averiguar cómo protegerlos. En la mayoría de los casos, la comunicación se establece desde el dispositivo a IoT Hub (el dispositivo inicia la conexión). |
| De dispositivo a dispositivo |TID |(D)TLS (PSK/RSA) para cifrar el tráfico. |Lectura de los datos en tránsito entre dispositivos. Manipulación de los datos. Sobrecarga del dispositivo con nuevas conexiones |Seguridad en el nivel de protocolo (MQTT/AMQP/HTTP/CoAP). En el caso de los protocolos personalizados, debe averiguar cómo protegerlos. La mitigación de la amenaza de denegación de servicio es emparejar los dispositivos a través de una puerta de enlace de campo o en la nube, y que actúen solo como clientes hacia la red. El emparejamiento puede dar lugar a una conexión directa entre los homólogos después de que la puerta de enlace haya intermediado |
| Entidad externa Dispositivo |TID |Emparejamiento seguro de la entidad externa con el dispositivo |Interceptación de la conexión con el dispositivo. Interferencia de la comunicación con el dispositivo |Emparejamiento seguro de la entidad externa con el dispositivo LE NFC o Bluetooth. Control del panel operativo del dispositivo (físico) |
| Puerta de enlace de campo Puerta de enlace en la nube |TID |TLS (PSK/RSA) para cifrar el tráfico. |La interceptación o interferencia de la comunicación entre el dispositivo y la puerta de enlace |Seguridad en el nivel de protocolo (MQTT/AMQP/HTTP/CoAP). En el caso de los protocolos personalizados, debe averiguar cómo protegerlos. |
| Puerta de enlace en la nube de dispositivo |TID |TLS (PSK/RSA) para cifrar el tráfico. |La interceptación o interferencia de la comunicación entre el dispositivo y la puerta de enlace |Seguridad en el nivel de protocolo (MQTT/AMQP/HTTP/CoAP). En el caso de los protocolos personalizados, debe averiguar cómo protegerlos. |

Estos son algunos ejemplos de las amenazas de esta categoría:

**Denegación de servicio**: los dispositivos restringidos suelen estar bajo la amenaza de ataque DoS cuando escuchan activamente conexiones entrantes o datagramas no solicitados en una red, porque un atacante puede abrir muchas conexiones en paralelo y no prestarles servicio o prestárselo lentamente, o bien el dispositivo se puede inundar con tráfico no solicitado. En ambos casos, el dispositivo puede representarse eficazmente como no operativo en la red.

**Suplantación, revelación de información**: tanto los dispositivos restringidos como los dispositivos especiales tienen a menudo funciones de seguridad del tipo "una para todos", como la protección mediante PIN o contraseña, o se basan totalmente en confiar en la red, lo que significa que conceden acceso a la información cuando un dispositivo se encuentra en la misma red y con frecuencia dicha red solo está protegida por una clave compartida. Esto significa que cuando el secreto compartido se revela al dispositivo o a la red, es posible controlar el dispositivo u observar los datos emitidos desde el dispositivo.  

**Suplantación**: un atacante puede interceptar o invalidar parcialmente la difusión y suplantar al originador ("Man in the middle").

**Alteración**: un atacante puede interceptar o invalidar parcialmente la difusión y enviar información falsa. 

**Divulgación de información**: un atacante puede interceptar una difusión y obtener información sin autorización. **Denegación de servicio**: un atacante puede bloquear la señal de difusión y denegar la distribución de la información.

#### <a name="storage"></a>Storage

Todos los dispositivos y puertas de enlace de campo tienen algún tipo de almacenamiento (temporal para poner en cola los datos, almacenamiento de imágenes del sistema operativo).

| **Componente** | **Amenaza** | **Mitigación** | **Riesgo** | **Implementación** |
| --- | --- | --- | --- | --- |
| Almacenamiento del dispositivo |TRID |Cifrado de almacenamiento, firma de los registros |Lectura de datos desde el almacenamiento de información (datos de DCP), manipulación de datos de telemetría. Manipulación de datos de control de comandos en cola o en la memoria caché. La manipulación de paquetes de actualización de configuración o firmware durante el almacenamiento en caché o en la cola local puede provocar que los componentes del sistema operativo o del sistema se encuentren en peligro. |Cifrado, código de autenticación de mensajes (MAC) o firma digital. Donde sea posible, control de acceso seguro a través de listas de control de acceso (ACL) a los recursos o permisos. |
| Imagen de sistema operativo de dispositivo |TRID | |Manipulación del sistema operativo o reemplazo de los componentes del sistema operativo |Partición de sistema operativo de solo lectura, imagen de sistema operativo firmada, cifrado |
| Almacenamiento en la puerta de enlace de campo (poner en cola los datos) |TRID |Cifrado de almacenamiento, firma de los registros |Leer datos desde el almacenamiento de información (datos de DCP), manipulación de datos de telemetría, manipule de datos de control de comandos en cola o almacenados en la caché. La manipulación de paquetes de actualización de configuración o firmware (destinados a los dispositivos o a la puerta de enlace de campo) durante el almacenamiento en caché o en la cola local puede provocar que los componentes del sistema operativo o del sistema se encuentren en peligro. |BitLocker |
| Imagen de sistema operativo de puerta de enlace de campo |TRID | |Manipulación del sistema operativo o reemplazo de los componentes del sistema operativo |Partición de sistema operativo de solo lectura, imagen de sistema operativo firmada, cifrado |

### <a name="device-and-event-processingcloud-gateway-zone"></a>Zona de procesamiento de dispositivos y eventos o de puerta de enlace en la nube

Una puerta de enlace en la nube es un sistema que permite la comunicación remota desde y hacia los dispositivos o puertas de enlace de campo de varios sitios a través de un espacio de la red pública, normalmente hacia un sistema de análisis de datos y de control basado en la nube, una federación de dichos sistemas. En algunos casos, una puerta de enlace en la nube puede facilitar inmediatamente el acceso a los dispositivos especiales desde terminales como teléfonos o tabletas. En el contexto que se trata aquí, "nube" hace referencia a un sistema dedicado al procesamiento de datos que no está enlazado con el mismo sitio que los dispositivos o puertas de enlace de campo conectados y en el que las medidas operativas evitan el acceso físico dirigido, pero no necesariamente a una infraestructura de "nube pública". Una puerta de enlace en la nube puede asignarse a una superposición de virtualización de redes para aislar la puerta de enlace en la nube y todos sus dispositivos o puertas de enlace de campo conectados del resto del tráfico de red. La puerta de enlace en la nube no es un sistema de control de dispositivos ni unas instalaciones de almacenamiento o procesamiento de los datos de los dispositivos; dichas instalaciones hacen de interfaz con la puerta de enlace en la nube. La zona de la puerta de enlace en la nube incluye la propia puerta de enlace en la nube, junto con todas las puertas de enlace de campo y los dispositivos conectados directa o indirectamente a ella.

Habitualmente, una puerta de enlace en la nube es un software personalizado integrado que se ejecuta como si fuera un servicio con puntos de conexión expuestos a los que se conectan tanto una puerta de enlace de campo como los dispositivos. Por lo tanto, debe diseñarse pensando en la seguridad. Siga el proceso de [SDL](https://www.microsoft.com/sdl) para diseñar y compilar este servicio.

#### <a name="services-zone"></a>Zona de servicios

Un sistema de control (o controlador) es una solución de software que interactúa con un dispositivo, una puerta de enlace de campo o una puerta de enlace en la nube y cuyo propósito es controlar uno o varios dispositivos, o recopilar o almacenar o analizar los datos del dispositivo para su presentación o con fines de control posterior. Los sistemas de control son las únicas entidades, entre las que entran dentro del ámbito de este análisis, que pueden facilitar inmediatamente la interacción con personas. Las excepciones son las superficies de control físico intermedias de dispositivos, como un conmutador que permite a una persona desactivar el dispositivo o cambiar otras propiedades, y para las que no hay ningún equivalente funcional a las que se pueda acceder digitalmente.

Las superficies de control físico intermedias son aquellas en las que la lógica rectora restringe la función de la superficie de control físico, de modo que se pueda iniciar una función equivalente de forma remota o que se puedan evitar conflictos de entrada remota (dichas superficies de control intermedias están vinculadas a un sistema de control local que saca provecho de la misma funcionalidad subyacente que cualquier otro sistema de mando a distancia al que el dispositivo pueda estar vinculado en paralelo). Las principales amenazas a la informática en la nube se pueden consultar en la página de [Cloud Security Alliance (CSA)](https://cloudsecurityalliance.org/articles/csa-releases-top-threats-to-cloud-computing-deep-dive/).

## <a name="additional-resources"></a>Recursos adicionales

Para más información, consulte los siguientes artículos.

* [SDL Threat Modeling Tool](https://www.microsoft.com/sdl/adopt/threatmodeling.aspx) (Herramienta de modelado de amenazas SDL)
* [Microsoft Azure IoT reference architecture available](https://azure.microsoft.com/updates/microsoft-azure-iot-reference-architecture-available/) (Arquitectura de referencia de IoT de Microsoft Azure disponible)