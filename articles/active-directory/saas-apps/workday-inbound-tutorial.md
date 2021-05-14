---
title: 'Tutorial: Configuración de Workday para el aprovisionamiento automático de usuarios con Azure Active Directory | Microsoft Docs'
description: Obtenga información sobre cómo configurar Azure Active Directory para aprovisionar y cancelar automáticamente el aprovisionamiento de cuentas de usuario de Workday.
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.topic: tutorial
ms.workload: identity
ms.date: 01/19/2021
ms.author: chmutali
ms.openlocfilehash: eb6f3c6a1463e2a3b71057145a6df770ea880b9d
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108181173"
---
# <a name="tutorial-configure-workday-for-automatic-user-provisioning"></a>Tutorial: Configuración de Workday para el aprovisionamiento automático de usuarios

El objetivo de este tutorial es mostrar los pasos que debe dar para aprovisionar perfiles de trabajo desde Workday en Active Directory (AD) local.

>[!NOTE]
>Use este tutorial si los usuarios que quiere aprovisionar de Workday necesitan una cuenta de AD local y una cuenta de Azure AD. 
>* Si los usuarios de Workday solo necesitan una cuenta de Azure AD (usuarios de solo nube), consulte el tutorial sobre la [configuración de Workday para el aprovisionamiento de usuarios de Azure AD](workday-inbound-cloud-only-tutorial.md). 
>* Para configurar la escritura diferida de atributos, como la dirección de correo electrónico, el nombre de usuario y el número de teléfono de Azure AD a Workday, consulte el tutorial sobre la [configuración de la escritura diferida de Workday](workday-writeback-tutorial.md).

## <a name="overview"></a>Información general

El [servicio de aprovisionamiento de usuarios de Azure Active Directory](../app-provisioning/user-provisioning.md) se integra con la [API de recursos humanos de Workday](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) para poder aprovisionar cuentas de usuario. El flujo de trabajo de aprovisionamiento de usuarios de Workday compatible con el servicio de aprovisionamiento de usuarios de Azure AD permite la automatización de los siguientes escenarios de recursos humanos y de administración del ciclo de vida de identidades:

* **Contratación de nuevos empleados**: cuando se agrega un nuevo empleado a Workday, se crea automáticamente una cuenta de usuario en Active Directory, Azure Active Directory y, opcionalmente, en Microsoft 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md), con escritura diferida de la información de contacto administrada por TI en Workday.

* **Actualizaciones de atributos y perfiles de los empleados**: si se actualiza un registro de empleado en Workday (por ejemplo, el nombre, el cargo o el administrador), su cuenta de usuario se actualizará automáticamente en Active Directory, Azure Active Directory y, opcionalmente, en Microsoft 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md).

* **Ceses de empleados**: cuando se prescinde de un empleado en Workday, su cuenta de usuario se deshabilita automáticamente en Active Directory, Azure Active Directory y, opcionalmente, en Microsoft 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md).

* **Recontrataciones de empleados**: cuando se vuelve a contratar a un empleado en Workday, se puede reactivar o volver a aprovisionar automáticamente su cuenta antigua (en función de sus preferencias) en Active Directory, Azure Active Directory y, opcionalmente, en Microsoft 365 y en [otras aplicaciones SaaS compatibles con Azure AD](../app-provisioning/user-provisioning.md).

### <a name="whats-new"></a>Novedades
En esta sección se capturan las mejoras recientes de la integración de Workday. Para obtener una lista completa de actualizaciones, cambios planificados y archivos, visite la página [¿Cuáles son las novedades de Azure Active Directory?](../fundamentals/whats-new.md) 

* **Octubre de 2020: aprovisionamiento a petición habilitado para WorkDay:** Con el [aprovisionamiento a petición](../app-provisioning/provision-on-demand.md), ahora puede probar el aprovisionamiento completo para un perfil de usuario específico en WorkDay para comprobar la lógica de la expresión y la asignación de atributos.   

* **Mayo de 2020: Capacidad de escritura diferida de números de teléfono en Workday:** Además del correo electrónico y el nombre de usuario, ahora puede escribir en diferido el número de teléfono laboral y el número de teléfono móvil de Azure AD a Workday. Para obtener más información, consulte el [tutorial de la aplicación de escritura diferida](workday-writeback-tutorial.md).

* **Abril de 2020: Compatibilidad con la versión más reciente de la API de Workday Web Services (WWS):** Dos veces al año, en marzo y septiembre, Workday ofrece actualizaciones con muchas características, que le ayudan a alcanzar sus objetivos empresariales y a satisfacer los cambios en las exigencias de los trabajadores. Para estar al día con las nuevas características que ofrece Workday, ahora puede especificar directamente la versión de la API de WWS que le gustaría usar en la dirección URL de conexión. Para obtener más información sobre cómo especificar la versión de la API de Workday, consulte la sección [Configuración de la conectividad de Workday](#part-3-in-the-provisioning-app-configure-connectivity-to-workday-and-active-directory). 

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>¿Para quién es más adecuada esta solución de aprovisionamiento de usuarios?

Esta solución de aprovisionamiento de usuarios de Workday resulta adecuada para:

* Organizaciones que desean una solución basada en la nube precompilada para el aprovisionamiento de usuarios de Workday

* Organizaciones que necesitan aprovisionamiento de usuarios directo de Workday a Active Directory, o a Azure Active Directory

* Organizaciones que necesitan aprovisionamiento de usuarios mediante los datos obtenidos del módulo HCM de Workday (consulte [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html))

* Organizaciones que necesitan combinar y mover usuarios y dejarles que se sincronicen con uno o varios bosques, dominios o unidades organizativas de Active Directory en función de únicamente la información de cambio detectada en el módulo HCM de Workday (consulte [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html))

* Organizaciones que usan Microsoft 365 para el correo electrónico

## <a name="solution-architecture"></a>Arquitectura de la solución

En esta sección se describe la arquitectura de la solución de aprovisionamiento de usuarios de un extremo a otro para entornos híbridos comunes. Hay dos flujos relacionados:

* **Flujo de datos de RR. HH. autoritativo, desde Workday a Active Directory local:** en este flujo de trabajo (por ejemplo, nuevas contrataciones, transferencias o finalizaciones), los eventos se producen en primer lugar en el inquilino de la nube de RR.HH. de Workday y, a continuación, los datos de eventos fluyen hacia Active Directory local mediante Azure AD y el agente de aprovisionamiento. En función del evento, puede dar lugar a las operaciones de creación, actualización, habilitación o deshabilitación en AD.
* **Flujo de escritura diferida: de Active Directory local a Workday:** una vez completada la creación de la cuenta en Active Directory, se sincroniza con Azure AD mediante Azure AD Connect, y la información, como el correo electrónico, el nombre de usuario y el número de teléfono, puede escribirse de forma diferida en Workday.

![Información general](./media/workday-inbound-tutorial/wd_overview.png)

### <a name="end-to-end-user-data-flow"></a>Flujo de datos de usuarios de un extremo a otro

1. El equipo de RR.HH. realiza las transacciones de trabajo (empleados que se incorporan a la empresa, se trasladan o la se van, o nuevas contrataciones, transferencias o terminaciones de contratos) en Workday HCM
2. El servicio de aprovisionamiento de Azure AD ejecuta las sincronizaciones programadas de las identidades de RR.HH. de Workday e identifica los cambios que deben procesarse para la sincronización con Active Directory local.
3. El servicio de aprovisionamiento de Azure AD invoca al agente de aprovisionamiento de Azure AD Connect local con una carga de solicitud que contiene las operaciones de creación, actualización, habilitación o deshabilitación de las cuentas de AD.
4. El agente de aprovisionamiento de Azure AD Connect usa una cuenta de servicio para agregar o actualizar los datos de la cuenta de AD.
5. El motor de Azure AD Connect / AD Sync ejecuta una sincronización diferencial para extraer las actualizaciones de AD.
6. Las actualizaciones de Active Directory se sincronizan con Azure Active Directory.
7. Si la aplicación de [escritura diferida de Workday](workday-writeback-tutorial.md) está configurada, escribirá de forma diferida los atributos, como el correo electrónico, el nombre de usuario y el número de teléfono, en Workday.

## <a name="planning-your-deployment"></a>Planeamiento de la implementación

La configuración del aprovisionamiento de usuarios de Workday a Active Directory requiere un planeamiento considerable que abarca distintos aspectos, como:
* La configuración del agente de aprovisionamiento de Azure AD Connect. 
* El número de aplicaciones de aprovisionamiento de usuarios de Workday a AD que se van a implementar.
* La selección del identificador de coincidencia, la asignación de atributos, y los filtros de transformación y ámbito adecuados.

Consulte el [plan de implementación de RR. HH. en la nube](../app-provisioning/plan-cloud-hr-provision.md) para obtener instrucciones detalladas y los procedimientos recomendados. 

## <a name="configure-integration-system-user-in-workday"></a>Configuración de un usuario de integración en Workday

Un requisito común de todos los conectores de aprovisionamiento de Workday es que requieren credenciales de un usuario del sistema de integración de Workday para poder conectarse a la API de recursos humanos de Workday. En esta sección se describe cómo crear un usuario del sistema de integración de Workday y tiene las siguientes secciones:

* [Creación de un usuario del sistema de integración](#creating-an-integration-system-user)
* [Creación de un grupo de seguridad de integración](#creating-an-integration-security-group)
* [Configuración de permisos de directiva de seguridad de dominio](#configuring-domain-security-policy-permissions)
* [Configuración de los permisos de directiva de seguridad del proceso de negocio](#configuring-business-process-security-policy-permissions)
* [Activación de cambios en directiva de seguridad](#activating-security-policy-changes)

> [!NOTE]
> Se puede omitir este procedimiento y usar una cuenta de administrador global de Workday como cuenta de integración de sistemas. Puede funcionar correctamente en las demostraciones, pero no se recomienda en las implementaciones de producción.

### <a name="creating-an-integration-system-user"></a>Creación de un usuario del sistema de integración

**Para crear un usuario del sistema de integración:**

1. Inicie sesión en el inquilino de Workday usando una cuenta de administrador. En **Workday Aplication** (Aplicación de Workday), escriba create user (crear usuario) en el cuadro de búsqueda y haga clic en **Create Integration System User** (Crear usuario del sistema de integración).

   >[!div class="mx-imgBorder"] 
   >![Crear usuario](./media/workday-inbound-tutorial/wd_isu_01.png "Crear usuario")
2. Para completar la tarea **Create Integration System User** (Crear usuario del sistema de integración), especifique el nombre de usuario y la contraseña del nuevo usuario del sistema de integración.  

   * Deje la opción **Require New Password at Next Sign In** (Solicitar una nueva contraseña en el siguiente inicio de sesión) sin activar, ya que este usuario iniciará sesión mediante programación.
   * Deje **Session Timeout Minutes** (Minutos de tiempo de espera de la sesión) en 0 (su valor predeterminado), con el fin de evitar que las sesiones del usuario agoten el tiempo de espera de manera prematura.
   * Seleccione la opción **Do Not Allow UI Sessions** (No permitir sesiones de la interfaz de usuario) ya que proporciona una capa adicional de seguridad que impide que un usuario con la contraseña del sistema de integración inicie sesión en Workday.

   > [!div class="mx-imgBorder"]
   > ![Creación de un usuario del sistema de integración](./media/workday-inbound-tutorial/wd_isu_02.png "Create Integration System User")

### <a name="creating-an-integration-security-group"></a>Creación de un grupo de seguridad de integración

En este paso, creará un grupo de seguridad del sistema de integración sin o con restricciones en Workday y asignará a este grupo el usuario del sistema de integración creado en el paso anterior.

**Para crear un grupo de seguridad:**

1. Escriba create security group (crear grupo de seguridad) en el cuadro de búsqueda y luego haga clic en **Create Security Group**(Crear grupo de seguridad).

   > [!div class="mx-imgBorder"]
   > ![Captura de pantalla que muestra "crear grupo de seguridad" escrito en el cuadro de búsqueda y "Crear grupo de seguridad - Tarea" mostrado en los resultados de búsqueda.](./media/workday-inbound-tutorial/wd_isu_03.png)
2. Complete la tarea **Create Security Group** (Crear grupo de seguridad). 

   * Hay dos tipos de grupos de seguridad en Workday:
     * **Sin restricciones**: todos los miembros del grupo de seguridad pueden tener acceso a todas las instancias de datos protegidas por el grupo de seguridad.
     * **Con restricciones**: todos los miembros del grupo de seguridad tienen acceso contextual a un subconjunto de las instancias de datos (filas) al que el grupo de seguridad puede acceder.
   * Consulte con su asociado de integración de Workday cuál es el tipo de grupo de seguridad apropiado que debe seleccionar para la integración.
   * Cuando sepa el tipo de grupo, seleccione **Integration System Security Group (Unconstrained)** (Grupo de seguridad del sistema de integración, sin restricciones) o **Integration System Security Group (Constrained)** (Grupo de seguridad del sistema de integración, con restricciones) en el menú desplegable **Type of Tenanted Security Group** (Tipo de grupo de seguridad con inquilinos).

     > [!div class="mx-imgBorder&quot;]
     >![Crear grupos de seguridad](./media/workday-inbound-tutorial/wd_isu_04.png &quot;Crear grupos de seguridad")

3. Una vez que se realice correctamente la creación del grupo de seguridad, verá una página donde puede asignar miembros al grupo de seguridad. Agregue a este grupo de seguridad el usuario del sistema de integración creado en el paso anterior. Si usa el grupo de seguridad *con restricciones*, también deberá seleccionar el ámbito apropiado de la organización.

   >[!div class="mx-imgBorder"]
   >![Editar grupo de seguridad](./media/workday-inbound-tutorial/wd_isu_05.png "Edit Security Group")

### <a name="configuring-domain-security-policy-permissions"></a>Configuración de permisos de directiva de seguridad de dominio

En ese paso, concederá al grupo de seguridad los permisos de directiva "seguridad de dominio" para los datos de los empleados.

**Para configurar permisos de directiva de seguridad de dominio:**

1. Escriba **Security Group Membership and Access** (Pertenencia a grupos de seguridad y acceso) en el cuadro de búsqueda y haga clic en el vínculo de informe.
   >[!div class="mx-imgBorder"]
   >![Búsqueda de pertenencia a grupos de seguridad](./media/workday-inbound-tutorial/security-group-membership-access.png)

1. Busque y seleccione el grupo de seguridad creado en el paso anterior. 
   >[!div class="mx-imgBorder"]
   >![Seleccionar grupo de seguridad](./media/workday-inbound-tutorial/select-security-group-workday.png)

1. Haga clic en los puntos suspensivos (...) junto al nombre del grupo y, en el menú, seleccione **Security Group > Maintain Domain Permissions for Security Group** (Grupo de seguridad > Mantener permisos de dominio para el grupo de seguridad)
   >[!div class="mx-imgBorder"]
   >![Selección de mantenimiento de permisos de dominio](./media/workday-inbound-tutorial/select-maintain-domain-permissions.png)

1. En **Integration Permissions** (Permisos de integración), agregue los siguientes dominios a la lista **Domain Security Policies permitting Put access** (Directivas de seguridad de dominio que permiten el acceso Put).
   * *External Account Provisioning* (Aprovisionamiento de cuentas externas)
   * *Worker Data: Public Worker Reports* (Datos de empleado: informes de trabajadores públicos) 
   * *Person Data: Work Contact Information* (Datos de persona: información de contacto del trabajo) (necesario si planea escritura diferida de datos de contacto de Azure AD a WorkDay)
   * *WorkDay Accounts* (Cuentas de Workday) (necesario si tiene prevista la escritura diferida del nombre de usuario o UPN de Azure AD a WorkDay)

1. En **Integration Permissions** (Permisos de integración), agregue los siguientes dominios a la lista **Domain Security Policies permitting Get access** (Directivas de seguridad de dominio que permiten el acceso Get).
   * *Worker Data: Workers* (Datos de empleado: empleados)
   * *Worker Data: All Positions* (Datos de empleado: todos los cargos)
   * *Worker Data: Current Staffing Information* (Datos de empleado: información de plantilla actual)
   * *Worker Data: Business Title on Worker Profile* (Datos de empleado: cargo empresarial en el perfil del trabajador)
   * *Worker Data: Qualified Workers* (Datos de empleado. trabajadores calificados) (opcional: Agregar esto para recuperar los datos de calificación del trabajador para el aprovisionamiento)
   * *Worker Data: Skills and Experience* (Datos de empleado: aptitudes y experiencia) (Opcional: Agregue esto para recuperar los datos de las aptitudes del empleado para el aprovisionamiento)

1. Después de completar los pasos anteriores, la pantalla de permisos aparecerá como se muestra a continuación:
   >[!div class="mx-imgBorder"]
   >![Todos los permisos de seguridad del dominio](./media/workday-inbound-tutorial/all-domain-security-permissions.png)

1. Haga clic en **OK** (Aceptar) y **Done** (Listo) en la pantalla siguiente para completar la configuración. 

### <a name="configuring-business-process-security-policy-permissions"></a>Configuración de los permisos de directiva de seguridad del proceso de negocio

En ese paso, concederá permisos de directiva "seguridad de proceso de negocio" al grupo de seguridad para los datos de trabajo. 

> [!NOTE]
> Este paso es necesario solo para configurar el conector de aplicaciones de escritura diferida de Workday.

**Para configurar los permisos de directiva de seguridad del proceso de negocio:**

1. Escriba **Business Process Policy** (Directiva de proceso de negocio) en el cuadro de búsqueda y, a continuación, haga clic en la tarea **Edit Business Process Security Policy** (Editar directiva de seguridad del proceso de negocio) del vínculo.  

   >[!div class="mx-imgBorder"]
   >![Captura de pantalla que muestra "Directiva de proceso de negocio" en el cuadro de búsqueda y "Editar directiva de seguridad del proceso de negocio - Tarea" seleccionado.](./media/workday-inbound-tutorial/wd_isu_12.png "Directivas de seguridad de procesos empresariales")  

2. En el cuadro de texto **Business Process Type** (Tipo de proceso de negocio), busque *Contact* (Contacto) y seleccione el proceso de negocio **Work Contact Change** (Cambio de contacto de trabajo) y haga clic en **OK** (Aceptar).

   >[!div class="mx-imgBorder"]
   >![Captura de pantalla que muestra la página "Editar directiva de seguridad del proceso de negocio" y "Cambio de contacto de trabajo" seleccionado en el menú "Tipo de proceso de negocio".](./media/workday-inbound-tutorial/wd_isu_13.png "Directivas de seguridad de procesos empresariales")  

3. En la página **Edit Business Process Security Policy** (Editar directiva de seguridad del proceso de negocio), desplácese a la sección **Change Work Contact Information (Web Service)** [Cambiar la información de contacto de trabajo (servicio web)].


4. Seleccione y agregue el nuevo grupo de seguridad del sistema de integración a la lista de grupos de seguridad que pueden iniciar la solicitud de servicios web. 

   >[!div class="mx-imgBorder"]
   >![Directivas de seguridad de procesos empresariales](./media/workday-inbound-tutorial/wd_isu_15.png "Directivas de seguridad de procesos empresariales")  

5. Haga clic en **Done** (Acabado). 

### <a name="activating-security-policy-changes"></a>Activación de cambios en directiva de seguridad

**Para activar cambios en directiva de seguridad:**

1. Escriba activate en el cuadro de búsqueda y, a continuación, haga clic en el vínculo **Activate Pending Security Policy Changes** (Activar cambios en la directiva de seguridad pendientes).
   >[!div class="mx-imgBorder"]
   >![Activar](./media/workday-inbound-tutorial/wd_isu_16.png "Activar")

1. Inicie la tarea Activate Pending Security Policy Changes (Activar cambios en la directiva de seguridad pendientes), para ello tiene que escribir un comentario para fines de auditoría y luego hacer clic en el botón **Aceptar**.
1. Complete la tarea que aparece en la pantalla siguiente; para ello, active la casilla etiquetada como **Confirm** (Confirmar) y luego haga clic en **OK** (Aceptar).

   >[!div class="mx-imgBorder"]
   >![Activar seguridad pendiente](./media/workday-inbound-tutorial/wd_isu_18.png "Activar seguridad pendiente")  

## <a name="provisioning-agent-installation-prerequisites"></a>Requisitos previos para la instalación del agente de aprovisionamiento

Revise los [requisitos previos de instalación del agente de aprovisionamiento](../cloud-sync/how-to-prerequisites.md) antes de continuar con la siguiente sección. 

## <a name="configuring-user-provisioning-from-workday-to-active-directory"></a>Configuración del aprovisionamiento de usuarios de Workday a Active Directory

En esta sección se describen los pasos para el aprovisionamiento de cuentas de usuario desde Workday en cada dominio de Active Directory dentro del ámbito de su integración.

* [Incorporación de la aplicación de conector de aprovisionamiento y descarga del agente de aprovisionamiento](#part-1-add-the-provisioning-connector-app-and-download-the-provisioning-agent)
* [Instalación y configuración de agentes de aprovisionamiento en el entorno local](#part-2-install-and-configure-on-premises-provisioning-agents)
* [Configuración de la conectividad con WorkDay y Active Directory](#part-3-in-the-provisioning-app-configure-connectivity-to-workday-and-active-directory)
* [Configuración de las asignaciones de atributos](#part-4-configure-attribute-mappings)
* [Habilitar e iniciar el aprovisionamiento de usuarios](#enable-and-launch-user-provisioning)

### <a name="part-1-add-the-provisioning-connector-app-and-download-the-provisioning-agent"></a>Parte 1: Incorporación de la aplicación de conector de aprovisionamiento y descarga del agente de aprovisionamiento

**Para configurar Workday en el aprovisionamiento de Active Directory:**

1. Ir a <https://portal.azure.com>.

2. En Azure Portal, busque y seleccione **Azure Active Directory**.

3. Seleccione **Enterprise Applications** (Aplicaciones empresariales) y **All Applications** (Todas las aplicaciones).

4. Seleccione **Add an application** (Agregar una aplicación) y seleccione la categoría **All** (Todos).

5. Busque **Workday to Active Directory User Provisioning** y agregue esa aplicación desde la galería.

6. Una vez que se haya agregado la aplicación y se muestre la pantalla de detalles de la aplicación, seleccione **Aprovisionamiento**.

7. Cambie el **Modo** de **aprovisionamiento** a **Automático**.

8. Haga clic en el banner de información que se muestra para descargar el agente de aprovisionamiento. 

   >[!div class="mx-imgBorder"]
   >![Descarga del agente](./media/workday-inbound-tutorial/pa-download-agent.png "Pantalla de descarga de agente")

### <a name="part-2-install-and-configure-on-premises-provisioning-agents"></a>Parte 2: Instalación y configuración de agentes de aprovisionamiento en el entorno local

Para realizar el aprovisionamiento en Active Directory local, el agente de aprovisionamiento debe estar instalado en un servidor unido a un dominio que tenga acceso de red a los dominios de Active Directory deseados.

Transfiera el instalador del agente descargado al host del servidor y siga los pasos indicados [en la sección de **instalación del agente**](../cloud-sync/how-to-install.md) para completar su configuración.

### <a name="part-3-in-the-provisioning-app-configure-connectivity-to-workday-and-active-directory"></a>Parte 3: En la aplicación de aprovisionamiento, configuración de la conectividad con WorkDay y Active Directory
En este paso, se establecerá la conectividad con WorkDay y Active Directory en Azure Portal. 

1. En Azure Portal, vuelva a la aplicación de aprovisionamiento de usuarios de Workday en Active Directory creada en la [Parte 1](#part-1-add-the-provisioning-connector-app-and-download-the-provisioning-agent).

1. Cumplimente la sección **Credenciales de administrador** del siguiente modo:

   * **Nombre de usuario de Workday**: escriba el nombre de usuario de la cuenta del sistema de integración de Workday, anexando el nombre de dominio del inquilino. Este debe tener un aspecto similar al siguiente:**username\@tenant_name**

   * **Contraseña de Workday**: escriba la contraseña de la cuenta del sistema de integración de Workday.

   * **URL de la API Workday Web Services**: Escriba la dirección URL al punto de conexión de Workday Web Services de su inquilino. La dirección URL determina la versión de la API de Workday Web Services utilizada por el conector. 

     | Formato de dirección URL | Versión de la API WWS usada | Cambios de XPATH necesarios |
     |------------|----------------------|------------------------|
     | https://####.workday.com/ccx/service/tenantName | v21.1 | No |
     | https://####.workday.com/ccx/service/tenantName/Human_Resources | v21.1 | No |
     | https://####.workday.com/ccx/service/tenantName/Human_Resources/v##.# | v##.# | Sí |

      > [!NOTE]
     > Si no se especifica ninguna información de versión en la dirección URL, la aplicación usa Workday Web Services (WWS) v21.1 y no se requieren cambios en las expresiones de la API XPATH predeterminadas que se incluyen con la aplicación. Para usar una versión específica de la API WWS, especifique el número de versión en la dirección URL. <br>
     > Ejemplo: `https://wd3-impl-services1.workday.com/ccx/service/contoso4/Human_Resources/v34.0` <br>
     > <br> Si usa la API WWS v30.0 o posterior, antes de activar el trabajo de aprovisionamiento, debe actualizar las **expresiones de la API XPATH** en **Asignación de atributos -> Opciones avanzadas -> Editar lista de atributos para Workday** en la sección [Administración de la configuración](#managing-your-configuration) y [Referencia de atributos de Workday](../app-provisioning/workday-attribute-reference.md#xpath-values-for-workday-web-services-wws-api-v30).  

   * **Bosque de Active Directory**: el nombre de su dominio de Active Directory, según se ha registrado con el agente. Use la lista desplegable para seleccionar el dominio de destino para el aprovisionamiento. Este valor suele ser una cadena como: *contoso.com*.

   * **Contenedor de Active Directory:** escriba el DN del contenedor donde el agente debe crear cuentas de usuario de forma predeterminada.
        Ejemplo: *OU=Standard Users,OU=Users,DC=contoso,DC=test*

     > [!NOTE]
     > Esta configuración solo entra en juego para creaciones de cuentas de usuario si el atributo *parentDistinguishedName* no está configurado en las asignaciones de atributos. Esta configuración no se usa para la búsqueda de usuarios o las operaciones de actualización. El subárbol de todo el dominio se encuentra en el ámbito de la operación de búsqueda.

   * **Correo electrónico de notificación**: escriba su dirección de correo electrónico y marque la casilla "Send email if failure occurs" (Enviar una notificación por correo electrónico cuando se produzca un error).

     > [!NOTE]
     > El servicio de aprovisionamiento de Azure AD envía la notificación por correo electrónico si el trabajo de aprovisionamiento entra en un estado de[cuarentena](../app-provisioning/application-provisioning-quarantine-status.md).

   * Haga clic en el botón **Probar conexión**. Si la prueba de conexión se lleva a cabo correctamente, haga clic en el botón **Guardar** situado en la parte superior. Si se produce un error, compruebe que las credenciales de Workday y las credenciales de AD configuradas en la instalación del agente sean válidas.

     >[!div class="mx-imgBorder"]
     >![Captura de pantalla que muestra la página "Aprovisionamiento" con las credenciales especificadas.](./media/workday-inbound-tutorial/wd_1.png)

   * Una vez que las credenciales se guardan correctamente, en la sección **Asignaciones** se mostrará la asignación predeterminada **Synchronize Workday Workers to On Premises Active Directory** (Sincronizar trabajadores de Workday con Active Directory local).

### <a name="part-4-configure-attribute-mappings"></a>Parte 4: configuración de las asignaciones de atributos

En esta sección configurará cómo fluyen los datos de los usuarios de Workday a Active Directory.

1. En la pestaña Aprovisionamiento, en **Asignaciones**, haga clic en **Synchronize Workday Workers to On Premises Active Directory** (Sincronizar trabajadores de Workday con Active Directory local).

1. En el campo **Ámbito de objeto de origen** puede seleccionar los conjuntos de usuarios de Workday que deben estar en el ámbito para el aprovisionamiento en AD; para ello, debe definir un conjunto de filtros basados en atributos. El ámbito predeterminado es "todos los usuarios de Workday". Filtros de ejemplo:

   * Ejemplo: ámbito de los usuarios que tengan los id. de trabajador comprendidos entre 1000000 y 2000000 (sin incluir 2000000)

      * Atributo: WorkerID

      * Operador: REGEX Match

      * Valor: (1[0-9][0-9][0-9][0-9][0-9][0-9])

   * Ejemplo: solo los empleados, no los trabajadores temporales

      * Atributo: EmployeeID

      * Operador: IS NOT NULL

   > [!TIP]
   > Al configurar la aplicación de aprovisionamiento por primera vez, deberá probar y verificar las expresiones y asignaciones de atributos para asegurarse de que ofrece el resultado deseado. Microsoft recomienda usar [filtros de ámbito](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md) en **Ámbito de objeto de origen** y [aprovisionamiento a petición](../app-provisioning/provision-on-demand.md) para probar las asignaciones con algunos usuarios de prueba de Workday. Una vez haya verificado que las asignaciones funcionan, puede quitar el filtro o expandirlo gradualmente para incluir más usuarios.

   > [!CAUTION] 
   > El comportamiento predeterminado del motor de aprovisionamiento es deshabilitar o eliminar usuarios que salen del ámbito. Puede que esta no sea la situación deseable en la integración de Workday con AD. Para invalidar este comportamiento predeterminado, consulte el artículo [Omisión de la eliminación de usuarios fuera del ámbito](../app-provisioning/skip-out-of-scope-deletions.md).

1. En el campo **Acciones del objeto de destino**, puede filtrar de forma global qué acciones se realizan en Active Directory. **Crear** y **Actualizar** son las más habituales.

1. En la sección **Asignaciones de atributos** puede definir cómo se asignan los distintos atributos de Workday a los atributos de Active Directory.

1. Haga clic en una asignación de atributos existente para actualizarla o haga clic en **Agregar nueva asignación** en la parte inferior de la pantalla para agregar asignaciones nuevas. Las asignaciones de atributos admiten estas propiedades:

      * **Tipo de asignación**

         * **Directo**: escribe el valor del atributo de Workday en el atributo de AD sin cambios.

         * **Constante**: escribe un valor de cadena estático y constante en el atributo de AD.

         * **Expresión**: le permite escribir un valor personalizado en el atributo de AD, en función de uno o varios atributos de Workday. [Para obtener más información, consulte este artículo sobre las expresiones](../app-provisioning/functions-for-customizing-application-data.md).

      * **Atributo de origen**: el atributo de usuario de Workday. Si el atributo que está buscando no está presente, consulte [Personalización de la lista de atributos de usuario de Workday](#customizing-the-list-of-workday-user-attributes).

      * **Valor predeterminado**: opcional. Si el atributo de origen tiene un valor vacío, la asignación escribirá este valor.
            La configuración más habitual consiste en dejarlo en blanco.

      * **Atributo de destino**: atributo de usuario de Active Directory.

      * **Hacer coincidir objetos con este atributo**: especifica si se debe usar o no esta asignación para identificar de forma unívoca a los usuarios entre Workday y Active Directory. Este valor se suele establecer en el campo Id. del trabajador de Workday, que se suele asignar a uno de los atributos de id. de empleado de Active Directory.

      * **Precedencia de coincidencia**: se pueden establecer varios atributos coincidentes. Si hay varios, se evalúan en el orden definido por este campo. En el momento en que se encuentre una coincidencia, no se evaluarán más atributos coincidentes.

      * **Aplicar esta asignación**

         * **Siempre**: esta asignación se aplica a las acciones de creación y actualización de usuarios

         * **Solo durante la creación**: esta asignación se aplica solo a las acciones de creación de usuarios

1. Para guardar las asignaciones, haga clic en **Guardar** en la parte superior de la sección Asignación de atributos.
   >[!div class="mx-imgBorder"]
   >![Captura de pantalla que muestra la página "Asignación de atributos" con la acción "Guardar" seleccionada.](./media/workday-inbound-tutorial/wd_2.png)

#### <a name="below-are-some-example-attribute-mappings-between-workday-and-active-directory-with-some-common-expressions"></a>A continuación se muestran algunos ejemplos de asignaciones de atributos entre Workday y Active Directory, con algunas expresiones comunes.

* La expresión que se asigna al atributo *parentDistinguishedName* se usa para aprovisionar a los usuarios en distintas unidades organizativas en función de uno o varios atributos de origen de Workday. En este ejemplo se coloca a los usuarios en distintas unidades organizativas en función de la ciudad en la que se encuentran.

* El atributo *userPrincipalName* de Active Directory se genera mediante la función de desduplicación [SelectUniqueValue](../app-provisioning/functions-for-customizing-application-data.md#selectuniquevalue) que comprueba la existencia de un valor generado en el dominio de AD de destino y solo se establece siempre que sea único.  

* [Aquí encontrará documentación sobre cómo escribir expresiones](../app-provisioning/functions-for-customizing-application-data.md). Esta sección incluye ejemplos sobre cómo quitar caracteres especiales.

| ATRIBUTO DE WORKDAY | ATRIBUTO DE ACTIVE DIRECTORY |  ¿IDENTIFICADOR COINCIDENTE? | CREAR / ACTUALIZAR |
| ---------- | ---------- | ---------- | ---------- |
| **WorkerID**  |  EmployeeID | **Sí** | Escrito únicamente en Crear |
| **PreferredNameData**    |  cn    |   |   Escrito únicamente en Crear |
| **SelectUniqueValue( Join("\@", Join(".",  \[FirstName\], \[LastName\]), "contoso.com"), Join("\@", Join(".",  Mid(\[FirstName\], 1, 1), \[LastName\]), "contoso.com"), Join("\@", Join(".",  Mid(\[FirstName\], 1, 2), \[LastName\]), "contoso.com"))**   | userPrincipalName     |     | Escrito únicamente en Crear 
| `Replace(Mid(Replace(\[UserID\], , "(\[\\\\/\\\\\\\\\\\\\[\\\\\]\\\\:\\\\;\\\\\|\\\\=\\\\,\\\\+\\\\\*\\\\?\\\\&lt;\\\\&gt;\])", , "", , ), 1, 20), , "([\\\\.)\*\$](file:///\\.)*$)", , "", , )`      |    sAMAccountName            |     |         Escrito únicamente en Crear |
| **Switch(\[Active\], , "0", "True", "1", "False")** |  accountDisabled      |     | Crear y Actualizar |
| **Nombre**   | givenName       |     |    Crear y Actualizar |
| **Apellidos**   |   sn   |     |  Crear y Actualizar |
| **PreferredNameData**  |  DisplayName |     |   Crear y Actualizar |
| **Company**         | company   |     |  Crear y Actualizar |
| **SupervisoryOrganization**  | department  |     |  Crear y Actualizar |
| **ManagerReference**   | manager  |     |  Crear y Actualizar |
| **BusinessTitle**   |  title     |     |  Crear y Actualizar | 
| **AddressLineData**    |  streetAddress  |     |   Crear y Actualizar |
| **Municipality**   |   l   |     | Crear y Actualizar |
| **CountryReferenceTwoLetter**      |   co |     |   Crear y Actualizar |
| **CountryReferenceTwoLetter**    |  c  |     |         Crear y Actualizar |
| **CountryRegionReference** |  st     |     | Crear y Actualizar |
| **WorkSpaceReference** | physicalDeliveryOfficeName    |     |  Crear y Actualizar |
| **PostalCode**  |   postalCode  |     | Crear y Actualizar |
| **PrimaryWorkTelephone**  |  telephoneNumber   |     | Crear y Actualizar |
| **Fax**      | facsimileTelephoneNumber     |     |    Crear y Actualizar |
| **Mobile**  |    mobile       |     |       Crear y Actualizar |
| **LocalReference** |  preferredLanguage  |     |  Crear y Actualizar |                                               
| **Switch(\[Municipality\], "OU=Default Users,DC=contoso,DC=com", "Dallas", "OU=Dallas,OU=Users,DC=contoso,DC=com", "Austin", "OU=Austin,OU=Users,DC=contoso,DC=com", "Seattle", "OU=Seattle,OU=Users,DC=contoso,DC=com", "London", "OU=London,OU=Users,DC=contoso,DC=com")**  | parentDistinguishedName     |     |  Crear y Actualizar |

Una vez completada la configuración de la asignación de atributos, puede probar el aprovisionamiento de un solo usuario mediante [el aprovisionamiento a petición](../app-provisioning/provision-on-demand.md) y, a continuación, [habilitar e iniciar el servicio de aprovisionamiento de usuarios](#enable-and-launch-user-provisioning).

## <a name="enable-and-launch-user-provisioning"></a>Habilitar e iniciar el aprovisionamiento de usuarios

Cuando se hayan completado las configuraciones de la aplicación de aprovisionamiento de Workday y se haya comprobado el aprovisionamiento de un solo usuario con el [aprovisionamiento a petición](../app-provisioning/provision-on-demand.md), puede activar el servicio de aprovisionamiento en Azure Portal.

> [!TIP]
> De forma predeterminada, al activar el servicio de aprovisionamiento, se iniciarán las operaciones de aprovisionamiento para todos los usuarios del ámbito. Si hay errores en la asignación o problemas con los datos de Workday, puede que se produzcan errores con el trabajo de aprovisionamiento y que entre en estado de cuarentena. Para evitar esto, como procedimiento recomendado, es conveniente configurar el filtro **Ámbito de objeto de origen** y probar las asignaciones de atributos con algunos usuarios de prueba mediante el [aprovisionamiento a petición](../app-provisioning/provision-on-demand.md) antes de iniciar la sincronización completa de todos los usuarios. Una vez haya verificado que las asignaciones funcionan y que obtiene los resultados deseados, puede quitar el filtro o expandirlo gradualmente para incluir más usuarios.

1. Vaya a la hoja **Aprovisionamiento** y haga clic en **Iniciar aprovisionamiento**.

1. Esta operación iniciará la sincronización inicial, que puede tardar una cantidad de horas variable, según el número de usuarios que haya en el inquilino de Workday. Puede consultar en la barra de progreso el seguimiento del progreso del ciclo de sincronización. 

1. En cualquier momento, compruebe la pestaña **Registros de auditoría** en Azure Portal para ver las acciones que ha realizado el servicio de aprovisionamiento. Los registros de auditoría muestran todos los eventos de sincronización individuales realizados por el servicio de aprovisionamiento, por ejemplo, los usuarios que se leen fuera de Workday y que luego se agregan o actualizan en Active Directory. Consulte la sección Solución de problemas para obtener instrucciones sobre cómo revisar los registros de auditoría y corregir los errores de aprovisionamiento.

1. Una vez completada la sincronización inicial, se escribe un informe resumido de auditoría en la pestaña **Aprovisionamiento**, tal y como se muestra a continuación.
   > [!div class="mx-imgBorder"]
   > ![Barra de progreso de aprovisionamiento](./media/sap-successfactors-inbound-provisioning/prov-progress-bar-stats.png)

## <a name="frequently-asked-questions-faq"></a>Preguntas más frecuentes

* **Preguntas sobre la funcionalidad de la solución**
  * [Al procesar una nueva contratación en Workday, ¿cómo configura la solución la contraseña para la nueva cuenta de usuario en Active Directory?](#when-processing-a-new-hire-from-workday-how-does-the-solution-set-the-password-for-the-new-user-account-in-active-directory)
  * [¿La solución admite el envío de notificaciones de correo electrónico una vez finalizadas las operaciones de aprovisionamiento?](#does-the-solution-support-sending-email-notifications-after-provisioning-operations-complete)
  * [¿La solución almacena en memoria caché los perfiles de usuario de Workday en la nube de Azure AD o en la capa del agente de aprovisionamiento?](#does-the-solution-cache-workday-user-profiles-in-the-azure-ad-cloud-or-at-the-provisioning-agent-layer)
  * [¿La solución permite asignar grupos de AD locales al usuario?](#does-the-solution-support-assigning-on-premises-ad-groups-to-the-user)
  * [¿Qué API de Workday usa la solución para consultar y actualizar los perfiles de trabajo de Workday?](#which-workday-apis-does-the-solution-use-to-query-and-update-workday-worker-profiles)
  * [¿Puede configurar mi inquilino de HCM en Workday con dos inquilinos de Azure AD?](#can-i-configure-my-workday-hcm-tenant-with-two-azure-ad-tenants)
  * [¿Cómo puedo sugerir mejoras o solicitar nuevas características relacionadas con la integración de Workday y Azure AD?](#how-do-i-suggest-improvements-or-request-new-features-related-to-workday-and-azure-ad-integration)

* **Preguntas sobre el agente de aprovisionamiento**
  * [¿Cuál es la versión de disponibilidad general del agente de aprovisionamiento?](#what-is-the-ga-version-of-the-provisioning-agent)
  * [¿Cómo puedo saber la versión de mi agente de aprovisionamiento?](#how-do-i-know-the-version-of-my-provisioning-agent)
  * [¿Microsoft inserta automáticamente las actualizaciones del agente de aprovisionamiento?](#does-microsoft-automatically-push-provisioning-agent-updates)
  * [¿Puedo instalar el agente de aprovisionamiento en el mismo servidor que ejecuta Azure AD Connect?](#can-i-install-the-provisioning-agent-on-the-same-server-running-azure-ad-connect)
  * [¿Cómo se puede configurar el agente de aprovisionamiento para usar un servidor proxy para la comunicación HTTP saliente?](#how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication)
  * [¿Cómo se puede garantizar que el agente de aprovisionamiento sea capaz de comunicarse con el inquilino de Azure AD y que ningún firewall bloquee los puertos requeridos por el agente?](#how-do-i-ensure-that-the-provisioning-agent-is-able-to-communicate-with-the-azure-ad-tenant-and-no-firewalls-are-blocking-ports-required-by-the-agent)
  * [¿Cómo puedo anular el registro del dominio asociado con mi agente de aprovisionamiento?](#how-do-i-de-register-the-domain-associated-with-my-provisioning-agent)
  * [¿Cómo puedo desinstalar el agente de aprovisionamiento?](#how-do-i-uninstall-the-provisioning-agent)

* **Preguntas sobre la asignación y configuración de atributos de Workday en AD**
  * [¿Cómo puedo realizar una copia de seguridad o exportar una copia de trabajo del esquema y la asignación de atributos de aprovisionamiento de Workday?](#how-do-i-back-up-or-export-a-working-copy-of-my-workday-provisioning-attribute-mapping-and-schema)
  * [Tengo atributos personalizados en Workday y Active Directory. ¿Cómo se puede configurar la solución para que funcione con los atributos personalizados?](#i-have-custom-attributes-in-workday-and-active-directory-how-do-i-configure-the-solution-to-work-with-my-custom-attributes)
  * [¿Puedo aprovisionar la foto de un usuario desde Workday en Active Directory?](#can-i-provision-users-photo-from-workday-to-active-directory)
  * [¿Cómo se pueden sincronizar los números de móvil desde Workday según el consentimiento del usuario para su uso público?](#how-do-i-sync-mobile-numbers-from-workday-based-on-user-consent-for-public-usage)
  * [¿Cómo puedo dar formato a los nombres para mostrar en AD según los atributos de departamento/país/ciudad del usuario y administrar las variaciones regionales?](#how-do-i-format-display-names-in-ad-based-on-the-users-departmentcountrycity-attributes-and-handle-regional-variances)
  * [¿Cómo se puede usar SelectUniqueValue para generar valores únicos para el atributo samAccountName?](#how-can-i-use-selectuniquevalue-to-generate-unique-values-for-samaccountname-attribute)
  * [¿Cómo se pueden quitar caracteres con signos diacríticos y convertirlos en caracteres habituales del alfabeto inglés?](#how-do-i-remove-characters-with-diacritics-and-convert-them-into-normal-english-alphabets)

### <a name="solution-capability-questions"></a>Preguntas sobre la funcionalidad de la solución

#### <a name="when-processing-a-new-hire-from-workday-how-does-the-solution-set-the-password-for-the-new-user-account-in-active-directory"></a>Al procesar una nueva contratación en Workday, ¿cómo configura la solución la contraseña para la nueva cuenta de usuario en Active Directory?

Cuando el agente de aprovisionamiento local obtiene una solicitud para crear una cuenta de AD, genera automáticamente una contraseña aleatoria compleja diseñada para satisfacer los requisitos de complejidad de contraseña definidos por el servidor de AD y la define en el objeto de usuario. Esta contraseña no se registra en ningún lugar.

#### <a name="does-the-solution-support-sending-email-notifications-after-provisioning-operations-complete"></a>¿La solución admite el envío de notificaciones de correo electrónico una vez finalizadas las operaciones de aprovisionamiento?

No, el envío de notificaciones por correo electrónico después de completar las operaciones de aprovisionamiento no se admite en la versión actual.

#### <a name="does-the-solution-cache-workday-user-profiles-in-the-azure-ad-cloud-or-at-the-provisioning-agent-layer"></a>¿La solución almacena en memoria caché los perfiles de usuario de Workday en la nube de Azure AD o en la capa del agente de aprovisionamiento?

No, la solución no mantiene una caché de perfiles de usuario. El servicio de aprovisionamiento de Azure AD solo actúa como procesador de datos, de tal forma que lee los datos en Workday y los escribe en la instancia de destino de Active Directory o Azure AD. Consulte la sección [Administración de datos personales](#managing-personal-data) para obtener información relacionada con la retención de datos y la privacidad del usuario.

#### <a name="does-the-solution-support-assigning-on-premises-ad-groups-to-the-user"></a>¿La solución permite asignar grupos de AD locales al usuario?

Actualmente no se admite esta funcionalidad. La solución alternativa recomendada es implementar un script de PowerShell que realiza consultas en el punto de conexión de Microsoft Graph API para los [datos de registro de auditoría](/graph/api/resources/azure-ad-auditlog-overview) y utilizar eso para desencadenar escenarios como la asignación de grupos. Este script de PowerShell puede anexarse a un programador de tareas e implementarse en el mismo equipo que ejecuta el agente de aprovisionamiento.  

#### <a name="which-workday-apis-does-the-solution-use-to-query-and-update-workday-worker-profiles"></a>¿Qué API de Workday usa la solución para consultar y actualizar los perfiles de trabajo de Workday?

Actualmente, la solución utiliza las siguientes API de Workday:

* El formato de **URL de la API de Servicios web de Workday** que se usa en la sección **Credenciales de administrador** determina la versión de API que se usa para Get_Workers.
  * Si el formato de la dirección URL es https://\#\#\#\#\.workday\.com/ccx/service/nombreDeInquilino, se usa la versión 21.1 de la API. 
  * Si el formato de la dirección URL es https://\#\#\#\#\.workday\.com/ccx/service/nombreDeInquilino/Human\_Resources, se usa la versión 21.1 de la API. 
  * Si el formato de la dirección URL es: https://\#\#\#\#\.workday\.com/ccx/service/nombreDeInquilino/Human\_Resources/v\#\#\.\#, se usa la versión de API especificada (por ejemplo, si se especifica la versión 34.0, se usa esta).  

* La característica de escritura diferida de correo electrónico de Workday usa Change_Work_Contact_Information (v30.0). 
* La característica de escritura diferida de nombre de usuario de Workday utiliza Update_Workday_Account (v31.2). 

#### <a name="can-i-configure-my-workday-hcm-tenant-with-two-azure-ad-tenants"></a>¿Puede configurar mi inquilino de HCM en Workday con dos inquilinos de Azure AD?

Sí, esta configuración es compatible. Estos son los pasos de alto nivel para configurar este escenario:

* Implemente el agente de aprovisionamiento 1 y regístrelo en el inquilino de Azure AD 1.
* Implemente el agente de aprovisionamiento 2 y regístrelo en el inquilino de Azure AD 2.
* En función de los "dominios secundarios" que cada agente de aprovisionamiento administrará, configure cada agente con los dominios. Un agente puede controlar varios dominios.
* En Azure Portal, configure la aplicación de aprovisionamiento de usuarios de Workday en AD en cada inquilino y configúrela con los dominios correspondientes.

#### <a name="how-do-i-suggest-improvements-or-request-new-features-related-to-workday-and-azure-ad-integration"></a>¿Cómo puedo sugerir mejoras o solicitar nuevas características relacionadas con la integración de Workday y Azure AD?

Sus comentarios son muy valiosos porque nos ayudan a definir la dirección de las futuras versiones y mejoras. Agradecemos todos los comentarios y le instamos a que envíe sus ideas o sugerencias de mejora en el [foro de comentarios de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory). Para encontrar comentarios específicos relacionados con la integración de Workday, seleccione la categoría *Aplicaciones SaaS* y busque con la palabra clave *Workday* para encontrar comentarios existentes relativos a Workday.

> [!div class="mx-imgBorder"]
> ![Aplicaciones SaaS para UserVoice](media/workday-inbound-tutorial/uservoice_saas_apps.png)

> [!div class="mx-imgBorder"]
> ![Workday para UserVoice](media/workday-inbound-tutorial/uservoice_workday_feedback.png)

Al sugerir una nueva idea, compruebe si alguien ha sugerido ya una característica similar. En su caso, puede votar a favor de la solicitud de característica o mejora. También puede dejar un comentario sobre su caso de uso específico para mostrar su apoyo a la idea y demostrar lo importante que también será para usted la característica.

### <a name="provisioning-agent-questions"></a>Preguntas sobre el agente de aprovisionamiento

#### <a name="what-is-the-ga-version-of-the-provisioning-agent"></a>¿Cuál es la versión de disponibilidad general del agente de aprovisionamiento?

Consulte [Agente de aprovisionamiento de Azure AD Connect: Historial de lanzamiento de versiones](../app-provisioning/provisioning-agent-release-version-history.md) para ver la versión más reciente en GA del agente de aprovisionamiento.  

#### <a name="how-do-i-know-the-version-of-my-provisioning-agent"></a>¿Cómo puedo saber la versión de mi agente de aprovisionamiento?

* Inicie sesión en el servidor de Windows en el que está instalado el agente de aprovisionamiento.
* Vaya al menú **Panel de control** -> **Desinstalar o cambiar un programa**.
* Busque la versión correspondiente a la entrada **Microsoft Azure AD Connect Provisioning Agent**.

  >[!div class="mx-imgBorder"]
  >![Azure Portal](./media/workday-inbound-tutorial/pa_version.png)

#### <a name="does-microsoft-automatically-push-provisioning-agent-updates"></a>¿Microsoft inserta automáticamente las actualizaciones del agente de aprovisionamiento?

Sí, Microsoft actualiza automáticamente el agente de aprovisionamiento si el servicio de Windows **Programa de actualización de Microsoft Azure Active Directory Connect** está en ejecución.

#### <a name="can-i-install-the-provisioning-agent-on-the-same-server-running-azure-ad-connect"></a>¿Puedo instalar el agente de aprovisionamiento en el mismo servidor que ejecuta Azure AD Connect?

Sí, puede instalar el agente de aprovisionamiento en el mismo servidor que ejecuta Azure AD Connect.

#### <a name="at-the-time-of-configuration-the-provisioning-agent-prompts-for-azure-ad-admin-credentials-does-the-agent-store-the-credentials-locally-on-the-server"></a>En el momento de la configuración, el agente de aprovisionamiento solicita las credenciales de administrador de Azure AD. ¿El agente almacena las credenciales localmente en el servidor?

Durante la configuración, el agente de aprovisionamiento solicita las credenciales de administrador de Azure AD solo para conectarse al inquilino de Azure AD. No almacena las credenciales localmente en el servidor. Sin embargo, conserva las credenciales usadas para conectarse al *dominio de Active Directory local* en un almacén de contraseñas local de Windows.

#### <a name="how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication"></a>¿Cómo se puede configurar el agente de aprovisionamiento para usar un servidor proxy para la comunicación HTTP saliente?

El agente de aprovisionamiento admite el uso de un proxy de salida. Puede configurarlo al editar el archivo de configuración del agente **C:\Archivos de programa\Microsoft Azure AD Connect Provisioning Agent\AADConnectProvisioningAgent.exe.config**. Agregue al archivo las siguientes líneas, hacia el final del archivo, justo antes de la etiqueta de cierre `</configuration>`.
Reemplace las variables [proxy-server] y [proxy-port] con los valores del puerto y el nombre del servidor proxy.

```xml
    <system.net>
          <defaultProxy enabled="true" useDefaultCredentials="true">
             <proxy
                usesystemdefault="true"
                proxyaddress="http://[proxy-server]:[proxy-port]"
                bypassonlocal="true"
             />
         </defaultProxy>
    </system.net>
```

#### <a name="how-do-i-ensure-that-the-provisioning-agent-is-able-to-communicate-with-the-azure-ad-tenant-and-no-firewalls-are-blocking-ports-required-by-the-agent"></a>¿Cómo se puede garantizar que el agente de aprovisionamiento sea capaz de comunicarse con el inquilino de Azure AD y que ningún firewall bloquee los puertos requeridos por el agente?

También puede comprobar si todos los [puertos necesarios](../app-proxy/application-proxy-add-on-premises-application.md#open-ports) están abiertos.

#### <a name="can-one-provisioning-agent-be-configured-to-provision-multiple-ad-domains"></a>¿Un agente de aprovisionamiento se puede configurar para aprovisionar varios dominios de AD?

Sí, un agente de aprovisionamiento puede configurarse para controlar varios dominios de AD, siempre que el agente tenga una línea de visión de los controladores de dominio correspondientes. Microsoft recomienda configurar un grupo de tres agentes de aprovisionamiento que abastezca al mismo conjunto de dominios de AD para garantizar una alta disponibilidad y proporcionar soporte técnico para la conmutación por error.

#### <a name="how-do-i-de-register-the-domain-associated-with-my-provisioning-agent"></a>¿Cómo puedo anular el registro del dominio asociado con mi agente de aprovisionamiento?

* En Azure Portal, obtenga el *Id. de inquilino* del inquilino de Azure AD.
* Inicie sesión en el servidor de Windows que ejecuta el agente de aprovisionamiento.
* Abra PowerShell como administrador de Windows.
* Cambie al directorio que contiene los scripts de registro y ejecute los siguientes comandos reemplazando el parámetro \[Id. de inquilino\] con el valor del identificador del inquilino.

  ```powershell
  cd "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\RegistrationPowershell\Modules\PSModulesFolder"
  Import-Module "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\RegistrationPowershell\Modules\PSModulesFolder\AppProxyPSModule.psd1"
  Get-PublishedResources -TenantId "[tenant ID]"
  ```

* En la lista de agentes que aparece: copie el valor del campo `id` de dicho recurso cuyo parámetro *resourceName* es igual al nombre de dominio de AD.
* Pegue el valor del identificador en este comando y ejecute el comando en PowerShell.

  ```powershell
  Remove-PublishedResource -ResourceId "[resource ID]" -TenantId "[tenant ID]"
  ```

* Vuelva a ejecutar el asistente de configuración del agente.
* Será necesario volver a configurar todos los demás agentes asignados anteriormente a este dominio.

#### <a name="how-do-i-uninstall-the-provisioning-agent"></a>¿Cómo puedo desinstalar el agente de aprovisionamiento?

* Inicie sesión en el servidor de Windows en el que está instalado el agente de aprovisionamiento.
* Vaya al menú **Panel de control** -> **Desinstalar o cambiar un programa**.
* Desinstale los programas siguientes:
  * Microsoft Azure AD Connect Provisioning Agent
  * Agent Updater de Microsoft Azure AD Connect
  * Microsoft Azure AD Connect Provisioning Agent Package

### <a name="workday-to-ad-attribute-mapping-and-configuration-questions"></a>Preguntas sobre la asignación y configuración de atributos de Workday en AD

#### <a name="how-do-i-back-up-or-export-a-working-copy-of-my-workday-provisioning-attribute-mapping-and-schema"></a>¿Cómo puedo realizar una copia de seguridad o exportar una copia de trabajo del esquema y la asignación de atributos de aprovisionamiento de Workday?

Puede usar Microsoft Graph API para exportar la configuración de aprovisionamiento de usuarios de Workday. Consulte los pasos descritos en la sección [Exportación e importación de la configuración de la asignación de atributos de aprovisionamiento de usuarios de Workday](#exporting-and-importing-your-configuration) para obtener más información.

#### <a name="i-have-custom-attributes-in-workday-and-active-directory-how-do-i-configure-the-solution-to-work-with-my-custom-attributes"></a>Tengo atributos personalizados en Workday y Active Directory. ¿Cómo se puede configurar la solución para que funcione con los atributos personalizados?

La solución es compatible con los atributos personalizados de Workday y Active Directory. Para agregar los atributos personalizados al esquema de asignación, abra la hoja **Asignación de atributos** y desplácese hacia abajo para expandir la sección **Mostrar opciones avanzadas**. 

![Editar lista de atributos](./media/workday-inbound-tutorial/wd_edit_attr_list.png)

Para agregar los atributos personalizados de Workday, seleccione la opción *Editar lista de atributos para Workday* y, para agregar los atributos personalizados de AD, seleccione la opción *Editar lista de atributos para Active Directory local*.

Consulte también:

* [Personalización de la lista de atributos de usuario de Workday](#customizing-the-list-of-workday-user-attributes)

#### <a name="how-do-i-configure-the-solution-to-only-update-attributes-in-ad-based-on-workday-changes-and-not-create-any-new-ad-accounts"></a>¿Cómo se puede configurar la solución para actualizar solo los atributos de AD según los cambios de Workday sin crear cuentas de AD?

Esta configuración puede realizarse en **Acciones del objeto de destino** en la hoja **Asignaciones de atributos** como se muestra a continuación:

![Acción de actualización](./media/workday-inbound-tutorial/wd_target_update_only.png)

Active la casilla "Actualizar" solo para que las operaciones de actualización fluyan de Workday a AD. 

#### <a name="can-i-provision-users-photo-from-workday-to-active-directory"></a>¿Puedo aprovisionar la foto de un usuario desde Workday en Active Directory?

La solución actualmente no permite establecer los atributos binarios como *thumbnailPhoto* y *jpegPhoto* en Active Directory.

#### <a name="how-do-i-sync-mobile-numbers-from-workday-based-on-user-consent-for-public-usage"></a>¿Cómo se pueden sincronizar los números de móvil desde Workday según el consentimiento del usuario para su uso público?

* Vaya a la hoja "Aprovisionamiento" de la aplicación de aprovisionamiento de Workday.
* Haga clic en las asignaciones de atributos. 
* En **Asignaciones**, seleccione **Synchronize Workday Workers to On Premises Active Directory** (Sincronizar trabajadores de Workday con Active Directory local) o **Synchronize Workday Workers to Azure AD** (Sincronizar trabajadores de Workday con Azure AD).
* En la página Asignaciones de atributos, desplácese hacia abajo y active la casilla "Mostrar opciones avanzadas".  Haga clic en **Edit attribute list for Workday** (Editar lista de atributos para Workday).
* En la hoja que se abre, busque el atributo "Mobile" y haga clic en la fila para poder editar la **Expresión de API** ![RGPD para dispositivos móviles](./media/workday-inbound-tutorial/mobile_gdpr.png)

* Reemplace la **Expresión de API** con la siguiente expresión nueva, que recupera el número de teléfono móvil profesional solo si la "Marca de uso público" está establecida en "True" en Workday.

    ```
     wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Contact_Data/wd:Phone_Data[translate(string(wd:Phone_Device_Type_Reference/@wd:Descriptor),'abcdefghijklmnopqrstuvwxyz','ABCDEFGHIJKLMNOPQRSTUVWXYZ')='MOBILE' and translate(string(wd:Usage_Data/wd:Type_Data/wd:Type_Reference/@wd:Descriptor),'abcdefghijklmnopqrstuvwxyz','ABCDEFGHIJKLMNOPQRSTUVWXYZ')='WORK' and string(wd:Usage_Data/@wd:Public)='1']/@wd:Formatted_Phone
    ```

* Guarde la lista de atributos.
* Guarde la asignación de atributos.
* Borre el estado actual y reinicie la sincronización completa.

#### <a name="how-do-i-format-display-names-in-ad-based-on-the-users-departmentcountrycity-attributes-and-handle-regional-variances"></a>¿Cómo puedo dar formato a los nombres para mostrar en AD según los atributos de departamento/país/ciudad del usuario y administrar las variaciones regionales?

Es un requisito común configurar el atributo *displayName* de AD para que también proporcione información sobre el departamento y el país o región del usuario. Por ejemplo, si John Smith trabaja en el departamento de marketing en Estados Unidos, puede que desee que su atributo *displayName* aparezca como *Smith, John (Marketing-EE. UU.)* .

Esta es la forma en que puede controlar tales requisitos para construir *CN* o *displayName* a fin de incluir atributos como empresa, unidad de negocio, ciudad o país o región.

* Cada atributo de Workday se recupera mediante una expresión de API XPATH subyacente, que se puede configurar en **Asignación de atributos -> Sección avanzada -> Editar lista de atributos para Workday**. Esta es la expresión de API XPATH predeterminada para los atributos de Workday *PreferredFirstName*, *PreferredLastName*, *Company* y *SupervisoryOrganization*.

     | Atributo de Workday | Expresión de API XPATH |
     | ----------------- | -------------------- |
     | PreferredFirstName | wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Name_Data/wd:Preferred_Name_Data/wd:Name_Detail_Data/wd:First_Name/text() |
     | PreferredLastName | wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Name_Data/wd:Preferred_Name_Data/wd:Name_Detail_Data/wd:Last_Name/text() |
     | Compañía | wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data[wd:Organization_Data/wd:Organization_Type_Reference/wd:ID[@wd:type='Organization_Type_ID']='Company']/wd:Organization_Reference/@wd:Descriptor |
     | SupervisoryOrganization | wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data/wd:Organization_Data[wd:Organization_Type_Reference/wd:ID[@wd:type='Organization_Type_ID']='Supervisory']/wd:Organization_Name/text() |

   Confirme con su equipo de Workday que la expresión de API anterior es válida para la configuración de su inquilino de Workday. Si es necesario, puede modificar los atributos como se describe en la sección [Personalización de la lista de atributos de usuario de Workday](#customizing-the-list-of-workday-user-attributes).

* Del mismo modo, la información de país o región existente en Workday se recupera con la siguiente expresión XPATH: *wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference*.

     Hay cinco atributos relacionados con el país o región que se encuentran disponibles en la sección de la lista de atributos de Workday.

     | Atributo de Workday | Expresión de API XPATH |
     | ----------------- | -------------------- |
     | CountryReference | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Alpha-3_Code']/text() |
     | CountryReferenceFriendly | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/@wd:Descriptor |
     | CountryReferenceNumeric | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Numeric-3_Code']/text() |
     | CountryReferenceTwoLetter | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Reference/wd:ID[@wd:type='ISO_3166-1_Alpha-2_Code']/text() |
     | CountryRegionReference | wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Address_Data/wd:Country_Region_Reference/@wd:Descriptor |

  Confirme con su equipo de Workday que las expresiones de API anteriores son válidas para la configuración de su inquilino de Workday. Si es necesario, puede modificar los atributos como se describe en la sección [Personalización de la lista de atributos de usuario de Workday](#customizing-the-list-of-workday-user-attributes).

* Para compilar la expresión de asignación de atributos correcta, identifique qué atributo de Workday representa "de forma confiable" el nombre, apellido, país o región y departamento del usuario. Supongamos que los atributos son *PreferredFirstName*, *PreferredLastName*, *CountryReferenceTwoLetter* y *SupervisoryOrganization*, respectivamente. Puede usar esto para compilar una expresión para el atributo *displayName* de AD como se indica a continuación para obtener un nombre para mostrar como *Smith, John (Marketing-EE. UU.)* .

    ```
     Append(Join(", ",[PreferredLastName],[PreferredFirstName]), Join(""," (",[SupervisoryOrganization],"-",[CountryReferenceTwoLetter],")"))
    ```
    Una vez obtenida la expresión apropiada, edite la tabla Asignaciones de atributos y modifique la asignación de atributos *displayName* como se muestra a continuación:   ![Asignación de DisplayName](./media/workday-inbound-tutorial/wd_displayname_map.png)

* Para ampliar el ejemplo anterior, supongamos que desea convertir los nombres de ciudad de Workday en valores abreviados y usarlos para crear nombres para mostrar como *Smith, John (CHI)* o *Doe, Jane (NYC)* ; este resultado se puede conseguir con una expresión Switch mediante el atributo *Municipalidad* de Workday como la variable determinante.

     ```
    Switch
    (
      [Municipality],
      Join(", ", [PreferredLastName], [PreferredFirstName]),  
           "Chicago", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(CHI)"),
           "New York", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(NYC)"),
           "Phoenix", Append(Join(", ",[PreferredLastName], [PreferredFirstName]), "(PHX)")
    )
     ```
    Consulte también:
  * [Sintaxis de la función Switch](../app-provisioning/functions-for-customizing-application-data.md#switch)
  * [Sintaxis de la función Join](../app-provisioning/functions-for-customizing-application-data.md#join)
  * [Sintaxis de la función Append](../app-provisioning/functions-for-customizing-application-data.md#append)

#### <a name="how-can-i-use-selectuniquevalue-to-generate-unique-values-for-samaccountname-attribute"></a>¿Cómo se puede usar SelectUniqueValue para generar valores únicos para el atributo samAccountName?

Supongamos que desea generar valores únicos para el atributo *samAccountName* mediante una combinación de los atributos *FirstName* y *LastName* de Workday. A continuación se presenta una expresión con la que puede empezar:

```
SelectUniqueValue(
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,1), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , ),
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,2), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , ),
    Replace(Mid(Replace(NormalizeDiacritics(StripSpaces(Join("",  Mid([FirstName],1,3), [LastName]))), , "([\\/\\\\\\[\\]\\:\\;\\|\\=\\,\\+\\*\\?\\<\\>])", , "", , ), 1, 20), , "(\\.)*$", , "", , )
)
```

Cómo funciona la expresión anterior: si el usuario es John Smith, primero intenta generar JSmith; si ya existe JSmith, entonces genera JoSmith, pero si también existe, genera JohSmith. La expresión también garantiza que el valor generado cumple las restricciones de longitud y la restricción de caracteres especiales asociadas con *samAccountName*.

Consulte también:

* [Sintaxis de la función Mid](../app-provisioning/functions-for-customizing-application-data.md#mid)
* [Sintaxis de la función Replace](../app-provisioning/functions-for-customizing-application-data.md#replace)
* [Sintaxis de la función SelectUniqueValue](../app-provisioning/functions-for-customizing-application-data.md#selectuniquevalue)

#### <a name="how-do-i-remove-characters-with-diacritics-and-convert-them-into-normal-english-alphabets"></a>¿Cómo se pueden quitar caracteres con signos diacríticos y convertirlos en caracteres habituales del alfabeto inglés?

Use la función [NormalizeDiacritics](../app-provisioning/functions-for-customizing-application-data.md#normalizediacritics) para quitar caracteres especiales en el nombre y apellido del usuario, mientras crea la dirección de correo electrónico o el valor CN para el usuario.

## <a name="troubleshooting-tips"></a>Sugerencias de solución de problemas

En esta sección se proporcionan instrucciones específicas sobre cómo solucionar problemas de aprovisionamiento con la integración de Workday mediante los registros de auditoría de Azure AD y los registros del Visor de eventos de Windows Server. Se basa en los conceptos y pasos de solución de problemas genéricos de [Tutorial: Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](../app-provisioning/check-status-user-account-provisioning.md).

En esta sección se tratan los siguientes aspectos sobre la solución de problemas:

* [Tutorial: Configuración de Workday para el aprovisionamiento automático de usuarios](#configure-provisioning-agent-to-emit-event-viewer-logs)
* [Configuración del Visor de eventos de Windows para solucionar problemas del agente](#setting-up-windows-event-viewer-for-agent-troubleshooting)
* [Configuración de registros de auditoría de Azure Portal para solucionar problemas con el servicio](#setting-up-azure-portal-audit-logs-for-service-troubleshooting)
* [Información sobre registros para las operaciones de creación de cuentas de usuario de AD](#understanding-logs-for-ad-user-account-create-operations)
* [Información sobre registros para las operaciones de actualización del administrador](#understanding-logs-for-manager-update-operations)
* [Resolución de errores comunes](#resolving-commonly-encountered-errors)

### <a name="configure-provisioning-agent-to-emit-event-viewer-logs"></a>Configuración del agente de aprovisionamiento para emitir registros de Visor de eventos
1. Inicie sesión en la máquina de Windows Server en la que se ha implementado el agente de aprovisionamiento.
1. Detenga el servicio **Agente de aprovisionamiento de Microsoft Azure AD Connect**.
1. Cree una copia del archivo de configuración original: *C:\Archivos de programa\Microsoft Azure AD Connect Provisioning Agent\AADConnectProvisioningAgent.exe.config*.
1. Reemplace la sección `<system.diagnostics>` existente por lo siguiente: 
   * La configuración del cliente de escucha **etw** emite mensajes a los registros de EventViewer.
   * La configuración del cliente de escucha **textWriterListener** envía mensajes de seguimiento al archivo *ProvAgentTrace.log*. Quite las marcas de comentario de las líneas relacionadas con textWriterListener solo para la solución de problemas avanzada. 

   ```xml
     <system.diagnostics>
         <sources>
         <source name="AAD Connect Provisioning Agent">
             <listeners>
             <add name="console"/>
             <add name="etw"/>
             <!-- <add name="textWriterListener"/> -->
             </listeners>
         </source>
         </sources>
         <sharedListeners>
         <add name="console" type="System.Diagnostics.ConsoleTraceListener" initializeData="false"/>
         <add name="etw" type="System.Diagnostics.EventLogTraceListener" initializeData="Azure AD Connect Provisioning Agent">
             <filter type="System.Diagnostics.EventTypeFilter" initializeData="All"/>
         </add>
         <!-- <add name="textWriterListener" type="System.Diagnostics.TextWriterTraceListener" initializeData="C:/ProgramData/Microsoft/Azure AD Connect Provisioning Agent/Trace/ProvAgentTrace.log"/> -->
         </sharedListeners>
     </system.diagnostics>

   ```
1. Inicie el servicio **Agente de aprovisionamiento de Microsoft Azure AD Connect**.

### <a name="setting-up-windows-event-viewer-for-agent-troubleshooting"></a>Configuración del Visor de eventos de Windows para solucionar problemas del agente

1. Inicie sesión en el equipo Windows Server en el que se ha implementado el agente de aprovisionamiento.
1. Abra la aplicación de escritorio **Visor de eventos de Windows Server**.
1. Seleccione **Registros de Windows > Aplicación**.
1. Use la opción **Filtrar registro actual...** para ver todos los eventos registrados en el origen del **agente de aprovisionamiento de Azure AD Connect** y excluir eventos con el identificador "5", mediante la especificación del filtro "-5", tal como se muestra a continuación.
   > [!NOTE]
   > El identificador de evento 5 captura los mensajes de arranque del agente en el servicio en la nube de Azure AD y, por tanto, se filtran al analizar los archivos de registro. 

   ![Visor de eventos de Windows](media/workday-inbound-tutorial/wd_event_viewer_01.png)

1. Haga clic en **Aceptar** y ordene la vista resultados según la columna **Fecha y hora**.

### <a name="setting-up-azure-portal-audit-logs-for-service-troubleshooting"></a>Configuración de registros de auditoría de Azure Portal para solucionar problemas con el servicio

1. Inicie [Azure Portal](https://portal.azure.com) y vaya a la sección **Registros de auditoría** de la aplicación de aprovisionamiento de Workday.
1. Use el botón **Columnas** de la página Registros de auditoría para mostrar solo las siguientes columnas en la vista (Fecha, Actividad, Estado y Motivo del estado). Esta configuración garantiza que pueda centrarse solo en los datos que sean pertinentes para solucionar problemas.

   ![Columnas del registro de auditoría](media/workday-inbound-tutorial/wd_audit_logs_00.png)

1. Use los parámetros de consulta **Destino** e **Intervalo de fechas** para filtrar la vista. 
   * Establezca el parámetro de consulta **Destino** con el "Id. del trabajador" o el "Id. de empleado" del objeto de trabajador de Workday.
   * Establezca el parámetro **Intervalo de fechas** en un período de tiempo apropiado durante el cual desea investigar los problemas o errores de aprovisionamiento.

   ![Filtros del registro de auditoría](media/workday-inbound-tutorial/wd_audit_logs_01.png)

### <a name="understanding-logs-for-ad-user-account-create-operations"></a>Información sobre registros para las operaciones de creación de cuentas de usuario de AD

Cuando se detecta un nuevo empleado en Workday (supongamos que con el identificador de empleado *21023*), el servicio de aprovisionamiento de Azure AD intenta crear una cuenta de usuario de AD para el empleado y, en el proceso, crea cuatro registros de auditoría como se describe a continuación:

  [![Operaciones de creación del registro de auditoría](media/workday-inbound-tutorial/wd_audit_logs_02.png)](media/workday-inbound-tutorial/wd_audit_logs_02.png#lightbox)

Al hacer clic en cualquiera de los registros de auditoría, se abre la página **Detalles de la actividad**. Esto es lo que muestra la página **Detalles de la actividad** para cada tipo de registro de auditoría.

* Registro **Workday Import**: este registro muestra la información del trabajador capturada de Workday. Use la información de la sección *Detalles adicionales* del registro para solucionar problemas relativos a la captura de datos de Workday. A continuación se muestra un registro de ejemplo junto con indicaciones sobre cómo interpretar cada campo.

  ```JSON
  ErrorCode : None  // Use the error code captured here to troubleshoot Workday issues
  EventName : EntryImportAdd // For full sync, value is "EntryImportAdd" and for delta sync, value is "EntryImport"
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID (usually the Worker ID or Employee ID field)
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the record
  ```

* Registro **AD Import**: este registro muestra información sobre la cuenta capturada de AD. Como durante la creación del usuario inicial no hay ninguna cuenta de AD, en *Activity Status Reason* (Motivo del estado de actividad) se indicará que no se encontró ninguna cuenta con el atributo de identificador de coincidencia en Active Directory. Use la información de la sección *Detalles adicionales* del registro para solucionar problemas relativos a la captura de datos de Workday. A continuación se muestra un registro de ejemplo junto con indicaciones sobre cómo interpretar cada campo.

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot Workday issues
  EventName : EntryImportObjectNotFound // Implies that object was not found in AD
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  ```

  Para buscar registros del agente de aprovisionamiento correspondientes a esta operación de importación de AD, abra los registros del Visor de eventos de Windows y use la opción de menú **Buscar…** para encontrar las entradas de registro que contengan el valor de atributo Id. de coincidencia/Propiedad de combinación (en este caso, *21023*).

  ![Buscar](media/workday-inbound-tutorial/wd_event_viewer_02.png)

  Busque la entrada con *Id. de evento = 9*, que proporcionará el filtro de búsqueda de LDAP utilizado por el agente para recuperar la cuenta de AD. Puede comprobar si este es el filtro de búsqueda adecuado para recuperar entradas de usuario únicas.

  ![Búsqueda LDAP](media/workday-inbound-tutorial/wd_event_viewer_03.png)

  El registro que le sigue inmediatamente con *Id. de evento = 2* captura el resultado de la operación de búsqueda y si devuelve algún resultado.

  ![Resultados LDAP](media/workday-inbound-tutorial/wd_event_viewer_04.png)

* Registro **Synchronization rule action**: este registro muestra los resultados de las reglas de asignación de atributos y los filtros de ámbito configurados junto con la acción de aprovisionamiento que se usará para procesar el evento de Workday de entrada. Use la información de la sección *Detalles adicionales* del registro para solucionar problemas relativos a la acción de sincronización. A continuación se muestra un registro de ejemplo junto con indicaciones sobre cómo interpretar cada campo.

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot sync issues
  EventName : EntrySynchronizationAdd // Implies that the object will be added
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the profile in Workday
  ```

  Si hay problemas con las expresiones de asignación de atributos o los datos de Workday de entrada presentan errores (por ejemplo, valores vacíos o NULL para los atributos obligatorios), aparecerá un error en esta fase con el texto ErrorCode en el que se proporcionarán los detalles del error.

* Registro **AD Export**: este registro muestra el resultado de la operación de creación de la cuenta de AD junto con los valores de atributo definidos en el proceso. Use la información de la sección *Detalles adicionales* del registro para solucionar problemas relativos a operación de creación de la cuenta. A continuación se muestra un registro de ejemplo junto con indicaciones sobre cómo interpretar cada campo. En la sección "Detalles adicionales", "EventName" se establece en "EntryExportAdd", "JoiningProperty" se establece en el valor del atributo Id. de coincidencia, "SourceAnchor" se establece en el valor WorkdayID (WID) asociado con el registro y "TargetAnchor" se establece en el valor del atributo "ObjectGuid" de AD del usuario recién creado. 

  ```JSON
  ErrorCode : None // Use the error code captured here to troubleshoot AD account creation issues
  EventName : EntryExportAdd // Implies that object will be created
  JoiningProperty : 21023 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : a071861412de4c2486eb10e5ae0834c3 // set to the WorkdayID (WID) associated with the profile in Workday
  TargetAnchor : 83f0156c-3222-407e-939c-56677831d525 // set to the value of the AD "objectGuid" attribute of the new user
  ```

  Para buscar registros del agente de aprovisionamiento correspondientes a esta operación de exportación de AD, abra los registros del Visor de eventos de Windows y use la opción de menú **Encontrar…** para encontrar las entradas de registro que contengan el valor de atributo Id. de coincidencia/Propiedad de combinación (en este caso, *21023*).  

  Busque un registro POST HTTP correspondiente a la marca de tiempo de la operación de exportación con el *Id. de evento = 2*. Este registro contendrá los valores de atributo enviados por el servicio de aprovisionamiento al agente de aprovisionamiento.

  :::image type="content" source="media/workday-inbound-tutorial/wd_event_viewer_05.png" alt-text="Captura de pantalla que muestra el registro &quot;HTTP POST&quot; en el registro del agente de aprovisionamiento." lightbox="media/workday-inbound-tutorial/wd_event_viewer_05.png":::

  Inmediatamente después del evento anterior, debe haber otro evento que captura la respuesta de la operación de creación de la cuenta de AD. Este evento devuelve el nuevo valor objectGuid creado en AD y se establece como el atributo TargetAnchor en el servicio de aprovisionamiento.

  :::image type="content" source="media/workday-inbound-tutorial/wd_event_viewer_06.png" alt-text="Captura de pantalla que muestra el registro del agente de aprovisionamiento con el elemento objectGuid creado en AD resaltado." lightbox="media/workday-inbound-tutorial/wd_event_viewer_06.png":::

### <a name="understanding-logs-for-manager-update-operations"></a>Información sobre registros para las operaciones de actualización del administrador

El atributo manager es un atributo de referencia en AD. El servicio de aprovisionamiento no establece el atributo manager como parte de la operación de creación de un usuario. En su lugar, el atributo manager se establece como parte de una operación de *actualización* después de que se haya creado la cuenta de AD para el usuario. Como ampliación del ejemplo anterior, supongamos que se activa un nuevo trabajador con el identificador de empleado "21451" en Workday y que el administrador del nuevo trabajador (*21023*) ya tiene una cuenta de AD. En este escenario, al buscar los registros de auditoría para el usuario 21451, se muestran cinco entradas.

  [![Actualización del administrador](media/workday-inbound-tutorial/wd_audit_logs_03.png)](media/workday-inbound-tutorial/wd_audit_logs_03.png#lightbox)

Los cuatro primeros registros son similares a los que hemos explorado como parte de la operación de creación del usuario. El quinto registro se corresponde con la exportación asociada con la actualización del atributo manager. El registro muestra el resultado de la operación de actualización del administrador de la cuenta de AD, que se realiza con el atributo *objectGuid* del administrador.

  ```JSON
  // Modified Properties
  Name : manager
  New Value : "83f0156c-3222-407e-939c-56677831d525" // objectGuid of the user 21023

  // Additional Details
  ErrorCode : None // Use the error code captured here to troubleshoot AD account creation issues
  EventName : EntryExportUpdate // Implies that object will be created
  JoiningProperty : 21451 // Value of the Workday attribute that serves as the Matching ID
  SourceAnchor : 9603bf594b9901693f307815bf21870a // WorkdayID of the user
  TargetAnchor : 43b668e7-1d73-401c-a00a-fed14d31a1a8 // objectGuid of the user 21451

  ```

### <a name="resolving-commonly-encountered-errors"></a>Resolución de errores comunes

Esta sección trata los errores habituales con el aprovisionamiento de usuarios de Workday y cómo resolverlos. Los errores se agrupan como sigue:

* [Errores del agente de aprovisionamiento](#provisioning-agent-errors)
* [Errores de conectividad](#connectivity-errors)
* [Errores de creación de cuentas de usuario de AD](#ad-user-account-creation-errors)
* [Errores de actualización de cuentas de usuario de AD](#ad-user-account-update-errors)

#### <a name="provisioning-agent-errors"></a>Errores del agente de aprovisionamiento

|#|Escenario de error |Causas probables|Resolución recomendada|
|--|---|---|---|
|1.| Error al instalar al agente de aprovisionamiento con el mensaje de error:  *No se pudo iniciar el servicio "Microsoft Azure AD Connect Provisioning Agent" (AADConnectProvisioningAgent). Compruebe que tiene suficientes privilegios para iniciar el sistema*. | Este error normalmente aparece si intenta instalar al agente de aprovisionamiento en un controlador de dominio y una directiva de grupo impide que el servicio se inicie.  También aparece si tiene una versión anterior del agente en ejecución y no la ha desinstalado antes de iniciar una nueva instalación.| Instale el agente de aprovisionamiento en un servidor que no sea controlador de dominio. Asegúrese de que las versiones anteriores del agente están desinstaladas antes de instalar el nuevo agente.|
|2.| El servicio de Windows "Microsoft Azure AD Connect Provisioning Agent" está en estado de *inicio* y no cambia al estado *En ejecución*. | Como parte de la instalación, el asistente del agente crea una cuenta local (**Servicio NT\\AADConnectProvisioningAgent**) en el servidor, que es la cuenta de inicio de sesión utilizada para iniciar el servicio. Si una directiva de seguridad del servidor de Windows impide que las cuentas locales ejecuten los servicios, se producirá este error. | Abra la *Consola de Servicios*. Haga clic con el botón derecho en el servicio de Windows "Microsoft Azure AD Connect Provisioning Agent" y, en la pestaña de inicio de sesión, especifique la cuenta de un administrador de dominio para ejecutar el servicio. Reinicie el servicio. |
|3.| Al configurar el agente de aprovisionamiento con el dominio de AD en el paso *Conectar a Active Directory*, el asistente tarda mucho tiempo intentando cargar el esquema de AD y finalmente se agota el tiempo de espera. | Este error normalmente se muestra si el asistente no puede ponerse en contacto con el servidor de controlador de dominio de AD debido a problemas de firewall. | En la pantalla del asistente *Conectar a Active Directory*, al proporcionar las credenciales del dominio de AD, hay una opción denominada *Select domain controller priority* (Seleccionar prioridad del controlador de dominio). Use esta opción para seleccionar un controlador de dominio que esté en el mismo sitio que el servidor del agente y asegúrese de que no haya ninguna regla de firewall que bloquee la comunicación. |

#### <a name="connectivity-errors"></a>Errores de conectividad

Si el servicio de aprovisionamiento no puede conectarse a Active Directory o Workday, podría provocar que el aprovisionamiento entre en estado de cuarentena. Use la tabla siguiente para solucionar los problemas de conectividad.

|#|Escenario de error |Causas probables|Resolución recomendada|
|--|---|---|---|
|1.| Al hacer clic en **Probar conexión**, obtendrá el mensaje de error: *Error al conectarse a Active Directory. Asegúrese de que está ejecutando el agente de aprovisionamiento local y de que está configurado con el dominio de Active Directory correcto*. | Este error suele aparecer si el agente de aprovisionamiento no está en ejecución o si hay un firewall que bloquea la comunicación entre Azure AD y el agente de aprovisionamiento. Este error también puede aparecer si el dominio no está configurado en el asistente del agente. | Abra la consola *Servicios* en el servidor de Windows para confirmar que el agente está en ejecución. Abra el asistente del agente de aprovisionamiento y confirme que el dominio correcto está registrado en el agente.  |
|2.| El trabajo de aprovisionamiento entra en estado de cuarentena los fines de semana (viernes y sábado) y recibimos una notificación por correo electrónico de que existe un error con la sincronización. | Una de las causas comunes de este error es el tiempo de inactividad planeado de Workday. Si usa un inquilino de implementación de Workday, tenga en cuenta que Workday programó un tiempo de inactividad para sus inquilinos de implementación durante los fines de semana (normalmente desde el viernes por la noche hasta el sábado por la mañana) y, durante dicho período, las aplicaciones de aprovisionamiento de Workday pueden entrar en estado de cuarentena porque no pueden conectarse a Workday. Vuelve a su estado habitual una vez que el inquilino de implementación de Workday puede volver a conectarse. En raras ocasiones, puede que este error también aparezca si la contraseña del usuario del sistema de integración cambió debido a la actualización del inquilino o si la cuenta tiene un estado bloqueado o expirado. | Póngase en contacto con el asociado de integración o el administrador de Workday para saber cuándo Workday tiene programados tiempos de inactividad, para ignorar los mensajes de alerta durante dicho período y confirmar la disponibilidad una vez que la instancia de Workday vuelva a recuperar la conexión.  |

#### <a name="ad-user-account-creation-errors"></a>Errores de creación de cuentas de usuario de AD

|#|Escenario de error |Causas probables|Resolución recomendada|
|--|---|---|---|
|1.| Errores en la operación de exportación en el registro de auditoría con el mensaje *Error: OperationsError-SvcErr: Error en la operación. No se configuró ninguna referencia superior para el servicio de directorio. Por tanto, el servicio de directorio no puede emitir referencias a los objetos fuera de este bosque.* | Este error suele aparecer si la unidad organizativa del *Contenedor de Active Directory* no está configurada correctamente o si hay problemas con la asignación de expresiones utilizada para *parentDistinguishedName*. | Compruebe si hay algún error ortográfico en el parámetro de la unidad organizativa del *Contenedor de Active Directory*. Si usa *parentDistinguishedName* en la asignación de atributos, asegúrese de que siempre se evalúa en un contenedor conocido dentro del dominio de AD. Consulte el evento *Exportar* en los registros de auditoría para ver el valor generado. |
|2.| Errores en la operación de exportación en el registro de auditoría con el código de error: *SystemForCrossDomainIdentityManagementBadResponse* y el mensaje *Error: ConstraintViolation-AtrErr: Un valor de la solicitud no es válido. Un valor del atributo no estaba en el rango de valores aceptables. \nDetalles del error: CONSTRAINT_ATT_TYPE - company*. | Aunque este error es específico del atributo *company*, puede que también vea este error para otros atributos como *CN*. Este error aparece debido a una restricción de esquema aplicada en AD. De forma predeterminada, los atributos como *company* y *CN* de AD tienen un límite máximo de 64 caracteres. Si el valor procedente de Workday tiene más de 64 caracteres, verá este mensaje de error. | Consulte el evento *Exportar* en los registros de auditoría para ver el valor del atributo indicado en el mensaje de error. Considere la posibilidad de truncar el valor procedente de Workday mediante la función [Mid](../app-provisioning/functions-for-customizing-application-data.md#mid) o con el cambio de las asignaciones por un atributo de AD que no tenga restricciones de longitud similares.  |

#### <a name="ad-user-account-update-errors"></a>Errores de actualización de cuentas de usuario de AD

Durante el proceso de actualización de cuentas de usuario de AD, el servicio de aprovisionamiento lee la información de Workday y AD, ejecuta las reglas de asignación de atributos y determina si es necesario aplicar algún cambio. En consecuencia, se desencadena un evento de actualización. Si se presenta algún error con cualquiera de estos pasos, este se refleja en los registros de auditoría. Use la tabla siguiente para solucionar errores de actualización comunes.

|#|Escenario de error |Causas probables|Resolución recomendada|
|--|---|---|---|
|1.| Errores de la acción de las reglas de sincronización en el registro de auditoría con el mensaje *EventName = EntrySynchronizationError and ErrorCode = EndpointUnavailable*. | Este error aparece si el servicio de aprovisionamiento no puede recuperar datos del perfil de usuario de Active Directory debido a un error de procesamiento detectado por el agente de aprovisionamiento local. | Consulte los registros del Visor de eventos del agente de aprovisionamiento para ver si existen eventos de error que indiquen problemas con la operación de lectura (Filtrar por id. de evento n.º 2). |
|2.| El atributo manager de AD no se actualiza para determinados usuarios de AD. | La causa más probable de este error es que usa reglas de ámbito y el administrador del usuario no forma parte del ámbito. Este problema también puede darse si el atributo de identificador de coincidencia del administrador (por ejemplo, EmployeeID) no se encuentra en el dominio de AD de destino o no está establecido en el valor correcto. | Revise el filtro de ámbito y agregue el usuario administrador al ámbito. Compruebe el perfil de administrador de AD para asegurarse de que haya un valor para el atributo de identificador de coincidencia. |

## <a name="managing-your-configuration"></a>Administración de la configuración

En esta sección se describe cómo puede ampliar, personalizar y administrar la configuración de aprovisionamiento de usuarios controlada por Workday. Se tratan los temas siguientes:

* [Personalización de la lista de atributos de usuario de Workday](#customizing-the-list-of-workday-user-attributes)  
* [Exportación e importación de la configuración](#exporting-and-importing-your-configuration)

### <a name="customizing-the-list-of-workday-user-attributes"></a>Personalización de la lista de atributos de usuario de Workday

Las aplicaciones de aprovisionamiento de Workday para Active Directory y Azure AD incluyen ambas una lista predeterminada de atributos de usuario de Workday que puede seleccionar. Sin embargo, estas listas no son exhaustivas. Workday admite muchos cientos de posibles atributos de usuario, que pueden ser estándar o exclusivos de su inquilino de Workday.

El servicio de aprovisionamiento de Azure AD admite la posibilidad de personalizar la lista o los atributos de Workday para incluir cualquiera de los atributos expuestos en la operación [Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v21.1/Get_Workers.html) de la API de Recursos Humanos.

Para realizar este cambio, debe usar [Workday Studio](https://community.workday.com/studio-download) para extraer las expresiones XPath que representan los atributos que quiere usar y, luego, agregarlas a la configuración de aprovisionamiento mediante el editor de atributos avanzados de Azure Portal.

**Para recuperar una expresión XPath para un atributo de usuario de Workday, siga estos pasos:**

1. Descargue e instale [Workday Studio](https://community.workday.com/studio-download). Necesitará una cuenta de la comunidad de Workday para acceder al instalador.

2. Descargue el archivo WSDL de Workday denominado **Human_Resources** específico para la versión de la API de WWS que va a usar desde el [directorio de servicios web de Workday](https://community.workday.com/sites/default/files/file-hosting/productionapi/index.html).

3. Inicie Workday Studio.

4. En la barra de comandos, seleccione la opción **Workday > Test Web Service in Tester** (Probar servicio web en Evaluador).

5. Seleccione **External** (Externo) y seleccione el archivo WSDL Human_Resources que descargó en el paso 2.

    ![Captura de pantalla que muestra el archivo "Human_Resources" abierto en Workday Studio.](./media/workday-inbound-tutorial/wdstudio1.png)

6. Establezca el campo **Location** (Ubicación) en `https://IMPL-CC.workday.com/ccx/service/TENANT/Human_Resources`, pero reemplace "IMPL-CC" por el tipo de instancia real y "TENANT" por el nombre real del inquilino.

7. Establezca **Operation** (Operación) en **Get_Workers**.

8.    Haga clic en el vínculo pequeño de **configuración** bajo los paneles de solicitud/respuesta para establecer las credenciales de Workday. Marque **Authentication** (Autenticación) y escriba el nombre de usuario y la contraseña de la cuenta del sistema de integración de Workday. Asegúrese de que el formato del nombre de usuario sea name\@tenant y deje seleccionada la opción **WS-Security UsernameToken**.
   ![Captura de pantalla que muestra la pestaña "Seguridad" con el "Nombre de usuario" y la "Contraseña" especificados y "Token de nombre de usuario de WS-Security" seleccionado.](./media/workday-inbound-tutorial/wdstudio2.png)

9. Seleccione **Aceptar**.

10. En el panel **Request** (Solicitud), pegue el código XML siguiente. Establezca como **Employee_ID** el id. de empleado de un usuario real en el inquilino de Workday. Establezca **wd:version** en la versión de WWS que va a usar. Seleccione un usuario que tenga rellenado el atributo que desea extraer.

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <env:Envelope xmlns:env="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsd="https://www.w3.org/2001/XMLSchema">
      <env:Body>
        <wd:Get_Workers_Request xmlns:wd="urn:com.workday/bsvc" wd:version="v21.1">
          <wd:Request_References wd:Skip_Non_Existing_Instances="true">
            <wd:Worker_Reference>
              <wd:ID wd:type="Employee_ID">21008</wd:ID>
            </wd:Worker_Reference>
          </wd:Request_References>
          <wd:Response_Group>
            <wd:Include_Reference>true</wd:Include_Reference>
            <wd:Include_Personal_Information>true</wd:Include_Personal_Information>
            <wd:Include_Employment_Information>true</wd:Include_Employment_Information>
            <wd:Include_Management_Chain_Data>true</wd:Include_Management_Chain_Data>
            <wd:Include_Organizations>true</wd:Include_Organizations>
            <wd:Include_Reference>true</wd:Include_Reference>
            <wd:Include_Transaction_Log_Data>true</wd:Include_Transaction_Log_Data>
            <wd:Include_Photo>true</wd:Include_Photo>
            <wd:Include_User_Account>true</wd:Include_User_Account>
          <wd:Include_Roles>true</wd:Include_Roles>
          </wd:Response_Group>
        </wd:Get_Workers_Request>
      </env:Body>
    </env:Envelope>
    ```

11. Haga clic en **Send Request** (Enviar solicitud) (flecha verde) para ejecutar el comando. Si se ejecuta correctamente, la respuesta debe aparecer en el panel **Response** (Respuesta). Compruebe la respuesta para asegurarse de que tiene los datos del identificador de usuario especificado y no un error.

12. Si es correcta, copie el código XML del panel **Response** (Respuesta) y guárdelo como un archivo XML.

13. En la barra de comandos de Workday Studio, seleccione **File > Open File...** (Archivo > Abrir archivo...) y abra el archivo XML que ha guardado. Esta acción abrirá el archivo en el editor XML de Workday Studio.

    ![Captura de pantalla de un archivo XML abierto en el editor XML de Workday Studio.](./media/workday-inbound-tutorial/wdstudio3.png)

14. En el árbol de archivos, desplácese a través de **/env: Envelope > env: Body > wd:Get_Workers_Response > wd:Response_Data > wd: Worker** para encontrar los datos del usuario.

15. En **wd: Worker**, busque el atributo que quiere agregar y selecciónelo.

16. Copie la expresión XPath del atributo seleccionado del campo **Document Path** (Ruta de acceso a documento).

17. Quite el prefijo **/env:Envelope/env:Body/wd:Get_Workers_Response/wd:Response_Data/** de la expresión copiada.

18. Si el último elemento de la expresión copiada es un nodo (ejemplo: "/ wd: Birth_Date"), anexe **/text()** al final de la expresión. Esto no es necesario si el último elemento es un atributo (ejemplo: "/@wd: tipo").

19. El resultado debe ser similar a `wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Birth_Date/text()`. Este valor es el que copiará en Azure Portal.

**Para agregar el atributo de usuario personalizado de Workday a la configuración de aprovisionamiento, siga estos pasos:**

1. Inicie [Azure Portal](https://portal.azure.com) y vaya a la sección Aprovisionamiento de la aplicación de aprovisionamiento de Workday, como se ha descrito anteriormente en este tutorial.

2. Establezca **Estado de aprovisionamiento** en **Desactivado** y seleccione **Guardar**. Este paso le ayudará a garantizar que los cambios se apliquen solo cuando esté listo.

3. En **Asignaciones**, seleccione **Synchronize Workday Workers to On Premises Active Directory** (Sincronizar trabajadores de Workday con Active Directory local) o **Synchronize Workday Workers to Azure AD** (Sincronizar trabajadores de Workday con Azure AD).

4. Desplácese a la parte inferior de la pantalla siguiente y seleccione **Mostrar opciones avanzadas**.

5. Seleccione **Edit attribute list for Workday** (Editar lista de atributos para Workday).

    ![Captura de pantalla que muestra la página "Aprovisionamiento de usuarios de Workday en Azure AD - Aprovisionamiento" con la acción "Editar lista de atributos para Workday" resaltada.](./media/workday-inbound-tutorial/wdstudio_aad1.png)

6. Desplácese a la parte inferior de la lista de atributos, al lugar donde están los campos de entrada.

7. En **Name** (Name), escriba un nombre para mostrar para el atributo.

8. En **Type** (Tipo), seleccione el tipo que corresponde adecuadamente a su atributo (el más común es **String** [Cadena]).

9. En **API Expression** (Expresión de API), escriba la expresión XPath que copió de Workday Studio. Ejemplo: `wd:Worker/wd:Worker_Data/wd:Personal_Data/wd:Birth_Date/text()`

10. Seleccione **Add Attribute** (Agregar atributo).

    ![Workday Studio](./media/workday-inbound-tutorial/wdstudio_aad2.png)

11. Seleccione **Save** (Guardar) arriba y, luego, **Yes** (Sí) en el cuadro de diálogo. Cierre la pantalla de asignación de atributos si aún está abierta.

12. De nuevo en la pestaña principal **Aprovisionamiento**, seleccione otra vez **Synchronize Workday Workers to On Premises Active Directory** (Sincronizar trabajadores de Workday con Active Directory local) o **Synchronize Workers to Azure AD** (Sincronizar trabajadores con Azure AD).

13. Seleccione **Add new mapping** (Agregar nueva asignación).

14. El nuevo atributo debe aparecer ahora en la lista **Atributo de origen**.

15. Agregue una asignación para el nuevo atributo según sea necesario.

16. Cuando termine, recuerde establecer **Estado de aprovisionamiento** de nuevo en **Activado** y haga clic en Guardar.

### <a name="exporting-and-importing-your-configuration"></a>Exportación e importación de la configuración

Consulte el artículo [Exportación e importación de la configuración de aprovisionamiento](../app-provisioning/export-import-provisioning-configuration.md).

## <a name="managing-personal-data"></a>Administración de datos personales

La solución de aprovisionamiento Workday para Active Directory requiere que se instale un agente de aprovisionamiento en un servidor de Windows local, y este agente crea registros en el registro de eventos de Windows, que puede contener datos personales en función de las asignaciones de atributos de Workday para AD. Para cumplir las obligaciones de privacidad del usuario, puede asegurarse de que no se conserva ningún dato en los registros de eventos durante más de 48 horas; para ello, configure una tarea programada de Windows para borrar el registro de eventos.

El servicio de aprovisionamiento de Azure AD se engloba dentro de la categoría de **procesador de datos** de la clasificación del RGPD. Como canalización del procesador de datos, el servicio proporciona servicios de procesamiento de datos a asociados clave y consumidores finales. El servicio de aprovisionamiento de Azure AD no genera datos de usuario y no tiene control independiente sobre qué datos personales se recopilan y cómo se usan. La recuperación de datos, la agregación, el análisis y los informes en el servicio de aprovisionamiento de Azure AD se basan en los datos empresariales existentes.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-hybrid-note.md)]

Con respecto a la retención de datos, el servicio de aprovisionamiento de Azure AD no genera informes, realiza análisis ni proporciona información más allá de 30 días. Por tanto, el servicio de aprovisionamiento de Azure AD no almacena, procesa ni retiene ningún dato más allá de 30 días. Este diseño es compatible con el reglamento del GDPR, las regulaciones de cumplimiento de la privacidad de Microsoft y las directivas de retención de datos de Azure AD.

## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre los escenarios de integración de Azure AD y Workday y las llamadas a servicios web](../app-provisioning/workday-integration-reference.md)
* [Aprenda a revisar los registros y a obtener informes sobre la actividad de aprovisionamiento](../app-provisioning/check-status-user-account-provisioning.md)
* [Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Workday](workday-tutorial.md)
* [Aprenda a configurar la escritura diferida de Workday](workday-writeback-tutorial.md)
* [Obtenga información sobre cómo usar Microsoft Graph API para administrar configuraciones de aprovisionamiento](/graph/api/resources/synchronization-overview)