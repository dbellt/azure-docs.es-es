---
title: Determinación del tamaño e intervalo de subred necesarios
titleSuffix: Azure SQL Managed Instance
description: En este tema se describe cómo calcular el tamaño de la subred en la que se implementará Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: deployment-configuration
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova
ms.date: 02/22/2019
ms.openlocfilehash: 18592f282cb2f06b5a305f2186aa6285bc50fcf3
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110689345"
---
# <a name="determine-required-subnet-size--range-for-azure-sql-managed-instance"></a>Determinación del tamaño e intervalo de subred necesarios para Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Instancia administrada de Azure SQL tiene que implementarse dentro de una [red virtual (VNet)](../../virtual-network/virtual-networks-overview.md) de Azure.

El número de instancias administradas que se puede implementar en la subred de una red virtual depende del tamaño de la subred (intervalo de subred).

Cuando se crea una instancia administrada, Azure asigna un número de máquinas virtuales en función del nivel que seleccione durante el aprovisionamiento. Debido a que estas máquinas virtuales están asociadas a la subred, requieren direcciones IP. Para garantizar la alta disponibilidad durante las operaciones normales y el mantenimiento del servicio, Azure puede asignar máquinas virtuales adicionales. Como resultado, el número de direcciones IP necesarias en una subred es mayor que el número de instancias administradas en esa subred.

Por definición, una instancia administrada necesita un mínimo de 32 direcciones IP en una subred. Como resultado, puede usar una máscara de subred mínima de /27 al definir los intervalos IP de la subred. Se recomienda una planeación cuidadosa del tamaño de la subred para las implementaciones de instancia administrada. Las entradas que se deben tener en cuenta durante la planeación son las siguientes:

- Número de instancias administradas que incluyen los siguientes parámetros de instancia:
  - Nivel de servicio
  - Generación de hardware
  - Número de núcleos virtuales
- Planes para escalar o reducir verticalmente, o cambiar el nivel de servicio

> [!IMPORTANT]
> Un tamaño de subred con 16 direcciones IP (máscara de subred /28) permitirá implementar la instancia administrada en ella, pero solo se debe usar para implementar una única instancia utilizada para la evaluación o en escenarios de desarrollo y pruebas, en los que no se realizarán operaciones de escalado.

## <a name="determine-subnet-size"></a>Determinación del tamaño de la subred

Ajuste el tamaño de la subred según las necesidades futuras de implementación y escalado de instancias. Los parámetros siguientes pueden ayudarle a realizar un cálculo:

- Azure utiliza cinco direcciones IP de la subred para sus propias necesidades
- Cada clúster virtual asigna un número adicional de direcciones 
- Cada instancia administrada usa el número de direcciones que depende del plan de tarifa y la generación de hardware

> [!IMPORTANT]
> No se puede cambiar el intervalo de direcciones de subred si existe algún recurso en la subred. Tampoco se pueden trasladar instancias administradas de una subred a otra. Siempre que sea posible, considere la posibilidad de usar subredes más grandes en lugar de más pequeñas para evitar problemas en el futuro.

GP = de uso general; BC = crítico para la empresa; VC = clúster virtual

| **Generación del hardware** | **Plan de tarifa** | **Uso de Azure** | **Uso de clústeres virtuales** | **Uso de instancias** | **Total*** |
| --- | --- | --- | --- | --- | --- |
| Gen4 | GP | 5 | 1 | 5 | 11 |
| Gen4 | BC | 5 | 1 | 5 | 11 |
| Gen5 | GP | 5 | 6 | 3 | 14 |
| Gen5 | BC | 5 | 6 | 5 | 16 |

  \* El total de columnas muestra el número de direcciones que se usarán al implementar una instancia en la subred. Cada instancia adicional en la subred agrega el número de direcciones representadas con la columna Uso de instancias. Las direcciones representadas con la columna Uso de Azure se comparten entre varios clústeres virtuales, mientras que las representadas con la columna Uso de clústeres virtuales se comparten entre las instancias colocadas en ese clúster virtual.

Para la operación de actualización normalmente es necesario cambiar el tamaño del clúster virtual. En algunos casos, para la operación de actualización será necesario crear clústeres virtuales (para obtener más detalles, consulte el [artículo sobre operaciones de administración](sql-managed-instance-paas-overview.md#management-operations)). En el caso de la creación de un clúster virtual, el número de direcciones adicionales necesario es igual al número de direcciones representado por la columna Uso de clústeres virtuales sumado con las direcciones necesarias para las instancias colocadas en el clúster virtual (columna Uso de instancias).

**Ejemplo 1**: planea tener una instancia administrada de uso general (hardware de Gen4) y una instancia administrada crítica para la empresa (hardware de Gen5). Esto significa que necesita un mínimo de 5 + 1 + 1 * 5 + 6 + 1 * 5 = 22 direcciones IP para poder realizar la implementación. Como los intervalos IP se definen como potencias de 2, la subred requiere un intervalo IP mínimo de 32 (2^5) para esta implementación.<br><br>
Como se ha mencionado antes, en algunas circunstancias, para la operación de actualización será necesario crear un clúster virtual. Esto significa que, por ejemplo, en el caso de una actualización de la instancia administrada crítica para la empresa Gen5 que requiere la creación de un clúster virtual, necesitará disponer de 6 + 5 = 11 direcciones IP adicionales. Como ya usa 22 de las 32 direcciones, no hay ninguna dirección disponible para esta operación. Por tanto, tendrá que reservar la subred con la máscara de subred de /26 (64 direcciones).

**Ejemplo 2**: planea tener tres instancias administradas de uso general (hardware de Gen5) y dos críticas para la empresa (hardware de Gen5) colocadas en la misma subred. Esto significa que necesita 5 + 6 + 3 * 3 + 2 * 5 = 30 direcciones IP. Por tanto, tendrá que reservar la subred con la máscara de subred de /26. La selección de una máscara de subred de /27 haría que el número restante de direcciones fuera 2 (32 – 30), lo que impediría las operaciones de actualización para todas las instancias, ya que se necesitan direcciones adicionales en la subred para realizar el escalado de las instancias.

**Ejemplo 3**: planea tener una instancia administrada de uso general (hardware de Gen5) y realizar la operación de actualización de núcleos virtuales de forma ocasional. Esto significa que necesita 5 + 6 * 1 * 3 + 3 = 17 direcciones IP. Como los intervalos de IP se definen en potencias de 2, necesita el intervalo de IP de 32 (2 ^ 5) direcciones IP. Por lo tanto, tiene que reservar la subred con la máscara de subred de /27.

### <a name="address-requirements-for-update-scenarios"></a>Requisitos de direcciones para escenarios de actualización

Durante la operación de escalado, las instancias necesitan capacidad de IP adicional temporal, que depende del plan de tarifa y la generación del hardware

| **Generación del hardware** | **Plan de tarifa** | **Escenario** | **Direcciones adicionales*** |
| --- | --- | --- | --- |
| Gen4 | GP o BC | Escalado de núcleos virtuales | 5 |
| Gen4 | GP o BC | Escalado de almacenamiento | 5 |
| Gen4 | GP o BC | Cambio de GP a BC o de BC a GP | 5 |
| Gen4 | GP | Cambio a Gen5* | 9 |
| Gen4 | BC | Cambio a Gen5* | 11 |
| Gen5 | GP | Escalado de núcleos virtuales | 3 |
| Gen5 | GP | Escalado de almacenamiento | 0 |
| Gen5 | GP | Cambio a BC | 5 |
| Gen5 | BC | Escalado de núcleos virtuales | 5 |
| Gen5 | BC | Escalado de almacenamiento | 5 |
| Gen5 | BC | Cambio a GP | 3 |

  \* El hardware de Gen4 está en proceso de eliminación gradual y ya no está disponible para implementaciones nuevas. Actualice la generación del hardware de Gen4 a Gen5 para aprovechar las funciones específicas de la generación de hardware de Gen5.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general, vea [¿Qué es Azure SQL Managed Instance?](sql-managed-instance-paas-overview.md)
- Más información sobre la [arquitectura de conectividad para SQL Managed Instance](connectivity-architecture-overview.md).
- Vea cómo [crear la red virtual en la que implementará SQL Managed Instance](virtual-network-subnet-create-arm-template.md).
- Para incidencias de DNS, vea [Configuración de un DNS personalizado](custom-dns-configure.md).
