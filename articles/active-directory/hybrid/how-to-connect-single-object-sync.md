---
title: 'Azure AD Connect sincronización de un solo objeto '
description: Obtenga información acerca de cómo sincronizar un objeto de Active Directory a Azure AD para la solución de problemas.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/19/2021
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: ee850765006af15d8e323831d70dfcd7849d1287
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108145968"
---
# <a name="azure-ad-connect-single-object-sync"></a>Azure AD Connect sincronización de un solo objeto 

La herramienta de sincronización de un solo objeto Azure AD Connect es un cmdlet de PowerShell que se puede usar para sincronizar un objeto individual de Active Directory a Azure Active Directory. El informe generado se puede utilizar para investigar y solucionar problemas relacionados con la sincronización de objetos. 

> [!NOTE]
> La herramienta admite la sincronización desde Active Directory a Azure Active Directory. No admite la sincronización desde Azure Active Directory a Active Directory. 
>
> La herramienta admite la sincronización de una modificación y actualización de objetos. No admite la sincronización de una eliminación de modificación de objetos. 

## <a name="how-it-works"></a>Funcionamiento
La herramienta de sincronización de un solo objeto requiere un nombre distintivo de Active Directory como entrada para buscar el conector de origen y la partición para la importación. Exporta los cambios a Azure Active Directory. La herramienta genera una salida JSON similar al tipo de recurso **provisioningObjectSummary**. 

La herramienta de sincronización de un solo objeto realiza los siguientes pasos: 

 1. Determine si el dominio del objeto (origen) (Active Directory Connector and Partition) está en el ámbito de sincronización. 
 2. Determine si el dominio del objeto (destino) (Azure Active Directory Connector and Partition) está en el ámbito de sincronización. 
 3. Determinar si la unidad organizativa del objeto está en el ámbito de sincronización. 
 4. Determine si el objeto es accesible mediante las credenciales de la cuenta de conector. 
 5. Determine si el tipo de objeto está en el ámbito de sincronización. 
 6. Determine si el objeto está en el ámbito de sincronización si está habilitado el filtrado de grupos. 
 7. Importe el objeto de Active Directory a Active Directory Connector Space. 
 8. Importe el objeto de Azure Active Directory a Active Directory Connector Space. 
 9. Sincronizar objeto desde Active Directory Connector Space. 
 10. Exporte el objeto desde Azure Active Directory Connector Space a Azure Active Directory. 

Además de la salida JSON, la herramienta genera un informe HTML con todos los detalles de la operación de sincronización. El informe HTML se encuentra en **C:\ProgramData\AADConnect\ADSyncObjectDiagnostics\ ADSyncSingleObjectSyncResult-<date>.htm**. Este informe se puede compartir con el equipo de soporte técnico para profundizar en la solución de problemas, en caso necesario. 

El informe HTML tiene lo siguiente: 

|Pestaña|Descripción|
|-----|-----|
|Pasos|Describe los pasos dados para sincronizar un objeto. Cada paso contiene detalles para la solución de problemas. Los pasos de importación, sincronización y exportación contienen información de atributos adicional, como el nombre, con varios valores, tipo, valor, adición de valor, eliminación de valor, operación, regla de sincronización, tipo de asignación y origen de datos.| 
|Solución de problemas y Recomendaciones|proporciona el código de error y el motivo. La información de error solo está disponible si se produce uno.| 
|propiedades modificadas|se muestran el valor anterior y el nuevo. Si no hay ningún valor anterior o si se elimina el nuevo valor, la celda está en blanco. En el caso de los atributos multivalor, muestra el recuento. El nombre del atributo es un vínculo a la pestaña pasos: exportar objeto desde Azure Active Directory Connector Space a Azure Active Directory: información del atributo que contiene detalles adicionales del atributo, como nombre, con varios valores, tipo, valor, agregar valor, eliminación de valor, operación, regla de sincronización, tipo de asignación y origen de datos.| 
|Resumen|se proporciona información general sobre lo que sucedió y los identificadores del objeto en los sistemas de origen y de destino.| 

## <a name="prerequisites"></a>Requisitos previos 

Para usar la herramienta de sincronización de un solo objeto, tendrá que usar la versión de marzo de 2021 de Azure AD Connect o posterior. 

### <a name="run-the-single-object-sync-tool"></a>Ejecutar la herramienta de sincronización de un solo objeto 

La herramienta de sincronización de un solo objeto realiza los siguientes pasos: 

 1. Abra una nueva sesión de Windows PowerShell en el servidor de Azure AD Connect mediante la opción Ejecutar como administrador. 

 2. Establezca la [directiva de ejecución](/powershell/module/microsoft.powershell.security/set-executionpolicy) en RemoteSigned o Unrestricted. 

 3. Deshabilite el programador de sincronización después de comprobar que no se está ejecutando ninguna operación de sincronización. 

     `Set-ADSyncScheduler -SyncCycleEnabled $false` 

 4. Importar el módulo de diagnósticos de AdSync 

     `Import-module -Name "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSyncDiagnostics\ADSyncDiagnostics.psm1"` 

 5. Invocar el cmdlet de sincronización de un solo objeto. 

     `Invoke-ADSyncSingleObjectSync -DistinguishedName "CN=testobject,OU=corp,DC=contoso,DC=com" | Out-File -FilePath ".\output.json"` 

 6. Vuelva a habilitar el programador de sincronización. 

     `Set-ADSyncScheduler -SyncCycleEnabled $true`

|Parámetros de entrada de sincronización de un solo objeto|Descripción| 
|-----|----|
|DistinguishedName|Este es un parámetro de cadena necesario. </br></br>Es el nombre distintivo del objeto de Active Directory que necesita sincronización y solución de problemas.| 
|StagingMode|Este es un parámetro de cambio opcional.</br></br>Este parámetro se puede usar para impedir que se exporten los cambios a Azure Active Directory.</br></br>**Nota**: el cmdlet confirmará la operación de sincronización. </br></br>**Nota**: el servidor de Azure AD Connect Staging no exportará los cambios a Azure Active Directory.|
|NoHtmlReport|Este es un parámetro de cambio opcional.</br></br>Este parámetro se puede usar para impedir que se genere el informe HTML. 

## <a name="single-object-sync-throttling"></a>Limitación de sincronización de un solo objeto 

La herramienta de sincronización de un solo objeto **está** diseñada para investigar y solucionar problemas relacionados con la sincronización de objetos. **No** está diseñado para reemplazar al ciclo de sincronización ejecutado por el Scheduler. La importación desde Azure Active Directory y la exportación a Azure Active Directory están sujetas a límites de limitación. Vuelva a intentarlo después de 5 minutos, si alcanza el límite de limitación. 

## <a name="next-steps"></a>Pasos siguientes
- [Solución de problemas de sincronización de objetos](tshoot-connect-objectsync.md)
- [Solución de problemas de objeto que no se sincroniza](tshoot-connect-object-not-syncing.md)
- [Solución de problemas de un extremo a otro de objetos y atributos de Azure AD Connect](/troubleshoot/azure/active-directory/troubleshoot-aad-connect-objects-attributes)