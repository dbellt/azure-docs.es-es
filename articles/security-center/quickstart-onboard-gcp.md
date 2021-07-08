---
title: Conexión de la cuentas de GCP a Azure Security Center
description: Supervisión de los recursos de GCP desde Azure Security Center
author: memildin
ms.author: memildin
ms.date: 02/08/2021
ms.topic: quickstart
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 5e32c5f02191e4bd60ad4334884df34ce0bc0537
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110537460"
---
#  <a name="connect-your-gcp-accounts-to-azure-security-center"></a>Conexión de las cuentas de GCP a Azure Security Center

Las cargas de trabajo de nube abarcan normalmente varias plataformas de nube, por lo que los servicios de seguridad de la nube deben hacer lo mismo.

Azure Security Center protege las cargas de trabajo de Azure, Amazon Web Services (AWS) y Google Cloud Platform (GCP).

Al incorporar las cuentas de GCP a Security Center, se integran el centro de comandos de seguridad GCP y Azure Security Center. Por lo tanto, Security Center ofrece visibilidad y protección en ambos entornos de nube para proporcionar:

- Detección de errores de configuración de seguridad
- Una sola vista que muestra recomendaciones de Security Center y resultados del centro de comandos de seguridad GCP
- Incorporación de los recursos de GCP a los cálculos de puntuación segura de Security Center
- Integración de recomendaciones del centro de comandos de seguridad GCP basadas en el estándar CIS en el panel de cumplimiento de normativas de Security Center

En la captura de pantalla siguiente puede ver que se muestran proyectos de GCP en el panel de información general de Security Center.

:::image type="content" source="./media/quickstart-onboard-gcp/gcp-account-in-overview.png" alt-text="3 proyectos de GCP mostrados en el panel de información general de Security Center" lightbox="./media/quickstart-onboard-gcp/gcp-account-in-overview.png":::


## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponibilidad general (GA)|
|Precios:|Requiere [Azure Defender para servidores](defender-for-servers-introduction.md).|
|Roles y permisos necesarios:|**propietario** o **colaborador** en la suscripción de Azure en cuestión|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||

## <a name="connect-your-gcp-account"></a>Conexión de una cuenta de GCP

Cree un conector para cada organización que desee supervisar desde Security Center.

Al conectar las cuentas de GCP a suscripciones de Azure específicas, tenga en cuenta la [jerarquía de recursos de Google Cloud](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy#resource-hierarchy-detail) y estas directrices:

- Puede conectar las cuentas de GCP a ASC en el nivel de la *organización*.
- Puede conectar varias organizaciones a una suscripción de Azure.
- Puede conectar varias organizaciones a varias suscripciones de Azure.
- Al conectar una organización, se agregan todos los *proyectos* de esa organización a Security Center.

Siga los pasos que se indican a continuación para crear su conector de nube de GCP. 

### <a name="step-1-set-up-gcp-security-command-center-with-security-health-analytics"></a>Paso 1. Configurar el centro de comandos de seguridad GCP con el análisis del estado de seguridad

En el caso de todos los proyectos de GCP de la organización, también debe:

1. Configurar el **centro de comandos de seguridad GCP** mediante [estas instrucciones de la documentación de GCP](https://cloud.google.com/security-command-center/docs/quickstart-scc-setup).
1. Habilitar el **análisis del estado de seguridad** mediante [estas instrucciones de la documentación de GCP](https://cloud.google.com/security-command-center/docs/how-to-use-security-health-analytics).
1. Compruebe que hay datos que fluyen al centro de comandos de seguridad.

Las instrucciones para conectar el entorno de GCP para la configuración de seguridad siguen las recomendaciones de Google para consumir recomendaciones de configuración de seguridad. La integración aprovecha el centro de comandos de seguridad de Google y consumirá recursos adicionales que podrían afectar a la facturación.

La primera vez que se habilita el análisis del estado de seguridad, los datos pueden tardar varias horas en estar disponibles.


### <a name="step-2-enable-gcp-security-command-center-api"></a>Paso 2. Habilitar la API del centro de comandos de seguridad GCP

1. En la **biblioteca de API de la consola en la nube** de Google, seleccione los proyectos de la organización que desea conectar a Azure Security Center.
1. En la biblioteca de API, busque y seleccione la **API del centro de comandos de seguridad**.
1. En la página de la API, seleccione **HABILITAR**.

Obtenga más información sobre la [API del centro de comandos de seguridad](https://cloud.google.com/security-command-center/docs/reference/rest/).


### <a name="step-3-create-a-dedicated-service-account-for-the-security-configuration-integration"></a>Paso 3. Crear una cuenta de servicio dedicada para la integración de la configuración de seguridad

1. En la **consola de GCP**, seleccione un proyecto de la organización donde vaya a crear la cuenta de servicio necesaria. 

    > [!NOTE]
    > Cuando esta cuenta de servicio se agrega en el nivel de organización, se usará para acceder a los datos recopilados por Security Command Center de todos los demás proyectos habilitados en la organización. 

1. En el **menú de navegación**, en las opciones **IAM y administrador**, seleccione **Cuentas de servicio**.
1. Seleccione **CREACIÓN DE LA CUENTA DE SERVICIO**.
1. Especifique un nombre de cuenta y seleccione **Crear**.
1. Especifique el **Rol** como **Visor del administrador de Security Center** y seleccione **Continuar**.
1. La sección sobre la **concesión a los usuarios de acceso a esta cuenta de servicio** es opcional. Seleccione **Listo**.
1. Copie el **valor Correo electrónico** de la cuenta de servicio creada y guárdelo para usarlo posteriormente.
1. En el **menú de navegación**, en las opciones **IAM y administrador**, seleccione **IAM**.
    1. Cambie al nivel de organización.
    1. Seleccione **AGREGAR**.
    1. En el campo **Nuevos miembros**, pegue el **valor Correo electrónico** que copió anteriormente.
    1. Especifique el rol como **Visor del administrador de Security Center** y, a continuación, seleccione **Guardar**.
        :::image type="content" source="./media/quickstart-onboard-gcp/iam-settings-gcp-permissions-admin-viewer.png" alt-text="Establecimiento de los permisos de GCP pertinentes":::


### <a name="step-4-create-a-private-key-for-the-dedicated-service-account"></a>Paso 4. Crear una clave privada para la cuenta de servicio dedicada
1. Cambie al nivel de proyecto.
1. En el **menú de navegación**, en las opciones **IAM y administrador**, seleccione **Cuentas de servicio**.
1. Abra la cuenta de servicio dedicada y seleccione Editar.
1. En la sección **Claves**, seleccione **AGREGAR CLAVE** y, a continuación, **Creación de una clave nueva**.
1. En la pantalla Creación de una clave privada, seleccione **JSON** y, a continuación, seleccione **CREAR**.
1. Guarde este archivo JSON para su uso posterior.


### <a name="step-5-connect-gcp-to-security-center"></a>Paso 5. Conectar GCP a Security Center
1. En el menú de Security Center, seleccione **Conectores de nube**.
1. Seleccione Agregar cuenta de GCP.
1. En la página Incorporación, haga lo siguiente y, a continuación, seleccione **Siguiente**.
    1. Valide la suscripción elegida.
    1. En el campo **Nombre para mostrar**, escriba un nombre para mostrar para el conector.
    1. En el campo **Id. de la organización**, escriba el identificador de la organización. Si no lo sabe, consulte el artículo sobre [creación y administración de organizaciones](https://cloud.google.com/resource-manager/docs/creating-managing-organization).
    1. En el cuadro Archivo de **clave privada**, vaya al archivo JSON que descargó en el [paso 4. Cree una clave privada para la cuenta de servicio dedicada](#step-4-create-a-private-key-for-the-dedicated-service-account).


### <a name="step-6-confirmation"></a>Paso 6. Confirmación

Cuando el conector se ha creado correctamente y el centro de comandos de seguridad GCP se ha configurado correctamente:

- El estándar CIS de GCP se mostrará en el panel de cumplimiento de normativas de Security Center.
- Las recomendaciones de seguridad para sus recursos de GCP aparecerán en el portal de Security Center y en el panel de cumplimiento de normativas entre 5 y 10 minutos después de que finalice la incorporación:   :::image type="content" source="./media/quickstart-onboard-gcp/gcp-resources-in-recommendations.png" alt-text="Recursos y recomendaciones de GCP en la página de recomendaciones de Security Center":::


## <a name="monitoring-your-gcp-resources"></a>Supervisión de los recursos de GCP

Como se mostró anteriormente, en la página de recomendaciones de seguridad de Azure Security Center aparecen los recursos de GCP junto con los recursos de Azure y AWS, lo que ofrece una verdadera vista multinube.

Para ver todas las recomendaciones activas de los recursos por tipo de recurso, use la página de inventario de recursos de Security Center y filtre por el tipo de recurso de GCP que le interesa:

:::image type="content" source="./media/quickstart-onboard-gcp/gcp-resource-types-in-inventory.png" alt-text="Filtro de tipo de recurso de la página de inventario de recursos que muestra las opciones de GCP"::: 


## <a name="faq---connecting-gcp-accounts-to-azure-security-center"></a>Preguntas más frecuentes: conexión de las cuentas de GCP a Azure Security Center

### <a name="can-i-connect-multiple-gcp-organizations-to-security-center"></a>¿Puedo conectar varias organizaciones de GCP a Security Center?
Sí. El conector GCP de Security Center conecta los recursos de Google Cloud al nivel de *organización*. 

Cree un conector para cada organización de GCP que desee supervisar desde Security Center. Al conectar una organización, se agregan todos los proyectos de esa organización a Security Center.

Aprenda sobre la jerarquía de recursos de Google Cloud en los [documentos en línea de Google](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy).


### <a name="is-there-an-api-for-connecting-my-gcp-resources-to-security-center"></a>¿Hay una API para conectar mis recursos de GCP a Security Center?
Sí. Para crear, editar o eliminar conectores en la nube en Security Center con una API REST, consulte los detalles de [Connectors API](/rest/api/securitycenter/connectors).

## <a name="next-steps"></a>Pasos siguientes

La conexión de la cuenta de GCP forma parte de la experiencia multinube disponible en Azure Security Center. Para información relacionada, consulte la página siguiente:

- [Conexión de las cuentas de AWS a Azure Security Center](quickstart-onboard-aws.md)
- [Jerarquía de recursos de Google Cloud](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy): aprenda sobre la jerarquía de recursos de Google Cloud en los documentos en línea de Google.