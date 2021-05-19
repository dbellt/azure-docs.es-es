---
title: Examen de varios orígenes en Azure Purview
description: Aprenda a examinar una suscripción o un grupo de recursos completo de Azure en el catálogo de datos de Azure Purview.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 05/08/2021
ms.openlocfilehash: ce78b36a07a0648e869a2a382554fdc41b408f96
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109656333"
---
# <a name="register-and-scan-multiple-sources-in-azure-purview"></a>Registro y examen de varios orígenes en Azure Purview

En este artículo se explica cómo registrar varios orígenes (suscripciones o grupos de recursos de Azure) en Azure Purview y cómo configurar un examen en él.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Puede examinar varios orígenes para capturar metadatos y esquemas en la mayoría de los tipos de recursos de Azure que admite Azure Purview. Azure Purview también clasifica los datos automáticamente en función de las reglas de clasificación personalizadas y del sistema.

## <a name="prerequisites"></a>Prerequisites

- Antes de registrar los orígenes de datos, cree una cuenta de Azure Purview. Para obtener más información, consulte [Guía de inicio rápido: Creación de una cuenta de Azure Purview](create-catalog-portal.md).
- Asegúrese de que es el administrador de orígenes de datos de Azure Purview. Asimismo, debe ser el propietario o el administrador de acceso de usuarios para agregar un rol a la suscripción o al grupo de recursos.
- Configure la autenticación tal como se explica en las secciones siguientes.

### <a name="set-up-authentication-for-enumerating-resources-under-a-subscription-or-resource-group"></a>Configuración de la autenticación para enumerar recursos en una suscripción o un grupo de recursos

1. Vaya a la suscripción o el grupo de recursos en Azure Portal.  
1. Seleccione  **Control de acceso (IAM)**   en el menú izquierdo. 
1. Seleccione **+Agregar**. 
1. En el cuadro **Seleccionar entrada**, seleccione el rol **Lector** y escriba el nombre de la cuenta de Azure Purview (que representa el nombre de archivo MSI). 
1. Haga clic en **Guardar** para finalizar la asignación de roles.

### <a name="set-up-authentication-to-scan-resources-under-a-subscription-or-resource-group"></a>Configuración de la autenticación para examinar recursos en una suscripción o un grupo de recursos

Existen dos maneras de configurar la autenticación de varios orígenes en Azure:

- Identidad administrada
- Entidad de servicio

Debe configurar la autenticación en cada recurso de la suscripción o del grupo de recursos que quiere registrar y examinar. Los tipos de recursos de Azure Storage (Azure Blob Storage y Azure Data Lake Storage Gen2) le facilitarán este trabajo, ya que le permiten agregar el archivo MSI o la entidad de servicio en el nivel de suscripción o grupo de recursos como un lector de datos de blobs de almacenamiento. A continuación, los permisos se aplican en cada cuenta de almacenamiento de la suscripción o del grupo de recursos. En cuanto al resto de tipos de recursos, debe aplicar el archivo MSI o la entidad de servicio en cada recurso, o crear una secuencia de comandos para hacerlo. 

Para saber cómo agregar permisos en cada tipo de recurso dentro de una suscripción o grupo de recursos, consulte los siguientes recursos:
    
- [Azure Blob Storage](register-scan-azure-blob-storage-source.md#setting-up-authentication-for-a-scan)
- [Azure Data Lake Storage Gen1](register-scan-adls-gen1.md#setting-up-authentication-for-a-scan)
- [Azure Data Lake Storage Gen2](register-scan-adls-gen2.md#setting-up-authentication-for-a-scan)
- [Azure SQL Database](register-scan-azure-sql-database.md)
- [Instancia administrada de Azure SQL](register-scan-azure-sql-database-managed-instance.md#setting-up-authentication-for-a-scan)
- [Azure Synapse Analytics](register-scan-azure-synapse-analytics.md#setting-up-authentication-for-a-scan)
 
## <a name="register-multiple-sources"></a>Registro de varios orígenes

Para registrar varios orígenes nuevos en el catálogo de datos, haga lo siguiente:

1. Vaya a la cuenta de Azure Purview.
1. Seleccione **Orígenes** en el menú izquierdo.
1. Seleccione **Registrar**.
1. En **Registrar orígenes**, seleccione **Azure (varios)** .

   :::image type="content" source="media/register-scan-azure-multiple-sources/register-azure-multiple.png" alt-text=" Captura de pantalla que muestra el icono de Azure Multiple en la pantalla para registrar varios orígenes.":::
1. Seleccione **Continuar**.
1. En la pantalla **Registrar orígenes (Azure)** , haga lo siguiente:

   1. En el cuadro **Nombre**, escriba un nombre con el que se enumerará el origen de datos en el catálogo. 
   1. En el cuadro **Grupo de administración**, elija la opción de un grupo de administración para realizar el filtrado.
   1. En los cuadros de lista desplegable **Suscripción** y **Grupo de recursos**, seleccione una suscripción o un grupo de recursos específico, respectivamente. El ámbito de registro se establece en la suscripción o el grupo de recursos seleccionados.  

      :::image type="content" source="media/register-scan-azure-multiple-sources/azure-multiple-source-setup.png" alt-text="Captura de pantalla que muestra los cuadros para seleccionar una suscripción y un grupo de recursos.":::
   1. En el cuadro **Seleccionar una colección**, seleccione una colección o cree una nueva (opcional).
   1. Seleccione **Registrar** para registrar los orígenes de datos.


## <a name="create-and-run-a-scan"></a>Creación y ejecución de un trabajo

Para crear y ejecutar un nuevo examen, siga estos pasos:

1. Vaya a la sección **Orígenes**.
1. Seleccione el origen de datos que ha registrado.
1. Seleccione **Ver detalles** >  **+ Nuevo examen**, o use el icono de acción rápida **Escanear** en el mosaico de origen.
1. En **Nombre**, escriba el nombre.
1. En **Tipo**, seleccione los tipos de recursos que quiere examinar dentro de este origen. Elija una de estas opciones:

    - Déjela como **Todos**. Esta selección incluye los tipos de recursos futuros que pueden no existir actualmente en esa suscripción o ese grupo de recursos.
    - Use los cuadros para seleccionar específicamente los tipos de recursos que quiere examinar. Si elige esta opción, los tipos de recursos futuros que puedan crearse en esta suscripción o este grupo de recursos no se van a incluir en los exámenes, a menos que el examen se edite explícitamente en el futuro.
    
    :::image type="content" source="media/register-scan-azure-multiple-sources/multiple-source-scan.png" alt-text="Captura de pantalla que muestra las opciones para examinar varios orígenes.":::

1. Seleccione la credencial para conectarse a los recursos del origen de datos: 
    - Puede seleccionar una credencial en el nivel principal como un archivo MSI, o bien puede seleccionar una credencial para un tipo de entidad de servicio determinado. A continuación, puede usar esa credencial para todos los tipos de recursos de la suscripción o el grupo de recursos.
    - También puede seleccionar el tipo de recurso específicamente y aplicar otra credencial para ese tipo de recurso.
    
    Cada credencial se considera el método de autenticación de todos los recursos de un tipo determinado. Debe establecer la credencial elegida en los recursos para examinarlos correctamente, tal como se explicó [anteriormente en este artículo](#set-up-authentication-to-scan-resources-under-a-subscription-or-resource-group).
1. En cada tipo, puede optar por examinar todos los recursos o un subconjunto de ellos en función del nombre:
    - Si deja la opción como **Todos**, los recursos futuros de ese tipo también se examinarán en próximas ejecuciones de exámenes.
    - Si selecciona cuentas de almacenamiento o bases de datos SQL específicas, los recursos futuros de ese tipo creados en esta suscripción o grupo de recursos no se incluirán en los exámenes, a menos que el examen se edite de manera explícita en el futuro.
 
1. Haga clic en **Continuar** para seguir. Azure Purview prueba el acceso para comprobar si ha aplicado el archivo MSI de Azure Purview como lector en la suscripción o el grupo de recursos. Si recibe un mensaje de error, siga [estas instrucciones](#set-up-authentication-for-enumerating-resources-under-a-subscription-or-resource-group) para resolverlo.

1. Seleccione los conjuntos de reglas de examen para cada tipo de recurso elegido en el paso anterior. También puede crear conjuntos de reglas de examen insertados.
  
   :::image type="content" source="media/register-scan-azure-multiple-sources/multiple-scan-rule-set.png" alt-text="Captura de pantalla que muestra las reglas de examen para cada tipo de recurso.":::

1. Elija el desencadenador del examen. Puede programarlo para que se ejecute semanalmente, mensualmente o una vez.

1. Revise el examen y seleccione **Guardar** para completar la configuración. 

## <a name="view-your-scans-and-scan-runs"></a>Visualización de los exámenes y las ejecuciones de exámenes

1. Para ver los detalles del origen, haga clic en **Ver detalles** en el icono de la sección de **Orígenes**. 

    :::image type="content" source="media/register-scan-azure-multiple-sources/multiple-source-detail.png" alt-text="Captura de pantalla que muestra los detalles del origen."::: 

1. Para ver los detalles de la ejecución del examen, vaya a la página **Detalles del examen**.
   
    La *barra de estado* es un breve resumen sobre el estado de ejecución de los recursos secundarios. Aparece en el nivel de la suscripción o del grupo de recursos. Los colores significan lo siguiente:
    
    - Verde: el examen se ha realizado correctamente.
    - Rojo: se produjo un error en el examen. 
    - Gris: el examen sigue en curso.
   
    Puede seleccionar cada examen para ver detalles más completos.

    :::image type="content" source="media/register-scan-azure-multiple-sources/multiple-scan-full-details.png" alt-text="Captura de pantalla que muestra los detalles del examen.":::

1. Vea un resumen de las ejecuciones de exámenes recientes con errores en la parte inferior de los detalles del origen. También puede ver detalles más precisos relacionados con estas ejecuciones.

## <a name="manage-your-scans-edit-delete-or-cancel"></a>Administración de exámenes: edición, eliminación o cancelación
Para administrar un examen, haga lo siguiente:

1. Vaya al centro de administración.
1. Seleccione **Orígenes de datos** en la sección **Orígenes y exámenes** y, a continuación, seleccione el origen de datos deseado.
1. Seleccione el examen que quiere administrar. Después: 

   - Seleccione **Edit** (Editar) para editar el examen.
   - Puede eliminar el examen si selecciona **Eliminar**.
   - Si el examen se está ejecutando, puede cancelarlo seleccionando **Cancelar**.

## <a name="next-steps"></a>Pasos siguientes

- [Navegación por el catálogo de datos de Azure Purview](how-to-browse-catalog.md)
- [Búsqueda en el catálogo de datos de Azure Purview](how-to-search-catalog.md)    
