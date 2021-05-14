---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 02/11/2021
ms.author: alkohli
ms.openlocfilehash: b2c1ebe390b1a2dec7be678b5d6f3a991056a23b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "102517583"
---
1. Antes de comenzar, asegúrese de que:

    1. Ha configurado y [activado el dispositivo Azure Stack Edge Pro](../articles/databox-online/azure-stack-edge-gpu-deploy-activate.md) con un recurso de Azure Stack Edge en Azure.
    1. Ha [configurado el proceso en este dispositivo en Azure Portal](../articles/databox-online/azure-stack-edge-deploy-configure-compute.md#configure-compute).
    
1. [Conéctese a la interfaz de PowerShell](../articles/databox-online/azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface).
1. Use el siguiente comando para habilitar MPS en el dispositivo.

    ```powershell
    Start-HcsGpuMPS
    ```