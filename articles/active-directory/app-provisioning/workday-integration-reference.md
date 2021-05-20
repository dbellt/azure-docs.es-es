---
title: Referencia de la integración de Azure Active Directory y Workday
description: Inmersión técnica profunda en el aprovisionamiento controlado por Workday-HR en Azure Active Directory
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: reference
ms.workload: identity
ms.date: 05/11/2021
ms.author: kenwith
ms.reviewer: arvinh, chmutali
ms.openlocfilehash: 66ff44c2ed1cdfe8b1381e4cbfb68fe03831bc79
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109784600"
---
# <a name="how-azure-active-directory-provisioning-integrates-with-workday"></a>Integración del aprovisionamiento de Azure Active Directory con Workday

El [servicio de aprovisionamiento de usuarios de Azure Active Directory](../app-provisioning/user-provisioning.md) se integra con [Workday HCM](https://www.workday.com) para administrar el ciclo de vida de identidad de los usuarios. Azure Active Directory ofrece tres integraciones precompiladas: 

* [Aprovisionamiento de usuarios de Workday a Active Directory en el entorno local](../saas-apps/workday-inbound-tutorial.md)
* [Aprovisionamiento de usuarios de Workday a Azure Active Directory](../saas-apps/workday-inbound-cloud-only-tutorial.md)
* [Escritura diferida de Workday](../saas-apps/workday-writeback-tutorial.md)

En este artículo se explica cómo funciona la integración y cómo puede personalizar el comportamiento de aprovisionamiento para diferentes escenarios de recursos humanos. 

## <a name="establishing-connectivity"></a>Establecimiento de la conectividad 

### <a name="restricting-workday-api-access-to-azure-ad-endpoints"></a>Restricción del acceso de la API de Workday a puntos de conexión de Azure AD
El servicio de aprovisionamiento de Azure AD usa la autenticación básica para conectarse a los puntos de conexión de la API de Workday Web Services.  

Para proteger aún más la conectividad entre el servicio de aprovisionamiento de Azure AD y Workday, puede restringir el acceso para que el usuario del sistema de integración designado solo tenga acceso a las API de Workday desde intervalos de IP permitidos de Azure AD. Póngase en contacto con el administrador de Workday para completar la siguiente configuración en su inquilino de Workday. 

1. Descargue los [intervalos IP más recientes](https://www.microsoft.com/download/details.aspx?id=56519) para la nube pública de Azure. 
1. Abra el archivo y busque la etiqueta **AzureActiveDirectory**. 

   >[!div class="mx-imgBorder"] 
   >![Intervalo IP de Azure AD](media/sap-successfactors-integration-reference/azure-active-directory-ip-range.png)

1. Copie todos los intervalos de direcciones IP que aparecen en el elemento *addressPrefixes* y use el intervalo para crear una lista de direcciones IP.
1. Inicie sesión en el portal de administración de Workday. 
1. Acceda a la tarea **Mantener intervalos IP** para crear un nuevo intervalo de direcciones IP para los centros de datos de Azure. Especifique los intervalos IP (mediante la notación CIDR) como una lista separada por comas.  
1. Acceda a la tarea **Administrar directivas de autenticación** para crear una nueva directiva de autenticación. En la directiva de autenticación, use la lista de permitidos de autenticación para especificar el intervalo IP de Azure AD y el grupo de seguridad a los que se permitirá el acceso desde este intervalo IP. Guarde los cambios. 
1. Acceda a la tarea **Activar todos los cambios de directivas de autenticación pendientes** para confirmar los cambios.

### <a name="limiting-access-to-worker-data-in-workday-using-constrained-security-groups"></a>Limitación del acceso a los datos de trabajadores de Workday mediante grupos de seguridad restringidos

Los pasos predeterminados para [configurar el usuario del sistema de integración de Workday](../saas-apps/workday-inbound-tutorial.md#configure-integration-system-user-in-workday) conceden acceso para recuperar todos los usuarios del inquilino de Workday. En ciertos escenarios de integración, puede que desee limitar el acceso, de modo que los usuarios que pertenecen únicamente a determinadas organizaciones de supervisión sean devueltos por la llamada API de Get_Workers y procesados por el conector de Azure AD para Workday. 

Puede cumplir este requisito si trabaja con el administrador de Workday y configura los grupos de seguridad del sistema de integración restringido. Para más información sobre cómo hacerlo, consulte [este artículo de la comunidad de Workday](https://community.workday.com/forums/customer-questions/620393) (*se requieren credenciales de inicio de sesión en la comunidad de Workday para acceder a este artículo*)

Esta estrategia de limitar el acceso mediante ISSG (grupos de seguridad del sistema de integración) restringidos es especialmente útil en los escenarios siguientes: 
* **Escenario de lanzamiento por fases**: tiene un inquilino de Workday de gran tamaño y planea realizar un lanzamiento por fases de Workday en el aprovisionamiento automatizado de Azure AD. En este escenario, en lugar de excluir a los usuarios que no están en el ámbito de la fase actual con filtros de ámbito de Azure AD, se recomienda configurar ISSG restringidos para que solo los trabajadores en el ámbito estén visibles para Azure AD.
* **Escenario con varios trabajos de aprovisionamiento**: tiene un inquilino de Workday de gran tamaño y varios dominios de AD, cada uno de los cuales respalda a una unidad de negocio, división o compañía diferente. Para admitir esta topología, le gustaría ejecutar varios trabajos de aprovisionamiento de Workday en Azure AD con cada trabajo aprovisionando un conjunto específico de trabajadores. En este escenario, en lugar de usar los filtros de ámbito de Azure AD para excluir los datos de trabajadores, se recomienda configurar ISSG restringidos para que solo los datos de trabajadores pertinentes estén visibles en Azure AD. 

### <a name="workday-test-connection-query"></a>Consulta de conexión de prueba de Workday

Para probar la conectividad con Workday, Azure AD envía la siguiente solicitud *Get_Workers* de Workday Web Services. 

```XML
<!-- Test connection query tries to retrieve one record from the first page -->
<!-- Replace version with Workday Web Services version present in your connection URL -->
<!-- Replace timestamps below with the UTC time corresponding to the test connection event -->
<Get_Workers_Request p1:version="v21.1" xmlns:p1="urn:com.workday/bsvc" xmlns="urn:com.workday/bsvc">
  <p1:Request_Criteria>
    <p1:Transaction_Log_Criteria_Data>
      <p1:Transaction_Date_Range_Data>
        <p1:Updated_From>2021-01-19T02:28:50.1491022Z</p1:Updated_From>
        <p1:Updated_Through>2021-01-19T02:28:50.1491022Z</p1:Updated_Through>
      </p1:Transaction_Date_Range_Data>
    </p1:Transaction_Log_Criteria_Data>
    <p1:Exclude_Employees>true</p1:Exclude_Employees>
    <p1:Exclude_Contingent_Workers>true</p1:Exclude_Contingent_Workers>
    <p1:Exclude_Inactive_Workers>true</p1:Exclude_Inactive_Workers>
  </p1:Request_Criteria>
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-01-19T02:28:50.1491022Z</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-01-19T02:28:50.1491022Z</p1:As_Of_Entry_DateTime>
    <p1:Page>1</p1:Page>
    <p1:Count>1</p1:Count>
  </p1:Response_Filter>
  <p1:Response_Group>
    <p1:Include_Reference>1</p1:Include_Reference>
    <p1:Include_Personal_Information>1</p1:Include_Personal_Information>
  </p1:Response_Group>
</Get_Workers_Request>
```

## <a name="how-full-sync-works"></a>Cómo funciona la sincronización completa

La **sincronización completa** en el contexto del aprovisionamiento controlado por Workday hace referencia al proceso de captura de todas las identidades de Workday y determina qué reglas de aprovisionamiento se aplican a cada objeto de trabajador. La sincronización completa se produce cuando se activa el aprovisionamiento por primera vez y también cuando *reinicia el aprovisionamiento* desde Azure Portal o mediante Graph API. 

Para recuperar los datos de trabajadores, Azure AD envía la siguiente solicitud *Get_Workers* de Workday Web Services. La consulta busca el registro de transacciones de Workday de todas las entradas de trabajadores con fecha de entrada en vigor a partir del momento correspondiente a la ejecución de la sincronización completa. 

```XML
<!-- Workday full sync query -->
<!-- Replace version with Workday Web Services version present in your connection URL -->
<!-- Replace timestamps below with the UTC time corresponding to full sync run -->
<!-- Count specifies the number of records to return in each page -->
<!-- Response_Group flags derived from provisioning attribute mapping -->

<Get_Workers_Request p1:version="v21.1" xmlns:p1="urn:com.workday/bsvc" xmlns="urn:com.workday/bsvc">
  <p1:Request_Criteria>
    <p1:Transaction_Log_Criteria_Data>
      <p1:Transaction_Type_References>
        <p1:Transaction_Type_Reference>
          <p1:ID p1:type="Business_Process_Type">Hire Employee</p1:ID>
        </p1:Transaction_Type_Reference>
        <p1:Transaction_Type_Reference>
          <p1:ID p1:type="Business_Process_Type">Contract Contingent Worker</p1:ID>
        </p1:Transaction_Type_Reference>
      </p1:Transaction_Type_References>
    </p1:Transaction_Log_Criteria_Data>
  </p1:Request_Criteria>
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-01-19T02:29:16.0094202Z</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-01-19T02:29:16.0094202Z</p1:As_Of_Entry_DateTime>
    <p1:Count>30</p1:Count>
  </p1:Response_Filter>
  <p1:Response_Group>
    <p1:Include_Reference>1</p1:Include_Reference>
    <p1:Include_Personal_Information>1</p1:Include_Personal_Information>
    <p1:Include_Employment_Information>1</p1:Include_Employment_Information>
    <p1:Include_Organizations>1</p1:Include_Organizations>
    <p1:Exclude_Organization_Support_Role_Data>1</p1:Exclude_Organization_Support_Role_Data>
    <p1:Exclude_Location_Hierarchies>1</p1:Exclude_Location_Hierarchies>
    <p1:Exclude_Cost_Center_Hierarchies>1</p1:Exclude_Cost_Center_Hierarchies>
    <p1:Exclude_Company_Hierarchies>1</p1:Exclude_Company_Hierarchies>
    <p1:Exclude_Matrix_Organizations>1</p1:Exclude_Matrix_Organizations>
    <p1:Exclude_Pay_Groups>1</p1:Exclude_Pay_Groups>
    <p1:Exclude_Regions>1</p1:Exclude_Regions>
    <p1:Exclude_Region_Hierarchies>1</p1:Exclude_Region_Hierarchies>
    <p1:Exclude_Funds>1</p1:Exclude_Funds>
    <p1:Exclude_Fund_Hierarchies>1</p1:Exclude_Fund_Hierarchies>
    <p1:Exclude_Grants>1</p1:Exclude_Grants>
    <p1:Exclude_Grant_Hierarchies>1</p1:Exclude_Grant_Hierarchies>
    <p1:Exclude_Business_Units>1</p1:Exclude_Business_Units>
    <p1:Exclude_Business_Unit_Hierarchies>1</p1:Exclude_Business_Unit_Hierarchies>
    <p1:Exclude_Programs>1</p1:Exclude_Programs>
    <p1:Exclude_Program_Hierarchies>1</p1:Exclude_Program_Hierarchies>
    <p1:Exclude_Gifts>1</p1:Exclude_Gifts>
    <p1:Exclude_Gift_Hierarchies>1</p1:Exclude_Gift_Hierarchies>
    <p1:Include_Management_Chain_Data>1</p1:Include_Management_Chain_Data>
    <p1:Include_Transaction_Log_Data>1</p1:Include_Transaction_Log_Data>
    <p1:Include_Additional_Jobs>1</p1:Include_Additional_Jobs>
  </p1:Response_Group>
</Get_Workers_Request>
```
El nodo *Response_Group* se usa para especificar los atributos de trabajo que se van a capturar de Workday. Para obtener una descripción de cada una de las marcas del nodo *Response_Group*, consulte la [documentación de Get_Workers API](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v35.2/Get_Workers.html#Worker_Response_GroupType) de Workday. 

Algunos valores de marca especificados en el nodo *Response_Group* se calculan en función de los atributos configurados en la aplicación de aprovisionamiento de Azure AD para Workday. Consulte la sección *Entidades admitidas* para ver los criterios que se usan para establecer los valores de las marcas. 

La respuesta de *Get_Workers* de Workday para la consulta anterior incluye el número de registros de trabajo y el de páginas.

```XML
  <wd:Response_Results>
    <wd:Total_Results>509</wd:Total_Results>
    <wd:Total_Pages>17</wd:Total_Pages>
    <wd:Page_Results>30</wd:Page_Results>
    <wd:Page>1</wd:Page>
  </wd:Response_Results>
```
Para recuperar la página siguiente del conjunto de resultados, la siguiente consulta *Get_Workers* especifica el número de páginas como un parámetro de *Response_Filter*.

```XML
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-01-19T02:29:16.0094202Z</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-01-19T02:29:16.0094202Z</p1:As_Of_Entry_DateTime>
    <p1:Page>2</p1:Page>
    <p1:Count>30</p1:Count>
  </p1:Response_Filter>
```
El servicio de aprovisionamiento de Azure AD procesa cada página y recorre en iteración todos los trabajos efectivos durante la sincronización completa. Para cada entrada de trabajador importada desde Workday:
* Se aplica la [expresión XPATH](workday-attribute-reference.md) para recuperar los valores de atributo de Workday.
* Se aplican la asignación de atributos y las reglas de coincidencia. 
* El servicio determina la operación que se realizará en el destino (Azure AD/AD). 

Una vez completado el procesamiento, guarda la marca de tiempo asociada al inicio de la sincronización completa como una marca de agua. Esta marca de agua sirve como punto inicial del ciclo de sincronización incremental. 

## <a name="how-incremental-sync-works"></a>Cómo funciona la sincronización incremental

Después de una sincronización completa, el servicio de aprovisionamiento de Azure AD mantiene `LastExecutionTimestamp` y lo usa para crear consultas diferenciales para recuperar los cambios incrementales. Durante la sincronización incremental, Azure AD envía los siguientes tipos de consultas a Workday: 

* [Consulta de actualizaciones manuales](#query-for-manual-updates)
* [Consulta de actualizaciones y bajas con fecha de entrada en vigor](#query-for-effective-dated-updates-and-terminations)
* [Consulta de contrataciones con fecha futura](#query-for-future-dated-hires)

### <a name="query-for-manual-updates"></a>Consulta de actualizaciones manuales

La siguiente solicitud *Get_Workers* consulta las actualizaciones manuales que se produjeron entre la última ejecución y el momento de ejecución actual. 

```xml
<!-- Workday incremental sync query for manual updates -->
<!-- Replace version with Workday Web Services version present in your connection URL -->
<!-- Replace timestamps below with the UTC time corresponding to last execution and current execution time -->
<!-- Count specifies the number of records to return in each page -->
<!-- Response_Group flags derived from provisioning attribute mapping -->

<Get_Workers_Request p1:version="v21.1" xmlns:p1="urn:com.workday/bsvc" xmlns="urn:com.workday/bsvc">
  <p1:Request_Criteria>
    <p1:Transaction_Log_Criteria_Data>
      <p1:Transaction_Date_Range_Data>
        <p1:Updated_From>2021-01-19T02:29:16.0094202Z</p1:Updated_From>
        <p1:Updated_Through>2021-01-19T02:49:06.290136Z</p1:Updated_Through>
      </p1:Transaction_Date_Range_Data>
    </p1:Transaction_Log_Criteria_Data>
  </p1:Request_Criteria>
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-01-19T02:49:06.290136Z</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-01-19T02:49:06.290136Z</p1:As_Of_Entry_DateTime>
    <p1:Count>30</p1:Count>
  </p1:Response_Filter>
  <p1:Response_Group>
    <p1:Include_Reference>1</p1:Include_Reference>
    <p1:Include_Personal_Information>1</p1:Include_Personal_Information>
    <p1:Include_Employment_Information>1</p1:Include_Employment_Information>
    <p1:Include_Organizations>1</p1:Include_Organizations>
    <p1:Exclude_Organization_Support_Role_Data>1</p1:Exclude_Organization_Support_Role_Data>
    <p1:Exclude_Location_Hierarchies>1</p1:Exclude_Location_Hierarchies>
    <p1:Exclude_Cost_Center_Hierarchies>1</p1:Exclude_Cost_Center_Hierarchies>
    <p1:Exclude_Company_Hierarchies>1</p1:Exclude_Company_Hierarchies>
    <p1:Exclude_Matrix_Organizations>1</p1:Exclude_Matrix_Organizations>
    <p1:Exclude_Pay_Groups>1</p1:Exclude_Pay_Groups>
    <p1:Exclude_Regions>1</p1:Exclude_Regions>
    <p1:Exclude_Region_Hierarchies>1</p1:Exclude_Region_Hierarchies>
    <p1:Exclude_Funds>1</p1:Exclude_Funds>
    <p1:Exclude_Fund_Hierarchies>1</p1:Exclude_Fund_Hierarchies>
    <p1:Exclude_Grants>1</p1:Exclude_Grants>
    <p1:Exclude_Grant_Hierarchies>1</p1:Exclude_Grant_Hierarchies>
    <p1:Exclude_Business_Units>1</p1:Exclude_Business_Units>
    <p1:Exclude_Business_Unit_Hierarchies>1</p1:Exclude_Business_Unit_Hierarchies>
    <p1:Exclude_Programs>1</p1:Exclude_Programs>
    <p1:Exclude_Program_Hierarchies>1</p1:Exclude_Program_Hierarchies>
    <p1:Exclude_Gifts>1</p1:Exclude_Gifts>
    <p1:Exclude_Gift_Hierarchies>1</p1:Exclude_Gift_Hierarchies>
    <p1:Include_Management_Chain_Data>1</p1:Include_Management_Chain_Data>
    <p1:Include_Additional_Jobs>1</p1:Include_Additional_Jobs>
  </p1:Response_Group>
</Get_Workers_Request>
```

### <a name="query-for-effective-dated-updates-and-terminations"></a>Consulta de actualizaciones y bajas con fecha de entrada en vigor

La siguiente solicitud *Get_Workers* consulta las actualizaciones con fecha de vigencia que se produjeron entre la última ejecución y la hora de ejecución actual. 

```xml
<!-- Workday incremental sync query for effective-dated updates -->
<!-- Replace version with Workday Web Services version present in your connection URL -->
<!-- Replace timestamps below with the UTC time corresponding to last execution and current execution time -->
<!-- Count specifies the number of records to return in each page -->
<!-- Response_Group flags derived from provisioning attribute mapping -->

<Get_Workers_Request p1:version="v21.1" xmlns:p1="urn:com.workday/bsvc" xmlns="urn:com.workday/bsvc">
  <p1:Request_Criteria>
    <p1:Transaction_Log_Criteria_Data>
      <p1:Transaction_Date_Range_Data>
        <p1:Effective_From>2021-01-19T02:29:16.0094202Z</p1:Effective_From>
        <p1:Effective_Through>2021-01-19T02:49:06.290136Z</p1:Effective_Through>
      </p1:Transaction_Date_Range_Data>
    </p1:Transaction_Log_Criteria_Data>
  </p1:Request_Criteria>
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-01-19T02:49:06.290136Z</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-01-19T02:49:06.290136Z</p1:As_Of_Entry_DateTime>
    <p1:Page>1</p1:Page>
    <p1:Count>30</p1:Count>
  </p1:Response_Filter>
  <p1:Response_Group>
    <p1:Include_Reference>1</p1:Include_Reference>
    <p1:Include_Personal_Information>1</p1:Include_Personal_Information>
    <p1:Include_Employment_Information>1</p1:Include_Employment_Information>
    <p1:Include_Organizations>1</p1:Include_Organizations>
    <p1:Exclude_Organization_Support_Role_Data>1</p1:Exclude_Organization_Support_Role_Data>
    <p1:Exclude_Location_Hierarchies>1</p1:Exclude_Location_Hierarchies>
    <p1:Exclude_Cost_Center_Hierarchies>1</p1:Exclude_Cost_Center_Hierarchies>
    <p1:Exclude_Company_Hierarchies>1</p1:Exclude_Company_Hierarchies>
    <p1:Exclude_Matrix_Organizations>1</p1:Exclude_Matrix_Organizations>
    <p1:Exclude_Pay_Groups>1</p1:Exclude_Pay_Groups>
    <p1:Exclude_Regions>1</p1:Exclude_Regions>
    <p1:Exclude_Region_Hierarchies>1</p1:Exclude_Region_Hierarchies>
    <p1:Exclude_Funds>1</p1:Exclude_Funds>
    <p1:Exclude_Fund_Hierarchies>1</p1:Exclude_Fund_Hierarchies>
    <p1:Exclude_Grants>1</p1:Exclude_Grants>
    <p1:Exclude_Grant_Hierarchies>1</p1:Exclude_Grant_Hierarchies>
    <p1:Exclude_Business_Units>1</p1:Exclude_Business_Units>
    <p1:Exclude_Business_Unit_Hierarchies>1</p1:Exclude_Business_Unit_Hierarchies>
    <p1:Exclude_Programs>1</p1:Exclude_Programs>
    <p1:Exclude_Program_Hierarchies>1</p1:Exclude_Program_Hierarchies>
    <p1:Exclude_Gifts>1</p1:Exclude_Gifts>
    <p1:Exclude_Gift_Hierarchies>1</p1:Exclude_Gift_Hierarchies>
    <p1:Include_Management_Chain_Data>1</p1:Include_Management_Chain_Data>
    <p1:Include_Additional_Jobs>1</p1:Include_Additional_Jobs>
  </p1:Response_Group>
</Get_Workers_Request>
```

### <a name="query-for-future-dated-hires"></a>Consulta de contrataciones con fecha futura

Si alguna de las consultas anteriores devuelve un contrato con fecha futura, se utilizará la siguiente solicitud *Get_Workers* para obtener información sobre un nuevo contrato con fecha futura. El atributo *WID* del nuevo contrato se usa para realizar la búsqueda y la fecha de entrada en vigor se establece en la fecha y hora de contratación. 

```xml
<!-- Workday incremental sync query to get new hire data effective as on hire date/first day of work -->
<!-- Replace version with Workday Web Services version present in your connection URL -->
<!-- Replace timestamps below hire date/first day of work -->
<!-- Count specifies the number of records to return in each page -->
<!-- Response_Group flags derived from provisioning attribute mapping -->

<Get_Workers_Request p1:version="v21.1" xmlns:p1="urn:com.workday/bsvc" xmlns="urn:com.workday/bsvc">
  <p1:Request_References>
    <p1:Worker_Reference>
      <p1:ID p1:type="WID">7bf6322f1ea101fd0b4433077f09cb04</p1:ID>
    </p1:Worker_Reference>
  </p1:Request_References>
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-02-01T08:00:00+00:00</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-02-01T08:00:00+00:00</p1:As_Of_Entry_DateTime>
    <p1:Count>30</p1:Count>
  </p1:Response_Filter>
  <p1:Response_Group>
    <p1:Include_Reference>1</p1:Include_Reference>
    <p1:Include_Personal_Information>1</p1:Include_Personal_Information>
    <p1:Include_Employment_Information>1</p1:Include_Employment_Information>
    <p1:Include_Organizations>1</p1:Include_Organizations>
    <p1:Exclude_Organization_Support_Role_Data>1</p1:Exclude_Organization_Support_Role_Data>
    <p1:Exclude_Location_Hierarchies>1</p1:Exclude_Location_Hierarchies>
    <p1:Exclude_Cost_Center_Hierarchies>1</p1:Exclude_Cost_Center_Hierarchies>
    <p1:Exclude_Company_Hierarchies>1</p1:Exclude_Company_Hierarchies>
    <p1:Exclude_Matrix_Organizations>1</p1:Exclude_Matrix_Organizations>
    <p1:Exclude_Pay_Groups>1</p1:Exclude_Pay_Groups>
    <p1:Exclude_Regions>1</p1:Exclude_Regions>
    <p1:Exclude_Region_Hierarchies>1</p1:Exclude_Region_Hierarchies>
    <p1:Exclude_Funds>1</p1:Exclude_Funds>
    <p1:Exclude_Fund_Hierarchies>1</p1:Exclude_Fund_Hierarchies>
    <p1:Exclude_Grants>1</p1:Exclude_Grants>
    <p1:Exclude_Grant_Hierarchies>1</p1:Exclude_Grant_Hierarchies>
    <p1:Exclude_Business_Units>1</p1:Exclude_Business_Units>
    <p1:Exclude_Business_Unit_Hierarchies>1</p1:Exclude_Business_Unit_Hierarchies>
    <p1:Exclude_Programs>1</p1:Exclude_Programs>
    <p1:Exclude_Program_Hierarchies>1</p1:Exclude_Program_Hierarchies>
    <p1:Exclude_Gifts>1</p1:Exclude_Gifts>
    <p1:Exclude_Gift_Hierarchies>1</p1:Exclude_Gift_Hierarchies>
    <p1:Include_Management_Chain_Data>1</p1:Include_Management_Chain_Data>
    <p1:Include_Additional_Jobs>1</p1:Include_Additional_Jobs>
  </p1:Response_Group>
</Get_Workers_Request>
```

## <a name="retrieving-worker-data-attributes"></a>Recuperación de atributos de datos de trabajadores

*Get_Workers* API puede devolver conjuntos de datos diferentes asociados a un trabajador. Según las [expresiones de XPATH API](workday-attribute-reference.md) configuradas en el esquema de aprovisionamiento, el servicio de aprovisionamiento de Azure AD determina qué conjuntos de datos se van a recuperar de Workday. En consecuencia, las marcas de *Response_Group* se establecen en la solicitud *Get_Workers*. 

En la tabla siguiente se proporcionan instrucciones sobre la configuración de la asignación que se utiliza para recuperar un conjunto de datos concreto. 

| \# | Entidad de Workday                       | Se incluye de manera predeterminada | Patrón XPATH que se va a especificar en la asignación para capturar entidades no predeterminadas             |
|----|--------------------------------------|---------------------|-------------------------------------------------------------------------------|
| 1  | Datos personales                        | Sí                 | wd:Worker\_Data/wd:Personal\_Data                                             |
| 2  | Datos de empleo                      | Sí                 | wd:Worker\_Data/wd:Employment\_Data                                           |
| 3  | Datos adicionales del trabajo                  | Sí                 | wd:Worker\_Data/wd:Employment\_Data/wd:Worker\_Job\_Data\[@wd:Primary\_Job=0\]|
| 4  | Datos de la organización                    | Sí                 | wd:Worker\_Data/wd:Organization\_Data                                         |
| 5  | Datos de la cadena de administración                | Sí                 | wd:Worker\_Data/wd:Management\_Chain\_Data                                    |
| 6  | Organización de supervisión             | Sí                 | "SUPERVISORY"                                                                 |
| 7  | Compañía                              | Sí                 | "COMPANY"                                                                     |
| 8  | Unidad de negocio                        | No                  | "BUSINESS\_UNIT"                                                              |
| 9  | Jerarquía de la unidad empresarial              | No                  | "BUSINESS\_UNIT\_HIERARCHY"                                                   |
| 10 | Jerarquía de la compañía                    | No                  | "COMPANY\_HIERARCHY"                                                          |
| 11 | Cost Center                          | No                  | "COST\_CENTER"                                                                |
| 12 | Jerarquía del centro de costo                | No                  | "COST\_CENTER\_HIERARCHY"                                                     |
| 13 | Financiación                                 | No                  | "FUND"                                                                        |
| 14 | Jerarquía de financiación                       | No                  | "FUND\_HIERARCHY"                                                             |
| 15 | Obsequio                                 | No                  | "GIFT"                                                                        |
| 16 | Jerarquía de obsequios                       | No                  | "GIFT\_HIERARCHY"                                                             |
| 17 | Conceder                                | No                  | "GRANT"                                                                       |
| 18 | Jerarquía de concesión                      | No                  | "GRANT\_HIERARCHY"                                                            |
| 19 | Jerarquía de sitios empresariales              | No                  | "BUSINESS\_SITE\_HIERARCHY"                                                   |
| 20 | Organización matricial                  | No                  | "MATRIX"                                                                      |
| 21 | Grupo de pago                            | No                  | "PAY\_GROUP"                                                                  |
| 22 | Programas                             | No                  | "PROGRAMS"                                                                    |
| 23 | Jerarquía de programas                    | No                  | "PROGRAM\_HIERARCHY"                                                          |
| 24 | Region                               | No                  | "REGION\_HIERARCHY"                                                           |
| 25 | Jerarquía de ubicaciones                   | No                  | "LOCATION\_HIERARCHY"                                                         |
| 26 | Datos de aprovisionamiento de cuentas            | No                  | wd:Worker\_Data/wd:Account\_Provisioning\_Data                                |
| 27 | Datos de comprobación de antecedentes                | No                  | wd:Worker\_Data/wd:Background\_Check\_Data                                    |
| 28 | Datos de elegibilidad de las ventajas             | No                  | wd:Worker\_Data/wd:Benefit\_Eligibility\_Data                                 |
| 29 | Datos de inscripciones de ventajas              | No                  | wd:Worker\_Data/wd:Benefit\_Enrollment\_Data                                  |
| 30 | Datos profesionales                          | No                  | wd:Worker\_Data/wd:Career\_Data                                               |
| 31 | Datos de remuneración                    | No                  | wd:Worker\_Data/wd:Compensation\_Data                                         |
| 32 | Datos de la entidad tributaria del trabajador temporal | No                  | wd:Worker\_Data/wd:Contingent\_Worker\_Tax\_Authority\_Form\_Type\_Data       |
| 33 | Datos de la unidad de desarrollo                | No                  | wd:Worker\_Data/wd:Development\_Item\_Data                                    |
| 34 | Datos de contratos de los empleados              | No                  | wd:Worker\_Data/wd:Employee\_Contracts\_Data                                  |
| 35 | Datos de revisión de empleados                 | No                  | wd:Worker\_Data/wd:Employee\_Review\_Data                                     |
| 36 | Datos de los comentarios recibidos               | No                  | wd:Worker\_Data/wd:Feedback\_Received\_Data                                   |
| 37 | Datos de objetivos del trabajador                     | No                  | wd:Worker\_Data/wd:Worker\_Goal\_Data                                         |
| 38 | Datos de fotografía                           | No                  | wd:Worker\_Data/wd:Photo\_Data                                                |
| 39 | Datos de cualificación                   | No                  | wd:Worker\_Data/wd:Qualification\_Data                                        |
| 40 | Datos de personas relacionadas                 | No                  | wd:Worker\_Data/wd:Related\_Persons\_Data                                     |
| 41 | Datos del puesto                            | No                  | wd:Worker\_Data/wd:Role\_Data                                                 |
| 42 | Datos de aptitudes                           | No                  | wd:Worker\_Data/wd:Skill\_Data                                                |
| 43 | Datos de perfiles de sucesión              | No                  | wd:Worker\_Data/wd:Succession\_Profile\_Data                                  |
| 44 | Datos de evaluación de capacidades               | No                  | wd:Worker\_Data/wd:Talent\_Assessment\_Data                                   |
| 45 | Datos de cuentas de usuario                    | No                  | wd:Worker\_Data/wd:User\_Account\_Data                                        |
| 46 | Datos de documentos del trabajador                 | No                  | wd:Worker\_Data/wd:Worker\_Document\_Data                                     |

>[!NOTE]
>Cada entidad de Workday que se enumera en la tabla está protegida por una **directiva de seguridad de dominio** en Workday. Si no puede recuperar ningún atributo asociado a la entidad después de establecer la XPATH adecuada, consulte con el administrador de Workday para asegurarse de que la directiva de seguridad de dominio adecuada esté configurada para el usuario del sistema de integración asociado a la aplicación de aprovisionamiento. Por ejemplo, para recuperar *Datos de aptitudes*, se necesita acceso *Get* en el dominio de Workday *Datos del trabajador: aptitudes y experiencia*. 

Estos son algunos ejemplos sobre cómo puede ampliar la integración con Workday para que cumpla requisitos específicos. 

**Ejemplo 1**

Supongamos que desea recuperar los siguientes conjuntos de datos de Workday y usarlos en las reglas de aprovisionamiento:

* Centro de costos
* Jerarquía del centro de costo
* Grupo de pago

Los conjuntos de datos anteriores no se incluyen de forma predeterminada. Para recuperar estos conjuntos de datos:
1. Inicie sesión en Azure Portal y abra la aplicación de aprovisionamiento de usuarios de AD o Azure AD para Workday. 
1. En la hoja Aprovisionamiento, edite las asignaciones y abra la lista de atributos de Workday en la sección Opciones avanzadas. 
1. Agregue las siguientes definiciones de atributos y márquelas como "Obligatorio". Estos atributos no se asignarán a ningún atributo de AD o Azure AD. Solo sirven como señales para el conector para recuperar el centro de costo, la jerarquía del centro de costo y la información del grupo de pago. 

     > [!div class="mx-tdCol2BreakAll"]
     >| Nombre del atributo | Expresión de XPATH API |
     >|---|---|
     >| CostCenterHierarchyFlag  |  wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data[wd:Organization_Data/wd:Organization_Type_Reference/wd:ID[@wd:type='Organization_Type_ID']='COST_CENTER_HIERARCHY']/wd:Organization_Reference/@wd:Descriptor |
     >| CostCenterFlag  |  wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data/wd:Organization_Data[wd:Organization_Type_Reference/wd:ID[@wd:type='Organization_Type_ID']='Supervisory']/wd:Organization_Name/text() |
     >| PayGroupFlag  |  wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data[wd:Organization_Data/wd:Organization_Type_Reference/wd:ID[@wd:type='Organization_Type_ID']='PAY_GROUP']/wd:Organization_Data/wd:Organization_Reference_ID/text() |

1. Una vez que el conjunto de datos del centro de costo y el grupo de pagos está disponible en la respuesta de *Get_Workers*, puede usar los siguientes valores de XPATH para recuperar el nombre del centro de costo, el código de este y el grupo de pago. 

     > [!div class="mx-tdCol2BreakAll"]
     >| Nombre del atributo | Expresión de XPATH API |
     >|---|---|
     >| CostCenterName  | wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data/wd:Organization_Data[wd:Organization_Type_Reference/@wd:Descriptor='Cost Center']/wd:Organization_Name/text() |
     >| CostCenterCode | wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data/wd:Organization_Data[wd:Organization_Type_Reference/@wd:Descriptor='Cost Center']/wd:Organization_Code/text() |
     >| PayGroup | wd:Worker/wd:Worker_Data/wd:Organization_Data/wd:Worker_Organization_Data/wd:Organization_Data[wd:Organization_Type_Reference/@wd:Descriptor='Pay Group']/wd:Organization_Name/text() |

**Ejemplo 2**

Supongamos que desea recuperar las certificaciones asociadas a un usuario. Esta información está disponible como parte del conjunto *Datos de cualificación*. Para obtener este conjunto de datos como parte de la respuesta de *Get_Workers*, utilice el siguiente código de XPATH: 

`wd:Worker/wd:Worker_Data/wd:Qualification_Data/wd:Certification/wd:Certification_Data/wd:Issuer/text()`

**Ejemplo 3**

Supongamos que desea recuperar los *grupos de aprovisionamiento* asignados a un trabajador. Esta información está disponible como parte del conjunto de *datos de aprovisionamiento de cuentas*. Para obtener este conjunto de datos como parte de la respuesta de *Get_Workers*, utilice el siguiente código de XPATH: 

`wd:Worker/wd:Worker_Data/wd:Account_Provisioning_Data/wd:Provisioning_Group_Assignment_Data[wd:Status='Assigned']/wd:Provisioning_Group/text()`

## <a name="handling-different-hr-scenarios"></a>Administración de distintos escenarios de recursos humanos

### <a name="retrieving-international-job-assignments-and-secondary-job-details"></a>Recuperación de asignaciones de trabajos internacionales y detalles de trabajos secundarios

De forma predeterminada, el conector de Workday recupera los atributos asociados con el trabajo principal del trabajador. El conector también admite la recuperación de *datos de trabajo adicionales* asociados con asignaciones de trabajos internacionales o trabajos secundarios. 

Siga los pasos que se indican a continuación para recuperar los atributos asociados con asignaciones de trabajos internacionales: 

1. Establezca la dirección URL de conexión de Workday en la versión 30.0 o superior de la API de servicio web de Workday. Establezca los [valores correctos de XPATH](workday-attribute-reference.md#xpath-values-for-workday-web-services-wws-api-v30) correspondientes en la aplicación de aprovisionamiento de Workday. 
1. Use el selector `@wd:Primary_Job=0` del nodo `Worker_Job_Data` para recuperar el atributo correcto. 
   * **Ejemplo 1:** Para obtener `SecondaryBusinessTitle`, use el valor de XPATH `wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Worker_Job_Data[@wd:Primary_Job=0]/wd:Position_Data/wd:Business_Title/text()`.
   * **Ejemplo 2:** Para obtener `SecondaryBusinessLocation`, use el valor de XPATH `wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Worker_Job_Data[@wd:Primary_Job=0]/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Location_Reference/@wd:Descriptor`.

 

## <a name="next-steps"></a>Pasos siguientes

* [Aprenda a configurar Workday en el aprovisionamiento de Active Directory](../saas-apps/workday-inbound-tutorial.md)
* [Aprenda a configurar la escritura en Workday](../saas-apps/workday-writeback-tutorial.md)
* [Aprenda sobre los atributos de Workday compatibles para el aprovisionamiento de entrada](workday-attribute-reference.md)

