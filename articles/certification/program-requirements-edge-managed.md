---
title: Requisitos de certificación de Edge Managed
description: Requisitos del programa de certificación de Edge Managed
author: cbroad
ms.author: cbroad
ms.topic: conceptual
ms.date: 03/15/2021
ms.custom: Edge Managed Certification Requirements
ms.service: certification
ms.openlocfilehash: 744f81d0544a765f60793ece1aa539c6c37e39cd
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105975878"
---
# <a name="azure-certification-edge-managed"></a>Edge Managed de certificación de Azure 

En este documento se describen las funcionalidades específicas del dispositivo que se representarán en el catálogo de Azure Certified Device. Una funcionalidad es un atributo de dispositivo singular que puede describir el dispositivo. 

## <a name="program-purpose"></a>Finalidad del programa

Certificación de Edge Managed, una certificación incremental más allá de la certificación de Azure Certified Device de línea de base. Edge Managed se centra en los estándares de administración de dispositivos para los dispositivos conectados de Azure y valida la compatibilidad del entorno de ejecución de Azure IoT Edge para la implementación y administración de módulos. (Anteriormente, este programa se identificaba como el programa de certificación IoT Edge). 

La certificación de Edge Managed valida la compatibilidad del entorno de ejecución de Azure IoT Edge para la implementación y administración de módulos. Este programa proporciona confianza en la administración de dispositivos IoT conectados a Azure.

## <a name="requirements"></a>Requisitos

La certificación de Edge Managed requiere todos los requisitos del [programa de línea base de Azure Certified Device](.\program-requirements-azure-certified-device.md).

**DPS: la finalidad de la prueba es comprobar que el dispositivo implementa y admite IoT Hub Device Provisioning Service con uno de los tres métodos de atestación**

| **Nombre**                | AzureReady.DPS                                               |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ignite (en versión preliminar)                                                |
| **Se aplica a**          | Cualquier dispositivo                                      |
| **SISTEMA OPERATIVO**                  | Independiente                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | AICS valida la compatibilidad del código del dispositivo con DPS. **1.** El usuario tiene seleccionar uno de los métodos de atestación (X.509, TPM y clave SAS). **2.** En función del método de atestación, el usuario debe realizar la acción correspondiente, por ejemplo, **a)** Cargar el certificado X.509 en el ámbito de DPS administrado de AICS **b)** Implementar la clave SAS o la clave de aprobación en el dispositivo. **3.** Después, el usuario presionará el botón "Conectar" para conectarse a una instancia de IoT Hub administrada de AICS a través de DPS.                                                    |
| **Recursos**           |                                                      |
| **Recomendado para Azure:**     | N/D                                                    |

## <a name="iot-edge"></a>IoT Edge

**Existe el entorno de ejecución de Azure IoT Edge: la finalidad de la prueba es asegurarse de que el dispositivo que contiene el entorno de ejecución de Azure IoT Edge ($edgehub y $edgeagent) funciona correctamente.**

| **Nombre**                | EdgeManaged.EdgeRT                                               |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Ahora disponible                                                          |
| **Se aplica a**          | Dispositivo de IoT Edge                                                   |
| **SISTEMA OPERATIVO**                  | [Sistema operativo de nivel 1 y nivel 2](../iot-edge/support.md)                                                     |
| **Tipo de validación**     | Automatizado                                                    |
| **Validación**          | AICS valida la capacidad de implementación de RT de IoT Edge instalado. **1.** El usuario debe especificar un sistema operativo específico (el sistema operativo que no se encuentre en la lista de nivel 1 y nivel 2 no se acepta) **2.** AICS genera su archivo config.yaml e implementa el [módulo perimetral del sensor de temperatura simulado](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/azure-iot.simulated-temperature-sensor?tab=Overview) canónico **3.** AICS valida que el subsistema de contenedor compatible con Docker (Moby) está instalado en el dispositivo **4.** El resultado de la prueba se determina en función de la implementación correcta del módulo perimetral del sensor de temperatura simulado y la funcionalidad del subsistema del contenedor compatible con Docker                                                    |
| **Recursos**           | **a)** [Blog de AICS](https://azure.microsoft.com/en-in/blog/expanding-azure-iot-certification-service-to-support-azure-iot-edge-device-certification/), **b)** [Pasos de certificación](./overview.md) (tiene todos los recursos adicionales), **c)** [Requisitos](./program-requirements-azure-certified-device.md) |
| **Recomendado para Azure:**     | N/D                                                    |

### <a name="capability-template"></a>Plantilla de funcionalidad:

**Configuración sencilla de IoT Edge: la finalidad de la prueba es asegurarse de que el dispositivo de IoT Edge es fácil de configurar y valida que el entorno de ejecución de Azure IoT Edge se preinstala durante la validación del dispositivo físico**

| **Nombre**                | EdgeManaged.PhysicalDevice                                             |
| ----------------------- | ------------------------------------------------------------ |
| **Disponibilidad de destino** | Disponible ahora (actualmente en espera debido a la COVID-19)                                            |
| **Se aplica a**          | Dispositivo de IoT Edge                                                   |
| **SISTEMA OPERATIVO**                  | [Sistema operativo de nivel 1 y nivel 2](../iot-edge/support.md)                                                     |
| **Tipo de validación**     | Manual/Comprobado en el laboratorio                                                    |
| **Validación**          | El OEM debe enviar el dispositivo físico a la administración de IoT (HCL). HCL realiza la validación manual en el dispositivo físico para comprobar lo siguiente: **1.** EdgeRT usa el subsistema Moby (versión de redistribución permitida). Sin Docker **2.** Elija el módulo perimetral más reciente para validar la capacidad de implementar Edge.                                                     |
| **Recursos**           | **a)** [Blog de AICS](https://azure.microsoft.com/en-in/blog/expanding-azure-iot-certification-service-to-support-azure-iot-edge-device-certification/), **b)** [Pasos de certificación](./overview.md) , **c)** [Requisitos](./program-requirements-azure-certified-device.md) |
| **Recomendado para Azure:**     | N/D                                                    |