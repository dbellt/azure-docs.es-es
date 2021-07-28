---
title: Compatibilidad de las versiones de Azure Spark
description: Versiones admitidas de Spark, Scala, Python, .NET
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: reference
ms.subservice: spark
ms.date: 05/26/2021
ms.author: midesa
ms.openlocfilehash: c312aad4cd3cea5af3c074c7e32498d9ee30cad5
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110450231"
---
# <a name="azure-synapse-runtimes"></a>Runtimes de Azure Synapse
Los grupos de Apache Spark en Azure Synapse usan runtimes para vincular versiones de componentes esenciales, optimizaciones de Azure Synapse, paquetes y conectores con una versión de Apache Spark específica. Estos runtimes se actualizarán periódicamente para incluir nuevas mejoras, características y revisiones. 

Al crear un grupo de Apache Spark sin servidor, tendrá la opción de seleccionar la versión de Apache Spark correspondiente. En función de esta opción, el grupo estará preinstalado con los componentes y paquetes de runtimes asociados. Estos runtimes tienen las siguientes ventajas:

- Tiempos de inicio de sesión más rápidos
- Compatibilidad probada con versiones de Apache Spark específicas
- Acceso a conectores y paquetes de código abierto populares y compatibles

> [!NOTE]
> - Las actualizaciones de mantenimiento se aplicarán automáticamente a las nuevas sesiones para un grupo de Apache Spark sin servidor determinado. 
> - Debe proba y validar que las aplicaciones se ejecutan correctamente cuando use las nuevas versiones de runtimes.

## <a name="supported-azure-synapse-runtime-releases"></a>Versiones de runtimes de Azure Synapse admitidas 
En la tabla siguiente se enumeran el nombre del runtime, la versión de Apache Spark y la fecha de lanzamiento de las versiones de runtimes de Azure Synapse admitidas.

|  Nombre del runtime  | Fecha de la versión |  Fase de versión |
| ----- | ----- | ----- |
| [Runtime de Azure Synapse para Apache Spark 2.4](./apache-spark-24-runtime.md) | 15 de diciembre de 2020 | Disponibilidad general|
| [Runtime de Azure Synapse para Apache Spark 3.0](./apache-spark-3-runtime.md) | 26 de mayo de 2021 | Vista previa |

## <a name="runtime-release-stages"></a>Fases de la versión del runtime

## <a name="preview-runtimes"></a>Runtimes de versión preliminar
Azure Synapse Analytics proporciona versiones preliminares para ofrecerle la oportunidad de evaluar y compartir comentarios sobre las características antes de que estén disponibles con carácter general (GA). Mientras un runtime está disponible en versión preliminar, se pueden introducir nuevas dependencias y versiones de componentes. Los Acuerdos de Nivel de Servicio no son aplicables a los runtimes de versión preliminar. 

## <a name="generally-available-runtimes"></a>Runtimes disponibles con carácter general
Los runtimes de disponibilidad general (GA) están abiertos a todos los clientes y están listos para su uso en producción. Una vez que un runtime está disponible con carácter general, se pueden realizar la portabilidad con versiones anteriores de correcciones de seguridad y mejoras de estabilidad. Además, solo se introducirán nuevos componentes si no cambian las dependencias subyacentes o las versiones de los componentes. 