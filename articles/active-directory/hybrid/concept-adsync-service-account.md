---
title: 'Azure AD Connect: Cuenta de servicio de ADSync | Microsoft Docs'
description: En este tema se describe la cuenta de servicio de ADSync y se proporcionan procedimientos recomendados relacionados con la cuenta.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/17/2021
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: dfa329999cb7b53835907196ceaa9b02920da149
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108124264"
---
# <a name="adsync-service-account"></a>Cuenta del servicio ADSync
Azure AD Connect instala un servicio local que coordina la sincronización entre Active Directory y Azure Active Directory.  El servicio de sincronización de Microsoft Azure AD Sync (ADSync) se ejecuta en un servidor en su entorno local.  Las credenciales del servicio se establecen de forma predeterminada en las instalaciones rápidas, pero se pueden personalizar para satisfacer los requisitos de seguridad de su organización.  Estas credenciales no se usan para conectarse a los bosques locales ni a Azure Active Directory.

Elegir la cuenta de servicio de ADSync es una decisión importante de planificación que debe tomarse antes de instalar Azure AD Connect.  Cualquier intento de cambiar las credenciales tras la instalación conllevará que el servicio no se pueda iniciar, que se pierda el acceso a la base de datos de sincronización y que no se pueda autenticar con sus directorios conectados (Azure y AD DS).  No se producirá ninguna sincronización hasta que se restauren las credenciales originales.

El servicio de sincronización puede ejecutarse con diferentes cuentas. Puede ejecutarse con una cuenta de servicio virtual (VSA), una cuenta de servicio administrada (gMSA/sMSA), o una cuenta de usuario normal. Las opciones admitidas se cambiaron con la versión de abril de 2017 y la versión de marzo de 2021 de Azure AD Connect cuando realiza una instalación nueva. Si actualiza desde una versión anterior de Azure AD Connect, estas opciones adicionales no están disponibles. 


|Tipo de cuenta|Opción de instalación|Descripción| 
|-----|------|-----|
|cuenta de servicio virtual|Rápida y personalizada, abril de 2017 y versiones posteriores| Se utiliza una cuenta de servicio virtual para todas las instalaciones rápidas, excepto para las instalaciones en un controlador de dominio. Cuando se usa una instalación personalizada, es la opción predeterminada a menos que se use otra opción.| 
|Cuenta de servicio administrada|Personalizada, abril de 2017 y versiones posteriores|Si usa un servidor SQL remoto, se recomienda utilizar una cuenta de servicio administrada de grupo. |
|Cuenta de servicio administrada|Rápido y personalizado, marzo de 2021 y posteriores|Cuando se instala en un controlador de dominio, se crea una cuenta de servicio administrada con el prefijo ADSyncMSA_. Cuando se usa una instalación personalizada, es la opción predeterminada a menos que se use otra opción.|
|Cuenta de usuario|Rápida y personalizada, abril de 2017 a marzo de 2021|Se crea una cuenta de usuario con el prefijo AAD_ durante la instalación de las instalaciones rápidas cuando se instala en un controlador de dominio. Cuando se usa una instalación personalizada, es la opción predeterminada a menos que se use otra opción.|
|Cuenta de usuario|Rápida y personalizada, marzo de 2017 y versiones anteriores|Se crea una cuenta de usuario con el prefijo AAD_ durante la instalación para las instalaciones rápidas. Cuando se utiliza la instalación personalizada, se puede especificar otra cuenta.| 

>[!IMPORTANT]
> Si utiliza Connect con una compilación de marzo de 2017 o anterior, no debe restablecer la contraseña en la cuenta de servicio, ya que Windows destruye las claves de cifrado por motivos de seguridad. No se puede cambiar la cuenta a cualquier otra cuenta sin volver a instalar Azure AD Connect. Si se actualiza a una compilación de abril de 2017 o posterior, es posible cambiar la contraseña de la cuenta de servicio, pero no puede cambiar la cuenta utilizada. 

> [!IMPORTANT]
> Solo se puede establecer la cuenta de servicio en la primera instalación. No es posible cambiar la cuenta de servicio una vez que se ha completado la instalación. Si necesita cambiar la contraseña de la cuenta de servicio, se admite esta información y las instrucciones se pueden encontrar [aquí](how-to-connect-sync-change-serviceacct-pass.md).

La siguiente es una tabla de las opciones predeterminadas, recomendadas y admitidas para la cuenta del servicio de sincronización. 

Leyenda: 

- **Negrita** indica la opción predeterminada y, en la mayoría de los casos, la opción recomendada. 
- *Cursiva* indica la opción recomendada si no es la opción predeterminada. 
- Sin negrita: opción admitida 
- Cuenta local: cuenta de usuario local en el servidor 
- Cuenta de dominio: cuenta de usuario de dominio 
- sMSA: [cuenta de servicio administrada independiente](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd548356(v=ws.10))
- gMSA: [cuenta de servicio administrada de grupo](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831782(v=ws.11)) 

|Tipo de máquina |**LocalDB </br> Rápido**|**LocalDB/LocalSQL </br> Personalizado**|**SQL remoto </br> Personalizado**|
|-----|-----|-----|-----|
|**equipo unido a un dominio**|**VSA**|**VSA**</br> *sMSA*</br> *gMSA*</br> Cuenta local</br> Cuenta de dominio| *gMSA* </br>Cuenta de dominio|
|Controlador de dominio| **sMSA**|**sMSA** </br>*gMSA*</br> Cuenta de dominio|*gMSA*</br>Cuenta de dominio| 

## <a name="virtual-service-account"></a>cuenta de servicio virtual 

Una cuenta de servicio virtual es un tipo especial de cuenta local administrada que no tiene contraseña y es administrada automáticamente por Windows. 

 ![Cuenta de servicio virtual](media/concept-adsync-service-account/account-1.png)

La cuenta de servicio virtual está pensada para utilizarse con escenarios en los que el motor de sincronización y SQL están en el mismo servidor. Si usa SQL remoto, es aconsejable utilizar una cuenta de servicio administrada de grupo en su lugar. 

La cuenta de servicio virtual no se puede usar en un controlador de dominio debido a problemas de la [API de protección de datos (DPAPI) de Windows](/previous-versions/ms995355(v=msdn.10)). 

## <a name="managed-service-account"></a>Cuenta de servicio administrada 

Si usa un servidor SQL remoto, se recomienda utilizar una cuenta de servicio administrada de grupo. Para más información sobre cómo preparar Active Directory para la cuenta de servicio administrada de grupo, consulte [Información general de las cuentas de servicio administradas de grupo](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831782(v=ws.11)). 

Para usar esta opción, en la página [Instalar los componentes necesarios](how-to-connect-install-custom.md#install-required-components), seleccione **Usar una cuenta de servicio existente** y seleccione **Cuenta de servicio administrada**. 

 ![cuenta de servicio administrada](media/concept-adsync-service-account/account-2.png)

También se puede usar una cuenta de servicio administrada independiente. Sin embargo, solo se pueden utilizar en el equipo local y no hay ningún beneficio al usarlas en lugar de la cuenta de servicio virtual predeterminada. 

### <a name="auto-generated-standalone-managed-service-account"></a>Cuenta de servicio administrada independiente 

Si instala Azure AD Connect en un controlador de dominio, el asistente de instalación crea una cuenta de servicio administrada independiente (a menos que especifique la cuenta para usar en la configuración personalizada). La cuenta lleva delante **ADSyncMSA_** y se usa para el servicio de sincronización real como cuenta de ejecución. 

Esta cuenta es una cuenta de dominio administrada que no tiene contraseña y es administrada automáticamente por Windows. 

Esta cuenta está diseñada para usarse con escenarios donde el motor de sincronización y SQL están en el controlador de dominio. 

## <a name="user-account"></a>Cuenta de usuario 

El asistente para instalación crea una cuenta de servicio local (a menos que especifique la cuenta que se desea usar en la configuración personalizada). La cuenta lleva delante AAD_ y se usa para el servicio de sincronización real como cuenta de ejecución. Si instala Azure AD Connect en un controlador de dominio, la cuenta se crea en el dominio. La cuenta de servicio AAD_ debe estar ubicada en el dominio si: 
- Se usa un servidor remoto que ejecuta SQL Server 
- Se usa a un proxy que requiere autenticación. 

 ![cuenta de usuario](media/concept-adsync-service-account/account-3.png)

La cuenta se crea con una contraseña larga compleja que no expira. 

Esta se utiliza para almacenar de forma segura las contraseñas de las otras cuentas. que se almacenan cifradas en la base de datos. Las claves privadas de las claves de cifrado se protegen con el cifrado de clave secreta de los servicios criptográficos mediante la API de protección de datos de Windows (DPAPI). 

Si usa SQL Server completo, la cuenta de servicio es el DBO de la base de datos creada para el motor de sincronización. El servicio no funcionará como se pretende con ningún otro permiso. También se crea un inicio de sesión SQL. 

A la cuenta también se le conceden permisos para archivos, claves del Registro y otros objetos relacionados con el motor de sincronización. 


## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre la [Integración de las identidades locales con Azure Active Directory](whatis-hybrid-identity.md).