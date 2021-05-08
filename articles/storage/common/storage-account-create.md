---
title: Crear una cuenta de almacenamiento
titleSuffix: Azure Storage
description: Aprenda a crear una cuenta de almacenamiento para almacenar blobs, archivos, colas y tablas. Una cuenta de Azure Storage proporciona un espacio de nombres único en Microsoft Azure para leer y escribir los datos.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 04/23/2021
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 30766938526c2ba897c2aabc2513be5d6c3f01b5
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107949099"
---
# <a name="create-a-storage-account"></a>Creación de una cuenta de Storage

Una cuenta de Azure Storage contiene todos los objetos de datos de Azure Storage: blobs, archivos, colas y tablas. La cuenta de almacenamiento proporciona un espacio de nombres único para los datos de Azure Storage que es accesible desde cualquier lugar del mundo a través de HTTP o HTTPS. Para obtener más información sobre las cuentas de Azure Storage, consulte [Introducción a las cuentas de almacenamiento](storage-account-overview.md).

En este artículo de procedimientos se aprende a crear una cuenta de almacenamiento con [Azure Portal](https://portal.azure.com/), [Azure PowerShell](/powershell/azure/), la [CLI de Azure](/cli/azure) o una [plantilla de Azure Resource Manager](../../azure-resource-manager/management/overview.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Ninguno.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para crear una cuenta de Azure Storage con PowerShell, asegúrese de que ha instalado la versión 0.7 o posterior del [módulo Az de PowerShell](https://www.powershellgallery.com/packages/Az). Para obtener más información, consulte [Presentación del módulo Az de Azure PowerShell](/powershell/azure/new-azureps-module-az).

Para saber cuál es su versión actual, ejecute el comando siguiente:

```powershell
Get-InstalledModule -Name "Az"
```

Para instalar o actualizar Azure PowerShell, consulte [Instalación del módulo Azure PowerShell](/powershell/azure/install-az-ps).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Puede iniciar sesión en Azure y ejecutar los comandos de la CLI de Azure de dos maneras:

- Puede ejecutar los comandos de la CLI desde Azure Portal, en Azure Cloud Shell.
- Puede instalar la CLI y ejecutar sus comandos en local.

### <a name="use-azure-cloud-shell"></a>Uso de Azure Cloud Shell

Azure Cloud Shell es un shell de Bash gratuito que puede ejecutarse directamente en Azure Portal. La CLI de Azure está preinstalada y configurada para su uso con la cuenta. Haga clic en el botón **Cloud Shell** del menú situado en la sección superior derecha de Azure Portal:

[![Cloud Shell](./media/storage-quickstart-create-account/cloud-shell-menu.png)](https://portal.azure.com)

El botón inicia un shell interactivo que se puede usar para ejecutar los pasos de este artículo de procedimientos:

[![Captura de pantalla en la que aparece la ventana de Cloud Shell del portal](./media/storage-quickstart-create-account/cloud-shell.png)](https://portal.azure.com)

### <a name="install-the-cli-locally"></a>Instalación local de la CLI

También puede instalar y usar la CLI de Azure localmente. Los ejemplos de este artículo requieren la CLI de Azure, versión 2.0.4 o posterior. Ejecute `az --version` para buscar la versión instalada. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

# <a name="template"></a>[Plantilla](#tab/template)

Ninguno.

---

Inicie de sesión en Azure.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Inicie sesión en [Azure Portal](https://portal.azure.com).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Inicie sesión en la suscripción de Azure con el comando `Connect-AzAccount` y siga las instrucciones que aparecen en pantalla para autenticarse.

```powershell
Connect-AzAccount
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para iniciar Azure Cloud Shell, inicie sesión en [Azure Portal](https://portal.azure.com).

Para iniciar sesión en la instalación local de la CLI, ejecute el comando [az login](/cli/azure/reference-index#az_login):

```azurecli-interactive
az login
```

# <a name="template"></a>[Plantilla](#tab/template)

N/D

---

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Una cuenta de almacenamiento es un recurso de Azure Resource Manager. Resource Manager es el servicio de implementación y administración de Azure. Para más información, consulte [Información general de Azure Resource Manager](../../azure-resource-manager/management/overview.md).

Cada recurso de Resource Manager, incluida una cuenta de Azure Storage, debe pertenecer a un grupo de recursos de Azure. Un grupo de recursos es un contenedor lógico para agrupar servicios de Azure. Al crear una cuenta de almacenamiento, puede elegir entre crear un grupo de recursos o usar uno existente. En este artículo de información se muestra cómo crear un nuevo grupo de recursos.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Siga estos pasos para crear una cuenta de Azure Storage con Azure Portal:

1. En el menú izquierdo del portal, seleccione **Cuentas de almacenamiento** para mostrar una lista de las cuentas de almacenamiento.
1. En la página **Cuentas de almacenamiento**, seleccione **Nuevo**.

Las opciones de la nueva cuenta de almacenamiento se organizan en pestañas en la página **Creación de una cuenta de almacenamiento**. En las secciones siguientes se describe cada una de las pestañas y sus opciones.

### <a name="basics-tab"></a>Pestaña Datos básicos

En la pestaña **Aspectos básicos**, proporcione la información esencial de la cuenta de almacenamiento. Después de completar la pestaña **Aspectos básicos**, puede personalizar aún más la nueva cuenta de almacenamiento; para ello, establezca las opciones en las otras pestañas, o bien seleccione **Revisar y crear** para aceptar las opciones predeterminadas y continuar con la validación y la creación de la cuenta.

En la siguiente tabla se describen los campos de la pestaña **Aspectos básicos**.

| Sección | Campo | Obligatorio u opcional | Descripción |
|--|--|--|--|
| Detalles del proyecto | Subscription | Requerido | Seleccione la suscripción para la nueva cuenta de almacenamiento. |
| Detalles del proyecto | Resource group | Requerido | Cree un nuevo grupo de recursos para esta cuenta de almacenamiento o seleccione uno existente. Para obtener más información, vea el apartado [Grupos de recursos](../../azure-resource-manager/management/overview.md#resource-groups). |
| Detalles de instancia | Nombre de la cuenta de almacenamiento | Requerido | Seleccione un nombre único para la cuenta de almacenamiento. Los nombres de las cuentas de almacenamiento deben tener entre 3 y 24 caracteres y solo pueden incluir números y letras en minúscula. |
| Detalles de instancia | Region | Requerido | Seleccione la región adecuada para la cuenta de almacenamiento. Para obtener más información, consulte [Regiones y zonas de disponibilidad en Azure](../../availability-zones/az-overview.md).<br /><br />No todas las regiones son compatibles con todos los tipos de cuentas de almacenamiento o las configuraciones de redundancia. Para más información, vea [Redundancia de Azure Storage](storage-redundancy.md).<br /><br />Tenga en cuenta que la elección de la región puede tener un impacto en la facturación. Para obtener más información, vea [Facturación de la cuenta de almacenamiento](storage-account-overview.md#storage-account-billing). |
| Detalles de instancia | Rendimiento | Requerido | Seleccione rendimiento **estándar** para las cuentas de almacenamiento de uso general v2 (valor predeterminado). Microsoft recomienda este tipo de cuenta para la mayoría de los escenarios. Para obtener más información, consulte la sección [Tipos de cuentas de almacenamiento](storage-account-overview.md#types-of-storage-accounts).<br /><br />Seleccione **Prémium** para aquellos escenarios que requieran baja latencia. Después de seleccionar **Prémium,** seleccione el tipo de cuenta de almacenamiento prémium que quiera crear. Están disponibles los siguientes tipos de cuentas de almacenamiento: <ul><li>[Blobs en bloques](../blobs/storage-blob-performance-tiers.md)</li><li>[Recursos compartidos de archivos](../files/storage-files-planning.md#management-concepts)</li><li>[Blobs en páginas](../blobs/storage-blob-pageblob-overview.md)</li></ul> |
| Detalles de instancia | Redundancia | Requerido | Seleccione la configuración de redundancia que quiera. No todas las opciones de redundancia están disponibles para todos los tipos de cuentas de almacenamiento en todas las regiones. Para obtener más información sobre la configuración de redundancia, consulte la [Redundancia de Azure Storage](storage-redundancy.md).<br /><br />Si selecciona una configuración con redundancia geográfica (GRS o GZRS), los datos se replican en un centro de datos de una región diferente. Para obtener acceso de lectura a los datos de la región secundaria, seleccione **Hacer que el acceso de lectura a los datos esté disponible en caso de que no estén disponibles a nivel regional**. |

En la imagen siguiente se muestra una configuración estándar para una nueva cuenta de almacenamiento.

:::image type="content" source="media/storage-account-create/create-account-basics-tab.png" alt-text="Captura de pantalla que muestra una configuración estándar para una nueva cuenta de almacenamiento: pestaña de Aspectos básicos":::.

### <a name="advanced-tab"></a>Ficha Opciones avanzadas

En la pestaña **Opciones avanzadas**, puede configurar opciones adicionales y modificar la configuración predeterminada de la nueva cuenta de almacenamiento. Algunas de estas opciones también se pueden configurar después de crear la cuenta de almacenamiento, mientras que otras deben configurarse en el momento de la creación.

En la siguiente tabla se describen los campos de la ficha **Opciones avanzadas**.

| Sección | Campo | Obligatorio u opcional | Descripción |
|--|--|--|--|
| Seguridad | Habilitar la transferencia segura | Opcionales | Habilite la transferencia segura para requerir que las solicitudes entrantes a esta cuenta de almacenamiento solo se realicen a través del protocolo HTTPS (valor predeterminado). Esto se recomienda para una seguridad óptima. Para obtener más información, consulte [Requisito de transferencia segura para garantizar conexiones seguras](storage-require-secure-transfer.md). |
| Seguridad | Habilitar el cifrado de infraestructura | Opcionales | De forma predeterminada, se comprueba que el cifrado de infraestructura no está habilitado. Habilite el cifrado de infraestructura para cifrar los datos tanto en el nivel de servicio como en el nivel de infraestructura. Para obtener más información, consulte [Creación de una cuenta de almacenamiento con el cifrado de infraestructura habilitado para realizar el cifrado doble de datos](infrastructure-encryption-enable.md). |
| Seguridad | Habilitar el acceso público a blobs | Opcionales | Cuando se habilita, esta opción permite a un usuario con los permisos adecuados habilitar el acceso público anónimo a un contenedor en la cuenta de almacenamiento (valor predeterminado). Deshabilitar esta configuración impide todo acceso público anónimo a la cuenta de almacenamiento. Para más información, consulte el artículo en el que se explica cómo [impedir el acceso de lectura público anónimo a contenedores y blobs](../blobs/anonymous-read-access-prevent.md).<br> <br> Aunque se habilite el acceso público a los blobs, los datos del blob no estarán disponibles para el acceso público a menos que el usuario realice el paso adicional para configurar explícitamente la opción de acceso público del contenedor. |
| Seguridad | Habilitar el acceso a la clave de la cuenta de almacenamiento (versión preliminar) | Opcionales | Cuando se habilita, esta configuración permite a los clientes autorizar las solicitudes a la cuenta de almacenamiento mediante las claves de acceso de la cuenta o una cuenta de Azure Active Directory (Azure AD) (valor predeterminado). Si se deshabilita esta configuración, impedirá que se realice la autorización con las claves de acceso de la cuenta. Para obtener más información, consulte [Impedir la autorización con clave compartida para una cuenta de Azure Storage (versión preliminar)](shared-key-authorization-prevent.md). |
| Seguridad | Versión de TLS mínima | Requerido | Seleccione la versión mínima necesaria de la Seguridad de la capa de transporte (TLS) para las solicitudes entrantes de una cuenta de almacenamiento. El valor predeterminado es la versión 1.2 de TLS. Cuando esta opción se establece en el valor predeterminado, se rechazan las solicitudes entrantes realizadas con TLS 1.0 o TLS 1.1. Para obtener más información, consulte [Aplicación de una versión mínima necesaria de Seguridad de la capa de transporte (TLS) para las solicitudes a una cuenta de almacenamiento](transport-layer-security-configure-minimum-version.md). |
| Data Lake Storage Gen2 | Habilitar el espacio de nombres jerárquico | Opcionales | Para usar esta cuenta de almacenamiento para las cargas de trabajo de Azure Data Lake Storage Gen2, configure un espacio de nombres jerárquico. Para más información, vea [Introducción a Azure Data Lake Storage Gen2](../blobs/data-lake-storage-introduction.md). |
| Blob Storage | Habilitación del recurso compartido de archivos de red (NFS) v3 (versión preliminar) | Opcionales | NFS v3 proporciona compatibilidad con el sistema de archivos de Linux en cuanto a la escala de almacenamiento de objetos, y permite a los clientes de Linux montar un contenedor en la instancia de Blob Storage desde una máquina virtual (VM) de Azure o un equipo local. Para obtener más información, consulte [Compatibilidad del protocolo Network File System (NFS) 3.0 en Azure Blob Storage (versión preliminar)](../blobs/network-file-system-protocol-support.md). |
| Blob Storage | Nivel de acceso | Requerido | Los niveles de acceso a blobs permiten almacenar datos de blob de la manera más rentable, en función del uso. Seleccione el nivel de acceso frecuente (valor predeterminado) para los datos a los que se accede con frecuencia. Seleccione el nivel de acceso esporádico para los datos a los que se accede con poca frecuencia. Para obtener más información, vea [Niveles de acceso de Azure Blob Storage: frecuente, esporádico y archivo](../blobs/storage-blob-storage-tiers.md). |
| Azure Files | Habilitación de recursos compartidos de archivos grandes | Opcionales | Solo está disponible para las cuentas de almacenamiento prémium para los recursos compartidos de archivos. Para obtener más información, consulte [Habilitación de recursos compartidos de archivos estándar para incluir hasta 100 TiB](../files/storage-files-planning.md#enable-standard-file-shares-to-span-up-to-100-tib). |
| Tablas y colas | Habilitar la compatibilidad con claves administradas por el cliente | Opcionales | Para habilitar la compatibilidad con claves administradas por el cliente para tablas y colas, debe seleccionar esta configuración en el momento de crear la cuenta de almacenamiento. Para obtener más información, consulte [Creación de una cuenta que admita las claves administradas por el cliente para tablas y colas](account-encryption-key-create.md). |

### <a name="networking-tab"></a>Pestaña Redes

En la pestaña **Redes**, puede configurar la conectividad de red y las preferencias de enrutamiento para la nueva cuenta de almacenamiento. Estas opciones también se pueden configurar después de crear la cuenta de almacenamiento.

En la siguiente tabla se describen los campos de la ficha **Redes**.

| Sección | Campo | Obligatorio u opcional | Descripción |
|--|--|--|--|
| Conectividad de red | Método de conectividad | Requerido | De forma predeterminada, el tráfico de red entrante se enruta al punto de conexión público de su cuenta de almacenamiento. Puede especificar que el tráfico se debe enrutar al punto de conexión público a través de una red virtual de Azure. También puede configurar puntos de conexión privados para la cuenta de almacenamiento. Para obtener más información, consulte [Uso de puntos de conexión privados para Azure Storage](storage-private-endpoints.md). |
| Enrutamiento de red | Preferencia de enrutamiento | Requerido | La preferencia de enrutamiento de red especifica cómo se enruta el tráfico de red al punto de conexión público de la cuenta de almacenamiento desde los clientes a través de Internet. De forma predeterminada, una nueva cuenta de almacenamiento usa el enrutamiento de red de Microsoft. También puede elegir enrutar el tráfico de red a través del punto de acceso POP más cercano a la cuenta de almacenamiento, lo que puede reducir los costos de red. Para obtener más información, consulte [Preferencias de enrutamiento de red para Azure Storage](network-routing-preference.md). |

### <a name="data-protection-tab"></a>Pestaña Protección de datos

En la pestaña **Protección de datos**, puede configurar opciones de protección de datos para los datos de blobs en la nueva cuenta de almacenamiento.  Estas opciones también se pueden configurar después de crear la cuenta de almacenamiento. Para obtener información general sobre las opciones de protección de datos de Azure Storage, consulte [Información general sobre la protección de datos](../blobs/data-protection-overview.md).

En la tabla siguiente se describen los campos de la pestaña **Protección de datos**.

| Sección | Campo | Obligatorio u opcional | Descripción |
|--|--|--|--|
| Recuperación | Habilitar la restauración a un momento dado para contenedores | Opcionales | La restauración a un momento dado proporciona protección contra eliminaciones accidentales o daños, ya que le permite restaurar los datos de blobs en bloques a un estado anterior. Para obtener más información, consulte la [Restauración a un momento dado de los blobs en bloques](../blobs/point-in-time-restore-overview.md).<br /><br />La habilitación de la restauración a un momento dado también permite el control de versiones de blobs, la eliminación temporal de los blobs y la fuente de cambios de blobs. Estas características de requisitos previos pueden tener un impacto en el costo. Para obtener más información, consulte [Precios y facturación](../blobs/point-in-time-restore-overview.md#pricing-and-billing) de la restauración a un momento dado. |
| Recuperación | Habilitación de la eliminación temporal para blobs | Opcionales | La eliminación temporal de blobs protege a cada uno de los blobs, instantáneas o versiones de errores accidentales al borrar o sobrescribir los datos, ya que conserva en el sistema los datos eliminados durante el período de retención que se especifique. Durante el período de retención, puede restaurar un objeto eliminado temporalmente a su estado en el momento en que se eliminó. Para obtener más información, consulte [Eliminación temporal para los blobs](../blobs/soft-delete-blob-overview.md).<br /><br />Microsoft recomienda habilitar la eliminación temporal de blobs para las cuentas de almacenamiento y establecer un período de retención mínimo de siete días. |
| Recuperación | Habilitación de la eliminación temporal para contenedores (versión preliminar) | Opcionales | La eliminación temporal de contenedores protege los contenedores y su contenido de eliminaciones accidentales, ya que conserva en el sistema los datos eliminados durante el período de retención que se especifique. Durante el período de retención, puede restaurar un contenedor eliminado temporalmente a su estado en el momento en que se eliminó. Para más información, consulte [Eliminación temporal de contenedores (versión preliminar)](../blobs/soft-delete-container-overview.md).<br /><br />Microsoft recomienda habilitar la eliminación temporal de contenedores para las cuentas de almacenamiento y establecer un período de retención mínimo de siete días. |
| Recuperación | Habilitar la eliminación temporal para recursos compartidos de archivos | Opcionales | La eliminación temporal de recursos compartidos de archivos protege un recurso compartido de archivos y su contenido de eliminaciones accidentales al mantener los datos eliminados en el sistema durante un período de retención específico. Durante el período de retención, puede restaurar un recurso compartido de archivos eliminado temporalmente a su estado en el momento en que se eliminó. Para obtener más información, consulte [Prevención de la eliminación accidental de recursos compartidos de archivos de Azure](../files/storage-files-prevent-file-share-deletion.md).<br /><br />Microsoft recomienda habilitar la eliminación temporal de los recursos compartidos de archivos de las cargas de trabajo de Azure Files y establecer un período de retención mínimo de siete días. |
| Seguimiento | Habilitar el control de versiones para blobs | Opcionales | El control de versiones de blobs guarda automáticamente el estado de un blob en una versión anterior cuando se sobrescribe ese blob. Para más información, consulte [Control de versiones de blobs](../blobs/versioning-overview.md).<br /><br />Microsoft recomienda habilitar el control de versiones de blobs para conseguir una protección de datos óptima para la cuenta de almacenamiento. |
| Seguimiento | Habilitar la fuente de cambios del blob | Opcionales | La fuente de cambios de blobs proporciona registros de transacciones de todos los cambios en todos los blobs de la cuenta de almacenamiento, así como en sus metadatos. Para obtener más información, consulte [Compatibilidad con la fuente de cambios en Azure Blob Storage](../blobs/storage-blob-change-feed.md). |

### <a name="tags-tab"></a>Pestaña Etiquetas

En la pestaña **Etiquetas**, puede especificar etiquetas de Resource Manager para ayudar a organizar los recursos de Azure. Para obtener más información, consulte [Etiquetado de recursos, grupos de recursos y suscripciones para una organización lógica](../../azure-resource-manager/management/tag-resources.md).

### <a name="review--create-tab"></a>Pestaña Revisar y crear

Al ir a la pestaña **Revisar y crear**, Azure ejecuta la validación en la configuración de la cuenta de almacenamiento que ha elegido. Si se supera la validación, puede continuar con la creación de la cuenta de almacenamiento.

En cambio, si se produce un error en la validación, el portal indica qué configuración debe modificarse.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para crear una cuenta de almacenamiento v2 de uso general con PowerShell, primero debe crear un grupo de recursos mediante una llamada al comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

```azurepowershell-interactive
$resourceGroup = "<resource-group>"
$location = "<location>"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

Si no está seguro de qué región especificar para el parámetro `-Location`, puede recuperar una lista de regiones admitidas para la suscripción con el comando [Get-AzLocation](/powershell/module/az.resources/get-azlocation):

```azurepowershell-interactive
Get-AzLocation | select Location
```

A continuación, para crear una cuenta de almacenamiento estándar v2 de uso general con almacenamiento con redundancia local con acceso de lectura (RA-GRS), use el comando [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount). Recuerde que el nombre de la cuenta de almacenamiento debe ser único en Azure, así que reemplace el valor de marcador de posición entre corchetes por su propio valor único:

```azurepowershell-interactive
New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name <account-name> `
  -Location $location `
  -SkuName Standard_RAGRS `
  -Kind StorageV2
```

Para habilitar un espacio de nombres jerárquico para que la cuenta de almacenamiento use [Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage/), incluya el parámetro `-EnableHierarchicalNamespace $True` en la llamada al comando **New-AzStorageAccount**.

En la tabla siguiente se muestran los valores que se usarán para los parámetros `-SkuName` y `-Kind` para crear un tipo determinado de cuenta de almacenamiento con la configuración de redundancia deseada.

| Tipo de cuenta de almacenamiento | Configuraciones de redundancia admitidas | Valor del parámetro -Kind | Valores posibles para el parámetro -SkuName | Admite el espacio de nombres jerárquico |
|--|--|--|--|--|
| De uso general estándar, v2 | LRS / GRS / RA-GRS / ZRS / GZRS / RA-GZRS | StorageV2 | Standard_LRS / Standard_GRS / Standard_RAGRS/ Standard_ZRS / Standard_GZRS / Standard_RAGZRS | Yes |
| Blobs en bloques Premium | LRS / ZRS | BlockBlobStorage | Premium_LRS / Premium_ZRS | Yes |
| Recursos compartidos de archivos Prémium | LRS / ZRS | FileStorage | Premium_LRS / Premium_ZRS | No |
| Blobs en páginas Premium | LRS | StorageV2 | Premium_LRS | No |
| De uso general v1, estándar, heredado | LRS / GRS / RA-GRS | Storage | Standard_LRS / Standard_GRS / Standard_RAGRS | No |
| Instancia de Blob Storage heredada | LRS / GRS / RA-GRS | BlobStorage | Standard_LRS / Standard_GRS / Standard_RAGRS | No |

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para crear una cuenta de almacenamiento v2 de uso general con la CLI de Azure, primero debe crear un grupo de recursos mediante una llamada al comando [az group create](/cli/azure/group#az_group_create).

```azurecli-interactive
az group create \
  --name storage-resource-group \
  --location westus
```

Si no está seguro de qué región especificar para el parámetro `--location`, puede recuperar una lista de regiones admitidas para la suscripción con el comando [az account list-locations](/cli/azure/account#az_account_list).

```azurecli-interactive
az account list-locations \
  --query "[].{Region:name}" \
  --out table
```

A continuación, para crear una cuenta de almacenamiento estándar de uso general v2 con almacenamiento con redundancia local con acceso de lectura (RA-GRS), use el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create). Recuerde que el nombre de la cuenta de almacenamiento debe ser único en Azure, así que reemplace el valor de marcador de posición entre corchetes por su propio valor único:

```azurecli-interactive
az storage account create \
  --name <account-name> \
  --resource-group storage-resource-group \
  --location westus \
  --sku Standard_RAGRS \
  --kind StorageV2
```

Para habilitar un espacio de nombres jerárquico para que la cuenta de almacenamiento use [Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage/), incluya el parámetro `--enable-hierarchical-namespace true` en la llamada al comando **az storage account create**. Para crear un espacio de nombres jerárquico debe usar la versión 2.0.79 o posterior de la CLI de Azure.

En la tabla siguiente se muestran los valores que se usarán para los parámetros `-sku` y `-kind` para crear un tipo determinado de cuenta de almacenamiento con la configuración de redundancia deseada.

| Tipo de cuenta de almacenamiento | Configuraciones de redundancia admitidas | Valor del parámetro -kind | Valores posibles del parámetro -sku | Admite el espacio de nombres jerárquico |
|--|--|--|--|--|
| De uso general estándar, v2 | LRS / GRS / RA-GRS / ZRS / GZRS / RA-GZRS | StorageV2 | Standard_LRS / Standard_GRS / Standard_RAGRS/ Standard_ZRS / Standard_GZRS / Standard_RAGZRS | Yes |
| Blobs en bloques Premium | LRS / ZRS | BlockBlobStorage | Premium_LRS / Premium_ZRS | Yes |
| Recursos compartidos de archivos Prémium | LRS / ZRS | FileStorage | Premium_LRS / Premium_ZRS | No |
| Blobs en páginas Premium | LRS | StorageV2 | Premium_LRS | No |
| De uso general v1, estándar, heredado | LRS / GRS / RA-GRS | Storage | Standard_LRS / Standard_GRS / Standard_RAGRS | No |
| Instancia de Blob Storage heredada | LRS / GRS / RA-GRS | BlobStorage | Standard_LRS / Standard_GRS / Standard_RAGRS | No |

# <a name="template"></a>[Plantilla](#tab/template)

Puede usar Azure PowerShell o la CLI de Azure para implementar una plantilla de Resource Manager y crear así una cuenta de almacenamiento. La plantilla empleada en este artículo de procedimientos proviene de [Plantillas de inicio rápido de Azure Resource Manager](https://azure.microsoft.com/resources/templates/101-storage-account-create/). Para ejecutar los scripts, seleccione **Pruébelo** para abrir Azure Cloud Shell. Para pegar el script, haga clic con el botón derecho en el shell y, a continuación, seleccione **Pegar**.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
```

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
```

> [!NOTE]
> Esta plantilla sirve solo como ejemplo. Hay muchas opciones de una cuenta de almacenamiento que no se configuran como parte de esta plantilla. Por ejemplo, si quiere utilizar [Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage/), debe modificar esta plantilla estableciendo la propiedad `isHnsEnabledad` del objeto `StorageAccountPropertiesCreateParameters` en `true`.

Para obtener información sobre cómo modificar esta plantilla o crear otras nuevas, consulte:

- [Documentación de Azure Resource Manager](../../azure-resource-manager/index.yml).
- [Referencia de plantilla de cuenta de almacenamiento](/azure/templates/microsoft.storage/allversions).
- [Ejemplos de plantillas de cuenta de almacenamiento adicionales](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Storage).

---

## <a name="delete-a-storage-account"></a>Eliminar una cuenta de almacenamiento

Si se elimina la cuenta de almacenamiento, se elimina toda la cuenta, incluidos todos los datos de la cuenta. Normalmente, esta operación no se puede deshacer. La recuperación de una cuenta de almacenamiento solo es posible en determinadas circunstancias y no está garantizada. Para obtener más información, consulte [Recuperación de una cuenta de almacenamiento eliminada](storage-account-recover.md).

> [!WARNING]
> Asegúrese de hacer una copia de seguridad de cualquier contenido que desee guardar antes de eliminar la cuenta. Normalmente, no es posible restaurar una cuenta de almacenamiento eliminada ni recuperar los recursos que contenía antes de la eliminación.

Si intenta eliminar una cuenta de almacenamiento asociada a una máquina virtual de Azure, puede obtener un error sobre que la cuenta de almacenamiento aún está usándose. Para obtener ayuda para solucionar este error, consulte [Solución de errores al eliminar las cuentas de almacenamiento](/troubleshoot/azure/virtual-machines/storage-resource-deletion-errors).

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Navegue hasta cuenta de almacenamiento en [Azure Portal](https://portal.azure.com).
1. Haga clic en **Eliminar**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para eliminar la cuenta de almacenamiento, utilice el comando [Remove-AzStorageAccount](/powershell/module/az.storage/remove-azstorageaccount):

```powershell
Remove-AzStorageAccount -Name <storage-account> -ResourceGroupName <resource-group>
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para eliminar la cuenta de almacenamiento, utilice el comando [az storage account delete](/cli/azure/storage/account#az_storage_account_delete):

```azurecli-interactive
az storage account delete --name <storage-account> --resource-group <resource-group>
```

# <a name="template"></a>[Plantilla](#tab/template)

Para eliminar la cuenta de almacenamiento, use Azure PowerShell o la CLI de Azure.

```azurepowershell-interactive
$storageResourceGroupName = Read-Host -Prompt "Enter the resource group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"
Remove-AzStorageAccount -Name $storageAccountName -ResourceGroupName $storageResourceGroupName
```

```azurecli-interactive
echo "Enter the resource group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az storage account delete --name storageAccountName --resource-group resourceGroupName
```

---

Como alternativa, puede eliminar el grupo de recursos, que elimina la cuenta de almacenamiento y otros recursos de ese grupo de recursos. Para obtener más información sobre cómo eliminar un grupo de recursos, consulte [Eliminación del grupo de recursos y los recursos](../../azure-resource-manager/management/delete-resource-group.md).

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a las cuentas de almacenamiento](storage-account-overview.md)
- [Actualización a una cuenta de almacenamiento de uso general v2](storage-account-upgrade.md)
- [Traslado de una cuenta de almacenamiento a otra región](storage-account-move.md)
- [recuperar una cuenta de almacenamiento eliminada](storage-account-recover.md)