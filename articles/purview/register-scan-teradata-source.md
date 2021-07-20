---
title: Registro de un origen de Teradata y configuración de los exámenes en Azure Purview
description: En este artículo se describe cómo registrar un origen de Teradata en Azure Purview y cómo configurar un examen.
author: chandrakavya
ms.author: kchandra
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: overview
ms.date: 2/25/2021
ms.openlocfilehash: 7dae716944426ce21cb81bdbb36992238e4bb227
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112298446"
---
# <a name="register-and-scan-teradata-source"></a>Registro y examen de un origen de Teradata

En este artículo se describe cómo registrar un origen de Teradata en Purview y cómo configurar un examen.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

El origen de Teradata admite **Examen completo** para extraer metadatos de una base de datos de Teradata y captura el **linaje** entre los recursos de datos.

## <a name="prerequisites"></a>Requisitos previos

1.  Configure la versión más reciente del [entorno de ejecución de integración autohospedado](https://www.microsoft.com/download/details.aspx?id=39717).
    Para obtener más información, vea [Creación y configuración de un entorno de ejecución de integración autohospedado](../data-factory/create-self-hosted-integration-runtime.md).

2.  Asegúrese de que [JDK 11](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) está instalado en la máquina virtual donde también lo está el entorno de ejecución de integración autohospedado.

3.  Asegúrese de que \"Visual C++ Redistributable 2012 Update 4\" está instalado en la máquina del entorno de ejecución de integración autohospedado. Si aún no lo está, descárguelo desde [aquí](https://www.microsoft.com/download/details.aspx?id=30679).

4.  Tiene que descargar manualmente el controlador JDBC de Teradata en la máquina virtual donde se ejecuta el entorno de ejecución de integración autohospedado.
    El archivo JAR ejecutable se puede descargar desde el sitio web de [Teradata](https://downloads.teradata.com/).

    > [!Note]
    > Todas las cuentas de la máquina virtual deben poder acceder al controlador. No lo instale en una cuenta de usuario.

5.  Las versiones de la base de datos de Teradata compatibles son 12.x a 16.x. Asegúrese de tener acceso de lectura al origen de Teradata que se está examinando.

## <a name="setting-up-authentication-for-a-scan"></a>Configuración de la autenticación para un examen

La única autenticación admitida en un origen de Teradata es la **autenticación básica**.

## <a name="register-a-teradata-source"></a>Registro de un origen de Teradata

Para registrar un nuevo origen de Teradata en el catálogo de datos, siga estos pasos:

1.  Vaya a la cuenta de Purview.
2.  Seleccione **Sources** (Orígenes) en el panel de navegación izquierdo.
3.  Seleccione **Registrar**.
4.  En Register sources (Registrar orígenes), seleccione **Teradata**. Seleccione **Continuar**

    :::image type="content" source="media/register-scan-teradata-source/register-sources.png" alt-text="Registro de opciones de Teradata" border="true":::

En la pantalla **Registrar orígenes** [Registrar orígenes (Teradata)], haga lo siguiente:

1.  Escriba un **nombre** con el que se mostrará el origen de datos en el catálogo.

2.  Escriba el nombre del **host** para conectarse a un origen de Teradata. También puede ser una dirección IP o una cadena de conexión completa al servidor.

3.  Seleccione una colección o cree una nueva (opcional).

4.  Seleccione Finish (Finalizar) para registrar el origen de datos.

    :::image type="content" source="media/register-scan-teradata-source/register-sources-2.png" alt-text="register Teradata" border="true":::

## <a name="creating-and-running-a-scan"></a>Creación y ejecución de un examen

Para crear y ejecutar un nuevo examen, siga estos pasos:

1.  En el centro de administración, haga clic en **Integration runtimes** (Entornos de ejecución de integración). Asegúrese de que está configurado un entorno de ejecución de integración autohospedado. Si no lo está, use los pasos que se indican [aquí](./manage-integration-runtimes.md) para configurar un entorno de ejecución de integración autohospedado.

2.  Seleccione la pestaña **Mapa de datos** en el panel izquierdo de Purview Studio.

3.  Seleccione el origen de Teradata registrado.

4.  Seleccione **New scan** (Nuevo examen).

5.  Especifique los detalles siguientes:

    a.  **Name** (Nombre): el nombre del examen.

    b.  **Connect via integration runtime** (Conectar mediante el entorno de ejecución de integración): seleccione el entorno de ejecución de integración autohospedado configurado.

    c.  **Credential** (Credencial): seleccione la credencial para conectarse al origen de datos. Asegúrese de que:

    -   Selecciona la autenticación básica al crear una credencial.
    -   Proporciona un nombre de usuario para conectarse al servidor de bases de datos en el campo de entrada de nombre de usuario.
    -   Almacena la contraseña del servidor de bases de datos en la clave secreta.

        Para obtener más información sobre credenciales, vea el vínculo que se indica [aquí](./manage-credentials.md).

6.  **Schema** (Esquema): indique el subconjunto de esquemas que se va a importar expresado como una lista separada por puntos y coma. Por ejemplo, schema1; schema2. Si esa lista está vacía, se importan todos los esquemas del usuario. De forma predeterminada, se ignoran todos los esquemas del sistema (por ejemplo, SysAdmin) y los objetos. Si la lista está vacía, se importan todos los esquemas disponibles.

    Los patrones de nombres de esquemas aceptables que usan la sintaxis de expresiones SQL LIKE incluyen el uso de %, por ejemplo, A%; %B; %C%; D
    - empieza por A o    
    - termina en B o    
    - contiene C o    
    - igual a D

    No se permite usar NOT ni caracteres especiales.

7.  **Driver location** (Ubicación del controlador): especifique la ruta de acceso a la ubicación del controlador JDBC en la máquina virtual donde se ejecuta el entorno de ejecución de integración autohospedado. Debe ser la ruta de acceso a la ubicación válida de la carpeta JAR.

8.  **Maximum memory available** (Memoria máxima disponible): memoria máxima (en GB) disponible en la máquina virtual del cliente que van a usar los procesos de examen. Depende del tamaño del origen de Teradata que se va a examinar.

    > [!Note] 
    > Como regla general, especifique 2 GB de memoria por cada 1000 tablas

    :::image type="content" source="media/register-scan-teradata-source/setup-scan.png" alt-text="examen de configuración" border="true":::

6.  Haga clic en **Continuar**.

7.  Elija el **desencadenador del examen**. Puede configurar una programación o ejecutar el examen una vez.

8.  Revise el examen y haga clic en **Save and run** (Guardar y ejecutar).

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