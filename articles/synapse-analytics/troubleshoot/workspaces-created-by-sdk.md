---
title: 'Solución de problemas: Las áreas de trabajo creadas por el SDK no pueden iniciar Synapse Studio.'
description: Pasos para resolver áreas de trabajo creadas por el SDK que no pueden iniciar Synapse Studio
author: julieMSFT
ms.author: jrasnick
ms.topic: troubleshooting
ms.service: synapse-analytics
ms.subservice: troubleshooting
ms.date: 11/24/2020
ms.openlocfilehash: 54341621e0b696d2bc4b8570c6936d1fae78514e
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108139910"
---
# <a name="troubleshoot-azure-synapse-analytics-workspaces-created-using-sdk"></a>Solución de problemas de áreas de trabajo de Azure Synapse Analytics creadas con el SDK

En este artículo se proporcionan pasos para la solución de problemas para iniciar Synapse Studio desde un área de trabajo de Synapse que se creó con un kit de desarrollo de software (SDK).


## <a name="prerequisites"></a>Requisitos previos

- Área de trabajo de Synapse creada con el SDK

## <a name="workaround"></a>Solución alternativa

Para iniciar Synapse Studio desde el área de trabajo de SDK que ha creado, complete los pasos siguientes: 
  1.    Cree el área de trabajo mediante la ejecución de `az synapse workspace create`.
  2.    Extraiga el identificador de identidad administrada mediante la ejecución de `$identity=$(az synapse workspace show --name {workspace name}  --resource-group {resource group name} --query "identity.principalId")`.
  3.    Ejecute ` az role assignment create --role "Storage Blob Data Contributor" --assignee-object-id {identity } --scope {storage account resource id}.` para conceder el rol Colaborador de datos de Blob Storage a la cuenta de almacenamiento de identidad administrada.
  4.    Agregue la regla de firewall mediante la ejecución de ` az synapse firewall-rule create --name allowAll --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255 `.

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es Azure Synapse?](../overview-what-is.md)
* [Primeros pasos](../get-started.md)
