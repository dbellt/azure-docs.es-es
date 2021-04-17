---
author: rayne-wiselman
ms.service: site-recovery
ms.topic: include
ms.date: 10/26/2018
ms.author: raynew
ms.openlocfilehash: 8076a6af2382ccec1ac832cd83c3946d8c7c6f57
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96019158"
---
|Nombre de parámetro| Tipo | Descripción| Valores posibles|
|-|-|-|-|
| /ServerMode|Mandatory|Especifica si se deben instalar los servidores de configuración y de procesos, o solo el servidor de procesos|CS<br>PS|
|/InstallLocation|Mandatory|La carpeta donde se instalan los componentes| Cualquier carpeta del equipo|
|/MySQLCredsFilePath|Mandatory|La ruta de acceso al archivo en que se almacenan las credenciales del servidor MySQL|El archivo debe tener el formato especificado a continuación|
|/VaultCredsFilePath|Mandatory|La ruta de acceso del archivo de credenciales del almacén|Ruta de acceso de archivo válido|
|/EnvType|Mandatory|El tipo de entorno que quiere proteger |VMware<br>NonVMware|
|/PSIP|Mandatory|Dirección IP de la NIC que se usará para la transferencia de datos de replicación| Cualquier dirección IP válida|
|/CSIP|Mandatory|Dirección IP de la NIC en la que el servidor de configuración realiza la escucha| Cualquier dirección IP válida|
|/PassphraseFilePath|Mandatory|Ruta de acceso completa a la ubicación del archivo de frase de contraseña|Ruta de acceso de archivo válido|
|/BypassProxy|Opcional|Especifica si el servidor de configuración se conecta a Azure sin proxy||
|/ProxySettingsFilePath|Opcional|Configuración del proxy (el proxy predeterminado requiere autenticación, o un proxy personalizado)|El archivo debe tener el formato especificado a continuación|
|DataTransferSecurePort|Opcional|Número de puerto en el PSIP que se usará para los datos de replicación| Número de puerto válido (el valor predeterminado es 9433)|
|/SkipSpaceCheck|Opcional|Omitir comprobación de espacio para disco de caché| |
|/AcceptThirdpartyEULA|Mandatory|La marca implica la aceptación de los términos de licencia de terceros| |
|/ShowThirdpartyEULA|Opcional|Muestra los términos de licencia de terceros. Si se proporciona como entrada, se omiten todos los demás parámetros| |
