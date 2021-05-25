---
title: 'Novedades de Azure Database for MySQL: servidor único'
description: 'Conozca las actualizaciones recientes de Azure Database for MySQL: servidor único, un servicio en la nube de base de datos relacional de Microsoft basado en MySQL Community Edition.'
author: hjtoland3
ms.service: mysql
ms.author: jtoland
ms.custom: mvc
ms.topic: conceptual
ms.date: 05/05/2021
ms.openlocfilehash: 9f0fd2b23fe363b3db353267908b2c466ba70b68
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109488004"
---
# <a name="whats-new-in-azure-database-for-mysql---single-server"></a>Novedades de Azure Database for MySQL: servidor único

Azure Database for MySQL es un servicio en la nube de base de datos relacional de Microsoft. El servicio se basa en el motor de base de datos de [MySQL Community Edition](https://www.mysql.com/products/community/) (disponible con la licencia GPLv2) y admite las versiones 5.6, 5.7 y 8.0. [Azure Database for MySQL: servidor único](https://docs.microsoft.com/azure/mysql/overview#azure-database-for-mysql---single-server) es un modo de implementación que proporciona un servicio de base de datos totalmente administrado con unos requisitos mínimos para las personalizaciones de base de datos. La plataforma de servidor único está diseñada para administrar la mayoría de las funciones de administración de bases de datos, como aplicación de revisiones, copias de seguridad, alta disponibilidad y seguridad, todo ello sin que el usuario tenga que intervenir apenas en la configuración y el control.

En este artículo se resumen las nuevas versiones y características de Azure Database for MySQL: servidor único a partir de enero de 2021. Las listas aparecen en orden cronológico inverso, con las actualizaciones más recientes en primer lugar.

## <a name="february-2021"></a>Febrero de 2021

Esta versión de Azure Database for MySQL: servidor único incluye las siguientes actualizaciones.

- Se han agregado nuevos procedimientos almacenados para admitir el identificador de transacción global (GTID) para la entrada de datos de la versión 5.7 y 8.0 del servidor de almacenamiento a gran escala.
- Se ha actualizado para admitir las versiones de MySQL 5.6.50 y 5.7.32.

## <a name="january-2021"></a>Enero de 2021

Esta versión de Azure Database for MySQL: servidor único incluye las siguientes actualizaciones.

- Se ha habilitado el "restablecimiento de contraseña" para corregir automáticamente el primer permiso de administrador.
- Se ha expuesto el parámetro de servidor `auto_increment_increment/auto_increment_offset` y `session_track_gtids`.
- Se han agregado nuevos procedimientos almacenados para controlar el volcado o la restauración del grupo de búferes innodb.
- Se ha expuesto el parámetro de servidor relacionado con la preparación de innodb para un servidor de almacenamiento a gran escala.

## <a name="contacts"></a>Contactos

Si tiene alguna pregunta o sugerencia con respecto al uso de Azure Database for MySQL, póngase en contacto con el equipo de Azure Database for MySQL ([@Ask Azure DB for MySQL](mailto:AskAzureDBforMySQL@service.microsoft.com)). Esta dirección de correo electrónico no es un alias de soporte técnico.

Además, tenga en cuenta los siguientes puntos de contacto según corresponda:

- Para ponerse en contacto con el servicio de soporte técnico de Azure, [presente una incidencia de soporte técnico en Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
- Para corregir un problema con la cuenta, envíe una [solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) en Azure Portal.
- Para proporcionar comentarios o solicitar nuevas características, cree una entrada mediante [UserVoice](https://feedback.azure.com/forums/597982-azure-database-for-mysql).

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre los [precios de Azure Database for MySQL](https://azure.microsoft.com/pricing/details/mysql/server/).
- Examine la [documentación pública](https://docs.microsoft.com/azure/mysql/single-server/) de Azure Database for MySQL: servidor único.
- Revise los detalles sobre la [solución de errores comunes](https://docs.microsoft.com/azure/mysql/howto-troubleshoot-common-errors).
