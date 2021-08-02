---
title: 'Guía de migración de MySQL local a Azure Database for MySQL, Apéndice B: Plantillas de ARM'
description: Esta plantilla implementará todos los recursos con puntos de conexión privados.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 05/26/2021
ms.openlocfilehash: 07e689b8458650ca39a0aa949504dd762621bac1
ms.sourcegitcommit: 1b698fb8ceb46e75c2ef9ef8fece697852c0356c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110656907"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-appendix-b-arm-templates"></a>Guía de migración de MySQL local a Azure Database for MySQL, Apéndice B: Plantillas de ARM

### <a name="secure"></a>Seguridad

Esta plantilla implementará todos los recursos con puntos de conexión privados. Esta acción elimina de forma eficaz cualquier acceso a los servicios PaaS desde Internet.

[Plantilla de ARM ](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide/arm-templates/ExampleWithMigration)

### <a name="non-secure"></a>No seguro

Esta plantilla implementará recursos mediante la implementación estándar, donde todos los recursos están disponibles desde Internet.

[Plantilla de ARM ](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide/arm-templates/ExampleWithMigrationSecure)