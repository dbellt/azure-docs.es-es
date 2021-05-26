---
title: 'Inicio rápido: Nueva asignación de directivas con Terraform'
description: En este inicio rápido se usan Terraform y la sintaxis de HCL para crear una asignación de directivas para identificar los recursos no compatibles.
ms.date: 03/31/2021
ms.topic: quickstart
ms.openlocfilehash: c9e33fc8070adc2685e26272565d95716c0df8db
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110094769"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-using-terraform"></a>Inicio rápido: Creación de una asignación de directivas para identificar los recursos no compatibles mediante Terraform

El primer paso para entender el cumplimiento en Azure es identificar el estado de sus recursos.
Esta guía de inicio rápido lo guiará por el proceso de creación de una asignación de directiva para identificar las máquinas virtuales que no están usando discos administrados.

Al finalizar este proceso, habrá identificado correctamente máquinas virtuales que no utilizan discos administrados. _No son compatibles_ con la asignación de directiva.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.
- [Terraform](https://www.terraform.io/) versión 0.12.0 o superior configurado en el entorno.
  Para obtener instrucciones, consulte [Configuración de Terraform mediante Azure Cloud Shell](/azure/developer/terraform/get-started-cloud-shell).
- Para realizar este inicio rápido es necesaria la versión 2.13.0 o superior de la CLI de Azure. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-the-terraform-configuration-variable-and-output-file"></a>Creación de la configuración, la variable y el archivo de salida de Terraform

En este inicio rápido, creará una asignación de directiva y asignará la definición **Auditoría de máquinas virtuales que no usan discos administrados** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Esta definición de directiva identifica los recursos que no cumplen las condiciones establecidas en la definición de directiva.

En primer lugar, configure Terraform, la variable y los archivos de salida. Los recursos de Terraform para Azure Policy usan el [proveedor de Azure](https://www.terraform.io/docs/providers/azurerm/index.html).

1. Cree una nueva carpeta llamada `policy-assignment` y vaya a ese directorio.

1. Cree el archivo `main.tf` con el siguiente código:

   ```hcl
   provider "azurerm" {
       version = "~>2.0"
       features {}
   }

   resource "azurerm_policy_assignment" "auditvms" {
       name = "audit-vm-manageddisks"
       scope = var.cust_scope
       policy_definition_id = "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d"
       description = "Shows all virtual machines not using managed disks"
       display_name = "Audit VMs without managed disks Assignment"
   }
   ```

1. Cree el archivo `variables.tf` con el siguiente código:

   ```hcl
   variable "cust_scope" {
       default = "{scope}"
   }
   ```

   Un ámbito determina en qué recursos o agrupación de recursos se implementa la asignación de directiva. Podría abarcar desde un grupo de administración a un recurso individual. Asegúrese de reemplazar `{scope}` por uno de los siguientes patrones:

   - Suscripción: `/subscriptions/{subscriptionId}`
   - Grupos de recursos: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`
   - Recurso: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`

1. Cree el archivo `output.tf` con el siguiente código:

   ```hcl
   output "assignment_id" {
       value = azurerm_policy_assignment.auditvms.id
   }
   ```

## <a name="initialize-terraform-and-create-plan"></a>Inicialización de Terraform y creación de un plan

A continuación, inicialice Terraform para descargar los proveedores necesarios y, a continuación, cree un plan.

1. Ejecute el comando [terraform init](https://www.terraform.io/docs/commands/init.html). Este comando descarga los módulos de Azure necesarios para crear los recursos de Azure en la configuración de Terraform.

   ```bash
   terraform init
   ```

   :::image type="content" source="./media/assign-policy-terraform/terraform-initialize.png" alt-text="Captura de pantalla de la ejecución del comando terraform init que muestra la descarga del módulo azurerm y un mensaje de operación correcta.":::

1. Autentique Terraform con la [CLI de Azure](/cli/azure/). Para más información, consulte [Proveedor de Azure: autenticación mediante la CLI de Azure](https://www.terraform.io/docs/providers/azurerm/guides/azure_cli.html).

   ```azurecli
   az login
   ```

1. Cree el plan de ejecución con el comando [terraform plan](https://www.terraform.io/docs/commands/plan.html) y el parámetro **out**.

   ```bash
   terraform plan -out assignment.tfplan
   ```

   :::image type="content" source="./media/assign-policy-terraform/terraform-plan-out.png" alt-text="Captura de pantalla de la ejecución del comando terraform plan y el parámetro out para mostrar el recurso de Azure que se crearía.":::

   > [!NOTE]
   > Para obtener información sobre la conservación de los planes de ejecución y la seguridad, consulte [Plan de Terraform: Advertencia de seguridad](https://www.terraform.io/docs/commands/plan.html#security-warning).

## <a name="apply-the-terraform-execution-plan"></a>Aplicación de un plan de ejecución de Terraform

Por último, aplique el plan de ejecución.

Ejecute el comando [terraform apply](https://www.terraform.io/docs/commands/apply.html) y especifique el archivo `assignment.tfplan` ya creado.

```bash
terraform apply assignment.tfplan
```

:::image type="content" source="./media/assign-policy-terraform/terraform-apply.png" alt-text="Captura de pantalla de la ejecución del comando terraform apply y la creación de recursos resultante.":::

Con el mensaje "Apply complete! Recursos: 1 added, 0 changed, 0 destroyed." (Aplicación finalizada: 1 agregado, 0 cambiado, 0 eliminado), ya se habrá creado la asignación de directivas. Como hemos definido el archivo `outputs.tf`, también se devuelve el valor de _assignment\_id_.

## <a name="identify-non-compliant-resources"></a>Identificación de recursos sin compatibilidad

Para ver los recursos que no son compatibles con esta nueva asignación, utilice el valor de _assignment\_id_ que devuelve `terraform apply`. Con dicho valor, ejecute el siguiente comando para obtener los identificadores de los recursos no compatibles que se generan en un archivo JSON:

```console
armclient post "/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$filter=IsCompliant eq false and PolicyAssignmentId eq '<policyAssignmentID>'&$apply=groupby((ResourceId))" > <json file to direct the output with the resource IDs into>
```

Los resultados deben tener una apariencia similar al ejemplo siguiente:

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
    "@odata.count": 3,
    "value": [{
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachineId>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine2Id>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionName>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine3ID>"
        }

    ]
}
```

Son comparables a lo que normalmente vería en **Recursos no compatibles**, en la vista de Azure Portal.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar la asignación creada, use la CLI de Azure o revierta el plan de ejecución de Terraform con `terraform destroy`.

- Azure CLI

  ```azurecli-interactive
  az policy assignment delete --name 'audit-vm-manageddisks' --scope '/subscriptions/<subscriptionID>/<resourceGroupName>'
  ```

- Terraform

  ```bash
  terraform destroy assignment.tfplan
  ```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se asigna una definición de directiva para identificar los recursos incompatibles en el entorno de Azure.

Para más información sobre la asignación de directivas para garantizar la compatibilidad de los nuevos recursos, continúe con el tutorial para:

> [!div class="nextstepaction"]
> [Crear y administrar directivas](./tutorials/create-and-manage.md)
