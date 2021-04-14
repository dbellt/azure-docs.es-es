---
title: Tutorial para configurar N8 Identity con Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Tutorial para configurar TheAccessHub Admin Tool con Azure Active Directory B2C para realizar la migración de cuentas de clientes y la administración de solicitudes de servicio al cliente (CSR).
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/26/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 9fee7965e7834f0e05dba91f8652a0e155087b14
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107257881"
---
# <a name="tutorial-for-configuring-theaccesshub-admin-tool-with-azure-active-directory-b2c"></a>Tutorial para configurar TheAccessHub Admin Tool con Azure Active Directory B2C

En este tutorial de ejemplo, se proporcionan instrucciones sobre cómo integrar Azure Active Directory (AD) B2C con [TheAccessHub Admin Tool](https://n8id.com/products/theaccesshub-admintool/) de N8 Identity. Esta solución se encarga de la migración de cuentas de clientes y la administración de solicitudes de servicio al cliente (CSR).  

Esta solución es adecuada para usted si tiene uno o varios de los siguientes requisitos:

- Tiene un sitio existente y quiere migrar a Azure AD B2C. Sin embargo, tiene dificultades con la migración de las cuentas de clientes, incluidas las contraseñas

- Necesita una herramienta para que CSR administre las cuentas de Azure AD B2C.

- Tiene como requisito usar la administración delegada para las CSR.

- Quiere sincronizar y combinar los datos de muchos repositorios en Azure AD B2C.

## <a name="pre-requisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- Un [inquilino de Azure AD B2C](./tutorial-create-tenant.md). El inquilino debe estar vinculado a la suscripción a Azure.

- Un entorno de TheAccessHub Admin Tool: Póngase en contacto con [N8 Identity](https://n8id.com/contact/) para aprovisionar un nuevo entorno.

- [Opcional] Información de conexión y credenciales para cualquier base de datos o protocolo ligero de acceso a directorios (LDAP) desde el que quiere migrar los datos del cliente.

- [Opcional] Entorno configurado de Azure AD B2C para el uso de [directivas personalizadas](./tutorial-create-user-flows.md?pivots=b2c-custom-policy), si quiere integrar TheAccessHub Admin Tool en el flujo de directivas de registro.

## <a name="scenario-description"></a>Descripción del escenario

TheAccessHub Admin Tool se ejecuta como cualquier otra aplicación de Azure. Puede ejecutarse en la suscripción a Azure de N8 Identity o en la suscripción del cliente. En el siguiente diagrama de arquitectura se muestra la implementación.

![Imagen que muestra el diagrama de la arquitectura de N8 Identity](./media/partner-n8identity/n8identity-architecture-diagram.png)

|Paso | Descripción |
|:-----| :-----------|
| 1. | El usuario llega a una página de inicio de sesión. Los usuarios optan por iniciar sesión para crear una nueva cuenta y escribir información en la página. Azure AD B2C recopila los atributos de usuario.
| 2. | Azure AD B2C llama a TheAccessHub Admin Tool y pasa los atributos de usuario.
| 3. | TheAccessHub Admin Tool comprueba la base de datos existente para obtener información del usuario actual.  
| 4. | El registro de usuario se sincroniza de la base de datos a TheAccessHub Admin Tool.
| 5. | TheAccessHub Admin Tool comparte los datos con el administrador de CSR/Helpdesk delegado.
| 6. | TheAccessHub Admin Tool sincroniza los registros de usuario con Azure AD B2C.
| 7. |En función de la respuesta correcta o con errores de TheAccessHub Admin Tool, Azure AD B2C envía un correo electrónico de bienvenida personalizado al usuario.

## <a name="create-a-global-admin-in-your-azure-ad-b2c-tenant"></a>Creación de un administrador global en el inquilino de Azure AD B2C

TheAccessHub Admin Tool requiere permisos para actuar en nombre de un administrador global con el fin de leer información de usuario y realizar cambios en el inquilino de Azure AD B2C. Los cambios realizados en los administradores normales no afectarán a la capacidad de TheAccessHub Admin Tool para interactuar con el inquilino.

Para crear un administrador global, siga estos pasos:

1. En Azure Portal, inicie sesión en su inquilino de Azure AD B2C como administrador. Vaya a **Azure Active Directory** > **Usuarios**.
2. Seleccione **Nuevo usuario**.
3. Elija **Crear usuario** para crear un usuario de directorio normal y no un cliente.
4. Complete el formulario de datos de identidad.

   a. Escriba el nombre de usuario, como "theaccesshub".

   b. Escriba el nombre, como "cuenta de servicio de TheAccessHub".

5. Seleccione **Mostrar contraseña** y copie la contraseña inicial para usarla posteriormente.
6. Asigne el rol Administrador global.

   a. Seleccione al **Usuario** de los roles actuales de usuario para cambiarlo.

   b. Compruebe el registro para el administrador global.

   c. **Seleccionar** > **Crear**.

## <a name="connect-theaccesshub-admin-tool-with-your-azure-ad-b2c-tenant"></a>Conexión de TheAccessHub Admin Tool con el inquilino de Azure AD B2C

TheAccessHub Admin Tool usa Graph API de Microsoft para leer y realizar cambios en el directorio. Actúa como administrador global en su inquilino. Se necesitan permisos adicionales para TheAccessHub Admin Tool, que puede conceder desde dentro de la herramienta.

Para autorizar a TheAccessHub Admin Tool el acceso a su directorio, siga estos pasos:

1. Inicie sesión en TheAccessHub Admin Tool con las credenciales que le proporcionó N8 Identity.

2. Vaya a **System Admin** > **Azure AD B2C Config** (Administrador del sistema > Configuración de Azure AD B2C).

3. Seleccione **Autorizar conexión**.

4. En la venta que se abre, inicie sesión en con la cuenta de administrador global. Se le podría pedir que restablezca la contraseña si va a iniciar sesión por primera vez con la nueva cuenta de servicio.

5. Siga las indicaciones y seleccione **Aceptar** para conceder los permisos solicitados a TheAccessHub Admin Tool.

## <a name="configure-a-new-csr-user-using-your-enterprise-identity"></a>Configuración de un nuevo usuario de CSR mediante la identidad de empresa

Cree un usuario CSR/Helpdesk que acceda a TheAccessHub Admin Tool con sus credenciales de Azure Active Directory empresariales existentes.

Para configurar el usuario CSR/Helpdesk con el inicio de sesión único (SSO), se recomiendan los siguientes pasos:

1. Inicie sesión en TheAccessHub Admin Tool con las credenciales proporcionadas por N8 Identity.

2. Vaya a **Manager Tools** > **Manage Colleagues** (Herramientas de administrador > Administrar compañeros).

3. Seleccione **Add Colleague** (Agregar compañero).

4. Seleccione **Colleague Type Azure Administrator** (Tipo de compañero > Administrador de Azure).

5. Escriba la información de perfil de su compañero.

   a. La selección de organización principal controlará quién tiene permiso para administrar este usuario.

   b. En id. de inicio de sesión o nombre de usuario de Azure AD, proporcione el nombre principal de usuario de la cuenta de Azure Active Directory del usuario.

   c. En la pestaña Roles de TheAccessHub, compruebe el rol administrado Helpdesk. Este permitirá que el usuario acceda a la vista de administración de compañeros. De todos modos, el usuario tendrá que incluirse en un grupo o convertirse en propietario de la organización para realizar acciones con los clientes.

6. Seleccione **Submit** (Enviar).

## <a name="configure-a-new-csr-user-using-a-new-identity"></a>Configuración de un nuevo usuario de CSR mediante una identidad nueva

Cree un usuario CSR/Helpdesk que tendrá acceso a TheAccessHub Admin Tool con una nueva credencial local única para dicha herramienta. Principalmente, lo usarán las organizaciones que no utilizan Azure AD para su empresa.

Para [configurar un usuario CSR/Helpdesk](https://youtu.be/iOpOI2OpnLI) sin inicio de sesión único, siga estos pasos:

1. Inicie sesión en TheAccessHub Admin Tool con las credenciales proporcionadas por N8 Identity.

2. Vaya a **Manager Tools** > **Manage Colleagues** (Herramientas de administrador > Administrar compañeros).

3. Seleccione **Add Colleague** (Agregar compañero).

4. Seleccione **Colleague Type Local Administrator** (Tipo de compañero > Administrador local).

5. Escriba la información de perfil de su compañero.

   a. La selección de organización principal controlará quién tiene permiso para administrar este usuario.

   b. En la pestaña **Roles** de TheAccessHub, seleccione el rol administrado **Helpdesk**. Este permitirá que el usuario acceda a la vista de administración de compañeros. De todos modos, el usuario tendrá que incluirse en un grupo o convertirse en propietario de la organización para realizar acciones con los clientes.

6. Copie el **Id. o correo electrónico de inicio de sesión** y los atributos de **Contraseña de un solo uso**. Proporcione dicha información al nuevo usuario. Este usará las credenciales para iniciar sesión en TheAccessHub Admin Tool. Se le pedirá al usuario que escriba una contraseña nueva en su primer inicio de sesión.

7. Seleccionar **Enviar**

## <a name="configure-partitioned-csr-administration"></a>Configuración de la administración de CSR con particiones

Los permisos para administrar usuarios de clientes y CSR/Helpdesk en TheAccessHub Admin Tool se administran mediante una jerarquía de organización. Todos los compañeros y clientes pertenecen a una organización principal. Puede asignar compañeros o grupos de compañeros específicos como propietarios de las organizaciones.  Los propietarios de la organización pueden administrar (realizar cambios) a los compañeros y clientes de las organizaciones u organizaciones secundarias que poseen. Para permitir que varios compañeros administren un conjunto de usuarios, puede crear un grupo con muchos miembros. Después, el grupo se puede asignar como propietario de la organización y todos los miembros de dicho grupo podrán administrar a los compañeros y clientes de la organización.

### <a name="create-a-new-group"></a>Creación de un grupo nuevo

1. Inicie sesión en TheAccessHub Admin Tool con las **credenciales** que le proporcionó N8 Identity.

2. Vaya a **Organization > Manage Groups** (Organización > Administrar grupos).

3. Seleccione > **Add Group** (Agregar grupo).

4. Escriba un **Nombre de grupo**, una **Descripción del grupo** y un **Propietario de grupo**.

5. Busque y active las casillas de los compañeros que quiere agregar como miembros del grupo y, a continuación, seleccione > **Add** (Agregar).

6. Podrá ver a todos los miembros del grupo en la parte inferior de la página.

7. De ser necesario, puede seleccionar la **x** al final de la fila para quitar a los miembros.

8. Seleccionar **Enviar**

### <a name="create-a-new-organization"></a>Creación de una organización

1. Inicie sesión en TheAccessHub Admin Tool con las credenciales que le proporcionó N8 Identity.

2. Vaya a **Organization > Manage Organizations** (Organización > Administrar organizaciones).

3. Seleccione > **Add Organization** (Agregar organización).

4. Proporcione un **Nombre de organización**, **Propietario de la organización** y **Organización principal**.

    a. Idealmente, el nombre de la organización es un valor que se corresponde con los datos del cliente. Al cargar los datos de compañeros y clientes, si proporciona el nombre de la organización durante la carga, el compañero se podrá asignar automáticamente a la organización.

    b. El propietario representa a la persona o grupo que administrará a los clientes y compañeros de esta organización y de todas las organizaciones secundarias que contiene.

    c. La organización primaria indica qué otra organización también es intrínsecamente responsable de esta organización.

5. Seleccione **Submit** (Enviar).

### <a name="modify-the-hierarchy-via-the-tree-view"></a>Modificación de la jerarquía mediante la vista de árbol

1. Inicie sesión en TheAccessHub Admin Tool con las credenciales que le proporcionó N8 Identity.

2. Vaya a **Manager Tools** > **Tree View** (Herramientas de administrador > Vista de árbol).

3. En esta representación, puede visualizar qué compañeros y grupos pueden administrar cuáles organizaciones.

4. Puede modificar las jerarquías al arrastrar las organizaciones sobre las organizaciones que quiera asignar como primarias.

5. Seleccione **Save** (Guardar) cuando haya terminado de modificar la jerarquía.

## <a name="customize-welcome-notification"></a>Personalización de la notificación de bienvenida

Mientras usa TheAccessHub para migrar usuarios de la solución de autenticación anterior a Azure AD B2C, se recomienda personalizar la notificación de bienvenida del usuario, que se envía mediante TheAccessHub durante la migración. Normalmente, este mensaje incluye un vínculo para que el cliente establezca una nueva contraseña en el directorio de Azure AD B2C.

Para personalizar la notificación:

1. Inicie sesión en TheAccessHub con las credenciales que le proporcionó N8 Identity.

2. Vaya a **System Admin** > **Notifications** (Administrador del sistema > Notificaciones).

3. Seleccione la opción **Create Colleague template** (Crear plantilla de compañero).

4. Seleccione **Editar**.

5. Modifique los campos de plantilla y mensaje según sea necesario. El campo de plantilla es compatible con HTML, y puede enviar notificaciones con formato HTML a los correos electrónicos del cliente.

6. Seleccione **Save** (Guardar) cuando termine.

## <a name="migrate-data-from-external-data-sources-to-azure-ad-b2c"></a>Migración de datos desde orígenes de datos externos a Azure AD B2C

Con TheAccessHub Admin Tool, puede importar datos de varias bases de datos, LDAP y archivos CSV y, a continuación, enviarlos a su inquilino de Azure AD B2C. Para ello, se cargan datos en el tipo de compañero del usuario de Azure AD B2C dentro de TheAccessHub Admin Tool.  Si el origen de datos no es Azure, los datos se colocarán tanto en TheAccessHub Admin Tool y Azure AD B2C. Si el origen de los datos externos no es un archivo. csv simple en el equipo, configure un origen de datos antes de realizar la carga de datos. Los pasos siguientes describen la creación de un origen de datos, así como la carga de datos.

### <a name="configure-a-new-data-source"></a>Configuración de un nuevo origen de datos

1. Inicie sesión en TheAccessHub Admin Tool con las credenciales que le proporcionó N8 Identity.

2. Vaya a **System Admin** > **Data Sources** (Administrador del sistema > Orígenes de datos).

3. Seleccione **Add Data Source** (Agregar origen de datos).

4. Proporcione un **Nombre** y **Tipo** para el origen de datos.

5. Rellene los datos del formulario, en función del tipo de origen de datos:

   **Para las bases de datos**

   a. **Tipo**: base de datos

   b. **Tipo de base de datos**: seleccione una base de datos de alguno de los tipos de base de datos admitidos.

   c. **URL de conexión**: escriba una cadena de conexión de JDBC con el formato correcto. Por ejemplo: ``jdbc:postgresql://myhost.com:5432/databasename``

   d. **Nombre de usuario**: escriba el nombre de usuario para acceder a la base de datos.

   e. **Contraseña**: escriba la contraseña para acceder a la base de datos.

   f. **Consulta**: escriba la consulta SQL para extraer los detalles del cliente. Por ejemplo: ``SELECT * FROM mytable;``

   g. Seleccione **Test Connection** (Probar conexión) y verá un ejemplo de los datos para asegurarse de que la conexión funciona.

   **Para los LDAP**

   a. **Tipo**: LDAP

   b. **Host**: escriba el nombre de host o la dirección IP de la máquina en la que se ejecuta el servidor de LDAP. Por ejemplo: ``mysite.com``

   c. **Puerto**: escriba el número de puerto en el que escucha el servidor de LDAP.

   d. **SSL**: active la casilla si TheAccessHub Admin Tool debe comunicarse con el LDAP de manera segura mediante SSL. Se recomienda el uso de SSL.

   e. **DN de inicio de sesión**: escriba el DN de la cuenta de usuario para iniciar sesión y realice la búsqueda LDAP.

   f. **Password**: escriba la contraseña del usuario.

   g. **DN base**: escriba el DN en la parte superior de la jerarquía en la que quiere realizar la búsqueda.

   h. **Filtro**: escriba la cadena de filtro LDAP, que obtendrá los registros de clientes.

   i. **Atributos**: especifique una lista separada por comas de atributos de los registros de cliente para pasarlos a TheAccessHub Admin Tool.

   j. Seleccione la opción **Test Connection** (Probar conexión) y verá un ejemplo de los datos para asegurarse de que la conexión funciona.

   **Para OneDrive**

   a. **Tipo**: OneDrive para la Empresa.

   b. Seleccione **Autorizar conexión**.

   c. Una nueva ventana le pedirá que inicie sesión en **OneDrive**; inicie sesión con un usuario con acceso de lectura a su cuenta de OneDrive. TheAccessHub Admin Tool actuará para que este usuario pueda leer los archivos de carga CSV.

   d. Siga las indicaciones y seleccione **Aceptar** para conceder los permisos solicitados a TheAccessHub Admin Tool.

6. Seleccione **Save** (Guardar) cuando termine.  

### <a name="synchronize-data-from-your-data-source-into-azure-ad-b2c"></a>Sincronización de datos del origen de datos en Azure AD B2C

1. Inicie sesión en TheAccessHub Admin Tool con las credenciales que le proporcionó N8 Identity.

2. Vaya a **System Admin** > **Data Synchronization** (Administrador del sistema > Sincronización de datos).

3. Seleccione **New Load** (Nueva carga).

4. Seleccione el **Colleague Type** (Tipo de compañero) del usuario de Azure AD B2C.

5. Seleccione **Source** (Origen) y, en el cuadro de diálogo emergente, seleccione el origen de datos. Si creó un origen de datos de OneDrive, seleccione también el archivo.

6. Si no quiere crear nuevas cuentas de cliente con esta carga, cambie la primera directiva: **SI no se encuentra al compañero en TheAccessHub, ENTONCES** a **No hacer nada**.

7. Si no quiere actualizar las cuentas de cliente existentes con esta carga, cambie la segunda directiva **SI los datos de origen y de TheAccessHub no coinciden, ENTONCES** a **No hacer nada**.

8. Seleccione **Siguiente**.

9. En **Search-Mapping configuration** (Configuración de asignación de búsqueda), se identifica cómo poner en correlación los registros de carga con los clientes que ya están cargados en TheAccessHub Admin Tool. Elija uno o más atributos de identificación en el origen. Haga coincidir los atributos con un atributo en TheAccessHub Admin Tool que contenga los mismos valores. Si se encuentra una coincidencia, se invalidará el registro existente; de lo contrario, se creará un cliente nuevo. Puede ordenar varias comprobaciones de este tipo. Por ejemplo, podría comprobar primero el correo electrónico y, después, el nombre y el apellido.

10. En el menú de la izquierda, seleccione **Data Mapping** (Asignación de datos).

11. En la configuración de asignación de datos, asigne qué atributos de TheAccessHub Admin Tool se deben rellenar a partir de los atributos de origen. No es necesario asignar todos los atributos. Los atributos sin asignar permanecerán sin cambios para los clientes existentes.

12. Si asigna al atributo org_name con un valor que sea el nombre de una organización existente, los clientes que se creen se colocarán en esa organización.

13. Seleccione **Siguiente**.

14. P|uede especificar una programación diaria, semanal o mensual si esta carga debe repetirse. De lo contrario, mantenga el valor predeterminado **Now** (Ahora).

15. Seleccionar **Enviar**

16. Si seleccionó la opción de programación **Now** (Ahora), se agregará un nuevo registro a la pantalla Sincronizaciones de datos inmediatamente. Una vez que la fase de validación haya alcanzado el 100 %, seleccione **new record** (nuevo registro) para ver el resultado esperado de la carga. En el caso de las cargas programadas, estos registros solo aparecerán después del tiempo programado.

17. Si no hay ningún error, seleccione **Run** (Ejecutar) para confirmar los cambios. De lo contrario, seleccione **Remove** (Quitar) en el menú **More** (Más) para quitar la carga. De este modo, podrá corregir los datos de origen o cargar las asignaciones y volver a intentarlo. En su lugar, si el número de errores es pequeño, puede actualizar manualmente los registros y seleccionar **Update** (Actualizar) en cada registro para realizar correcciones. Por último, puede continuar con los errores presentes y resolverlos más adelante como **Support Interventions** (Intervenciones de soporte) en TheAccessHub Admin Tool.

18. Cuando la fase de carga del registro de **Sincronización de datos** alcanza el 100 %, se inician todos los cambios resultantes de la carga. Los clientes deben comenzar a aparecer o recibir cambios en Azure AD B2C.

## <a name="synchronize-azure-ad-b2c-customer-data"></a>Sincronización de datos del cliente de Azure AD B2C 

Como operación única o continua, TheAccessHub Admin Tool puede sincronizar toda la información del cliente de Azure AD B2C en TheAccessHub Admin Tool. Esto garantiza que los administradores de CSR/Helpdesk ven la información actualizada del cliente.

Para sincronizar los datos de Azure AD B2C en TheAccessHub Admin Tool:

1. Inicie sesión en TheAccessHub Admin Tool con las credenciales que le proporcionó N8 Identity.

2. Vaya a **System Admin** > **Data Synchronization** (Administrador del sistema > Sincronización de datos).

3. Seleccione **New Load** (Nueva carga).

4. Seleccione el **Colleague Type** (Tipo de compañero) del usuario de Azure AD B2C.

5. Para el paso de **Options** (Opciones), deje los valores predeterminados.

6. Seleccione **Siguiente**.

7. Deje los valores predeterminados para el pasto **Data Mapping & Search** (Asignación y búsqueda de datos). A excepción de si asigna al atributo **org_name** con un valor que sea el nombre de una organización existente; en ese caso, los clientes que se creen se colocarán en esa organización.

8. Seleccione **Siguiente**.

9. P|uede especificar una programación diaria, semanal o mensual si esta carga debe repetirse. De lo contrario, mantenga el valor predeterminado: **Now** (Ahora). Se recomienda sincronizar desde Azure AD B2C de manera periódica.

10. Seleccionar **Enviar**

11. Si seleccionó la opción de programación **Now** (Ahora), se agregará un nuevo registro a la pantalla Sincronizaciones de datos inmediatamente. Una vez que la fase de validación haya alcanzado el 100 %, seleccione el nuevo registro para ver el resultado esperado de la carga. En el caso de las cargas programadas, estos registros solo aparecerán después del tiempo programado.

12. Si no hay ningún error, seleccione **Run** (Ejecutar) para confirmar los cambios. De lo contrario, seleccione **Remove** (Quitar) en el menú Más para quitar la carga. De este modo, podrá corregir los datos de origen o cargar las asignaciones y volver a intentarlo. En su lugar, si el número de errores es pequeño, puede actualizar manualmente los registros y seleccionar **Update** (Actualizar) en cada registro para realizar correcciones. Por último, puede continuar con los errores presentes y resolverlos más adelante como intervenciones de soporte en TheAccessHub Admin Tool.

13. Cuando la fase de carga del registro de **Sincronización de datos** alcanza el 100 %, se inician todos los cambios resultantes de la carga.

## <a name="configure-azure-ad-b2c-policies"></a>Configuración de directivas de Azure AD B2C

Ocasionalmente, la sincronización de TheAccessHub Admin Tool está limitada por su capacidad para mantener su estado actualizado con Azure AD B2C. Podemos aprovechar las directivas de Azure AD B2C y la API de TheAccessHub Admin Tool para informar a TheAccessHub Admin Tool de los cambios a medida que se producen. Esta solución requiere conocimientos técnicos de las [directivas personalizadas de Azure AD B2C](./user-flow-overview.md). En la siguiente sección, le daremos los detalles de un ejemplo de directiva y un certificado seguro para notificar a TheAccessHub Admin Tool de cuentas nuevas en las directivas personalizadas de registro.

### <a name="create-a-secure-credential-to-invoke-theaccesshub-admin-tools-api"></a>Creación de una credencial segura para invocar a la API de TheAccessHub Admin Tool

1. Inicie sesión en TheAccessHub Admin Tool con las credenciales que le proporcionó N8 Identity.

2. Vaya a **System Admin** > **Admin Tools** > **API Security** (Administrador del sistema > Herramientas de administrador > Seguridad de API).

3. Seleccione **Generate** (Generar).

4. Copie la **Contraseña del certificado**.

5. Seleccione **Download** (Descargar) para obtener el certificado de cliente.

6. Siga este [tutorial](./secure-rest-api.md#https-client-certificate-authentication ) para agregar el certificado de cliente a Azure AD B2C.

### <a name="retrieve-your-custom-policy-examples"></a>Recuperación de los ejemplos de directivas personalizadas

1. Inicie sesión en TheAccessHub Admin Tool con las credenciales que le proporcionó N8 Identity.

2. Vaya a **System Admin** > **Admin Tools** > **Azure B2C Policies** (Administrador del sistema > Herramientas de administrador > Directivas de Azure B2C).

3. Especifique su dominio de inquilino de Azure AD B2C y el id. de las dos instancias de Identity Experience Framework de la configuración de Identity Experience Framework.

4. Seleccione **Guardar**.

5. Seleccione **Download** (Descargar) para obtener un archivo ZIP con directivas básicas que agreguen clientes a TheAccessHub Admin Tool a medida que los clientes se registren.

6. Siga este [tutorial](./tutorial-create-user-flows.md?pivots=b2c-custom-policy) para empezar a diseñar directivas personalizadas en Azure AD B2C.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](./custom-policy-overview.md)

- [Introducción a las directivas personalizadas en Azure AD B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy)