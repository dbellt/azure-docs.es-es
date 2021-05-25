---
title: Instalación del agente de aprovisionamiento de Azure AD Connect
description: Aprenda a instalar el agente de aprovisionamiento de Azure AD Connect y cómo configurarlo en Azure Portal.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/16/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0d83b617f08c143ee09251667c4f9bee89577a23
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108758868"
---
# <a name="install-the-azure-ad-connect-provisioning-agent"></a>Instalación del agente de aprovisionamiento de Azure AD Connect
Este documento le guiará en el proceso de instalación del agente de aprovisionamiento de Azure Active Directory (Azure AD) Connect y en el modo de configurarlo inicialmente en Azure Portal.

>[!IMPORTANT]
>En las siguientes instrucciones de instalación se supone que se han cumplido todos los [requisitos previos](how-to-prerequisites.md).

La instalación y configuración de la sincronización en la nube de Azure AD Connect se realiza en los pasos siguientes:

- [Cuentas de servicio administradas de grupo](#group-managed-service-accounts) 
- [Instalación del agente](#install-the-agent)
- [Comprobación de la instalación del agente](#verify-agent-installation)

>[!NOTE]
>En este documento se habla de la instalación del agente de aprovisionamiento mediante el asistente.  Para obtener información sobre cómo instalar el agente de Azure AD Connect mediante una interfaz de línea de comandos (CLI), consulte [Instalación del agente de aprovisionamiento de Azure AD Connect mediante una interfaz de línea de comandos (CLI) y PowerShell](how-to-install-pshell.md).

## <a name="group-managed-service-accounts"></a>Cuentas de servicio administradas de grupo
Una cuenta de servicio administradas de grupo es una cuenta de dominio administrado que proporciona administración automática de contraseñas, administración simplificada del nombre de entidad de seguridad de servicio (SPN), la posibilidad de delegar la administración a otros administradores y, además, amplía esta funcionalidad a varios servidores.  La sincronización en la nube de Azure AD Connect admite y recomienda el uso de una cuenta de servicio administrada de grupo para ejecutar el agente.  Para obtener más información sobre gMSA, consulte [Cuentas de servicio administradas de grupo](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview). 


### <a name="upgrading-an-existing-agent-to-use-the-gmsa-account"></a>Actualización de un agente existente para usar la cuenta gMSA
Para actualizar un agente existente para que use la cuenta gMSA creada durante la instalación, simplemente actualice el servicio del agente a la versión más reciente mediante la ejecución de AADConnectProvisioningAgent.msi.  Esto actualizará el servicio a la versión más reciente.  Ahora vuelva a ejecutar el asistente para la instalación y escriba las credenciales para crear la cuenta cuando se lo solicite.





## <a name="install-the-agent"></a>Instalación del agente
Para instalar el agente, siga estos pasos.

 1. Inicie sesión en el servidor que va a usar con permisos de administrador de empresa.
 2. Inicie sesión en Azure Portal y, a continuación, vaya a **Azure Active Directory**.
 3. En el menú de la izquierda, seleccione **Azure AD Connect**.
 4. Seleccione **Manage cloud sync** (Administrar sincronización en la nube)  > **Revisar todos los agentes**.
 5. Descargue el agente de aprovisionamiento de Azure AD Connect en Azure Portal.
   ![Descargar el agente local](media/how-to-install/install-9.png)</br>
 6. Acepte los términos y haga clic en Descargar.
 7. Ejecute el instalador de aprovisionamiento de Azure AD Connect, AADConnectProvisioningAgentSetup.msi.
 8. En la pantalla **Microsoft Azure AD Connect Provisioning Agent Package**, acepte los términos de licencia y seleccione **Instalar**.
   ![Pantalla Microsoft Azure AD Connect Provisioning Agent Package](media/how-to-install/install-1.png)</br>
 9. Una vez finalizada esta operación, se inicia el Asistente para configuración. Inicie sesión con su cuenta de administrador global de Azure AD.
 10. En la **pantalla Configurar cuenta de servicio**, seleccione **Create gMSA** (Crear gMSA) o **Use Custom gMSA** (Usar gMSA personalizada).  Si permite que el agente cree la cuenta, se denominará provAgentgMSA$. Si especifica **Use custom gMSA** (Usar gMSA personalizada) se le pedirá que proporcione esta cuenta.
 11. Escriba las credenciales de administrador de dominio para crear la cuenta de servicio administrada de grupo que se usará para ejecutar el servicio del agente. Haga clic en **Siguiente**.  
   ![Crear gMSA](media/how-to-install/install-12.png)</br>
 12. En la pantalla **Conectar Active Directory**, seleccione **Agregar directorio**. Después, inicie sesión con su cuenta de administrador de Active Directory. Esta operación permitirá agregar su directorio local. 
 13. De manera opcional, puede administrar la preferencia de los controladores de dominio que usará el agente; para ello, seleccione **Select domain controller priority** (Seleccionar prioridad del controlador de dominio) y ordene la lista de controladores de dominio.   Haga clic en **OK**.
  ![Solicitar controladores de dominio](media/how-to-install/install-2a.png)</br>
 14. Seleccione **Next** (Siguiente).
  ![Pantalla Conexión de Active Directory](media/how-to-install/install-3a.png)</br>
 15.  En la pantalla **Agent Installation** (Instalación del agente), confirme la configuración y la cuenta que se creará y haga clic en **Confirmar**.
  ![Confirmar configuración](media/how-to-install/install-11.png)</br>
 16. Una vez finalizada esta operación, debería ver **Your agent installation is complete** (Se completó la instalación del agente). Seleccione **Salir**.
  ![Pantalla de configuración finalizada](media/how-to-install/install-4a.png)</br>
 17. Si todavía ve la pantalla inicial **Microsoft Azure AD Connect Provisioning Agent Package**, seleccione **Cerrar**.

## <a name="verify-agent-installation"></a>Comprobación de la instalación del agente
La comprobación del agente se produce en Azure Portal y en el servidor local que ejecuta el agente.

### <a name="azure-portal-agent-verification"></a>Comprobación del agente en Azure Portal
Para comprobar que Azure ve el agente, siga estos pasos.

 1. Inicie sesión en Azure Portal.
 2. En la parte izquierda, seleccione **Azure Active Directory** > **Azure AD Connect**. En el centro, seleccione **Manage cloud sync** (Administrar sincronización en la nube).

   ![Portal de Azure](media/how-to-install/install-6.png)</br>

 3.  En la pantalla **Azure AD Connect cloud sync** (Sincronización en la nube de Azure AD Connect), seleccione **Revisar todos los agentes**.

   ![Opción Revisar todos los agentes](media/how-to-install/install-7.png)</br>
 
 4. En la pantalla **Agentes de aprovisionamiento locales**, verá los agentes que ha instalado. Compruebe que el agente en cuestión está ahí y que se ha marcado como *Activo*.

   ![Pantalla de agentes de aprovisionamiento local](media/how-to-install/verify-1.png)</br>



### <a name="on-the-local-server"></a>En el servidor local
Para comprobar que el agente se ejecuta, siga estos pasos.

1.  Inicie sesión en el servidor con una cuenta de administrador.
1.  Abra **Servicios**. Para ello, vaya ahí o a **Inicio** > **Ejecutar** > **Services.msc**.
1.  En **Servicios**, asegúrese de que tanto el **Actualizador del Agente de Microsoft Azure AD Connect** como el **Agente de aprovisionamiento de Microsoft Azure AD Connect** están ahí y que su estado es *En ejecución*.

    ![Pantalla de servicios](media/how-to-install/troubleshoot-1.png)

>[!IMPORTANT]
>El agente se ha instalado, pero debe configurarse y habilitarse antes de que empiece a sincronizar a los usuarios. Para configurar un nuevo agente, consulte [Creación de una configuración para la sincronización en la nube de Azure AD Connect](how-to-configure.md).




## <a name="next-steps"></a>Pasos siguientes 

- [¿Qué es el aprovisionamiento?](what-is-provisioning.md)
- [¿Qué es la sincronización en la nube de Azure AD Connect?](what-is-cloud-sync.md)
- [Creación de una configuración para la sincronización en la nube de Azure AD Connect](how-to-configure.md).

