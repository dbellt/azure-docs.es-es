---
title: Tutorial de instalación del dispositivo físico Azure Stack Edge Mini R | Microsoft Docs
description: El segundo tutorial sobre la instalación del dispositivo Azure Stack Edge Mini R explica cómo cablear el dispositivo físico a la alimentación y la red.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 03/18/2021
ms.author: alkohli
ms.openlocfilehash: 8bf73374545fa10c1b26ccbc9b932962fcabb6b3
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108759876"
---
# <a name="tutorial-install-azure-stack-edge-mini-r"></a>Tutorial: Instalación de Azure Stack Edge Mini R

Este tutorial describe cómo instalar un dispositivo físico Azure Stack Edge Mini R. El procedimiento de instalación implica el cableado del dispositivo.

La instalación puede tardar alrededor de 30 minutos.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Inspección del dispositivo
> * Cableado del dispositivo

## <a name="prerequisites"></a>Requisitos previos

Los requisitos previos para instalar un dispositivo físico son los siguientes:

### <a name="for-the-azure-stack-edge-resource"></a>Para el recurso de Azure Stack Edge

Antes de comenzar, asegúrese de que:

* Ha completado todos los pasos descritos en [Preparación de la implementación de Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-prep.md).
    * Ha creado un recurso de Azure Stack Edge para implementar el dispositivo.
    * Ha generado la clave de activación para activar el dispositivo con el recurso de Azure Stack Edge.

 
### <a name="for-the-azure-stack-edge-mini-r-physical-device"></a>Para el dispositivo físico Azure Stack Edge Mini R

Antes de implementar un dispositivo:

- Asegúrese de colocar el dispositivo de forma segura sobre una superficie de trabajo plana, estable y nivelada.
- Compruebe que el sitio donde desea realizar la instalación tiene:
    - Corriente alterna estándar de una fuente independiente, o
    - Una unidad de distribución de energía (PDU) de bastidor. 
    

### <a name="for-the-network-in-the-datacenter"></a>Para la red del centro de datos

Antes de empezar:

- Revise los requisitos de redes para implementar Azure Stack Edge Mini R y configure la red del centro de datos según dichos requisitos. Para más información, consulte los [Requisitos de red de Azure Stack Edge](azure-stack-edge-mini-r-system-requirements.md#networking-port-requirements).

- Asegúrese de que el ancho de banda mínimo de Internet sea de 20 Mbps para que el dispositivo funcione de forma óptima. <!-- engg TBC -->


## <a name="inspect-the-device"></a>Inspección del dispositivo

Este dispositivo se suministra como una única unidad. Realice los pasos siguientes para desempaquetar el dispositivo.

1. Coloque la caja en una superficie plana y nivelada.
2. Compruebe si la caja del dispositivo tiene daños. Abra la caja e inspeccione el dispositivo. Si la caja o el dispositivo parecen dañados, póngase en contacto con el servicio de soporte técnico de Microsoft para que le ayude a determinar si el dispositivo está en buen estado.
3. Una vez abierta la caja, asegúrese de que dispone de los siguientes elementos:
    - Un dispositivo Azure Stack Edge Mini R portátil con protectores laterales adjuntos
    - Una batería y la cubierta trasera conectada al dispositivo. 
    - Un cable de alimentación para conectar la batería a la fuente de alimentación 

Si no recibió alguno de los elementos enumerados aquí, póngase en contacto con el soporte técnico de Azure Stack Edge. El siguiente paso consiste en cablear el dispositivo.


## <a name="cable-the-device"></a>Cableado del dispositivo

En los siguientes procedimientos se describe cómo realizar el cableado del dispositivo de Azure Stack Edge para la conexión a la corriente eléctrica y a la red.

Antes de empezar el cableado del dispositivo, necesita lo siguiente:

- El dispositivo físico Azure Stack Edge Mini R en el sitio de instalación.
- Un cable de alimentación.
- Al menos un cable de red RJ-45 de 1-GbE cable para conectarse a la interfaz de administración de red. Hay dos interfaces de red de 1-GbE, uno de administración y otro de datos, en el dispositivo.
- Un cable SFP+ de 10 GbE para cada interfaz de red de datos que se va a configurar. Al menos una interfaz de red de datos desde el puerto 3 o el puerto 4 debe estar conectada a Internet (con conectividad con Azure).

  Se recomienda el uso del transceptor SFP+ de cobre de mayor rendimiento (10 Gbps). Se pueden usar transceptores de fibra óptica compatibles, pero no se han probado. Para más información, consulte [Especificaciones de transceptor y cable](azure-stack-edge-mini-r-technical-specifications-compliance.md#transceivers-cables) para Azure Stack Edge Mini R.
   
- Acceso a una unidad de distribución de energía (recomendado).

> [!NOTE]
> - Si va a conectar solo una interfaz de red de datos, es recomendable que use una interfaz de red de 10 GbE en el puerto 3 o el puerto 4 para enviar datos a Azure. 
> - Para obtener el mejor rendimiento y controlar grandes volúmenes de datos, considere la posibilidad de conectar todos los puertos de datos.
> - El dispositivo de Azure Stack Edge debe estar conectado a la red del centro de datos para que pueda realizar la ingesta de datos desde los servidores de origen de datos. <!-- engg TBC -->

En el dispositivo de Azure Stack Edge:

- El panel frontal tiene una bahía para SSD. 

    - El dispositivo tiene 1 disco SSD en la ranura. 
    - El dispositivo también tiene una tarjeta CFx que sirve de almacenamiento para el disco del sistema operativo.
    
- El panel frontal tiene interfaces de red y acceso a Wi-Fi. 

    - 2 interfaces de red RJ 45 de 1 GbE (PUERTO 1 y PUERTO 2 en la interfaz de usuario local del dispositivo)
    - 2 interfaces de red SFP+ de 10 GbE (PUERTO 3 y PUERTO 4 en la interfaz de usuario local del dispositivo) 
    - Un puerto Wi-Fi con un transceptor Wi-Fi conectado a él.

- El panel frontal también tiene un botón de encendido. 

- El panel posterior incluye una batería y una cubierta que están instaladas en el dispositivo. 


Realice los pasos siguientes para realizar el cableado de los cables de alimentación y de red del dispositivo.

1. Identifique los distintos componentes de red y almacenamiento en el plano frontal del dispositivo.

    ![Interfaces de red y de almacenamiento en el dispositivo](./media/azure-stack-edge-mini-r-deploy-install/ports-front-plane.png)

2. Busque el botón de encendido en la esquina inferior izquierda de la parte frontal del dispositivo. 

    ![Plano frontal de un dispositivo con el botón de encendido en el dispositivo](./media/azure-stack-edge-mini-r-deploy-install/device-power-button.png)

3. La batería está conectada al plano posterior del dispositivo. Identifique el segundo botón de encendido que se encuentra en la batería. 

    ![Plano frontal de un dispositivo con el botón de encendido en la batería](./media/azure-stack-edge-mini-r-deploy-install/battery-power-button.png)


    Conecte un extremo del cable de alimentación a la batería y el otro a la toma de corriente. 

    ![Conexión del cable de alimentación](./media/azure-stack-edge-mini-r-deploy-install/power-cord-connector-1.png) 

    Cuando solo se utiliza con batería (la batería no está conectada a la fuente de alimentación), el interruptor de alimentación de la parte frontal y el conmutador de la batería deben estar activados en posición ON. Cuando la batería está conectada a una fuente de alimentación, solo el botón de encendido situado en la parte frontal del dispositivo debe estar activado en posición ON. 

4. Pulse el botón de encendido del plano frontal para encender el dispositivo. 
    
    > [!NOTE]
    > Para activar o desactivar la alimentación del dispositivo, tiene que presionar el botón negro situado en la parte superior del botón de encendido y, a continuación, alternar el botón de encendido hasta la posición ON u OFF. 

5. Si va a configurar Wi-Fi en este dispositivo, use el siguiente diagrama de cableado:

    ![Cableado para Wi-Fi](./media/azure-stack-edge-mini-r-deploy-install/wireless-cabled.png)  

    - Conecte el PUERTO 1 de la interfaz de red de 1-GbE al equipo que se usa para configurar el dispositivo físico. El PUERTO 1 es la interfaz de administración dedicada.


    Si usa una configuración con cables para este dispositivo, use el siguiente diagrama:
     
    ![Cableado para conexión con cables](./media/azure-stack-edge-mini-r-deploy-install/wired-cabled.png)     
    - Conecte el PUERTO 1 de la interfaz de red de 1-GbE al equipo que se usa para configurar el dispositivo físico. El PUERTO 1 es la interfaz de administración dedicada.
    - Conecte uno o varios de los puertos 2, 3 o 4 a la red del centro de datos o a Internet.
    
        - Si conecta el PUERTO 2, utilice el cable de red RJ-45.
        - Para las interfaces de red de 10 GbE, use cables de cobre SFP+.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha obtenido información acerca de varios temas relacionados con Azure Stack Edge, como:

> [!div class="checklist"]
> * Desempaquetado del dispositivo
> * Cableado del dispositivo

Pase al siguiente tutorial para aprender a conectar, configurar y activar el dispositivo.

> [!div class="nextstepaction"]
> [Conexión y configuración de Azure Stack Edge](./azure-stack-edge-mini-r-deploy-connect.md)
