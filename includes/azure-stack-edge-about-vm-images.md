---
author: v-dalc
ms.service: databox
ms.author: alkohli
ms.topic: include
ms.date: 04/14/2021
ms.openlocfilehash: a33a702128be0d8e90b3db635001dbfbdb279f2a
ms.sourcegitcommit: d3bcd46f71f578ca2fd8ed94c3cdabe1c1e0302d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107575955"
---
|Tipo de imagen  |Generalizada  |Especializada  |
|---------|---------|---------|
|Destino     |Implementado en cualquier sistema.         | Destinado a un sistema específico.        |
|Configuración después del arranque     | Configuración requerida en el primer arranque de la máquina virtual.          | No se requiere ninguna instalación. <br> La plataforma enciende la máquina virtual.        |
|Configuración     |Nombre de host, administrador-usuario y otros valores específicos de la máquina virtual necesarios.         |Preconfigurado.         |
|Se usa al     |Crear varias máquinas virtuales nuevas a partir de la misma imagen.         |Migrar una máquina específica o restaurar una máquina virtual a partir de una copia de seguridad anterior.         |
