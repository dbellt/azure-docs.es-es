---
title: 'Directiva de compatibilidad de versiones - Azure Database for MySQL: servidor único y servidor flexible (versión preliminar)'
description: Aquí se describe la directiva relacionada con las versiones principales y secundarias de MySQL en Azure Database for MySQL
author: sr-msft
ms.author: srranga
ms.service: mysql
ms.topic: conceptual
ms.date: 11/03/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: 0dd856970e2139f816947f1b26589289a823ea9a
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108129358"
---
# <a name="azure-database-for-mysql-version-support-policy"></a>Directiva compatibilidad de versión de Azure Database for MySQL

En esta página se describe la directiva de versión de Azure Database for MySQL y se puede aplicar a los modos de implementación Azure Database for MySQL: servidor único y Azure Database for MySQL: servidor flexible (versión preliminar).

## <a name="supported--mysql-versions"></a>Versiones de MySQL admitidas

Azure Database for MySQL se ha desarrollado a partir de [MySQL Community Edition](https://www.mysql.com/products/community/), con el motor de almacenamiento InnoDB. El servicio es compatible con todas las versiones principales actuales admitidas por la comunidad: MySQL 5.6, 5.7 y 8.0. MySQL usa el esquema de nomenclatura X.Y.Z, donde X es la versión principal, Y es la secundaria y Z es la versión de corrección de errores. Para más información sobre el esquema, consulte la [documentación de MySQL](https://dev.mysql.com/doc/refman/5.7/en/which-version.html).

Actualmente, Azure Database for MySQL admite las siguientes versiones principales y secundarias de MySQL:

| Versión | [Servidor único](overview.md) <br/> Versión secundaria actual |[Servidor flexible (versión preliminar)](./flexible-server/overview.md) <br/> Versión secundaria actual  |
|:-------------------|:-------------------------------------------|:---------------------------------------------|
|MySQL versión 5.6 |  [5.6.47](https://dev.mysql.com/doc/relnotes/mysql/5.6/en/news-5-6-47.html) (retirada) | No compatible|
|MySQL versión 5.7 | [5.7.29](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-29.html) | [5.7.29](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-29.html)|
|MySQL versión 8.0 | [8.0.15](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-15.html) | [8.0.21](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-21.html)|

> [!NOTE]
> En la opción de implementación de un solo servidor, se usa una puerta de enlace para redirigir las conexiones a las instancias de servidor. Una vez establecida la conexión, el cliente de MySQL muestra la versión de MySQL establecida en la puerta de enlace, no la versión real que se ejecuta en la instancia del servidor MySQL. Para determinar la versión de la instancia del servidor MySQL, use el comando `SELECT VERSION();` en el símbolo del sistema de MySQL. Si la aplicación tiene el requisito de conectarse a una versión principal específica, por ejemplo v5.7 o v8.0, puede hacerlo cambiando el puerto en la cadena de conexión del servidor como se explica en nuestra documentación [aquí.](concepts-supported-versions.md#connect-to-a-gateway-node-that-is-running-a-specific-mysql-version)

> [!IMPORTANT]
> MySQL v 5.6 se retira en un solo servidor a partir de febrero 2021. A partir del 1 de septiembre de 2021, no podrá crear nuevos servidores v 5.6 en Azure Database for MySQL opción de implementación de un solo servidor. Sin embargo, podrá realizar recuperaciones a un momento dado, así como crear réplicas de lectura para los servidores existentes.

Lea la directiva de compatibilidad de versiones para las versiones retiradas en la [documentación de la directiva de compatibilidad de versiones.](concepts-version-policy.md#retired-mysql-engine-versions-not-supported-in-azure-database-for-mysql)

## <a name="major-version-support"></a>Compatibilidad con la versión principal
Cada versión principal de MySQL será compatible con Azure Database for MySQL a partir de la fecha en la que Azure comience a admitir la versión y hasta que la comunidad de MySQL retire la versión, tal como se proporciona en la [directiva de versión](https://www.mysql.com/support/eol-notice.html).

## <a name="minor-version-support"></a>Compatibilidad con la versión secundaria
Azure Database for MySQL realiza automáticamente actualizaciones de las versiones secundarias a la versión de MySQL preferida de Azure como parte del mantenimiento periódico. 

## <a name="major-version-retirement-policy"></a>Directiva de retirada de la versión principal
En la tabla siguiente se proporcionan los detalles de la retirada de las versiones principales de MySQL. Las fechas siguen la [directiva de versión de MySQL](https://www.mysql.com/support/eol-notice.html).

| Versión | Novedades | Fecha de inicio del soporte técnico de Azure | Fecha de retirada|
| ----- | ----- | ------ | ----- |
| [MySQL 5.6](https://dev.mysql.com/doc/relnotes/mysql/5.6/en/)| [Características](https://dev.mysql.com/doc/relnotes/mysql/5.6/en/news-5-6-49.html)  | 20 de marzo de 2018 | Febrero de 2021
| [MySQL 5.7](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/) | [Características](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-31.html) | 20 de marzo de 2018 | Octubre de 2023
| [MySQL 8](https://mysqlserverteam.com/whats-new-in-mysql-8-0-generally-available/) | [Características](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-21.html) | 11 de diciembre de 2019 | Abril de 2026


## <a name="retired-mysql-engine-versions-not-supported-in-azure-database-for-mysql"></a>Versiones del motor de MySQL retiradas que no se admiten en Azure Database for MySQL

Después de la fecha de retirada de cada versión de la base de datos de MySQL, si continúa ejecutando la versión retirada, tenga en cuenta las siguientes restricciones:
- Dado que la comunidad no va a publicar más correcciones de errores ni correcciones de seguridad, Azure Database for MySQL no revisará el motor de base de datos retirado en busca de errores o problemas de seguridad, ni tomará medidas de seguridad relacionadas con el motor de base de datos retirado. No obstante, Azure continuará realizando tareas periódicas de mantenimiento y revisión para el host, el sistema operativo, los contenedores y cualquier otro componente relacionado con el servicio.
- Si experimenta algún problema de compatibilidad con la base de datos de MySQL, quizá no le podamos proporcionar soporte técnico. En tales casos, tendrá que actualizar la base de datos para que se le proporcione soporte técnico.
- No podrá crear nuevos servidores de bases de datos para la versión retirada. Sin embargo, podrá realizar recuperaciones a un momento dado, así como crear réplicas de lectura para los servidores existentes.
- Las nuevas funcionalidades de servicio que ha desarrollado Azure Database for MySQL podrían solo estar disponibles para las versiones de servidor de bases de datos admitidas.
- Los acuerdos de nivel de servicio de tiempo de actividad solo se aplicarán a los problemas relacionados con los servicios de Azure Database for MySQL, no a los tiempos de inactividad causados por los errores relacionados con el motor.  
- En el caso de una amenaza grave para el servicio causada por la vulnerabilidad del motor de base de datos MySQL identificada en la versión de base de datos retirada, Azure puede optar por detener el nodo de proceso del servidor de bases de datos para proteger primero el servicio. Se le pedirá que actualice el servidor antes de ponerlo en línea. Durante el proceso de actualización, los datos siempre se protegerán mediante copias de seguridad automáticas realizadas en el servicio que se pueden usar para restaurar la versión anterior si se desea. 



## <a name="next-steps"></a>Pasos siguientes
- Consulte las [versiones admitidas](./concepts-supported-versions.md) en Azure Database for MySQL: servidor único.
- Consulte las [versiones admitidas](flexible-server/concepts-supported-versions.md) en Azure Database for MySQL: servidor flexible (versión preliminar).
- Consulte el artículo sobre [volcado y restauración](./concepts-migrate-dump-restore.md) de MySQL para realizar actualizaciones.