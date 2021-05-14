---
title: Validación controlada de la unión a Azure AD híbrido - Azure AD
description: Obtenga información sobre cómo realizar una validación controlada de la unión a Azure AD híbrido antes de habilitarla en toda la organización a la vez.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 06/28/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 50e3272be139e8b0cd30f9e5867e263efc92ae24
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108178761"
---
# <a name="controlled-validation-of-hybrid-azure-ad-join"></a>Validación controlada de la unión a Azure AD híbrido

Una vez cumplidos todos los requisitos previos, los dispositivos Windows se registrarán automáticamente como dispositivos en el inquilino de Azure AD. El estado de estas identidades de dispositivo en Azure AD se conoce como unión a Azure AD híbrido. Puede obtener más información sobre los conceptos tratados aquí en los artículos [Introducción a la administración de dispositivos en Azure Active Directory](overview.md) y [Planear la implementación de la unión a Azure Active Directory híbrido](hybrid-azuread-join-plan.md).

Es posible las organizaciones estén interesadas en realizar una validación controlada de la unión a Azure AD híbrido antes de habilitarla en toda la organización a la vez. En este artículo se explica cómo llevar a cabo una validación controlada de la unión a Azure AD híbrido.

## <a name="controlled-validation-of-hybrid-azure-ad-join-on-windows-current-devices"></a>Validación controlada de la unión a Azure AD híbrido en dispositivos Windows actuales

Para dispositivos que ejecutan el sistema operativo de escritorio de Windows, la versión admitida es la Actualización de aniversario de Windows 10 (versión 1607), o una versión posterior. Como procedimiento recomendado, actualice a la versión más reciente de Windows 10.

Para llevar a cabo una validación controlada de la unión a Azure AD híbrido en dispositivos Windows actuales, debe hacer lo siguiente:

1. Borrar la entrada del punto de conexión de servicio (SCP) de Active Directory (AD), si existe
1. Configurar el registro del cliente de SCP en los equipos unidos a un dominio con un objeto de directiva de grupo (GPO)
1. Si utiliza AD FS, también debe configurar el registro del cliente de SCP en el servidor de AD FS mediante un GPO  
1. Es posible que también necesite [personalizar las opciones de sincronización](../hybrid/how-to-connect-post-installation.md#additional-tasks-available-in-azure-ad-connect) de Azure AD Connect para habilitar la sincronización de dispositivos. 


### <a name="clear-the-scp-from-ad"></a>Borrar el SCP de AD

Utilice el Editor de interfaces de servicios de Active Directory (ADSI Edit) para modificar los objetos de SCP de AD.

1. Inicie la aplicación de escritorio **ADSI Edit** desde una estación de trabajo administrativa o un controlador de dominio como administrador de Enterprise.
1. Conéctese al **contexto de nomenclatura de configuración** del dominio.
1. Vaya a **CN = Configuración, DC = contoso, DC = com** > **CN = Servicios** > **CN = Configuración del registro del dispositivo**.
1. Haga clic con el botón derecho en el objeto hoja **CN=62a0ff2e-97b9-4513-943f-0d221bd30080** y seleccione **Propiedades**.
   1. Seleccione **palabras clave** en la ventana del **Editor de atributos** y haga clic en **Editar**.
   1. Seleccione los valores de **azureADId** y **azureADName** (de uno en uno) y haga clic en **Quitar**.
1. Cierre **Editor ADSI**.


### <a name="configure-client-side-registry-setting-for-scp"></a>Configuración del registro del cliente de SCP

Use el ejemplo siguiente para crear un objeto de directiva de grupo (GPO) y configurar una entrada de SCP en el registro de los dispositivos para implementar una configuración del registro.

1. Abra una consola de administración de directivas de grupo y cree un nuevo objeto de directiva de grupo en el dominio.
   1. Asigne un nombre al GPO recién creado (por ejemplo, ClientSideSCP).
1. Edite el GPO y busque la ruta de acceso siguiente: **Configuración del equipo** > **Preferencias** > **Configuración de Windows** > **Registro**.
1. Haga doble clic en el registro y seleccione **Nuevo** > **Elemento del registro**.
   1. En la pestaña **General**, configure lo siguiente:
      1. Acción: **Actualizar**
      1. Hive: **HKEY_LOCAL_MACHINE**
      1. Ruta de acceso de la clave: **SOFTWARE\Microsoft\Windows\CurrentVersion\CDJ\AAD**
      1. Nombre del valor: **TenantId**
      1. Tipo de valor: **REG_SZ**
      1. Datos del valor: El GUID o el **Identificador de inquilino** de la instancia de Azure AD (este valor puede encontrarse en **Azure Portal** > **Azure Active Directory** > **Propiedades** > **Identificador de inquilino**)
   1. Haga clic en **Aceptar**
1. Haga doble clic en el registro y seleccione **Nuevo** > **Elemento del registro**.
   1. En la pestaña **General**, configure lo siguiente:
      1. Acción: **Actualizar**
      1. Hive: **HKEY_LOCAL_MACHINE**
      1. Ruta de acceso de la clave: **SOFTWARE\Microsoft\Windows\CurrentVersion\CDJ\AAD**
      1. Nombre del valor: **TenantName**
      1. Tipo de valor: **REG_SZ**
      1. Datos del valor: El **nombre de dominio** comprobado si usa un entorno federado como AD FS. El **nombre de dominio** comprobado o el nombre de dominio de onmicrosoft.com (por ejemplo, `contoso.onmicrosoft.com`) si usa un entorno administrado.
   1. Haga clic en **Aceptar**
1. Cierre el editor para el GPO recién creado.
1. Vincule el GPO recién creado a la OU deseada que contiene equipos unidos a un dominio que pertenecen a la población de lanzamiento controlado

### <a name="configure-ad-fs-settings"></a>Configuración de AD FS

Si utiliza AD FS, primero deberá configurar el SCP del cliente mediante las instrucciones mencionadas anteriormente mediante la vinculación del GPO a los servidores de AD FS. El objeto SCP define el origen de autoridad de los objetos del dispositivo. Puede ser local o Azure AD. Cuando el SCP del cliente se configura para AD FS, el origen de los objetos del dispositivo se establece como Azure AD.

> [!NOTE]
> Si no pudo configurar SCP en el cliente en los servidores de AD FS, el origen de las identidades de los dispositivos se considerará local. ADFS comenzará a eliminar los objetos de dispositivo del directorio local después del período estipulado definido en el atributo "MaximumInactiveDays" del registro de dispositivos de ADFS. Los objetos del registro de dispositivos de ADFS se pueden encontrar con el [cmdlet Get-AdfsDeviceRegistration](/powershell/module/adfs/get-adfsdeviceregistration).

## <a name="controlled-validation-of-hybrid-azure-ad-join-on-windows-down-level-devices"></a>Validación controlada de la unión a Azure AD híbrido en dispositivos Windows de nivel inferior

Para registrar dispositivos de nivel inferior de Windows, las organizaciones deben instalar [Microsoft Workplace Join para equipos sin Windows 10](https://www.microsoft.com/download/details.aspx?id=53554) que se encuentra disponible en el Centro de descarga de Microsoft.

El paquete se puede implementar mediante un sistema de distribución de software como  [Microsoft Endpoint Configuration Manager](/configmgr/). El paquete admite las opciones de instalación silenciosa estándar mediante el parámetro quiet. La rama actual de Configuration Manager ofrece ventajas adicionales sobre las versiones anteriores, como la posibilidad de realizar el seguimiento de los registros completados.

El instalador crea una tarea programada en el sistema que se ejecuta en el contexto del usuario. La tarea se desencadena cuando el usuario inicia sesión en Windows. La tarea une de forma silenciosa el dispositivo con Azure AD con las credenciales de usuario después de realizar la autenticación con Azure AD.

Para controlar el registro de dispositivos, debe implementar el paquete de Windows Installer en el grupo seleccionado de dispositivos Windows de nivel inferior.

> [!NOTE]
> Si no se configura un SCP en Active Directory, debe seguir el mismo enfoque tal como se describe en [Configuración del registro del cliente de SCP](#configure-client-side-registry-setting-for-scp) en los equipos unidos a un dominio con un objeto de directiva de grupo (GPO).


Después de comprobar que todo funciona según lo previsto, puede registrar automáticamente el resto de los dispositivos Windows actuales y de nivel inferior en Azure AD; para ello, debe [configurar SCP con Azure AD Connect](hybrid-azuread-join-managed-domains.md#configure-hybrid-azure-ad-join).

## <a name="next-steps"></a>Pasos siguientes

[Planeamiento de la implementación de la unión a Azure Active Directory híbrido](hybrid-azuread-join-plan.md)
