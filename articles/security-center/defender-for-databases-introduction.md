---
title: 'Azure Defender para bases de datos relacionales de código abierto: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para bases de datos relacionales de código abierto, como PostgreSQL, MySQL y MariaDB.
author: memildin
ms.author: memildin
ms.date: 05/25/2021
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: eb460fc214cd9dfb65e1ef500cfc3c152452030e
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110483211"
---
# <a name="introduction-to-azure-defender-for-open-source-relational-databases"></a>Introducción a Azure Defender para bases de datos relacionales de código abierto

Este plan de Azure Defender ofrece protección contra amenazas para las siguientes bases de datos relacionales de código abierto:

- [Azure Database para PostgreSQL](../postgresql/index.yml)
- [Azure Database for MySQL](../mysql/index.yml)
- [Azure Database for MariaDB](../mariadb/index.yml)

Azure Defender detecta actividades anómalas que indican intentos inusuales y potencialmente peligrosos de obtener acceso a las bases de datos o de vulnerar su seguridad. El plan facilita la solución de las posibles amenazas a las bases de datos sin necesidad de ser un experto en seguridad ni administrar sistemas de supervisión de seguridad avanzada.

## <a name="availability"></a>Disponibilidad

| Aspecto                             | Detalles                                                                                                                                    |
|------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------|
| Estado de la versión:                     | Disponibilidad general (GA)                                                     |
| Precios:                           | **Azure Defender para bases de datos relacionales de código abierto** se factura como se indica en [Precios de Security Center](https://azure.microsoft.com/pricing/details/security-center/).   |
| Versiones protegidas de PostgreSQL:  | Servidor único: De uso general y Optimizado para memoria. Obtenga más información en [Planes de tarifa de PostgreSQL](../postgresql/concepts-pricing-tiers.md).   |
| Versiones protegidas de MySQL:       | Servidor único: De uso general y Optimizado para memoria. Obtenga más información en [Planes de tarifa de MySQL](../mysql/concepts-pricing-tiers.md).                        |
| Versiones protegidas de MariaDB:     | De uso general y Optimizado para memoria. Obtenga más información en [Planes de tarifa de MariaDB](../mariadb/concepts-pricing-tiers.md).                      |
| Nubes:                            | ![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos) |
|                                    |                                                                                                                                            |

## <a name="what-are-the-benefits-of-azure-defender-for-open-source-relational-databases"></a>¿Qué ventajas presenta Azure Defender para bases de datos relacionales de código abierto?

Azure Defender proporciona alertas de seguridad sobre actividades anómalas para que pueda detectar posibles amenazas y responder a estas a medida que se producen.

Al habilitar este plan, Azure Defender proporcionará alertas cuando detecte patrones anómalos de consulta y acceso a la base de datos, así como actividades sospechosas en las bases de datos.

Estas alertas aparecen en la página de alertas de seguridad de Azure Defender e incluyen:

- Detalles de la actividad sospechosa que las desencadenó.
- La táctica de MITRE ATT&CK asociada.
- Acciones recomendadas para investigar y mitigar la amenaza.
- Opciones para continuar con las investigaciones con Azure Sentinel.

:::image type="content" source="media/defender-for-databases-introduction/defender-alerts.png" alt-text="Algunas de las alertas de seguridad que puede ver con las bases de datos protegidas por Azure Defender para bases de datos relacionales de código abierto." lightbox="./media/defender-for-databases-introduction/defender-alerts.png":::

## <a name="what-kind-of-alerts-does-azure-defender-for-open-source-relational-databases-provide"></a>¿Qué tipos de alertas proporciona Azure Defender para bases de datos relacionales de código abierto?

Las alertas de seguridad enriquecidas con inteligencia sobre amenazas se desencadenan en estos casos:

- **Patrones de consulta y acceso a bases de datos anómalos**: por ejemplo, un número anormalmente alto de intentos de inicio de sesión incorrectos con credenciales diferentes (un intento por fuerza bruta).
- **Actividades sospechosas en las bases de datos**: por ejemplo, un usuario legítimo que accede a un servidor SQL Server desde un equipo vulnerado que se comunica con un servidor de comando y control de minería de datos de cifrado.
- **Ataques por fuerza bruta**: con la capacidad de separar la fuerza bruta simple de la fuerza bruta en un usuario válido o una fuerza bruta correcta.

> [!TIP]
> Consulte la lista completa de alertas de seguridad de los servidores de bases de datos [en la página de referencia de alertas](alerts-reference.md#alerts-osrdb).



## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre Azure Defender para bases de datos relacionales de código abierto.

> [!div class="nextstepaction"]
> [Habilitación de Azure Defender](enable-azure-defender.md)
