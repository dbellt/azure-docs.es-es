---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 04/19/2021
ms.author: alkohli
ms.openlocfilehash: aef81c1ad80d29df05ab8e8c5f27b8827d7993f7
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109805425"
---
|    Patrón de URL |    Componente o funcionalidad  |
|-----------------------------------------------------|-----------------------------------------|
|    https://\*.databoxedge.azure.com/\*<br>https://\*.servicebus.windows.net/\*<br>https://login.microsoftonline.com |    Servicio Azure Stack Edge<br>Azure Service Bus<br>Servicio Authentication: Azure Active Directory                           |
|    http:\//crl.microsoft.com/pki/\*<br>http:\//www.microsoft.com/pki/\*                                                                                                                                                                                                                                                                                                                                                                                                  |    Revocación de certificados                                                                               |
|    https://\*.core.windows.net/\*<br>https://\*.data.microsoft.com<br>http://\*.msftncsi.com                                                                                                                                                                                                                                                                                                                                                                            |    Supervisión y cuentas de Almacenamiento de Azure                                                                |
|    http:\//windowsupdate.microsoft.com<br>http://\*.windowsupdate.microsoft.com<br>https://\*.windowsupdate.microsoft.com<br>http://\*.update.microsoft.com<br>https://\*.update.microsoft.com<br>http://\*.windowsupdate.com<br>http://download.microsoft.com<br>http://\*.download.windowsupdate.com<br>http://wustat.windows.com<br>http://ntservicepack.microsoft.com<br>http://\*.ws.microsoft.com<br>https://\*.ws.microsoft.com<br>http://\*.mp.microsoft.com |    Servidores de Microsoft Update                                                                             |
|    http://\*.deploy.akamaitechnologies.com                                                                                                                                                                                                                                                                                                                                                                                                                          |    CDN de Akamai                                                                                           |
|    http://\*.data.microsoft.com     |    Para información sobre el servicio de telemetría en Windows, consulte la actualización para la experiencia del usuario y la telemetría de diagnóstico. |
<!--|    http://www.msftconnecttest.com/connecttest.txt  |    En el caso de diagnósticos     ||  |-->   

