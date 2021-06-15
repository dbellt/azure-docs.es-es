---
title: Actualizaciones graduales de aplicaciones
description: Aprenda a usar la replicación geográfica en Azure SQL Database para admitir las actualizaciones graduales de la aplicación en la nube.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: sqldbrb=1, devx-track-azurepowershell
ms.devlang: ''
ms.topic: how-to
author: BustosMSFT
ms.author: robustos
ms.reviewer: mathoma
ms.date: 02/13/2019
ms.openlocfilehash: 30b5d1f7e5fd3a052ecce3c28b75fe020b2257c9
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110694263"
---
# <a name="manage-rolling-upgrades-of-cloud-applications-by-using-sql-database-active-geo-replication"></a>Administración de actualizaciones graduales de aplicaciones en la nube mediante la replicación geográfica activa de SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Aprenda a usar la [replicación geográfica activa](auto-failover-group-overview.md) en Azure SQL Database para habilitar las actualizaciones graduales de su aplicación en la nube. Ya que las actualizaciones son operación problemáticas, deben formar parte del diseño y planeamiento de la continuidad empresarial. En este artículo veremos dos métodos diferentes de organizar el proceso de actualización y comentaremos las ventajas y desventajas de cada opción. Para los fines de este artículo, haremos referencia a una aplicación que consta de un sitio web conectado a una base de datos única como su capa de datos. Nuestro objetivo es actualizar la versión 1 (V1) de la aplicación a la versión 2 (V2) sin que se observe ningún efecto importante en la experiencia del usuario.

Al evaluar las opciones de actualización, tenga en cuenta estos factores:

* Efecto sobre la disponibilidad de las aplicaciones durante las actualizaciones, por ejemplo, el tiempo durante el cual las funciones de la aplicación podrían estar limitadas o degradadas.
* Posibilidad de revertir si la actualización produce un error.
* Vulnerabilidad de la aplicación si se produce un error grave no relacionado durante la actualización.
* El costo total en dólares. Este factor incluye la redundancia adicional de la base de datos y los costos incrementales de los componentes temporales usados en el proceso de actualización.

## <a name="upgrade-applications-that-rely-on-database-backups-for-disaster-recovery"></a>Actualización de aplicaciones que se basan en las copias de seguridad de base de datos para la recuperación ante desastres

Si la aplicación se basa en copias de seguridad de base de datos automáticas y emplea la restauración geográfica para la recuperación ante desastres, se implementa en una única región de Azure. Para reducir las interrupciones del usuario final, cree un entorno de ensayo en dicha región con todos los componentes de la aplicación que intervienen en la actualización. En el primer diagrama se ilustra el entorno operativo antes del proceso de actualización. El punto de conexión `contoso.azurewebsites.net` representa un entorno de producción de la aplicación web. Para poder revertir la actualización, debe crear un entorno de ensayo con una copia totalmente sincronizada de la base de datos. Siga estos pasos para crear un entorno de ensayo para la actualización:

1. Cree una base de datos secundaria en la misma región de Azure. Supervise esta base de datos secundaria para ver si se completa el proceso de inicialización (1).
2. Cree un nuevo entorno para la aplicación web y llámelo "Ensayo". Se registrará en Azure DNS con la dirección URL `contoso-staging.azurewebsites.net` (2).

> [!NOTE]
> Estos pasos de preparación no afectarán el entorno de producción, que puede funcionar en modo de acceso completo.

![Diagrama que muestra la configuración de la replicación geográfica de SQL Database para la recuperación ante desastres en la nube.](./media/manage-application-rolling-upgrade/option1-1.png)

Una vez completados los pasos preparatorios, la aplicación está preparada para la actualización real. En el siguiente diagrama se ilustran los pasos que intervienen en el proceso de actualización:

1. Establecimiento de la base de datos principal en modo de solo lectura (3). Este modo garantiza que el entorno de producción de la aplicación web (V1) seguirá siendo de solo lectura durante la actualización, con lo que se impide la divergencia de datos entre las instancias de base de datos V1 y V2.
2. Desconecte la base de datos secundaria con el modo de finalización planeada (4). Esta acción crea una copia independiente totalmente sincronizada de la base de datos principal. Esta base de datos se actualizará.
3. Cambie la base de datos secundaria al modo de lectura y escritura y ejecute el script de actualización (5).

![Diagrama que muestra la configuración de la replicación geográfica de SQL Database para la recuperación ante desastres en la nube que ejecuta el script de actualización.](./media/manage-application-rolling-upgrade/option1-2.png)

Si la actualización finaliza correctamente, ahora está listo para cambiar a los usuarios a la copia actualizada de la aplicación, que se convierte en un entorno de producción. El cambio implica unos cuantos pasos más, como se ilustra en el diagrama siguiente:

1. Active una operación de intercambio entre los espacios de ensayo y producción de la aplicación web (6). Esta operación cambia las direcciones URL de los dos entornos. Ahora `contoso.azurewebsites.net` apunta a la versión V2 del sitio web y la base de datos (entorno de producción). 
2. Si ya no necesita la versión V1, que se convirtió en una copia de ensayo después del intercambio, puede retirar el entorno de ensayo (7).

![Configuración de la replicación geográfica de SQL Database para la recuperación ante desastres en la nube.](./media/manage-application-rolling-upgrade/option1-3.png)

Si el proceso de actualización es incorrecto, por ejemplo, debido a un error en el script de actualización, el entorno de ensayo debe considerarse comprometido. Para revertir la aplicación al estado previo a la actualización, revierta la aplicación en el entorno de producción a acceso completo. En el diagrama siguiente se muestran los pasos de reversión:

1. Establezca la copia de la base de datos en modo de lectura y escritura (8). Esta acción restaura la funcionalidad de V1 completa de la copia de producción.
2. Realice el análisis de la causa principal y retire el entorno de ensayo (9).

En este punto, la aplicación es totalmente funcional y se pueden repetir los pasos de actualización.

> [!NOTE]
> La reversión no requiere cambios de DNS, ya que no ha realizado aún ninguna operación de intercambio.

![Diagrama que muestra la configuración de la replicación geográfica de SQL Database para la recuperación ante desastres en la nube con el entorno de ensayo retirado.](./media/manage-application-rolling-upgrade/option1-4.png)

La principal ventaja de esta opción es que se puede actualizar una aplicación en una sola región mediante un conjunto de pasos sencillos. El costo de la actualización es relativamente bajo. 

La principal desventaja es que, si se produce un error grave durante la actualización, la recuperación al estado previo a la actualización implica que hay que volver a implementar la aplicación en una región distinta y restaurar la base de datos desde la copia de seguridad mediante la restauración geográfica. Este proceso tiene como resultado un tiempo de inactividad considerable.

## <a name="upgrade-applications-that-rely-on-database-geo-replication-for-disaster-recovery"></a>Actualización de aplicaciones que se basan en la replicación geográfica de la base de datos para la recuperación ante desastres

Si la aplicación usa la replicación geográfica activa o grupos de conmutación por error automática para la continuidad empresarial, se implementa en al menos dos regiones diferentes. Hay una base de datos principal activa en una región primaria y una base de datos secundaria de solo lectura en una región de copia de seguridad. Junto con los factores mencionados al principio de este artículo, el proceso de actualización también debe garantizar que:

* La aplicación permanece protegida frente a errores graves en todo momento durante el proceso de actualización.
* Los componentes con redundancia geográfica de la aplicación se actualizan en paralelo con los componentes activos.

Para lograr estos objetivos, además de usar los entornos de Web Apps, podrá aprovechar Azure Traffic Manager mediante un perfil de conmutación por error con un punto de conexión activo y un punto de conexión de copia de seguridad. En el siguiente diagrama se ilustra el entorno operativo antes del proceso de actualización. Los sitios web `contoso-1.azurewebsites.net` y `contoso-dr.azurewebsites.net` representan un entorno de producción de la aplicación con redundancia geográfica completa. El entorno de producción incluye los siguientes componentes:

* El entorno de producción de la aplicación web `contoso-1.azurewebsites.net` en la región primaria (1)
* La base de datos principal en la región primaria (2)
* Una instancia en espera de la aplicación web en la región de copia de seguridad (3)
* La base de datos secundaria de replicación geográfica en la región de copia de seguridad (4)
* Un perfil de rendimiento de Traffic Manager con un punto de conexión en línea llamado `contoso-1.azurewebsites.net` y un punto de conexión sin conexión llamado `contoso-dr.azurewebsites.net`

Para que sea posible revertir la actualización, debe crear un entorno de ensayo con una copia totalmente sincronizada de la aplicación. Dado que debe asegurarse de que la aplicación puede recuperarse rápidamente en caso de que se produzca un error grave durante el proceso de actualización, el entorno de ensayo tiene que tener también redundancia geográfica. Estos son los pasos necesarios para crear un entorno de ensayo para la actualización:

1. Implemente un entorno de ensayo de la aplicación web en la región primaria (6).
2. Cree una base de datos secundaria en la región principal de Azure (7). Configure el entorno de ensayo de la aplicación web para conectarse a él. 
3. Cree otra base de datos secundaria con redundancia geográfica en la región de copia de seguridad mediante la replicación de la base de datos secundaria en la región primaria. (Este método se denomina *replicación geográfica encadenada*) (8).
4. Implemente un entorno de ensayo de la instancia de la aplicación web en la región de copia de seguridad (9) y configúrelo para conectar la base de datos secundaria con redundancia geográfica que ha creado en el paso (8).

> [!NOTE]
> Estos pasos de preparación no afectan a la aplicación en el entorno de producción. Seguirá siendo completamente funcional en modo de lectura y escritura.

![Diagrama que muestra la configuración de la replicación geográfica de SQL Database para la recuperación ante desastres en la nube con una copia completamente sincronizada de la aplicación.](./media/manage-application-rolling-upgrade/option2-1.png)

Una vez completados los pasos preparatorios, el entorno de ensayo está listo para la actualización. En el diagrama siguiente se muestran estos pasos de actualización:

1. Establezca la base de datos principal en el entorno de producción en modo de solo lectura (10). Este modo garantiza que la base de datos de producción (V1) no cambiará durante la actualización, con lo que se impide la divergencia de datos entre las instancias de base de datos V1 y V2.

```sql
-- Set the production database to read-only mode
ALTER DATABASE <Prod_DB>
SET (ALLOW_CONNECTIONS = NO)
```

2. Finalice la replicación geográfica mediante la desconexión de la secundaria (11). Esta acción crea una copia independiente aunque totalmente sincronizada de la base de datos de producción. Esta base de datos se actualizará. En el ejemplo siguiente se utiliza Transact-SQL, pero [PowerShell](/powershell/module/az.sql/remove-azsqldatabasesecondary) también está disponible. 

```sql
-- Disconnect the secondary, terminating geo-replication
ALTER DATABASE <Prod_DB>
REMOVE SECONDARY ON SERVER <Partner-Server>
```

3. Ejecute el script de actualización en `contoso-1-staging.azurewebsites.net`, `contoso-dr-staging.azurewebsites.net` y la base de datos principal de ensayo (12). Los cambios de la base de datos se replicarán automáticamente en la base de datos secundaria de ensayo.

![Diagrama que muestra la configuración de la replicación geográfica de SQL Database para la recuperación ante desastres en la nube con cambios en la base de datos replicados en el ensayo.](./media/manage-application-rolling-upgrade/option2-2.png)

Si la actualización se completó correctamente, ya está listo para cambiar a los usuarios a la versión V2 de la aplicación. En el siguiente diagrama se ilustran los pasos que intervienen.

1. Active una operación de intercambio entre los entornos de ensayo y producción de la aplicación web en la región primaria (13) y en la región de copia de seguridad (14). La versión V2 de la aplicación se convierte ahora en un entorno de producción con una copia redundante en la región de copia de seguridad.
2. Si ya no necesita la aplicación V1 (15 y 16), puede retirar el entorno de ensayo.

![Diagrama que muestra la configuración de la replicación geográfica de SQL Database para la recuperación ante desastres en la nube con retirada opcional del entorno de ensayo.](./media/manage-application-rolling-upgrade/option2-3.png)

Si el proceso de actualización no se realiza correctamente, por ejemplo, debido a un error en el script de actualización, el entorno de ensayo debe considerarse en estado incoherente. Para revertir la aplicación al estado previo a la actualización, revierta al uso de la versión V1 de la aplicación en el entorno de producción. En el diagrama siguiente se muestran los pasos necesarios:

1. Establezca la copia de la base de datos principal en el entorno de producción en modo de lectura y escritura (17). Esta acción restaura la funcionalidad de V1 completa en el entorno de producción.
2. Realice el análisis de la causa principal y repare o quite el entorno de ensayo (18 y 19).

En este punto, la aplicación es totalmente funcional y se pueden repetir los pasos de actualización.

> [!NOTE]
> La reversión no requiere cambios de DNS, ya que no ha realizado una operación de intercambio.

![Diagrama que muestra la configuración de la replicación geográfica de SQL Database para la recuperación ante desastres en la nube con el proceso de actualización revertido.](./media/manage-application-rolling-upgrade/option2-4.png)

La principal ventaja de esta opción es que puede actualizar tanto la aplicación como su copia con redundancia geográfica en paralelo, sin poner en peligro la continuidad empresarial durante la actualización.

La principal desventaja es que requiere redundancia doble de cada componente de aplicación y, por tanto, implica un mayor costo. También implica un flujo de trabajo más complicado.

## <a name="summary"></a>Resumen

Los dos métodos de actualización que se describen en el artículo difieren en complejidad y costo, pero ambos se centran en reducir la cantidad de tiempo que el usuario está limitado a las operaciones de solo lectura. La duración del script de actualización define directamente ese tiempo. No depende del tamaño de la base de datos, del nivel de servicio que haya seleccionado, de la configuración del sitio web ni de otros factores que no se puedan controlar fácilmente. Todos los pasos preparatorios están separados de los pasos de actualización y no afectan a la aplicación de producción. La eficacia del script de actualización es un factor clave que determina la experiencia del usuario durante las actualizaciones. Por lo tanto, la mejor manera de mejorar esa experiencia es centrar sus esfuerzos en conseguir que el script de actualización sea lo más eficaz posible.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener una descripción general y los escenarios de la continuidad empresarial, consulte [Información general sobre la continuidad empresarial](business-continuity-high-availability-disaster-recover-hadr-overview.md).
* Para información sobre la replicación geográfica activa de Azure SQL Database, consulte [Creación de bases de datos secundarias legibles mediante la replicación geográfica activa](active-geo-replication-overview.md).
* Para información sobre los grupos de conmutación por error automática de Azure SQL Database, consulte [Uso de grupos de conmutación por error automática para permitir la conmutación por error de varias bases de datos de manera transparente y coordinada](auto-failover-group-overview.md).
* Para información sobre las ranuras de implementación en Azure App Service, consulte [Configuración de entornos de ensayo en Azure App Service](../../app-service/deploy-staging-slots.md).
* Para información sobre los perfiles de Azure Traffic Manager, consulte [Administrar un perfil de Azure Traffic Manager](../../traffic-manager/traffic-manager-manage-profiles.md).
