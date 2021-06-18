---
title: Creación de una copia de seguridad a petición de Oracle Database con SnapCenter
description: Obtenga información sobre cómo crear una copia de seguridad a petición de Oracle Database con SnapCenter en Oracle BareMetal Infrastructure.
ms.topic: how-to
ms.subservice: baremetal-oracle
ms.date: 05/07/2021
ms.openlocfilehash: 699c070a3eeca2904f9620ca5f12c95c509210ab
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110579162"
---
# <a name="create-on-demand-backup-of-your-oracle-database-in-snapcenter"></a>Creación de una copia de seguridad a petición de Oracle Database con SnapCenter

En este artículo, se explica cómo crear una copia de seguridad a petición de Oracle Database con SnapCenter. 

Una vez [configurado SnapCenter](configure-snapcenter-oracle-baremetal.md), las copias de seguridad de los archivos de datos, los archivos de control y los registros de archivo seguirán basándose en la programación que haya especificado al crear los grupos de recursos. Sin embargo, como parte de la protección normal de la base de datos, también puede querer copias de seguridad a petición.

## <a name="steps-to-create-an-on-demand-backup"></a>Pasos para crear una copia de seguridad a petición

1. Seleccione **Resources** (Recursos) en el menú izquierdo. A continuación, en el menú desplegable situado junto a **View** (Ver), seleccione **Resource Group** (Grupo de recursos). Seleccione el nombre del grupo de recursos correspondiente a la copia de seguridad a petición que quiere crear.

2. Seleccione **Back up now** (Hacer copia de seguridad ahora) en la esquina superior derecha.

3. Compruebe que el grupo de recursos y la directiva de protección sean correctos para la copia de seguridad a petición. Active la casilla **Verify after backup** (Comprobar después de la copia de seguridad) si quiere comprobar esta copia de seguridad. Seleccione **Backup** (Hacer copia de seguridad).

Una vez completada la copia de seguridad, estará disponible en la lista de copias de seguridad en **Resources** (Recursos). Seleccione la o las bases de datos asociadas al grupo de recursos del que ha hecho la copia de seguridad. Esta copia de seguridad se conservará según la directiva de retención a petición que estableció cuando creó la directiva de protección.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo restaurar Oracle Database con SnapCenter:

> [!div class="nextstepaction"]
> [Restaurar Oracle Database](restore-oracle-database-baremetal.md)
