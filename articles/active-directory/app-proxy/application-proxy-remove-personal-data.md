---
title: Quitar datos personales - Azure Active Directory Application Proxy
description: Quitar datos personales de los conectores instalados en los dispositivos para Azure Active Directory Application Proxy.
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: how-to
ms.date: 04/28/2021
ms.author: kenwith
ms.reviewer: harshja
ms.openlocfilehash: e1bfc51e4770bfb9d9aafd8f3a3e6da4d9bd8c68
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108230025"
---
# <a name="remove-personal-data-for-azure-active-directory-application-proxy"></a>Quitar datos personales para Azure Active Directory Application Proxy

Azure Active Directory Application Proxy requiere la instalación de los conectores en los dispositivos, lo que significa que podría haber datos personales en los dispositivos. En este artículo se indican los pasos a seguir para eliminar datos personales con el fin de mejorar la privacidad.

## <a name="where-is-the-personal-data"></a>¿Dónde están los datos personales?

Es posible que Application Proxy escriba datos personales en los siguientes tipos de registro:

- Registros de eventos de conector
- Registros de eventos de Windows

## <a name="remove-personal-data-from-windows-event-logs"></a>Quitar datos personales de los registros de eventos de Windows

Para más información sobre cómo configurar la retención de datos de los registros de eventos de Windows, consulte [Settings for event logs](https://technet.microsoft.com/library/cc952132.aspx) (Configuración de registro de eventos). Para obtener información sobre los registros de eventos de Windows, vea [Using Windows Event Log](/windows/win32/wes/using-windows-event-log) (Uso del registro de eventos de Windows).

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-hybrid-note.md)]

## <a name="remove-personal-data-from-connector-event-logs"></a>Quitar datos personales de los registros de eventos de conector

Para asegurarse de que los registros de Application Proxy no incluyen datos personales, puede hacer lo siguiente:

- Eliminar o ver los datos cuando sea necesario, o
- Desactivación del registro

Lea las secciones siguientes para quitar datos personales de los registros de eventos de conector. El proceso de eliminación debe realizarse para todos los dispositivos en los que está instalado el conector.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

### <a name="view-or-export-specific-data"></a>Ver o exportar datos específicos

Para ver o exportar datos específicos, busque las correspondientes entradas en cada uno de los registros de eventos del conector. Los registros se encuentran en `C:\ProgramData\Microsoft\Microsoft AAD Application Proxy Connector\Trace`.

Puesto que los registros son archivos de texto, puede usar [findstr](/windows-server/administration/windows-commands/findstr) para buscar entradas de texto relacionadas con un usuario.  

Para encontrar datos personales, busque el id. de usuario en los archivos de registro.

Para encontrar datos personales registrados por una aplicación que utiliza la delegación limitada de Kerberos, busque estos componentes del tipo de nombre de usuario:

- Nombre principal de usuario local
- Parte del nombre de usuario del nombre principal de usuario
- Parte del nombre de usuario del nombre principal de usuario local
- Nombre de la cuenta de administrador de cuentas de seguridad (SAM) locales

### <a name="delete-specific-data"></a>Eliminar datos específicos

Para eliminar datos específicos:

1. Reinicie el servicio de conector de Microsoft Azure AD Application Proxy para generar un nuevo archivo de registro. El nuevo archivo de registro le permite eliminar o modificar los archivos de registro antiguos. 
1. Siga el proceso [Ver o exportar datos específicos](#view-or-export-specific-data) descrito anteriormente para buscar la información que debe eliminarse. Busque en todos los registros de conector.
1. Elimine los archivos de registro correspondientes o elimine de forma selectiva los campos que contienen datos personales. También puede eliminar todos los archivos de registro antiguos si ya no los necesita.

### <a name="turn-off-connector-logs"></a>Desactivación de los registros de conector

Una opción para asegurarse de que el conector no contiene datos personales es desactivar la generación de registros. Para dejar de generar registros de conector, quite la siguiente línea resaltada de `C:\Program Files\Microsoft AAD App Proxy Connector\ApplicationProxyConnectorService.exe.config`.

![Muestra un fragmento de código con el código resaltado que se va a quitar](./media/application-proxy-remove-personal-data/01.png)

## <a name="next-steps"></a>Pasos siguientes

Para obtener información general sobre Application Proxy, consulte [Provisión de acceso remoto seguro a aplicaciones locales](application-proxy.md).