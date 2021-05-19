---
title: Cómo examinar Azure Storage Blob
description: Aprenda a examinar Azure Storage Blob en el catálogo de datos de Azure Purview.
author: shsandeep123
ms.author: sandeepshah
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 05/08/2021
ms.openlocfilehash: 9ae996d344604ebdaf4e166f6c0536eb6acb169b
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109656618"
---
# <a name="register-and-scan-azure-blob-storage"></a>Registro y examen de Azure Blob Storage

En este artículo se describe cómo registrar una cuenta de Azure Blob Storage en Purview y cómo configurar un examen.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Azure Blob Storage admite exámenes completos e incrementales para capturar los metadatos y el esquema. También clasifica los datos automáticamente en función de las reglas de clasificación personalizadas y del sistema.

## <a name="prerequisites"></a>Requisitos previos

- Antes de registrar los orígenes de datos, cree una cuenta de Azure Purview. Para más información sobre cómo crear una cuenta de Purview, consulte [Inicio rápido: creación de una cuenta de Azure Purview](create-catalog-portal.md).
- Tenga en cuenta que debe ser administrador de los orígenes de datos de Azure Purview.

## <a name="setting-up-authentication-for-a-scan"></a>Configuración de la autenticación para un examen

Hay tres maneras de configurar la autenticación de Azure Blob Storage:

- Identidad administrada
- Clave de cuenta
- Entidad de servicio

### <a name="managed-identity-recommended"></a>Identidad administrada (recomendada)

Al elegir la **Identidad administrada** para configurar la conexión, primero debe conceder a la cuenta de Purview el permiso para analizar el origen de datos:

1. Vaya a la cuenta de almacenamiento.
1. Seleccione **Access Control (IAM)** (Control de acceso [IAM]) en el menú de navegación izquierdo. 
1. Seleccione **+Agregar**.
1. Establezca el **Rol** en el **Lector de datos de blobs de almacenamiento** y escriba el nombre de la cuenta de Azure Purview en el cuadro de entrada **Seleccionar**. A continuación, seleccione **Save** (Guardar) para dar esta asignación de rol a su cuenta de Purview.

> [!Note]
> Para obtener más información, consulte los pasos que se detallan en [Autorización del acceso a blobs y colas con Azure Active Directory](../storage/common/storage-auth-aad.md).

### <a name="account-key"></a>Clave de cuenta

Cuando el método de autenticación seleccionado es **Clave de cuenta**, debe obtener la clave de acceso y almacenarla en el almacén de claves:

1. Vaya a la cuenta de almacenamiento.
1. Seleccione **Configuración > Claves de acceso**.
1. Copie la *clave* y guárdela para usarla en los pasos siguientes.
1. Vaya a almacén de claves.
1. Seleccione **Configuración > Secretos**.
1. Seleccione **+ Generar/Importar** y escriba el **nombre** y el **valor** como la *clave* de la cuenta de almacenamiento.
1. Seleccione **Crear** para completar la acción.
1. Si el almacén de claves no está conectado todavía a Purview, necesitará [crear una conexión del almacén de claves](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Por último, [cree una nueva credencial](manage-credentials.md#create-a-new-credential) mediante la clave para configurar el examen.

### <a name="service-principal"></a>Entidad de servicio

Puede usar una entidad de servicio existente o crear una. 

> [!Note]
> Si tiene que crear una entidad de servicio, siga estos pasos:
> 1. Acceda a [Azure Portal](https://portal.azure.com).
> 1. Seleccione **Azure Active Directory** en el menú de la izquierda.
> 1. Seleccione **App registrations** (Registros de aplicaciones).
> 1. Seleccione **+ Nuevo registro de aplicaciones**.
> 1. Escriba un nombre para la **aplicación** (el nombre de la entidad de servicio).
> 1. Seleccione **Solo las cuentas de este directorio organizativo**.
> 1. En el URI de redirección, seleccione **Web** y escriba cualquier dirección URL; no tiene que ser real ni del trabajo.
> 1. Después, seleccione **Register** (Registrar).

Es necesario obtener el id. de aplicación y el secreto de la entidad de servicio:

1. Vaya a la entidad de servicio en [Azure Portal](https://portal.azure.com).
1. Copie los valores de **Id. de aplicación (cliente)** de **Información general** y **Secreto de cliente** que están en **Certificados y secretos**.
1. Vaya a almacén de claves.
1. Seleccione **Configuración > Secretos**.
1. Seleccione **+ Generar/Importar** y escriba el **nombre** que quiera y el **valor** como **Secreto de cliente** de la entidad de servicio.
1. Seleccione **Crear** para completar la acción.
1. Si el almacén de claves no está conectado todavía a Purview, necesitará [crear una conexión del almacén de claves](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Por último, [cree una credencial](manage-credentials.md#create-a-new-credential) mediante la entidad de servicio para configurar el examen.

#### <a name="granting-the-service-principal-access-to-your-blob-storage"></a>Acceso de la entidad de servicio al almacenamiento de blobs

1. Vaya a la cuenta de almacenamiento.
1. Seleccione **Access Control (IAM)** (Control de acceso [IAM]) en el menú de navegación izquierdo. 
1. Seleccione **+Agregar**.
1. Establezca el **Rol** en el **Lector de datos de blobs de almacenamiento** y escriba el nombre de entidad de seguridad de servicio o el id. del objeto en el cuadro de entrada **Seleccionar**. A continuación, seleccione **Guardar** para dar esta asignación de rol a la entidad de servicio.

## <a name="firewall-settings"></a>Configuración de firewall

> [!NOTE]
> Si tiene el firewall habilitado para la cuenta de almacenamiento, debe usar el método de autenticación de **Identidad administrada** al configurar un examen.

1. Vaya a la cuenta de almacenamiento en [Azure Portal](https://portal.azure.com).
1. Vaya a **Configuración > Redes**.
1. Seleccione **Redes seleccionadas** en **Permitir acceso desde**.
1. En la sección del **Firewall**, seleccione **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento** y haga clic en **Guardar**.

:::image type="content" source="./media/register-scan-azure-blob-storage-source/firewall-setting.png" alt-text="Captura de pantalla que muestra la configuración del firewall":::

## <a name="register-an-azure-blob-storage-account"></a>Registro de una cuenta de Azure Blob Storage

Para registrar una nueva cuenta de blob en el catálogo de datos, haga lo siguiente:

1. Vaya a la cuenta de Purview.
1. Seleccione **Sources** (Orígenes) en el panel de navegación izquierdo.
1. Seleccione **Registrar**.
1. En **Registrar orígenes**, seleccione **Azure Blob Storage**.
1. Seleccione **Continuar**

En la pantalla **Registrar orígenes (Azure Blob Storage)** , haga lo siguiente:

1. Escriba un **nombre** con el que se mostrará el origen de datos en el catálogo. 
1. Elija la suscripción para filtrar las cuentas de almacenamiento.
1. Seleccione una cuenta de almacenamiento.
1. Seleccione una colección o cree una nueva (opcional).
1. Seleccione **Registrar** para registrar el origen de datos.

:::image type="content" source="media/register-scan-azure-blob-storage-source/register-sources.png" alt-text="Opciones de registro de orígenes" border="true":::

[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Examen del catálogo de datos de Azure Purview](how-to-browse-catalog.md)
- [Búsqueda en el catálogo de datos de Azure Purview](how-to-search-catalog.md)