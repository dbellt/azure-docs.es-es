---
title: Actualización de Ubuntu 18.04 para Azure HDInsight
description: Obtenga información sobre los cambios en el sistema operativo Ubuntu 18.04 para Azure HDInsight.
ms.service: hdinsight
ms.topic: conceptual
ms.author: yanacai
author: yanancai
ms.date: 05/25/2021
ms.openlocfilehash: 61d801d7091d2a619ff6a8b6436f386c125ca4be
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111415379"
---
# <a name="hdinsight-ubuntu-1804-os-update"></a>Actualización del sistema operativo Ubuntu 18.04 para HDInsight

En este artículo se proporcionan más detalles sobre la actualización del sistema operativo Ubuntu 18.04 para HDInsight y los posibles cambios que se necesitan.

## <a name="update-overview"></a>Información general de actualización

HDInsight ha empezado a implementar la nueva imagen de clúster de HDInsight 4.0 que se ejecuta en Ubuntu 18.04 en mayo de 2021. Los clústeres de HDInsight 4.0 recién creados se ejecutarán en Ubuntu 18.04 de forma predeterminada una vez que esté disponible. Los clústeres existentes en Ubuntu 16.04 se ejecutarán tal y como están con soporte técnico completo.

HDInsight 3.6 seguirá funcionando en Ubuntu 16.04. El soporte técnico estándar terminará el 30 de junio de 2021 y cambiará a soporte técnico Basic a partir del 1 de julio de 2021. Para más información sobre las fechas y las opciones de soporte técnico, consulte las [versiones de Azure HDInsight](./hdinsight-component-versioning.md). Ubuntu 18.04 no será compatible con HDInsight 3.6. Si desea usar Ubuntu 18.04, deberá migrar los clústeres a HDInsight 4.0.

Quite y vuelva a crear los clústeres si desea mover los existentes a Ubuntu 18.04. Piense en crear o volver a crear el clúster. 

## <a name="script-actions-changes"></a>Cambios en las acciones de script

Las acciones de script de HDInsight se usan para instalar componentes adicionales y cambiar la configuración. Una acción de script es un script de Bash que se ejecuta en los nodos de un clúster de HDInsight.

Puede haber algunos posibles cambios que debe realizar para las acciones de script.

**Cambie cada instancia de `xenial` a `bionic` al capturar los paquetes siempre que sea necesario:**

Por ejemplo:
- Actualice `http://packages.treasuredata.com/3/ubuntu/xenial/ xenial contrib` a `http://packages.treasuredata.com/3/ubuntu/bionic/ bionic contrib`.
- Actualice `http://azure.archive.ubuntu.com/ubuntu/ xenial main restricted` a `http://azure.archive.ubuntu.com/ubuntu/ bionic main restricted`.

**Algunas versiones de paquete no están presentes para bionic:** 

Por ejemplo, [Node.js versión 4.x](https://deb.nodesource.com/node_4.x/dists/) no está presente en el repositorio de bionic. [Node.js versión 12.x](https://deb.nodesource.com/node_12.x/dists/bionic/) está presente.

Los scripts que instalan versiones anteriores que no están presentes para bionic deben actualizarse a versiones posteriores.

**/etc/rc.local no existe de forma predeterminada en la versión 18.04:**

Algunos scripts se usan `/etc/rc.local` para inicios de servicio, pero no existe de forma predeterminada en Ubuntu 18.04. Debe convertirse en una unidad de sistema adecuada. 

**Se han actualizado los paquetes base del sistema operativo:**

Si los scripts se basan en un paquete de versión anterior en Ubuntu 16.04, es posible que no funcione. Utilice SSH en el nodo de clúster y ejecute `dpkg --list` en dicho nodo para mostrar los detalles de todos los paquetes instalados.
 
**En general, Ubuntu 18.04 tiene reglas más estrictas que la versión 16.04.**

## <a name="custom-applications"></a>Aplicaciones personalizadas
Algunas [aplicaciones de terceros](./hdinsight-apps-install-applications.md) se pueden instalar en el clúster de HDInsight. Es posible que esas aplicaciones no funcionen bien con Ubuntu 18.04. Para reducir el riesgo de cambios importantes, HDInsight no implantará la nueva imagen para las suscripciones que tenían instaladas aplicaciones personalizadas desde el 25 de febrero de 2021. Si desea probar la nueva imagen con las suscripciones de prueba, abra una incidencia de soporte técnico para habilitar la suscripción.

## <a name="edge-nodes"></a>Nodos perimetrales
Con la nueva imagen, el sistema operativo para los nodos perimetrales del clúster también se actualizará a Ubuntu 18.04. Los clientes existentes deben probarse con Ubuntu 18.04. Para reducir el riesgo de cambios importantes, HDInsight no implantará la nueva imagen para las suscripciones que habían usado nodos perimetrales desde el 25 de febrero de 2021. Si desea probar la nueva imagen con las suscripciones de prueba, abra una incidencia de soporte técnico para habilitar la suscripción.

## <a name="references"></a>Referencias
 - [Notas de la versión de Ubuntu 18.04 LTS](https://wiki.ubuntu.com/BionicBeaver/ReleaseNotes/)





