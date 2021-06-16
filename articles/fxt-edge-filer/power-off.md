---
title: Cómo apagar una unidad de Microsoft Azure FXT Edge Filer
description: Obtenga información sobre los procedimientos de inicio y apagado seguro de un nodo de Azure FXT Edge Filer mediante el software del panel de control del clúster.
author: ekpgh
ms.service: fxt-edge-filer
ms.topic: how-to
ms.date: 07/01/2019
ms.author: v-erkel
ms.openlocfilehash: 062a5ff1c335b64e8e9cd3fc2eba54c186599034
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111415391"
---
# <a name="how-to-safely-power-off-azure-fxt-edge-filer-hardware"></a>Cómo apagar de forma segura el hardware de Azure FXT Edge Filer

Aunque puede usar el botón de encendido físico para activar un nodo individual, no debe usarlo para apagar la unidad en circunstancias normales.

Después de que un nodo de Azure FXT Edge Filer esté en uso como parte de un clúster, debe usar el software de panel de control del clúster para apagar el hardware.

> [!NOTE]
> Para evitar la posible pérdida o corrupción de los datos, use siempre el software Panel de control para apagar una instancia de Azure FXT Edge Filer. No utilice el botón de encendido físico para apagarla, salvo que el soporte técnico y servicio al cliente de Microsoft le indique que lo haga.
>
> En caso de emergencia eléctrica, desconecte los cables de alimentación o use el mecanismo de desconexión de la electricidad del centro de datos.

## <a name="shut-down-a-node-from-the-control-panel"></a>Apagar un nodo desde el Panel de control

Siga estas instrucciones para apagar un nodo de Azure FXT Edge Filer con seguridad:

1. Inicie sesión en el Panel de control del clúster. (Consulte las instrucciones en [Apertura de las páginas de configuración](cluster-create.md#open-the-settings-pages))
1. Haga clic en la pestaña **Configuración** y, después, cargue la página **Clúster** > **FXT Nodes** (Nodos FXT).
1. En la lista de nodos del clúster, busque el que quiere apagar. Haga clic en el botón **Apagar** de la columna **Acciones** correspondiente.
1. Espere unos segundos. El nodo se apagará.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre los LED de estado y otros indicadores en [Supervisión del estado del hardware de Azure FXT Edge Filer](monitor.md).
* Obtenga más información sobre las fuentes de alimentación de Azure FXT Edge Filer en [Conexión de los cables de alimentación](network-power.md#connect-power-cables).
