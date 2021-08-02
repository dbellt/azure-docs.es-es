---
title: Lista de comprobación previa a la implementación para implementar un dispositivo de Azure Stack Edge Pro con GPU | Microsoft Docs
description: En este artículo se describe la información que se puede recopilar antes de implementar un dispositivo de Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 06/07/2021
ms.author: alkohli
ms.openlocfilehash: 9bb7f552fccf4b6f58def2046b80473a57e6116f
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111756390"
---
# <a name="deployment-checklist-for-your-azure-stack-edge-pro-gpu-device"></a>Lista de comprobación para implementación de un dispositivo de Azure Stack Edge Pro con GPU  

En este artículo se describe la información que se puede recopilar antes de la implementación real del dispositivo de Azure Stack Edge Pro. 

Use la lista de comprobación siguiente para asegurarse de tener esta información después de haber hecho un pedido de un dispositivo de Azure Stack Edge Pro y antes de haberlo recibido. 

## <a name="deployment-checklist"></a>Lista de comprobación de la implementación 

| Fase                             | Parámetro                                                                                                                                                                                                                           | Detalles                                                                                                           |
|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| Administración de dispositivos               | <li>Suscripción de Azure</li><li>Proveedores de recursos registrados</li><li>Cuenta de Azure Storage</li>|<li>Se habilitad para el acceso de Azure Stack Edge Pro/Data Box Gateway, de propietario o de colaborador.</li><li>En Azure Portal, vaya a **Inicio > Suscripciones > Su suscripción > Proveedores de recursos**. Busque `Microsoft.DataBoxEdge` y regístrese. Repita el procedimiento para `Microsoft.Devices` si va a implementar cargas de trabajo de IoT.</li><li>Se necesitan credenciales de acceso</li> |
| Instalación de dispositivos               | Cables de alimentación en el paquete. <br>Para EE. UU., se envía un cable SVE 18/3 con una clasificación de 125 V y 15 A con un conector NEMA de 5-15P a C13 (de entrada a salida). | Para obtener más información, consulte[Cables de alimentación compatibles por país](azure-stack-edge-technical-specifications-power-cords-regional.md).  |
|                                   | <li>Al menos un cable de red 1 X 1-GbE RJ-45 para el puerto 1  </li><li> Al menos un cable de cobre 1 X 25/10-GbE SFP+ para el puerto 3, el puerto 4, el puerto 5 o el puerto 6</li>| El cliente debe adquirir estos cables.<br>Para una lista completa de los cables de red, enchufes y transceptores compatibles para las tarjetas de red del dispositivo de Cavium, consulte el artículo sobre la [matriz de interoperabilidad de la serie Cavium FastlinQ 41000](https://www.marvell.com/documents/xalflardzafh32cfvi0z/).<br>Para una lista completa de los módulos y cables compatibles para 25 GbE y 10 GbE de Mellanox, consulte el artículo [Productos compatibles con el adaptador de red de 4 canales, doble puerto 25G ConnectX de Mellanox](https://docs.mellanox.com/display/ConnectX4LxFirmwarev14271016/Firmware+Compatible+Products).| 
| Primera conexión del dispositivo      | <li>Portátil cuya configuración de IPv4 se puede cambiar. Este portátil se conecta al puerto 1 a través de un conmutador o un adaptador de USB a Ethernet.  </li><!--<li> A minimum of 1 GbE switch must be used for the device once the initial setup is complete. The local web UI will not be accessible if the connected switch is not at least 1 Gbe.</li>-->|   |
| Inicio de sesión en el dispositivo                      | Contraseña del administrador de dispositivos, que debe tener entre 8 y 16 caracteres e incluir tres de los tipos de caracteres siguientes: mayúsculas, minúsculas, números y caracteres especiales.                                            | La contraseña predeterminada es *Password1*, que expira al iniciar sesión por primera vez.                                                     |
| Configuración de red                  | El dispositivo incluye 2 puertos de red de 1 GbE y 4 de 25 GbE. <li>El puerto 1 se usa para configurar las opciones de administración únicamente. Se pueden conectar y configurar uno o varios puertos de datos. </li><li> Al menos una interfaz de red de datos de entre los puertos 2 a 6 debe estar conectada a Internet (para la conectividad a Azure).</li><li> Configuración de DHCP e IPv4 estática. | La configuración de IPv4 estática requiere IP, el servidor DNS y la puerta de enlace predeterminada.   |
| Configuración de la red de proceso     | <li>Requiere dos direcciones IP contiguas, estáticas y libres para los nodos Kubernetes, y una dirección IP estática para el servicio IoT Edge.</li><li>Requiere una dirección IP adicional para cada servicio o módulo adicional que se vayan a implementar.</li>| Solo se admite la configuración de IPv4 estática.|
| (Opcional) Configuración del proxy web     | <li>IP/FQDN, puerto del servidor proxy web </li><li>Nombre de usuario, contraseña del proxy web</li> |  |
| Configuración de firewall y puertos        | Si va a usar el firewall, asegúrese de que los [patrones y puertos de direcciones URL enumerados](azure-stack-edge-system-requirements.md#networking-port-requirements) se permiten en las direcciones IP de los dispositivos. |  |
| (Recomendado) Configuración de hora       | Configure la zona horaria, el servidor NTP principal, el servidor NTP secundario. | Configure el servidor NTP principal y secundario en la red local.<br>Si el servidor local no está disponible, se pueden configurar servidores NTP públicos.                                                    |
| (Opcional) Configuración del servidor de actualización | <li>Se requiere la dirección IP del servidor de actualización en la red local, ruta de acceso al servidor WSUS. </li> | De manera predeterminada, se usa el servidor público de Windows Update.|
| Configuración del dispositivo | <li>Nombre de dominio completo (FQDN) del dispositivo </li><li>Dominio DNS</li> | |
| (Opcional) Certificados  | Para probar las cargas de trabajo que no sean de producción, use la [opción Generar certificados](azure-stack-edge-gpu-deploy-configure-certificates.md#generate-device-certificates). <br><br> Si aporta sus propios certificados, incluidas las cadenas de firma, utilice [Agregar certificado](azure-stack-edge-gpu-deploy-configure-certificates.md#bring-your-own-certificates) y elija el formato adecuado.| Configure los certificados solo si cambia el nombre o el dominio DNS del dispositivo. |
| Activación  | Se requiere una clave de activación del recurso de Azure Stack Edge Pro/Data Box Gateway.    | Una vez generada, la clave expira en 3 días. |

<!--
| (Optional) MAC Address            | If MAC address needs to be on the allowed list, get the address of the connected port from local UI of the device. |                                                                                                                   |
| (Optional) Network switch port    | Device hosts Hyper-V VMs for compute. Some network switch port configurations don’t accommodate these setups by default.                                                                                                        |                                                                                                                   |-->


## <a name="next-steps"></a>Pasos siguientes

Prepare para implementar el [dispositivo de Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md).
