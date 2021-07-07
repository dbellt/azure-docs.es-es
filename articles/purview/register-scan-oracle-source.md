---
title: Registro de un origen de Oracle y configuración de los exámenes en Azure Purview
description: En este artículo se explica cómo registrar un origen de Oracle en Azure Purview y cómo configurar un examen.
author: chandrakavya
ms.author: kchandra
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: overview
ms.date: 2/25/2021
ms.openlocfilehash: dadf457c2fc3f617e2473a853e802fe863709ac5
ms.sourcegitcommit: 8651d19fca8c5f709cbb22bfcbe2fd4a1c8e429f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112072296"
---
# <a name="register-and-scan-oracle-source"></a>Registro y examen de un origen de Oracle

En este artículo se explica cómo registrar una base de datos de Oracle en Purview y cómo configurar un examen.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

El origen de Oracle admite un **examen completo** para extraer metadatos de una base de datos de Oracle y captura el **linaje** entre recursos de datos.

## <a name="prerequisites"></a>Requisitos previos

1.  Configure la versión más reciente del [entorno de ejecución de integración autohospedado](https://www.microsoft.com/download/details.aspx?id=39717).
    Para obtener más información, consulte [Creación y configuración de un entorno de ejecución de integración autohospedado](../data-factory/create-self-hosted-integration-runtime.md).

2.  Asegúrese de que [JDK 11](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) está instalado en la máquina virtual donde también lo está el entorno de ejecución de integración autohospedado.

3.  Asegúrese de que \"Visual C++ Redistributable 2012 Update 4\" está instalado en la máquina del entorno de ejecución de integración autohospedado. Si aún no está instalado, descárguelo [aquí](https://www.microsoft.com/download/details.aspx?id=30679).

4.  Tiene que descargar manualmente un controlador JDBC de Oracle desde [aquí](https://www.oracle.com/database/technologies/appdev/jdbc-downloads.html) en la máquina virtual donde se ejecuta el entorno de ejecución de integración autohospedado.

    > [!Note] 
    > Todas las cuentas de la máquina virtual deben poder acceder al controlador. No lo instale en una cuenta de usuario.

5.  Las versiones de bases de datos de Oracle admitidas abarcan desde la 6i a la 19c.

6.  Permiso del usuario: se requiere un acceso de solo lectura a las tablas del sistema. El usuario debe contar con permiso para crear una sesión y tener asignado el rol SELECT\_CATALOG\_ROLE. También puede tener concedido el permiso SELECT en cada tabla del sistema en donde este conector consulte metadatos:
       > grant create session to \[usuario\];\
        grant select on all\_users to \[usuario\];\
        grant select on dba\_objects to \[usuario\];\
        grant select on dba\_tab\_comments to \[usuario\];\
        grant select on dba\_external\_locations to \[usuario\];\
        grant select on dba\_directories to \[usuario\];\
        grant select on dba\_mviews to \[usuario\];\
        grant select on dba\_clu\_columns to \[usuario\];\
        grant select on dba\_tab\_columns to \[usuario\];\
        grant select on dba\_col\_comments to \[usuario\];\
        grant select on dba\_constraints to \[usuario\];\
        grant select on dba\_cons\_columns to \[usuario\];\
        grant select on dba\_indexes to \[usuario\];\
        grant select on dba\_ind\_columns to \[usuario\];\
        grant select on dba\_procedures to \[usuario\];\
        grant select on dba\_synonyms to \[usuario\];\
        grant select on dba\_views to \[usuario\];\
        grant select on dba\_source to \[usuario\];\
        grant select on dba\_triggers to \[usuario\];\
        grant select on dba\_arguments to \[usuario\];\
        grant select on dba\_sequences to \[usuario\];\
        grant select on dba\_dependencies to \[usuario\];\
        grant select on V\_\$INSTANCE to \[usuario\];\
        grant select on v\_\$database to \[usuario\];
    
## <a name="setting-up-authentication-for-a-scan"></a>Configuración de la autenticación para un examen

La única autenticación admitida en un origen de Oracle es la **autenticación básica**.

## <a name="register-an-oracle-source"></a>Registro de un origen de Oracle

Para registrar un nuevo origen de Oracle en el catálogo de datos, haga lo siguiente:

1.  Vaya a la cuenta de Purview.
2.  Seleccione **Sources** (Orígenes) en el panel de navegación izquierdo.
3.  Seleccione **Registrar**.
4.  En Register sources (Registrar orígenes), seleccione **Oracle**. Seleccione **Continuar**.

    :::image type="content" source="media/register-scan-oracle-source/register-sources.png" alt-text="Registro de orígenes" border="true":::

En la pantalla **Register sources (Oracle)** (Registrar orígenes (Oracle)), haga lo siguiente:

1.  Escriba un **Name** (Nombre) con el que se muestre el origen de datos en el catálogo.

2.  Escriba el nombre de **Host** para conectarse a un origen de Oracle. Puede ser:
    - Un nombre de host usado por JDBC para conectarse al servidor de bases de datos. Por ejemplo, MyDatabaseServer.com o
    - Dirección IP. Por ejemplo, 192.169.1.2 o
    - Su cadena de conexión de JDBC completa. Por ejemplo,\
        jdbc:oracle:thin:@(DESCRIPTION=(LOAD\_BALANCE=on)(ADDRESS=(PROTOCOL=TCP)(HOST=oracleserver1)(PORT=1521))(ADDRESS=(PROTOCOL=TCP)(HOST=oracleserver2)(PORT=1521))(ADDRESS=(PROTOCOL=TCP)(HOST=oracleserver3)(PORT=1521))(CONNECT\_DATA=(SERVICE\_NAME=orcl)))

3.  Escriba el **Port number** (Número de puerto) que use JDBC para conectarse al servidor de bases de datos (1521 de manera predeterminada en el caso de Oracle).

4.  Escriba el **Oracle service name** (Nombre del servicio de Oracle) que usa JDBC para conectarse al servidor de bases de datos.

5.  Seleccione una colección o cree una nueva (opcional).

6.  Seleccione Finish (Finalizar) para registrar el origen de datos.

    :::image type="content" source="media/register-scan-oracle-source/register-sources-2.png" alt-text="Opciones de registro de orígenes" border="true":::

## <a name="creating-and-running-a-scan"></a>Creación y ejecución de un examen

Para crear y ejecutar un nuevo examen, siga estos pasos:

1.  En el centro de administración, haga clic en Integration runtimes (Entornos de ejecución de integración). Asegúrese de que está configurado un entorno de ejecución de integración autohospedado. Si no lo está, use los pasos que se indican [aquí](./manage-integration-runtimes.md) para crear un entorno de ejecución de integración autohospedado.

2.  Vaya a **Sources** (Orígenes).

3.  Seleccione el origen de Oracle registrado.

4.  Seleccione **+ New scan** (+ Nuevo examen).

5.  Especifique los detalles siguientes:

    a.  **Name** (Nombre): el nombre del examen.

    b.  **Connect via integration runtime** (Conectar mediante IR): seleccione el entorno de ejecución de integración configurado.

    c.  **Credential** (Credencial): seleccione la credencial para conectarse al origen de datos. Asegúrese de que:
    - Selecciona la autenticación básica al crear una credencial.        
    - Proporciona el nombre de usuario que usa JDBC para conectarse al servidor de bases de datos en el campo de entrada de nombre de usuario.        
    - Almacena la contraseña de usuario que usa JDBC para conectarse al servidor de bases de datos en la clave secreta.

    d.  **Schema** (Esquema): indique el subconjunto de esquemas que se va a importar expresado como una lista separada por puntos y coma. Por ejemplo, schema1; schema2. Si esa lista está vacía, se importan todos los esquemas del usuario. De forma predeterminada, se ignoran todos los esquemas del sistema (por ejemplo, SysAdmin) y los objetos. Si la lista está vacía, se importan todos los esquemas disponibles.
        Los patrones de nombres de esquema aceptables que usan la sintaxis de expresiones SQL LIKE incluyen el uso de %, por ejemplo, A%; %B; %C%; D
       -   empieza por A o        
       -   termina en B o        
       -   contiene C o        
       -   igual a D

    No se acepta el empleo de NOT ni de caracteres especiales.

6.  **Driver location** (Ubicación del controlador): especifique la ruta de acceso a la ubicación del controlador JDBC en la máquina virtual donde se ejecuta el entorno de ejecución de integración autohospedado. Debe ser la ruta de acceso a la ubicación válida de la carpeta JAR.
    > [!Note]
    > Todas las cuentas de la máquina virtual deben poder acceder al controlador. No lo instale en una cuenta de usuario.

7.  **Maximum memory available** (Memoria máxima disponible): memoria máxima (en GB) disponible en la máquina virtual del cliente que van a usar los procesos de examen. Depende del tamaño del origen de SAP S/4HANA que se va a examinar.

    > [!Note] 
    > Como regla general, especifique 1 GB de memoria por cada 1000 tablas.

    :::image type="content" source="media/register-scan-oracle-source/scan.png" alt-text="Examen de Oracle" border="true":::

8.  Haga clic en **Continuar**.

9.  Elija el **desencadenador del examen**. Puede configurar una programación o ejecutar el examen una vez.

10.  Revise el examen y haga clic en **Save and run** (Guardar y ejecutar).

## <a name="viewing-your-scans-and-scan-runs"></a>Visualización de los exámenes y las ejecuciones de exámenes

1. Vaya al centro de administración. Seleccione **Data sources** (Orígenes de datos) en la sección **Sources and scanning** (Orígenes y examen).

2. Seleccione el origen de datos que desee. Verá una lista de los exámenes existentes en ese origen de datos.

3. Seleccione el examen cuyos resultados le interesa ver.

4. En esta página se muestran todas las ejecuciones de exámenes anteriores junto con las métricas y el estado de cada ejecución del examen. También mostrará si el análisis se ha programado o es manual, a cuántos recursos se han aplicado clasificaciones, cuántos recursos totales se han detectado, la hora de inicio y finalización del examen y la duración total del examen.

## <a name="manage-your-scans"></a>Administración de exámenes

Para administrar o eliminar un examen, haga lo siguiente:

1. Vaya al centro de administración. Seleccione **Data sources** (Orígenes de datos) en la sección **Sources and scanning** (Orígenes y examen) y, a continuación, seleccione el origen de datos deseado.

2. Seleccione el examen que desea administrar. Seleccione **Edit** (Editar) para editar el examen.

3. Seleccione **Delete** (Eliminar) para eliminar el examen.

## <a name="next-steps"></a>Pasos siguientes

- [Examen del catálogo de datos de Azure Purview](how-to-browse-catalog.md)
- [Búsqueda en el catálogo de datos de Azure Purview](how-to-search-catalog.md)