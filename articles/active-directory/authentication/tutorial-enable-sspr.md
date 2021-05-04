---
title: Habilitación del autoservicio de restablecimiento de contraseña de Azure Active Directory
description: En este tutorial aprenderá a habilitar el autoservicio de restablecimiento de contraseña de Azure Active Directory para un grupo de usuarios y a probar el proceso de restablecimiento de contraseña.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: tutorial
ms.date: 04/21/2021
ms.author: justinha
author: justinha
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8c18dd231a708030e3a454ab8708e3f0f11dbecf
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107861830"
---
# <a name="tutorial-enable-users-to-unlock-their-account-or-reset-passwords-using-azure-active-directory-self-service-password-reset"></a>Tutorial: Habilitación del autoservicio de restablecimiento de contraseña de Azure Active Directory para que los usuarios puedan desbloquear su cuenta o restablecer contraseñas

El autoservicio de restablecimiento de contraseña (SSPR) de Azure Active Directory (Azure AD) ofrece a los usuarios la posibilidad de cambiar o restablecer su contraseña sin necesidad de que intervenga el administrador o el departamento de soporte técnico. Si Azure AD bloquea la cuenta de un usuario o este se ha olvidado su contraseña, puede seguir las indicaciones para desbloquearla y volver al trabajo. Esta capacidad reduce las llamadas al departamento de soporte técnico y la pérdida de productividad cuando un usuario no puede iniciar sesión en su dispositivo o en una aplicación. Se recomienda este vídeo sobre [cómo habilitar y configurar SSPR en Azure AD](https://www.youtube.com/watch?v=rA8TvhNcCvQ). También disponemos de un vídeo para los administradores de TI en [Resolución de los seis mensajes de error de usuario final más comunes con SSPR](https://www.youtube.com/watch?v=9RPrNVLzT8I).

> [!IMPORTANT]
> Este tutorial muestra al administrador cómo habilitar SSPR. Si es un usuario final registrado para el autoservicio de restablecimiento de contraseña y necesita volver a su cuenta, visite la página de [restablecimiento de contraseña de Microsoft Online](https://passwordreset.microsoftonline.com/).
>
> Si el equipo de TI no ha habilitado la capacidad para restablecer su propia contraseña, póngase en contacto con el departamento de soporte técnico para obtener ayuda adicional.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Habilitar el autoservicio de restablecimiento de contraseña para un grupo de usuarios de Azure AD
> * Configuración de métodos de autenticación y opciones de registro
> * Probar el proceso SSPR como usuario

## <a name="prerequisites"></a>Requisitos previos

Para finalizar este tutorial, necesitará los siguientes recursos y privilegios:

* Un inquilino de Azure AD activo con al menos una licencia de Azure AD gratis o de prueba habilitada. En el nivel Gratis, SSPR solo funciona para los usuarios de la nube en Azure AD. En el nivel Gratis es posible cambiar la contraseña, pero no restablecerla. 
    * En los tutoriales posteriores de esta serie, se necesita una licencia de Azure AD Premium P1 o de prueba para la escritura diferida de contraseñas local.
    * Si es necesario, [cree una cuenta de Azure de forma gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Una cuenta con privilegios de *Administrador global*.
* Un usuario que no sea administrador con una contraseña que conozca, como *usuarioDePrueba*. En este tutorial utilizará esta cuenta para probar la experiencia de autoservicio de restablecimiento de contraseña por parte del usuario final.
    * Si necesita crear un usuario, consulte [Inicio rápido: Incorporación de nuevos usuarios a Azure Active Directory](../fundamentals/add-users-azure-active-directory.md).
* Un grupo del que sea miembro el usuario que no es administrador, como *Grupo-prueba-SSPR*. En este tutorial, habilitará el autoservicio de restablecimiento de contraseña para este grupo.
    * Si necesita crear un grupo, consulte [Creación de un grupo básico e incorporación de miembros con Azure Active Directory](../fundamentals/active-directory-groups-create-azure-portal.md).

## <a name="enable-self-service-password-reset"></a>Habilitar el autoservicio de restablecimiento de contraseña

Azure AD permite habilitar el autoservicio de restablecimiento de contraseña para *Ninguno*, *Seleccionados* o *Todos* los usuarios. Esta capacidad para pormenorizar permite elegir un subconjunto de usuarios para probar el proceso de registro y el flujo de trabajo de SSPR. Cuando esté familiarizado con el proceso y haya llegado el momento de comunicar los requisitos con un conjunto más amplio de usuarios, podrá seleccionar un grupo de usuarios para habilitarlos en el autoservicio de restablecimiento de contraseña. O bien, podrá habilitar SSPR para todos los usuarios del inquilino de Azure AD.

> [!NOTE]
> Actualmente, solo se puede habilitar un grupo de Azure AD para SSPR mediante Azure Portal. Como parte de una implementación más amplia del autoservicio de restablecimiento de contraseña, Azure AD admite los grupos anidados. Asegúrese de que los usuarios de los grupos que elija tienen asignadas las licencias correspondientes. Actualmente no hay ningún proceso de validación de estos requisitos de licencia.

En este tutorial, configurará el autoservicio de restablecimiento de contraseña para un conjunto de usuarios de un grupo de prueba. Use *Grupo-prueba-SSPR* y proporcione su propio grupo de Azure AD según sea necesario:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) mediante una cuenta con permisos de *administrador global*.
1. Busque y seleccione **Azure Active Directory** y, a continuación, elija **Restablecer la contraseña** en el menú del lado izquierdo.
1. En la página **Propiedades**, bajo la opción *Se habilitó el restablecimiento de contraseña del autoservicio*, elija **Seleccionar grupo**.
1. Busque y seleccione el grupo de Azure AD, como *Grupo-prueba-SSPR* y, a continuación, elija *Seleccionar*.

    [![Selección de un grupo en Azure Portal para habilitar el autoservicio de restablecimiento de contraseña](media/tutorial-enable-sspr/enable-sspr-for-group-cropped.png)](media/tutorial-enable-sspr/enable-sspr-for-group.png#lightbox)

1. Para habilitar SSPR para los usuarios seleccionados, seleccione **Guardar**.

## <a name="select-authentication-methods-and-registration-options"></a>Selección de métodos de autenticación y opciones de registro

Cuando los usuarios necesitan desbloquear su cuenta o restablecer su contraseña, se les pide otro método de confirmación. Este factor de autenticación adicional garantiza que Azure AD finalizan solo los eventos de autoservicio de restablecimiento de contraseña aprobados. Puede elegir los métodos de autenticación que se permitirán, en función de la información de registro que proporciona el usuario.

1. En el menú de la izquierda de la página **Métodos de autenticación**, establezca el valor de **Número de métodos requeridos para el restablecimiento** en *1*.

    Si desea mejorar la seguridad, puede aumentar el número de métodos de autenticación necesarios para el autoservicio de restablecimiento de contraseña.

1. En **Métodos disponibles para los usuarios**, elija qué métodos desea permitir en la organización. En este tutorial, active las casillas para habilitar los siguientes métodos:

    * *Notificación en aplicación móvil*
    * *Código de aplicación móvil*
    * *Correo electrónico*
    * *Teléfono móvil*

    Es posible habilitar métodos de autenticación adicionales, como el *teléfono del trabajo* o *preguntas de seguridad*, de acuerdo con las necesidades de su empresa.

1. Para aplicar los métodos de autenticación, seleccione **Guardar**.

Para que los usuarios puedan desbloquear su cuenta o restablecer una contraseña, deben registrar su información de contacto. Azure AD utiliza esta información de contacto para los distintos métodos de autenticación configurados en los pasos anteriores.

Un administrador puede proporcionar manualmente esta información de contacto, o bien los usuarios pueden ir a un portal de registro para proporcionarla ellos mismos. En este tutorial, configure Azure AD para solicitar el registro a los usuarios la próxima vez que inicien sesión.

1. En el menú de la izquierda de la página **Registro**, seleccione *Sí* en la opción **¿Desea exigir a los usuarios que se registren al iniciar sesión?**
1. Establezca el valor de **Número de días que pasan hasta que se pide a los usuarios que vuelvan a confirmar su información de autenticación** en *180*.

    Es importante mantener actualizada la información de contacto. Si no está actualizada cuando se inicia un evento de SSPR, es posible que el usuario no pueda desbloquear su cuenta ni restablecer su contraseña.

1. Para aplicar la configuración de registro, seleccione **Guardar**.

## <a name="set-up-notifications-and-customizations"></a>Configuración de notificaciones y personalizaciones

Para mantener informados a los usuarios sobre la actividad de la cuenta, puede configurar Azure AD para que envíe notificaciones por correo electrónico cuando se produzca un evento de SSPR. Estas notificaciones pueden abarcar tanto las cuentas de usuario normales como las cuentas de administrador. En el caso de las cuentas de administrador, esta notificación proporciona otra capa de reconocimiento cuando se restablece la contraseña de una cuenta de administrador con privilegios mediante SSPR. Azure AD enviará una notificación a todos los administradores globales cuando alguien use SSPR en una cuenta de administrador.

1. En el menú de la izquierda de la página **Notificaciones**, configure las siguientes opciones:

   * Establezca la opción **Notificar a los usuarios el restablecimiento de contraseña** en *Sí*.
   * Establezca **Notificar a todos los administradores cuando otros administradores restablezcan su contraseña** en *Sí*.

1. Para aplicar las preferencias de notificación, seleccione **Guardar**.

Si los usuarios necesitan más ayuda con el proceso de SSPR, puede personalizar el vínculo "Póngase en contacto con el administrador". El usuario puede seleccionar esta vínculo en el proceso de registro de SSPR y cuando desbloquea su cuenta o restablece su contraseña. Para asegurarse de que los usuarios obtengan el soporte técnico necesario, se recomienda encarecidamente que indique una dirección URL o un correo electrónico de soporte técnico personalizados.

1. En el menú de la izquierda de la página **Personalización**, establezca **Personalizar el vínculo del departamento de soporte técnico** en *Sí*.
1. En el campo **Dirección URL o correo electrónico del departamento de soporte técnico personalizados**, indique una dirección de correo electrónico o una dirección URL de la página web donde los usuarios puedan obtener más ayuda de su organización, como *https:\//support.contoso.com/* .
1. Para aplicar el vínculo personalizado, seleccione **Guardar**.

## <a name="test-self-service-password-reset"></a>Autoservicio de restablecimiento de contraseña de prueba

Con el autoservicio de restablecimiento de contraseña habilitado y configurado, pruebe este proceso con un usuario que forme parte del grupo que seleccionó en la sección anterior, como *Test-SSPR-Group*. En el ejemplo siguiente, se usa la cuenta *usuarioDePrueba*. Indique su propia cuenta de usuario, que forma parte del grupo que ha habilitado para SSPR en la primera sección de este tutorial.

> [!NOTE]
> Para probar el autoservicio de restablecimiento de contraseña, use una cuenta que no sea de administrador. De forma predeterminada, Azure AD habilita el autoservicio de restablecimiento de contraseña para administradores. Deben utilizar dos métodos de autenticación para restablecer la contraseña. Para más información, consulte [Diferencias entre directivas de restablecimiento de administrador](concept-sspr-policy.md#administrator-reset-policy-differences).

1. Para ver el proceso de registro manual, abra una nueva ventana del explorador en modo InPrivate o incógnito y vaya a *https:\//aka.ms/ssprsetup*. Azure AD dirigirá a los usuarios a este portal de registro la próxima vez que inicien sesión.
1. Inicie sesión con un usuario de prueba que no sea administrador, como *usuarioDePrueba* y registre la información de contacto de los métodos de autenticación.
1. Cuando haya terminado, seleccione el botón marcado como **Parece correcto** y cierre la ventana del explorador.
1. Abra una nueva ventana del explorador en modo de incógnito o InPrivate y vaya a *https:\//aka.ms/sspr*.
1. Escriba la información de cuenta del usuario de prueba sin privilegios de administrador, como *usuarioDePrueba*, y los caracteres del CAPTCHA; después, seleccione **Siguiente**.

    ![Introducción de la información de la cuenta de usuario para restablecer la contraseña](media/tutorial-enable-sspr/password-reset-page.png)

1. Siga los pasos de comprobación para restablecer la contraseña. Cuando termine, recibirá una notificación por correo electrónico que le notificará que se ha restablecido la contraseña.

## <a name="clean-up-resources"></a>Limpieza de recursos

En un tutorial posterior de esta serie, configurará la escritura diferida de contraseñas. Esta característica escribe los cambios de contraseña del autoservicio de restablecimiento de contraseña de Azure AD de nuevo en un entorno de AD local. Si quiere continuar con esta serie de tutoriales para configurar la escritura diferida de contraseñas, no deshabilite SSPR por ahora.

Si ya no desea usar la funcionalidad de SSPR que ha configurado como parte de este tutorial, establezca el estado de SSPR en **Ninguno** mediante los pasos siguientes:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque y seleccione **Azure Active Directory** y, a continuación, elija **Restablecer la contraseña** en el menú del lado izquierdo.
1. En la página **Propiedades**, bajo la opción *Se habilitó el restablecimiento de contraseña del autoservicio*, elija **Ninguno**.
1. Para aplicar el cambio de SSPR, seleccione **Guardar**.

## <a name="faqs"></a>Preguntas más frecuentes

En esta sección se explican las preguntas comunes de los administradores y los usuarios finales que prueba el autoservicio de restablecimiento de contraseña (SSPR):

- ¿Por qué los usuarios federados esperan hasta 2 minutos después de ver el mensaje **Se ha restablecido su contraseña** y antes de usar contraseñas que se sincronizan desde el entorno local?

  En el caso de los usuarios federados cuyas contraseñas están sincronizadas, el origen de autoridad de las contraseñas es el en el entorno local. Como consecuencia, SSPR solo actualiza las contraseñas locales. La sincronización de hash de contraseñas en Azure AD está programada para producirse cada 2 minutos.

- Cuando un usuario recién creado cuyos datos de SSPR se rellenan previamente (por ejemplo, el teléfono y el correo electrónico) visita la página de registro de SSPR, el mensaje **No perder el acceso a su cuenta** aparece como el título de la página. ¿Por qué otros usuarios cuyos datos de SSPR se rellenan previamente no ven el mensaje?

  Si un usuario ve el mensaje **No perder el acceso a su cuenta**, pertenece a los grupos de registro combinado/SSPR configurados para el inquilino. Los usuarios que no ven el mensaje **No perder el acceso a su cuenta** no pertenecían a los grupos de registro combinado/SSPR.

- Cuando algunos usuarios pasan por el proceso de SSPR y restablecen la contraseña, ¿por qué no ven el indicador del nivel de seguridad de la contraseña?

  Los usuarios que no ven si el nivel de contraseña es débil o fuerte tienen habilitada la escritura diferida de contraseñas. Dado que SSPR no puede establecer la directiva de contraseñas utilizada en el entorno local del cliente, no podrá confirmar si la contraseña es fuerte o débil. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial habilitó el autoservicio de restablecimiento de contraseña en Azure AD para un grupo seleccionado de usuarios. Ha aprendido a:

> [!div class="checklist"]
> * Habilitar el autoservicio de restablecimiento de contraseña para un grupo de usuarios de Azure AD
> * Configuración de métodos de autenticación y opciones de registro
> * Probar el proceso SSPR como usuario

> [!div class="nextstepaction"]
> [Habilitar Azure AD Multi-Factor Authentication](./tutorial-enable-azure-mfa.md)
