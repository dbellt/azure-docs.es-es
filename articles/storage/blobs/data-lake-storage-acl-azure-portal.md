---
title: Uso de Azure Portal para administrar ACL en Azure Data Lake Storage Gen2
description: Use Azure Portal para administrar listas de control de acceso (ACL) en cuentas de almacenamiento que tengan habilitado el espacio de nombres jerárquico (HNS).
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 04/15/2021
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: 623395ad09fd29d7102657f93a24085d388485a0
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109635371"
---
# <a name="use-the-azure-portal-to-manage-acls-in-azure-data-lake-storage-gen2"></a>Uso de Azure Portal para administrar ACL en Azure Data Lake Storage Gen2

En este artículo se muestra cómo usar [Azure Portal](https://ms.portal.azure.com/) para administrar la lista de control de acceso (ACL) de un directorio o blob en cuentas de almacenamiento que tienen habilitado el espacio de nombres jerárquico. 

Para obtener información sobre la estructura de la ACL, vea [Listas de control de acceso en Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).

Para obtener información sobre cómo usar listas de control de acceso y reglas de Azure en conjunto, vea [Modelo de control de acceso de Azure Data Lake Storage Gen2](data-lake-storage-access-control-model.md).

## <a name="prerequisites"></a>Requisitos previos

- Suscripción a Azure. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

- Una cuenta de almacenamiento con la característica de espacio de nombres jerárquico habilitada. Siga [estas](create-data-lake-storage-account.md) instrucciones para crear uno.

- Debe tener uno de los siguientes permisos de seguridad:

  - Se ha asignado a su identidad de usuario el rol [Propietario de datos de blobs de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) en el ámbito del contenedor de destino, la cuenta de almacenamiento el grupo de recursos primario o la suscripción.  

  - Es el usuario propietario del contenedor de destino, el directorio o el blob al que pretende aplicar la configuración de ACL. 
  
## <a name="manage-an-acl"></a>Administración de una lista de control de acceso

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) para empezar a trabajar.

2. Busque la cuenta de almacenamiento y muestre la información general de la cuenta.

3. Seleccione **Contenedores** en **Almacenamiento de datos**.
   
   Aparecen los contenedores de la cuenta de almacenamiento. 

   > [!div class="mx-imgBorder"]
   > ![ubicación de los contenedores de la cuenta de almacenamiento en Azure Portal](./media/data-lake-storage-acl-azure-portal/find-containers-in-azure-portal.png)

5. Vaya al contenedor, directorio o blob. Haga clic con el botón derecho en el objeto y, a continuación, seleccione **Manage ACL** (Administrar ACL).

   > [!div class="mx-imgBorder"]
   > ![menú contextual para administrar una acl](./media/data-lake-storage-acl-azure-portal/manage-acl-menu-item.png)

   Aparece la pestaña **Access permissions** (Permisos de acceso) de la página para **administrar la ACL**. Use los controles de esta pestaña para administrar el acceso al objeto. 

   > [!div class="mx-imgBorder"]
   > ![acceso a la pestaña de ACL de la página para administrar la ACL](./media/data-lake-storage-acl-azure-portal/access-acl-page.png)

7. Para agregar una *entidad de seguridad* a la ACL, seleccione el botón **Agregar entidad de seguridad**. 

   > [!TIP]
   > Una entidad de seguridad es un objeto que representa a un usuario, un grupo, una entidad de servicio o una identidad administrada que está definido en Azure Active Directory (AD). 

   Busque la entidad de seguridad mediante el cuadro de búsqueda y, a continuación, haga clic en el botón **Seleccionar**. 

   > [!div class="mx-imgBorder"]
   > ![Adición de una entidad de seguridad a la ACL](./media/data-lake-storage-acl-azure-portal/get-security-principal.png)

   > [!NOTE]
   > Se recomienda crear un grupo de seguridad en Azure AD y, a continuación, mantener los permisos en el grupo en lugar de para usuarios individuales. Para obtener más detalles sobre esta recomendación, así como otros procedimientos recomendados, consulte los [Modelo de control de acceso de Azure Data Lake Storage Gen2](data-lake-storage-explorer-acl.md).

8. Para administrar la *ACL predeterminada*, seleccione la pestaña **Permisos predeterminados** y, a continuación, active la casilla **Configure default permissions** (Configurar permisos predeterminados). 

   > [!TIP]
   > Una ACL predeterminada es una plantilla de una ACL que determina las ACL de acceso para los elementos secundarios que se crean en un directorio. Un blob no tiene una ACL predeterminada, por lo que esta pestaña solo aparece para los directorios. 

   > [!div class="mx-imgBorder"]
   > ![pestaña de ACL de la página para administrar la ACL](./media/data-lake-storage-acl-azure-portal/default-acl-page.png)

## <a name="apply-an-acl-recursively"></a>Aplicación de una ACL de forma recursiva

Puede aplicar entradas de ACL de forma recursiva en los elementos secundarios existentes de un directorio primario sin tener que realizar estos cambios individualmente para cada elemento secundario. Sin embargo, no se pueden aplicar entradas de ACL de forma recursiva mediante Azure Portal. 

Para aplicar las ACL de forma recursiva, use el Explorador de Azure Storage, PowerShell o la CLI de Azure. Si prefiere escribir código, también puede usar las API de .NET, Java, Python o Node.js. 

Puede encontrar la lista completa de guías aquí: [Establecimiento de las ACL](data-lake-storage-access-control.md#how-to-set-acls). 

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el modelo de permisos de Data Lake Storage Gen2.

> [!div class="nextstepaction"]
> [Modelo de control de acceso de Azure Data Lake Storage Gen2](./data-lake-storage-access-control-model.md)