---
title: Consideraciones de aplicación de revisiones de BareMetal para Oracle
description: Obtenga información acerca de las consideraciones de aplicación de revisiones de kernel o sistema operativo correspondientes a BareMetal Infrastructure para Oracle.
ms.topic: reference
ms.subservice: baremetal-oracle
ms.date: 04/14/2021
ms.openlocfilehash: 1c7a4a4f9e2395bbe29ab8bf27b18afb21f98917
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110578574"
---
# <a name="patching-considerations-for-baremetal-for-oracle"></a>Consideraciones de aplicación de revisiones de BareMetal para Oracle

En este artículo, examinaremos consideraciones importantes de aplicación de revisiones de kernel o sistema operativo correspondientes a BareMetal Infrastructure para Oracle.

Para lograr un rendimiento de red y estabilidad del sistema adecuados, instale la versión específica del sistema operativo de los controladores eNIC y fNIC, tal y como se muestra en la siguiente tabla de compatibilidad. 

Los servidores se entregan a los clientes con versiones compatibles. Sin embargo, durante la aplicación de revisiones del kernel o sistema operativo, los controladores se pueden revertir a las versiones predeterminadas del controlador. Por lo tanto, asegúrese de confirmar que se esté ejecutando la versión adecuada del controlador después de las operaciones de aplicación de revisiones del sistema operativo o kernel.

| Fabricante del sistema operativo | Versión del paquete del sistema operativo | Versión de firmware | Controlador eNIC | Controlador fNIC |
| --- | --- | --- | --- | --- |
| Red Hat | RHEL 7.6 | 3.2.3i | 2.3.0.53 | 1.6.0.34 |
| Red Hat | RHEL 7.6 | 4.1.1b | 2.3.0.53 | 1.6.0.34 |

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de la configuración de Ethernet de BareMetal para Oracle.

> [!div class="nextstepaction"]
> [Configuración de Ethernet de BareMetal para Oracle](oracle-baremetal-ethernet.md)

