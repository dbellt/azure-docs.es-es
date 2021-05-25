---
title: ¿Qué es el control de acceso basado en atributos de Azure (Azure ABAC)? (versión preliminar)
description: Obtenga información general del control de acceso basado en atributos de Azure (Azure ABAC). Use asignaciones de roles con condiciones para controlar el acceso a los recursos de Azure.
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.subservice: conditions
ms.topic: overview
ms.workload: identity
ms.date: 05/06/2021
ms.author: rolyon
ms.openlocfilehash: 8f2849500d46c48f62ee203b7e40176b618f514c
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109490035"
---
# <a name="what-is-azure-attribute-based-access-control-azure-abac-preview"></a>¿Qué es el control de acceso basado en atributos de Azure (Azure ABAC)? (versión preliminar)

> [!IMPORTANT]
> Las condiciones de asignación de roles de Azure y Azure ABAC se encuentran actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

El control de acceso basado en atributos (ABAC) es un sistema de autorización que define el acceso en función de los atributos asociados a las entidades de seguridad, los recursos y el entorno. Con ABAC, puede conceder a una entidad de seguridad acceso a un recurso en función de los atributos. Azure ABAC hace referencia a la implementación del control de acceso basado en atributos para Azure.

## <a name="what-are-role-assignment-conditions"></a>¿Qué son las condiciones de asignación de roles?

El control de acceso basado en rol de Azure (Azure RBAC) es un sistema de autorización que ayuda a administrar quién tiene acceso a los recursos de Azure, qué puede hacer con esos recursos y a qué áreas puede acceder. En la mayoría de los casos, Azure RBAC proporcionará la administración de acceso que necesita mediante definiciones de roles y asignaciones de roles. Sin embargo, en algunos casos es posible que desee proporcionar una administración de acceso más precisa o simplificar la administración de cientos de asignaciones de roles.

Azure ABAC se basa en Azure RBAC con la adición de condiciones de asignación de roles basadas en atributos en el contexto de acciones específicas. Una *condición de asignación de roles* es una comprobación adicional que puede agregar opcionalmente a la asignación de roles para proporcionar un control de acceso más preciso. Una condición filtra los permisos concedidos como parte de la definición de roles y la asignación de roles. Por ejemplo, puede agregar una condición que requiera que un objeto tenga una etiqueta específica para leer el objeto. No se puede denegar explícitamente el acceso a recursos específicos mediante condiciones.

## <a name="why-use-conditions"></a>¿Por qué usar condiciones?

El uso de condiciones de asignación de roles ofrece tres ventajas principales:

- **Proporcionar un control de acceso más preciso**: una asignación de roles usa una definición de roles con acciones y acciones de datos para conceder permisos a una entidad de seguridad. Puede escribir condiciones para filtrar esos permisos para un control de acceso más preciso. También puede agregar condiciones a acciones específicas. Por ejemplo, puede conceder a John acceso de lectura a los blobs de la suscripción solo si los blobs tienen la etiqueta Project=Blue. 
- **Ayudar a reducir el número de asignaciones de roles:** cada suscripción de Azure tiene actualmente un límite de asignación de roles de 2000. Hay escenarios que requieren miles de asignaciones de roles. Todas esas asignaciones de roles tendrían que administrarse. En estos escenarios, podría agregar condiciones para usar una cantidad de asignaciones de roles considerablemente menor. 
- **Usar atributos que tienen un significado empresarial específico:** las condiciones le permiten usar atributos que tienen un significado empresarial específico en el control de acceso. Algunos ejemplos de atributos son el nombre del proyecto, la fase de desarrollo de software y los niveles de clasificación. Los valores de estos atributos de recursos son dinámicos y cambian a medida que los usuarios se mueven entre equipos y proyectos.

## <a name="example-scenarios-for-conditions"></a>Escenarios de ejemplo para condiciones

Hay varios escenarios en los que es posible que desee agregar una condición a la asignación de roles. Estos son algunos ejemplos.

- Acceso de lectura a blobs con la etiqueta Project=Cascade.
- Nuevos blobs que deben incluir la etiqueta Project=Cascade.
- Blobs existentes que deben etiquetarse con al menos una clave de proyecto o clave de programa.
- Blobs existentes que deben etiquetarse con una clave de proyecto y los valores Cascade, Baker o Skagit.
- Lectura, escritura o eliminación de blobs en contenedores denominados blobs-example-container.
- Acceso de lectura a blobs en contenedores denominados blobs-example-container con una ruta de acceso de solo lectura.
- Acceso de escritura a blobs en contenedores denominados Contosocorp con una ruta de acceso de uploads/contoso.
- Acceso de lectura a blobs con la etiqueta Program=Alpine y una ruta de acceso de registros.

Para más información sobre cómo crear estos ejemplos, consulte [Ejemplos de condiciones de asignación de roles de Azure](../storage/common/storage-auth-abac-examples.md).

## <a name="where-can-conditions-be-added"></a>¿Dónde se pueden agregar condiciones?

Actualmente, se pueden agregar condiciones a asignaciones de roles integradas o personalizadas que tienen [acciones de datos de Storage Blob](conditions-format.md#actions). Entre estas se incluyen los siguientes roles integrados:

- [Colaborador de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-contributor)
- [Propietario de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-owner)
- [Lector de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-reader)

Las condiciones se agregan en el mismo ámbito que la asignación de roles. Al igual que con las asignaciones de roles, debe tener permisos `Microsoft.Authorization/roleAssignments/write` para agregar una condición.

Estos son los atributos de almacenamiento que puede usar en sus condiciones.

- Nombre del contenedor
- Ruta del blob
- Claves de etiquetas de índice de blobs
- Etiquetas de índice de blobs

> [!TIP]
> Los blobs también admiten la capacidad de almacenar metadatos arbitrarios de clave-valor definidos por el usuario. Aunque los metadatos son similares a las etiquetas de índice de blobs, debe usar etiquetas de índice de blobs con condiciones. Para más información, consulte [Administración y búsqueda de datos de Azure Blob con etiquetas de índice de blobs (versión preliminar)](../storage/blobs/storage-manage-find-blobs.md).

## <a name="what-does-a-condition-look-like"></a>¿Qué aspecto tiene una condición?

Puede agregar condiciones a asignaciones de roles nuevas o existentes. Este es el rol [Lector de datos de blobs de almacenamiento](built-in-roles.md#storage-blob-data-reader) que se ha asignado a un usuario denominado Chandra en un ámbito de grupo de recursos. También se ha agregado una condición que solo permite el acceso de lectura a blobs con la etiqueta Project=Cascade.

![Diagrama de asignación de roles con una condición.](./media/conditions-overview/condition-role-assignment-rg.png)

Si Chandra intenta leer un blob sin la etiqueta Project=Cascade, no se permitirá el acceso.

![Diagrama de acceso no permitido con una condición.](./media/conditions-overview/condition-access-multiple.png)

Este es el aspecto de la condición en el código:

```
(
    (
        !(ActionMatches{'Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read'}
        AND
        @Request[subOperation] ForAnyOfAnyValues:StringEqualsIgnoreCase {'Blob.Read.WithTagConditions'})
    )
    OR
    (
        @Resource[Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags:Project<$key_case_sensitive$>] StringEquals 'Cascade'
    )
)
```

Para más información sobre el formato de las condiciones, consulte [Sintaxis y formato de las condiciones de asignación de roles de Azure](conditions-format.md).

## <a name="terminology"></a>Terminología

Para comprender mejor Azure RBAC y Azure ABAC, puede consultar la siguiente lista de términos.

| Término | Definición |
| --- | --- |
| control de acceso basado en atributos (ABAC) | Sistema de autorización que define el acceso en función de los atributos asociados a las entidades de seguridad, los recursos y el entorno. Con ABAC, puede conceder a una entidad de seguridad acceso a un recurso en función de los atributos. |
| Azure ABAC | Hace referencia a la implementación del control de acceso basado en atributos para Azure. |
| condición de asignación de roles | Comprobación adicional que puede agregar opcionalmente a la asignación de roles para proporcionar un control de acceso más preciso. |
| atributo | En este contexto, un par clave-valor como Project=Blue, donde Project es la clave de atributo y Blue es el valor del atributo. Los atributos y etiquetas son sinónimos en lo que respecta al control de acceso. |
| expresión | Instrucción en una condición que se evalúa como true o false. Una expresión tiene el formato de &lt;atributo&gt; &lt;operador&gt; &lt;valor&gt;. |

## <a name="next-steps"></a>Pasos siguientes

- [Preguntas frecuentes sobre las condiciones de asignación de roles de Azure (versión preliminar)](conditions-faq.md)
- [Condiciones de asignación de roles de Azure de ejemplo (versión preliminar)](../storage/common/storage-auth-abac-examples.md)
- [Tutorial: Adición de una condición de asignación de roles para restringir el acceso a blobs mediante Azure Portal (versión preliminar)](../storage/common/storage-auth-abac-portal.md)
