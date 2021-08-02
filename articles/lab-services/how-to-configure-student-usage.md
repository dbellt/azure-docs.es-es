---
title: Configuración de los valores de uso de los laboratorios de Azure Lab Services
description: Obtenga información sobre cómo configurar el número de alumnos del laboratorio, registrarlos en el laboratorio, controlar el número de horas que pueden usar la máquina virtual y mucho más.
ms.topic: article
ms.date: 12/01/2020
ms.openlocfilehash: f7c0fb7b0228ee17705af2dfd70279f6fc7bda3c
ms.sourcegitcommit: a9f131fb59ac8dc2f7b5774de7aae9279d960d74
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110190719"
---
# <a name="add-and-manage-lab-users"></a>Incorporación y administración de usuarios de laboratorio

En este artículo se describe cómo agregar alumnos a un laboratorio, registrarlos en él, controlar el número de horas extra que pueden usar la máquina virtual y mucho más. 

Al agregar usuarios, de forma predeterminada, la opción **Restringir el acceso** está activada y, a menos que figuren en la lista de usuarios, los alumnos no podrán registrarse en el laboratorio, aunque tengan un vínculo de registro. Solo los usuarios de la lista pueden registrarse en el laboratorio por medio del vínculo de registro enviado. Puede desactivar **Restringir el acceso**, que permite que los alumnos se registren en el laboratorio, siempre y cuando tengan el vínculo de registro. 

En este artículo se muestra cómo agregar usuarios a un laboratorio.

## <a name="add-users-from-an-azure-ad-group"></a>Incorporación de usuarios desde un grupo de Azure AD

### <a name="overview"></a>Información general

Ahora puede sincronizar una lista de usuarios de laboratorio con un grupo de Azure Active Directory (Azure AD) existente para no tener que agregar ni eliminar usuarios manualmente. 

Se puede crear un grupo de Azure AD en la instancia de Azure Active Directory de su organización para administrar el acceso a los recursos de la organización y a las aplicaciones basadas en la nube. Para más información, consulte el artículo sobre [grupos de Azure AD](../active-directory/fundamentals/active-directory-manage-groups.md). Si su organización utiliza Microsoft Office 365 o los servicios de Azure, ya tendrá administradores que administren su instancia de Azure Active Directory. 

### <a name="sync-users-with-azure-ad-group"></a>Sincronización de usuarios con el grupo de Azure AD

> [!IMPORTANT]
> Asegúrese de que la lista de usuarios esté vacía. Si ya existen usuarios dentro de un laboratorio que haya agregado manualmente o al importar un archivo CSV, la opción para sincronizar el laboratorio con un grupo existente no aparecerá. 

1. Inicie sesión en el [sitio web de Azure Lab Services](https://labs.azure.com/).
1. Seleccione el laboratorio con el que quiere trabajar.
1. En el panel izquierdo, seleccione **Usuarios**. 
1. Haga clic en **Sincronizar desde un grupo**. 

    :::image type="content" source="./media/how-to-configure-student-usage/add-users-sync-group.png" alt-text="Incorporación de usuarios mediante la sincronización desde un grupo de Azure AD":::
    
1. Se le pedirá que elija un grupo de Azure AD existente con el que sincronizar el laboratorio. 
    
    Si no ve ningún grupo de Azure AD en la lista, puede deberse a los siguientes motivos:

    -   Si es un usuario invitado de una instancia de Azure Active Directory (normalmente, si está fuera de la organización propietaria de Azure AD) y no puede buscar grupos dentro de Azure AD. En este caso, no podrá agregar un grupo de Azure AD al laboratorio. 
    -   Los grupos de Azure AD creados mediante Teams no aparecen en esta lista. Puede agregar la aplicación Azure Lab Services a Teams para crear y administrar laboratorios directamente desde este. Vea más información sobre cómo [administrar la lista de usuarios de un laboratorio desde Teams](how-to-manage-user-lists-within-teams.md). 
1. Una vez que haya seleccionado el grupo de Azure AD con el que se va a sincronizar el laboratorio, haga clic en **Agregar**.
1. Una vez que se sincronice un laboratorio, este extraerá a todos los miembros del grupo de Azure AD en el laboratorio como usuarios, y podrá ver la lista de usuarios actualizada. Solo las personas de este grupo de Azure AD tendrán acceso a su laboratorio. La lista de usuarios se actualizará cada 24 horas para que coincida con la última pertenencia al grupo de Azure AD. También puede hacer clic en el botón Sincronizar de la pestaña Usuarios para sincronizar manualmente los cambios más recientes del grupo de Azure AD.
1. Para invitar a los usuarios a su laboratorio, haga clic en el botón **Invitar a todos**, con lo que se enviará un mensaje de correo electrónico a todos los usuarios con el vínculo de registro al laboratorio. 

### <a name="automatic-management-of-virtual-machines-based-on-changes-to-the-azure-ad-group"></a>Administración automática de máquinas virtuales en función de los cambios en el grupo de Azure AD 

Una vez que el laboratorio se haya sincronizado con un grupo de Azure AD, el número de máquinas virtuales del laboratorio coincidirá automáticamente con el número de usuarios del grupo. Ya no podrá actualizar manualmente la capacidad de laboratorio. Al agregar un usuario al grupo de Azure AD, un laboratorio agregará automáticamente una máquina virtual para ese usuario. Al eliminar un usuario del grupo de Azure AD, un laboratorio eliminará automáticamente la máquina virtual del usuario del laboratorio. 

## <a name="add-users-manually-from-emails-or-csv-file"></a>Incorporación de usuarios manualmente desde mensajes de correo electrónico o archivos CSV

En esta sección, agregará alumnos manualmente, (mediante direcciones de correo electrónico o bien cargando un archivo CSV). 

### <a name="add-users-by-email-address"></a>Adición de usuarios por dirección de correo electrónico

1. En el panel izquierdo, seleccione **Usuarios**. 
1. Haga clic en **Add users manually** (Agregar usuarios manualmente). 

    :::image type="content" source="./media/how-to-configure-student-usage/add-users-manually.png" alt-text="Agregar usuarios manualmente":::
1. Seleccione **Agregar por dirección de correo electrónico** (opción predeterminada), escriba las direcciones de correo electrónico de los alumnos en líneas independientes o en una sola línea separadas por punto y coma. 

    :::image type="content" source="./media/how-to-configure-student-usage/add-users-email-addresses.png" alt-text="Adición de direcciones de correo electrónico de los usuarios":::
1. Seleccione **Guardar**. 

    En la lista se muestran las direcciones de correo electrónico y los estados de los usuarios actuales, tanto si están registrados con el laboratorio como si no. 

    :::image type="content" source="./media/how-to-configure-student-usage/list-of-added-users.png" alt-text="Lista de usuarios":::

    > [!NOTE]
    > Una vez que los alumnos estén registrados en el laboratorio, la lista muestra sus nombres. El nombre que se muestra en la lista se forma usando el nombre y los apellidos de los alumnos en Azure Active Directory. 

### <a name="add-users-by-uploading-a-csv-file"></a>Adición de usuarios mediante la carga de un archivo CSV

También puede agregar usuarios cargando un archivo CSV que contenga sus direcciones de correo electrónico. 

Se usa un archivo de texto CSV para almacenar datos tabulares (números y texto) separados por comas (CSV). En lugar de almacenar información en campos de columnas (por ejemplo, en hojas de cálculo), un archivo CSV almacena información separada por comas. Cada línea de un archivo CSV tendrá el mismo número de "campos" separados por comas. Puede utilizar Excel para crear y editar archivos CSV fácilmente.

1. En Microsoft Excel, cree un archivo CSV e incluya en él las direcciones de correo electrónico de los estudiantes en una columna.

    :::image type="content" source="./media/how-to-configure-student-usage/csv-file-with-users.png" alt-text="Lista de usuarios en un archivo CSV":::
1. En la parte superior del panel **Usuarios**, seleccione **Agregar usuarios** y, después, seleccione **Cargar CSV**.
1. Seleccione el archivo CSV que contiene las direcciones de correo electrónico de los alumnos y, después, seleccione **Abrir**.

    La ventana **Agregar usuarios** muestra la lista de direcciones de correo electrónico del archivo CSV. 
1. Seleccione **Guardar**. 
1. En el panel **Usuarios**, vea la lista de los alumnos agregados. 

    :::image type="content" source="./media/how-to-configure-student-usage/list-of-added-users.png" alt-text="Lista de usuarios agregados en el panel Usuarios":::

## <a name="send-invitations-to-users"></a>Envío de invitaciones a los usuarios

Use uno de los siguientes métodos para enviar un vínculo de registro a los usuarios nuevos. 

Si la opción **Restringir el acceso** está habilitada en el laboratorio, solo los usuarios de la lista de usuarios pueden usar el vínculo de registro para registrarse en el laboratorio. Esta opción está habilitada de manera predeterminada. 

### <a name="invite-all-users"></a>Invitar a todos los usuarios

Este método muestra cómo enviar un correo electrónico con un vínculo de registro y un mensaje opcional a todos los alumnos de la lista.

1. En el panel **Usuarios**, seleccione **Invitar a todos**. 

    ![El botón "Invitar a todos"](./media/tutorial-setup-classroom-lab/invite-all-button.png)

1. En la ventana **Enviar invitación por correo electrónico**, escriba un mensaje opcional y seleccione **Enviar**. 

    El correo electrónico incluye automáticamente el vínculo de registro. Para obtener y guardar el vínculo de registro por separado, seleccione el botón de puntos suspensivos ( **...** ) en la parte superior del panel **Usuarios** y, después, seleccione **Vínculo de registro**. 

    ![La ventana "Enviar invitación por correo electrónico"](./media/tutorial-setup-classroom-lab/send-email.png)

    En la columna **Invitación** de la lista **Usuarios** se refleja el estado de la invitación de cada usuario agregado. El estado debería cambiar a **Enviando** y luego a **Enviado el \<date>** . 

### <a name="invite-selected-users"></a>Invitar a usuarios seleccionados

Este método muestra cómo invitar solo a determinados estudiantes y obtener un vínculo de registro que se puede compartir con otras personas.

1. En el panel **Usuarios**, seleccione uno o varios alumnos de la lista. 

1. En la fila del alumno que haya seleccionado, seleccione el icono **sobre** o, en la barra de herramientas, seleccione **Invitar**. 

    ![Invitar a usuarios seleccionados](./media/how-to-configure-student-usage/invite-selected-users.png)

1. En la página **Send invitation by email** (Enviar invitación por correo electrónico), escriba un **mensaje** opcional y seleccione **Send** (Enviar). 

    ![Enviar un correo electrónico a los usuarios seleccionados](./media/how-to-configure-student-usage/send-invitation-to-selected-users.png)

    El panel **Usuarios**  muestra el estado de esta operación en la columna **Invitación** de la tabla. El correo electrónico de invitación incluye el vínculo de registro que los alumnos pueden usar para registrarse en el laboratorio.

## <a name="get-the-registration-link"></a>Obtención del vínculo de registro

En esta sección, puede obtener el vínculo de registro del portal y enviarlo a través de la aplicación de correo electrónico que use habitualmente. 

1. En el panel **Usuarios**, seleccione **Vínculo de registro**.

    ![Vínculo de registro del estudiante](./media/how-to-configure-student-usage/registration-link-button.png)

1. En la ventana **Registro de usuarios**, seleccione **Copiar** y, después, seleccione **Listo**. 

    ![La ventana "Registro de usuarios"](./media/how-to-configure-student-usage/registration-link.png)

    El vínculo se copia en el Portapapeles. 
    
1. En la aplicación de correo electrónico, pegue el vínculo de registro y, después, envíe el correo electrónico a un alumno para que pueda registrarse en la clase. 

## <a name="view-registered-users"></a>Visualización de usuarios registrados

1. Vaya al [sitio web de Azure Lab Services](https://labs.azure.com). 
1. Seleccione **Iniciar sesión** y escriba las credenciales. Azure Lab Services es compatible con cuentas profesionales y cuentas Microsoft.
1. En la página **Mis laboratorios**, seleccione el laboratorio de cuyo uso quiera realizar un seguimiento. 
1. En el panel izquierdo, seleccione **Usuarios** o seleccione el mosaico **Usuarios**. 

    En el panel **Usuarios** se muestra una lista de los alumnos que se han registrado en el laboratorio.  

    ![Lista de usuarios registrados](./media/tutorial-track-usage/registered-users.png)

    > [!NOTE] 
    > Si [vuelve a publicar un laboratorio](how-to-create-manage-template.md#publish-the-template-vm) o [restablece las máquinas virtuales de los alumnos](how-to-set-virtual-machine-passwords.md#reset-vms), los alumnos permanecerán registrados en las máquinas virtuales de los laboratorios.  Sin embargo, el contenido de las máquinas virtuales se eliminará y las máquinas virtuales se volverán a crear con la imagen de la máquina virtual de la plantilla.

## <a name="set-quotas-for-users"></a>Establecimiento de cuotas para los usuarios

Para establecer una cuota de horas para cada alumno, haga lo siguiente: 

1. En el panel **Usuarios**, seleccione **Cuota por usuario: \<number>hora(s)** en la barra de herramientas. 
1. En la ventana **Cuota por usuario**, especifique el número de horas que quiere dar a cada alumno fuera del tiempo de clase programado y, luego, seleccione **Guardar**.

    ![La ventana "Cuota por usuario"](./media/how-to-configure-student-usage/quota-per-user.png)    

    Los valores modificados se muestran ahora en el botón **Cuota por usuario: \<number of hours>** de la barra de herramientas y en la lista de usuarios, como se muestra aquí:

    ![Horas de cuota por usuario](./media/how-to-configure-student-usage/quot-per-user-after.png)

    > [!IMPORTANT]
    > El [tiempo de ejecución programado de las máquinas virtuales](how-to-create-schedules.md) no se tiene en cuenta en la cuota asignada a un alumno. La cuota corresponde al tiempo fuera del horario programado que un alumno pasa en las máquinas virtuales. 

## <a name="set-additional-quotas-for-specific-users"></a>Establecimiento de más cuotas para usuarios concretos

Se pueden especificar más cuotas para determinados alumnos aparte de las cuotas comunes que ya establecimos para todos los usuarios en la sección anterior. Por ejemplo, si usted (el instructor) establece la cuota de todos los alumnos en 10 horas y establece una cuota adicional de 5 horas para un alumno específico, este alumno tendrá 15 (10 + 5) horas de cuota. Si cambia la cuota común más adelante a, por ejemplo, 15, el alumno dispondrá de 20 (15 + 5) horas de cuota. Recuerde que esta cuota general está fuera del tiempo programado. El tiempo que un alumno invierte en una máquina virtual de laboratorio durante el tiempo programado no se tiene en cuenta en esta cuota. 

Para establecer más cuotas, haga lo siguiente:

1. En el panel **Usuarios**, seleccione un alumno de la lista y, después, seleccione **Ajustar cuota** en la barra de herramientas. 

    ![El botón "Ajustar cuota"](./media/how-to-configure-student-usage/adjust-quota-button.png)

1. En **Ajustar cuota para \<selected user or users email address>** , escriba el número de horas de laboratorio extra que quiera conceder al alumno o alumnos seleccionados y, después, seleccione **Aplicar**. 

    ![La ventana "Ajustar cuota..."](./media/how-to-configure-student-usage/additional-quota.png)

    La columna **Uso** refleja la cuota actualizada de los alumnos seleccionados. 

    ![Nuevo uso del usuario](./media/how-to-configure-student-usage/new-usage-hours.png)

## <a name="student-accounts"></a>Cuentas de alumno

Para agregar alumnos a un laboratorio de clase, use sus cuentas de correo electrónico. Los alumnos pueden tener los siguientes tipos de cuentas de correo electrónico:

- Una cuenta de correo electrónico de alumno proporcionada por la instancia de Azure Active Directory de la universidad.
- Una cuenta de correo electrónico con un dominio de Microsoft, como *outlook.com*, *hotmail.com*, *msn.com* o *live.com*.
- Una cuenta de correo electrónico que no sea de Microsoft, como las proporcionadas por Yahoo! o Google. No obstante, estos tipos de cuentas se deben vincular a una cuenta de Microsoft.
- Una cuenta de GitHub. Esta cuenta se debe vincular a una cuenta de Microsoft.

### <a name="use-a-non-microsoft-email-account"></a>Uso de una cuenta de correo electrónico que no es de Microsoft

Los alumnos pueden usar cuentas de correo electrónico que no sean de Microsoft para registrarse e iniciar sesión en un laboratorio de clase.  Pero el registro exige que deben crear primero una cuenta de Microsoft que esté vinculada a su dirección de correo electrónico que no es de Microsoft.

Es posible que muchos alumnos ya tengan una cuenta de Microsoft vinculada a su dirección de correo electrónico que no es de Microsoft. Por ejemplo, los alumnos ya tienen una cuenta de Microsoft si han usado su dirección de correo electrónico con otros productos o servicios de Microsoft, como Office, Skype, OneDrive o Windows.  

Cuando los alumnos usan el vínculo de registro para iniciar sesión en una clase, se les pide su dirección de correo electrónico y contraseña. Los alumnos que intenten iniciar sesión con una cuenta que no es de Microsoft que no está vinculada a una cuenta de Microsoft verán el siguiente mensaje de error: 

![Mensaje de error al iniciar sesión](./media/how-to-configure-student-usage/cant-find-account.png)

Con este vínculo, los alumnos pueden [registrarse para obtener una cuenta de Microsoft](http://signup.live.com).  

> [!IMPORTANT]
> Cuando los alumnos inician sesión en un laboratorio de clase, no se les ofrece la opción de crear una cuenta de Microsoft. Por este motivo, se recomienda incluir este vínculo de suscripción (http://signup.live.com ) en el correo electrónico de registro del laboratorio de clase que envíe a los alumnos que usen cuentas que no sean de Microsoft.

### <a name="use-a-github-account"></a>Uso de una cuenta de GitHub

Los alumnos también pueden usar una cuenta de GitHub existente para registrarse e iniciar sesión en un laboratorio de clase. Si ya tienen una cuenta de Microsoft vinculada a su cuenta de GitHub, los alumnos pueden iniciar sesión y proporcionar su contraseña como se muestra en la sección anterior. 

Si aún no han vinculado su cuenta de GitHub a una cuenta de Microsoft, pueden hacer lo siguiente:

1. Seleccione el vínculo **Opciones de inicio de sesión**, como el de aquí:

    ![El vínculo "Opciones de inicio de sesión"](./media/how-to-configure-student-usage/signin-options.png)

1. En la ventana **Opciones de inicio de sesión**, seleccione **Iniciar sesión con GitHub**.

    ![El vínculo "Iniciar sesión con GitHub"](./media/how-to-configure-student-usage/signin-github.png)

    Cuando se les pida, los alumnos crean una cuenta de Microsoft que esté vinculada a su cuenta de GitHub. La vinculación se producirá automáticamente cuando seleccionen **Siguiente**. Su sesión se iniciará inmediatamente y se conectarán al laboratorio de clase.

## <a name="export-a-list-of-users-to-a-csv-file"></a>Exportación de una lista de usuarios a un archivo CSV

1. Vaya al panel **Usuarios**.
1. En la barra de herramientas, seleccione los puntos suspensivos **(...)** y, luego, seleccione **Exportar CSV**). 

    ![El botón "Exportar CSV"](./media/how-to-export-users-virtual-machines-csv/users-export-csv.png)

## <a name="next-steps"></a>Pasos siguientes

Vea los artículos siguientes:

- Para los administradores: [Creación y administración de cuentas de laboratorio](how-to-manage-lab-accounts.md)
- Para los propietarios de laboratorios: [Creación y administración de laboratorios](how-to-manage-classroom-labs.md) y [Configuración y publicación de plantillas](how-to-create-manage-template.md)
- Para los usuarios de laboratorios: [Laboratorios de acceso](how-to-use-classroom-lab.md)