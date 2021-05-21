---
title: Conversión de los volúmenes SMB de Azure NetApp Files existentes para usar la disponibilidad continua de SMB | Microsoft Docs
description: Se describe la habilitación de la disponibilidad continua de SMB mediante la conversión de un volumen SMB de Azure NetApp Files existente.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/10/2021
ms.author: b-juche
ms.openlocfilehash: b55b7e5632ad852b199121179772fa078129cdb5
ms.sourcegitcommit: 1b19b8d303b3abe4d4d08bfde0fee441159771e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109755107"
---
# <a name="convert-existing-smb-volumes-to-use-continuous-availability"></a>Conversión de volúmenes SMB existentes para usar la disponibilidad continua 

Puede habilitar la característica de disponibilidad continua (CA) de SMB al [crear un nuevo volumen SMB](azure-netapp-files-create-volumes-smb.md#add-an-smb-volume). También puede convertir un volumen SMB existente para habilitar la característica de disponibilidad continua de SMB.  En este artículo se muestra cómo habilitar la disponibilidad continua de SMB mediante la conversión de un volumen existente.  

> [!IMPORTANT]
> Este procedimiento incluye un paso del volumen original al nuevo volumen habilitado para los recursos compartidos con la disponibilidad continua habilitada. Por lo tanto, debe planear una ventana de mantenimiento para este proceso. 

## <a name="steps"></a>Pasos

1. Asegúrese de que ha registrado la característica de [recursos compartidos con disponibilidad continua de SMB](https://aka.ms/anfsmbcasharespreviewsignup).  
2. Detenga la aplicación que usa el volumen SMB. 
3. [Cree una instantánea a petición](azure-netapp-files-manage-snapshots.md#create-an-on-demand-snapshot-for-a-volume) del volumen existente. 
4. Seleccione **Instantáneas** en el volumen existente para mostrar la lista de instantáneas.
5. Haga clic con el botón derecho en la instantánea que se va a restaurar y seleccione la opción de menú **Restaurar al nuevo volumen**.
    
    ![Instantánea que muestra la opción Restaurar al nuevo volumen.](../media/azure-netapp-files/azure-netapp-files-snapshot-restore-to-new-volume.png)

6. En la ventana para crear un volumen que aparece, especifique la información del nuevo volumen:  

    * **Nombre del volumen**    
    Especifique el nombre para el volumen que va a crear.   
    El nombre debe ser único dentro de un grupo de recursos. Debe tener tres caracteres de longitud, como mínimo. Puede usar cualquier carácter alfanumérico.

    * **Cuota**   
    Especifique la cantidad de almacenamiento lógico que desee asignar al volumen.

    ![Instantánea que muestra la ventana para crear un volumen.](../media/azure-netapp-files/snapshot-restore-new-volume.png) 

7. En la sección Protocolo de la ventana Crear un volumen, asegúrese de seleccionar la opción **Enable Continuous Availability** (Habilitar la disponibilidad continua).

    ![Instantánea que muestra la opción para habilitar la disponibilidad continua.](../media/azure-netapp-files/enable-continuous-availability-option.png) 

8. Haga clic en **Revisar + crear**. Haga clic en **Crear**.   
    El nuevo volumen utiliza el mismo protocolo que la instantánea.
   El nuevo volumen en el que se restaura la instantánea aparece en la vista Volúmenes.

9. Una vez creado el nuevo volumen, haga clic en **Instrucciones de montaje** en la hoja del volumen seleccionado. A continuación, siga las instrucciones para montar el nuevo volumen.    

10. Vuelva a configurar la aplicación para que use el nuevo punto de montaje del volumen.   

11. Reinicie la aplicación que detuvo durante el paso 2. 

12. Una vez que la aplicación comienza a aprovechar el nuevo volumen y cuando se complete el proceso de restauración en el nuevo volumen, opcionalmente puede eliminar el volumen original.  

## <a name="next-steps"></a>Pasos siguientes  

* [Cree un volumen SMB para Azure NetApp Files](azure-netapp-files-create-volumes-smb.md)
* [Montaje o desmontaje de un volumen para máquinas virtuales Windows o Linux](azure-netapp-files-mount-unmount-volumes-for-virtual-machines.md)