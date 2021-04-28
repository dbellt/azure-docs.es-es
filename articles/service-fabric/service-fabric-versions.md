---
title: Versiones de Azure Service Fabric
description: Obtenga información sobre las versiones de clúster de Azure Service Fabric y las versiones de plataforma que se admiten activamente
ms.topic: troubleshooting
ms.date: 04/12/2021
ms.openlocfilehash: 4e656b0cd051079d4504ee2a55a9bf6993065396
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107515318"
---
# <a name="service-fabric-supported-versions"></a>Versiones admitidas de Service Fabric
En las tablas de este artículo se describen las versiones de Service Fabric y de plataforma que se admiten activamente.

## <a name="windows"></a>Windows

| Entorno de ejecución de Service Fabric |Versión desde la que se puede actualizar directamente|Versión anterior a la que se puede cambiar|Versiones del SDK o paquete de NuGet compatibles|Entornos de ejecución de dotnet admitidos** |Versión del SO. |Finalización del soporte técnico |
| --- | --- | --- | --- | --- | --- | --- |
| 8.0 RTO | 7.1 CU10 | 7.2 | Versión 5.0 o anterior | .NET 5.0 (GA), .NET Core 3.1, .NET Core 2.1, <br>All >=4.5 .NET Full Framework| [Consulte la versión del sistema operativo admitida](#supported-windows-versions-and-support-end-date) | Versión actual |
| 7.2 CU7 | 7.0 CU9 | 7.1 | Versión 4.2 o anterior | .NET 5.0 (compatibilidad con la versión preliminar), .NET Core 3.1, .NET Core 2.1,<br>Todas las versiones iguales o posteriores a 4.5 Net Full Framework | [Consulte la versión del sistema operativo admitida](#supported-windows-versions-and-support-end-date) | 30 de noviembre de 2021 |
| 7.2 CU6 | 7.0 CU4 |7.1 | Versión 4.2 o anterior | .NET 5.0 (compatibilidad con la versión preliminar), .NET Core 3.1, .NET Core 2.1,<br>Todas las versiones iguales o posteriores a 4.5 Net Full Framework | [Consulte la versión del sistema operativo admitida](#supported-windows-versions-and-support-end-date)| 30 de noviembre de 2021 |
| 7.2 RTO-CU5 | 7.0 CU4 | 7.1 |Versión 4.2 o anterior | .NET Core 3.1, .NET Core 2.1,<br>Todas las versiones iguales o posteriores a 4.5 Net Full Framework | [Consulte la versión del sistema operativo admitida](#supported-windows-versions-and-support-end-date)| 30 de noviembre de 2021 |
| 7.1 |7.0 CU3 |N/D | Versión 4.1 o anterior | .NET Core 3.1, .NET Core 2.1,<br>Todas las versiones iguales o posteriores a 4.5 Net Full Framework | [Consulte la versión del sistema operativo admitida](#supported-windows-versions-and-support-end-date) | 31 de julio de 2021 |

** Service Fabric no proporciona un entorno de ejecución de .NET Core. El autor del servicio es responsable de asegurarse de que esté <a href="https://docs.microsoft.com/dotnet/core/deploying/">disponible</a>.

## <a name="supported-windows-versions-and-support-end-date"></a>Versiones de Windows admitidas y fecha de finalización del soporte técnico
El soporte técnico para Service Fabric en un sistema operativo específico finaliza cuando la compatibilidad con la versión del sistema operativo alcanza su fin de ciclo de vida.


### <a name="windows-server"></a>Windows Server

| Versión del SO | Fecha de finalización de soporte técnico de Service Fabric | Vínculo del ciclo de vida del sistema operativo |
|---|---|---|
|Windows Server 2019|9/1/2029|<a href="https://docs.microsoft.com/lifecycle/products/windows-server-2019">Windows Server 2019: ciclo de vida de Microsoft</a>|
|Windows Server 2016 |12/1/2027|<a href="https://docs.microsoft.com/lifecycle/products/windows-server-2016">Windows Server 2016: ciclo de vida de Microsoft</a>|
|Windows Server 2012 R2 |10/10/2023|<a href="https://docs.microsoft.com/lifecycle/products/windows-server-2012-r2">Windows Server 2012 R2: ciclo de vida de Microsoft</a>|
|Versión 20H2 |10/5/2022|<a href="https://docs.microsoft.com/lifecycle/products/windows-server">Windows Server: ciclo de vida de Microsoft</a>|
|Versión 2004 |14/12/2021|<a href="https://docs.microsoft.com/lifecycle/products/windows-server">Windows Server: ciclo de vida de Microsoft</a>|
|Version 1909 |11/5/2021|<a href="https://docs.microsoft.com/lifecycle/products/windows-server">Windows Server: ciclo de vida de Microsoft</a>|

<br>

### <a name="windows-10"></a>Windows 10

| Versión del SO | Fecha de finalización de soporte técnico de Service Fabric | Vínculo del ciclo de vida del sistema operativo |
| --- | --- | --- |
| Windows 10 2019 LTSC | 9/1/2029 | <a href="https://docs.microsoft.com/lifecycle/products/windows-10-2019-ltsc">Windows 10 2019 LTSC: ciclo de vida de Microsoft</a> |
| Versión 20H2 | 9/5/2023 | <a href="https://docs.microsoft.com/lifecycle/products/windows-10-enterprise-and-education">Windows 10 Enterprise y Education: ciclo de vida de Microsoft</a> |
| Versión 2004 | 14/12/2021| <a href="https://docs.microsoft.com/lifecycle/products/windows-10-enterprise-and-education">Windows 10 Enterprise y Education: ciclo de vida de Microsoft</a> |
| Version 1909 | 10/5/2022 | <a href="https://docs.microsoft.com/lifecycle/products/windows-10-enterprise-and-education">Windows 10 Enterprise y Education: ciclo de vida de Microsoft</a> |
| Versión 1809 | 11/5/2021 | <a href="https://docs.microsoft.com/lifecycle/products/windows-10-enterprise-and-education">Windows 10 Enterprise y Education: ciclo de vida de Microsoft</a> |
| Versión 1803 | 11/5/2021 | <a href="https://docs.microsoft.com/lifecycle/products/windows-10-enterprise-and-education">Windows 10 Enterprise y Education: ciclo de vida de Microsoft</a> |

## <a name="linux"></a>Linux

| Entorno de ejecución de Service Fabric | Versión desde la que se puede actualizar directamente |Versión anterior a la que se puede cambiar |Versiones del SDK o paquete de NuGet compatibles | Entornos de ejecución de dotnet admitidos** | Versión del SO | Finalización del soporte técnico |
| --- | --- | --- | --- | --- | --- | --- |
| 8.0 RTO | 7.1 CU8 | 7.2 | Versión 5.0 o anterior | .NET Core 3.1, .NET Core 2.1 | [Consulte la versión del sistema operativo admitida](#supported-linux-versions-and-support-end-date) | Versión actual |
| 7.2 CU7 | 7.0 CU9 | 7.1 | Versión 4.2 o anterior | .NET Core 3.1, .NET Core 2.1 | [Consulte la versión del sistema operativo admitida](#supported-linux-versions-and-support-end-date) | 30 de noviembre de 2021 |
| 7.2 RTO-CU6 | 7.0 CU4 | 7.1 | Versión 4.2 o anterior | .NET Core 3.1, .NET Core 2.1 | [Consulte la versión del sistema operativo admitida](#supported-linux-versions-and-support-end-date) | 30 de noviembre de 2021 |
| 7.1 | 7.0 CU3 | N/D | Versión 4.1 o anterior | .NET Core 3.1, .NET Core 2.1 | [Consulte la versión del sistema operativo admitida](#supported-linux-versions-and-support-end-date) | 31 de julio de 2021 |

** Service Fabric no proporciona un entorno de ejecución de .NET Core y el autor del servicio es responsable de asegurarse de que esté <a href="https://docs.microsoft.com/dotnet/core/deploying/">disponible</a>

## <a name="supported-linux-versions-and-support-end-date"></a>Versiones de Linux admitidas y fecha de finalización del soporte técnico
El soporte técnico para Service Fabric en un sistema operativo específico finaliza cuando la compatibilidad con la versión del sistema operativo alcanza su fin de ciclo de vida.

#### <a name="ubuntu"></a>Ubuntu
| Versión del SO | Fecha de finalización de soporte técnico de Service Fabric| Vínculo del ciclo de vida del sistema operativo |
| --- | --- | --- |
| Ubuntu 18.04 | Abril de 2028 | <a href="https://wiki.ubuntu.com/Releases">Ciclo de vida de Ubuntu</a>|
| Ubuntu 16.04 | Abril de 2024 | <a href="https://wiki.ubuntu.com/Releases">Ciclo de vida de Ubuntu</a>|

<br>

## <a name="service-fabric-version-name-and-number-reference"></a>Nombre y número de referencia de la versión de Service Fabric
En la tabla siguiente se indican los nombres de versión de Service Fabric y sus números de versión correspondientes.

| Nombre de versión | Número de versión de Windows | Número de versión de Linux |
| --- | --- | --- |
| 8.0 RTO | 8.0.514.9590 | 8.0.513.1 | 
| 7.2 CU7 | 7.2.477.9590 | 7.2.476.1 |
| 7.2 CU6 | 7.2.457.9590 | 7.2.456.1 |
| 7.2 CU5 | 7.2.452.9590 | 7.2.454.1 |
| 7.2 CU4 | 7.2.445.9590 | 7.2.447.1 |
| 7.2 CU3 | 7.2.433.9590 | N/D |
| 7.2 CU2 | 7.2.432.9590 | 7.2.431.1 |
| 7.2 RTO | 7.2.413.9590 | N/D |
| 7.1 CU10 | 7.1.510.9590 | N/D |
| 7.1 CU8 | 7.1.503.9590 | 7.1.508.1 |
| 7.1 CU6 | 7.1.459.9590 | 7.1.455.1 |
| 7.1 CU5 | 7.1.458.9590 | 7.1.454.1 |
| 7.1 CU3 | 7.1.456.9590 | 7.1.452.1 |
| 7.1 CU2 | 7.1.428.9590 | 7.1.428.1 |
| 7.1 CU1 | 7.1.417.9590 | 7.1.418.1 |
| 7.1 RTO | 7.1.409.9590 | 7.1.410.1 |
| 7.0 CU9 | 7.0.478.9590 | 7.0.472.1 |
| 7.0 CU6 | 7.0.472.9590 | 7.0.471.1 |
| 7.0 CU4 | 7.0.470.9590 | 7.0.469.1 |
| 7.0 CU3 | 7.0.466.9590 | 7.0.465.1 |
| 7.0 CU2 | 7.0.464.9590 | 7.0.464.1 |
| 7.0 RTO | 7.0.457.9590 | 7.0.457.1 |
| 6.5 CU5 | 6.5.676.9590 | 6.5.467.1 |
| 6.5 CU3 | 6.5.664.9590 | 6.5.466.1 |
| 6.5 CU2 | 6.5.658.9590 | 6.5.460.1 |
| 6.5 CU1 | 6.5.641.9590 | 6.5.454.1 |
| 6.5 RTO | 6.5.639.9590 | 6.5.435.1 |
| 6.4 CU8 | 6.4.670.9590 | No aplicable|
| 6.4 CU7 | 6.4.664.9590 | 6.4.661.1 |
| 6.4 CU6 | 6.4.658.9590 | No aplicable|
| 6.4 CU5 | 6.4.654.9590 | 6.4.649.1 |
| 6.4 CU4 | 6.4.644.9590 | 6.4.639.1 |
| 6.4 CU3 | 6.4.637.9590 | 6.4.634.1 |
| 6.4 CU2 | 6.4.622.9590 | No aplicable|
| 6.4 RTO | 6.4.617.9590 | 6.4.625.1 |
| 6.3 CU1 | 6.3.187.9494 | 6.3.129.1 |
| 6.3 RTO | 6.3.162.9494 | 6.3.119.1 |
| 6.2 CU3 | 6.2.301.9494 | 6.2.199.1 |
| 6.2 CU2 | 6.2.283.9494 | 6.2.194.1 |
| 6.2 CU1 | 6.2.274.9494 | 6.2.191.1 |
| 6.2 RTO | 6.2.269.9494 | 6.2.184.1 |
| 6.1 CU4 | 6.1.480.9494 | 6.1.187.1 |
| 6.1 CU3 | 6.1.472.9494 | No aplicable|
| 6.1 CU2 | 6.1.467.9494 | 6.1.185.1 |
| 6.1 CU1 | 6.1.456.9494 | 6.1.183.1 |
| 6.0 CU2 | 6.0.232.9494 | 6.0.133.1 |
| 6.0 CU1 | 6.0.219.9494 | 6.0.127.1 |
| 6.0 RTO | 6.0.211.9494 | 6.0.120.1 |
| 5.7 CU4 | 5.7.221.9494 | No aplicable|
| 5.7 RTO | 5.7.198.9494 | No aplicable|
| 5.6 CU3 | 5.6.220.9494 | No aplicable|
| 5.6 CU2 | 5.6.210.9494 | No aplicable|
| 5.6 RTO | 5.6.204.9494 | No aplicable|
| 5.5 CU4 | 5.5.232.0 | No aplicable|
| 5.5 CU3 | 5.5.227.0 | No aplicable|
| 5.5 CU2 | 5.5.219.0 | No aplicable|
| 5.5 CU1 | 5.5.216.0    | No aplicable|
| 5.4 CU2 | 5.4.164.9494 | No aplicable|
| 5.3 CU3 | 5.3.311.9590 | No aplicable|
| 5.3 CU2 | 5.3.301.9590 | No aplicable|
| 5.3 CU1 | 5.3.204.9494 | No aplicable|
| 5.3 RTO | 5.3.121.9494 | No aplicable|
