---
title: Ejecución de una aplicación desde un paquete ZIP
description: Implemente el paquete ZIP de la aplicación con atomicidad. Mejore la previsibilidad y la confiabilidad del comportamiento de su aplicación durante el proceso de implementación de ZIP.
ms.topic: article
ms.date: 01/14/2020
ms.openlocfilehash: d3315370342f54091598aa3f77f70f03bda4ad33
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107772746"
---
# <a name="run-your-app-in-azure-app-service-directly-from-a-zip-package"></a>Ejecución de una aplicación en Azure App Service directamente desde un paquete ZIP

En [Azure App Service](overview.md), puede ejecutar sus aplicaciones directamente desde un archivo de paquete ZIP de implementación. En este artículo se muestra cómo habilitar esta funcionalidad en la aplicación.

Todos los demás métodos de implementación de App Service tienen algo en común: los archivos se implementan en *D:\home\site\wwwroot* en la aplicación (o */home/site/wwwroot* para las aplicaciones de Linux). Dado que la aplicación usa el mismo directorio en tiempo de ejecución, es posible que la implementación no se realice correctamente debido a conflictos de bloqueo de archivos, y que la aplicación se comporte de forma impredecible porque algunos de los archivos aún no se han actualizado.

Por el contrario, cuando se ejecuta directamente desde un paquete, los archivos del paquete no se copian en el directorio *wwwroot*. En su lugar, el propio paquete ZIP se monta directamente como directorio *wwwroot* de solo lectura. Son varias las ventajas de ejecutar directamente desde un paquete:

- Elimina los conflictos de bloqueo de archivos entre la implementación y el tiempo de ejecución.
- Garantiza que solo se ejecuten aplicaciones de implementación completa en todo momento.
- Se pueden implementar en una aplicación de producción (con reinicio).
- Mejora el rendimiento de las implementaciones de Azure Resource Manager.
- Puede reducir los tiempos de arranque en frío, especialmente para las funciones de JavaScript con árboles de paquete de npm grandes.

> [!NOTE]
> Actualmente, solo se admiten archivos de paquete ZIP.

[!INCLUDE [Create a project ZIP file](../../includes/app-service-web-deploy-zip-prepare.md)]

## <a name="enable-running-from-package"></a>Habilitación de la ejecución desde el paquete

La configuración de aplicación `WEBSITE_RUN_FROM_PACKAGE` permite ejecutar desde un paquete. Para establecerlo, ejecute los siguientes comandos con la CLI de Azure.

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITE_RUN_FROM_PACKAGE="1"
```

`WEBSITE_RUN_FROM_PACKAGE="1"` permite ejecutar la aplicación desde un paquete local en la aplicación. También puede [ejecutar desde un paquete remoto](#run-from-external-url-instead).

## <a name="run-the-package"></a>Ejecutar el paquete

La manera más sencilla de ejecutar un paquete en App Service es con el comando [az webapp deployment source config-zip](/cli/azure/webapp/deployment/source#az_webapp_deployment_source_config_zip) de la CLI de Azure. Por ejemplo:

```azurecli-interactive
az webapp deployment source config-zip --resource-group <group-name> --name <app-name> --src <filename>.zip
```

Dado que la configuración de la aplicación `WEBSITE_RUN_FROM_PACKAGE` está establecida, este comando no extrae el contenido del paquete en el directorio *D:\home\site\wwwroot* de la aplicación. En su lugar, carga el archivo ZIP tal cual en *D:\home\data\SitePackages* y crea un *nombredelpaquete.txt* en el mismo directorio, que contiene el nombre del paquete ZIP que se cargará en tiempo de ejecución. Si carga el paquete ZIP de manera diferente (por ejemplo, [FTP](deploy-ftp.md)), debe crear el directorio *D:\home\data\SitePackages* y el archivo *nombredelpaquete.txt* manualmente.

El comando también reinicia la aplicación. Dado que `WEBSITE_RUN_FROM_PACKAGE` se ha establecido, App Service monta el paquete cargado como directorio *wwwroot* de solo lectura y ejecuta la aplicación directamente desde ese directorio montado.

## <a name="run-from-external-url-instead"></a>Ejecución desde una dirección URL externa en su lugar

También puede ejecutar un paquete desde una dirección URL externa, como Azure Blob Storage. Puede usar el [Explorador de Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md) para cargar archivos de paquete en la cuenta de Blob Storage. Debe usar un contenedor de almacenamiento privado con una [firma de acceso compartido (SAS)](../vs-azure-tools-storage-manage-with-storage-explorer.md#generate-a-sas-in-storage-explorer) para permitir que sistema en tiempo de ejecución de App Service acceda al paquete de manera segura. 

Una vez que cargue el archivo en Blob Storage y tenga una dirección URL de SAS para el archivo, establezca la configuración de la aplicación `WEBSITE_RUN_FROM_PACKAGE` en la dirección URL. En el ejemplo siguiente se usa la CLI de Azure:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings WEBSITE_RUN_FROM_PACKAGE="https://myblobstorage.blob.core.windows.net/content/SampleCoreMVCApp.zip?st=2018-02-13T09%3A48%3A00Z&se=2044-06-14T09%3A48%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=bNrVrEFzRHQB17GFJ7boEanetyJ9DGwBSV8OM3Mdh%2FM%3D"
```

Si publica un paquete actualizado con el mismo nombre en Blob Storage, debe reiniciar la aplicación para que el paquete actualizado se cargue en App Service.

## <a name="troubleshooting"></a>Solución de problemas

- La ejecución directa desde un paquete hace que `wwwroot` sea de solo lectura. La aplicación recibirá un error si intenta escribir archivos en este directorio.
- No se admiten los formatos de archivo TAR y GZIP.
- El archivo ZIP puede tener un máximo de 1 GB
- Esta característica no es compatible con [caché local](overview-local-cache.md).
- Para mejorar el rendimiento del arranque en frío, utilice la opción de archivo zip local (`WEBSITE_RUN_FROM_PACKAGE` = 1).

## <a name="more-resources"></a>Más recursos

- [Implementación continua en Azure App Service](deploy-continuous-deployment.md)
- [Implementación de código con un archivo ZIP o WAR](deploy-zip.md)
