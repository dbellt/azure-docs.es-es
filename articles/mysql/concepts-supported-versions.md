---
title: Versiones admitidas de Azure Database for MySQL
description: Obtenga información sobre las versiones del servidor de MySQL que se admiten en el servicio Azure Database for MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 6/3/2020
ms.openlocfilehash: 77b43561ea0c1fdf9ff7a8d0fcce7fa7b9a2e0fa
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108129394"
---
# <a name="supported-azure-database-for-mysql-server-versions"></a>Versiones admitidas de servidores de Azure Database for MySQL

Azure Database for MySQL se ha desarrollado a partir de [MySQL Community Edition](https://www.mysql.com/products/community/), con el motor de almacenamiento InnoDB. El servicio es compatible con todas las versiones principales actuales admitidas por la comunidad: MySQL 5.6, 5.7 y 8.0. MySQL usa el esquema de nomenclatura X.Y.Z, donde X es la versión principal, Y es la secundaria y Z es la versión de corrección de errores. Para más información sobre el esquema, consulte la [documentación de MySQL](https://dev.mysql.com/doc/refman/5.7/en/which-version.html).



## <a name="connect-to-a-gateway-node-that-is-running-a-specific-mysql-version"></a>Conexión a un nodo de puerta de enlace que ejecuta una versión específica de MySQL

En la opción de implementación de un solo servidor, se usa una puerta de enlace para redirigir las conexiones a las instancias de servidor. Una vez establecida la conexión, el cliente de MySQL muestra la versión de MySQL establecida en la puerta de enlace, no la versión real que se ejecuta en la instancia del servidor MySQL. Para determinar la versión de la instancia del servidor MySQL, use el comando `SELECT VERSION();` en el símbolo del sistema de MySQL. Revise la [arquitectura de conectividad](./concepts-connectivity-architecture.md#connectivity-architecture) para aprender sobre las puertas de enlace en la arquitectura de servicio de Azure Database for MySQL.

Como Azure Database for MySQL admite las versiones principales v5.6, v5.7 y v8.0, el puerto predeterminado 3306 para conectarse a dicho servicio ejecuta la versión 5.6 del cliente de MySQL (mínimo común denominador) para permitir conexiones a servidores de las tres versiones principales admitidas. Sin embargo, si la aplicación tiene el requisito de conectarse a una versión principal específica, por ejemplo v5.7 o v8.0, puede hacerlo cambiando el puerto en la cadena de conexión del servidor.

En el servicio Azure Database for MySQL, los nodos de puerta de enlace escuchan en el puerto 3308 en el caso de los clientes v5.7 y en el puerto 3309 en los clientes v8.0. En otras palabras, si quiere conectar con el cliente de puerta de enlace v5.7, debe usar el nombre de servidor completo y el puerto 3308 para conectarse a su servidor desde la aplicación cliente. Del mismo modo, si quiere conectarse al cliente de puerta de enlace v8.0, puede usar el nombre de servidor completo y el puerto 3309 para conectarse al servidor. Para mayor claridad, consulte el ejemplo siguiente.

:::image type="content" source="./media/concepts-supported-versions/concepts-supported-versions-gateway.png" alt-text="Ejemplo de conexión a través de diferentes versiones de MySQL de puerta de enlace":::

> [!NOTE]
> La conexión a Azure Database for MySQL a través de los puertos 3308 y 3309 solo se admite para la conectividad pública; Private Link y los puntos de conexión de servicio de red virtual solo se pueden usar con el puerto 3306.

## <a name="azure-database-for-mysql-currently-supports-the-following-major-and-minor-versions-of-mysql"></a>Actualmente, Azure Database for MySQL admite las siguientes versiones principales y secundarias de MySQL:


| Versión | [Servidor único](overview.md) <br/> Versión secundaria actual |[Servidor flexible (versión preliminar)](./flexible-server/overview.md) <br/> Versión secundaria actual  |
|:-------------------|:-------------------------------------------|:---------------------------------------------|
|MySQL versión 5.6 |  [5.6.47](https://dev.mysql.com/doc/relnotes/mysql/5.6/en/news-5-6-47.html) (retirada) | No compatible|
|MySQL versión 5.7 | [5.7.29](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-29.html) | [5.7.29](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-29.html)|
|MySQL versión 8.0 | [8.0.15](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-15.html) | [8.0.21](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-21.html)|

Lea la directiva de compatibilidad de versiones para las versiones retiradas en la [documentación de la directiva de compatibilidad de versiones.](concepts-version-policy.md#retired-mysql-engine-versions-not-supported-in-azure-database-for-mysql)

## <a name="managing-updates-and-upgrades"></a>Administración de actualizaciones
El servicio administra automáticamente la aplicación de partes para las actualizaciones de las versiones de corrección de errores. Por ejemplo, 5.7.20 a 5.7.21.  

El servicio admite actualmente la actualización de la versión principal para las actualizaciones de MySQL versión 5.6 a 5.7. Para obtener más información, consulte [cómo realizar actualizaciones de versión principal](how-to-major-version-upgrade.md). Si quiere actualizar de la versión 5.7 a la 8.0, se recomienda que realice un [volcado y restauración de memoria](./concepts-migrate-dump-restore.md) en un servidor que se haya creado con la nueva versión del motor.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre la directiva de control de versiones de Azure Database for MySQL, consulte [este documento](concepts-version-policy.md).
- Para obtener información sobre las cuotas y limitaciones aplicables a recursos específicos en función de su **nivel de servicio**, consulte [Niveles de servicio](./concepts-pricing-tiers.md).