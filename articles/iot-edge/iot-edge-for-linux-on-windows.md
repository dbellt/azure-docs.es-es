---
title: Qué es Azure IoT Edge para Linux en Windows | Microsoft Docs
description: Información general sobre la ejecución de módulos de IoT Edge para Linux en dispositivos Windows 10
author: kgremban
manager: philmea
ms.reviewer: twarwick
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 01/20/2021
ms.author: kgremban
monikerRange: =iotedge-2018-06
ms.openlocfilehash: 3c7fd6c842d465dd5af5257628044666f10f2ece
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107538206"
---
# <a name="what-is-azure-iot-edge-for-linux-on-windows-preview"></a>Qué es Azure IoT Edge para Linux en Windows (versión preliminar)

[!INCLUDE [iot-edge-version-201806](../../includes/iot-edge-version-201806.md)]

Azure IoT Edge para Linux en Windows le permite ejecutar cargas de trabajo de Linux en contenedores junto con aplicaciones Windows en implementaciones de Windows IoT. Las empresas que se basan en la tecnología de Windows IoT para sus dispositivos perimetrales ahora pueden aprovechar las soluciones de análisis nativas de nube que se compilan en Linux.

IoT Edge para Linux en Windows funciona mediante la ejecución de una máquina virtual Linux en un dispositivo Windows. La máquina virtual Linux viene preinstalada con el entorno de ejecución de Azure IoT Edge. Los módulos de IoT Edge implementados en el dispositivo se ejecutan dentro de la máquina virtual. Mientras tanto, las aplicaciones de Windows que se ejecutan en el dispositivo host de Windows pueden comunicarse con los módulos que se ejecutan en la máquina virtual Linux.

[Empiece a usar](how-to-install-iot-edge-on-windows.md) la versión preliminar hoy mismo.

>[!NOTE]
>Considere la posibilidad de completar nuestra [encuesta del producto](https://aka.ms/AzEFLOW-Registration) para ayudarnos a mejorar Azure IoT Edge para Linux en Windows en función de sus objetivos y antecedentes de IoT Edge. También puede usar esta encuesta para registrarse para futuros anuncios de Azure IoT Edge para Linux en Windows.

## <a name="components"></a>Componentes

IoT Edge para Linux en Windows usa los componentes siguientes para permitir que las cargas de trabajo de Windows y Linux se ejecuten unas junto a otras y se comuniquen sin problemas:

* **Una máquina virtual Linux que ejecuta Azure IoT Edge**: Una máquina virtual Linux, basada en el sistema operativo [Mariner](https://github.com/microsoft/CBL-Mariner) propio de Microsoft se crea con el entorno de ejecución de Azure IoT Edge y se valida como un entorno admitido de nivel 1 para cargas de trabajo de IoT Edge.

* **Windows Admin Center**: Una extensión de IoT Edge para Windows Admin Center facilita la instalación, la configuración y el diagnóstico de IoT Edge en la máquina virtual Linux. Windows Admin Center puede implementar IoT Edge para Linux en Windows en el dispositivo local, o bien puede conectarse a dispositivos de destino y administrarlos de forma remota.

* **Microsoft Update**: La integración de Microsoft Update mantiene actualizados los componentes de Windows Runtime, la VM Linux con Mariner y IoT Edge.

![Windows y la VM Linux se ejecutan en paralelo, mientras que Windows Admin Center controla ambos componentes.](./media/iot-edge-for-linux-on-windows/architecture-and-communication.png)

La comunicación bidireccional entre el proceso de Windows y la máquina virtual de Linux significa que los procesos de Windows pueden proporcionar interfaces de usuario o proxies de hardware para las cargas de trabajo que se ejecutan en los contenedores de Linux.

## <a name="samples"></a>Ejemplos

IoT Edge para Linux en Windows destaca la interoperabilidad entre los componentes de Windows y Linux.

Para obtener ejemplos que muestran la comunicación entre las aplicaciones de Windows y los módulos de IoT Edge, consulte [Ejemplos de IoT para Windows 10 y EFLOW](https://aka.ms/AzEFLOW-Samples).

## <a name="public-preview"></a>Versión preliminar pública

IoT Edge para Linux en Windows se encuentra actualmente en [versión preliminar pública](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Los procesos de instalación y administración pueden ser diferentes de los de las características disponibles con carácter general.

## <a name="support"></a>Soporte técnico

Use los canales de soporte y comentarios de IoT Edge para obtener asistencia con IoT Edge para Linux en Windows.

**Informes de errores**: se pueden notificar errores en la [página de problemas](https://github.com/azure/iotedge/issues) del proyecto de código abierto de IoT Edge. Los errores relacionados con Azure IoT Edge para Linux en Windows se pueden indicar en la [Página de problemas iotedge-eflow](https://aka.ms/AzEFLOW-Issues).

**Equipo de soporte técnico al cliente de Microsoft**: los usuarios que dispongan de un [plan de soporte técnico](https://azure.microsoft.com/support/plans/) pueden ponerse en contacto con este equipo creando una incidencia de soporte técnico directamente en [Azure Portal](https://ms.portal.azure.com/signin/index/?feature.settingsportalinstance=mpac).

**Solicitudes de características**: el producto Azure IoT Edge realiza un seguimiento de las solicitudes de características a través de la [página Voz del usuario](https://feedback.azure.com/forums/907045-azure-iot-edge).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información y un ejemplo en acción, vea [IoT Edge para Linux en Windows 10 IoT Enterprise](https://aka.ms/EFLOWPPC9).

Siga los pasos descritos en [Instalación y aprovisionamiento de Azure IoT Edge para Linux en un dispositivo Windows](how-to-install-iot-edge-on-windows.md) para configurar un dispositivo con IoT Edge para Linux en Windows.
