---
title: Caso de uso representativo de la guía para la migración de MySQL en el entorno local a Azure Database for MySQL
description: El siguiente caso de uso se basa en un escenario de cliente real de una empresa que migró su carga de trabajo de MySQL a Azure Database for MySQL.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 06/11/2021
ms.openlocfilehash: 9d1a6b4d5e4fa9f3c05219f40b2852eed44aaa37
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082854"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-representative-use-case"></a>Caso de uso representativo de la guía para la migración de MySQL en el entorno local a Azure Database for MySQL

## <a name="prerequisites"></a>Prerrequisitos

[Introducción](01-mysql-migration-guide-intro.md)
## <a name="overview"></a>Información general

El siguiente caso de uso se basa en un escenario de cliente real de una empresa que migró su carga de trabajo de MySQL a Azure Database for MySQL.

La empresa World-Wide Importers (WWI) es un fabricante con sede en San Francisco (California) y distribuidor mayorista de artículos de novedades. Inicio sus operaciones en 2002 y desarrolló un modelo eficaz de negocio a negocio (B2B), en el que vende los artículos que produce directamente a clientes minoristas de todo Estados Unidos. Entre sus clientes se incluyen tiendas especializadas, supermercados, tiendas de informática, tiendas de interés turístico y algunos particulares. Este modelo B2B permite ofrecer un sistema de distribución simplificado para sus productos, de modo que la empresa puede reducir los costos y proporcionar precios más competitivos en sus artículos de fabricación. También vende productos a otros mayoristas a través de una red de agentes que promueven sus productos en nombre de WWI.

Antes de iniciarse en nuevas áreas, WWI quiere asegurarse de que su infraestructura de TI puede controlar el crecimiento esperado. Actualmente, WWI hospeda toda su infraestructura de TI local en su sede corporativa y cree que mover estos recursos a la nube permite un crecimiento futuro. Como resultado, ha encargado a su director de sistemas de información que supervise la migración de su portal de clientes y las cargas de trabajo de datos asociadas a la nube.

A WWI le gustaría seguir aprovechando las numerosas funcionalidades avanzadas disponibles en la nube y está interesado en migrar sus bases de datos y cargas de trabajo asociadas a Azure. Quiere hacerlo rápidamente y sin tener que realizar ningún cambio en sus aplicaciones o bases de datos. Inicialmente, planea migrar su aplicación web del portal de clientes basado en Java y las bases de datos y cargas de trabajo de MySQL asociadas a la nube.

### <a name="migration-goals"></a>Objetivos de la migración

Los objetivos principales para migrar sus bases de datos y las cargas de trabajo de SQL asociadas a la nube incluyen los siguientes:

  - Mejorar su nivel de seguridad general con datos en reposo y en tránsito.

  - Mejorar las funcionalidades de alta disponibilidad y recuperación ante desastres (HADR).

  - Posicionar a la organización para que use funcionalidades y tecnologías nativas de la nube, como la restauración a un momento dado.

  - Aprovechar las características administrativas y de optimización del rendimiento de Azure Database for MySQL.

  - Crear una plataforma escalable que pueda usar para expandir su empresa en más regiones geográficas.

  - Permitir el cumplimiento mejorado de los distintos requisitos legales en los que se almacena la información personal.

WWI usó [Cloud Adoption Framework (CAF)](/azure/cloud-adoption-framework/) para informar a su equipo sobre las siguientes directrices de procedimientos recomendados para la migración a la nube. A continuación, con CAF como guía de migración de nivel superior, WWI personalizó su migración en tres fases principales. Por último, definió actividades que había que abordar dentro de cada fase para garantizar una migración correcta a la nube mediante lift-and-shift.

Estas fases incluyen:

| Fase | Nombre | Actividades |
|-------|------|------------|
| 1 | Antes de la migración  | Evaluación, planeamiento, evaluación de métodos de migración, implicaciones de la aplicación, planes de pruebas y base de referencia de rendimiento |
| 2 | Migración      | Ejecutar la migración y ejecutar planes de pruebas                                                                          |
| 3 | Después de la migración | Continuidad empresarial, recuperación ante desastres, administración, seguridad, optimización del rendimiento y modernización de la plataforma |

WWI tiene varias instancias de MySQL que se ejecutan con distintas versiones que van de 5.5 a 5.7. Le gustaría mover sus instancias a la versión más reciente lo antes posible, pero quiere asegurarse de que sus aplicaciones pueden seguir funcionando si se mueven a las versiones más recientes. Le parece bien pasar a la misma versión en la nube y realizar la actualización posteriormente, pero prefiere esa ruta si puede realizar dos tareas a la vez.

También le gustaría asegurarse de que sus cargas de trabajo de datos son seguras y están disponibles en varias regiones geográficas si se produce un error, así como ver las opciones de configuración disponibles.

WWI quiere empezar con una aplicación sencilla para la primera migración y, a continuación, pasar a aplicaciones más críticas para la empresa en una fase posterior. Esto proporciona al equipo el conocimiento y la experiencia que necesita para preparar y planear esas migraciones futuras.  

> [!div class="nextstepaction"]
> [Valoración](./03-assessment.md)