---
title: 'PgBouncer - Azure Database for PostgreSQL: servidor flexible'
description: En este artículo se proporciona información general sobre la extensión PgBouncer integrada.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: conceptual
ms.date: 04/20/2021
ms.openlocfilehash: 92068911c2df95a835de45ea6bb0ba786baf596f
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107989487"
---
# <a name="pgbouncer-in-azure-database-for-postgresql---flexible-server"></a>PgBouncer en Azure Database for PostgreSQL: servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible en versión preliminar

Azure Database for PostgreSQL: servidor flexible ofrece [PgBouncer](https://github.com/pgbouncer/pgbouncer) como una solución de agrupación de conexiones integrada. Se trata de un servicio opcional que se puede habilitar en cada servidor de base de datos y es compatible con el acceso público y privado. PgBouncer se ejecuta en la misma máquina virtual que el servidor de bases de datos de Postgres. Postgres usa un modelo basado en procesos para las conexiones, lo que hace que sea costoso mantener muchas conexiones inactivas. Así, el propio Postgres se encuentra con limitaciones de recursos una vez que el servidor ejecuta más de unos pocos miles de conexiones. La principal ventaja de PgBouncer es que mejora las conexiones inactivas y las conexiones de corta duración en el servidor de bases de datos.

PgBouncer usa un modelo más ligero que usa E/S asincrónica y solo recurre a conexiones Postgres reales cuando es necesario, es decir, cuando se encuentra dentro de una transacción abierta o cuando una consulta está activa. Este modelo puede admitir miles de conexiones más fácilmente con una sobrecarga baja y permite escalar hasta 10 000 conexiones con una sobrecarga baja.

Cuando se habilita, PgBouncer se ejecuta en el puerto 6432 del servidor de bases de datos. Puede cambiar la configuración de conexión de la base de datos de la aplicación para usar el mismo nombre de host, pero cambie el puerto a 6432 para empezar a usar PgBouncer y beneficiarse de la mejora en el escalado de conexiones inactivas.

> [!Note]
> PgBouncer solo se admite en los niveles de proceso De uso general y Con optimización para memoria.

## <a name="enabling-and-configuring-pgbouncer"></a>Habilitación y configuración de protocolos

Para habilitar PgBouncer, puede ir a la hoja "Parámetros del servidor" en Azure Portal, buscar "PgBouncer" y cambiar el parámetro pgbouncer.enabled a "true" para que PgBouncer se habilite. No es necesario reiniciar el servidor. Sin embargo, para establecer otros parámetros de PgBouncer, consulte la sección de limitaciones.

Puede configurar PgBouncer con estos parámetros:

| Nombre de parámetro             | Descripción | Default | 
|----------------------|--------|-------------|
| pgbouncer.default_pool_size | Establezca este valor de parámetro en el número de conexiones por par de usuario/base de datos.      | 50       | 
| pgBouncer.max_client_conn | Establezca este valor de parámetro en el mayor número de conexiones de cliente a PgBouncer que desea admitir.      | 5000     | 
| pgBouncer.pool_mode | Establezca este valor de parámetro en TRANSACTION para la agrupación de transacciones (que es la configuración recomendada para la mayoría de las cargas de trabajo).      | TRANSACTION     |
| pgBouncer.min_pool_size | Agregue más conexiones de servidor al grupo si está por debajo de este número.    |   0 (Deshabilitado)   |
| pgBouncer.stats_users | Opcional. Establezca este valor de parámetro en el nombre de un usuario existente para poder iniciar sesión en la base de datos de estadísticas de PgBouncer especial (denominada "PgBouncer").    |      |

> [!Note] 
> Azure administrará la actualización de PgBouncer.

## <a name="switching-your-application-to-use-pgbouncer"></a>Cambio de la aplicación para usar PgBouncer

Para empezar a usar PgBouncer, siga estos pasos:
1. Conéctese al servidor de bases de datos, pero use el **puerto 6432** en lugar del puerto normal 5432; compruebe que esta conexión funciona.
```azurecli-interactive
psql "host=myPgServer.postgres.database.azure.com port=6432 dbname=postgres user=myUser password=myPassword sslmode=require"
```
2. Pruebe la aplicación en un entorno de control de calidad con PgBouncer para asegurarse de que no tiene ningún problema de compatibilidad. El proyecto PgBouncer proporciona una matriz de compatibilidad, y se recomienda usar la **agrupación de transacciones** para la mayoría de los usuarios: https://www.PgBouncer.org/features.html#sql-feature-map-for-pooling-modes.
3. Cambie la aplicación de producción para conectarse al puerto **6432** en lugar de a **5432** y supervise los errores del lado de la aplicación que puedan apuntar a cualquier problema de compatibilidad.

> [!Note] 
> Incluso si había habilitado PgBouncer, todavía puede conectarse al servidor de bases de datos directamente a través del puerto 5432 con el mismo nombre de host.

## <a name="pgbouncer-in-zone-redundant-high-availability"></a>PgBouncer en alta disponibilidad con redundancia de zona

En los servidores configurados de alta disponibilidad con redundancia de zona, el servidor principal ejecuta PgBouncer. Puede conectarse a la instancia de PgBouncer del servidor principal a través del puerto 6432. Después de una conmutación por error, PgBouncer se reinicia en el modo de espera recién promocionado, que es el nuevo servidor principal. Por lo tanto, la cadena de conexión de la aplicación sigue siendo la misma después de la conmutación por error. 

## <a name="using-pgbouncer-with-other-connection-pools"></a>Uso de PgBouncer con otros grupos de conexiones

En algunos casos, es posible que ya tenga un grupo de conexiones del lado de la aplicación o que PgBouncer se haya configurado en el lado de la aplicación, como un sidecar de AKS. En estos casos, todavía puede ser útil utilizar la instancia de PgBouncer integrada, ya que proporciona ventajas de escalado de conexiones inactivas.

El uso de un grupo del lado de la aplicación junto con PgBouncer en el servidor de bases de datos puede ser beneficioso. En este caso, el grupo del lado de la aplicación aporta la ventaja de una latencia de conexión inicial reducida (ya que el recorrido de ida y vuelta inicial para inicializar la conexión es mucho más rápido) y la instancia de pgBouncer del lado de la base de datos proporciona el escalado de conexiones inactivas.

## <a name="limitations"></a>Limitaciones
 
* PgBouncer no se admite actualmente con el nivel de proceso de servidor Flexible. 
* Si cambia el nivel de proceso de De uso general o Con optimización para memoria al nivel Flexible, perderá la funcionalidad PgBouncer.
* Cada vez que se reinicia el servidor durante las operaciones de escalado, conmutación por error de alta disponibilidad o reinicio, PgBouncer también se reinicia junto con la máquina virtual del servidor. Por lo tanto, las conexiones existentes deben volver a establecerse.
* Debido a un problema conocido, el portal no muestra todos los parámetros de PgBouncer. Una vez que habilite PgBouncer y guarde el parámetro, tendrá que salir de la pantalla Parámetros (por ejemplo, haga clic en Información general) y, a continuación, volver a la página Parámetros. 
  
## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre los [conceptos de redes](./concepts-networking.md).
- [Información general](./overview.md) sobre el servidor flexible
