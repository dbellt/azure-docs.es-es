---
title: Patrones de aplicaciones SaaS multiinquilino
description: Aprenda sobre los requisitos y patrones de arquitectura de datos comunes de las aplicaciones de base de datos de software como servicio (SaaS) multiinquilino que se ejecutan en el entorno de la nube de Azure.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.topic: conceptual
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 01/25/2019
ms.custom: seoapril2019, sqldbrb=1
ms.openlocfilehash: bf25325f12fba6edaf106dc582f843c0c3c0adec
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693421"
---
# <a name="multi-tenant-saas-database-tenancy-patterns"></a>Patrones de inquilinato de base de datos SaaS multiinquilino
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se describen los distintos modelos de arrendamiento disponibles para una aplicación SaaS multiinquilino.

Al diseñar una aplicación SaaS multiinquilino, debe elegir cuidadosamente el modelo de inquilinato que mejor se adapte a las necesidades de su aplicación.  Un modelo de inquilinato determina cómo se asignan los datos de cada inquilino al almacenamiento.  La elección del modelo de inquilinato afecta a la administración y el diseño de la aplicación.  A veces es costoso cambiar más adelante a un modelo diferente.

## <a name="a-saas-concepts-and-terminology"></a>A. Conceptos y terminología de SaaS

En el modelo de software como servicio (SaaS), su compañía no vende *licencias* para su software. En su lugar, cada cliente paga un "alquiler" a la empresa, lo que hace que se convierta en un *inquilino* de ella.

A cambio de ese alquiler, cada inquilino recibe acceso a los componentes de su aplicación de SaaS y sus datos se almacenan en el sistema de SaaS.

El término *modelo de arrendamiento* hace referencia a la forma en que organizan los datos almacenados de los inquilinos:

- *Inquilino único:* &nbsp; cada base de datos almacena los datos de un solo inquilino.
- *Multiinquilino:* &nbsp; cada base de datos almacena los datos de varios inquilinos independientes (con mecanismos para proteger la privacidad de los datos).
- También hay disponibles modelos de arrendamiento híbrido.

## <a name="b-how-to-choose-the-appropriate-tenancy-model"></a>B. Cómo elegir el modelo adecuado de inquilinato

En general, el modelo de inquilinato no afecta a la función de una aplicación, pero es probable que afecte a otros aspectos de la solución en general.  Los siguientes criterios se utilizan para evaluar cada uno de los modelos:

- **Escalabilidad:**
    - Número de inquilinos.
    - Almacenamiento por inquilino.
    - Almacenamiento de forma agregada.
    - Carga de trabajo.

- **Aislamiento de inquilinos:** &nbsp; aislamiento de los datos y rendimiento (si la carga de trabajo de un inquilino afecta a otros).

- **Costo por inquilino:** &nbsp; costos de la base de datos.

- **Complejidad de desarrollo:**
    - Cambios en el esquema.
    - Cambios en las consultas (requeridos por el patrón).

- **Complejidad operativa:**
    - Supervisión y administración del rendimiento
    - Administración de esquemas
    - Restauración de un inquilino.
    - Recuperación ante desastres

- **Capacidad de personalización:** &nbsp; facilidad para admitir las personalizaciones del esquema que son específicas del inquilino o específicas de la clase de inquilino.

El análisis de inquilinos se centra en la capa de *datos*.  Pero tenga en cuenta durante un momento la capa de *aplicación*.  La capa de aplicación se trata como una entidad monolítica.  Si se divide la aplicación en muchos componentes pequeños, podría cambiar su elección de modelo de inquilinato.  Puede tratar algunos componentes de forma diferente a otros tanto con respecto al inquilinato como a la tecnología de almacenamiento o plataforma utilizada.

## <a name="c-standalone-single-tenant-app-with-single-tenant-database"></a>C. Aplicación de inquilino único independiente con base de datos de un solo inquilino

#### <a name="application-level-isolation"></a>Aislamiento en el nivel de aplicación

En este modelo, la aplicación entera se instala varias veces, una vez para cada inquilino.  Cada instancia de la aplicación es una instancia independiente, por lo que nunca interactúa con cualquier otra instancia independiente.  Cada instancia de la aplicación tiene un solo inquilino y, por tanto, necesita solo una base de datos.  El inquilino tiene la base de datos entera para él.

![Diseño de aplicación independiente con base de datos de un único inquilino.][image-standalone-app-st-db-111a]

Cada instancia de la aplicación se instala en un grupo de recursos de Azure independiente.  El grupo de recursos puede pertenecer a una suscripción que a su vez pertenezca al proveedor de software o al inquilino.  En cualquier caso, el proveedor puede administrar el software para el inquilino.  Cada instancia de la aplicación se configura para conectarse a su base de datos correspondiente.

Cada base de datos de inquilino se implementa como una base de datos única.  Este modelo proporciona el mayor aislamiento de base de datos.  Pero el aislamiento requiere que se hayan asignado suficientes recursos para cada base de datos para controlar sus cargas máximas.  Aquí es importante saber que no se pueden usar grupos elásticos para bases de datos implementadas en distintos grupos de recursos o en suscripciones diferentes.  Esta limitación hace que este modelo de aplicación de inquilino único independiente sea la solución más costosa desde una perspectiva de costos de base de datos generales.

#### <a name="vendor-management"></a>Administración por el proveedor

El proveedor puede tener acceso a todas las bases de datos en todas las instancias de aplicación independientes, incluso si las instancias de la aplicación se instalan en suscripciones de inquilinos diferentes.  El acceso se consigue a través de conexiones SQL.  Este acceso entre instancias puede permitir al proveedor centralizar la administración de esquemas y las consultas entre bases de datos con fines de elaboración de informes o análisis.  Si se desea este tipo de administración centralizada, se debe implementar un catálogo que asigne identificadores de inquilino a los identificadores URI de la base de datos.  Azure SQL Database proporciona una biblioteca de particionamiento que se usan juntos para proporcionar un catálogo.  La biblioteca de particionamiento se llama formalmente [biblioteca cliente Elastic Database][docu-elastic-db-client-library-536r].

## <a name="d-multi-tenant-app-with-database-per-tenant"></a>D. Aplicación multiinquilino con una base de datos por inquilino

El patrón siguiente utiliza una aplicación multiinquilino con muchas bases de datos, cada una de ellas de un único inquilino.  Se aprovisiona una nueva base de datos para cada nuevo inquilino.  La capa de aplicación se escala *verticalmente* mediante la adición de más recursos por nodo.  O la aplicación se escala *horizontalmente* agregando más nodos.  El escalado se basa en la carga de trabajo y es independiente del número o la escala de las bases de datos individuales.

![Diseño de una aplicación multiinquilino con una base de datos por inquilino][image-mt-app-db-per-tenant-132d]

#### <a name="customize-for-a-tenant"></a>Personalización para un inquilino

Al igual que el patrón de aplicación independiente, el uso de bases de datos de un único inquilino ofrece un aislamiento de inquilinos seguro.  En cualquier aplicación cuyo modelo especifique solo bases de datos de un único inquilino, el esquema para una base de datos determinada se puede personalizar y optimizar para su inquilino.  Esta personalización no afecta a otros inquilinos en la aplicación. Quizás un inquilino podría necesitar datos más allá de los campos de datos básicos que necesitan todos los inquilinos.  Además, el campo de datos adicionales podría necesitar un índice.

Con una base de datos por inquilino, la personalización del esquema para uno o varios inquilinos individuales es fácil de lograr.  El proveedor de la aplicación debe diseñar procedimientos para administrar cuidadosamente las personalizaciones de esquema a escala.

#### <a name="elastic-pools"></a>Grupos elásticos

Cuando las bases de datos se implementan en el mismo grupo de recursos, se pueden agrupar en grupos elásticos.  Los grupos proporcionan una manera rentable de compartir recursos entre varias bases de datos.  Esta opción de grupo resulta más económica que exigir que cada base de datos sea lo suficientemente grande como para dar cabida a los picos de uso que producen.  Aunque las bases de datos agrupadas comparten el acceso a los recursos, pueden mantener un alto grado de aislamiento del rendimiento.

![Diseño de aplicación multiinquilino con una base de datos por inquilino mediante un grupo elástico.][image-mt-app-db-per-tenant-pool-153p]

Azure SQL Database proporciona las herramientas necesarias para configurar, supervisar y administrar el uso compartido.  Las métricas de rendimiento en el nivel de grupo y en el nivel de base de datos están disponibles en Azure Portal y a través de los registros de Azure Monitor.  Las métricas pueden dar detalles sobre el rendimiento agregado y específico del inquilino.  Las bases de datos individuales se pueden mover entre grupos para proporcionar recursos reservados para un inquilino específico.  Estas herramientas permiten garantizar un buen rendimiento en forma rentable.

#### <a name="operations-scale-for-database-per-tenant"></a>Escala de operaciones para una base de datos por inquilino

Azure SQL Database tiene muchas características de administración diseñadas para administrar grandes cantidades de bases de datos a escala, por encima de 100 000 bases de datos.  Estas características hacen que el patrón de una base de datos por inquilino sea plausible.

Por ejemplo, suponga que un sistema tiene una base de datos de 1000 inquilinos como única base de datos.  La base de datos podría tener 20 índices.  Si el sistema pasa a tener 1000 bases de datos de un único inquilino, el número de índices aumenta hasta 20 000.  En Azure SQL Database, como parte del [ajuste automático][docu-sql-db-automatic-tuning-771a], las características de indexación automática están habilitadas de forma predeterminada.  La indexación automática administra automáticamente los 20.000 índices y sus optimizaciones de creación y eliminación.  Estas acciones automatizadas se producen dentro de una base de datos individual y no están coordinadas ni restringidas por acciones similares en otras bases de datos.  La indexación automática trata los índices de manera diferente en una base de datos ocupada que en una base de datos de menos actividad.  Este tipo de personalización de la administración de índices no sería práctico en la escala de una base de datos por inquilino si esta enorme tarea de administración tuviera que realizarse manualmente.

Otras características de administración que escalan bien incluyen las siguientes:

- Copias de seguridad integradas.
- Alta disponibilidad.
- Cifrado en el disco.
- Telemetría de rendimiento.

#### <a name="automation"></a>Automation

Las operaciones de administración se pueden incluir en un script y ofrecer con un modelo de [devops](/azure/devops/user-guide/what-is-azure-devops).  Las operaciones incluso pueden ser automatizadas y expuestas en la aplicación.

Por ejemplo, puede automatizar la recuperación de un solo inquilino a un momento anterior en el tiempo.  La recuperación solo necesita restaurar la base de datos de un solo inquilino que almacena al inquilino.  Esta restauración no influye en otros inquilinos, lo que confirma que las operaciones de administración se encuentran en el nivel granular de cada inquilino individual.

## <a name="e-multi-tenant-app-with-multi-tenant-databases"></a>E. Aplicación multiinquilino con bases de datos multiinquilino

Otro patrón disponible consiste en almacenar a muchos inquilinos en una base de datos de varios inquilinos.  La instancia de la aplicación puede tener cualquier número de bases de datos multiinquilino.  El esquema de una base de datos multiinquilino debe tener una o varias columnas de identificadores del inquilino para que los datos de cualquier inquilino determinado se puedan recuperar de forma selectiva.  Además, el esquema puede requerir algunas tablas o columnas que son utilizadas solo por un subconjunto de los inquilinos.  Sin embargo, el código estático y los datos de referencia se almacenan una sola vez y se comparten entre todos los inquilinos.

#### <a name="tenant-isolation-is-sacrificed"></a>Se sacrifica el aislamiento de inquilinos

*Datos:* &nbsp; una base de datos multiinquilino necesariamente sacrifica el aislamiento de inquilinos.  Los datos de varios inquilinos se almacenan juntos en una base de datos.  Durante el desarrollo, asegúrese de que las consultas no exponen nunca los datos de más de un inquilino.  SQL Database admite [seguridad en el nivel de fila][docu-sql-svr-db-row-level-security-947w], que puede exigir que los datos devueltos por una consulta estén en el ámbito de un solo inquilino.

*Procesamiento:* &nbsp; una base de datos multiinquilino comparte recursos de proceso y almacenamiento entre todos sus inquilinos.  La base de datos como un todo puede supervisarse para asegurarse de que está teniendo un rendimiento aceptable.  Sin embargo, el sistema de Azure no tiene ningún medio integrado para supervisar o administrar el uso de estos recursos por un inquilino individual.  Por lo tanto, la base de datos multiinquilino implica un mayor riesgo de que se produzcan molestias entre vecinos, donde la carga de trabajo de un inquilino muy activo afecta al rendimiento de los demás inquilinos en la misma base de datos.  La supervisión adicional en el nivel de aplicación puede supervisar el rendimiento en el nivel del inquilino.

#### <a name="lower-cost"></a>Menor costo

En general, las bases de datos multiinquilino tienen el costo por inquilino más bajo.  Los costos en recursos para una base de datos única son menores que para un grupo de bases de datos elásticas de tamaño equivalente.  Además, para escenarios donde los inquilinos necesitan solo un almacenamiento limitado, se pueden almacenar millones de inquilinos potenciales en una sola base de datos.  Ningún grupo elástico puede contener millones de bases de datos.  Sin embargo, una solución que contenga 1000 bases de datos por grupo, con 1000 grupos, podría llegar a la escala de millones con el riesgo de ser difícil de administrar.

A continuación se describen dos variaciones de un modelo de base de datos multiinquilino, siendo el modelo multiinquilino con particiones el más flexible y escalable.

## <a name="f-multi-tenant-app-with-a-single-multi-tenant-database"></a>F. Aplicación multiinquilino con una única base de datos multiinquilino

El patrón de base de datos multiinquilino más simple usa una base de datos única para hospedar los datos de todos los inquilinos.  Cuando se agregan más inquilinos, la base de datos se escala verticalmente con más recursos de proceso y almacenamiento.  Este escalado vertical puede ser suficiente, aunque siempre hay un límite de escala último.  Sin embargo, antes de que ese límite se alcance la base de datos pasa a ser difícil de administrar.

Las operaciones de administración que se centran en inquilinos individuales tienen una implementación más compleja en una base de datos multiinquilino.  Y a escala estas operaciones podrían resultar demasiado lentas.  Un ejemplo es una restauración a un momento determinado en el tiempo de los datos de un solo inquilino.

## <a name="g-multi-tenant-app-with-sharded-multi-tenant-databases"></a>G. Aplicación multiinquilino con bases de datos multiinquilino con particiones

La mayoría de las aplicaciones de SaaS acceden a los datos de un solo inquilino a la vez.  Este patrón de acceso permite que los datos del inquilino se distribuyan a través de varias bases de datos o particiones, donde todos los datos de cualquier inquilino determinado están contenidos en una partición.  Combinado con un patrón de base de datos multiinquilino, un modelo con particiones permite una escala casi infinita.

![Diseño de aplicación multiinquilino con bases de datos multiinquilino con particiones][image-mt-app-sharded-mt-db-174s]

#### <a name="manage-shards"></a>Administración de particiones

El particionamiento agrega complejidad tanto para el diseño como para la administración operativa.  Se requiere un catálogo en el que se va a mantener la asignación entre los inquilinos y las bases de datos.  Además, se requieren procedimientos de administración para administrar las particiones y el rellenado de inquilinos.  Por ejemplo, deben diseñarse procedimientos para agregar y eliminar particiones y para mover datos de inquilinos entre particiones.  Una forma de escalar es la adición de una nueva partición y rellenarla con los nuevos inquilinos.  En otras ocasiones se podría dividir una partición densamente poblada en dos particiones con menor densidad.  Después de que se hayan movido o eliminado algunos inquilinos, podría combinar particiones poco pobladas.  Esta combinación traería consigo una utilización de los recursos más eficiente en costos.  Los inquilinos también se pueden mover entre particiones para equilibrar las cargas de trabajo.

SQL Database proporciona una herramienta para dividir y combinar que funciona junto con la biblioteca de particionamiento y el catálogo de base de datos.  La aplicación proporcionada puede dividir y combinar particiones y puede mover datos de inquilino entre particiones.  La aplicación también mantiene el catálogo durante estas operaciones y marca a los inquilinos afectados como sin conexión antes de moverlos.  Tras el movimiento, la aplicación actualiza el catálogo de nuevo con la nueva asignación y se vuelve a marcar al inquilino como conectado.

#### <a name="smaller-databases-more-easily-managed"></a>Bases de datos menores más fácil de administrar

Mediante la distribución de los inquilinos en varias bases de datos, la solución multiinquilino con particiones da como resultado bases de datos más pequeñas que se administran con más facilidad.  Por ejemplo, la restauración de un inquilino específico a un momento anterior en el tiempo ahora implica la restauración de una base de datos más pequeña desde una copia de seguridad, en lugar de una base de datos más grande que contiene a todos los inquilinos. Se puede elegir el tamaño de la base de datos y el número de inquilinos por base de datos para equilibrar la carga de trabajo y los esfuerzos de administración.

#### <a name="tenant-identifier-in-the-schema"></a>Identificador del inquilino en el esquema

Según el enfoque de particionamiento usado, se pueden imponer restricciones adicionales en el esquema de base de datos.  La aplicación de dividir y combinar de SQL Database requiere que el esquema incluya la clave de particionamiento, que suele ser el identificador del inquilino.  El identificador del inquilino es el elemento inicial de la clave principal de todas las tablas con particiones.  El identificador del inquilino permite que la aplicación de dividir y combinar localice rápidamente y mueva los datos asociados a un inquilino específico.

#### <a name="elastic-pool-for-shards"></a>Grupo elástico para particiones

Las bases de datos multiinquilino con particiones pueden colocarse en grupos elásticos.  En general, tener muchas bases de datos de un solo inquilino en un grupo es tan rentable como tener varios inquilinos en algunas bases de datos multiinquilino.  Las bases de datos multiinquilino son beneficiosas cuando hay un gran número de inquilinos relativamente inactivos.

## <a name="h-hybrid-sharded-multi-tenant-database-model"></a>H. Modelo híbrido de base de datos multiinquilino con particiones

En el modelo híbrido, todas las bases de datos tienen el identificador del inquilino en su esquema.  Las bases de datos son capaces de almacenar a más de un inquilino y las bases de datos pueden tener particiones.  Por lo que, en el sentido del esquema, son todas bases de datos multiinquilino.  Aunque en la práctica algunas de estas bases de datos contengan a un solo inquilino.  En cualquier caso, la cantidad de inquilinos que se almacenan en una base de datos no tiene ningún efecto en el esquema de base de datos.

#### <a name="move-tenants-around"></a>Mover los inquilinos

En cualquier momento, puede mover a un inquilino determinado a su propia base de datos multiinquilino.  Y en cualquier momento, puede cambiar de opinión y mover al inquilino de nuevo a una base de datos que contiene a varios inquilinos.  También puede asignar a un inquilino a la nueva base de datos de un solo inquilino al aprovisionar la nueva base de datos.

El modelo híbrido destaca cuando existen grandes diferencias entre las necesidades de recursos de los grupos identificables de inquilinos.  Por ejemplo, suponga que los inquilinos que participan en una evaluación gratuita no tienen garantizado el mismo nivel alto de rendimiento que los inquilinos suscriptores.  La directiva podría ser que los inquilinos en la fase de evaluación gratuita se almacenan en una base de datos multiinquilino que se comparte entre todos los inquilinos de prueba gratuitos.  Cuando un inquilino de evaluación gratuita se suscribe en el nivel de servicio básico, el inquilino se puede mover a otra base de datos multiinquilino que tenga menos inquilinos.  Un suscriptor que paga por el nivel de servicio premium se podría mover a su propia base de datos de inquilino único nueva.

#### <a name="pools"></a>Grupos

En este modelo híbrido, las bases de datos de un solo inquilino para inquilinos suscriptores pueden colocarse en grupos de recursos para reducir los costos de base de datos por inquilino.  Esto también se realiza en el modelo de una base de datos por inquilino.

## <a name="i-tenancy-models-compared"></a>I. Comparación de los modelos de inquilinato

En la tabla siguiente se resumen las diferencias entre los principales modelos de inquilinato.

| Medición | Aplicación independiente | Una base de datos por inquilino | Multiinquilino con particiones |
| :---------- | :------------- | :------------------ | :------------------- |
| Escala | Media<br />1-100 | Muy alto<br />1-100.000 | Sin límite<br />1-1.000.000 |
| Aislamiento de inquilino | Muy alto | Alto | Bajo; excepto para los inquilinos únicos (que están solos en una base de datos multiinquilino). |
| Costo de base de datos por inquilino | Alta; tiene un tamaño adecuado para picos. | Baja; se utilizan grupos. | Más bajo, para inquilinos pequeños en bases de datos multiinquilino. |
| Administración y supervisión del rendimiento | Por inquilino solo | Agregado + por inquilino | Agregado; aunque es por inquilino solo para los únicos. |
| Complejidad de desarrollo | Bajo | Bajo | Media; debido al particionamiento. |
| Complejidad operativa | Baja-Alta. Individualmente simples y complejas a escala. | Baja-Media. Los patrones tienen complejidad a escala. | Baja-Alta. La administración de inquilinos individuales es compleja. |
| &nbsp; ||||

## <a name="next-steps"></a>Pasos siguientes

- [Implementación y exploración de la aplicación Wingtip multiinquilino que usa el modelo SaaS de una base de datos por inquilino con Azure SQL Database][docu-sql-db-saas-tutorial-deploy-wingtip-db-per-tenant-496y]

- [Bienvenidos a la aplicación SaaS de ejemplo Wingtip Tickets de Azure SQL Database][docu-saas-tenancy-welcome-wingtip-tickets-app-384w]


<!--  Article link references.  -->

[http-visual-studio-devops-485m]: https://www.visualstudio.com/devops/

[docu-sql-svr-db-row-level-security-947w]: /sql/relational-databases/security/row-level-security

[docu-elastic-db-client-library-536r]:elastic-database-client-library.md
[docu-sql-db-saas-tutorial-deploy-wingtip-db-per-tenant-496y]: saas-dbpertenant-get-started-deploy.md
[docu-sql-db-automatic-tuning-771a]:automatic-tuning-overview.md
[docu-saas-tenancy-welcome-wingtip-tickets-app-384w]: saas-tenancy-welcome-wingtip-tickets-app.md


<!--  Image references.  -->

[image-standalone-app-st-db-111a]: media/saas-tenancy-app-design-patterns/saas-standalone-app-single-tenant-database-11.png "Diseño de aplicación independiente con base de datos de un único inquilino."

[image-mt-app-db-per-tenant-132d]: media/saas-tenancy-app-design-patterns/saas-multi-tenant-app-database-per-tenant-13.png "Diseño de una aplicación multiinquilino con una base de datos por inquilino."

[image-mt-app-db-per-tenant-pool-153p]: media/saas-tenancy-app-design-patterns/saas-multi-tenant-app-database-per-tenant-pool-15.png "Diseño de aplicación multiinquilino con una base de datos por inquilino mediante un grupo elástico."

[image-mt-app-sharded-mt-db-174s]: media/saas-tenancy-app-design-patterns/saas-multi-tenant-app-sharded-multi-tenant-databases-17.png "Diseño de aplicación multiinquilino con bases de datos multiinquilino con particiones."