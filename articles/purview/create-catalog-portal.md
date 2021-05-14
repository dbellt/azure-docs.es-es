---
title: 'Inicio rápido: Creación de una cuenta de Azure Purview en Azure Portal (versión preliminar)'
description: En este inicio rápido se describe cómo crear una cuenta de Azure Purview y configurar los permisos necesarios para empezar a usarla.
author: nayenama
ms.author: nayenama
ms.date: 10/23/2020
ms.topic: quickstart
ms.service: purview
ms.subservice: purview-data-catalog
ms.custom:
- mode-portal
ms.openlocfilehash: 158eed6d287fa384023defbb20a7a1c39ea3d838
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107728597"
---
# <a name="quickstart-create-an-azure-purview-account-in-the-azure-portal"></a>Inicio rápido: Creación de una cuenta de Azure Purview en Azure Portal

> [!IMPORTANT]
> Azure Purview se encuentra actualmente en VERSIÓN PRELIMINAR. Los [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) incluyen términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado con disponibilidad general.

En este inicio rápido, creará una cuenta de Azure Purview.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Su propio [inquilino de Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

* La cuenta debe tener permiso para crear recursos en la suscripción

* Si **Azure Policy** impide a todas las aplicaciones crear una **cuenta de Storage** y un **espacio de nombres de EventHub**, debe crear una excepción de directiva mediante etiqueta, que se especificará durante el proceso de creación de una cuenta de Purview. La razón principal es que con cada cuenta de Purview creada se debe crear un grupo de recursos administrado y dentro de este, una cuenta de Storage y un espacio de nombres de EventHub.

    > [!important]
    > No es preciso realizar este paso si no se tiene Azure Policy o si una instancia de Azure Policy no bloquea la creación de una **cuenta de Storage** y un **espacio de nombres de EventHub**.

    1. Vaya a Azure Portal y busque **Directiva**.
    1. Consulte el tutorial [Creación de una definición de directiva personalizada](../governance/policy/tutorials/create-custom-policy-definition.md) o modifique la directiva existente para agregar dos excepciones con el operador `not` y la etiqueta `resourceBypass`:

        ```json
        {
          "mode": "All",
          "policyRule": {
            "if": {
              "anyOf": [
              {
                "allOf": [
                {
                  "field": "type",
                  "equals": "Microsoft.Storage/storageAccounts"
                },
                {
                  "not": {
                    "field": "tags['<resourceBypass>']",
                    "exists": true
                  }
                }]
              },
              {
                "allOf": [
                {
                  "field": "type",
                  "equals": "Microsoft.EventHub/namespaces"
                },
                {
                  "not": {
                    "field": "tags['<resourceBypass>']",
                    "exists": true
                  }
                }]
              }]
            },
            "then": {
              "effect": "deny"
            }
          },
          "parameters": {}
        }
        ```
        
        > [!Note]
        > La etiqueta puede ser cualquier cosa que haya junto a `resourceBypass` y el usuario es quien decide si define un valor al crear Purview en los pasos posteriores, siempre que la directiva pueda detectar la etiqueta.

        :::image type="content" source="./media/create-catalog-portal/policy-definition.png" alt-text="Captura de pantalla que muestra cómo crear una definición de directiva.":::

    1. [Cree una asignación de directiva](../governance/policy/assign-policy-portal.md) mediante la directiva personalizada creada.

        [ ![Captura de pantalla que muestra cómo crear una asignación de directiva](./media/create-catalog-portal/policy-assignment.png)](./media/create-catalog-portal/policy-assignment.png#lightbox)

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

## <a name="configure-your-subscription"></a>Configuración de su suscripción

Si fuera necesario, siga estos pasos para configurar una suscripción para que Azure Purview se pueda ejecutar en ella:

   1. En Azure Portal, seleccione **Suscripciones**.

   1. En la lista de suscripciones, seleccione la que desee usar. Se requiere el permiso de acceso administrativo para la suscripción.

      :::image type="content" source="./media/create-catalog-portal/select-subscription.png" alt-text="Captura de pantalla que muestra cómo seleccionar una suscripción en Azure Portal.":::

   1. Para la suscripción, seleccione **Proveedores de recursos**. En el panel **Proveedores de recursos**, busque los tres proveedores de recursos siguientes y regístrelos: 
       1. **Microsoft.Purview**
       1. **Microsoft.Storage**
       1. **Microsoft.EventHub** 
      
      Si no están registrados, regístrelos ahora, para lo que debe seleccionar **Registrar**.

      :::image type="content" source="./media/create-catalog-portal/register-purview-resource-provider.png" alt-text="Captura de pantalla que muestra cómo registrar el proveedor de recursos Microsoft.Purview en Azure Portal.":::

## <a name="create-an-azure-purview-account-instance"></a>Creación de una instancia de una cuenta de Azure Purview

1. Vaya a la página **Purview accounts** (Cuentas de Purview) en Azure Portal y seleccione **Agregar** para crear una cuenta de Azure Purview. También puede ir a Marketplace, buscar **Purview Accounts** (Cuentas de Purview) y seleccionar **Crear**. Tenga en cuentas que las cuentas de Azure Purview deben agregarse de una en una.

   :::image type="content" source="./media/create-catalog-portal/add-purview-instance.png" alt-text="Captura de pantalla que muestra cómo crear una instancia de una cuenta de Azure Purview en Azure Portal.":::

    > [!Note] 
    > Azure Purview no permite mover su cuenta entre regiones. Puede obtener más información al respecto en [Compatibilidad con la operación de traslado para recursos](../azure-resource-manager/management/move-support-resources.md).

1. En la pestaña **Aspectos básicos**, siga estos pasos:
    1. Seleccione un **grupo de recursos**.
    1. Escriba un **nombre de cuenta de Purview** para el catálogo. No se permiten espacios ni símbolos.
    1. Elija un valor en **Location** (Ubicación) y, después, seleccione **Next: Configuration** (Siguiente: Configuración).
1. En la pestaña **Configuration** (Configuración) seleccione el **tamaño de plataforma** que desee, los valores permitidos son 4 unidades de capacidad (CU) y 16 CU. Seleccione **Siguiente: Etiquetas**.
1. En la pestaña **Tags** (Etiquetas), puede agregar una o varias etiquetas. Estas etiquetas solo se usan en Azure Portal, no en Azure Purview. 

    > [!Note] 
    > Si tiene **Azure Policy** y necesita agregar una excepción, como se hace en **Requisitos previos**, debe agregar la etiqueta correcta. Por ejemplo, puede agregar la etiqueta `resourceBypass`: :::image type="content" source="./media/create-catalog-portal/add-purview-tag.png" alt-text="Agregar una etiqueta a la cuenta de Purview.":::

1. Seleccione **Review & Create** (Revisar y crear) y, después, seleccione **Create** (Crear). La creación tarda unos minutos en completarse. La instancia de la cuenta de Azure Purview recién creada aparece en la lista de la página **Purview accounts** (Cuentas de Purview).
1. Cuando se complete el aprovisionamiento de la nueva cuenta, seleccione **Go to resource** (Ir al recurso).

    > [!Note]
    > Si no se realizó el aprovisionamiento y el estado es `Conflict`, significa que hay alguna directiva de Azure que impide que Purview cree una **cuenta de Storage** y un **espacio de nombres de EventHub**. Para agregar excepciones, debe seguir los pasos de **Requisitos previos**.
    > :::image type="content" source="./media/create-catalog-portal/purview-conflict-error.png" alt-text="Mensaje de error de conflicto de Purview":::

1. Seleccione **Launch purview account** (Iniciar cuenta de Purview).

   :::image type="content" source="./media/use-purview-studio/launch-from-portal.png" alt-text="Captura de pantalla de la opción seleccionada para iniciar el catálogo de la cuenta de Azure Purview.":::

## <a name="add-a-security-principal-to-a-data-plane-role"></a>Adición de una entidad de seguridad a un rol del plano de datos

Antes de que usted o su equipo puedan empezar a usar Azure Purview, es preciso agregar una o varias entidades de seguridad a uno de los roles predefinidos del plano de datos: **Purview Data Reader** (Lector de datos de Purview), **Purview Data Curator** (Administrador de datos de Purview) o **Purview Data Source Administrator** (Administrador de orígenes de datos de Purview). Para más información sobre los permisos del catálogo de datos de Azure Purview, consulte [Permisos del catálogo](catalog-permissions.md).

Para agregar una entidad de seguridad al rol del plano datos **Purview Data Curator** (Administrador de datos de Purview) en una cuenta de Azure Purview:

1. En Azure Portal, vaya a la página [**Purview accounts**](https://aka.ms/purviewportal) (Cuentas de Purview).

1. Seleccione la cuenta de Azure Purview que quiere modificar.

1. En la página **Purview account** (Cuenta de Purview), seleccione la pestaña **Access control (IAM)** (Control de acceso [IAM]).

1. Haga clic en **+Agregar**.

Si al hacer clic en Agregar aparecen dos opciones y aparecen ambas marcadas (deshabilitadas), significa que no tiene los permisos adecuados para agregar ningún usuario a un rol de plano de datos en la cuenta de Azure Purview. Debe encontrar a un propietario o administrador de acceso de usuario, o bien a alguien con autoridad para asignar roles en su cuenta de Azure Purview. Para buscar a la persona adecuada, seleccione la pestaña **Role assignments** (Asignaciones de roles), desplácese hacia abajo para buscar Propietario o Administrador de acceso de usuario y póngase en contacto con esas personas.

1. Seleccione **Agregar asignación de roles**.

1. En Role type (Tipo de rol), escriba **Purview Data Curator Role** (Rol Administrador de datos de Purview) o **Purview Data Source Administrator Role** (Administrador de orígenes de datos de Purview), en función de la entidad de seguridad que se va a usar (para más información, consulte [Permisos del catálogo](catalog-permissions.md) y [Objetos de aplicación y de entidad de servicio de Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md)).

1. En **Assign access to** (Asignar acceso a), deje el valor predeterminado, **User, group, or service principal** (Usuario, grupo o entidad de servicio).

1. En **Select** (Seleccionar), escriba el nombre del usuario, grupo de Azure Active Directory o entidad de servicio que desee asignar y haga clic en su nombre en el panel de resultados.

1. Haga clic en **Guardar**.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita esta cuenta de Azure Purview, siga estos pasos para eliminarla:

1. En Azure Portal, vaya a la página **Purview accounts** (Cuentas de Purview).

2. Seleccione la cuenta de Azure Purview que creó al principio de este inicio rápido. Seleccione **Delete** (Eliminar), escriba el nombre de la cuenta y, seleccione **Delete** (Eliminar).

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cuenta de Azure Purview.

Vaya al siguiente artículo, donde aprenderá a permitir que los usuarios accedan a su cuenta de Azure Purview. 

> [!div class="nextstepaction"]
> [Adición de usuarios a una cuenta de Azure Purview](catalog-permissions.md)
