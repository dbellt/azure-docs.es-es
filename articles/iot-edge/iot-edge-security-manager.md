---
title: 'Administrador de seguridad de Azure IoT Edge: Azure IoT Edge'
description: Administra la posición de seguridad del dispositivo IoT Edge y la integridad de los servicios de seguridad.
services: iot-edge
keywords: seguridad, elemento seguro, enclave, TEE, IoT Edge
author: eustacea
manager: philmea
ms.author: eustacea
ms.date: 08/30/2019
ms.topic: conceptual
ms.service: iot-edge
ms.openlocfilehash: ed73c114c015ce38fefa02d2b71e27e8f85dca39
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107028679"
---
# <a name="azure-iot-edge-security-manager"></a>Administrador de seguridad de IoT Edge

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

El Administrador de seguridad de Azure IoT Edge es un núcleo de seguridad bien delimitado para proteger el dispositivo IoT Edge y todos sus componentes mediante la abstracción de hardware de silicio seguro. El responsable de seguridad es el punto focal para fortalecer la seguridad y proporciona a los fabricantes de equipos originales (OEM) el punto de integración de la tecnología.

![Administrador de seguridad de IoT Edge](media/edge-security-manager/iot-edge-security-manager.png)

El objeto del administrador de seguridad de IoT Edge es defender la integridad del dispositivo IoT Edge y todas las operaciones de software inherentes. El administrador de seguridad transiciona la confianza desde la raíz de confianza de hardware subyacente (si está disponible) para arrancar en el entorno de ejecución de IoT Edge y supervisar la integridad de sus operaciones.  El administrador de seguridad de IoT Edge es software que funciona junto con el hardware de silicio seguro (donde esté disponible) para ayudar a entregar las garantías de seguridad más altas posible.  

Las responsabilidades del administrador de seguridad de IoT Edge incluyen, entre otras:

* Arranque protegido y medido del dispositivo Azure IoT Edge.
* Aprovisionamiento de identidad del dispositivo y transición de confianza donde corresponda.
* Hospedaje y protección de los componentes del dispositivo de servicios en la nube como servicio Device Provisioning.
* Aprovisionamiento seguro de los módulos de IoT Edge con identidades automáticas.
* Equipo selector de la raíz de confianza de hardware de dispositivo a través de servicios notariales.
* Supervisión de la integridad de las operaciones de IoT Edge en el entorno de ejecución.

El administrador de seguridad de IoT Edge consta de tres componentes:

* Demonio de seguridad de IoT Edge.
* Capa de abstracción de plataforma del módulo de seguridad de hardware (HSM PAL).
* HSM o raíz de confianza de silicio de hardware opcional o altamente recomendado.

## <a name="the-iot-edge-security-daemon"></a>El demonio de seguridad de IoT Edge

El demonio de seguridad de IoT Edge es el responsable de las operaciones lógicas del administrador de seguridad de IoT Edge. Representa una porción importante de la base informática de confianza del dispositivo IoT Edge.

### <a name="design-principles"></a>Principios de diseño

El demonio de seguridad de IoT Edge sigue dos principios básicos: maximizar la integridad operativa y minimizar el sobredimensionamiento y renovación.

#### <a name="maximize-operational-integrity"></a>Maximización de la integridad de las operaciones

El demonio de seguridad de IoT Edge funciona con la mayor integridad posible dentro de la funcionalidad de defensa de cualquier hardware de raíz de confianza determinado. Con una integración correcta, el hardware de raíz de confianza mide y supervisa el demonio de seguridad de manera estática y en el entorno de ejecución para resistir manipulaciones.

El acceso físico siempre es una amenaza para los dispositivos IoT. La raíz de confianza de hardware juega un papel importante en la defensa de la integridad del demonio de seguridad de IoT Edge.  La raíz de confianza de hardware tiene dos variantes:

* elementos seguros para la protección de información confidencial, como secretos y claves criptográficas.
* enclaves seguros para la protección de secretos como claves y cargas de trabajo confidenciales, como la medición y facturación.

Existen dos tipos de entornos de ejecución para usar la raíz de confianza del hardware:

* El entorno de ejecución estándar o enriquecido (REE) que se basa en el uso de elementos seguros para proteger información confidencial.
* El entorno de ejecución de confianza (TEE) que se basa en el uso de la tecnología de enclave seguro para proteger información confidencial y ofrecer protección para la ejecución de software.

En el caso de los dispositivos que usan los enclaves seguros como raíz de confianza de hardware, la lógica confidencial dentro del demonio de seguridad de IoT Edge debe encontrarse dentro del enclave.  Las partes no confidenciales del demonio de seguridad pueden estar fuera del entorno de ejecución de confianza.  En cualquier caso, los original design manufacturers (ODM) y los fabricantes de equipos originales (OEM) deben ampliar la confianza desde su HSM para medir y defender la integridad del demonio de seguridad de IoT Edge en el arranque y en el entorno de ejecución.

#### <a name="minimize-bloat-and-churn"></a>Minimización del sobredimensionamiento y la renovación

Otro principio básico para el demonio de seguridad de IoT Edge es minimizar la renovación.  Para obtener el máximo nivel de confianza, el demonio de seguridad de IoT Edge puede acoplarse estrechamente con la raíz de confianza de hardware de dispositivo y funcionar como código nativo.  Es común que estos tipos de realizaciones actualicen el software de demonio a través de las rutas de actualización segura de la raíz de confianza de hardware (en lugar de los mecanismos de actualización que proporciona el SO), algo que puede ser complicado en determinados escenarios.  Aunque la renovación de seguridad es una recomendación para los dispositivos IoT, excesivos requisitos excesivos de actualización o cargas de actualizaciones de gran tamaño pueden expandir de muchas maneras la superficie expuesta a amenazas.  Algunos ejemplos son la omisión de las actualizaciones para maximizar la disponibilidad operativa o hardware de raíz de confianza demasiado limitado para procesar grandes cargas de actualizaciones.  Por tanto, el diseño del demonio de seguridad de IoT Edge es conciso para que la superficie y la base informática de confianza no crezcan y para minimizar los requisitos de actualizaciones.

### <a name="architecture-of-iot-edge-security-daemon"></a>Arquitectura del demonio de seguridad de IoT Edge

![Demonio de seguridad de Azure IoT Edge](media/edge-security-manager/iot-edge-security-daemon.png)

El demonio de seguridad de IoT Edge aprovecha las ventajas de cualquier tecnología de raíz de confianza de hardware disponible para fortalecer la seguridad.  También permite la operación de división global entre un entorno de ejecución estándar o enriquecido (REE) y un entorno de ejecución de confianza (TEE) cuando las tecnologías de hardware ofrecen los entornos de ejecución de confianza. Las interfaces específicas de rol habilitan los componentes importantes de IoT Edge para garantizar la integridad del dispositivo IoT Edge y sus operaciones.

#### <a name="cloud-interface"></a>Interfaz de la nube

La interfaz de la nube permite que el demonio de seguridad de IoT Edge tenga acceso a los servicios en la nube como cortesía de la nube para la seguridad de los dispositivos, como la renovación de seguridad.  Por ejemplo, el demonio de seguridad de IoT Edge actualmente usa esta interfaz para tener acceso a Azure IoT Hub [Device Provisioning Service](../iot-dps/index.yml) para la administración del ciclo de vida de la identidad.  

#### <a name="management-api"></a>API de administración

El demonio de seguridad de IoT Edge ofrece una API de administración, que el agente de IoT Edge llama en el momento de crear, iniciar, detener o eliminar un módulo de IoT Edge. El demonio de seguridad almacena "registros" para todos los módulos activos. Estos registros asignan la identidad de un módulo a alguna propiedades del módulo. Por ejemplo, estas propiedades del módulo incluyen el identificador de proceso (pid) del proceso que se ejecuta en el contenedor y el hash del contenido del contenedor de Docker.

La API de carga de trabajo (que se describe a continuación) usa estas propiedades para comprobar que el autor de llamada está autorizado para hacer una acción.

La API de administración es una API con privilegios, a la que solo se puede llamar desde el agente de IoT Edge.  Como el demonio de seguridad de IoT Edge arranca e inicia al agente de IoT Edge, comprueba que el agente de IoT Edge no se ha alterado y entonces puede crear un registro implícito para él. El mismo proceso de atestación que usa la API de carga de trabajo restringe también el acceso a la API de administración solo para el agente de IoT Edge.

#### <a name="container-api"></a>API de contenedor

La API de contenedor interactúa con el sistema de contenedor en uso para la administración de módulos, como Moby o Docker.

#### <a name="workload-api"></a>API de carga de trabajo

La API de carga de trabajo es accesible para todos los módulos. Proporciona una prueba de identidad, en forma de un certificado X509 o token firmado liberado de HSM, y la agrupación de confianza correspondiente a un módulo. La agrupación de confianza contiene certificados de entidad de certificación para todos los demás servidores en que deben confiar los módulos.

El demonio de seguridad de IoT Edge usa un proceso de atestación para proteger esta API. Cuando un módulo llama a esta API, el demonio de seguridad intenta encontrar un registro para la identidad. Si eso se completa correctamente, usa las propiedades del registro para medir el módulo. Si el resultado del proceso de medición coincide con el registro, se genera una nueva prueba de identidad. Los certificados de entidad de certificación correspondientes (agrupación de confianza) se devuelven al módulo.  El módulo usa este certificado para conectarse a IoT Hub, otros módulos, o iniciar un servidor. Cuando el certificado o un token firmado se acerca a la expiración, es responsabilidad del módulo solicitar un certificado nuevo.

### <a name="integration-and-maintenance"></a>Integración y mantenimiento

Microsoft mantiene el código base principal para el [demonio de seguridad de IoT Edge en GitHub](https://github.com/Azure/iotedge/tree/master/edgelet).

#### <a name="installation-and-updates"></a>Instalación y actualizaciones

La instalación y las actualizaciones del demonio de seguridad de IoT Edge se administra a través del sistema de administración de paquetes del sistema operativo. Los dispositivos IoT Edge con raíz de confianza de hardware deben proporcionar una protección adicional a la integridad del demonio a través de la administración de su ciclo de vida mediante los sistemas de administración de actualizaciones y el arranque seguro. Los fabricantes de dispositivos deben explorar estas vías en función de sus respectivas funcionalidades del dispositivo.

#### <a name="versioning"></a>Control de versiones

El entorno de ejecución de IoT Edge hace el seguimiento de la versión del demonio de seguridad de IoT Edge y la notifica. La versión se notifica como el atributo *runtime.platform.version* de la propiedad notificada del módulo del agente de IoT Edge.

### <a name="hardware-security-module-platform-abstraction-layer-hsm-pal"></a>Capa de abstracción de plataforma del módulo de seguridad de hardware (HSM PAL)

HSM PAL abstrae todo el hardware de raíz de confianza para aislar al desarrollador o usuario de IoT Edge de sus complejidades.  Incluye una combinación de la interfaz de programación de aplicaciones (API) y procedimientos de comunicaciones entre dominios, por ejemplo, la comunicación entre un entorno de ejecución estándar y un enclave seguro.  La implementación real de HSM PAL depende del hardware seguro específico que esté en uso. Su existencia permite el uso de prácticamente cualquier hardware de silicio seguro.

## <a name="secure-silicon-root-of-trust-hardware"></a>Hardware de raíz de confianza de silicio seguro

El silicio seguro es necesario para delimitar la confianza dentro del hardware del dispositivo IoT Edge.  El silicio seguro viene en una gran variedad de formas que incluyen Módulo de plataforma segura (TPM), Elemento seguro integrado (eSE), ARM TrustZone, Intel SGX y tecnologías de silicio seguro.  Se recomienda usar la raíz de confianza de silicio seguro en dispositivos, dadas las amenazas asociadas con la accesibilidad física de los dispositivos IoT.

## <a name="iot-edge-security-manager-integration-and-maintenance"></a>Mantenimiento e integración del administrador de seguridad de IoT Edge

El objetivo del administrador de seguridad de IoT Edge es identificar y aislar los componentes que se encargan de defender la seguridad y la integridad de la plataforma de Azure IoT Edge para el sistema de protección personalizado. Los terceros (como fabricantes de dispositivos) deben usar las características de seguridad personalizadas disponibles con su hardware de dispositivo.  

Aprenda a proteger el administrador de seguridad de Azure IoT con el Módulo de plataforma segura (TPM) mediante TPM virtuales o de software:  

Cree y aprovisione un [dispositivo IoT Edge con un TPM virtual en una máquina virtual con Linux](how-to-auto-provision-simulated-device-linux.md).

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"
Cree y aprovisione un [dispositivo IoT Edge con un TPM simulado en Windows](how-to-auto-provision-simulated-device-windows.md).
:::moniker-end

## <a name="next-steps"></a>Pasos siguientes

Lea el blog sobre [Securing the intelligent edge](https://azure.microsoft.com/blog/securing-the-intelligent-edge/) (Protección del perímetro inteligente).
