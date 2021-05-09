---
title: Creación de directivas de Configuración de invitado para Windows
description: Aprenda a crear una directiva de Configuración de invitado de Azure Policy para Windows.
ms.date: 03/31/2021
ms.topic: how-to
ms.openlocfilehash: e1c71acd8544073c861a8ad62fb06d78e9d139c5
ms.sourcegitcommit: 62e800ec1306c45e2d8310c40da5873f7945c657
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108165342"
---
# <a name="how-to-create-guest-configuration-policies-for-windows"></a>Creación de directivas de Configuración de invitado para Windows

Antes de crear definiciones de directivas personalizadas, es conveniente leer la información conceptual general en la página [Información sobre Guest Configuration de Azure Policy](../concepts/guest-configuration.md).

Para obtener información sobre cómo crear directivas de Configuración de invitado para Linux, consulte la página [Cómo crear una directiva de configuración de invitados para Linux](./guest-configuration-create-linux.md).

Durante la auditoría en Windows, Configuración de invitado usa un módulo de recursos [Desired State Configuration](/powershell/scripting/dsc/overview/overview) (DSC) para crear el archivo de configuración. La configuración de DSC define la condición en la que debe estar la máquina. Si se produce un error en la evaluación de la configuración, se desencadena el efecto **auditIfNotExists** de la directiva y se considera que la máquina **no es compatible**.

Solo se puede usar la [configuración de invitados de Azure Policy](../concepts/guest-configuration.md) para auditar la configuración dentro de las máquinas. Todavía no está disponible la opción para corregir la configuración dentro de las máquinas.

Use las siguientes acciones para crear su propia configuración para validar el estado de una máquina de Azure o que no sea de Azure.

> [!IMPORTANT]
> Las definiciones de directivas personalizadas con configuración de invitado en los entornos de Azure Government y Azure China es una característica en versión preliminar.
>
> La extensión de configuración de invitado es necesaria para realizar auditorías en las máquinas virtuales de Azure. Para implementar la extensión a gran escala en todas las máquinas Windows, asigne las siguientes definiciones de directiva: `Deploy prerequisites to enable Guest Configuration Policy on Windows VMs`
> 
> No use secretos ni información confidencial en paquetes de contenido personalizado.

## <a name="install-the-powershell-module"></a>Instalación del módulo de PowerShell

El módulo Configuración de invitado automatiza el proceso de creación de contenido personalizado, entre los que se incluyen lo siguiente:

- Creación de un artefacto de contenido de Configuración de invitado (.zip)
- Pruebas automatizadas del artefacto
- Creación de una definición de directiva
- Publicación de la directiva

El módulo se puede instalar en una máquina que ejecute Windows, macOS o Linux con PowerShell 6.2 o una versión posterior que se ejecute localmente, con [Azure Cloud Shell](https://shell.azure.com) o con la [imagen de Azure PowerShell de Docker](https://hub.docker.com/r/azuresdk/azure-powershell-core).

> [!NOTE]
> Aún no se admite la compilación de configuraciones en Linux.

### <a name="base-requirements"></a>Requisitos básicos

Sistemas operativos donde se puede instalar el módulo:

- Linux
- macOS
- Windows

El módulo de recursos de configuración de invitados requiere el siguiente software:

- PowerShell 6.2 o posterior. Si todavía no está instalado, siga [estas instrucciones](/powershell/scripting/install/installing-powershell).
- Azure PowerShell 1.5.0 o posterior. Si todavía no está instalado, siga [estas instrucciones](/powershell/azure/install-az-ps).
  - Solo se requieren los módulos Az "Az.Accounts" y "Az.Resources".

### <a name="install-the-module"></a>Instalación del módulo

Para instalar el módulo **GuestConfiguration** en PowerShell:

1. En una secuencia de comandos de PowerShell, ejecute el siguiente comando:

   ```azurepowershell-interactive
   # Install the Guest Configuration DSC resource module from PowerShell Gallery
   Install-Module -Name GuestConfiguration
   ```

1. Asegúrese de que el módulo se haya importado:

   ```azurepowershell-interactive
   # Get a list of commands for the imported GuestConfiguration module
   Get-Command -Module 'GuestConfiguration'
   ```

## <a name="guest-configuration-artifacts-and-policy-for-windows"></a>Artefactos y directivas de Configuración de invitado para Windows

Configuración de invitado usa Desired State Configuration de PowerShell como abstracción de lenguaje para escribir lo que se va a auditar en Windows. El agente carga una instancia independiente de PowerShell 6.2, por lo que no hay ningún conflicto con el uso de DSC de PowerShell en Windows PowerShell 5.1 y no hay ningún requisito para instalar previamente PowerShell 6.2 o posterior.

Para obtener información general sobre los conceptos y la terminología consulte [Información general de DSC de PowerShell](/powershell/scripting/dsc/overview/overview).

### <a name="how-guest-configuration-modules-differ-from-windows-powershell-dsc-modules"></a>Diferencias entre los módulos de Configuración de invitado y los módulos de DSC de Windows PowerShell

Cuando la configuración de invitado audita una máquina, la secuencia de eventos es diferente de la de DSC de Windows PowerShell.

1. el agente primero ejecuta `Test-TargetResource` para determinar si la configuración se encuentra en el estado correcto.
1. El valor booleano devuelto por la función determina si el estado de Azure Resource Manager para Asignación de invitado debe ser Compatible o No compatible.
1. El proveedor ejecuta `Get-TargetResource` para devolver el estado actual de cada configuración, de modo que haya detalles disponibles tanto sobre el motivo por el que una máquina no es compatible como para confirmar que el estado actual es compatible.

Los parámetros de Azure Policy que pasan valores a las asignaciones de configuración de invitado deben ser de tipo _String_. No es posible pasar matrices mediante parámetros, aunque el recurso de DSC admita matrices.

### <a name="get-targetresource-requirements"></a>Requisitos de Get-TargetResource

La función `Get-TargetResource` tiene requisitos especiales para la configuración de invitados que no se han necesitado para Windows Desired State Configuration.

- La tabla hash que se devuelve debe incluir una propiedad denominada **Reasons**.
- La propiedad Reasons debe ser una matriz.
- Cada elemento de la matriz debe ser una tabla hash con claves denominadas **Code** y **Phrase**.

El servicio utiliza la propiedad Reasons para estandarizar el modo en que se presenta la información cuando una máquina no cumple los requisitos. Puede pensar en cada elemento de Reasons como un "motivo" por el que el recurso no es compatible. La propiedad es una matriz porque un recurso podría no cumplir los requisitos por más de un motivo.

El servicio espera las propiedades **Code** y **Phrase**. Al crear un recurso personalizado, establezca el texto (normalmente stdout) que le gustaría mostrar como el motivo por el que el recurso no es compatible como el valor de **Phrase**. **Code** tiene requisitos de formato específicos, por lo que los informes pueden mostrar claramente información sobre el recurso usado para realizar la auditoría. Esta solución hace que la configuración de invitados sea extensible. Se podría ejecutar cualquier comando, siempre que la salida se pueda devolver como un valor de cadena para la propiedad **Phrase**.

- **Code** (cadena): el nombre del recurso, repetido, y luego un nombre corto sin espacios como identificador del motivo. Estos tres valores deben estar delimitados por signos de dos puntos sin espacios.
  - Un ejemplo sería `registry:registry:keynotpresent`
- **Phrase** (cadena): texto legible para explicar el motivo por el que la configuración no es compatible.
  - Un ejemplo sería `The registry key $key is not present on the machine.`

```powershell
$reasons = @()
$reasons += @{
  Code = 'Name:Name:ReasonIdentifer'
  Phrase = 'Explain why the setting is not compliant'
}
return @{
    reasons = $reasons
}
```

La propiedad Reasons debe agregarse al archivo MOF del esquema del recurso como una clase insertada.

```mof
[ClassVersion("1.0.0.0")] 
class Reason
{
    [Read] String Phrase;
    [Read] String Code;
};

[ClassVersion("1.0.0.0"), FriendlyName("ResourceName")]
class ResourceName : OMI_BaseResource
{
    [Key, Description("Example description")] String Example;
    [Read, EmbeddedInstance("Reason")] String Reasons[];
};
```

Si el recurso tiene las propiedades necesarias, esas propiedades se deben devolver en `Get-TargetResource` en paralelo con la clase `reasons`. Si no se incluye `reasons`, el servicio incluye un comportamiento "comodín" que compara los valores de entrada con `Get-TargetResource` y los valores devueltos por `Get-TargetResource` y proporciona una comparación detallada como `reasons`.

### <a name="configuration-requirements"></a>Requisitos de configuración

El nombre de la configuración personalizada debe ser coherente en todas partes. El nombre del archivo .zip para el paquete de contenido, el nombre de la configuración en el archivo MOF y el nombre de la asignación de invitado en la plantilla de Azure Resource Manager (plantilla de ARM) deben ser el mismo.

### <a name="policy-requirements"></a>Requisitos de la directiva

La sección `metadata` de la definición de directiva debe incluir dos propiedades para que el servicio de configuración de invitado automatice el aprovisionamiento y la generación de informes de las asignaciones de la configuración de invitado. La propiedad `category` debe establecerse en "Configuración de invitado" y una sección denominada `Guest Configuration` debe contener información sobre la asignación de la configuración de invitado. El cmdlet `New-GuestConfigurationPolicy` crea este texto automáticamente.
Consulte las instrucciones paso a paso en esta página.

En el siguiente ejemplo se muestra la sección `metadata`.

```json
    "metadata": {
      "category": "Guest Configuration",
      "guestConfiguration": {
        "name": "test",
        "version": "1.0.0",
        "contentType": "Custom",
        "contentUri": "CUSTOM-URI-HERE",
        "contentHash": "CUSTOM-HASH-VALUE-HERE",
        "configurationParameter": {}
      }
    },
```

### <a name="scaffolding-a-guest-configuration-project"></a>Aplicación de la técnica scaffolding a un proyecto de configuración de invitados

Los desarrolladores que deseen acelerar el proceso de introducción y trabajar desde código de ejemplo pueden instalar un proyecto de la comunidad denominado **Guest Configuration Project**. El proyecto instala una plantilla para el módulo [Plaster](https://github.com/powershell/plaster) de PowerShell. Esta herramienta se puede usar para aplicar scaffolding a un proyecto, incluida una configuración de trabajo y un recurso de ejemplo, y un conjunto de pruebas [Pester](https://github.com/pester/pester) para validar el proyecto. La plantilla también incluye ejecutores de tareas para Visual Studio Code para automatizar la compilación y validación del paquete de configuración de invitados. Para más información, consulte el proyecto de GitHub [Proyecto de configuración de invitados](https://github.com/microsoft/guestconfigurationproject).

Para obtener más información sobre cómo trabajar con configuraciones en general, consulte [Escritura, compilación y aplicación de una configuración](/powershell/scripting/dsc/configurations/write-compile-apply-configuration).

### <a name="expected-contents-of-a-guest-configuration-artifact"></a>Contenido esperado de un artefacto de Configuración de invitado

La configuración de invitados usa el paquete completado para crear las definiciones de Azure Policy. El paquete consta de:

- La configuración de DSC compilada como MOF
- Carpeta Modules
  - Módulo GuestConfiguration
  - Módulo DscNativeResources
  - (Windows) Módulos de recursos de DSC requeridos por MOF

Los cmdlets de PowerShell ayudan en la creación del paquete. No se requiere ninguna carpeta de nivel raíz ni carpeta de versiones. El formato del paquete debe ser un archivo .zip y no puede superar el tamaño total de 100 MB cuando se descomprima.

### <a name="storing-guest-configuration-artifacts"></a>Almacenamiento de artefactos de Configuración de invitado

El paquete .zip debe almacenarse en una ubicación a la que puedan obtener acceso las máquinas virtuales administradas.
Algunos ejemplos son los repositorios de GitHub, un repositorio de Azure o Azure Storage. Si prefiere no hacer que el paquete sea público, puede incluir un [token de SAS](../../../storage/common/storage-sas-overview.md) en la dirección URL. También puede implementar el [punto de conexión de servicio](../../../storage/common/storage-network-security.md#grant-access-from-a-virtual-network) para las máquinas de una red privada, aunque esta configuración solo se aplica para acceder al paquete y no para comunicarse con el servicio.

## <a name="step-by-step-creating-a-custom-guest-configuration-audit-policy-for-windows"></a>Creación de una directiva de auditoría de Configuración de invitado personalizada para Windows, paso a paso

Cree una configuración de DSC para la configuración de auditoría. En el ejemplo siguiente de script de PowerShell se crea una configuración denominada **AuditBitLocker**, se importa el módulo de recursos **PsDscResources** y se usa el recurso `Service` para auditar un servicio en ejecución. El script de configuración se puede ejecutar desde una máquina Windows o macOS.

```powershell
# Add PSDscResources module to environment
Install-Module 'PSDscResources'

# Define the DSC configuration and import GuestConfiguration
Configuration AuditBitLocker
{
    Import-DscResource -ModuleName 'PSDscResources'

    Node AuditBitlocker {
      Service 'Ensure BitLocker service is present and running'
      {
          Name = 'BDESVC'
          Ensure = 'Present'
          State = 'Running'
      }
    }
}

# Compile the configuration to create the MOF files
AuditBitLocker
```

Ejecute este script en un terminal de PowerShell o guarde el archivo con `config.ps1` el nombre en la carpeta del proyecto. Ejecútelo en PowerShell mediante la ejecución de `./config.ps1` en el terminal. Se crea un archivo mof nuevo.

El comando `Node AuditBitlocker` no es técnicamente necesario, pero genera un archivo denominado `AuditBitlocker.mof`, en lugar del valor predeterminado, `localhost.mof`. El hecho de que el nombre de archivo. mof siga la configuración facilita la organización de muchos archivos cuando se trabaja a escala.

Una vez compilado el MOF, los archivos auxiliares deben empaquetarse juntos. La configuración de invitados usa el paquete completado para crear las definiciones de Azure Policy.

El cmdlet `New-GuestConfigurationPackage` crea el paquete. Los módulos necesarios para la configuración deben estar disponibles en `$Env:PSModulePath`. Parámetros del cmdlet `New-GuestConfigurationPackage` al crear el contenido de Windows:

- **Name**: nombre del paquete de configuración de invitados.
- **Configuración**: ruta de acceso completa del documento de configuración de DSC compilado.
- **Ruta de acceso**: ruta de acceso de la carpeta de salida. Este parámetro es opcional. Si no se especifica, el paquete se crea en el directorio actual.

Ejecute el siguiente comando para crear un paquete con la configuración proporcionada en el paso anterior:

```azurepowershell-interactive
New-GuestConfigurationPackage `
  -Name 'AuditBitlocker' `
  -Configuration './AuditBitlocker/AuditBitlocker.mof'
```

Después de crear el paquete de configuración, pero antes de publicarlo en Azure, puede probar el paquete desde la estación de trabajo o el entorno de integración continua e implementación continua (CI/CD). El cmdlet `Test-GuestConfigurationPackage` de GuestConfiguration incluye el mismo agente en el entorno de desarrollo que se usa en las máquinas de Azure. Con esta solución, puede realizar pruebas de integración localmente antes de la publicación en entornos de nube con facturación.

Dado que el agente en realidad está evaluando el entorno local, en la mayoría de los casos es necesario ejecutar el cmdlet Test- en la misma plataforma del sistema operativo que se va a auditar. La prueba solo usa módulos incluidos en el paquete de contenido.

Parámetros del cmdlet `Test-GuestConfigurationPackage`:

- **Name**: Nombre de la directiva de Configuración de invitado.
- **Parámetro**: Parámetros de directiva proporcionados en formato de tabla hash.
- **Ruta de acceso**: ruta de acceso completa del paquete de configuración de invitados.

Ejecute el siguiente comando para probar el paquete creado en el paso anterior:

```azurepowershell-interactive
Test-GuestConfigurationPackage `
  -Path ./AuditBitlocker.zip
```

El cmdlet también admite la entrada de la canalización de PowerShell. Canaliza la salida del cmdlet `New-GuestConfigurationPackage` al cmdlet `Test-GuestConfigurationPackage`.

```azurepowershell-interactive
New-GuestConfigurationPackage -Name AuditBitlocker -Configuration ./AuditBitlocker/AuditBitlocker.mof | Test-GuestConfigurationPackage
```

El siguiente paso consiste en publicar el archivo en Azure Blob Storage. No hay ningún requisito especial para la cuenta de almacenamiento, pero es una buena idea hospedar el archivo en una región cercana a las máquinas. Si no tiene una cuenta de almacenamiento, use el ejemplo siguiente. Los comandos siguientes, incluido `Publish-GuestConfigurationPackage`, requieren el módulo `Az.Storage`.

```azurepowershell-interactive
# Creates a new resource group, storage account, and container
New-AzResourceGroup -name myResourceGroupName -Location WestUS
New-AzStorageAccount -ResourceGroupName myResourceGroupName -Name myStorageAccountName -SkuName 'Standard_LRS' -Location 'WestUs' | New-AzStorageContainer -Name guestconfiguration -Permission Blob
```

Parámetros del cmdlet `Publish-GuestConfigurationPackage`:

- **Ruta de acceso**: Ubicación del paquete que se va a publicar.
- **ResourceGroupName**: Nombre del grupo de recursos donde se encuentra la cuenta de almacenamiento.
- **StorageAccountName**: Nombre de la cuenta de almacenamiento donde se debe publicar el paquete.
- **StorageContainerName** (valor predeterminado: _guestconfiguration_): Nombre del contenedor de almacenamiento en la cuenta de almacenamiento.
- **Force**: Permite sobrescribir el paquete existente en la cuenta de almacenamiento con el mismo nombre.

En el ejemplo siguiente se publica el paquete en un contenedor de almacenamiento denominado "guestconfiguration".

```azurepowershell-interactive
Publish-GuestConfigurationPackage -Path ./AuditBitlocker.zip -ResourceGroupName myResourceGroupName -StorageAccountName myStorageAccountName
```

Una vez que se ha creado y cargado un paquete de directivas personalizadas de Configuración de invitado, cree la definición de la directiva de Configuración de invitado. El cmdlet `New-GuestConfigurationPolicy` toma un paquete de directivas personalizado y crea una definición de directivas.

Parámetros del cmdlet `New-GuestConfigurationPolicy`:

- **ContentUri**: Uri de http(s) público del paquete de contenido de configuración de invitados.
- **DisplayName**: Nombre para mostrar de la directiva.
- **Descripción**: Descripción de la directiva.
- **Parámetro**: Parámetros de directiva proporcionados en formato de tabla hash.
- **Versión**: Versión de la directiva.
- **Ruta de acceso**: Ruta de acceso de destino donde se crean las definiciones de directiva.
- **Plataforma**: Plataforma de destino (Windows/Linux) para la directiva de configuración de invitados y el paquete de contenido.
- **Tag** agrega uno o varios filtros de etiquetas a la definición de directiva.
- **Category** establece el campo de metadatos de categoría en la definición de directiva.

En el ejemplo siguiente se crean las definiciones de directivas en una ruta de acceso especificada desde un paquete de directivas personalizado:

```azurepowershell-interactive
New-GuestConfigurationPolicy `
    -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditBitLocker.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
    -DisplayName 'Audit BitLocker Service.' `
    -Description 'Audit if BitLocker is not enabled on Windows machine.' `
    -Path './policies' `
    -Platform 'Windows' `
    -Version 1.0.0 `
    -Verbose
```

`New-GuestConfigurationPolicy` se encarga de crear los siguientes archivos:

- **auditIfNotExists.json**

La salida del cmdlet devuelve un objeto que contiene el nombre para mostrar de la iniciativa y la ruta de acceso de los archivos de directiva.

Por último, publique las definiciones de directivas con el cmdlet `Publish-GuestConfigurationPolicy`. El cmdlet solo tiene el parámetro **Path** que apunta a la ubicación de los archivos JSON que creó `New-GuestConfigurationPolicy`.

Para ejecutar el comando Publish, necesita acceso para crear directivas en Azure. Los requisitos de autorización específicos se documentan en la página [Información general de Azure Policy](../overview.md). El mejor rol integrado es **Colaborador de la directiva de recursos**.

```azurepowershell-interactive
Publish-GuestConfigurationPolicy -Path '.\policies'
```

El cmdlet `Publish-GuestConfigurationPolicy` acepta la ruta de acceso de la canalización de PowerShell. Esta característica significa que puede crear los archivos de directiva y publicarlos en un único conjunto de comandos canalizados.

```azurepowershell-interactive
New-GuestConfigurationPolicy `
  -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditBitLocker.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
  -DisplayName 'Audit BitLocker service.' `
  -Description 'Audit if the BitLocker service is not enabled on Windows machine.' `
  -Path './policies' `
| Publish-GuestConfigurationPolicy
```

Con la directiva creada en Azure, el último paso es asignar la definición. Consulte cómo asignar la definición con el [portal](../assign-policy-portal.md), la [CLI de Azure](../assign-policy-azurecli.md) y [Azure PowerShell](../assign-policy-powershell.md).

### <a name="filtering-guest-configuration-policies-using-tags"></a>Filtrado de directivas de Configuración de invitado mediante etiquetas

De forma opcional, las definiciones de directiva que crean los cmdlets en el módulo Configuración de invitado pueden incluir un filtro para las etiquetas. El parámetro **Tag** de `New-GuestConfigurationPolicy` admite una matriz de tablas hash que contiene entradas de etiquetas individuales. Las etiquetas se agregan a la sección `If` de la definición de directiva y no se pueden modificar mediante una asignación de directiva.

A continuación se muestra un fragmento de código de ejemplo de una definición de directiva que filtra por etiquetas.

```json
"if": {
  "allOf" : [
    {
      "allOf": [
        {
          "field": "tags.Owner",
          "equals": "BusinessUnit"
        },
        {
          "field": "tags.Role",
          "equals": "Web"
        }
      ]
    },
    {
      // Original Guest Configuration content
    }
  ]
}
```

### <a name="using-parameters-in-custom-guest-configuration-policy-definitions"></a>Uso de parámetros en definiciones de directivas de Configuración de invitados personalizadas

La configuración de invitados admite la invalidación de propiedades de una configuración en tiempo de ejecución. Esta característica significa que los valores del archivo MOF del paquete no tienen que considerarse estáticos. Los valores de invalidación se proporcionan mediante Azure Policy y no cambian el modo en que se crean o compilan las configuraciones.

Los cmdlets `New-GuestConfigurationPolicy` e `Test-GuestConfigurationPolicyPackage` incluyen un parámetro denominado **Parámetros**. Este parámetro toma una definición de tabla hash que incluye todos los detalles sobre cada parámetro y crea las secciones necesarias de cada archivo que se usa para crear la definición de Azure Policy.

En el ejemplo siguiente se crea una definición de directiva para auditar un servicio que el usuario selecciona de una lista en el momento de la asignación de la directiva.

```azurepowershell-interactive
# This DSC Resource text:
Service 'UserSelectedNameExample'
      {
          Name = 'ParameterValue'
          Ensure = 'Present'
          State = 'Running'
      }

# Would require the following hashtable:
$PolicyParameterInfo = @(
    @{
        Name = 'ServiceName'                                            # Policy parameter name (mandatory)
        DisplayName = 'windows service name.'                           # Policy parameter display name (mandatory)
        Description = "Name of the windows service to be audited."      # Policy parameter description (optional)
        ResourceType = "Service"                                        # DSC configuration resource type (mandatory)
        ResourceId = 'UserSelectedNameExample'                          # DSC configuration resource id (mandatory)
        ResourcePropertyName = "Name"                                   # DSC configuration resource property name (mandatory)
        DefaultValue = 'winrm'                                          # Policy parameter default value (optional)
        AllowedValues = @('BDESVC','TermService','wuauserv','winrm')    # Policy parameter allowed values (optional)
    }
)

New-GuestConfigurationPolicy
    -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditBitLocker.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
    -DisplayName 'Audit Windows Service.' `
    -Description 'Audit if a Windows Service is not enabled on Windows machine.' `
    -Path '.\policies' `
    -Parameter $PolicyParameterInfo `
    -Version 1.0.0
```

## <a name="extending-guest-configuration-with-third-party-tools"></a>Ampliación de Configuración de invitado con herramientas de terceros

Los paquetes de artefactos para Configuración de invitado se pueden ampliar para que incluyan herramientas de terceros.
La ampliación de Configuración de invitados requiere el desarrollo de dos componentes.

- Un recurso de Desired State Configuration que controla toda la actividad relacionada con la administración de la herramienta de terceros.
  - Instalar
  - Invocar
  - Convertir salida
- Contenido en el formato correcto para que la herramienta consuma de forma nativa.

El recurso de DSC requiere un desarrollo personalizado si aún no existe una solución de la comunidad.
Las soluciones de la comunidad se pueden descubrir buscando en Galería de PowerShell por la etiqueta [GuestConfiguration](https://www.powershellgallery.com/packages?q=Tags%3A%22GuestConfiguration%22).

> [!Note]
> La extensibilidad de Configuración de invitado es un escenario de "traiga su propia licencia". Asegúrese de que ha cumplido los términos y condiciones de las herramientas de terceros antes de usarlas.

Una vez instalado el recurso de DSC en el entorno de desarrollo, use el parámetro **FilesToInclude** para que `New-GuestConfigurationPackage` incluya el contenido de la plataforma de terceros en el artefacto de contenido.

## <a name="policy-lifecycle"></a>Ciclo de vida de la directiva

Si desea publicar una actualización de la directiva, realice el cambio en el paquete de configuración de invitado y en los detalles de definición de Azure Policy.

> [!NOTE]
> La propiedad `version` de la asignación de configuración de invitado solo afecta a los paquetes hospedados por Microsoft. El procedimiento recomendado para el control de versiones del contenido personalizado es incluir la versión en el nombre de archivo.

En primer lugar, al ejecutar `New-GuestConfigurationPackage`, especifique un nombre para el paquete que lo haga único con respecto a las versiones anteriores. Puede incluir un número de versión en el nombre, como `PackageName_1.0.0`. El número de este ejemplo solo se usa para hacer que el paquete sea único, no para especificar que el paquete sea más reciente o más antiguo que otros paquetes.

En segundo lugar, actualice los parámetros que se usan con el cmdlet `New-GuestConfigurationPolicy` después de cada una de las explicaciones siguientes.

- **Versión**: al ejecutar el cmdlet `New-GuestConfigurationPolicy`, debe especificar un número de versión mayor que el que se ha publicado actualmente.
- **contentUri**: al ejecutar el cmdlet `New-GuestConfigurationPolicy`, debe especificar un URI para la ubicación del paquete. La inclusión de una versión del paquete en el nombre de archivo garantiza que el valor de esta propiedad cambie en cada versión.
- **contentHash**: el cmdlet `New-GuestConfigurationPolicy` actualiza automáticamente esta propiedad. Es un valor hash del paquete que creó `New-GuestConfigurationPackage`. La propiedad debe ser correcta para el archivo `.zip` que se publica. Si solo se actualiza la propiedad **contentUri**, la extensión no aceptará el paquete de contenido.

La manera más fácil de publicar un paquete actualizado es repetir el proceso que se describe en este artículo y proporcionar un número de versión actualizado. Este proceso garantiza que todas las propiedades se hayan actualizado correctamente.

## <a name="optional-signing-guest-configuration-packages"></a>Opcional: Firma de paquetes de configuración de invitados

Las directivas personalizadas de Configuración de invitado usan el hash SHA256 para validar que el paquete de directivas no haya cambiado.
Opcionalmente, los clientes también pueden usar un certificado para firmar paquetes y forzar a la extensión de configuración de invitados a permitir solo el contenido firmado.

Para habilitar este escenario, hay dos pasos que debe completar. Ejecute el cmdlet para firmar el paquete de contenido y anexe una etiqueta a las máquinas que deben solicitar la firma del código.

Para usar la característica de validación de firmas, ejecute el cmdlet `Protect-GuestConfigurationPackage` para firmar el paquete antes de publicarlo. Este cmdlet requiere un certificado de "firma de código".

```azurepowershell-interactive
$Cert = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object {($_.Subject-eq "CN=mycert") }
Protect-GuestConfigurationPackage -Path .\package\AuditWindowsService\AuditWindowsService.zip -Certificate $Cert -Verbose
```

Parámetros del cmdlet `Protect-GuestConfigurationPackage`:

- **Ruta de acceso**: ruta de acceso completa del paquete de configuración de invitados.
- **Certificate**: certificado de firma de código para firmar el paquete. Este parámetro solo se admite cuando se firma contenido para Windows.

El agente GuestConfiguration espera que la clave pública del certificado esté presente en la opción "Entidades de certificación raíz de confianza" de las máquinas Windows y en la ruta de acceso `/usr/local/share/ca-certificates/extra` de las máquinas Linux. Para que el nodo compruebe el contenido firmado, instale la clave pública del certificado en la máquina antes de aplicar la directiva personalizada. Este proceso se puede realizar con cualquier técnica dentro de la máquina virtual o mediante Azure Policy. [Aquí se proporciona](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-push-certificate-windows) una plantilla de ejemplo.
La directiva de acceso de Key Vault debe permitir que el proveedor de recursos del proceso obtenga acceso a los certificados durante las implementaciones. Para obtener los pasos detallados, consulte [Configuración de Key Vault para máquinas virtuales en Azure Resource Manager](../../../virtual-machines/windows/key-vault-setup.md#use-templates-to-set-up-key-vault).

El siguiente es un ejemplo para exportar la clave pública de un certificado de firma, para importarla a la máquina.

```azurepowershell-interactive
$Cert = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object {($_.Subject-eq "CN=mycert3") } | Select-Object -First 1
$Cert | Export-Certificate -FilePath "$env:temp\DscPublicKey.cer" -Force
```

Una vez publicado el contenido, anexe una etiqueta con el nombre `GuestConfigPolicyCertificateValidation` y el valor `enabled` a todas las máquinas virtuales en las que se debe solicitar la firma de código. Consulte los [ejemplos de etiqueta](../samples/built-in-policies.md#tags) sobre cómo se pueden entregar etiquetas a escala mediante Azure Policy. Una vez que esta etiqueta esté en su lugar, la definición de la directiva que se genera mediante el cmdlet `New-GuestConfigurationPolicy` habilita el requisito a través de la extensión de configuración de invitados.

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre la auditoría de VM con la [configuración de invitados](../concepts/guest-configuration.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](./programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](./get-compliance-data.md).
