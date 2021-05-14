---
title: Registro de exámenes de configuración y origen de SAP ECC (versión preliminar) en Azure Purview
description: En este artículo se describe cómo registrar un origen de SAP ECC en Azure Purview y cómo configurar un examen.
author: chandrakavya
ms.author: kchandra
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: overview
ms.date: 2/25/2021
ms.openlocfilehash: bc35f79bae0cd75b24b119e951f27b289ebef704
ms.sourcegitcommit: aaba99b8b1c545ad5d19f400bcc2d30d59c63f39
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2021
ms.locfileid: "108007365"
---
# <a name="register-and-scan-sap-ecc-source-preview"></a>Registro y examen de un origen de SAP ECC (versión preliminar)

En este artículo se describe cómo registrar un origen de SAP ECC en Purview y cómo configurar un examen.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

El origen de SAP ECC admite un **examen completo** para extraer metadatos de una instancia de SAP ECC y captura el **linaje** entre recursos de datos.

## <a name="prerequisites"></a>Requisitos previos

1.  Configure la versión más reciente del [entorno de ejecución de integración autohospedado](https://www.microsoft.com/download/details.aspx?id=39717).
    Para obtener más información, consulte [Creación y configuración de un entorno de ejecución de integración autohospedado](../data-factory/create-self-hosted-integration-runtime.md).

2.  Asegúrese de que el [JDK 11](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) esté instalado en la máquina virtual donde está instalado el entorno de ejecución de integración autohospedado.

3.  Asegúrese de que \"Visual C++ Redistributable 2012 Update 4\" está instalado en la máquina del entorno de ejecución de integración autohospedado. Si aún no está instalado, descárguelo [aquí](https://www.microsoft.com/download/details.aspx?id=30679).

4.  Descargue el [conector de SAP para Microsoft.NET 3.0](https://support.sap.com/en/product/connectors/msnet.html) de 64 bits del sitio web de SAP\' e instálelo en la máquina del entorno de ejecución de integración autohospedado. Al instalarlo, asegúrese de seleccionar la opción **Install Assemblies to GAC** (Instalar ensamblados en GAC) en la ventana **Optional setup steps** (Pasos de configuración opcionales).

    :::image type="content" source="media/register-scan-sapecc-source/requirement.png" alt-text="requisito previo" border="true":::

5.  El conector lee los metadatos de SAP mediante la versión 3.0 de la API del [conector de Java (JCo) para SAP](https://support.sap.com/en/product/connectors/jco.html). Asegúrese de que el conector de Java esté disponible en la máquina virtual donde está instalado el entorno de ejecución de integración autohospedado.
    Asegúrese de que usa la distribución de JCo correcta para su entorno. Por ejemplo, en una máquina de Microsoft Windows, asegúrese de que los archivos sapjco3.jar y sapjco3.dll estén disponibles.

    > [!Note] 
    > Todas las cuentas de la máquina virtual deben poder acceder al controlador. No lo instale en una cuenta de usuario.

6.  Implemente el módulo de función de ABAP de extracción de metadatos en el servidor SAP mediante los pasos indicados en la [guía de implementación de funciones de ABAP](abap-functions-deployment-guide.md). Necesitará una cuenta de desarrollador de ABAP para crear el módulo de función de RFC en el servidor SAP. La cuenta de usuario necesita los permisos suficientes para conectarse al servidor SAP y ejecutar los siguientes módulos de función de RFC:
    -   STFC_CONNECTION (comprobación de la conectividad)
    -   RFC_SYSTEM_INFO (comprobación de la información del sistema)


## <a name="setting-up-authentication-for-a-scan"></a>Configuración de la autenticación para un examen

La única autenticación admitida en un origen de SAP ECC es la **autenticación básica**.

## <a name="register-sap-ecc-source"></a>Registro de un origen de SAP ECC

Para registrar un nuevo origen de SAP ECC en el catálogo de datos, haga lo siguiente:

1.  Vaya a la cuenta de Purview.
2.  Seleccione **Sources** (Orígenes) en el panel de navegación izquierdo.
3.  Seleccione **Registrar**.
4.  En Register sources (Registrar orígenes), seleccione **SAP ECC**. Seleccione **Continue** (Continuar).

    :::image type="content" source="media/register-scan-sapecc-source/register-sapecc.png" alt-text="registro de opciones de SAP ECC" border="true":::

En la pantalla **Register sources (SAP ECC)** (Registrar orígenes [SAP ECC]), haga lo siguiente:

1.  Escriba un **nombre** con el que se mostrará el origen de datos en el catálogo.

2.  Escriba el nombre de **Application server** (Servidor de aplicaciones) para conectarse al origen de SAP ECC.
    También puede ser una dirección IP del host del servidor de aplicaciones de SAP.

3.  Escriba el **número del sistema** de SAP. Se trata de un entero de dos dígitos comprendido entre 00 y 99.

4.  Seleccione una colección o cree una nueva (opcional).

5.  Seleccione Finish (Finalizar) para registrar el origen de datos.

    :::image type="content" source="media/register-scan-sapecc-source/register-sapecc-2.png" alt-text="registro de SAP ECC" border="true":::

## <a name="creating-and-running-a-scan"></a>Creación y ejecución de un examen

Para crear y ejecutar un nuevo examen, siga estos pasos:

1.  En el centro de administración, haga clic en Integration runtimes (Entornos de ejecución de integración). Asegúrese de que está configurado un entorno de ejecución de integración autohospedado. Si no lo está, use los pasos que se indican [aquí](./manage-integration-runtimes.md) para crear un entorno de ejecución de integración autohospedado.

2.  Vaya a **Sources** (Orígenes).

3.  Seleccione el origen de SAP ECC registrado.

4.  Seleccione **+ New scan** (+ Nuevo examen).

5.  Especifique los detalles siguientes:

    a.  **Name** (Nombre): el nombre del examen.

    b.  **Connect via integration runtime** (Conectar mediante IR): seleccione el entorno de ejecución de integración configurado.

    c.  **Credential** (Credencial): seleccione la credencial para conectarse al origen de datos. Asegúrese de que:

    -   Selecciona la autenticación básica al crear una credencial.
    -   Proporciona un identificador de usuario para conectarse al servidor SAP en el campo de entrada de nombre de usuario.
    -   Almacena la contraseña de usuario usada para conectarse al servidor SAP en la clave secreta.

    d.  **Client ID** (Id. de cliente): escriba el identificador de cliente de SAP. Se trata de un número de tres dígitos comprendido entre 000 y 999.

    e.  **JCo library path** (Ruta de acceso de la biblioteca de JCo): ruta de acceso del directorio donde se encuentran las bibliotecas de JCo.

    f.  **Maximum memory available** (Memoria máxima disponible): memoria máxima (en GB) disponible en la máquina virtual del cliente que se va a usar en los procesos de examen. Depende del tamaño del origen de SAP ECC que se va a examinar.

    :::image type="content" source="media/register-scan-sapecc-source/scan-sapecc.png" alt-text="examen de SAPECC" border="true":::

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
