---
ms.service: data-factory
ms.topic: include
ms.date: 11/09/2018
author: linda33wj
ms.author: jingwang
ms.openlocfilehash: e96e3fe6a1c93c914e9dec7263852b047a05e38d
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108741778"
---
## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

El conducto crea automáticamente la carpeta de salida en el contenedor de blob adftutorial. A continuación, copia el archivo emp.txt de la carpeta de entrada a la carpeta de salida. 

1. En Azure Portal, en la página del contenedor **adftutorial**, seleccione **Actualizar** para ver la carpeta de salida. 
    
    ![Captura de pantalla que muestra la página del contenedor donde puede actualizar la página.](media/data-factory-quickstart-verify-output-cleanup/output-refresh.png)

2. En la lista de carpetas, seleccione **output**. 

3. Confirme que **emp.txt** se ha copiado en la carpeta de salida. 

    ![Captura de pantalla que muestra el contenido de la carpeta de salida.](media/data-factory-quickstart-verify-output-cleanup/output-file.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Hay dos forma de eliminar los recursos que ha creado en la guía de inicio rápido. Puede eliminar el [grupo de recursos de Azure](../../azure-resource-manager/management/overview.md), lo que incluye todos los recursos del grupo de recursos. Si quiere mantener intacto el resto de recursos, elimine solo la factoría de datos que creó en este tutorial.

Al eliminar un grupo de recursos, se eliminan todos los recursos incluidas las factorías de datos. Ejecute el comando siguiente para eliminar el grupo de recursos completo: 

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourcegroupname
```

> [!Note]
> La eliminación de un grupo de recursos puede tardar algún tiempo. Tenga paciencia con el proceso.

Si desea eliminar solo la factoría de datos, no el grupo de recursos completo, ejecute el siguiente comando: 

```powershell
Remove-AzDataFactoryV2 -Name $dataFactoryName -ResourceGroupName $resourceGroupName
```