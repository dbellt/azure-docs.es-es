---
title: 'Sistemas operativos y motores de contenedor compatibles: Azure IoT Edge'
description: Obtenga información sobre qué sistemas operativos pueden ejecutar el demonio y el entorno de ejecución de Azure IoT Edge y los motores de contenedor admitidos para los dispositivos de producción
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 06/09/2021
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: e2701a04c1e8ea631bdbf37470608cf66b8d242e
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111985371"
---
# <a name="azure-iot-edge-supported-systems"></a>Sistemas compatibles con Azure IoT Edge

[!INCLUDE [iot-edge-version-201806-or-202011](../../includes/iot-edge-version-201806-or-202011.md)]

En este artículo se proporcionan detalles sobre qué sistemas y componentes son compatibles con IoT Edge, ya sea oficialmente o en versión preliminar.

## <a name="get-support"></a>Obtención de soporte técnico

Si tiene problemas al usar el servicio Azure IoT Edge, hay varias maneras de buscar soporte técnico. Pruebe uno de los siguientes canales para obtener soporte técnico:

**Notificación de errores**: la mayor parte del desarrollo que lleva el producto de Azure IoT Edge se produce en el proyecto de código abierto de IoT Edge. Los errores se pueden notificar en la [página de problemas](https://github.com/azure/iotedge/issues) del proyecto. Los errores relacionados con Azure IoT Edge para Linux en Windows se pueden indicar en la [Página de problemas iotedge-eflow](https://github.com/azure/iotedge-eflow/issues). Las correcciones pasan pronto de los proyectos a las actualizaciones de producto.

**Equipo de soporte técnico al cliente de Microsoft**: los usuarios que dispongan de un [plan de soporte técnico](https://azure.microsoft.com/support/plans/) pueden ponerse en contacto con este equipo creando una incidencia de soporte técnico directamente en [Azure Portal](https://ms.portal.azure.com/signin/index/?feature.settingsportalinstance=mpac).

**Solicitudes de características**: el producto Azure IoT Edge realiza un seguimiento de las solicitudes de características a través de la [página Voz del usuario](https://feedback.azure.com/forums/907045-azure-iot-edge).

## <a name="container-engines"></a>Motores de contenedor

Los módulos de Azure IoT Edge se implementan como contenedores, por lo que IoT Edge necesita un motor de contenedores para iniciarlos. Microsoft ofrece un motor de contenedor, moby-engine, para satisfacer este requisito. Este motor de contenedores se basa en el proyecto de código abierto de Moby. Docker CE y Docker EE son otros motores de contenedores conocidos. También se basan en el proyecto de código abierto de Moby y son compatibles con Azure IoT Edge. Microsoft proporciona el mejor soporte técnico posible para los sistemas que usan esos motores de contenedores; sin embargo, Microsoft no puede proporcionar soluciones para los problemas de ellos. Por esta razón, Microsoft recomienda el uso de moby-engine en sistemas de producción.

<br>
<center>

![Motor de Moby como entorno de ejecución de contenedor](./media/support/only-moby-for-production.png)
</center>

## <a name="operating-systems"></a>Sistemas operativos

Azure IoT Edge se ejecuta en la mayoría de los sistemas operativos que pueden ejecutar contenedores. No obstante, la compatibilidad no es igual en todos. Los sistemas operativos se agrupan en niveles que representan el nivel de compatibilidad que los usuarios pueden esperar.

* Se admiten los sistemas de nivel 1. Para los sistemas de nivel 1, Microsoft:
  * tiene este sistema operativo en pruebas automatizadas
  * proporciona paquetes de instalación para ellos
* Los sistemas de nivel 2 son compatibles con Azure IoT Edge y pueden usarse con relativa facilidad. Para los sistemas de nivel 2:
  * Microsoft ha realizado pruebas informales en las plataformas o conoce a un asociado que ha ejecutado correctamente Azure IoT Edge en la plataforma
  * Los paquetes de información de otras plataformas pueden funcionar en estas plataformas

La familia del sistema operativo host siempre debe coincidir con la familia del sistema operativo invitado que se utilice en el contenedor de un módulo.

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"
En otras palabras, solo se pueden usar contenedores de Linux en Linux y contenedores de Windows en Windows. En el caso de contenedores Windows, solo se admiten contenedores aislados de los procesos; no se admiten los contenedores aislados de Hyper-V.  

IoT Edge para Linux en Windows usa IoT Edge en una máquina virtual Linux que se ejecuta en un host Windows. De esta manera, puede ejecutar módulos de Linux en un dispositivo Windows.
:::moniker-end
<!-- end 1.1 -->

### <a name="tier-1"></a>Nivel 1

Los sistemas que se enumeran en las tablas siguientes son compatibles con Microsoft, tanto si están disponibles con carácter general como en versión preliminar pública, y se prueban con cada versión nueva.

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"
Azure IoT Edge admite módulos creados como contenedores de Linux o Windows. Los contenedores de Linux pueden implementarse en dispositivos Linux o implementarse en dispositivos Windows mediante IoT Edge para Linux en Windows. Los contenedores de Windows solo se pueden implementar en dispositivos Windows.
:::moniker-end
<!-- end 1.1 -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"
Azure IoT Edge versión 1.2 solo admite módulos creados como contenedores de Linux.

Actualmente, no hay una manera compatible de ejecutar IoT Edge versión 1.2 en dispositivos Windows. [IoT Edge para Linux en Windows](iot-edge-for-linux-on-windows.md) es el método recomendado para ejecutar IoT Edge en dispositivos Windows, pero actualmente solo ejecuta IoT Edge 1.1. Para más información, consulte la versión [1.1 de IoT Edge](?view=iotedge-2018-06&preserve-view=true) de este artículo.

:::moniker-end
<!-- end 1.2 -->

#### <a name="linux-containers"></a>Contenedores de Linux

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"
Los módulos creados como contenedores de Linux se pueden implementar en dispositivos Linux o Windows. En el caso de los dispositivos Linux, el entorno de ejecución de Azure IoT Edge se instala directamente en el dispositivo host. En el caso de los dispositivos Windows, una máquina virtual Linux pregenerada con el entorno de ejecución de Azure IoT Edge se ejecuta en el dispositivo host.

[IoT Edge para Linux en Windows](iot-edge-for-linux-on-windows.md) se encuentra actualmente en versión preliminar pública, pero es el método recomendado para ejecutar IoT Edge en dispositivos Windows.

| Sistema operativo | AMD64 | ARM32v7 | ARM64 |
| ---------------- | ----- | ------- | ----- |
| Raspberry Pi OS Stretch |  | ![Raspberry Pi OS Stretch + ARM32v7](./media/tutorial-c-module/green-check.png) |  |
| Ubuntu Server 18.04 | ![Ubuntu Server 18.04 + AMD64](./media/tutorial-c-module/green-check.png) |  | Versión preliminar pública |
| Windows 10 Pro | Versión preliminar pública |  |  |
| Windows 10 Enterprise | Versión preliminar pública |  |  |
| Windows 10 IoT Enterprise | Versión preliminar pública |  |  |
| Windows Server 2019 | Versión preliminar pública |  |  |

Todos los sistemas operativos Windows deben estar en la versión 1809 (compilación 17763) o posterior.
:::moniker-end
<!-- end 1.1 -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

| Sistema operativo | AMD64 | ARM32v7 | ARM64 |
| ---------------- | ----- | ------- | ----- |
| Raspberry Pi OS Stretch |  | ![Raspberry Pi OS Stretch + ARM32v7](./media/tutorial-c-module/green-check.png) |  |
| Ubuntu Server 18.04 | ![Ubuntu Server 18.04 + AMD64](./media/tutorial-c-module/green-check.png) |  | Versión preliminar pública |

:::moniker-end
<!-- end 1.2 -->

>[!NOTE]
>La compatibilidad con Ubuntu Server 16.04 finalizó con el lanzamiento de la versión 1.1 de IoT Edge.

#### <a name="windows-containers"></a>Contenedores de Windows

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"
>[!IMPORTANT]
>IoT Edge 1.1 LTS es el último canal de versión que será compatible con los contenedores de Windows. A partir de la versión 1.2, no se admitirán los contenedores de Windows. Considere la posibilidad de usar o pasarse a [IoT Edge para Linux en Windows](iot-edge-for-linux-on-windows.md) para ejecutar IoT Edge en dispositivos Windows.

Los módulos creados como contenedores de Windows solo se pueden implementar en dispositivos Windows.

| Sistema operativo | AMD64 | ARM32v7 | ARM64 |
| ---------------- | ----- | ------- | ----- |
| Windows 10 IoT Enterprise | ![check1](./media/tutorial-c-module/green-check.png) |  |  |
| Windows Server 2019  | ![check1](./media/tutorial-c-module/green-check.png) |  |  |
| Windows Server IoT 2019 | ![check1](./media/tutorial-c-module/green-check.png) |  |  |

Todos los sistemas operativos Windows deben estar en la versión 1809 (compilación 17763). La compilación específica de Windows es necesaria para IoT Edge en Windows porque la versión de los contenedores de Windows debe coincidir exactamente con la versión del dispositivo Windows del host. Actualmente, los contenedores de Windows solo usan la compilación 17763.

>[!NOTE]
>La compatibilidad con Windows 10 IoT Core finalizó con la versión 1.1 de IoT Edge.
:::moniker-end
<!-- end 1.1 -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"
IoT Edge 1.1 LTS es el último canal de versión compatible con los contenedores de Windows. A partir de la versión 1.2, no se admiten los contenedores de Windows.

Para información sobre los sistemas operativos compatibles con contenedores de Windows, consulte la versión [1.1 de IoT Edge](?view=iotedge-2018-06&preserve-view=true) de este artículo.

:::moniker-end
<!-- end 1.2 -->

### <a name="tier-2"></a>Nivel 2

Los sistemas que se enumeran en la tabla siguiente se consideran compatibles con Azure IoT Edge, pero Microsoft no los prueba ni mantiene activamente.

| Sistema operativo | AMD64 | ARM32v7 | ARM64 |
| ---------------- | ----- | ------- | ----- |
| [CentOS-7](https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7) | ![CentOS + AMD64](./media/tutorial-c-module/green-check.png) | ![CentOS + ARM32v7](./media/tutorial-c-module/green-check.png) | ![CentOS + ARM64](./media/tutorial-c-module/green-check.png) |
| [Ubuntu 20.04 <sup>1</sup>](https://wiki.ubuntu.com/FocalFossa/ReleaseNotes) | ![Ubuntu 20.04 + AMD64](./media/tutorial-c-module/green-check.png) | ![Ubuntu 20.04 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Ubuntu 20.04 + ARM64](./media/tutorial-c-module/green-check.png) |
| [Debian 9](https://www.debian.org/releases/stretch/) | ![Debian 9 + AMD64](./media/tutorial-c-module/green-check.png) | ![Debian 9 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Debian 9 + ARM64](./media/tutorial-c-module/green-check.png) |
| [Debian 10](https://www.debian.org/releases/buster/) | ![Debian 10 + AMD64](./media/tutorial-c-module/green-check.png) | ![Debian 10 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Debian 10 + ARM64](./media/tutorial-c-module/green-check.png) |
| [Mentor Embedded Linux Flex OS](https://www.mentor.com/embedded-software/linux/mel-flex-os/) | ![Mentor Embedded Linux Flex OS + AMD64](./media/tutorial-c-module/green-check.png) | ![Mentor Embedded Linux Flex OS + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Mentor Embedded Linux Flex OS + ARM64](./media/tutorial-c-module/green-check.png) |
| [Mentor Embedded Linux Omni OS](https://www.mentor.com/embedded-software/linux/mel-omni-os/) | ![Mentor Embedded Linux Omni OS + AMD64](./media/tutorial-c-module/green-check.png) |  | ![Mentor Embedded Linux Omni OS + ARM64](./media/tutorial-c-module/green-check.png) |
| [RHEL 7](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7) | ![RHEL 7 + AMD64](./media/tutorial-c-module/green-check.png) | ![RHEL 7 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![RHEL 7 + ARM64](./media/tutorial-c-module/green-check.png) |
| [Ubuntu 18.04](https://wiki.ubuntu.com/BionicBeaver/ReleaseNotes) | ![Ubuntu 18.04 + AMD64](./media/tutorial-c-module/green-check.png) | ![Ubuntu 18.04 + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Ubuntu 18.04 + ARM64](./media/tutorial-c-module/green-check.png) |
| [Wind River 8](https://docs.windriver.com/category/os-wind_river_linux) | ![Wind River 8 + AMD64](./media/tutorial-c-module/green-check.png) |  |  |
| [Yocto](https://www.yoctoproject.org/) | ![Yocto + AMD64](./media/tutorial-c-module/green-check.png) | ![Yocto + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Yocto + ARM64](./media/tutorial-c-module/green-check.png) |
| Raspberry Pi OS Buster |  | ![Raspberry Pi OS Buster + ARM32v7](./media/tutorial-c-module/green-check.png) | ![Raspberry Pi OS Buster + ARM64](./media/tutorial-c-module/green-check.png) |

<sup>1</sup> Los pasos de instalación de Ubuntu Server 18.04 que se describen en [Instalación o desinstalación de Azure IoT Edge para Linux](how-to-install-iot-edge.md) deberían funcionar sin que sea necesario realizar ningún cambio en Ubuntu 20.04.

## <a name="releases"></a>Lanzamientos

Los recursos de versión y las notas de la versión de IoT Edge están disponibles en la página de [versiones de azure-iotedge](https://github.com/Azure/azure-iotedge/releases). En esta sección se refleja la información de las notas de la versión para ayudarle a visualizar los componentes de cada versión más fácilmente.

En la tabla siguiente se enumeran los componentes incluidos en cada versión, empezando por la versión 1.2.0. Los componentes enumerados en esta tabla se pueden instalar o actualizar individualmente y son compatibles con versiones anteriores.

| Release | aziot-edge | edgeHub<br>edgeAgent | aziot-identity-service |
| ------- | ---------- | -------------------- | ---------------------- |
| **1.2** | 1.2.0<br>1.2.1 | 1.2.0<br>1.2.1   | 1.2.0<br><br> |

En la tabla siguiente se enumeran los componentes incluidos en cada versión hasta la versión 1.1 LTS. Los componentes enumerados en esta tabla se pueden instalar o actualizar individualmente y son compatibles con versiones anteriores.

| Release | iotedge | edgeHub<br>edgeAgent | libiothsm | moby |
|--|--|--|--|--|
| **1.1 LTS**<sup>1</sup> | 1.1.0<br>1.1.1<br><br>1.1.3 | 1.1.0<br>1.1.1<br>1.1.2<br>1.1.3 | 1.1.0<br>1.1.1<br><br>1.1.3 |   |
| **1.0.10** | 1.0.10<br>1.0.10.1<br>1.0.10.2<br><br>1.0.10.4 | 1.0.10<br>1.0.10.1<br>1.0.10.2<br>1.0.10.3<br>1.0.10.4 | 1.0.10<br>1.0.10.1<br>1.0.10.2<br><br>1.0.10.4 |  |
| **1.0.9** | 1.0.9<br>1.0.9.1<br>1.0.9.2<br>1.0.9.3<br>1.0.9.4<br>1.0.9.5 | 1.0.9<br>1.0.9.1<br>1.0.9.2<br>1.0.9.3<br>1.0.9.4<br>1.0.9.5 | 1.0.9<br>1.0.9.1<br>1.0.9.2<br>1.0.9.3<br>1.0.9.4<br>1.0.9.5 |  |
| **1.0.8** | 1.0.8 | 1.0.8<br>1.0.8.1<br>1.0.8.2<br>1.0.8.3<br>1.0.8.4<br>1.0.8.5 | 1.0.8 | 3.0.6 |
| **1.0.7** | 1.0.7<br>1.0.7.1 | 1.0.7<br>1.0.7.1 | 1.0.7<br>1.0.7.1 | 3.0.4 (ARMv7hl, CentOS)<br>3.0.5 |
| **1.0.6** | 1.0.6<br>1.0.6.1 | 1.0.6<br>1.0.6.1 | 1.0.6<br>1.0.6.1 |  |
| **1.0.5** | 1.0.5 | 1.0.5 | 1.0.5 | 3.0.2 |

<sup>1</sup>IoT Edge 1.1 es el primer canal de versión con soporte técnico a largo plazo (LTS). Esta versión no presentó nuevas características, pero recibirá actualizaciones de seguridad y correcciones de las regresiones. IoT Edge 1.1 LTS usa .NET Core 3.1 y recibirá soporte técnico hasta el 3 de diciembre de 2022 para que coincida con el [ciclo de vida de versión de .NET Core y .NET 5](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).

>[!IMPORTANT]
>Con el lanzamiento de un canal de soporte técnico a largo plazo, se recomienda que todos los clientes actuales que ejecuten la versión 1.0.x actualicen sus dispositivos a 1.1.x para recibir soporte técnico continuado.

IoT Edge usa el SDK de Microsoft.Azure.Devices.Client. Para más información, consulte el [repositorio de GitHub de SDK de para Azure IoT en C#](https://github.com/Azure/azure-iot-sdk-csharp) o el [contenido de referencia del SDK de Azure para .NET](/dotnet/api/overview/azure/iot/client). En la siguiente lista se muestra la versión del SDK de cliente con el que se prueba cada versión:

| Versión de IoT Edge | Versión del SDK de Microsoft.Azure.Devices.Client |
|------------------|--------------------------------------------|
| 1.2.0            | 1.33.4-NestedEdge
| 1.1 (LTS)        | 1.28.0                                     |
| 1.0.10           | 1.28.0                                     |
| 1.0.9            | 1.21.1                                     |
| 1.0.8            | 1.20.3                                     |
| 1.0.7            | 1.20.1                                     |
| 1.0.6            | 1.17.1                                     |
| 1.0.5            | 1.17.1                                     |

## <a name="virtual-machines"></a>Virtual Machines

Azure IoT Edge puede ejecutarse en máquinas virtuales. El uso de una máquina virtual como dispositivo de IoT Edge es una práctica habitual cuando los clientes desean mejorar la infraestructura existente con inteligencia perimetral. La familia del sistema operativo de la máquina virtual host debe coincidir con la familia del sistema operativo invitado que se utiliza en el contenedor de un módulo. Este requisito también se aplica cuando Azure IoT Edge se ejecuta directamente en un dispositivo. Azure IoT Edge es independiente de la tecnología de virtualización subyacente y funciona en máquinas virtuales que incorporan plataformas como Hyper-V y vSphere.

<br>

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

<center>

![Azure IoT Edge en una máquina virtual](./media/support/edge-on-vm-with-windows.png)

</center>

::: moniker-end

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

<center>

![Azure IoT Edge en una máquina virtual](./media/support/edge-on-vm.png)

</center>

:::moniker-end

## <a name="minimum-system-requirements"></a>Requisitos mínimos del sistema

Azure IoT Edge funciona perfectamente en dispositivos tan pequeños como Raspberry Pi3 y en hardware de nivel de servidor. La elección del hardware adecuado para el escenario dependerá de las cargas de trabajo que se deseen ejecutar. Tomar la decisión final sobre el dispositivo que se va a utilizar puede resultar complicado; sin embargo, puede empezar creando un prototipo de una solución en portátiles o equipos de escritorio tradicionales.

La experiencia que adquiera mientras crea ese prototipo le ayudará a realizar la selección final del dispositivo. Debe considerar las siguientes cuestiones:

* ¿Cuántos módulos hay en la carga de trabajo?
* ¿Cuántas capas comparten los contenedores de los módulos?
* ¿En qué idioma se escriben los módulos?
* ¿Cuántos datos procesarán los módulos?
* ¿Necesitan los módulos algún hardware especializado para acelerar sus cargas de trabajo?
* ¿Cuáles son las características del rendimiento deseado de la solución?
* ¿Cuál es su presupuesto para hardware?
