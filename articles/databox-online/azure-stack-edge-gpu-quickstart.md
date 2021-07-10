---
title: Inicio rápido para configurar e implementar una GPU de Azure Stack Edge | Microsoft Docs
description: Comience la implementación de la GPU de Azure Stack Edge en cuanto reciba el dispositivo.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: quickstart
ms.date: 06/09/2021
ms.author: alkohli
ms.openlocfilehash: 0dd8b103f81ea911a079aee6590287a939819b52
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111982383"
---
# <a name="quickstart-get-started-with-azure-stack-edge-pro-with-gpu"></a>Inicio rápido: Introducción a Azure Stack Edge Pro con GPU 

En este inicio rápido se detallan los requisitos previos y los pasos necesarios para implementar un dispositivo GPU de Azure Stack Edge Pro. Los pasos del inicio rápido se realizan en Azure Portal y en la interfaz de usuario web local del dispositivo. 

El procedimiento total debe tardar aproximadamente 1,5 horas en completarse. Para obtener instrucciones detalladas paso a paso, consulte [Tutorial: Preparación de la implementación de Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-deploy-prep.md#deployment-configuration-checklist). 


## <a name="prerequisites"></a>Requisitos previos

Antes de realizar la implementación, asegúrese de que se cumplen los siguientes requisitos previos:

1. El dispositivo GPU de Azure Stack Edge Pro se entrega a domicilio [desempaquetado](azure-stack-edge-gpu-deploy-install.md#unpack-the-device) y [montado en un bastidor](azure-stack-edge-gpu-deploy-install.md#rack-the-device). 
1. Configure la red de forma que el dispositivo pueda acceder a los [patrones de direcciones URL y puertos de la lista](azure-stack-edge-gpu-system-requirements.md#networking-port-requirements). 
1. Debe tener acceso de colaborador o propietario a la suscripción de Azure.
1. En **Azure Portal, vaya a Inicio > Suscripciones > Su suscripción> Proveedores de recursos**. Busque `Microsoft.DataBoxEdge` y registre el proveedor de recursos. Repita este procedimiento para registrar `Microsoft.Devices` si va a crear un recurso de IoT Hub para implementar cargas de trabajo de proceso.
1. Asegúrese de que tiene un mínimo de dos direcciones IP gratis, estáticas y contiguas para los nodos de Kubernetes y al menos una dirección IP estática para el servicio IoT Edge. Para cada módulo o servicio externo que implemente, necesitará una IP más.
1. Consulte la [lista de comprobación para la implementación](azure-stack-edge-gpu-deploy-checklist.md) para obtener todo lo necesario para la configuración del dispositivo. 


## <a name="deployment-steps"></a>Pasos de implementación

1. **Instalar**: conecte PUERTO 1 a un equipo cliente a través de un cable cruzado o un adaptador Ethernet USB. Conecte al menos otro puerto de dispositivo para los datos, preferiblemente de 25 GbE, (del PUERTO 3 al PUERTO 6) a Internet mediante cables de cobre SFP+ o utilice el PUERTO 2 con un latiguillo RJ45. Conecte los cables de alimentación proporcionados a las unidades de fuente de alimentación y separe las salidas de distribución de la alimentación. Presione el botón de encendido del panel delantero encender el dispositivo.  

    Consulte el documento sobre la [matriz de interoperabilidad de la serie FastlinQ 41000 de Cavium](https://www.marvell.com/documents/xalflardzafh32cfvi0z/) y los [productos compatibles con el adaptador de red de 4 canales y doble puerto 25G ConnectX de Mellanox](https://docs.mellanox.com/display/ConnectX4LxFirmwarev14271016/Firmware+Compatible+Products) para obtener cables de red y conmutadores compatibles.

    Esta es la configuración de cableado mínima necesaria para implementar el dispositivo:  ![Backplane de un dispositivo cableado](./media/azure-stack-edge-gpu-quickstart/backplane-min-cabling-1.png)

2. **Conectar**: configure los valores de IPv4 del adaptador Ethernet en el equipo con la dirección IP estática **192.168.100.5** y la subred **255.255.255.0**. Abra el explorador y conéctese a la interfaz de usuario web local del dispositivo, en https://192.168.100.10. Esta operación puede tardar unos minutos. Vaya a la página web cuando vea la advertencia sobre el certificado de seguridad.

3. **Iniciar sesión**: inicie sesión en el dispositivo con la contraseña predeterminada, *Password1*. Cambie la contraseña del administrador de dispositivos. Esta debe contener entre 8 y 16 caracteres, y tres de ellos deben ser mayúsculas, minúsculas, números y caracteres especiales.

4. **Configurar la red**: acepte la configuración predeterminada de DHCP para el puerto de datos conectado si tiene un servidor DHCP en la red. Si no lo tiene, especifique una dirección IP, un servidor DNS y una puerta de enlace predeterminada. Vea más información sobre la [configuración de red](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md#configure-network).

5. **Configurar una red de procesos**: cree un conmutador virtual habilitando un puerto en el dispositivo. Escriba 2 direcciones IP gratis que sean estáticas y contiguas para los nodos de Kubernetes que estén en la misma red en que creó el conmutador. Especifique al menos una IP estática para el servicio IoT Edge Hub para acceder a los módulos de proceso y una dirección IP estática para cada servicio o contenedor adicionales a los que desee acceder desde fuera del clúster de Kubernetes. 

    Kubernetes es necesario para implementar todas las cargas de trabajo que estén en contenedores. Vea más información sobre la [configuración de una red de procesos](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md#enable-compute-network).

6. **Configurar un proxy web**: si usa un proxy web en su entorno, escriba la dirección IP del servidor proxy web en `http://<web-proxy-server-FQDN>:<port-id>`. Establezca la autenticación en **No**. Encontrará más información en el apartado sobre la [configuración de un proxy web](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md#configure-web-proxy).

7. **Configurar un dispositivo**: escriba un nombre de dispositivo y un dominio DNS, o acepte los valores predeterminados. 

8. **Configurar un servidor de actualizaciones**: Acepte el servidor de Microsoft Update predeterminado o especifique un servidor Windows Server Update Services (WSUS) y la ruta de acceso al mismo. 

9. **Configurar valores de tiempo**: acepte la configuración de tiempo predeterminada o establezca la zona horaria, el servidor NTP principal y el servidor NTP secundario en la red local o como servidores públicos.

10. **Configurar certificados**: si ha cambiado el nombre del dispositivo o el dominio DNS, debe generar certificados o agregar certificados para activar el dispositivo. 

    - Para probar las cargas de trabajo que no sean de producción, use la [opción Generar certificados](azure-stack-edge-gpu-deploy-configure-certificates.md#generate-device-certificates). 
    - Si aporta sus propios certificados, incluidas las cadenas de firma, utilice [Agregar certificado](azure-stack-edge-gpu-deploy-configure-certificates.md#bring-your-own-certificates) y elija el formato adecuado. Asegúrese de cargar primero la cadena de firma. Consulte los artículos sobre [creación de certificados](azure-stack-edge-gpu-create-certificates-tool.md) y de [carga de certificados mediante la interfaz de usuario local](azure-stack-edge-gpu-deploy-configure-certificates.md#bring-your-own-certificates).

11. **Activar**: para obtener la clave de activación 

    1. En Azure Portal, vaya al **recurso de Azure Stack Edge > Información general > Instalación del dispositivo > Activar > Generar clave**. Copie la clave. 
    1. En la interfaz de usuario web local, vaya a **Comenzar > Activar** y escriba la clave de activación. Una vez que se aplica la clave, el dispositivo tarda unos minutos en activarse. Descargue el archivo `<device-serial-number>`.json cuando se le pida para almacenar de forma segura las claves de dispositivo necesarias para una futura recuperación. 

12. **Configuración del proceso**: En Azure Portal, vaya a **Información general > Dispositivo**. Compruebe que el dispositivo está **en línea**. En el panel izquierdo, vaya a **Proceso de Edge > Comenzar > Configurar el proceso de Edge > Proceso**. Especifique un servicio de IoT Hub existente o cree uno nuevo, y espere unos 20 minutos para que se configure el proceso. Encontrará más información en [Tutorial: Configuración del proceso en un dispositivo de Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-deploy-configure-compute.md)

Está listo para implementar cargas de trabajo de proceso en el dispositivo [a través de IoT Edge](azure-stack-edge-gpu-deploy-sample-module-marketplace.md), [de `kubectl`](azure-stack-edge-gpu-create-kubernetes-cluster.md) o [de Kubernetes habilitado para Azure Arc](azure-stack-edge-gpu-deploy-arc-kubernetes-cluster.md). Si experimenta algún problema durante la instalación, consulte la solución de problemas para [dispositivos de GPU de Azure Stack Edge Pro](azure-stack-edge-gpu-troubleshoot.md), [problemas con certificados](azure-stack-edge-gpu-certificate-troubleshooting.md) o [problemas de IoT Edge](azure-stack-edge-gpu-troubleshoot-iot-edge.md). 

## <a name="next-steps"></a>Pasos siguientes

[Instalación de Azure Stack Edge Pro con GPU](./azure-stack-edge-gpu-deploy-install.md)
