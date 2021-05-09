---
title: 'Configuración de la autenticación multifactor de Azure para Windows Virtual Desktop: Azure'
description: Cómo configurar la autenticación multifactor de Azure para mejorar la seguridad de Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 12/10/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: 719b4579d97992cdfc3563ccb72b31809569182c
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107898268"
---
# <a name="enable-azure-multifactor-authentication-for-windows-virtual-desktop"></a>Habilitación de la autenticación multifactor de Azure para Windows Virtual Desktop

>[!IMPORTANT]
> Si va a visitar esta página desde la documentación de Windows Virtual Desktop (clásico), asegúrese de [volver a la documentación de Windows Virtual Desktop (clásico)](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md) una vez que haya terminado.

El cliente de Windows para Windows Virtual Desktop es una opción excelente para integrar Windows Virtual Desktop con el equipo local. Sin embargo, al configurar la cuenta de Windows Virtual Desktop en el cliente de Windows, hay ciertas medidas que debe seguir para mantener a todos los usuarios protegidos.

Al iniciar sesión por primera vez, el cliente solicita el nombre de usuario, la contraseña y la autenticación multifactor de Azure. Posteriormente, la próxima vez que inicie sesión, el cliente recordará el token de la aplicación empresarial de Azure Active Directory (AD). Al seleccionar **Recordarme** cuando se le soliciten las credenciales para el host de sesión, los usuarios pueden iniciar sesión después de reiniciar el cliente sin necesidad de volver a escribir sus credenciales.

Aunque recordar credenciales resulta práctico, también puede hacer que las implementaciones en escenarios empresariales o dispositivos personales sean menos seguras. Para proteger a los usuarios, debe asegurarse de que el cliente sigue solicitando las credenciales de autenticación multifactor de Azure con más frecuencia. En este artículo se muestra cómo configurar la directiva de acceso condicional de Windows Virtual Desktop para habilitar esta opción de configuración.

## <a name="prerequisites"></a>Prerrequisitos

Requisitos para poder comenzar:

- Asignar a los usuarios una licencia que incluya Azure Active Directory Premium P1 o P2.
- Un grupo de Azure Active Directory con los usuarios asignados como miembros del grupo.
- Habilite la autenticación multifactor de Azure para todos los usuarios. Para obtener más información sobre cómo realizar ese proceso, consulte [Exigencia de verificación en dos pasos para un usuario](../active-directory/authentication/howto-mfa-userstates.md#view-the-status-for-a-user).

> [!NOTE]
> La configuración siguiente también se aplica al [cliente web de Windows Virtual Desktop](https://rdweb.wvd.microsoft.com/arm/webclient/index.html).

## <a name="create-a-conditional-access-policy"></a>Creación de una directiva de acceso condicional

Aquí se muestra cómo crear una directiva de acceso condicional que exija autenticación multifactor al conectarse a Windows Virtual Desktop:

1. Inicie sesión en **Azure Portal** como administrador global, administrador de seguridad o administrador de acceso condicional.
2. Vaya a **Azure Active Directory** > **Seguridad** > **Acceso condicional**.
3. Seleccione **Nueva directiva**.
4. Asigne un nombre a la directiva. Se recomienda que las organizaciones creen un estándar significativo para los nombres de sus directivas.
5. En **Asignaciones**, seleccione **Usuarios y grupos**.
6. En **Incluir**, haga clic en **Seleccionar usuarios y grupos** > **Usuarios y grupos**  > Seleccionar el grupo creado en la fase de [requisitos previos](#prerequisites).
7. Seleccione **Listo**.
8. En **Aplicaciones en la nube o acciones** > **Incluir**, escoger **Seleccionar aplicaciones**.
9. Seleccione una de las siguientes aplicaciones en función de la versión de Windows Virtual Desktop que use.
   
   - Si usa Windows Virtual Desktop (clásico), elija estas aplicaciones:
       
       - **Windows Virtual Desktop** (id. de aplicación 5a0aa725-4958-4b0c-80a9-34562e23f3b7)
       - **Windows Virtual Desktop Client** (id. de aplicación fa4345a4-a730-4230-84a8-7d9651b86739), lo que le permitirá establecer directivas en el cliente web
       
        Después, vaya al paso 11.

   - Si usa Windows Virtual Desktop, en su lugar, elija esta otra:
       
       -  **Windows Virtual Desktop** (id. de aplicación 9cdead84-a844-4324-93f2-b2e6bb768d07)
       
        Después, vaya al paso 10.

   >[!IMPORTANT]
   > No seleccione la aplicación llamada Proveedor de Azure Resource Manager de Windows Virtual Desktop (50e95039-b200-4007-bc97-8d5790743a63). Esta aplicación solo se usa para recuperar la fuente de usuario y no debe tener autenticación multifactor.
   > 
   > Si usa Windows Virtual Desktop (clásico), si la directiva de acceso condicional bloquea todo el acceso y solo excluye los identificadores de aplicaciones de Windows Virtual Desktop, puede solucionarlo agregando el identificador de la aplicación 9cdead84-a844-4324-93f2-b2e6bb768d07 a la directiva. Si no se agrega este identificador de aplicación, se bloqueará la detección de fuentes de los recursos de Windows Virtual Desktop (clásico).

10. Vaya a **Condiciones**  >  **Aplicaciones cliente**. En **Configurar**, seleccione **Sí** y luego seleccione dónde aplicar la directiva:
    
    - Seleccione **Explorador** si quiere que la directiva se aplique al cliente web.
    - Seleccione **Aplicaciones móviles y aplicaciones de escritorio** si quiere aplicar la directiva a otros clientes.
    - Active ambas casillas si quiere aplicar la directiva a todos los clientes.
   
    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la página Aplicaciones cliente. El usuario ha seleccionado la casilla Aplicaciones móviles y aplicaciones de escritorio.](media/select-apply.png)

11. Una vez que haya seleccionado la aplicación, elija **Seleccionar** y, después, seleccione **Listo**.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la página Aplicaciones en la nube o acciones. Las aplicaciones de Windows Virtual Desktop y Windows Virtual Desktop Client están resaltadas en rojo.](media/cloud-apps-enterprise.png)

    >[!NOTE]
    >Para buscar el identificador de la aplicación que quiere seleccionar, diríjase a **Aplicaciones empresariales** y seleccione **Aplicaciones de Microsoft** en el menú desplegable Tipo de aplicación.

12. En **Controles de acceso** > **Conceder**, seleccionar **Conceder acceso**, **Requerir autenticación multifactor** y luego **Seleccionar**.
13. En **Controles de acceso** > **Sesión**, seleccione **Frecuencia de inicio de sesión**, establezca el valor en el tiempo que desee entre solicitudes y, luego, elija **Seleccionar**. Por ejemplo, si establece el valor en **1** y la unidad en **horas**, se requerirá la autenticación multifactor si una conexión se inicia una hora después de la última.
14. Confirme la configuración y establezca **Habilitar directiva** en **Activado**.
15. Seleccionar **Crear** para habilitar la directiva.

>[!NOTE]
>Cuando use el cliente web para iniciar sesión en Windows Virtual Desktop a través del explorador, el registro mostrará el identificador de la aplicación cliente como a85cf173-4192-42f8-81fa-777a763e6e2c (cliente de Windows Virtual Desktop). Esto se debe a que la aplicación cliente está vinculada internamente al identificador de la aplicación del servidor donde se estableció la directiva de acceso condicional. 

## <a name="next-steps"></a>Pasos siguientes

- [Más información sobre las directivas de acceso condicional](../active-directory/conditional-access/concept-conditional-access-policies.md)

- [Más información sobre la frecuencia de inicio de sesión de los usuarios](../active-directory/conditional-access/howto-conditional-access-session-lifetime.md#user-sign-in-frequency)
