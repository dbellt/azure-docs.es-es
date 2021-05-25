---
title: 'Inicio rápido: Nueva asignación de directivas con JavaScript'
description: En este inicio rápido, se usa JavaScript para crear una asignación de Azure Policy para identificar recursos no compatibles.
ms.date: 03/31/2021
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: c0ff6ec2457c0669d055ef2da50032a99b0029c6
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108772574"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-using-javascript"></a>Inicio rápido: Creación de una asignación de directivas para identificar los recursos no compatibles mediante JavaScript

El primer paso para entender el cumplimiento en Azure es identificar el estado de sus recursos. En esta guía de inicio rápido, se crea una asignación de directiva para identificar máquinas virtuales que no usan discos administrados. Cuando haya finalizado, podrá identificar máquinas virtuales que _no estén conformes_.

La biblioteca de JavaScript se usa para administrar los recursos de Azure desde la línea de comandos o en scripts. En esta guía se explica cómo usar la biblioteca JavaScript para crear una asignación de directiva.

## <a name="prerequisites"></a>Requisitos previos

- **Suscripción de Azure**: si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

- **Node.js**: se requiere [Node.js](https://nodejs.org/) versión 12 o posterior.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-policy-libraries"></a>Adición de las bibliotecas de Policy

Para que JavaScript pueda trabajar con Azure Policy, se deben agregar las bibliotecas. Estas bibliotecas funcionan siempre que se pueda usar JavaScript, incluido [bash en Windows 10](/windows/wsl/install-win10).

1. Configure un nuevo proyecto de Node.js ejecutando el siguiente comando.

   ```bash
   npm init -y
   ```

1. Agregue una referencia a la biblioteca de yargs.

   ```bash
   npm install yargs
   ```

1. Agregue una referencia a las bibliotecas de Azure Policy.

   ```bash
   # arm-policy is for working with Azure Policy objects such as definitions and assignments
   npm install @azure/arm-policy

   # arm-policyinsights is for working with Azure Policy compliance data such as events and states
   npm install @azure/arm-policyinsights
   ```

1. Agregue una referencia a la biblioteca de autenticación de Azure.

   ```bash
   npm install @azure/ms-rest-nodeauth
   ```

   > [!NOTE]
   > Compruebe en _package.json_ que `@azure/arm-policy` es de la versión **3.1.0** o superior, `@azure/arm-policyinsights` de la versión **3.2.0** o superior, y `@azure/ms-rest-nodeauth` de la versión **3.0.5** o superior.

## <a name="create-a-policy-assignment"></a>Creación de una asignación de directiva

En este inicio rápido, creará una asignación de directiva y asignará la definición **Auditoría de máquinas virtuales que no usan discos administrados** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Esta definición de directiva identifica los recursos que no cumplen las condiciones establecidas en la definición de directiva.

1. Cree un archivo denominado _policyAssignment.js_ y escriba el siguiente código.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const policyObjects = require("@azure/arm-policy");

   if (argv.subID && argv.name && argv.displayName && argv.policyDefID && argv.scope && argv.description) {

       const createAssignment = async () => {
           const credentials = await authenticator.interactiveLogin();
           const client = new policyObjects.PolicyClient(credentials, argv.subID);
           const assignments = new policyObjects.PolicyAssignments(client);

           const result = await assignments.create(
               argv.scope,
               argv.name,
               {
                   displayName: argv.displayName,
                   policyDefinitionId: argv.policyDefID,
                   description: argv.description
               }
           );
           console.log(result);
       };

       createAssignment();
   }
   ```

1. En el terminal, escriba el siguiente comando:

   ```bash
   node policyAssignment.js `
      --subID "{subscriptionId}" `
      --name "audit-vm-manageddisks" `
      --displayName "Audit VMs without managed disks Assignment" `
      --policyDefID "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d" `
      --description "Shows all virtual machines not using managed disks" `
      --scope "{scope}"
   ```

Los comandos anteriores usan la siguiente información:

- **subID**: el identificador de la suscripción para el contexto de autenticación. Asegúrese de reemplazar `{subscriptionId}` por su suscripción.
- **name**: el nombre único para el objeto de asignación de directiva. En el ejemplo anterior se usa _audit-vm-manageddisks_.
- **displayName**: nombre para mostrar de la asignación de directiva. En este caso, usará _Auditoría de máquinas virtuales sin discos administrados_.
- **policyDefID**: la ruta de acceso de la definición de directiva, según la opción utilizada para crear la asignación. En este caso, es el identificador de la definición de directiva _Auditoría de máquinas virtuales que no usan discos administrados_.
- **description**: una explicación más detallada de lo que hace la directiva o de por qué se asigna a este ámbito.
- **scope**: un ámbito determina en qué recursos o agrupación de recursos se aplica la asignación de directiva. Podría abarcar desde un grupo de administración a un recurso individual. Asegúrese de reemplazar `{scope}` por uno de los siguientes patrones:
  - Grupo de administración: `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Suscripción: `/subscriptions/{subscriptionId}`
  - Grupos de recursos: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`
  - Recurso: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`

Ahora ya está listo para identificar los recursos no compatibles a fin de conocer el estado de cumplimiento de su entorno.

## <a name="identify-non-compliant-resources"></a>Identificación de recursos sin compatibilidad

Ahora que se ha creado la asignación de directiva, puede identificar los recursos que no son compatibles.

1. Cree un nuevo archivo denominado _policyState.js_ y escriba el código siguiente.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const policyInsights = require("@azure/arm-policyinsights");

   if (argv.subID && argv.name) {

       const getStates = async () => {

           const credentials = await authenticator.interactiveLogin();
           const client = new policyInsights.PolicyInsightsClient(credentials);
           const policyStates = new policyInsights.PolicyStates(client);
           const result = await policyStates.listQueryResultsForSubscription(
               "latest",
               argv.subID,
               {
                   queryOptions: {
                       filter: "IsCompliant eq false and PolicyAssignmentId eq '" + argv.name + "'",
                       apply: "groupby((ResourceId))"
                   }
               }
           );
           console.log(result);
       };

       getStates();
   }
   ```

1. En el terminal, escriba el siguiente comando:

   ```bash
   node policyState.js --subID "{subscriptionId}" --name "audit-vm-manageddisks"
   ```

Reemplace `{subscriptionId}` por la suscripción en que desea ver los resultados de cumplimiento de la asignación de directiva denominada "audit-vm-manageddisks" que hemos creado en los pasos anteriores. Para obtener una lista de otros ámbitos y maneras de resumir los datos, consulte los métodos de [PolicyStates*](/javascript/api/@azure/arm-policyinsights/).

Los resultados deben tener una apariencia similar al ejemplo siguiente:

```output
{
    'additional_properties': {
        '@odata.nextLink': None
    },
    'odatacontext': 'https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest',
    'odatacount': 12,
    'value': [{data}]
}
```

Los resultados coinciden con lo que ve en la pestaña de **cumplimiento de recursos** de una asignación de directiva en la vista de Azure Portal.

## <a name="clean-up-resources"></a>Limpieza de recursos

- Elimine la asignación de directiva _Audit VMs without managed disks Assignment_ (Auditar VM sin asignación de discos administrados) a través del portal. La definición de directiva está integrada, por lo que no hay que quitar ninguna definición.

- Si desea quitar las bibliotecas instaladas de la aplicación, ejecute el siguiente comando.

  ```bash
  npm uninstall @azure/arm-policy @azure/arm-policyinsights @azure/ms-rest-nodeauth yargs
  ```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se asigna una definición de directiva para identificar los recursos incompatibles en el entorno de Azure.

Para más información sobre la asignación de definiciones de directivas para asegurarse de la compatibilidad de los nuevos recursos, continúe con el tutorial para:

> [!div class="nextstepaction"]
> [Crear y administrar directivas](./tutorials/create-and-manage.md)
