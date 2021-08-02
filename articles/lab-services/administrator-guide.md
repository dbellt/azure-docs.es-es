---
title: 'Azure Lab Services: guía del administrador | Microsoft Docs'
description: Esta guía sirve de ayuda para los administradores que crean y administran cuentas de laboratorio mediante Azure Lab Services.
ms.topic: article
ms.date: 10/20/2020
ms.openlocfilehash: 0dd7cb9f23d820cc8a4001d430e8ef446ecc5460
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958836"
---
# <a name="azure-lab-services---administrator-guide"></a>Azure Lab Services: guía del administrador
Los administradores de tecnología de la información (TI) que administran los recursos en la nube de una universidad suelen ser también responsables de configurar la cuenta de laboratorio en su institución. Una vez configurada una cuenta de laboratorio, los administradores o educadores crean los laboratorios que están incluidos en la cuenta. En este artículo se proporciona información general de alto nivel sobre los recursos de Azure relacionados e instrucciones para crearlos.

![Diagrama de una vista de alto nivel de los recursos de Azure de una cuenta de laboratorio](./media/administrator-guide/high-level-view.png)

- Los laboratorios se hospedan en una suscripción de Azure propiedad de Azure Lab Services.
- Las cuentas de laboratorio, la galería de imágenes compartidas y las versiones de imágenes se hospedan en la suscripción.
- La cuenta de laboratorio y la galería de imágenes compartidas pueden residir en el mismo grupo de recursos. En este diagrama, están en distintos grupos de recursos.

Para más información sobre la arquitectura, consulte [Aspectos básicos de la arquitectura en Azure Lab Services](./classroom-labs-fundamentals.md).

## <a name="subscription"></a>Suscripción
Su universidad podría tener una o varias suscripciones de Azure. Las suscripciones se usan para administrar la facturación y la seguridad de todos los recursos y servicios de Azure que se usan dentro de esta plataforma, incluidas las cuentas de laboratorio.

La relación entre una cuenta de laboratorio y su suscripción es importante por los siguientes motivos:

- La facturación se notifica mediante la suscripción que contiene la cuenta de laboratorio.
- Puede conceder a los usuarios del inquilino de Azure Active Directory (Azure AD) de la suscripción acceso a Azure Lab Services. Puede agregar un usuario como propietario o colaborador de la cuenta de laboratorio, creador del laboratorio o propietario del laboratorio.

Los laboratorios y sus máquinas virtuales (VM) se administran y hospedan en una suscripción propiedad de Azure Lab Services.

## <a name="resource-group"></a>Resource group
Una suscripción contiene uno o varios grupos de recursos. Los grupos de recursos se usan para crear agrupaciones lógicas de recursos de Azure que se emplean juntos dentro de la misma solución.  

Al crear una cuenta de laboratorio, debe configurar el grupo de recursos que la contiene. 

También se requiere un grupo de recursos para crear una [galería de imágenes compartidas](#shared-image-gallery). Puede colocar su cuenta de laboratorio y la galería de imágenes compartidas en el mismo grupo de recursos o en dos grupos de recursos distintos. Es posible que quiera adoptar este segundo enfoque si tiene previsto compartir la galería de imágenes entre varias soluciones.

Al crear una cuenta de laboratorio, puede crear y adjuntar automáticamente una galería de imágenes compartidas al mismo tiempo.  Esta opción da como resultado la creación de la cuenta de laboratorio y la galería de imágenes compartidas en grupos de recursos independientes. Observará este comportamiento cuando siga los pasos que se describen en el tutorial [Configuración de la galería de imágenes compartidas en el momento de creación de la cuenta de laboratorio](how-to-attach-detach-shared-image-gallery.md#configure-at-the-time-of-lab-account-creation). En la imagen que se encuentra al principio de este artículo se usa esta configuración. 

Se recomienda planear con antelación la estructura de los grupos de recursos, ya que, una vez creado, *no* es posible cambiar el grupo de recursos de una cuenta de laboratorio o de la galería de imágenes compartidas. Si necesita cambiar el grupo de estos recursos, deberá eliminar y volver a crear la cuenta de laboratorio o la galería de imágenes compartidas.

## <a name="lab-account"></a>Cuenta de laboratorio

Una cuenta de laboratorio funciona como contenedor de uno o varios laboratorios. Al empezar a trabajar con Azure Lab Services, es habitual tener solo una cuenta de laboratorio. A medida que el uso del laboratorio se escala verticalmente, puede optar por crear más cuentas de laboratorio más adelante.

En la lista siguiente se resaltan escenarios en los que puede ser beneficioso tener más de una cuenta de laboratorio:

- **Administración de distintos requisitos de directivas entre laboratorios**

    Cuando se configura una cuenta de laboratorio, se establecen directivas que se aplican a *todos* los laboratorios de la cuenta de laboratorio, por ejemplo:
    - La red virtual de Azure con recursos compartidos a los que puede acceder el laboratorio. Por ejemplo, puede tener un conjunto de laboratorios que necesiten acceso a un conjunto de datos compartido dentro de una red virtual.
    - Las imágenes de máquina virtual que los laboratorios pueden usarse para crear las máquinas virtuales. Por ejemplo, puede tener un conjunto de laboratorios que necesiten acceso a la imagen [Data Science VM for Linux](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804) de Azure Marketplace.

    Si cada uno de los laboratorios tiene unos requisitos de directiva únicos, puede ser beneficioso crear cuentas de laboratorio distintas para administrar cada laboratorio por separado.

- **Asignación de un presupuesto distinto a cada cuenta de laboratorio**
  
    En lugar de presentar todos los costos de laboratorio en una sola cuenta de laboratorio, es posible que necesite tener un presupuesto distribuido con más claridad. Por ejemplo, puede crear una cuenta de laboratorio para el departamento de matemáticas de la universidad, otra para el departamento de informática, etc., a fin de separar el presupuesto entre los departamentos.  Con [Azure Cost Management](../cost-management-billing/cost-management-billing-overview.md), puede ver luego el costo de cada cuenta de laboratorio.

- **Aislamiento de los laboratorios piloto de los laboratorios activos o de producción**
  
    Puede que haya casos en los que quiera realizar cambios en la directiva piloto de una cuenta de laboratorio sin que ello afecte a los laboratorios activos o de producción. En este tipo de escenario, la creación de una cuenta de laboratorio independiente con fines piloto permite aislar los cambios. 

## <a name="lab"></a>Laboratorio

Un laboratorio contiene máquinas virtuales (VM) que se asignan a un único alumno.  En general, puede esperar lo siguiente:

- Tener un laboratorio para cada clase.
- Cree un conjunto de laboratorios para cada semestre, trimestre u otro sistema académico que use. En el caso de clases que necesiten usar la misma imagen, debe emplear una [galería de imágenes compartidas](#shared-image-gallery). De este modo, podrá reutilizar las imágenes entre laboratorios y períodos académicos.

A la hora de determinar cómo estructurar los laboratorios, tenga en cuenta los siguientes puntos:

- **Todas las VM de un laboratorio se implementan con la misma imagen con la que se han publicado**

    Como resultado, si tiene una clase que requiere que se publiquen distintas imágenes de laboratorio al mismo tiempo, deben crearse laboratorios distintos para cada una.
  
- **La cuota de uso se establece en el nivel de laboratorio y se aplica a todos los usuarios del laboratorio**

    Para establecer cuotas diferentes para los usuarios, debe crear distintos laboratorios. Sin embargo, es posible agregar más horas a un usuario específico después de haber establecido la cuota.
  
- **La programación de inicio o apagado se establece en el nivel de laboratorio y se aplica a todas las máquinas virtuales del laboratorio**

    Al igual que con la configuración de la cuota, si necesita establecer diferentes programaciones para los usuarios, debe crear un laboratorio aparte para cada programación.

De forma predeterminada, cada laboratorio tiene su propia red virtual.  Si tiene habilitado el emparejamiento de red virtual, cada laboratorio tendrá su propia subred emparejada a la red virtual especificada.

## <a name="shared-image-gallery"></a>Galería de imágenes compartidas

Una galería de imágenes compartidas se asocia a una cuenta de laboratorio y sirve como repositorio central para almacenar imágenes. Una imagen se guarda en la galería cuando un educador elige exportarla desde la máquina virtual de plantilla de un laboratorio. Cada vez que un formador realiza cambios en la máquina virtual de plantilla y la exporta, se crean definiciones de imagen o versiones en la galería.  

Los instructores pueden publicar una versión de una imagen desde la galería de imágenes compartidas cuando crean un laboratorio. Aunque la galería almacena varias versiones de una imagen, los formadores solo pueden seleccionar la última versión durante la creación del laboratorio.  La última versión se elige en función del valor más alto de MajorVersion, luego MinorVersion y, por último, Patch.  Para obtener más información sobre las versiones, vea [Versiones de la imagen](../virtual-machines/shared-image-galleries.md#image-versions).

El servicio Shared Image Gallery es un recurso opcional que puede que no necesite inmediatamente si está empezando solo por unos cuantos laboratorios. Sin embargo, este servicio ofrece muchas ventajas que le resultarán útiles a medida que escale verticalmente a laboratorios adicionales:

- **Puede guardar y administrar versiones de una imagen de máquina virtual de plantilla**

    Resulta útil cuando se crea una imagen personalizada o se realizan cambios (software, configuración, etc.) en una imagen de la galería de Azure Marketplace.  Por ejemplo, es habitual que los educadores requieran la instalación de software o herramientas diferentes. En lugar de solicitar a los alumnos que instalen manualmente estos requisitos previos por su cuenta, se pueden exportar diferentes versiones de la imagen de máquina virtual de plantilla en una galería de imágenes compartidas. Después, puede usar estas versiones de imagen al crear otros laboratorios.

- **Puede compartir y volver a usar imágenes de máquina virtual de plantilla entre laboratorios**

    Puede guardar y reutilizar una imagen para que no tenga que configurarla desde cero cada vez que cree un laboratorio. Por ejemplo, si varias clases necesitan usar la misma imagen, puede crearla una vez y exportarla a la galería de imágenes compartidas para que se pueda compartir entre laboratorios.

- **Puede cargar sus propias imágenes personalizadas desde otros entornos fuera de los laboratorios**.

    Puede [cargar imágenes personalizadas en otros entornos fuera del contexto de los laboratorios](how-to-attach-detach-shared-image-gallery.md).  Por ejemplo, puede cargar imágenes desde su propio entorno de laboratorio físico o desde una máquina virtual de Azure a Shared Image Gallery.  Una vez que se importa una imagen en la galería, puede usar las imágenes para crear laboratorios.

Para agrupar imágenes compartidas de manera lógica, puede realizar una de las siguientes acciones:

- Crear varias galerías de imágenes compartidas. Cada cuenta de laboratorio solo se puede conectar a una galería de imágenes compartidas, por lo que esta opción también requerirá la creación de varias cuentas de laboratorio.
- Use una sola galería de imágenes compartidas entre varias cuentas de laboratorio. En este caso, cada cuenta de laboratorio solo puede habilitar las imágenes aplicables a los laboratorios de esa cuenta.

## <a name="naming"></a>Nomenclatura

Cuando empiece a trabajar con Azure Lab Services, se recomienda establecer convenciones de nomenclatura para los grupos de recursos, las cuentas de laboratorio, los laboratorios y la galería de imágenes compartidas. Aunque las convenciones de nomenclatura que establezca serán únicas para las necesidades de su organización, en la tabla siguiente se describen las directrices generales:

| Tipo de recurso | Role | Patrón sugerido | Ejemplos |
| ------------- | ---- | ----------------- | -------- | 
| Resource group | Contiene una o varias cuentas de laboratorio y una o varias galerías de imágenes compartidas. | \<organization short name\>-\<environment\>-rg<ul><li>**Organization short name** (Nombre corto de la organización): identifica el nombre de la organización que admite el grupo de recursos.</li><li>**Environment** (Entorno): identifica el entorno del recurso, por ejemplo, *piloto* o *producción*.</li><li>**Rg**: significa el tipo de recurso: grupo de recursos (del inglés *resource group*).</li></ul> | contosouniversitylabs-rg<br/>contosouniversitylabs-pilot-rg<br/>contosouniversitylabs-prod-rg |
| Cuenta de laboratorio | Contiene uno o varios laboratorios. | \<organization short name\>-\<environment\>-la<ul><li>**Organization short name** (Nombre corto de la organización): identifica el nombre de la organización que admite el grupo de recursos.</li><li>**Environment** (Entorno): identifica el entorno del recurso, por ejemplo, *piloto* o *producción*.</li><li>**La**: significa el tipo de recurso: cuenta de laboratorio (del inglés *lab account*).</li></ul> | contosouniversitylabs-la<br/>mathdeptlabs-la<br/>sciencedeptlabs-pilot-la<br/>sciencedeptlabs-prod-la |
| Laboratorio | Contiene una o varias máquinas virtuales. |\<class name\>-\<timeframe\>-\<educator identifier\><ul><li>**Class name** (Nombre de clase): identifica el nombre de la clase que admite el laboratorio.</li><li>**Período de tiempo**: identifica el período de tiempo en el que se ofrece la clase.</li>**Educator identifier** (Identificador del educador): identifica al educador propietario del laboratorio.</li></ul> | CS1234-fall2019-johndoe<br/>CS1234-spring2019-johndoe |
| Galería de imágenes compartidas | Contiene una o más versiones de imágenes de máquina virtual. | \<organization short name\>gallery | contosouniversitylabsgallery |

Para más información sobre la asignación de nombres a otros recursos de Azure, consultes [Convenciones de nomenclatura para recursos de Azure](/azure/architecture/best-practices/naming-conventions).

## <a name="regionslocations"></a>Regiones o ubicaciones

Al configurar los recursos de Azure Lab Services, se le pedirá que proporcione una región o ubicación del centro de datos que hospedará el recurso. En las secciones siguientes se describe cómo una región o una ubicación pueden afectar a cada recurso implicado en la configuración de un laboratorio.

### <a name="resource-group"></a>Resource group

La región especifica el centro de datos donde se almacena la información sobre un grupo de recursos. Los recursos de Azure contenidos en el grupo de recursos pueden estar en otras regiones que no sean la principal.

### <a name="lab-account"></a>Cuenta de laboratorio

La ubicación de una cuenta de un laboratorio indica la región en que existe un recurso.  

### <a name="lab"></a>Laboratorio    

La ubicación en la que existe un laboratorio varía en función de estos factores:

  - **La cuenta de laboratorio está emparejada con una red virtual**
  
    Puede [emparejar una cuenta de laboratorio con una red virtual](./how-to-connect-peer-virtual-network.md) cuando están en la misma región.  Cuando una cuenta de laboratorio se empareja con una red virtual, los laboratorios se crean automáticamente en la misma región que la cuenta de laboratorio y la red virtual.

    > [!NOTE]
    > Cuando una cuenta de laboratorio se empareja con una red virtual, la opción **Permitir al creador del laboratorio seleccionar la ubicación correspondiente** está deshabilitada. Para más información, consulte [Permitir al creador del laboratorio seleccionar la ubicación correspondiente](./allow-lab-creator-pick-lab-location.md).
    
  - **No hay ninguna red virtual emparejada *y* no se permite a los creadores del laboratorio elegir la ubicación de este**
  
    Cuando *no* hay ninguna red virtual emparejada con la cuenta de laboratorio y [no se permite *a los* creadores del laboratorio elegir la ubicación de este](./allow-lab-creator-pick-lab-location.md), los laboratorios se crean automáticamente en una región que tenga capacidad de máquina virtual disponible.  En concreto, Azure Lab Services busca disponibilidad en [regiones que se encuentran en la misma geografía que la cuenta de laboratorio](https://azure.microsoft.com/global-infrastructure/regions).

  - **No hay ninguna red virtual emparejada *y* se permite a los creadores del laboratorio elegir la ubicación de este**
       
    Cuando *no* hay ninguna red virtual emparejada y *se permite* a los [creadores del laboratorio elegir la ubicación de este](./allow-lab-creator-pick-lab-location.md), el creador del laboratorio puede seleccionar las ubicaciones según la capacidad disponible.

> [!NOTE]
> Para que una región tenga suficiente capacidad de máquina virtual, es importante que primero solicite capacidad a través de la cuenta de laboratorio al crear el laboratorio.

Una regla general es establecer la región de un recurso en la más próxima a sus usuarios. En el caso de los laboratorios, esto significa crear el laboratorio lo más cerca posible de sus alumnos. En el caso de los cursos en línea donde los alumnos están ubicados en cualquier parte del mundo, use su mejor criterio para crear un laboratorio que esté ubicado en un lugar central. También puede dividir una clase en varios laboratorios de acuerdo con las regiones de los alumnos.

## <a name="vm-sizing"></a>Tamaño de máquina virtual

Cuando los administradores o creadores de laboratorios crean un laboratorio, pueden elegir entre distintos tamaños de máquina virtual, según las necesidades de su aula. Recuerde que la disponibilidad del tamaño depende de la región en la que se encuentra la cuenta de laboratorio.

En la tabla siguiente, observe que varios de los tamaños de máquina virtual se asignan a más de una serie de máquinas virtuales.  En función de la disponibilidad de la capacidad, Lab Services puede usar cualquiera de las series de máquinas virtuales que se muestran para un tamaño de máquina virtual.  Por ejemplo, el tamaño de máquina virtual *Pequeño* se asigna al uso de las series de máquina virtual [Standard_A2_v2](../virtual-machines/av2-series.md) o [Standard_A2](../virtual-machines/sizes-previous-gen.md#a-series).  Al elegir *Pequeño* como tamaño de máquina virtual para el laboratorio, Lab Services intentará usar primero la serie *Standard_A2_v2*.  Sin embargo, cuando no haya capacidad suficiente disponible, Lab Services usará la serie *Standard_A2* en su lugar.  El precio viene determinado por el tamaño de la máquina virtual y es el mismo independientemente de la serie de máquinas virtuales que Lab Services use para ese tamaño específico. Para más información sobre los precios de cada tamaño de máquina virtual, lea la [guía de precios de Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).


| Size | Especificaciones mínimas | Serie | Sugerencia de uso |
| ---- | ----- | ------ | ------------- |
| Pequeña| <ul><li>2&nbsp;núcleos</li><li>3,5 gigabytes (GB) de RAM</li> | [Standard_A2_v2](../virtual-machines/av2-series.md) y [Standard_A2](../virtual-machines/sizes-previous-gen.md#a-series) | Es el más adecuado para la línea de comandos, la apertura del explorador web, los servidores web de poco tráfico o las bases de datos pequeñas o medianas. |
| Media | <ul><li>4&nbsp;núcleos</li><li>7&nbsp;GB&nbsp;de RAM</li> | [Standard_A4_v2](../virtual-machines/av2-series.md) y [Standard_A3](../virtual-machines/sizes-previous-gen.md#a-series) | Es el más adecuado para bases de datos relacionales, análisis y almacenamiento en caché en memoria. |
| Mediano (virtualización anidada) | <ul><li>4&nbsp;núcleos</li><li>16&nbsp;GB&nbsp; de RAM</li></ul> | [Standard_D4s_v3](../virtual-machines/dv3-dsv3-series.md#dsv3-series) | Es el más adecuado para bases de datos relacionales, análisis y almacenamiento en caché en memoria.  Este tamaño admite la virtualización anidada.
| grande | <ul><li>8&nbsp;núcleos</li><li>16&nbsp;GB&nbsp; de RAM</li></ul>  | [Standard_A8_v2](../virtual-machines/av2-series.md) y [Standard_A7](../virtual-machines/sizes-previous-gen.md#a-series) | Es el más adecuado para aplicaciones que necesitan CPU más rápidas, un mejor rendimiento de los discos locales, bases de datos grandes y cachés de memoria grandes. |
| Grande (virtualización anidada) | <ul><li>8&nbsp;núcleos</li><li>32&nbsp;GB&nbsp; de RAM</li></ul>  | [Standard_D8s_v3](../virtual-machines/dv3-dsv3-series.md#dsv3-series) | Es el más adecuado para aplicaciones que necesitan CPU más rápidas, un mejor rendimiento de los discos locales, bases de datos grandes y cachés de memoria grandes.  Este tamaño admite la virtualización anidada. |
| GPU pequeña (visualización) | <ul><li>6&nbsp;núcleos</li><li>56&nbsp;GB&nbsp;de RAM</li>  | [Standard_NV6](../virtual-machines/nv-series.md) | Es el más adecuado para la visualización, la realización de streaming, los juegos y la codificación de forma remota con marcos como OpenGL y DirectX. |
| GPU pequeña (proceso) | <ul><li>6&nbsp;núcleos</li><li>56&nbsp;GB&nbsp;de RAM</li></ul>  | [Standard_NC6](../virtual-machines/nc-series.md) y [Standard_NC6s_v3](../virtual-machines/ncv3-series.md) |Es el más adecuado para aplicaciones con un uso intensivo de proceso, como inteligencia artificial y aprendizaje profundo. |
| GPU mediana (visualización) | <ul><li>12&nbsp;núcleos</li><li>112&nbsp;GB&nbsp;de RAM</li></ul>  | [Standard_NV12](../virtual-machines/nv-series.md), [Standard_NV12s_v3](../virtual-machines/nvv3-series.md) y [Standard_NV12s_v2](../virtual-machines/sizes-previous-gen.md#nvv2-series)  | Es el más adecuado para la visualización, la realización de streaming, los juegos y la codificación de forma remota con marcos como OpenGL y DirectX. |

## <a name="manage-identity"></a>Administración de identidades

Mediante el [control de acceso basado en rol (RBAC) de Azure](../role-based-access-control/overview.md) para acceder a las cuentas de laboratorio y los laboratorios, puede asignar los siguientes roles:

- **Propietario** de la cuenta del laboratorio

    A un administrador que crea una cuenta de laboratorio se le asigna automáticamente el rol de propietario de la cuenta de laboratorio. El rol de propietario puede hacer lo siguiente:
     - Cambiar la configuración de la cuenta de laboratorio.
     - Conceder a otros administradores acceso a la cuenta de laboratorio como propietarios o colaborador.
     - Conceder a los educadores acceso a los laboratorios como creadores, propietarios o colaboradores.
     - Crear y administrar todos los laboratorios de la cuenta de laboratorio.

- **Colaborador** de la cuenta de laboratorio

    Un administrador con el rol de colaborador asignado puede hacer lo siguiente:
    - Cambiar la configuración de la cuenta de laboratorio.
    - Crear y administrar todos los laboratorios de la cuenta de laboratorio.

    Sin embargo, el colaborador *no puede* conceder a otros usuarios acceso a cuentas de laboratorio o laboratorios.

- **Creador de laboratorio**

    Para crear laboratorios en una cuenta de laboratorio, un educador debe ser miembro del rol Creador de laboratorio.  Un educador que crea un laboratorio se agrega automáticamente como propietario de un laboratorio. Para más información, consulte [Incorporación de un usuario al rol Creador de laboratorio](./tutorial-setup-lab-account.md#add-a-user-to-the-lab-creator-role). 

- **Propietario** o **colaborador** del laboratorio
  
    Un educador con un rol de propietario o colaborador de laboratorio puede ver y cambiar la configuración de un laboratorio. La persona también debe ser miembro del rol Lector de la cuenta de laboratorio.

    Una diferencia importante entre los roles de propietario y colaborador del laboratorio es que solo el propietario puede conceder acceso a otros usuarios para administrar un laboratorio. Un colaborador *no puede* conceder a otros usuarios acceso para administrar un laboratorio.

- **Galería de imágenes compartidas**

    Al asociar una galería de imágenes compartidas a una cuenta de laboratorio, se concede acceso automáticamente a los propietarios y colaboradores de la cuenta de laboratorio y a los creadores, propietarios y colaboradores del laboratorio para ver y guardar imágenes en la galería.

Al asignar roles, estas sugerencias le serán de utilidad:

   - Normalmente, solo los administradores deben ser miembros de un rol de colaborador o propietario de la cuenta de laboratorio. La cuenta de laboratorio puede tener más de un propietario o colaborador.
   - Para proporcionar a los educadores la posibilidad de crear nuevos laboratorios y administrar los laboratorios creados, solo necesita asignarles el rol de creador del laboratorio.
   - Para proporcionar a los educadores la posibilidad de administrar laboratorios específicos, pero *no* la de crear laboratorios, asígneles el rol de propietario o colaborador para cada laboratorio que vayan a administrar. Por ejemplo, puede que quiera permitir que un profesor titular y un profesor ayudante sean propietarios conjuntamente de un laboratorio. Para más información, consulte [Adición de propietarios a un laboratorio](./how-to-add-user-lab-owner.md).

## <a name="content-filtering"></a>Filtrado de contenido

Puede que la escuela tenga que filtrar contenido para evitar que los alumnos accedan a sitios web inadecuados.  Por ejemplo, para cumplir con la [Ley de Protección de la Infancia en Internet (Children's Internet Protection Act, CIPA)](https://www.fcc.gov/consumers/guides/childrens-internet-protection-act).  Lab Services no ofrece compatibilidad integrada con el filtrado de contenido.

Hay dos enfoques que las escuelas suelen tener en cuenta para filtrar contenido:
- Configurar un firewall para filtrar el contenido en el nivel de red.
- Instalar software de terceros directamente en cada equipo que realiza el filtrado de contenido.

Lab Services no admite actualmente el primer enfoque.  Lab Services hospeda la red virtual de cada laboratorio dentro de una suscripción de Azure administrada por Microsoft.  Como resultado, no se puede acceder a la red virtual subyacente para realizar el filtrado de contenido en el nivel de red.  Para más información sobre la arquitectura de Lab Services, lea el artículo [Aspectos básicos de la arquitectura en Azure Lab Services](./classroom-labs-fundamentals.md).

En su lugar, se recomienda el segundo enfoque, que consiste en instalar software de terceros en la máquina virtual de plantilla de cada laboratorio.  Es necesario resaltar algunos puntos clave como parte de esta solución:
- Si tiene previsto usar la [configuración de apagado automático](./cost-management-guide.md#automatic-shutdown-settings-for-cost-control), deberá desbloquear varios nombres de host de Azure con el software de terceros.  La configuración de apagado automático usa una extensión de diagnóstico que debe ser capaz de comunicarse con Lab Services.  De lo contrario, la configuración de apagado automático no se habilitará para el laboratorio.
- Es posible que también quiera que cada alumno use una cuenta que no sea de administrador en su máquina virtual para que no pueda desinstalar el software de filtrado de contenido.  De forma predeterminada, Lab Services crea una cuenta de administrador que cada alumno usa para iniciar sesión en su máquina virtual.  Es posible agregar una cuenta que no sea de administrador mediante una imagen especializada, pero hay algunas limitaciones conocidas.

Si su escuela necesita realizar el filtrado de contenido, póngase en contacto con nosotros en los [foros de Azure Lab Services](https://techcommunity.microsoft.com/t5/azure-lab-services/bd-p/AzureLabServices) para obtener más información.

## <a name="endpoint-management"></a>Administración de puntos de conexión

Muchas herramientas de administración de puntos de conexión, como [Microsoft Endpoint Manager](https://techcommunity.microsoft.com/t5/azure-lab-services/configuration-manager-azure-lab-services/ba-p/1754407), requieren que las máquinas virtuales Windows tengan identificadores de seguridad de máquina (SID) únicos.  El uso de SysPrep para crear una imagen *generalizada* normalmente garantiza que cada máquina Windows tendrá un nuevo SID de máquina único generado cuando la máquina virtual arranca a partir de la imagen.

Con Lab Services, incluso si usa una imagen *generalizada* para crear un laboratorio, la máquina virtual de plantilla y las máquinas virtuales de los alumnos tendrán el mismo SID de máquina.  Las máquinas virtuales tienen el mismo SID porque la imagen de la máquina virtual de la plantilla está en un estado *especializado* cuando se publica para crear las máquinas virtuales de los alumnos.

Por ejemplo, las imágenes de Azure Marketplace se generalizan.  Si crea un laboratorio a partir de la imagen de Marketplace de Win 10 y publica la máquina virtual de plantilla, todas las máquinas virtuales de los alumnos dentro de un laboratorio tendrán el mismo SID de máquina que la máquina virtual de plantilla.  Los SID de la máquina se pueden comprobar mediante una herramienta como [PsGetSid](/sysinternals/downloads/psgetsid).

Si planea usar una herramienta de administración de puntos de conexión o un software similar, se recomienda probarla con máquinas virtuales de laboratorio para asegurarse de que funciona correctamente cuando los SID de la máquina son los mismos.  

## <a name="pricing"></a>Precios

### <a name="azure-lab-services"></a>Azure Lab Services

Para información sobre los precios, consulte [Precios de Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

### <a name="shared-image-gallery"></a>Galería de imágenes compartidas

También debe tener en cuenta los precios del servicio Shared Image Gallery si tiene previsto usar galerías de imágenes compartidas para almacenar y administrar versiones de imágenes. 

La creación de una galería de imágenes compartidas y la asociación a su cuenta de laboratorio es gratuita. No se generan gastos hasta que se guarda una versión de la imagen en la galería. El precio por usar una galería de imágenes compartidas es bastante insignificante, pero es importante comprender cómo se calcula, ya que no se incluye en los precios de Azure Lab Services.  

#### <a name="storage-charges"></a>Cargos de almacenamiento

Para almacenar versiones de imágenes, una galería de imágenes compartidas usa discos estándar administrados por unidades de disco duro (HDD) de forma predeterminada.  Se recomienda usar discos administrados por HDD al usar Shared Image Gallery con Lab Services.  El tamaño del disco administrado por HDD que se usa depende del tamaño de la versión de la imagen que se almacena.  Lab Services admite tamaños de imagen y disco de hasta 128 GB.  Para información sobre los precios, consulte [Precios de Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks/).

#### <a name="replication-and-network-egress-charges"></a>Cargos por salida de replicación y red

Cuando se guarda una versión de una imagen mediante una máquina virtual de plantilla de laboratorio, Azure Lab Services la almacena primero en una región de origen y, luego, replica automáticamente la versión de la imagen de origen en una o varias regiones de destino. 

Es importante tener en cuenta que Azure Lab Services replica automáticamente la versión de la imagen de origen en todas las [regiones de destino dentro de la geografía](https://azure.microsoft.com/global-infrastructure/regions/) donde se encuentra el laboratorio. Por ejemplo, si el laboratorio está en la geografía de EE. UU., se replica una versión de la imagen en cada una de las ocho regiones que existen en EE. UU.

Un cargo por salida de red se produce cuando una versión de una imagen se replica de la región de origen a regiones de destino adicionales. La cantidad que se cobra se basa en el tamaño de la versión de la imagen cuando los datos de la imagen se transfieren inicialmente desde la región de origen.  Para más información sobre los precios, consulte [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/).

Los clientes de [soluciones de educación](https://www.microsoft.com/licensing/licensing-programs/licensing-for-industries?rtc=1&activetab=licensing-for-industries-pivot:primaryr3) pueden quedar eximidos de los cargos de salida. Para más información, póngase en contacto con su administrador de cuentas. 

Para más información, consulte "¿Qué programas de transferencia de datos hay para clientes académicos y cuáles son los requisitos para optar a ellos?" en la sección de preguntas más frecuentes de la página [Programas para instituciones educativas](https://azure.microsoft.com/pricing/details/bandwidth/).

#### <a name="pricing-example"></a>Ejemplo de precios

Echemos un vistazo a un ejemplo del costo de guardar una imagen de máquina virtual de plantilla en una galería de imágenes compartidas. Considere los casos siguientes:

- Tiene una imagen de máquina virtual personalizada.
- Va a guardar dos versiones de la imagen.
- El laboratorio está en Estados Unidos, que tiene ocho regiones en total.
- Cada versión de la imagen tiene un tamaño de 32 GB; como resultado, el precio del disco administrado por HDD es de 1,54 USD al mes.

El costo total al mes se calcula como:

* *Número de imágenes &times; número de versiones &times; número de réplicas &times; precio del disco administrado = costo total al mes*

En este ejemplo, el costo es:

* 1 imagen personalizada (32 GB) &times; 2 versiones &times; 8 regiones de EE. UU. &times; 1,54 USD = 24,64 USD al mes

> [!NOTE]
> El cálculo anterior solo se usa con fines de ejemplo. Cubre los costos de almacenamiento asociados al uso de Shared Image Gallery y *no* incluye los costos de salida. Para información sobre los precios reales de almacenamiento, consulte [Precios de Managed Disks](https://azure.microsoft.com/en-us/pricing/details/managed-disks/).

#### <a name="cost-management"></a>Administración de costos

Es importante que los administradores de la cuenta de laboratorio administren los costos mediante la eliminación rutinaria de las versiones de imágenes innecesarias de la galería. 

No elimine la replicación en regiones específicas para reducir los costos, aunque esta opción existe en la galería de imágenes compartidas. Los cambios en la replicación pueden tener efectos negativos sobre la capacidad de Azure Lab Services para publicar máquinas virtuales a partir de imágenes guardadas en una galería de imágenes compartidas.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la configuración y la administración de laboratorios, consulte:

- [Guía de configuración de una cuenta de laboratorio](account-setup-guide.md)  
- [Guía de instalación de un laboratorio](setup-guide.md)  
- [Cost Management para laboratorios](cost-management-guide.md)  
- [Uso de Azure Lab Services en Teams](lab-services-within-teams-overview.md)