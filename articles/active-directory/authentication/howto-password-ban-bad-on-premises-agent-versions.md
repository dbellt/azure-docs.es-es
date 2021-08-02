---
title: 'Historial de versiones del agente de protección con contraseña: Azure Active Directory'
description: Historial de cambios de comportamiento y versiones de los documentos
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: article
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0698d07268c416b0a10055ec5480971fc10c142c
ms.sourcegitcommit: 67cdbe905eb67e969d7d0e211d87bc174b9b8dc0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111854385"
---
# <a name="azure-ad-password-protection-agent-version-history"></a>Historial de versiones del agente de protección con contraseña de Azure AD

## <a name="121760"></a>1.2.176.0

Fecha de lanzamiento: 4 de junio de 2021

* Correcciones de errores menores para problemas que impidieron que el proxy y los agentes de controlador de dominio se ejecutaran correctamente en determinados entornos.

## <a name="121720"></a>1.2.172.0

Fecha de lanzamiento: 22 de febrero de 2021

Ha transcurrido casi dos años desde que se lanzaron las versiones de disponibilidad general de los agentes de protección de contraseñas de Azure AD local. Ahora hay disponible una nueva actualización; vea las descripciones de los cambios a continuación. Gracias a todos los usuarios que nos han hecho llegar sus comentarios sobre el producto. 

* El agente del controlador de dominio y el software del agente proxy requieren ahora .NET 4.7.2 para instalarse.
  * Si .NET 4.7.2 no está instalado aún, descargue y ejecute el instalador que se encuentra en [el instalador sin conexión de .NET Framework 4.7.2 para Windows](https://support.microsoft.com/topic/microsoft-net-framework-4-7-2-offline-installer-for-windows-05a72734-2127-a15d-50cf-daf56d5faec2).
* El módulo AzureADPasswordProtection de PowerShell también se instala ahora mediante el software del agente del controlador de dominio.
* Se han agregado dos nuevos cmdlets de PowerShell relacionados con el mantenimiento: Test-AzureADPasswordProtectionDCAgent y Test-AzureADPasswordProtectionProxy.
* El archivo. dll del filtro de contraseñas del agente del controlador de dominio AzureADPasswordProtection ahora se cargará y ejecutará en las máquinas en los que lsass.exe esté configurado para ejecutarse en modo PPL.
* Se ha corregido un error en el algoritmo de contraseñas que permitía aceptar incorrectamente las contraseñas prohibidas de menos de cinco caracteres.
  * Este error solo es aplicable si la directiva de longitud mínima de contraseñas de AD local se configuró para permitir menos de cinco contraseñas de caracteres en primer lugar.
* Otras correcciones de errores menores.

Los nuevos instaladores actualizarán automáticamente las versiones anteriores del software. Si ha instalado el agente del controlador de dominio y el software proxy en una sola máquina (solo se recomienda para entornos de prueba), debe actualizar ambos al mismo tiempo.

Se admite la ejecución de versiones anteriores y recientes del agente del controlador de dominio y el software proxy en un dominio o bosque, aunque se recomienda actualizar todos los agentes a la versión más reciente. Es indiferente el orden en las actualizaciones de los agentes: los nuevos agentes del controlador de dominio pueden comunicarse a través de agentes proxy más antiguos y los agentes del controlador de dominio más antiguos pueden comunicarse a través de agentes proxy más recientes.

## <a name="121250"></a>1.2.125.0

Fecha de lanzamiento: 2 de marzo de 2019

* Corregir pequeños errores tipográficos en los mensajes de registro de eventos
* Actualizar el contrato CLUF a la versión final de disponibilidad general

> [!NOTE]
> La compilación 1.2.125.0 es la compilación de disponibilidad general. Gracias de nuevo a todos los que han dado su opinión sobre el producto.

## <a name="121160"></a>1.2.116.0

Fecha de lanzamiento: 3 de marzo de 2019

* Los cmdlets Get-AzureADPasswordProtectionProxy y Get-AzureADPasswordProtectionDCAgent ahora informan de la versión de software y del inquilino actual de Azure con las siguientes limitaciones:
  * La versión de software y los datos del inquilino de Azure solo están disponibles para los agentes de controlador de dominio y servidores proxy que ejecuten la versión 1.2.116.0 o posterior.
  * Los datos de los inquilinos de Azure no se pueden informar hasta que se haya producido un nuevo registro (o renovación) del proxy o del bosque.
* El servicio de proxy requiere ahora que esté instalado .NET 4.7.
  * Si .NET 4.7 no está instalado aún, descargue y ejecute el instalador que se encuentra en [el instalador sin conexión de .NET Framework 4.7 para Windows](https://support.microsoft.com/help/3186497/the-net-framework-4-7-offline-installer-for-windows).
  * En los sistemas Server Core puede que sea necesario pasar la marca /q al instalador de .NET 4.7 para que se realice correctamente.
* El servicio de proxy ahora admite la actualización automática. La actualización automática utiliza el servicio Agent Updater de Microsoft Azure AD Connect, que se instala junto con el servicio de proxy. La actualización automática está activada de forma predeterminada.
* La actualización automática se puede habilitar o deshabilitar con el cmdlet Set-AzureADPasswordProtectionProxyConfiguration. La configuración actual se puede consultar mediante el cmdlet Get-AzureADPasswordProtectionProxyConfiguration.
* El binario del servicio para el servicio de agente de controlador de dominio ha cambiado a AzureADPasswordProtectionDCAgent.exe.
* Ha cambiado el binario del servicio para el servicio de proxy a AzureADPasswordProtectionProxy.exe. Es posible que haya que modificar las reglas del firewall en consecuencia si se utiliza un firewall de terceros.
  * NOTA: Si se estaba utilizando un archivo de configuración de proxy http en una instalación anterior del servicio de proxy, habrá que cambiarle el nombre (de *proxyservice.exe.config* a *AzureADPasswordProtectionProxy.exe.config*) después de esta actualización.
* Todas las comprobaciones de funcionalidad limitadas en el tiempo se han eliminado del agente de DC.
* Correcciones de errores menores y mejoras en el registro.

## <a name="12650"></a>1.2.65.0

Fecha de lanzamiento: 1 de febrero de 2019

Cambios:

* Ahora se admiten el servicio de proxy y el agente de controlador de dominio en Server Core. Los requisitos mínimos del sistema operativo son los mismos que antes: Windows Server 2012 para agentes de controlador de dominio, y Windows Server 2012 R2 para los servidores proxy.
* Los cmdlets Register-AzureADPasswordProtectionProxy y Register-AzureADPasswordProtectionForest ahora admiten los modos de autenticación de Azure basada en código de dispositivo.
* El cmdlet Get-AzureADPasswordProtectionDCAgent omitirá los puntos de conexión de servicio alterados o no válidos. Este cambio corrige el error por el que los controladores de dominio en ocasiones aparecen varias veces en la salida.
* El cmdlet Get-AzureADPasswordProtectionSummaryReport omitirá los puntos de conexión de servicio alterados o no válidos. Este cambio corrige el error por el que los controladores de dominio en ocasiones aparecen varias veces en la salida.
* El módulo de PowerShell del servidor proxy ahora se registra desde %ProgramFiles%\WindowsPowerShell\Modules. Ya no se modifica la variable de entorno PSModulePath del equipo.
* Se agregó un nuevo cmdlet Get-AzureADPasswordProtectionProxy para ayudar a detectar servidores proxy registrados en un bosque o dominio.
* El agente de controlador de dominio usa una nueva carpeta en el recurso compartido sysvol para replicar las directivas de contraseñas y otros archivos.

   Ubicación de la carpeta antigua:

   `\\<domain>\sysvol\<domain fqdn>\Policies\{4A9AB66B-4365-4C2A-996C-58ED9927332D}`

   Ubicación de la carpeta nueva:

   `\\<domain>\sysvol\<domain fqdn>\AzureADPasswordProtection`

   (Este cambio se realizó para evitar advertencias de "GPO huérfano" falso positivas).

   > [!NOTE]
   > No se realizará ninguna migración ni intercambio de datos entre la antigua carpeta y la nueva carpeta. Las versiones anteriores del agente de controlador de dominio seguirán usando la ubicación anterior hasta que actualice a esta versión o una versión posterior. Una vez que todos los agentes de controlador de dominio ejecuten la versión 1.2.65.0 o posterior, se podrá eliminar manualmente la carpeta sysvol anterior.

* El agente de controlador de dominio y servicio de proxy ahora detectarán y eliminarán las copias alteradas de sus puntos de conexión de servicio correspondientes.
* Cada agente de controlador de dominio eliminará periódicamente los puntos de conexión de servicio alterados y obsoletos en su dominio, para puntos de conexión tanto del agente de controlador de dominio como del servicio de proxy. Los puntos de conexión del agente de controlador de dominio y del servicio de proxy se consideran obsoletos si su marca de tiempo de latido es anterior a siete días.
* El agente de controlador de dominio ahora renovará el certificado del bosque según sea necesario.
* El servicio de proxy ahora renovará el certificado del servidor proxy según sea necesario.
* Actualizaciones al algoritmo de validación de contraseña: la lista global de contraseñas prohibidas y la lista de contraseñas prohibidas específica del cliente (si está configurada) se combinan antes de las validaciones de contraseñas. Ahora se puede rechazar una contraseña determinada (error o solo auditoría) si contiene los tokens tanto de la lista global como de la específica del cliente. La documentación de registro de eventos se ha actualizado para reflejar esto; consulte [Supervisión de la protección con contraseña de Azure AD](howto-password-ban-bad-on-premises-monitor.md).
* Correcciones de rendimiento y estabilidad
* Registro mejorado

> [!WARNING]
> Funcionalidad de tiempo limitado: el servicio del agente de controlador de dominio en esta versión (1.2.65.0) detendrá el procesamiento de solicitudes de validación de contraseñas a partir del 1 de septiembre de 2019.  Los servicios del agente de controlador de dominio en versiones anteriores (consulte la lista siguiente) detendrán el procesamiento a partir del 1 de julio de 2019. El servicio del agente de controlador de dominio en todas las versiones registrará eventos 10021 en el registro de eventos de administración en los dos meses anteriores a estas fechas límite. Se quitarán todas las restricciones de límite de tiempo en la próxima versión de disponibilidad general. El servicio del agente de proxy no está limitado por el tiempo en ninguna versión, pero aun así se debe actualizar a la versión más reciente para sacar provecho de todas las de errores posteriores y otras mejoras correcciones.

## <a name="12250"></a>1.2.25.0

Fecha de lanzamiento: 1 de noviembre de 2018

Correcciones:

* El agente de controlador de dominio y el servicio de proxy ya no deberían funcionar mal debido a errores de confianza de certificados.
* El agente del controlador de dominio y el servicio de proxy tienen correcciones para máquinas compatibles con FIPS.
* El servicio de proxy funcionará ahora correctamente en un entorno de red solo TLS 1.2.
* Correcciones menores de rendimiento y estabilidad
* Registro mejorado

Cambios:

* El nivel de sistema operativo mínimo necesario para el servicio de proxy es ahora Windows Server 2012 R2. El nivel de sistema operativo mínimo necesario para el servicio de agente de controlador de dominio sigue siendo Windows Server 2012.
* El servicio de proxy ahora requiere .NET versión 4.6.2.
* El algoritmo de validación de contraseña usa una tabla de normalización de caracteres ampliada. Este cambio puede tener como consecuencia que se rechacen contraseñas que se aceptaron en versiones anteriores.

## <a name="12100"></a>1.2.10.0

Fecha de lanzamiento: 17 de agosto de 2018

Correcciones:

* Register-AzureADPasswordProtectionProxy y Register-AzureADPasswordProtectionForest ya admiten la autenticación multifactor.
* Register-AzureADPasswordProtectionProxy requiere un controlador de dominio WS2012 o posterior en el dominio para evitar los errores de cifrado.
* El servicio del agente del controlador de dominio es más confiable en cuanto a la solicitud de una directiva de contraseñas nueva de Azure en el inicio.
* El servicio del agente del controlador de dominio solicitará una directiva de contraseñas nueva de Azure cada hora si es necesario, pero ahora lo hará en una hora de inicio seleccionada aleatoriamente.
* El servicio del agente del controlador de dominio ya no producirá un retraso indefinido en el nuevo anuncio del controlador de dominio cuando se instale en un servidor antes de su promoción como una réplica.
* El servicio del agente del controlador de dominio ahora admite la opción de configuración "Habilitar protección con contraseña en Windows Server Active Directory".
* Ahora, los instaladores de proxy y agente del controlador de dominio son compatibles con la actualización local al realizar la actualización a versiones futuras.

> [!WARNING]
> La actualización local de la versión 1.1.10.3 no se admite y producirá un error de instalación. Para actualizar a la versión 1.2.10 o posterior, primero deberá desinstalar por completo el software del servicio de proxy y el agente de controlador de dominio y, luego, deberá instalar la nueva versión desde cero. Es necesario eliminar el registro del servicio de proxy de protección con contraseña de Azure AD.  No es necesario volver a registrar el bosque.

> [!NOTE]
> Las actualizaciones locales del software del agente del controlador de dominio requerirán un reinicio.

* El servicio de proxy y de agente del controlador de dominio ahora se pueden ejecutar en un servidor configurado para usar solo algoritmos conformes a FIPS.
* Correcciones menores de rendimiento y estabilidad
* Registro mejorado

## <a name="11103"></a>1.1.10.3

Fecha de lanzamiento: 15 de junio de 2018

Versión preliminar pública inicial

## <a name="next-steps"></a>Pasos siguientes

[Implementación de la protección de contraseñas de Azure AD](howto-password-ban-bad-on-premises-deploy.md)
