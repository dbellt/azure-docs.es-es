---
title: Requisitos para la certificación de Edge Secured-Core
description: Requisitos para la certificación de Edge Secured-Core
author: cbroad
ms.author: cbroad
ms.topic: conceptual
ms.date: 03/15/2021
ms.custom: Edge Secured-core Certification Requirements
ms.service: certification
ms.openlocfilehash: 6a4e969c592e844164d2f031914740e24fdc8f99
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111540249"
---
# <a name="edge-secured-core-certification-requirements-preview"></a>Requisitos para la certificación de Edge Secured-Core (versión preliminar) #

En este documento se describen las capacidades y los requisitos específicos del dispositivo que se cumplirán para completar la certificación y mostrar un dispositivo en el catálogo de dispositivos de Azure IoT con la etiqueta de Edge Secured-Core.

## <a name="program-purpose"></a>Propósito del programa ##
Edge Secured-Core es una certificación incremental en el programa de Azure Certified Device para dispositivos IoT que ejecutan un sistema operativo completo, como Linux o Windows 10 IoT. Este programa permite a los asociados de dispositivos diferenciar sus dispositivos mediante la aplicación de un conjunto de criterios de seguridad adicional. Los dispositivos que cumplen estos criterios habilitan estas promesas:
1. Identidad de dispositivo basada en hardware 
2. Capacidad de aplicar la integridad del sistema 
3. Permanece actualizado y se administra de forma remota
4. Proporciona protección de los datos en reposo
5. Proporciona protección de los datos en tránsito
6. Protección y agente de seguridad integrado
## <a name="requirements"></a>Requisitos ##

---
|Nombre|SecuredCore.Built-in.Security|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es asegurarse de que los dispositivos puedan notificar la información de seguridad y los eventos mediante el envío de datos a Azure Defender para IoT.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación |El dispositivo debe generar alertas y registros de seguridad. Registros de dispositivo y mensajes de alerta para Azure Security Center.<ol><li>Descarga e implementación del agente de seguridad desde GitHub</li><li>Valide el mensaje de alerta de Azure Defender para IoT.</li></ol>|
|Recursos|[Azure Docs IoT Defender para IoT](../defender-for-iot/how-to-configure-agent-based-solution.md)|

---
|Nombre|SecuredCore.Encryption.Storage|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es validar que los datos confidenciales se puedan cifrar en el almacenamiento no volátil.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar mediante el conjunto de herramientas para asegurarse de que el cifrado de almacenamiento está habilitado y el algoritmo predeterminado es XTS-AES, con una longitud de clave de 128 bits o superior.|
|Recursos||

---
|Nombre|SecuredCore.Hardware.SecureEnclave|
|:---|:---|
|Estado|Opcional|
|Descripción|El propósito de la prueba es validar la existencia de un enclave seguro y que el enclave sea accesible desde un agente seguro.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar mediante el conjunto de herramientas para asegurarse de que el agente de seguridad de Azure puede comunicarse con el enclave seguro|
|Recursos|https://github.com/openenclave/openenclave/blob/master/samples/BuildSamplesLinux.md|

---
|Nombre|SecuredCore.Hardware.Identity|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es validar que la identificación del dispositivo está arraigada en el hardware.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar mediante el conjunto de herramientas para asegurarse de que el dispositivo tiene un TPM presente y que se puede aprovisionar a través de IoT Hub mediante la clave de aprobación de TPM.|
|Recursos|[Configurar aprovisionamiento automático con DPS](../iot-dps/quick-setup-auto-provision.md)|

---
|Nombre|SecuredCore.Update|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es validar que el dispositivo puede recibir y actualizar el firmware y el software.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Confirmación del asociado que pudo enviar una actualización al dispositivo a través de la actualización de Microsoft, la actualización del dispositivo de Azure u otros servicios aprobados.|
|Recursos|[Device Update para IoT Hub](../iot-hub-device-update/index.yml)|

---
|Nombre|SecuredCore.Manageability.Configuration|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es validar que los dispositivos admiten la administración de la seguridad remota.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar mediante el conjunto de herramientas para asegurarse de que el dispositivo admite la capacidad de administrar de forma remota y concretamente configuraciones de seguridad. Y el estado se devuelve a IoT Hub/Azure Defender para IoT.|
|Recursos||

---
|Nombre|SecuredCore.Manageability.Reset|
|:---|:---|
|Estado|Obligatorio|
|Descripción|La finalidad de esta prueba es validar el dispositivo con dos casos de uso: a) para realizar un restablecimiento (quitar los datos de usuario, quitar las configuraciones de usuario), b) restaurar el dispositivo a la última buena conocida en caso de que se produzcan problemas.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar a través de una combinación de conjunto de herramientas y documentación enviada acerca de que el dispositivo admite esta funcionalidad. El fabricante del dispositivo puede determinar si debe implementar estas funcionalidades para admitir el restablecimiento remoto o solo el restablecimiento local.|
|Recursos||

---
|Nombre|SecuredCore.Updates.Duration|
|:---|:---|
|Estado|Obligatorio|
|Descripción|La finalidad de esta directiva es asegurarse de que el dispositivo permanezca seguro.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual|
|Validación|Se requerirá el compromiso del envío de los dispositivos certificados para mantener actualizados los dispositivos durante 60 meses a partir de la fecha de envío. Las especificaciones disponibles para el comprador y los propios dispositivos de alguna manera deben indicar la duración de la actualización del software.|
|Recursos||

---
|Nombre|SecuredCore.Policy.Vuln.Disclosure|
|:---|:---|
|Estado|Obligatorio|
|Descripción|La finalidad de esta directiva es asegurarse de que existe un mecanismo para recopilar y distribuir informes de vulnerabilidades en el producto.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual|
|Validación|Se revisará la documentación sobre el proceso de envío y recepción de informes de vulnerabilidad para los dispositivos certificados.|
|Recursos||

---
|Nombre|SecuredCore.Policy.Vuln.Fixes|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de esta directiva es asegurarse de que las vulnerabilidades que son altas o críticas (mediante CVSS 3.0) se abordan en un plazo de 180 días a partir de la corrección disponible.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual|
|Validación|Se revisará la documentación sobre el proceso de envío y recepción de informes de vulnerabilidad para los dispositivos certificados.|
|Recursos||


---
|Nombre|SecuredCore.Encryption.TLS|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es validar la compatibilidad con las versiones de TLS y los conjuntos de cifrado necesarios.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
Validación|Dispositivo que se va a validar mediante el conjunto de herramientas para asegurarse de que el dispositivo admite una versión de TLS mínima de 1.2 y admite los siguientes conjuntos de cifrado TLS necesarios.<ul><li>TLS_RSA_WITH_AES_128_GCM_SHA256</li><li>TLS_RSA_WITH_AES_128_CBC_SHA256</li><li>TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256</li><li>TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256</li><li>TLS_DHE_RSA_WITH_AES_128_GCM_SHA256</li><li>TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256</li><li>TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256</li></ul>|
|Recursos| [Compatibilidad con TLS en IoT Hub](../iot-hub/iot-hub-tls-support.md) <br /> [Conjuntos de cifrado TLS en Windows 10](/windows/win32/secauthn/tls-cipher-suites-in-windows-10-v1903) |

---
|Nombre|SecuredCore.Protection.SignedUpdates|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es validar que las actualizaciones se deben firmar.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar a través del conjunto de herramientas para asegurarse de que no se aplicarán las actualizaciones del sistema operativo, los controladores, el software de aplicación, las bibliotecas, los paquetes y el firmware, a menos que se firmen y se validen correctamente.
|Recursos||

---
|Nombre|SecuredCore.Firmware.SecureBoot|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es validar la integridad del arranque del dispositivo.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar mediante el conjunto de herramientas para asegurarse de que las firmas de firmware y kernel se validan cada vez que se inicia el dispositivo. <ul><li>UEFI: el arranque seguro está habilitado</li><li>Uboot: el arranque verificado está habilitado</li></ul>|
|Recursos||

---
|Nombre|SecuredCore.Protection.CodeIntegrity|
|:---|:---|
|Estado|Obligatorio|
|Descripción|La finalidad de esta prueba es validar que la integridad del código está disponible en este dispositivo.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar mediante el conjunto de herramientas para asegurarse de que está habilitada la integridad del código. </br> Windows: HVCI </br> Linux: dm-verity y IMA|
|Recursos||

---
|Nombre|SecuredCore.Protection.NetworkServices|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es validar que las aplicaciones que aceptan la entrada desde la red no se ejecutan con privilegios elevados.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar mediante el conjunto de herramientas para asegurarse de que los servicios que aceptan conexiones de red no se ejecutan con privilegios de SISTEMA o raíz.|
|Recursos||

---
|Nombre|SecuredCore.Protection.Baselines|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es validar que el sistema cumple con una configuración de seguridad de línea de base.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar a través del conjunto de herramientas para asegurarse de que se han ejecutado los puntos de referencia de las configuraciones del sistema IOT de Defender.|
|Recursos| https://techcommunity.microsoft.com/t5/microsoft-security-baselines/bg-p/Microsoft-Security-Baselines <br> https://www.cisecurity.org/cis-benchmarks/ |

---
|Nombre|SecuredCore.Firmware.Protection|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es asegurarse de que el dispositivo tiene mitigaciones adecuadas de las amenazas de seguridad de firmware.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar mediante el conjunto de herramientas para confirmar que está protegido contra amenazas de seguridad de firmware mediante uno de los siguientes métodos: <ul><li>DRTM + mitigaciones del modo de administración de UEFI</li><li>DRTM + protección del modo de administración de UEFI</li><li>FW aprobado que hace SRTM + protección de firmware en tiempo de ejecución</li></ul> |
|Recursos| https://trustedcomputinggroup.org/ |

---
|Nombre|SecuredCore.Firmware.Attestation|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es asegurarse de que el dispositivo pueda confirmar de forma remota el servicio de Microsoft Azure Attestation.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar mediante el conjunto de herramientas para asegurarse de que los registros de arranque de la plataforma y las medidas de la actividad de arranque se pueden recopilar y confirmar de forma remota en el servicio de Microsoft Azure Attestation.|
|Recursos| [Microsoft Azure Attestation](../attestation/index.yml) |

---
|Nombre|SecuredCore.Hardware.MemoryProtection|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es validar que DMA no está habilitado en puertos accesibles externamente.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Si existen puertos externos compatibles con DMA en el dispositivo, conjunto de herramientas para validar que el IOMMU o SMMU está habilitado y configurado para esos puertos.|
|Recursos||

---
|Nombre|SecuredCore.Protection.Debug|
|:---|:---|
|Estado|Obligatorio|
|Descripción|El propósito de la prueba es validar que la funcionalidad de depuración en el dispositivo está deshabilitada.|
|Disponibilidad de destino|2021|
|Se aplica a|Cualquier dispositivo|
|SO|Independiente|
|Tipo de validación|Manual/herramientas|
|Validación|Dispositivo que se va a validar mediante el conjunto de herramientas para asegurarse de que la funcionalidad de depuración requiere autorización para habilitar.|
|Recursos||