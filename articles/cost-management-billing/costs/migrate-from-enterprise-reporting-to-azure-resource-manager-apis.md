---
title: Migración desde las API de Enterprise Reporting a las de Azure Resource Manager
description: En este artículo se explican las diferencias entre las API de Reporting y las de Azure Resource Manager, qué se puede esperar al migrar a las API de Azure Resource Manager y las nuevas funcionalidades que están disponibles con las nuevas API de Azure Resource Manager.
author: bandersmsft
ms.reviewer: adwise
ms.service: cost-management-billing
ms.subservice: common
ms.topic: reference
ms.date: 03/10/2021
ms.author: banders
ms.openlocfilehash: 24fbf52c8fd0338537862a54e15e8a249541a701
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/11/2021
ms.locfileid: "102635797"
---
# <a name="migrate-from-enterprise-reporting-to-azure-resource-manager-apis"></a>Migración desde las API de Enterprise Reporting a las de Azure Resource Manager

Este artículo ayuda a los desarrolladores que han creado soluciones personalizadas mediante las [API de Azure Reporting para clientes de Enterprise](../manage/enterprise-api.md) a migrar a las API de Azure Resource Manager para Cost Management. La compatibilidad de la entidad de servicio con las nuevas API de Azure Resource Manager ya está disponible con carácter general. Las API de Azure Resource Manager están en desarrollo activo. Considere la posibilidad de migrar a ellas, en lugar de usar las API de Azure Reporting anteriores para clientes de Enterprise. Las API anteriores están en desuso. En este artículo se explican las diferencias entre las API de Reporting y las de Azure Resource Manager, qué se puede esperar al migrar a las API de Azure Resource Manager y las nuevas funcionalidades que están disponibles con las nuevas API de Azure Resource Manager.

## <a name="api-differences"></a>Diferencias entre las API

En la siguiente información se describen las diferencias entre las antiguas API de Reporting para clientes de Enterprise y las API de Azure Resource Manager más recientes.

| **Uso** | **API del Contrato Enterprise** | **API de Azure Resource Manager**: |
| --- | --- | --- |
| Authentication | Clave de API aprovisionada en el portal de Contrato Enterprise (EA) | Autenticación de Azure Active Directory (Azure AD) mediante tokens de usuario o entidades de servicio. Las entidades de servicio sustituyen a las claves de API. |
| Permisos y ámbitos | Todas las solicitudes se encuentran en el ámbito de inscripción. Las asignaciones de permisos de la clave de API determinarán si se devuelven los datos de toda la inscripción, de un departamento o de una cuenta concreta. Sin autenticación de usuario. | A los usuarios o entidades de servicio se les asigna acceso a los ámbitos de inscripción, departamento o cuenta. |
| Punto de conexión de identificador URI | https://consumption.azure.com | https://management.azure.com |
| Estado de desarrollo | En modo de mantenimiento. Camino del desuso. | En desarrollo activo |
| API disponibles | Limitado a lo que está disponible actualmente | Hay API equivalentes disponibles para reemplazar todas Ias de Contrato Enterprise. <p> También tiene a su disposición otras [API de Cost Management](/rest/api/cost-management/), entre las que se incluyen: <p> <ul><li>Presupuestos</li><li>Alertas<li>Exports</li></ul> |

## <a name="migration-checklist"></a>Lista de comprobación para la migración

- Familiarícese con las [API REST de Azure Resource Manager](/rest/api/azure).
- Determine cuáles son las API de Contrato Enterprise que usa y vea a qué API de Azure Resource Manager debe moverlas en [Asignación de API de Contrato Enterprise a las nuevas API de Azure Resource Manager](#ea-api-mapping-to-new-azure-resource-manager-apis).
- Configure la autorización y autenticación del servicio para las API de Azure Resource Manager
  - Si todavía no usa las API de Azure Resource Manager, [registre su aplicación cliente con Azure AD](/rest/api/azure/#register-your-client-application-with-azure-ad). El registro crea una entidad de servicio que puede usar para llamar a las API.
  - Asigne el acceso de la entidad de servicio a los ámbitos necesarios, tal como se describe a continuación.
  - Actualice el código de programación para que use la [autenticación de Azure AD](/rest/api/azure/#create-the-request) con su entidad de servicio.
- Pruebe las API y, después, actualice el código de programación para reemplazar las llamadas a las API de EA por llamadas a las API de Azure Resource Manager.
- Actualice el control de errores para usar códigos de errores nuevos. Estas son algunas de las consideraciones a tener en cuenta:
  - Las API de Azure Resource Manager tienen un período de tiempo de expiración de 60 segundos.
  - Las API de Azure Resource Manager tienen una limitación de velocidad vigente y si se supera aparece el error de limitación 429. Compile sus soluciones de forma que no realice demasiadas llamadas API en poco tiempo.
- Examine las otras API de Cost Management disponibles a través de Azure Resource Manager y evalúelas para usarlas más adelante. Para más información, consulte el apartado sobre el [uso de otras API de Cost Management](#use-additional-cost-management-apis).

## <a name="assign-service-principal-access-to-azure-resource-manager-apis"></a>Asignación del acceso de una entidad de servicio a las API de Azure Resource Manager

Después de crear una entidad de servicio para llamar mediante programación a las API de Azure Resource Manager, debe asignarle los permisos adecuados para autorizar y ejecutar solicitudes en Azure Resource Manager. Hay dos marcos de permisos para los distintos escenarios.

### <a name="azure-billing-hierarchy-access"></a>Acceso a la jerarquía de facturación de Azure

Para asignar permisos de entidad de servicio a los ámbitos de cuenta de facturación de Enterprise, departamentos o cuenta de inscripción, consulte [Asignación de roles a nombres de entidad de seguridad de servicio de Contrato Enterprise de Azure](../manage/assign-roles-azure-service-principals.md).

### <a name="azure-role-based-access-control"></a>Control de acceso basado en roles de Azure

La nueva compatibilidad con la entidad de servicio se extiende a ámbitos específicos de Azure, como los grupos de administración, las suscripciones y los grupos de recursos. Puede asignar permisos de entidad de servicio a estos ámbitos directamente [en Azure Portal](../../active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application) o mediante [Azure PowerShell](../../active-directory/develop/howto-authenticate-service-principal-powershell.md#assign-the-application-to-a-role).

## <a name="ea-api-mapping-to-new-azure-resource-manager-apis"></a>Asignación de API de Contrato Enterprise a las nuevas API de Azure Resource Manager

Use la tabla siguiente para identificar las API de Contrato Enterprise que usa actualmente y las API de Azure Resource Manager que va a usar en su lugar.

| **Escenario** | **API de Contrato Enterprise** | **API de Azure Resource Manager**: |
| --- | --- | --- |
| Resumen del saldo | [/balancesummary](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary) |[Microsoft.Consumption/balances](/rest/api/consumption/balances/getbybillingaccount) |
| Hoja de precios | [/pricesheet](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet) | [Microsoft.Consumption/pricesheets/default](/rest/api/consumption/pricesheet): se usa para precios negociados <p> [Retail Prices API](/rest/api/cost-management/retail-prices/azure-retail-prices): se usa para precios minoristas |
| Detalles de instancias reservadas | [/reservationdetails](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) | [Microsoft.CostManagement/generateReservationDetailsReport](/rest/api/cost-management/generatereservationdetailsreport) |
| Resumen de instancias reservadas | [/reservationsummaries](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) | [Microsoft.Consumption/reservationSummaries](/rest/api/consumption/reservationssummaries/list#reservationsummariesdailywithbillingaccountid) |
| Recomendaciones de instancias reservadas | [/SharedReservationRecommendations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation)<p>[/SingleReservationRecommendations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation) | [Microsoft.Consumption/reservationRecommendations](/rest/api/consumption/reservationrecommendations/list) |
| Cargos de instancias reservadas | [/reservationcharges](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-charges) | [Microsoft.Consumption/reservationTransactions](/rest/api/consumption/reservationtransactions/list) |

## <a name="migration-details-by-api"></a>Detalles de migración por API

En las secciones siguientes se muestran ejemplos de solicitudes de API anteriores junto con los ejemplos de las API de reemplazo nuevas.

### <a name="balance-summary-api"></a>Balance Summary API

Use los siguientes URI de solicitud al llamar a la nueva Balance Summary API. El número de inscripción debe usarse como billingAccountId.

#### <a name="supported-requests"></a>Solicitudes admitidas

[Obtención para la inscripción](/rest/api/consumption/balances/getbybillingaccount)

```json
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/providers/Microsoft.Consumption/balances?api-version=2019-10-01
```

### <a name="response-body-changes"></a>Cambios en el cuerpo de respuesta

_Cuerpo de respuesta anterior_:

```json
{
        "id": "enrollments/100/billingperiods/201507/balancesummaries",
          "billingPeriodId": 201507,
          "currencyCode": "USD",
          "beginningBalance": 0,
          "endingBalance": 1.1,
          "newPurchases": 1,
          "adjustments": 1.1,
          "utilized": 1.1,
          "serviceOverage": 1,
          "chargesBilledSeparately": 1,
          "totalOverage": 1,
          "totalUsage": 1.1,
          "azureMarketplaceServiceCharges": 1,
          "newPurchasesDetails": [
            {
              "name": "",
              "value": 1
            }
          ],
          "adjustmentDetails": [
            {
              "name": "Promo Credit",
              "value": 1.1
            },
            {
              "name": "SIE Credit",
              "value": 1.0
            }
          ]
    }

```

_Cuerpo de respuesta nuevo_:

Los mismos datos ahora están disponibles en el campo `properties` de la nueva respuesta de la API. Puede haber cambios secundarios en la ortografía de algunos de los nombres de campo.

```json
{
  "id": "/providers/Microsoft.Billing/billingAccounts/123456/providers/Microsoft.Billing/billingPeriods/201702/providers/Microsoft.Consumption/balances/balanceId1",
  "name": "balanceId1",
  "type": "Microsoft.Consumption/balances",
  "properties": {
    "currency": "USD  ",
    "beginningBalance": 3396469.19,
    "endingBalance": 2922371.02,
    "newPurchases": 0,
    "adjustments": 0,
    "utilized": 474098.17,
    "serviceOverage": 0,
    "chargesBilledSeparately": 0,
    "totalOverage": 0,
    "totalUsage": 474098.17,
    "azureMarketplaceServiceCharges": 609.82,
    "billingFrequency": "Month",
    "priceHidden": false,
    "newPurchasesDetails": [
      {
        "name": "Promo Purchase",
        "value": 1
      }
    ],
    "adjustmentDetails": [
      {
        "name": "Promo Credit",
        "value": 1.1
      },
      {
        "name": "SIE Credit",
        "value": 1
      }
    ]
  }
}

```

### <a name="price-sheet"></a>Hoja de precios

Cuando llame a la nueva Price Sheet API use los siguientes identificadores URI de solicitud.

#### <a name="supported-requests"></a>Solicitudes admitidas

 Puede llamar a la API desde los siguientes ámbitos:

- Inscripción: `providers/Microsoft.Billing/billingAccounts/{billingAccountId}`
- Suscripción: `subscriptions/{subscriptionId}`

[_Obtener para el período de facturación actual_](/rest/api/consumption/pricesheet/get)


```json
https://management.azure.com/{scope}/providers/Microsoft.Consumption/pricesheets/default?api-version=2019-10-01
```

[_Obtener para el período de facturación especificado_](/rest/api/consumption/pricesheet/getbybillingperiod)

```json
https://management.azure.com/{scope}/providers/Microsoft.Billing/billingPeriods/{billingPeriodName}/providers/Microsoft.Consumption/pricesheets/default?api-version=2019-10-01
```

#### <a name="response-body-changes"></a>Cambios en el cuerpo de respuesta

_Respuesta anterior_:

```json
      [
        {
              "id": "enrollments/57354989/billingperiods/201601/products/343/pricesheets",
              "billingPeriodId": "201704",
            "meterId": "dc210ecb-97e8-4522-8134-2385494233c0",
              "meterName": "A1 VM",
              "unitOfMeasure": "100 Hours",
              "includedQuantity": 0,
              "partNumber": "N7H-00015",
              "unitPrice": 0.00,
              "currencyCode": "USD"
        },
        {
              "id": "enrollments/57354989/billingperiods/201601/products/2884/pricesheets",
              "billingPeriodId": "201404",
            "meterId": "dc210ecb-97e8-4522-8134-5385494233c0",
              "meterName": "Locally Redundant Storage Premium Storage - Snapshots - AU East",
              "unitOfMeasure": "100 GB",
              "includedQuantity": 0,
              "partNumber": "N9H-00402",
              "unitPrice": 0.00,
              "currencyCode": "USD"
        },
        ...
    ]

```

_Nueva respuesta_:

Los datos anteriores ahora están en el campo `pricesheets` de la nueva respuesta de la API. También se proporcionan los detalles del medidor.

```json
{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Billing/billingPeriods/201702/providers/Microsoft.Consumption/pricesheets/default",
  "name": "default",
  "type": "Microsoft.Consumption/pricesheets",
  "properties": {
    "nextLink": "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/providers/microsoft.consumption/pricesheets/default?api-version=2018-01-31&$skiptoken=AQAAAA%3D%3D&$expand=properties/pricesheets/meterDetails",
    "pricesheets": [
      {
        "billingPeriodId": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Billing/billingPeriods/201702",
        "meterId": "00000000-0000-0000-0000-000000000000",
        "unitOfMeasure": "100 Hours",
        "includedQuantity": 100,
        "partNumber": "XX-11110",
        "unitPrice": 0.00000,
        "currencyCode": "EUR",
        "offerId": "OfferId 1",
        "meterDetails": {
          "meterName": "Data Transfer Out (GB)",
          "meterCategory": "Networking",
          "unit": "GB",
          "meterLocation": "Zone 2",
          "totalIncludedQuantity": 0,
          "pretaxStandardRate": 0.000
        }
      }
    ]
  }
}

```

### <a name="reserved-instance-usage-details"></a>Detalles de uso de instancias reservadas

Microsoft no trabaja activamente en API de detalles de reserva basadas en sincrónicas. Es aconsejable que pase al patrón de llamada API asincrónica compatible con SPN más reciente como parte de la migración. Las solicitudes asincrónicas controlan mejor grandes cantidades de datos y reducen los errores de tiempo de espera.

#### <a name="supported-requests"></a>Solicitudes admitidas

Use los siguientes identificadores URI de solicitud al llamar a la nueva API de detalles de reserva asincrónica. Su número de inscripción debe usarse como `billingAccountId`. Puede llamar a la API con los siguientes ámbitos:

- Inscripción: `providers/Microsoft.Billing/billingAccounts/{billingAccountId}`

#### <a name="sample-request-to-generate-a-reservation-details-report"></a>Solicitud de ejemplo para generar un informe de detalles de reserva

```json
POST 
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/providers/Microsoft.CostManagement/generateReservationDetailsReport?startDate={startDate}&endDate={endDate}&api-version=2019-11-01

```

#### <a name="sample-request-to-poll-report-generation-status"></a>Solicitud de ejemplo para sondear el estado de generación de informes

```json
GET
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/providers/Microsoft.CostManagement/reservationDetailsOperationResults/{operationId}?api-version=2019-11-01

```

#### <a name="sample-poll-response"></a>Respuesta de sondeo de ejemplo

```json
{
  "status": "Completed",
  "properties": {
    "reportUrl": "https://storage.blob.core.windows.net/details/20200911/00000000-0000-0000-0000-000000000000?sv=2016-05-31&sr=b&sig=jep8HT2aphfUkyERRZa5LRfd9RPzjXbzB%2F9TNiQ",
    "validUntil": "2020-09-12T02:56:55.5021869Z"
  }
}

```

#### <a name="response-body-changes"></a>Cambios en el cuerpo de respuesta

A continuación se muestra la respuesta de la API de detalles de reserva basada en sincrónicas anterior.

_Respuesta anterior_:

```json
{
    "reservationOrderId": "00000000-0000-0000-0000-000000000000",
    "reservationId": "00000000-0000-0000-0000-000000000000",
    "usageDate": "2018-02-01T00:00:00",
    "skuName": "Standard_F2s",
    "instanceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/resourvegroup1/providers/microsoft.compute/virtualmachines/VM1",
    "totalReservedQuantity": 18.000000000000000,
    "reservedHours": 432.000000000000000,
    "usedHours": 400.000000000000000
}

```

_Nueva respuesta_:

La nueva API crea automáticamente un archivo .csv. Vea los siguientes campos de archivo.

| **Propiedad anterior** | **Propiedad nueva** | **Notas** |
| --- | --- | --- |
|  | InstanceFlexibilityGroup | Nueva propiedad para la flexibilidad de la instancia. |
|  | InstanceFlexibilityGroup | Nueva propiedad para la flexibilidad de la instancia. |
| instanceId | InstanceName |  |
|  | Kind | Es una propiedad nueva. El valor es `None`, `Reservation` o `IncludedQuantity`. |
| reservationId | ReservationId |  |
| reservationOrderId | ReservationOrderId |  |
| reservedHours | ReservedHours |  |
| skuName | SkuName |  |
| totalReservedQuantity | TotalReservedQuantity |  |
| usageDate | UsageDate |  |
| usedHours | UsedHours |  |

### <a name="reserved-instance-usage-summary"></a>Resumen de uso de instancias reservadas

Use los siguientes identificadores URI de solicitud para llamar a la nueva Reservation Summaries API.

#### <a name="supported-requests"></a>Solicitudes admitidas

 Llame a la API con los siguientes ámbitos:

- Inscripción: `providers/Microsoft.Billing/billingAccounts/{billingAccountId}`

[_Obtener resumen de reservas diariamente_](/rest/api/consumption/reservationssummaries/list#reservationsummariesdailywithbillingaccountid)

```json
https://management.azure.com/{scope}/Microsoft.Consumption/reservationSummaries?grain=daily&$filter=properties/usageDate ge 2017-10-01 AND properties/usageDate le 2017-11-20&api-version=2019-10-01
```

[_Obtener resumen de reservas mensualmente_](/rest/api/consumption/reservationssummaries/list#reservationsummariesmonthlywithbillingaccountid)

```json
https://management.azure.com/{scope}/Microsoft.Consumption/reservationSummaries?grain=daily&$filter=properties/usageDate ge 2017-10-01 AND properties/usageDate le 2017-11-20&api-version=2019-10-01
```

#### <a name="response-body-changes"></a>Cambios en el cuerpo de respuesta

_Respuesta anterior_:

```json
[
     {
        "reservationOrderId": "00000000-0000-0000-0000-000000000000",
        "reservationId": "00000000-0000-0000-0000-000000000000",
        "skuName": "Standard_F1s",
        "reservedHours": 24,
        "usageDate": "2018-05-01T00:00:00",
        "usedHours": 23,
        "minUtilizationPercentage": 0,
        "avgUtilizationPercentage": 95.83,
        "maxUtilizationPercentage": 100
    }
]

```

_Nueva respuesta_:

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/12345/providers/Microsoft.Consumption/reservationSummaries/reservationSummaries_Id1",
      "name": "reservationSummaries_Id1",
      "type": "Microsoft.Consumption/reservationSummaries",
      "tags": null,
      "properties": {
        "reservationOrderId": "00000000-0000-0000-0000-000000000000",
        "reservationId": "00000000-0000-0000-0000-000000000000",
        "skuName": "Standard_B1s",
        "reservedHours": 720,
        "usageDate": "2018-09-01T00:00:00-07:00",
        "usedHours": 0,
        "minUtilizationPercentage": 0,
        "avgUtilizationPercentage": 0,
        "maxUtilizationPercentage": 0
      }
    }
  ]
}

```

### <a name="reserved-instance-recommendations"></a>Recomendaciones de instancias reservadas

Use los siguientes identificadores URI de solicitud para llamar a la nueva Reservation Recommendations API.

#### <a name="supported-requests"></a>Solicitudes admitidas

 Llame a la API con los siguientes ámbitos:

- Inscripción: `providers/Microsoft.Billing/billingAccounts/{billingAccountId}`
- Suscripción: `subscriptions/{subscriptionId}`
- Grupos de recursos: `subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`

[_Obtener recomendaciones_](/rest/api/consumption/reservationrecommendations/list)

Tanto las recomendaciones compartidas como las de ámbito único están disponibles a través de esta API. También puede filtrar por el ámbito como parámetro opcional de la API.

```json
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/123456/providers/Microsoft.Consumption/reservationRecommendations?api-version=2019-10-01
```

#### <a name="response-body-changes"></a>Cambios en el cuerpo de respuesta

Las recomendaciones para ámbitos compartidos y únicos se combinan en una API.

_Respuesta anterior_:

```json
[{
    "subscriptionId": "1111111-1111-1111-1111-111111111111",
    "lookBackPeriod": "Last7Days",
    "meterId": "2e3c2132-1398-43d2-ad45-1d77f6574933",
    "skuName": "Standard_DS1_v2",
    "term": "P1Y",
    "region": "westus",
    "costWithNoRI": 186.27634908960002,
    "recommendedQuantity": 9,
    "totalCostWithRI": 143.12931642978083,
    "netSavings": 43.147032659819189,
    "firstUsageDate": "2018-02-19T00:00:00"
}
]

```

_Nueva respuesta_:

```json
{
  "value": [
    {
      "id": "billingAccount/123456/providers/Microsoft.Consumption/reservationRecommendations/00000000-0000-0000-0000-000000000000",
      "name": "00000000-0000-0000-0000-000000000000",
      "type": "Microsoft.Consumption/reservationRecommendations",
      "location": "westus",
      "sku": "Standard_DS1_v2",
      "kind": "legacy",
      "properties": {
        "meterId": "00000000-0000-0000-0000-000000000000",
        "term": "P1Y",
        "costWithNoReservedInstances": 12.0785105,
        "recommendedQuantity": 1,
        "totalCostWithReservedInstances": 11.4899644807748,
        "netSavings": 0.588546019225182,
        "firstUsageDate": "2019-07-07T00:00:00-07:00",
        "scope": "Shared",
        "lookBackPeriod": "Last7Days",
        "instanceFlexibilityRatio": 1,
        "instanceFlexibilityGroup": "DSv2 Series",
        "normalizedSize": "Standard_DS1_v2",
        "recommendedQuantityNormalized": 1,
        "skuProperties": [
          {
            "name": "Cores",
            "value": "1"
          },
          {
            "name": "Ram",
            "value": "1"
          }
        ]
      }
    },
   ]
}

```

### <a name="reserved-instance-charges"></a>Cargos de instancias reservadas

Use los siguientes identificadores URI de solicitud para llamar a la nueva Reserved Instance Charges API.

#### <a name="supported-requests"></a>Solicitudes admitidas

[_Obtener cargos de reserva por rango de fechas_](/rest/api/consumption/reservationtransactions/list)

```json
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/providers/Microsoft.Consumption/reservationTransactions?$filter=properties/eventDate+ge+2020-05-20+AND+properties/eventDate+le+2020-05-30&api-version=2019-10-01
```

#### <a name="response-body-changes"></a>Cambios en el cuerpo de respuesta

_Respuesta anterior_:

```json
[
    {
        "purchasingEnrollment": "string",
        "armSkuName": "Standard_F1s",
        "term": "P1Y",
        "region": "eastus",
        "PurchasingsubscriptionGuid": "00000000-0000-0000-0000-000000000000",
        "PurchasingsubscriptionName": "string",
        "accountName": "string",
        "accountOwnerEmail": "string",
        "departmentName": "string",
        "costCenter": "",
        "currentEnrollment": "string",
        "eventDate": "string",
        "reservationOrderId": "00000000-0000-0000-0000-000000000000",
        "description": "Standard_F1s eastus 1 Year",
        "eventType": "Purchase",
        "quantity": int,
        "amount": double,
        "currency": "string",
        "reservationOrderName": "string"
    }
]

```
_Nueva respuesta_:

```json
{
  "value": [
    {
      "id": "/billingAccounts/123456/providers/Microsoft.Consumption/reservationtransactions/201909091919",
      "name": "201909091919",
      "type": "Microsoft.Consumption/reservationTransactions",
      "tags": {},
      "properties": {
        "eventDate": "2019-09-09T19:19:04Z",
        "reservationOrderId": "00000000-0000-0000-0000-000000000000",
        "description": "Standard_DS1_v2 westus 1 Year",
        "eventType": "Cancel",
        "quantity": 1,
        "amount": -21,
        "currency": "USD",
        "reservationOrderName": "Transaction-DS1_v2",
        "purchasingEnrollment": "123456",
        "armSkuName": "Standard_DS1_v2",
        "term": "P1Y",
        "region": "westus",
        "purchasingSubscriptionGuid": "11111111-1111-1111-1111-11111111111",
        "purchasingSubscriptionName": "Infrastructure Subscription",
        "accountName": "Microsoft Infrastructure",
        "accountOwnerEmail": "admin@microsoft.com",
        "departmentName": "Unassigned",
        "costCenter": "",
        "currentEnrollment": "123456",
        "billingFrequency": "recurring"
      }
    },
  ]
}

```

## <a name="use-additional-cost-management-apis"></a>Uso de API adicionales de Cost Management

Después de migrar a las Azure Resource Manager API en escenarios de informes existentes, también puede usar muchas otras API. Las API también están disponibles a través de Azure Resource Manager y se pueden automatizar mediante la autenticación basada en la entidad de servicio. A continuación se muestra un resumen rápido de las nuevas funcionalidades que puede usar.

- [Presupuestos](/rest/api/consumption/budgets/createorupdate): se usa para establecer umbrales para supervisar de forma proactiva los costos, alertar a las partes interesadas pertinentes y automatizar acciones en respuesta a las infracciones del umbral.

- [Alertas](/rest/api/cost-management/alerts): se usa para ver información de alertas, entre las que se incluyen las alertas de presupuesto, las alertas de factura, las alertas de crédito y las alertas de cuota.

- [Exports](/rest/api/cost-management/exports): se usa para programar la exportación de datos recurrente de sus cargos a la cuenta de Azure Storage que prefiera. Es la solución recomendada para clientes con una gran presencia de Azure que quieren analizar sus datos y usarlos en sus propios sistemas internos.

## <a name="next-steps"></a>Pasos siguientes

- Familiarícese con las [API REST de Azure Resource Manager](/rest/api/azure).
- Si fuera necesario, determine cuáles son las API de Contrato Enterprise usa y vea a qué API de Azure Resource Manager debe mover al [asignar API de EA a las nuevas API de Azure Resource Manager](#ea-api-mapping-to-new-azure-resource-manager-apis).
- Si todavía no usa las API de Azure Resource Manager, [registre su aplicación cliente con Azure AD](/rest/api/azure/#register-your-client-application-with-azure-ad).
- Si fuera necesario, actualice cualquier código de programación para que use [la autenticación de Azure AD](/rest/api/azure/#create-the-request) con su entidad de servicio.