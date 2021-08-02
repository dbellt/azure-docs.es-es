---
title: Apéndice de la guía para la migración de MySQL en el entorno local a Azure Database for MySQL
description: Descargue la documentación adicional que hemos creado para esta Guía de migración y conozca los pasos de configuración.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 06/11/2021
ms.openlocfilehash: feb02ded8e597f38c941b96eb6027061180a0177
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082999"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-appendix"></a>Apéndice de la guía para la migración de MySQL en el entorno local a Azure Database for MySQL

## <a name="prerequisites"></a>Prerrequisitos

[Resumen](14-summary.md)

## <a name="overview"></a>Información general

En este artículo se indica cómo implementar una aplicación de ejemplo con una guía de migración completa de MySQL y cómo revisar los parámetros de servidor disponibles.

## <a name="environment-setup"></a>Configuración del entorno

[Descargue la documentación adicional](https://github.com/Azure/azure-mysql/blob/master/MigrationGuide/MySQL%20Migration%20Guide_v1.1%20Appendix%20A.pdf) que hemos creado para esta guía de migración y aprenda a configurar un entorno para realizar los pasos de migración de la guía para la [aplicación de demostración de conferencias](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide/sample-app) de ejemplo.

## <a name="azure-resource-manager-arm-templates"></a>Plantillas de Azure Resource Manager (ARM)

### <a name="secure"></a>Seguridad

La plantilla de ARM implementa todos los recursos con puntos de conexión privados. La plantilla de ARM elimina de forma eficaz cualquier acceso a los servicios PaaS desde Internet.

[Plantilla de ARM segura](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide/arm-templates/ExampleWithMigration)

### <a name="non-secure"></a>No segura

Esta plantilla de ARM implementará recursos mediante la implementación estándar, en la que todos los recursos están disponibles desde Internet.

[Plantilla de ARM no segura](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide/arm-templates/ExampleWithMigrationSecure)

## <a name="default-server-parameters-mysql-55-and-azure-database-for-mysql"></a>Parámetros de servidor predeterminados de MySQL 5.5 y Azure Database for MySQL

Puede encontrar la [lista completa de parámetros de servidor predeterminados de MySQL 5.5 y Azure Database for MySQL](https://github.com/Azure/azure-mysql/blob/master/MigrationGuide/MySQL%20Migration%20Guide_v1.1%20Appendix%20C.pdf) en nuestro repositorio de GitHub.