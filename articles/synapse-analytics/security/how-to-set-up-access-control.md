---
title: Cómo configurar el control de acceso para el área de trabajo de Synapse
description: En este artículo aprenderá controlar el acceso a un área de trabajo de Synapse con roles de Azure y Synapse y permisos de SQL y GIT.
services: synapse-analytics
author: RonyMSFT
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 12/03/2020
ms.author: ronytho
ms.reviewer: jrasnick
ms.openlocfilehash: 91eaf655a3259cff31767353fb2c2b7fcd787d63
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108122968"
---
# <a name="how-to-set-up-access-control-for-your-synapse-workspace"></a>Cómo configurar el control de acceso para el área de trabajo de Synapse 

En este artículo aprenderá controlar el acceso a un área de trabajo de Synapse con roles de Azure y Synapse y permisos de SQL y GIT.   

En esta guía, configurará un área de trabajo y un sistema de control de acceso básico adecuado para muchos proyectos de Synapse.  A continuación, se describen opciones más avanzadas para que pueda realizar un control más preciso en caso necesario.  

Para simplificar el control de acceso de Synapse, se pueden usar grupos de seguridad que estén alineados con los roles y las personas de la organización.  Para administrar el acceso solo hay que agregar y quitar usuarios de los grupos de seguridad.

Antes de comenzar este tutorial, lea la [información general sobre el control de acceso de Synapse](./synapse-workspace-access-control-overview.md) para familiarizarse con los mecanismos de control de acceso que usa Synapse.   

## <a name="access-control-mechanisms"></a>Mecanismos de control de acceso

> [!NOTE]
> El enfoque que se toma en esta guía es la creación de varios grupos de seguridad y, a continuación, la asignación de roles a estos grupos. Una vez configurados los grupos, solo necesita administrar la pertenencia en los grupos de seguridad a fin de controlar el acceso al área de trabajo.

Para proteger un área de trabajo de Synapse, debe seguir un patrón de configuración para los siguientes elementos:

- **Grupos de seguridad**, para agrupar usuarios con requisitos de acceso similares.
- **Roles de Azure**, para controlar quién puede crear y administrar grupos de SQL, grupos de Apache Spark y entornos de ejecución de integración, y acceder al almacenamiento de ADLS Gen2.
- **Roles de Synapse**, para controlar el acceso a los artefactos de código publicados, el uso de recursos de proceso de Apache Spark y los entornos de ejecución de integración. 
- **Permisos de SQL**, para controlar el acceso administrativo y del plano de datos a los grupos de SQL. 
- **Permisos de GIT**, para controlar quién puede acceder a los artefactos de código en el control de código fuente si configura la compatibilidad con GIT para el área de trabajo. 
 
## <a name="steps-to-secure-a-synapse-workspace"></a>Pasos para proteger un área de trabajo de Synapse

En este documento se usan nombres estándar para simplificar las instrucciones. Reemplácelos por los nombres de su preferencia.

|Configuración | Nombre estándar | Descripción |
| :------ | :-------------- | :---------- |
| **Área de trabajo de Synapse** | `workspace1` |  Nombre que tendrá el área de trabajo de Synapse. |
| **Cuenta de ADLSGEN2** | `storage1` | Cuenta de ADLS que se usará con el área de trabajo. |
| **Contenedor** | `container1` | Contenedor en STG1 que el área de trabajo usará de forma predeterminada. |
| **Inquilino de Active Directory** | `contoso` | Nombre del inquilino de Active Directory.|
||||

## <a name="step-1-set-up-security-groups"></a>PASO 1: Configuración de los grupos de seguridad

>[!Note] 
>Durante la versión preliminar, le recomendamos que cree grupos de seguridad asignados a los roles de **Administrador de SQL de Synapse SQL** y **Administrador de Apache Spark de Synapse**.  Con la introducción de nuevos ámbitos y roles de RBAC de Synapse más precisos, ahora le recomendamos que use estas nuevas funcionalidades para controlar el acceso al área de trabajo.  Estos nuevos roles y ámbitos proporcionan más flexibilidad de configuración y reconocen que los desarrolladores a menudo usan una combinación de SQL y Spark en la creación de aplicaciones de análisis. Así mismo, puede que se les deba conceder acceso a recursos específicos, en lugar de a todo el área de trabajo. [Más información](./synapse-workspace-synapse-rbac.md) acerca de RBAC de Synapse.

Cree los siguientes grupos de seguridad para su área de trabajo:

- **`workspace1_SynapseAdministrators`** , para los usuarios que necesitan control completo sobre el área de trabajo.  Agréguese a usted mismo a este grupo de seguridad, al menos inicialmente.
- **`workspace1_SynapseContributors`** , para los desarrolladores que necesitan desarrollar, depurar y publicar código en el servicio.   
- **`workspace1_SynapseComputeOperators`** , para los usuarios que necesitan administrar y supervisar los grupos de Apache Spark y los entornos de ejecución de integración.
- **`workspace1_SynapseCredentialUsers`** , para los usuarios que necesitan depurar y ejecutar canalizaciones de orquestación mediante la credencial de las identidades administradas para recursos de Azure del área de trabajo, así como cancelar ejecuciones de canalización.   

En breve, asignará roles de Synapse a estos grupos en el ámbito del área de trabajo.  

Cree también este grupo de seguridad: 
- **`workspace1_SQLAdmins`** , grupo para los usuarios que necesitan autoridad de administrador de SQL Active Directory en los grupos de SQL del área de trabajo. 

El grupo de `workspace1_SQLAdmins` se usará al configurar permisos SQL en grupos de SQL a medida que se creen. 

En el caso de una configuración básica, estos cinco grupos son suficientes. Más adelante, podrá agregar grupos de seguridad para administrar los usuarios que necesiten acceso más especializado o para proporcionar acceso a los usuarios solo a recursos específicos.

> [!NOTE]
>- Obtenga información sobre cómo crear un grupo de seguridad en [este artículo](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).
>- Obtenga información sobre cómo agregar un grupo de seguridad de otro grupo de seguridad en [este artículo](../../active-directory/fundamentals/active-directory-groups-membership-azure-portal.md).

>[!Tip]
>Los usuarios individuales de Synapse pueden usar Azure Active Directory en Azure Portal para ver las pertenencias a grupos y determinar qué roles se les han concedido.

## <a name="step-2-prepare-your-adls-gen2-storage-account"></a>PASO 2: Prepare la cuenta de almacenamiento de ADLS Gen2.

Un área de trabajo de Synapse usa un contenedor de almacenamiento predeterminado para lo siguiente:
  - Almacenar los archivos de datos de copia de seguridad para las tablas de Spark
  - Los registros de ejecución de los trabajos de Spark
  - Administrar las bibliotecas que decida instalar

Identifique la siguiente información sobre su almacenamiento:

- Cuenta de ADLS Gen2 que usará para el área de trabajo. En este documento, se llama `storage1`. `storage1` se considera la cuenta de almacenamiento "principal" del área de trabajo.
- Contenedor dentro de `workspace1` que el área de trabajo de Synapse usará de forma predeterminada. En este documento, se llama `container1`. 

- Mediante Azure Portal, asigne los grupos de seguridad a los siguientes roles de Azure en `container1`. 

  - Asigne el rol **Colaborador de datos de Storage Blob** a `workspace1_SynapseAdmins`. 
  - Asigne el rol **Colaborador de datos de Storage Blob** a `workspace1_SynapseContributors`.
  - Asigne el rol **Colaborador de datos de Storage Blob** a `workspace1_SynapseComputeOperators`.

## <a name="step-3-create-and-configure-your-synapse-workspace"></a>PASO 3: Creación y configuración del área de trabajo de Synapse

En Azure Portal, cree un área de trabajo de Synapse:

- Seleccione su suscripción.
- Seleccione o cree un grupo de recursos para el que tenga el rol **Propietario** de Azure.
- Asigne el nombre `workspace1` al área de trabajo.
- Seleccione `storage1` en la cuenta de almacenamiento.
- Elija `container1` para el contenedor que se usará como "sistema de archivos".
- Abra WS1 en Synapse Studio.
- Navegue hasta **Administrar** > **Control de acceso** y asigne los siguientes roles de Synapse del *ámbito del área de trabajo* a los grupos de seguridad, de la manera siguiente:
  - Asigne el rol **Administrador de Synapse** en `workspace1_SynapseAdministrators`. 
  - Asigne el rol **Colaborador de Synapse** en `workspace1_SynapseContributors`. 
  - Asigne el rol **Operador de proceso de Synapse** a `workspace1_SynapseComputeOperators`.

## <a name="step-4-grant-the-workspace-msi-access-to-the-default-storage-container"></a>PASO 4: Concesión de acceso a las identidades administradas para recursos de Azure del área de trabajo al contenedor de almacenamiento predeterminado

Para ejecutar canalizaciones y realizar tareas del sistema, Synapse requiere que las identidades de servicio administradas (MSI) del área de trabajo tengan acceso a `container1` de la cuenta de ADLS Gen2 predeterminada.

- Abra Azure Portal.
- Busque la cuenta de almacenamiento, `storage1`, y, luego, `container1`.
- Con **Access Control (IAM)** , asegúrese de que el rol **Colaborador de datos de blobs de almacenamiento** esté asignado a las identidades administradas para recursos de Azure del área de trabajo.
  - Si no lo está, asígnelo.
  - El MSI tiene el mismo nombre que el área de trabajo. En este ejemplo, sería `workspace1`.

## <a name="step-5-grant-synapse-administrators-the-azure-contributor-role-on-the-workspace"></a>PASO 5: Concesión del rol Colaborador de Azure a los administradores de Synapse en el área de trabajo 

Para crear grupos de SQL, grupos de Apache Spark y entornos de ejecución de integración, los usuarios deben tener como mínimo el rol Colaborador de Azure en el área de trabajo. El rol Colaborador también permite a estos usuarios administrar los recursos, incluidas la pausa y el escalado. Si usa Azure Portal o Synapse Studio para crear grupos de SQL, grupos de Apache Spark y entornos de ejecución de integración, necesitará el rol Colaborador de Azure en el nivel de grupo de recursos. 

- Abra Azure Portal.
- Busque el área de trabajo `workspace1`.
- Asigne el rol **Colaborador** de Azure de `workspace1` a `workspace1_SynapseAdministrators`. 

## <a name="step-6-assign-sql-active-directory-admin-role"></a>PASO 6: Asignación del rol Administrador de SQL Active Directory

El creador del área de trabajo se configura automáticamente como el administrador de SQL Active Directory para el área de trabajo.  Este rol solo se puede conceder a un solo usuario o grupo. En este paso, asignará el administrador de SQL Active Directory del área de trabajo al grupo de seguridad `workspace1_SQLAdmins`.  La asignación de este rol proporciona a este grupo el acceso de administrador con privilegios elevados a todos los grupos y base de datos de SQL en el área de trabajo.   

- Abra Azure Portal.
- Vaya a `workspace1`.
- En **Configuración**, seleccione **Administrador de SQL Active Directory**.
- Seleccione **Establecer administrador** y elija **`workspace1_SQLAdmins`** .

>[!Note]
>El paso 6 es opcional.  Puede optar por conceder al grupo `workspace1_SQLAdmins` un rol con menos privilegios. Para asignar `db_owner` u otros roles de SQL, debe ejecutar scripts en cada base de datos SQL. 

## <a name="step-7-grant-access-to-sql-pools"></a>PASO 7: Concesión de acceso a grupos de SQL

De forma predeterminada, a todos los usuarios a los que se asigna el rol Administrador de Synapse también se les asigna el rol `db_owner` de SQL en el grupo de SQL sin servidor, "Integrado", y todas sus bases de datos.

El acceso a los grupos de SQL para otros usuarios y para las identidades administradas para recursos de Azure del área de trabajo se controla mediante permisos SQL.  La asignación de permisos de SQL requiere que los scripts de SQL se ejecuten en cada base de datos SQL después de la creación.  Hay tres casos en los que es necesario ejecutar estos scripts:
1. Al conceder a otros usuarios acceso al grupo de SQL sin servidor, "Integrado", y sus bases de datos
2. Al conceder acceso a cualquier usuario a bases de datos de grupos dedicados
3. Al conceder acceso a las identidades de servicio administradas del área de trabajo a una base de datos de grupo de SQL a fin de habilitar canalizaciones que requieran el acceso al grupo de SQL para ejecutarse correctamente.

A continuación, se incluyen scripts SQL de ejemplo.

Para conceder acceso a una base de datos de grupo de SQL dedicado, el creador del área de trabajo o cualquier miembro del grupo `workspace1_SQLAdmins` puede ejecutar los scripts.  

Para conceder acceso al grupo de SQL sin servidor ("Integrado"), cualquier miembro del grupo `workspace1_SQLAdmins` o del grupo `workspace1_SynapseAdministrators` puede ejecutar los scripts. 

> [!TIP]
> Los pasos siguientes se deben realizar para **cada** grupo de SQL con el fin de conceder a los usuarios el acceso a todas las bases de datos SQL, excepto en la sección [Permiso de ámbito de área de trabajo](#workspace-scoped-permission), en que puede asignar un rol sysadmin a un usuario en el nivel de área de trabajo.

### <a name="step-71-serverless-sql-pool-built-in"></a>PASO 7.1: Grupo de SQL sin servidor, integrado

En esta sección, hay ejemplos de script que muestran cómo conceder a un usuario el permiso para acceder a una base de datos determinada o a todas las bases de datos del grupo de SQL sin servidor, "Integrado".

> [!NOTE]
> En los ejemplos de script, reemplace *alias* por el alias del usuario o grupo al que se concede el acceso y *dominio* por el dominio de la empresa que usa.

#### <a name="database-scoped-permission"></a>Permiso de ámbito de base de datos

Para conceder a un usuario el acceso a una **única** base de datos SQL sin servidor, siga los pasos de este ejemplo:

1. Cree LOGIN

    ```sql
    use master
    go
    CREATE LOGIN [alias@domain.com] FROM EXTERNAL PROVIDER;
    go
    ```

2. Cree USER

    ```sql
    use yourdb -- Use your database name
    go
    CREATE USER alias FROM LOGIN [alias@domain.com];
    ```

3. Agregue USER a los miembros del rol especificado

    ```sql
    use yourdb -- Use your database name
    go
    alter role db_owner Add member alias -- Type USER name from step 2
    ```

#### <a name="workspace-scoped-permission"></a>Permiso de ámbito de área de trabajo

Para conceder acceso completo a **todos** los grupos de SQL sin servidor en el área de trabajo, use el script en este ejemplo:

```sql
use master
go
CREATE LOGIN [alias@domain.com] FROM EXTERNAL PROVIDER;
ALTER SERVER ROLE sysadmin ADD MEMBER [alias@domain.com];
```

### <a name="step-72-dedicated-sql-pools"></a>PASO 7.2: Grupos de SQL dedicados

Para conceder acceso a un **único** grupo de SQL dedicado, siga estos pasos en el editor de scripts de Synapse SQL:

1. Cree el usuario en la base de datos. Para ello, ejecute el siguiente comando en la base de datos de destino, que seleccionó mediante la lista desplegable *Conectar a*:

    ```sql
    --Create user in the database
    CREATE USER [<alias@domain.com>] FROM EXTERNAL PROVIDER;
    ```

2. Conceda al usuario un rol para acceder a la base de datos:

    ```sql
    --Grant role to the user in the database
    EXEC sp_addrolemember 'db_owner', '<alias@domain.com>';
    ```

> [!IMPORTANT]
> *db_datareader* y *db_datawriter* pueden funcionar para los permisos de lectura y escritura si no se quiere conceder el permiso *db_owner*.
> Para que los usuarios de Spark lean el contenido directamente desde Spark de un grupo de SQL o escriban en él, se requiere el permiso *db_owner*.

Después de crear los usuarios, ejecute consultas para validar que el grupo de SQL sin servidor pueda realizar consultas en la cuenta de almacenamiento.

### <a name="step-73-sql-access-control-for-synapse-pipeline-runs"></a>PASO 7.3: Control de acceso SQL para las ejecuciones de canalización de Synapse

### <a name="workspace-managed-identity"></a>Identidad administrada del área de trabajo

> [!IMPORTANT]
> Para ejecutar correctamente canalizaciones que incluyen conjuntos de datos o actividades que hacen referencia a un grupo de SQL, es preciso que se conceda a la identidad del área de trabajo acceso al grupo de SQL.

Ejecute los siguientes comandos en cada grupo de SQL para permitir que la identidad del sistema administrada del área de trabajo ejecute canalizaciones en las bases de datos del grupo de SQL:  

>[!note]
>En los siguientes scripts, para una base de datos de grupo de SQL dedicado, databasename es igual que el nombre del grupo.  En el caso de una base de datos del grupo de SQL sin servidor "Integrado", databasename es el nombre de la base de datos.

```sql
--Create a SQL user for the workspace MSI in database
CREATE USER [<workspacename>] FROM EXTERNAL PROVIDER;

--Granting permission to the identity
GRANT CONTROL ON DATABASE::<databasename> TO <workspacename>;
```

Este permiso se puede quitar. Para ello, es preciso ejecutar el siguiente script en el mismo grupo de SQL:

```sql
--Revoke permission granted to the workspace MSI
REVOKE CONTROL ON DATABASE::<databasename> TO <workspacename>;

--Delete the workspace MSI user in the database
DROP USER [<workspacename>];
```

## <a name="step-8-add-users-to-security-groups"></a>PASO 8: Adición de usuarios a grupos de seguridad

La configuración inicial del sistema de control de acceso está completa.

Para administrar el acceso, puede agregar y quitar usuarios de los grupos de seguridad que configuró.  Aunque puede asignar los usuarios manualmente a los roles de Synapse, al hacerlo no configurará los permisos de forma coherente. En su lugar, solo tiene que agregar o quitar usuarios de los grupos de seguridad.

## <a name="step-9-network-security"></a>PASO 9: Seguridad de redes

Como paso final para proteger el área de trabajo, debe proteger el acceso a la red mediante lo siguiente:
- [Firewall del área de trabajo](./synapse-workspace-ip-firewall.md)
- [Red virtual administrada](./synapse-workspace-managed-vnet.md) 
- [Puntos de conexión privados](./synapse-workspace-managed-private-endpoints.md)
- [Private Link](../../azure-sql/database/private-endpoint-overview.md)

## <a name="step-10-completion"></a>PASO 10: Completion

El área de trabajo ahora está completamente configurada y protegida.

## <a name="supporting-more-advanced-scenarios"></a>Compatibilidad con escenarios más avanzados

Esta guía se centra en la configuración de un sistema de control de acceso básico. Para admitir escenarios más avanzados, puede crear grupos de seguridad adicionales y asignarles roles más precisos en ámbitos más específicos. Considere los casos siguientes:

**Habilite la compatibilidad con GIT** para el área de trabajo a fin de obtener escenarios de desarrollo más avanzados, como CI/CD.  En el modo GIT, los permisos de GIT determinarán si un usuario puede confirmar los cambios en su rama de trabajo.  La publicación en el servicio solo tiene lugar desde la rama de colaboración.  Considere la posibilidad de crear un grupo de seguridad para los desarrolladores que necesitan desarrollar y depurar las actualizaciones en una rama de trabajo, pero que no necesitan publicar los cambios en el servicio en directo.

**Restrinja el acceso de desarrollador** a recursos específicos.  Cree otros grupos de seguridad adicionales más precisos para los desarrolladores que solo necesitan acceder a recursos específicos.  Asigne a estos grupos roles de Synapse adecuados que tengan como ámbito grupos de Spark específicos, entornos de ejecución de integración o credenciales.

**Restrinja el acceso a los artefactos de código a los operadores**.  Cree grupos de seguridad para los operadores que deban supervisar el estado operativo de los recursos de proceso de Synapse y ver los registros, pero que no necesitan acceder al código ni publicar actualizaciones en el servicio. Asigne a estos grupos el rol Operador de proceso en el ámbito de grupos de Spark y de entornos de ejecución de integración específicos.  

## <a name="next-steps"></a>Pasos siguientes

Aprenda a [administrar asignaciones de roles RBAC de Synapse](./how-to-manage-synapse-rbac-role-assignments.md). Cree una [área de trabajo de Synapse](../quickstart-create-workspace.md).
