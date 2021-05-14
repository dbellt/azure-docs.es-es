---
title: Creación de usuarios de Hiperescala (Citus) en Azure Database for PostgreSQL
description: En este artículo se describe cómo puede crear nuevas cuentas de usuario para interactuar con Azure Database for PostgreSQL - Hiperescala (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 1/8/2019
ms.openlocfilehash: 3772918d1a7e505b3994afa7849568bb309bd0cf
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109715419"
---
# <a name="create-users-in-azure-database-for-postgresql---hyperscale-citus"></a>Crear usuarios en Azure Database for PostgreSQL - Hiperescala (Citus)

## <a name="the-server-admin-account"></a>La cuenta de administrador del servidor

El motor de PostgreSQL usa [roles](https://www.postgresql.org/docs/current/sql-createrole.html) para controlar el acceso a los objetos de base de datos y un grupo de servidores de Hiperescala (Citus) recién creado incluye varios roles predefinidos:

* [Roles predeterminados de PostgreSQL](https://www.postgresql.org/docs/current/default-roles.html)
* `azure_pg_admin`
* `postgres`
* `citus`

Dado que Hiperescala (Citus) es un servicio PaaS administrado, solo Microsoft puede iniciar sesión con el rol de superusuario `postgres`. Para el acceso administrativo limitado, Hiperescala (Citus) proporciona el rol `citus`.

Permisos para el rol de `citus`:

* Leer todas las variables de configuración, incluso las variables que normalmente solo son visibles para los superusuarios.
* Leer todas las vistas de pg\_stat\_\* y usar varias extensiones relacionadas con estadísticas, incluso las vistas o extensiones que normalmente solo son visibles para los superusuarios.
* Ejecutar funciones de supervisión que puedan tener bloqueos de uso compartido de acceso en tablas, potencialmente durante mucho tiempo.
* [Crear extensiones de PostgreSQL](concepts-hyperscale-extensions.md) (porque el rol es miembro de `azure_pg_admin`).

En particular, el rol `citus` tiene algunas restricciones:

* No puede crear roles.
* No puede crear bases de datos.

## <a name="how-to-create-additional-user-roles"></a>Cómo crear roles de usuario adicionales

Como se ha mencionado, la cuenta de administrador `citus` carece de permiso para crear usuarios adicionales. Para agregar un usuario, use la interfaz de Azure Portal.

1. Vaya a la página **Roles** del grupo de servidores de Hiperescala (Citus) y haga clic en **+ Agregar**:

   :::image type="content" source="media/howto-hyperscale-create-users/1-role-page.png" alt-text="Página de roles":::

2. Introduzca el nombre y la contraseña del rol. Haga clic en **Save**(Guardar).

   :::image type="content" source="media/howto-hyperscale-create-users/2-add-user-fields.png" alt-text="Agregar rol":::

El usuario se creará en el nodo coordinador del grupo de servidores y se propagará a todos los nodos de trabajo. Los roles creados a través de Azure Portal tienen el atributo `LOGIN`, lo que significa que son verdaderos usuarios que pueden iniciar sesión en la base de datos.

## <a name="how-to-modify-privileges-for-user-role"></a>Cómo modificar los privilegios para el rol de usuario

Los nuevos roles se usan comúnmente para proporcionar acceso a la base de datos con privilegios restringidos. Para modificar los privilegios de usuario, use los comandos estándar de PostgreSQL con una herramienta como PgAdmin o psql. (Consulte [conectarse con psql](quickstart-create-hyperscale-portal.md#connect-to-the-database-using-psql) en el inicio rápido de la Hiperescala (Citus).)

Por ejemplo, para permitir que `db_user` lea `mytable`, conceda el permiso:

```sql
GRANT SELECT ON mytable TO db_user;
```

Hiperescala (Citus) propaga las instrucciones GRANT de tabla única a través de todo el clúster y las aplica a todos los nodos de trabajo. También propaga instrucciones GRANT que están en todo el sistema (por ejemplo, para todas las tablas de un esquema):

```sql
-- applies to the coordinator node and propagates to workers
GRANT SELECT ON ALL TABLES IN SCHEMA public TO db_user;
```

## <a name="how-to-delete-a-user-role-or-change-their-password"></a>Cómo eliminar un rol de usuario o cambiar su contraseña

Para actualizar un usuario, visite la página **Roles** del grupo de servidores de servidores de Hiperescala (Citus)y haga clic en los puntos suspensivos **...** junto al usuario. Los puntos suspensivos abrirán un menú para eliminar el usuario o restablecer su contraseña.

   :::image type="content" source="media/howto-hyperscale-create-users/edit-role.png" alt-text="Editar un rol":::

El rol `citus` tiene privilegios y no se puede eliminar.

## <a name="next-steps"></a>Pasos siguientes

Abra el firewall para las direcciones IP de las máquinas de los nuevos usuarios para permitirles conectarse: [Creación y administración de las reglas de firewall en Hiperescala (Citus) con Azure Portal](howto-hyperscale-manage-firewall-using-portal.md).

Para más información sobre la gestión de cuentas de usuario de la base de datos, consulte la documentación del producto PostgreSQL:

* [Roles y privilegios de la base de datos](https://www.postgresql.org/docs/current/static/user-manag.html)
* [Sintaxis GRANT](https://www.postgresql.org/docs/current/static/sql-grant.html)
* [Privilegios](https://www.postgresql.org/docs/current/static/ddl-priv.html)
