---
title: Aspectos básicos de la autenticación de Azure Key Vault
description: Más información sobre cómo funciona el modelo de autenticación de Key Vault
author: ShaneBala-keyvault
ms.author: sudbalas
ms.date: 09/25/2020
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.openlocfilehash: c43995a8b3a072d98db0ba2c8219694f17e49a26
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107363434"
---
# <a name="key-vault-authentication-fundamentals"></a>Aspectos básicos de la autenticación de Key Vault

Azure Key Vault le permite almacenar y administrar de forma segura las credenciales de la aplicación, como secretos, claves y certificados, en un repositorio central y seguro en la nube. Key Vault elimina la necesidad de almacenar credenciales en las aplicaciones. Asimismo, las aplicaciones pueden autenticarse en Key Vault en tiempo de ejecución para recuperar las credenciales.

Como administrador, puede controlar estrechamente qué usuarios y aplicaciones pueden acceder a su almacén de claves y puede limitar y auditar las operaciones que realicen. En este documento se explican los conceptos fundamentales del modelo de acceso del almacén de claves. Le proporcionará un nivel de conocimientos introductorio y le mostrará todos los pasos necesarios para autenticar un usuario o una aplicación en el almacén de claves.

## <a name="required-knowledge"></a>Conocimientos necesarios

En este documento se da por supuesto que está familiarizado con los siguientes conceptos. Si no está familiarizado con ninguno de estos conceptos, siga los pasos de los vínculos de ayuda antes de continuar.

* [Vínculo](../../active-directory/fundamentals/active-directory-whatis.md) de Azure Active Directory
* [Vínculo](./authentication.md#app-identity-and-security-principals) de las entidades de seguridad

## <a name="key-vault-configuration-steps-summary"></a>Resumen de los pasos de configuración de Key Vault

1. Registre el usuario o la aplicación en Azure Active Directory como entidad de seguridad.
1. Configure una asignación de roles para la entidad de seguridad en Azure Active Directory.
1. Configure las directivas de acceso del almacén de claves para la entidad de seguridad.
1. Configure el acceso del firewall de Key Vault al almacén de claves (opcional).
1. Pruebe la capacidad de la entidad de seguridad para acceder a Key Vault.

## <a name="register-a-user-or-application-in-azure-active-directory-as-a-security-principal"></a>Registro de un usuario o una aplicación en Azure Active Directory como entidad de seguridad

Cuando un usuario o una aplicación realiza una solicitud al almacén de claves, la solicitud debe autenticarse primero mediante Azure Active Directory. Para que esto funcione, el usuario o la aplicación deben registrarse en Azure Active Directory como entidad de seguridad.

Siga los vínculos de la documentación siguiente para saber cómo registrar un usuario o una aplicación en Azure Active Directory.
**Asegúrese de crear una contraseña para el registro de usuarios y un secreto de cliente o una credencial de certificado de cliente para las aplicaciones.**

* [Vínculo](../../active-directory/fundamentals/add-users-azure-active-directory.md) para registrar un usuario en Azure Active Directory.
* [Vínculo](../../active-directory/develop/quickstart-register-app.md) para registrar una aplicación en Azure Active Directory.

## <a name="assign-your-security-principal-a-role"></a>Asignación de la entidad de seguridad como rol

Puede usar el control de acceso basado en roles de Azure (Azure RBAC) para asignar permisos a las entidades de seguridad. Estos permisos se denominan asignaciones de roles.

En el contexto del almacén de claves, estas asignaciones de roles determinan el nivel de acceso de una entidad de seguridad al plano de administración (también conocido como plano de control) del almacén de claves. Estas asignaciones de roles no proporcionan acceso a los secretos del plano de datos directamente, sino que proporcionan acceso para administrar las propiedades del almacén de claves. Por ejemplo, si se ha asignado el **rol de lector** a un usuario o una aplicación, no podrán realizar cambios en la configuración del firewall del almacén de claves; en cambio, un usuario o una aplicación con un **rol de colaborador** asignado puede realizar cambios. Aún así, ninguno de estos roles obtendrá acceso directo para realizar operaciones en secretos, claves y certificados como la creación o recuperación de su valor hasta que se les asigne acceso al plano de datos del almacén de claves. Esto se describe en el paso siguiente.

>[!IMPORTANT]
> Aunque los usuarios con el rol de Colaborador o Propietario no tienen acceso para realizar operaciones en secretos almacenados en el almacén de claves de forma predeterminada, los roles de Colaborador y Propietario, proporcionan permisos para agregar o quitar directivas de acceso a secretos almacenados en el almacén de claves. Por lo tanto, un usuario con estas asignaciones de roles puede concederse a si mismo acceso a los secretos del almacén de claves. Por esta razón, se recomienda que solo los administradores tengan acceso a los roles de Colaborador o Propietario. Los usuarios y las aplicaciones que solo necesitan recuperar secretos del almacén de claves deben tener el rol de Lector. **Obtenga más detalles en la sección siguiente.**

>[!NOTE]
> Cuando asigna una asignación de roles a un usuario en el nivel de inquilino de Azure Active Directory, este conjunto de permisos se filtrará a todas las suscripciones, grupos de recursos y recursos dentro del ámbito de la asignación. Para seguir el principio de privilegios mínimos, puede realizar esta asignación de roles en un ámbito más granular. Por ejemplo, puede asignar a un usuario un rol de Lector en el nivel de suscripción y un rol de Propietario para un único almacén de claves. Para ello, vaya a la configuración de la Administración de identidad y acceso (IAM) de una suscripción, un grupo de recursos o un almacén de claves para realizar una asignación de roles en un ámbito más granular.

* [Vínculo](../../role-based-access-control/built-in-roles.md) para obtener más información sobre los roles de Azure.
* [Vínculo](../../role-based-access-control/role-assignments-portal.md) para obtener más información acerca de cómo asignar o quitar asignaciones de roles.

## <a name="configure-key-vault-access-policies-for-your-security-principal"></a>Configuración de directivas de acceso al almacén de claves para la entidad de seguridad

Antes de conceder acceso al almacén de claves a los usuarios y aplicaciones, es importante comprender los distintos tipos de operaciones que se pueden realizar en un almacén de claves. Existen dos tipos principales de operaciones del almacén de claves: las operaciones del plano de administración (también denominadas "plano de control") y las operaciones del plano de datos.

En esta tabla se muestran varios ejemplos de las distintas operaciones que controlan el plano de administración y el plano de datos. Las operaciones que cambian las propiedades del almacén de claves son operaciones del plano de administración. En cambio, las operaciones que cambian o recuperan el valor de los secretos almacenados en el almacén de claves son operaciones del plano de datos.

|Operaciones del plano de administración (ejemplos)|Operaciones del plano de datos (ejemplos)|
| --- | --- |
| Create Key Vault | Crear una clave, secreto y certificado
| Eliminar Key Vault | Eliminar una clave, secreto y certificado
| Agregar o quitar las asignaciones de roles de Key Vault | Enumerar y obtener valores de claves, secretos y certificados
| Agregar o eliminar las directivas de acceso de Key Vault | Realizar copias de seguridad y restaurar claves, secretos y certificados
| Modificar la configuración del firewall de Key Vault | Renovar claves, secretos y certificados
| Modificar la configuración de recuperación de Key Vault | Purgar o recuperar las claves, secretos y certificados eliminados temporalmente
| Modificar la configuración de registros de diagnóstico de Key Vault

### <a name="management-plane-access--azure-active-directory-role-assignments"></a>Acceso al plano de administración y a las asignaciones de roles de Azure Active Directory

Las asignaciones de roles de Azure Active Directory conceden acceso para realizar operaciones del plano de administración en un almacén de claves. Normalmente, este acceso se concede a los usuarios y no a las aplicaciones. Puede restringir las operaciones del plano de administración que puede realizar un usuario; para ello, cambie la asignación de roles de un usuario.

Por ejemplo, si asigna un rol de Lector de Key Vault a un usuario, le permitirá ver las propiedades de su almacén de claves, como las directivas de acceso, pero no podrán realizar ningún cambio. En cambio, si asigna a un usuario un rol de Propietario, le permitirá tener acceso completo para cambiar la configuración del plano de administración del almacén de claves.

Las asignaciones de roles se controlan en la hoja de Access Control del almacén de claves (IAM). Si quiere que un usuario determinado tenga acceso de lector o quiere que sea el administrador de varios recursos del almacén de claves, puede crear una asignación de roles en el almacén, en el grupo de recursos o en el nivel de suscripción, y la asignación de roles se agregará a todos los recursos dentro del ámbito de la asignación.

El acceso al plano de datos o el acceso para realizar operaciones en claves, secretos y certificados que están almacenados en el almacén de claves, se puede agregar de una de estas dos maneras.

### <a name="data-plane-access-option-1-classic-key-vault-access-policies"></a>Opción 1 de acceso al plano de datos: directivas de acceso de Azure Key Vault

Las directivas de acceso del almacén de claves conceden a los usuarios y las aplicaciones acceso para realizar operaciones de plano de datos en un almacén de claves.

> [!NOTE]
> Este modelo de acceso no es compatible con Azure RBAC del almacén de claves (opción 2) que se describe a continuación. Debe elegir una. Tendrá la oportunidad de realizar esta selección al hacer clic en la pestaña de la Directiva de acceso del almacén de claves.

Las directivas de acceso clásico son granulares, lo que significa que puede permitir o denegar la capacidad de cada usuario o aplicación individual para realizar operaciones individuales en un almacén de claves. Estos son algunos ejemplos:

* La entidad de seguridad 1 puede realizar cualquier operación de clave, pero no puede realizar ninguna operación secreta o de certificado.
* La entidad de seguridad 2 puede enumerar y leer todas las claves, secretos y certificados, pero no puede realizar ninguna operación de creación, eliminación o renovación.
* La entidad de seguridad 3 puede realizar copias de seguridad y restaurar todos los secretos, pero no puede leer el valor de los secretos.

Sin embargo, las directivas de acceso clásico no admiten permisos de nivel por objeto y los permisos asignados se aplican al ámbito de un almacén de claves individual. Por ejemplo, si concede el permiso de la directiva de acceso "Obtener secretos" a una entidad de seguridad en un almacén de claves determinado, la entidad de seguridad tendrá la capacidad de obtener todos los secretos de ese almacén de claves en particular. Sin embargo, este permiso "Obtener secretos" no se extenderá automáticamente a otros almacenes de claves y debe asignarse explícitamente.

> [!IMPORTANT]
> Las directivas de acceso de almacén de claves clásicas y las asignaciones de roles de Azure Active Directory son independientes entre sí. Si asigna a una entidad de seguridad un rol de "Colaborador" en el nivel de suscripción, la entidad de seguridad no podrá de realizar automáticamente operaciones de plano de datos en cada almacén de claves del ámbito de la suscripción. Así pues, la entidad de seguridad aún deberá recibir o concederse los permisos de la directiva de acceso para realizar operaciones en el plano de datos.

### <a name="data-plane-access-option-2--azure-rbac-for-key-vault"></a>Opción 2 de acceso al plano de datos:  Azure RBAC para Key Vault

Una nueva forma de conceder acceso al plano de datos del almacén de claves es mediante el control de acceso basado en roles de Azure (Azure RBAC) del almacén de claves.

> [!NOTE]
> Este modelo de acceso no es compatible con las directivas de acceso clásico del almacén de claves que se mostraron anteriormente. Debe elegir una. Tendrá la oportunidad de realizar esta selección al hacer clic en la pestaña de la Directiva de acceso del almacén de claves.

Las asignaciones de roles de Key Vault son un conjunto de asignaciones de roles integradas de Azure que abarcan conjuntos de permisos comunes que se usan para obtener acceso a claves, secretos y certificados. Este modelo de permiso también habilita funcionalidades adicionales que no están disponibles en el modelo clásico de directivas de acceso del almacén de claves.

* Los permisos de Azure RBAC se pueden administrar a escala, lo que permite a los usuarios tener estos roles asignados en un nivel de suscripción, grupo de recursos o almacén de claves individual. Asimismo, un usuario tendrá permisos de plano de datos en todos los almacenes de claves del ámbito de la asignación de Azure RBAC. Esto elimina la necesidad de asignar permisos de directiva de acceso individuales por usuario o aplicación por almacén de claves.

* Los permisos de Azure RBAC son compatibles con Privileged Identity Management o PIM. Esto le permite configurar controles de acceso de tipo Just-in-Time para roles con privilegios, como el administrador de Key Vault. Esta es una práctica recomendada de seguridad y sigue el principio de privilegios mínimos al eliminar el acceso permanente a los almacenes de claves.

Para obtener más información sobre Azure RBAC para Key Vault, consulte los siguientes documentos:

* [Vínculo](./secure-your-key-vault.md#management-plane-and-azure-rbac) de Azure RBAC para Key Vault
* [Vínculo](../../role-based-access-control/built-in-roles.md#key-vault-administrator) de Azure RBAC para roles de Key Vault

## <a name="configure-key-vault-firewall"></a>Configuración del firewall de Key Vault

De forma predeterminada, el almacén de claves permite que el tráfico procedente de la red pública de envío llegue al almacén de claves a través de un punto de conexión público. Para obtener una capa de seguridad adicional, puede configurar el firewall de Azure Key Vault para restringir el acceso al punto de conexión público del almacén de claves.

Para habilitar el firewall del almacén de claves, haga clic en la pestaña Redes del portal del almacén de claves y seleccione el botón de radio para permitir el acceso desde: "Punto de conexión privado y redes seleccionadas". Si decide habilitar el firewall del almacén de claves, estas son las formas en que puede permitir el tráfico a través del mismo.

* Agregue direcciones IPv4 a la lista de permitidos del firewall del almacén de claves. Esta opción funciona mejor para las aplicaciones que tienen direcciones IP estáticas.

* Agregue una red virtual al firewall del almacén de claves. Esta opción funciona mejor para los recursos de Azure que tienen direcciones IP dinámicas como Virtual Machines. Puede agregar recursos de Azure a una red virtual y agregar la red virtual a la lista de elementos permitidos del firewall del almacén de claves. Esta opción usa un punto de conexión de servicio que es una dirección IP privada dentro de la red virtual. Gracias a ello, tendrá una capa adicional de protección para que no se enrute el tráfico entre el almacén de claves y la red virtual a través de la red pública de Internet. Para obtener más información acerca del punto de conexión de servicio, consulte la documentación siguiente. [vínculo](./network-security.md)

* Agregue una conexión de vínculo privado al almacén de claves. Esta opción conecta la red virtual directamente a una instancia determinada del almacén de claves, gracias a la cual el almacén de claves se coloca dentro de la red virtual. Para obtener más información sobre la configuración de una conexión de punto de conexión privado en el almacén de claves, consulte el siguiente [vínculo](./private-link-service.md).

## <a name="test-your-service-principals-ability-to-access-key-vault"></a>Prueba de la capacidad de la entidad de servicio para acceder al almacén de claves

Una vez que haya seguido todos los pasos anteriores, podrá establecer y recuperar secretos desde el almacén de claves.

### <a name="authentication-process-for-users-examples"></a>Proceso de autenticación para usuarios (ejemplos)

* Los usuarios pueden iniciar sesión en Azure Portal para usar el almacén de claves. [Inicio rápido del portal de Key Vault](./quick-create-portal.md)

* El usuario puede usar la CLI de Azure para usar el almacén de claves. [Inicio rápido de la CLI de Azure de Key Vault](./quick-create-cli.md)

* El usuario puede usar Azure PowerShell para usar el almacén de claves. [Inicio rápido de Azure PowerShell de Key Vault](./quick-create-powershell.md)

### <a name="azure-active-directory-authentication-process-for-applications-or-services-examples"></a>Proceso de autenticación de Azure Active Directory para aplicaciones o servicios (ejemplos)

* Una aplicación proporciona un secreto de cliente y un id. de cliente en una función para obtener un token de Azure Active Directory. 

* Una aplicación proporciona un certificado para obtener un token de Azure Active Directory. 

* Un recurso de Azure usa la autenticación de MSI para obtener un token de Azure Active Directory. 

* [Vínculo](../../active-directory/managed-identities-azure-resources/overview.md) para obtener más información sobre la autenticación de MSI

### <a name="authentication-process-for-application-python-example"></a>Proceso de autenticación para la aplicación (ejemplo de Python)

Use el siguiente ejemplo de código para comprobar si la aplicación puede recuperar un secreto del almacén de claves mediante la entidad de servicio configurada.

>[!NOTE]
>Este ejemplo solo se utiliza con fines de demostración y pruebas. **NO USE LA AUTENTICACIÓN DEL SECRETO DE CLIENTE EN LA PRODUCCIÓN**. Esta no es una práctica de diseño segura. Debe utilizar el certificado de cliente o la autenticación MSI como práctica recomendada.

```python
from azure.identity import ClientSecretCredential
from azure.keyvault.secrets import SecretClient

tenant_id = "{ENTER YOUR TENANT ID HERE}"                          ##ENTER AZURE TENANT ID
vault_url = "https://{ENTER YOUR VAULT NAME}.vault.azure.net/"     ##ENTER THE URL OF YOUR KEY VAULT
client_id = "{ENTER YOUR CLIENT ID HERE}"                          ##ENTER THE CLIENT ID OF YOUR SERVICE PRINCIPAL
cert_path = "{ENTER YOUR CLIENT SECRET HERE}"                      ##ENTER THE CLIENT SECRET OF YOUR SERVICE PRINCIPAL

def main():

    #AUTHENTICATION TO Azure Active Directory USING CLIENT ID AND CLIENT CERTIFICATE (GET Azure Active Directory TOKEN)
    token = ClientSecretCredential(tenant_id=tenant_id, client_id=client_id, client_secret=client_secret)

    #AUTHENTICATION TO KEY VAULT PRESENTING Azure Active Directory TOKEN
    client = SecretClient(vault_url=vault_url, credential=token)

    #CALL TO KEY VAULT TO GET SECRET
    #ENTER NAME OF A SECRET STORED IN KEY VAULT
    secret = client.get_secret('{SECRET_NAME}')

    #GET PLAINTEXT OF SECRET
    print(secret.value)

#CALL MAIN()
if __name__ == "__main__":
    main()
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la autenticación del almacén de claves, consulte el siguiente documento. [Autenticación en Azure Key Vault](./authentication.md)
