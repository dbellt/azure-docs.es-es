---
title: 'Tutorial: Instalación de un dispositivo físico de Azure FXT Edge Filer'
description: Cómo desempaquetar, colocar en un bastidor e instalar los cables del componente de dispositivo físico de la memoria caché de almacenamiento híbrido Microsoft Azure FXT Edge Filer
author: ekpgh
ms.author: v-erkel
ms.service: fxt-edge-filer
ms.topic: tutorial
ms.date: 07/01/2019
ms.openlocfilehash: 626ec99d59979e13121b8c32ee2f24843328697b
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111358437"
---
# <a name="tutorial-install-azure-fxt-edge-filer"></a>Tutorial: Instalación de Azure FXT Edge Filer

Este tutorial describe cómo instalar un nodo de hardware para la caché de almacenamiento híbrido de Azure FXT Edge Filer. Debe instalar al menos tres nodos de hardware para crear un clúster de Azure FXT Edge Filer.

El procedimiento de instalación consiste en desempaquetar y montar en un bastidor el dispositivo; a continuación, debe colocar el brazo de administración de cables (CMA) y el bisel frontal. En un tutorial independiente se explica cómo conectar los cables de red y cómo conectar la alimentación.

Se tarda aproximadamente una hora en instalar un nodo de Azure FXT Edge Filer.

En este tutorial se incluyen estos pasos de configuración:

> [!div class="checklist"]
>
> * Desempaquetado del dispositivo
> * Montaje en bastidor del dispositivo
> * Instalación del bisel frontal (opcional)

## <a name="installation-prerequisites"></a>Requisitos previos de instalación

Antes de comenzar, asegúrese de que el centro de datos y el bastidor que usará tengan estas características:

* Una ranura 1U disponible en el bastidor en el que se va a montar el dispositivo.
* Una fuente de alimentación de CA y sistemas de refrigeración que satisfagan las necesidades de Azure FXT Edge Filer. (Lea [Power and thermal specifications](fxt-specs.md#power-and-thermal-specifications) [Especificaciones de energía y térmicas] para obtener ayuda para planificar y calibrar la instalación).  

  > [!NOTE]
  > Para aprovechar al máximo las dos unidades de alimentación redundantes (PSU), use las unidades de distribución de energía en dos circuitos de rama diferentes cuando conecte la alimentación CA. Lea [Connect power cables](fxt-network-power.md#connect-power-cables) (Conectar los cables de alimentación) para obtener más detalles.  

## <a name="unpack-the-hardware-node"></a>Desempaquetar el nodo de hardware

Cada nodo de Azure FXT Edge Filer se envía en una sola caja. Complete estos pasos para desempaquetar un dispositivo.

1. Coloque la caja en una superficie plana y nivelada.

2. Compruebe si la caja y la espuma del embalaje presentan golpes, cortes, daños por agua o cualquier otro daño evidente. Si la caja o el embalaje están muy dañados, no los abra. Póngase en contacto con el soporte técnico de Microsoft para ayudarle a determinar si el dispositivo está en buen estado.

3. Desempaquete la caja. Asegúrese de que incluye los siguientes elementos:
   * Un único dispositivo contenedor FXT
   * Dos cables de alimentación
   * Un bisel frontal y una llave
   * Un ensamblaje de kit de raíl
   * Un brazo de administración de cables (CMA)
   * Un folleto con las instrucciones de instalación de CMA
   * Un folleto con las instrucciones de instalación del bastidor
   * Un folleto informativo sobre la seguridad, el entorno y las normativas

Si no recibió alguno de los elementos enumerados aquí, póngase en contacto con el vendedor del dispositivo.

Asegúrese de que el dispositivo haya tenido el tiempo suficiente para alcanzar la misma temperatura que la habitación antes de instalarlo o encenderlo. Si observa condensación en cualquier parte del dispositivo, espere al menos 24 horas antes de instalarlo.

El paso siguiente es el montaje en bastidor del dispositivo.

## <a name="rack-the-device"></a>Montaje del dispositivo en el bastidor

El dispositivo Azure FXT Edge Filer debe instalarse en un bastidor estándar de 19 pulgadas.

La memoria caché de almacenamiento híbrido de Azure FXT Edge Filer se compone de tres o más dispositivos de Azure FXT Edge Filer. Repita los pasos de instalación del bastidor para cada dispositivo que vaya a formar parte de su sistema.

### <a name="rack-install-prerequisites"></a>Requisitos previos de instalación del bastidor

* Antes de comenzar, lea las instrucciones de seguridad en el folleto sobre la seguridad, el entorno y las normativas que se envió junto con el dispositivo.

  > [!NOTE]
  > Asegúrese de que siempre hay dos personas para levantar el nodo, incluso al instalarlo en un bastidor o lo retirarlo del mismo.

* Identifique el tipo de instalación de raíl que se usa con el bastidor de su equipo.
  * Para los bastidores a presión con orificios cuadrados o redondos, siga las instrucciones de montaje de los raíles no mecanizados.
  * Para los bastidores con orificios a rosca, siga las instrucciones para montar los raíles de forma mecanizada.
  
    Para la configuración de montaje del raíl con herramientas, necesitará los siguientes ocho tornillos: 10-32, 12-24, M5 o M6. El diámetro de los cabezales de los tornillos debe ser inferior a 10 mm (0,4 pulgadas).

### <a name="identify-the-rail-kit-contents"></a>Identificación del contenido del kit de raíl

Ubique los componentes de instalación de ensamblaje del kit de raíl:

1. Dos ensamblajes de raíl deslizantes A7 Dell ReadyRails II (1)
1. Dos correas anchas con cierre de velcro (2)

![Dibujo numerado de los contenidos del kit correspondiente al raíl](media/fxt-install/identify-rail-kit-contents-400.png)

### <a name="rail-assembly---tool-less-rails-square-hole-or-round-hole-racks"></a>Montaje de los raíles no mecanizados (bastidores de agujeros cuadrados y redondos)

Para bastidores con orificios cuadrados o redondos a presión, siga este procedimiento para ensamblar e instalar los raíles.

1. Coloque las piezas de los raíles izquierdo y derecho etiquetadas como **FRONTAL** mirando hacia adentro. Coloque cada pieza de manera que coincida con los orificios del lado frontal de las bridas verticales del bastidor. (1)

2. Alinee el extremo de cada pieza en los orificios inferior y superior del bastidor en el espacio que quiera montar. (2)

3. Ajuste el extremo posterior del raíl hasta que quede fijado en la pestaña del bastidor en vertical y el pestillo haga clic. Repita estos pasos para colocar y fijar la pieza del extremo anterior en la pestaña del bastidor en vertical. (3)

> [!TIP]
> Para quitar los raíles, suelte el botón de desbloqueo del pestillo de la parte central de la pieza del extremo (4) y levante los raíles.

![Diagrama de instalación y extracción de raíles no mecanizados con pasos numerados](media/fxt-install/installing-removing-tool-less-rails-400.png)

### <a name="rail-assembly---tooled-rails-threaded-hole-racks"></a>Montaje de raíles mecanizados (bastidores con agujeros a rosca)

Para bastidores con orificios a rosca, siga este procedimiento para ensamblar e instalar los raíles.

1. Quite las fijaciones de los soportes de montaje anterior y posterior con un destornillador plano. (1)
1. Tire de los submódulos de pestillo del raíl y gírelos para sacarlos de los soportes de montaje. (2)
1. Encaje los raíles de montaje izquierdo y derecho a las pestañas anteriores del bastidor en vertical con 4 tornillos. (3)
1. Deslice los soportes posteriores izquierdo y derecho hacia delante contra las pestañas posteriores del bastidor en vertical y encájelos con 4 tornillos. (4)

![Diagrama de instalación y extracción de raíles, con pasos numerados](media/fxt-install/installing-removing-tooled-rails-400.png)

### <a name="install-the-system-in-the-rack"></a>Instalación del sistema en un bastidor

Siga estos pasos para montar el dispositivo Azure FXT Edge Filer en el bastidor.

1. Tire de los raíles deslizantes interiores hacia fuera del bastidor hasta que se bloqueen en su sitio. (1)
1. Ubique los separadores de los raíles posteriores laterales del dispositivo y encájelos en las ranuras en J de los ensamblajes deslizantes. (2)
1. Gire el dispositivo hacia abajo hasta que todos los separadores de los raíles queden fijos en las ranuras en J. (3)
1. Inserte el dispositivo hasta que las pestañas de cierre hagan clic.
1. Presione los botones de bloqueo deslizantes de los dos raíles (4) y deslice el dispositivo para que encaje en el bastidor.

![Instalación del sistema en un diagrama de bastidor, con pasos numerados](media/fxt-install/installing-system-rack-400.png)

### <a name="remove-the-system-from-the-rack"></a>Extracción del sistema del bastidor

Para extraer el dispositivo del bastidor, siga este procedimiento.

1. Ubique las pestañas de cierre laterales en los raíles internos (1).
2. Desbloquee las pestañas; para ello, gírelas hasta la posición de desbloqueo (2).
3. Sujete firmemente los laterales del sistema e inserte este hasta que los separadores de los raíles queden frente a las ranuras en J. Levante el sistema y sáquelo del bastidor para colocarlo en una superficie nivelada (3).

![Ilustración de cómo retirar un sistema del bastidor, con los pasos numerados](media/fxt-install/removing-system-rack-400.png)

### <a name="engage-the-slam-latch"></a>Enganchar el pestillo a presión

1. De cara a la parte anterior, ubique el pestillo a presión (1) en alguno de los laterales del sistema.
2. Los pestillos se encajan automáticamente al insertar el sistema en el bastidor.

Para liberar los pestillos al retirar el sistema, tire de ellos hacia arriba (2).

Se proporcionan tornillos rígidos para asegurar el sistema al bastidor para su envío o si se va a usar en entornos inestables. Busque los tornillos debajo de cada pestillo y apriételos con un destornillador Phillips n.º 2 (3).

![Ilustración numerada para acoplar y soltar el pestillo a presión](media/fxt-install/engaging-releasing-slam-latch-400.png)

### <a name="install-the-cable-management-arm"></a>Instale el brazo de administración de cables

Se proporciona un brazo de administración de cables (CMA) opcional con FXT Edge Filer. Las instrucciones impresas para instalarlo se proporcionan en el paquete.

1. Desembale e identifique los componentes del kit del brazo de administración de cables:
   * Bandeja de CMA (1)
   * CMA (2)
   * Envolturas de cables de nylon (3)
   * Soportes de fijación CMA (4)
   * Cable indicador de estado (5)

   > [!TIP]
   > Para asegurar el CMA para su envío en el bastidor, enrolle las envolturas de en torno a ambas cestas y a la bandeja y átelas firmemente. Si asegura el CMA de esta manera, también asegurará su sistema en entornos inestables.

   ![Ilustración de piezas de CMA](media/fxt-install/cma-kit-400.png)

2. Instale la bandeja de CMA.

   La bandeja de CMA proporciona soporte y actúa de sostén para el CMA.

   1. Alinee y enganche cada lado de la bandeja con los soportes del receptor en los bordes internos de los raíles.
   1. Empuje la bandeja hacia adelante hasta que encaje en su lugar. (1)
   1. Para retirar la bandeja, presione los botones de liberación hacia el centro y tire de la bandeja para extraerla de los soportes del receptor (2).

   ![Ilustración de la instalación de la bandeja de CMA](media/fxt-install/cma-tray-install-400.png)

3. Instale los soportes de fijación de CMA.

   > [!NOTE]
   >
   > * Puede colocar el CMA en el raíl de montaje derecho o izquierdo, dependiendo de cómo intente enrutar los cables desde el sistema.
   > * Para mayor comodidad, monte el CMA en el lado opuesto a las fuentes de alimentación (lado A). Si está montado en el lado B, el CMA debe desconectarse para poder quitar la fuente de alimentación externa.
   > * Retire siempre la bandeja antes de retirar las fuentes de alimentación.

   ![Ilustración de la instalación del soporte de CMA](media/fxt-install/cma-bracket-l-r-install-400.png)

   1. Seleccione el soporte de fijación de CMA apropiado para el lado donde quiera montar el CMA (lado B o lado A).
   1. Instale el soporte de fijación de CMA con la marca correspondiente del lado A o del lado B en la parte posterior del raíl deslizante.
   1. Alinee los orificios del soporte con los anclajes del raíl deslizante. Empuje el soporte hacia abajo hasta que encaje en su lugar.

4. Instale el CMA.

   1. En la parte posterior del sistema, coloque el pestillo en la parte delantera del CMA en el soporte más interno del ensamblaje deslizante hasta que el pestillo encaje (1).
   1. Coloque el otro pestillo en el extremo del soporte más externo hasta que el pestillo encaje (2).
   1. Para retirar el CMA, desenganche ambos pestillos presionando los botones de liberación de CMA que están en la parte superior de las carcasas del pestillo interior y exterior (3).

   ![Ilustración de la instalación principal de CMA](media/fxt-install/cma-install-400.png)

   El CMA se puede girar mas allá del sistema para el acceso y el servicio. En el extremo con bisagras, levante el CMA de la bandeja para sacarlo (1). Una vez que haya salido de la bandeja, saque el CMA del sistema (2).

   ![Ilustración del CMA girado y abierto para el servicio](media/fxt-install/cma-swing-over-tray-400.png)

## <a name="install-the-front-bezel-optional"></a>Instalación del bisel frontal (opcional)

En esta sección se explica cómo instalar y quitar el bisel frontal (placa frontal) del hardware de Azure FXT Edge Filer.

Para instalar el bisel frontal:

1. Localice y quite la llave del bisel que se proporciona en el paquete del mismo.
1. Alinee el bisel con la parte frontal del chasis e inserte los anclajes en el lado derecho del bisel, justo en los orificios de la brida de montaje del bastidor que está en el lado derecho del nodo.
1. Coloque el extremo izquierdo del bisel en el chasis. Presione el bisel hasta que el botón del lado izquierdo encaje en su lugar.
1. Bloquee el bisel con la llave.

Para quitar el bisel frontal:

1. Desbloquee el bisel mediante la llave del mismo.
1. Presione el botón de liberación del lado izquierdo y tire del extremo izquierdo del bisel para extraerlo del chasis.
1. Desenganche el extremo derecho y retire el bisel.

   ![Imagen que muestra el botón de liberación a la izquierda del bisel y cómo extraerlo tirando hacia afuera desde el lado izquierdo](media/fxt-install/remove-bezel-edited-600.png)

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya desempaquetado y montado el dispositivo en el bastidor, continúe con la instalación conectando los cables de red y la alimentación de CA a Azure FXT Edge Filer.

> [!div class="nextstepaction"]
> [Conectar los cables de los puertos de red y del suministro de energía](fxt-network-power.md)
