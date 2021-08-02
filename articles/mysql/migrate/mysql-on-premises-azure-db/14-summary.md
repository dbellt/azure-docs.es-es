---
title: Resumen de la guía para la migración de MySQL en el entorno local a Azure Database for MySQL
description: En este documento se abordan varios temas relacionados con la migración de una aplicación de MySQL local a Azure Database for MySQL.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 06/11/2021
ms.openlocfilehash: 06885636d299fa406550f9bd332724e2e01d775f
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112083014"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-summary"></a>Resumen de la guía para la migración de MySQL en el entorno local a Azure Database for MySQL

## <a name="prerequisites"></a>Prerrequisitos

[Seguridad](13-security.md)

## <a name="overview"></a>Información general

En este documento se abordan varios temas relacionados con la migración de una aplicación de MySQL local a Azure Database for MySQL. Se aborda cómo comenzar y evaluar el proyecto hasta la aplicación.

El equipo de migración debe revisar detenidamente los temas, ya que las elecciones realizadas pueden tener efectos en la escala de tiempo del proyecto. El costo total de propiedad es importante dadas las numerosas características preparadas para la empresa que se proporcionan.

El enfoque del proyecto de migración es importante. El equipo debe evaluar la complejidad de la aplicación y la base de datos para determinar la cantidad de tiempo de conversión. Las herramientas de conversión facilitan la transición, pero siempre se requiere un elemento de revisión y actualizaciones manuales. Es importante crear scripts de tareas previas a la migración y pruebas posteriores a la migración.

La arquitectura y el diseño de la aplicación pueden proporcionar indicadores sólidos sobre el nivel de esfuerzo necesario. Por ejemplo, las aplicaciones que utilizan marcos ORM pueden ser excelentes candidatas, especialmente si la lógica de negocios está contenida en la aplicación en lugar de objetos de base de datos.

Al final, existen varias herramientas en el marketplace que pueden ser gratuitas o comercializarse. En este documento se tratan los pasos necesarios si el equipo planea una migración de base de datos mediante una de las opciones de herramientas de código abierto más populares. Independientemente de la ruta de acceso elegida, Microsoft y la comunidad de MySQL tienen las herramientas y la experiencia necesarias para que la migración de la base de datos sea correcta.

## <a name="questions-and-feedback"></a>Preguntas y comentarios

Para cualquier pregunta o sugerencia sobre el uso de Azure Database for MySQL, envíe un correo electrónico al equipo de Azure Database for MySQL (AskAzureDBforMySQL@service.microsoft.com). 

> [!Important]
> La dirección del equipo de Azure Database for MySQL es solo para preguntas generales y no para incidencias de soporte técnico.

Además, tenga en cuenta estos puntos de contacto según corresponda:

  - Para ponerse en contacto con el servicio de Soporte técnico de Azure o solucionar cualquier problema de su cuenta, [presente una incidencia desde Azure Portal. ](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)

  - Para proporcionar comentarios o solicitar nuevas características, cree una entrada mediante [UserVoice.](https://feedback.azure.com/forums/597982-azure-database-for-mysql)
fwill
## <a name="find-a-partner-to-help-migrate"></a>Búsqueda de un asociado para ayudarle en la migración

Esta guía puede ser abrumadora, pero no se preocupe. Hay muchos expertos en la comunidad con un historial de migración probado. [Busque un asociado de Microsoft](https://www.microsoft.com/solution-providers/home) o [Microsoft MVP](https://mvp.microsoft.com/MvpSearch) para que le ayude a encontrar la estrategia de migración más adecuada. No está solo.

También puede examinar los foros técnicos y los grupos sociales para información más detallada del mundo real:

  - [Foro de Microsoft Community ](/answers/topics/azure-database-mysql.html)

  - [Comunidad técnica de Azure Database for MySQL ](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/bg-p/ADforMySQL)

  - [MySQL de StackOverflow para Azure](https://stackoverflow.com/questions/tagged/azure-database-mysql)

  - [Grupo de Azure de Facebook ](https://www.facebook.com/groups/MsftAzure)

  - [Grupo de Azure de LinkedIn ](https://www.linkedin.com/groups/2733961/)

  - [Grupo de desarrolladores de Azure de LinkedIn ](https://www.linkedin.com/groups/1731317/)

## <a name="next-steps"></a>Pasos siguientes

Para implementar una aplicación de ejemplo con una guía de migración completa de MySQL y revisar los parámetros de servidor disponibles, visite el [apéndice](15-appendix.md).