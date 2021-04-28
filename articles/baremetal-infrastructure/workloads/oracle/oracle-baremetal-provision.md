---
title: Aprovisionamiento de BareMetal para Oracle
description: Obtenga información sobre el aprovisionamiento de BareMetal Infrastructure para Oracle.
ms.topic: reference
ms.subservice: workloads
ms.date: 04/14/2021
ms.openlocfilehash: 45ca2bf8bfc61c3820b4b14daa998e2e82e972fa
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107558791"
---
# <a name="provision-baremetal-for-oracle"></a>Aprovisionamiento de BareMetal para Oracle

En este artículo veremos cómo aprovisionar las instancias de BareMetal Infrastructure para cargas de trabajo de Oracle. 

El primer paso para aprovisionar las instancias de BareMetal es trabajar con Microsoft CSA. Le ayudarán en función de sus necesidades de carga de trabajo específicas y de la arquitectura que se vaya a implementar, ya sea una instancia única, RAC de un nodo o RAC. Para más información sobre estas topologías, consulte [Arquitectura BareMetal Infrastructure para Oracle](oracle-baremetal-architecture.md).

## <a name="prerequisites"></a>Requisitos previos

> [!div class="checklist"]
> * Una suscripción de Azure activa
> * Contrato de soporte técnico Premier de Microsoft
> * Licencias para Red Hat Enterprise Linux 7.6
> * Contrato de soporte técnico de Oracle 
> * Licencias y componentes de instalación de software para Oracle
> * ExpressRoute conectado **localmente a Azure** (**opcionalmente**, configure Global Reach de ExpressRoute para la conectividad directa desde el entorno local a Oracle Database)   
> * Virtual network
> * Creación de puerta de enlace
> * Máquinas virtuales (VM) en la red virtual (hosts de salto)

## <a name="information-to-provide-microsoft-operations"></a>Información para proporcionar a Microsoft Operations

Debe especificar la siguiente información a CSA:

1. Espacio de direcciones de red virtual. Este intervalo debe ser una subred /24; por ejemplo, 10.11.0.0/24.
2. Intervalo P2P. Este intervalo debe ser una subred /29; por ejemplo, 10.12.0.0/29.
3. Grupo de direcciones IP del servidor. El intervalo recomendado es /24; por ejemplo, 10.13.0.0/24.
4. Dirección IP del servidor. Elija una dirección IP del grupo de direcciones IP del servidor.

    > [!Note] 
    > Las primeras treinta direcciones IP están reservadas para la configuración de la infraestructura de Microsoft. Por lo tanto, en este ejemplo, la primera dirección IP disponible para una hoja sería 10.13.0.30.

5. Región de Azure necesaria; por ejemplo, Oeste de EE. UU. 2.
6. SKU de BareMetal Infrastructure necesaria; por ejemplo, S192 (dos máquinas).

## <a name="storage-requirements"></a>Requisitos de almacenamiento

Trabaje con el representante de CSA para satisfacer sus necesidades de almacenamiento durante la solicitud de aprovisionamiento, incluidas las necesidades de almacenamiento esperadas en función del crecimiento futuro. El almacenamiento se agrega en incrementos de 1 TB.

En el caso de los volúmenes, seguimos el [estándar de arquitectura flexible óptima (OFA) de Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/19/ladbi/about-the-optimal-flexible-architecture-standard.html#GUID-6619CDB7-9667-426E-8471-5A996707D093), con el nivel Básico y la configuración Enterprise. Si tiene requisitos de almacenamiento personalizados distintos del "tamaño de camiseta" estándar, realice la solicitud personalizada mediante el CSA.

| Configuración básica (POC/Pruebas) | Descripción | Pequeña | Media | grande |
| --- | --- | --- | --- | --- |
| /u01 | Archivos binarios de Oracle | 500 GB | 500 GB | 500 GB |
| /u02 | Lectura intensiva/administración | 500 GB | 1 TB | 5 TB |
| /u03 | Escritura intensiva/registros | 500 GB | 1 TB | 5 TB |
| /u09 | Backup | 5 TB | 10 TB | 15 TB |

| Configuración de la empresa | Descripción | Pequeña | Media | grande | Extragrande |
| --- | --- | --- | --- | --- | --- |
| /u01 | Archivos binarios de Oracle | 500 GB | 500 GB | 500 GB | 500 GB |
| /u02 | Administración | 100 GB | 100 GB | 100 GB | 100 GB |
| De /u10 a /u59 | Lectura intensiva | 500 GB | 5 TB | 10 TB | 20 TB |
| De /u60 a /u89 | Escritura intensiva | 500 GB | 5 TB | 10 TB | 20 TB |
| De /u90 a /u91 | Registros de rehacer | 500 GB | 500 GB | 1 TB | 1 TB |
| /u95 | Archivar | 10 TB | 10 TB | 20 TB | 20 TB |
| /u98 | Backup | 25 TB | 25 TB | 50 TB | 50 TB |

## <a name="next-step"></a>Paso siguiente

Más información sobre BareMetal Infrastructure para Oracle.

> [!div class="nextstepaction"]
> [¿Qué es BareMetal Infrastructure en Azure?](../../concepts-baremetal-infrastructure-overview.md)
