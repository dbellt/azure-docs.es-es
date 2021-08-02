---
title: Envío de notificaciones de Azure Notification Hubs a aplicaciones iOS y Android
description: Obtenga información sobre las funcionalidades multiplataforma de Azure Notification Hubs.
author: sethmanheim
ms.author: sethm
ms.service: notification-hubs
ms.topic: conceptual
ms.date: 06/14/2021
ms.custom: template-concept
ms.openlocfilehash: 7158fa486483f1ff26599e47e43ee3219a23045a
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082878"
---
# <a name="send-notifications-to-android-and-ios-applications"></a>Envío de notificaciones a aplicaciones Android e iOS

En este artículo se proporciona información general sobre la aplicación de ejemplo de Azure Notification Hub creada para demostrar las funcionalidades de Azure Notification Hub en varias plataformas. La aplicación usa un escenario de encuesta de llegada, en el que la aplicación de escritorio **Contoso Land Survey** envía notificaciones, que pueden recibir las aplicaciones Contoso de Android e iOS.

Puede descargar la [muestra completa en GitHub](https://github.com/Azure/azure-notificationhubs-samples/tree/main/NotificationHubSample).

## <a name="prerequisites"></a>Prerrequisitos

Para compilar la muestra, necesita los siguientes requisitos previos:

- Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Microsoft Visual Studio 2019 o posterior. En este ejemplo se utiliza [Visual Studio 2019](https://www.visualstudio.com/products).
- Visual Studio 2019 con las cargas de trabajo siguientes instaladas:
  - SDK de .NET 5.0
  - ASP.NET y desarrollo web
  - Desarrollo de Azure
  - Desarrollo de Node.js
  - [Herramientas de desarrollo de aplicaciones UWP](/windows/uwp/get-started/get-set-up)
- Cuenta de Firebase para habilitar las notificaciones push para Android.
- Cuenta de desarrollador de Apple para habilitar las notificaciones push para iOS.
- Una instancia de base de datos SQL Server, hospedada en Azure.
- Un espacio de nombres y un centro de Azure Notification Hub.

## <a name="sample-architecture"></a>Arquitectura de muestra

La solución consta de los siguientes componentes:

- Instancia de Azure Notification Hub: un espacio de nombres y un centro de ANH configurados en el [Azure Portal](https://portal.azure.com).
- Base de datos SQL Server: una instancia de base de datos SQL Server, configurada en el [Azure Portal](https://portal.azure.com).
- Back-end de la aplicación ASP.NET: un back-end de API web basado en .NET 5.0, que se conecta con el centro de notificaciones y se hospeda como un Azure App Service.
- Aplicación UWP de Windows: una aplicación UWP compilada con React Native y que actúa como una aplicación de "administrador" que envía información de noticias y encuestas a varios usuarios y grupos de encuestas. La aplicación también ayuda a crear nuevos usuarios y a editar grupos a los que se asigna un usuario.
- Aplicaciones cliente de iOS y Android: aplicaciones "Land Survey" creadas con React Native. Estas aplicaciones muestran a los usuarios la información enviada por la aplicación de administrador de UWP.

## <a name="sample-folder-structure"></a>Estructura de carpetas de muestra

La aplicación de ejemplo en GitHub contiene las siguientes carpetas:

- **NotificationHub.Sample.API:** una solución Visual Studio 2019 ASP.NET de API web que actúa como back-end.
- **app:** una aplicación multiplataforma React Native que permite enviar notificaciones con un inicio de sesión de administrador y, a continuación, recibir notificaciones con un inicio de sesión de usuario de encuesta.
- **azure-template:** plantillas Azure Resource Manager (`parameters.json` y `template.json`) que puede usar para implementar todos los recursos necesarios para configurar esta implementación en la suscripción de Azure. Para obtener más información sobre implementación de plantillas de Resource Manager, consulte [Creación e implementación de plantillas de ARM mediante el Azure Portal](/azure/azure-resource-manager/templates/quickstart-create-templates-use-the-portal).

## <a name="sample-overview"></a>Información general de la muestra

En las secciones siguientes se proporciona información general de los componentes que componen la muestra.

### <a name="controllers"></a>Controladores

#### <a name="authentication"></a>Authentication

Los métodos siguientes de AuthenticateController.cs se usan para autenticar a un usuario que ha iniciado sesión:

```cs
[HttpPost]
[Route("login")]
public async Task<IActionResult> Login([FromBody] LoginModel model)
{
   var user = await userManager.FindByNameAsync(model.Username);
    if (user != null && await userManager.CheckPasswordAsync(user, model.Password))
    {
        var userRoles = await userManager.GetRolesAsync(user);

        var authClaims = new List<Claim>
        {
            new Claim(ClaimTypes.Name, user.UserName),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
        };

        foreach (var userRole in userRoles)
        {
            authClaims.Add(new Claim(ClaimTypes.Role, userRole));
        }

        var authSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["JWT:Secret"]));

        var token = new JwtSecurityToken(
            issuer: _configuration["JWT:ValidIssuer"],
            audience: _configuration["JWT:ValidAudience"],
            expires: DateTime.Now.AddHours(3),
            claims: authClaims,
            signingCredentials: new SigningCredentials(authSigningKey, SecurityAlgorithms.HmacSha256)
            );

        UserDetails userDetails = new UserDetails();
        userDetails.FirstName = model.Username;
        userDetails.LastName = model.Username;
        userDetails.UserName = model.Username;

        return Ok(new
        {
            token = new JwtSecurityTokenHandler().WriteToken(token),
            expiration = token.ValidTo,
            username = model.Username,
            email = user.Email,
            role = userRoles != null ? userRoles[0] : "Site-Manager",
            user = userDetails
        });
    }
    return Unauthorized();
}

[HttpPost]
[Route("register")]
public async Task<IActionResult> Register([FromBody] RegisterModel model)
{
    var userExists = await userManager.FindByNameAsync(model.Username);
    if (userExists != null)
        return StatusCode(StatusCodes.Status500InternalServerError, new Response { Status = "Error", Message = "User already exists!" });

    ApplicationUser user = new ApplicationUser()
    {
        Email = model.Email,
        SecurityStamp = Guid.NewGuid().ToString(),
        UserName = model.Username
    };
    var result = await userManager.CreateAsync(user, model.Password);
    if (!result.Succeeded)
        return StatusCode(StatusCodes.Status500InternalServerError, new Response { Status = "Error", Message = "User creation failed! Please check user details and try again." });

    if (!await roleManager.RoleExistsAsync(UserRoles.SiteManager))
        await roleManager.CreateAsync(new IdentityRole(UserRoles.SiteManager));

    if (await roleManager.RoleExistsAsync(UserRoles.SiteManager))
    {
        await userManager.AddToRoleAsync(user, UserRoles.SiteManager);
    }

    return Ok(new Response { Status = "Success", Message = "User created successfully!" });
}
```

### <a name="dashboard"></a>Panel

El controlador de panel de DashboardController.cs devuelve toda la información de notificación:

```cs
public class DashboardController : ControllerBase
{
    private readonly ApplicationDbContext _db;
    private readonly INotificationService _notificationService;

    public DashboardController(ApplicationDbContext dbContext, INotificationService notificationService)
    {
        _db = dbContext;
        _notificationService = notificationService;
    }

    [HttpGet("insights")]
    public async Task<IActionResult> GetDashboardInsight(string duration)
    {
        DashboardInsight dashboardInsight = new DashboardInsight();

        dashboardInsight.DeviceTrends = await _notificationService.GetAllRegistrationInfoAsync();

        var notificationMessages = _db.NotificationMessages.ToList();

        switch (duration)
        {
            case "Daily":
                {
                    dashboardInsight.NotificationTrends = _db.NotificationMessages
                                                            .GroupBy(m => m.SentTime.Date)
                                                            .Select(m => new NotificationTrend()
                                                            {
                                                                Timestamp = m.Key.ToShortDateString(),
                                                                NotificationsSent = m.Count()
                                                            }).ToList();
                }
                break;
            case "Weekly":
                {
                    dashboardInsight.NotificationTrends = notificationMessages
                                                            .GroupBy(m => WeekNumber(m.SentTime.Date))
                                                            .Select(m => new NotificationTrend()
                                                            {
                                                                Timestamp = FirstDateOfWeekISO8601(DateTime.Now.Year, m.Key).ToShortDateString(),
                                                                NotificationsSent = m.Count()
                                                            }).ToList();
                }
                break;
            case "Monthly":
                {
                    dashboardInsight.NotificationTrends = _db.NotificationMessages
                                                            .GroupBy(m => m.SentTime.Date.Month)
                                                            .Select(m => new NotificationTrend()
                                                            {
                                                                Timestamp = m.Key + "-" + DateTime.Now.Year,
                                                                NotificationsSent = m.Count()
                                                            }).ToList();
                }
                break;
            default:
                break;
        }

        dashboardInsight.TotalGroups = _db.SurveyGroups.Count();
        dashboardInsight.TotalUsers = _db.Users.Count();
        dashboardInsight.TotalNotificationsSent = _db.NotificationMessages.Count();

        return Ok(dashboardInsight);
    }
```

### <a name="front-end"></a>Front-end

Para llamar a cualquier API de back-end, la muestra crea el servicio notification.service.js, que realiza una llamada API real. Este código se encuentra en app\data\services\notification.service.js:

```nodejs
export const sendNotificationAPI = async (userInfo) => {
  let url = `${api}notification/send`;
  let authHeader = await getAuthHeaders();
  return await post(url, userInfo, { ...authHeader });
};

export const getNotificationsAPI = async () => {
  let url = `${api}notification/get`;
  let authHeader = await getAuthHeaders();
  return await get(url, { ...authHeader });
};
```

### <a name="manager-application"></a>Aplicación de administrador

Esta muestra tiene una aplicación para UWP compilada con React Native y que actúa como una aplicación de "administrador" que envía información de noticias y encuestas a varios usuarios y grupos de encuestas. La aplicación también ayuda a crear nuevos usuarios y editar grupos a los que se asigna un usuario.

Para el inicio de sesión del administrador, use el siguiente punto de conexión y el cuerpo POST para generar sus credenciales de usuario. Puede usar cualquier cliente REST HTTP:

#### <a name="endpoint"></a>Punto de conexión

```http
POST {{endpoint}}api/authenticate/register-admin
```

#### <a name="body"></a>Cuerpo

```http
{
  "username": "<USER_NAME>",
  "email": "<EMAIL>",
  "password": "<PASSWORD>"
}
```

Una vez registrado, debería poder iniciar sesión en la aplicación para UWP con las mismas credenciales.

### <a name="notification-controller"></a>Controlador de notificación

El código siguiente, en NotificationController.cs, obtiene y envía notificaciones:

```cs
[Produces("application/json")]
[Consumes("application/json")]
[HttpPost("send")]
public async Task<ActionResult> SendNotification([FromBody] NotificationMessage notificationMessage)
{
    try
    {
        List<string> tags = new List<string>();

        // attach survey group and user information with notificationMessage
        notificationMessage.SurveyGroupTags.ForEach(surveyGroupId =>
        {
            var group = _db.SurveyGroups.Where(g => g.Id == surveyGroupId).FirstOrDefault();
            if (group != null)
            {
                notificationMessage.SurveyGroups.Add(group);
                tags.Add($"group:{group.GroupName.Replace(' ', '-')}");
            }
        });

        notificationMessage.UserTags.ForEach(userId =>
        {
            var user = _db.Users.Where(u => u.Id == userId).FirstOrDefault();
            if (user != null)
            {
                notificationMessage.Users.Add(user);
                tags.Add($"username:{user.UserName}");
            }
        });
        _db.NotificationMessages.Add(notificationMessage);

        // send template notification
        var notification = new Dictionary<string, string>();
        notification.Add("title", notificationMessage.NotificationTitle);
        notification.Add("message", notificationMessage.NotificationDescription);

        var res = await _notificationService.RequestNotificationAsync(notificationMessage, tags, HttpContext.RequestAborted);

        await _db.SaveChangesAsync();
        return Ok(notificationMessage);
    }
    catch (Exception ex)
    {
        return BadRequest(ex.Message);
    }
}

[Produces("application/json")]
[HttpGet("get")]
public async Task<ActionResult> Get()
{
    try
    {
        var surveyGroups = _db.NotificationMessages.Include(message => message.SurveyGroups).Include(message => message.Users).ToList();
        return Ok(surveyGroups);
    }
    catch (Exception ex)
    {
        return BadRequest();
    }
}
```

### <a name="notification-service"></a>Servicio de notificación

El servicio de notificación ubicado en **NotificationHub.Sample.API/NotificationHub.Sample.API/Services/Notifications/INotificationService.cs** dispone de métodos para crear y eliminar la instalación. También hay un método para enviar notificaciones a todos los usuarios registrados y para obtener toda la información de registro:

```cs
public interface INotificationService
{
   Task<bool> CreateOrUpdateInstallationAsync(DeviceInstallation deviceInstallation, CancellationToken cancellationToken);
   Task<bool> DeleteInstallationByIdAsync(string installationId, CancellationToken cancellationToken);
   Task<bool> RequestNotificationAsync(NotificationMessage notificationMessage, IList<string> tags, CancellationToken cancellationToken);
   Task<List<DeviceTrend>> GetAllRegistrationInfoAsync();
}
```

## <a name="deploy-the-solution"></a>Implementación de la solución

Para ejecutar la muestra, se precisan los siguientes requisitos previos:

- Suscripción a Azure. Si no tiene una suscripción a Azure, [cree una cuenta de Azure gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Cuenta de Firebase para configurar las notificaciones push para Android.
- Cuenta de desarrollador de Apple para configurar las notificaciones push para iOS.
- Un equipo Windows con Visual Studio 2019 instalado, junto con compatibilidad con desarrollo de aplicaciones para UWP y ASP.NET Core.
- Android SDK y Android Studio en su máquina de desarrollo si se ejecuta la aplicación Android.
- Una máquina Mac OSX con XCode y el SDK de iOS más reciente, si se ejecuta la aplicación iOS.

### <a name="deploy-resources"></a>Implementación de recursos

- **[Instancia de Azure Notification Hub](#create-resource-notification-hub):** un centro de notificaciones configurado en Azure.
- **[Una instancia de base de datos SQL Server](#create-resource-sql-database):** una base de datos SQL Server hospedada en Azure.
- **back-end de aplicación ASP.NET:** Back-end de API web creado a través de .NET 5.0, que se conecta con Azure Notification Hub hospedado como Azure App Service. Para obtener más información, consulte [Inicio rápido: Implementación de una aplicación web de ASP.NET](/azure/app-service/quickstart-dotnetcore?tabs=net50&pivots=development-environment-vs).

Si no desea implementar todos los recursos manualmente, puede usar el archivo template.jsen de Azure Resource Manager proporcionado en el repositorio de GitHub para implementar todas las instancias necesarias en un solo paso. El archivo de plantilla está disponible en el repositorio en /azure-template. Para obtener más información sobre cómo usar plantillas de Resource Manager, consulte [Tutorial: uso de plantillas de inicio rápido de Azure](/azure/azure-resource-manager/templates/template-tutorial-quickstart-template?tabs=azure-powershell).

### <a name="set-up-notifications-for-android-and-ios"></a>Configuración de notificaciones para Android e iOS

- **Firebase:** para recibir notificaciones en Android, configure el servicio Firebase y conéctelo con la instancia del centro de notificaciones de Azure. Para más información, consulte [Configuración de Google Firebase](configure-google-firebase-cloud-messaging.md).
- **Apple Push Notification Service (APNS):** para recibir notificaciones en iOS, configure el servicio APNS con su cuenta de desarrollador de Apple y conéctelo con la instancia del centro de notificaciones de Azure. Para obtener más información, consulte [Configurar Apple Push Notification Service](configure-apple-push-notification-service.md).

## <a name="build-the-solution"></a>Compilar la solución

Para compilar la muestra, siga estos pasos.

### <a name="create-resource-sql-database"></a>Creación de un recurso: base de datos SQL

[Cree una instancia de base de datos SQL Server](/azure/azure-sql/database/single-database-create-quickstart?tabs=azure-portal) en Azure Portal. Por ejemplo:

:::image type="content" source="media/uwp-react/resources-sql.png" alt-text="Recursos de instancia de SQL":::

:::image type="content" source="media/uwp-react/template-screenshot.png" alt-text="Plantilla de implementación":::

### <a name="create-resource-notification-hub"></a>Creación de un recurso: centro de notificaciones

Creación de un centro de notificaciones en Azure Portal de la siguiente manera:

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-backend"></a>Configuración del back-end

Para configurar el back-end de la aplicación, busque el archivo **/NotificationHub.Sample.API/appsettings.js** y configure la cadena de conexión de SQL Server:

```json
"ConnectionStrings": {
    "SQLServerConnectionString": "Server=tcp:<SERVER_NAME>,1433;Initial Catalog=<DB_NAME>;Persist Security Info=False;User ID=<DB_USER_NAME>;Password=<PASSWORD>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
  },
```

Reemplace `<SERVER_NAME>` por el nombre del servidor SQL Server, `<DB_NAME>` por la dirección URL de la base de datos implementada, `<DB_USER_NAME>` por el nombre de usuario configurado y `<PASSWORD>` por la contraseña configurada.

Puede ejecutar la solución API localmente o en cualquier servidor IIS, o implementarla como una instancia de Azure Web App Service. Mantenga la dirección URL de la API a mano.

### <a name="build-the-notificationhubsampleapi-application"></a>Compilación de la aplicación NotificationHub.Sample.API

1. En Visual Studio, cargue la solución /NotificationHubSample/NotificationHub.Sample.API/NotificationHub.Sample.API.sln.
2. En el menú **Compilar**, seleccione **Compilar solución**.
3. Publique la solución en Azure: en Explorador de soluciones, haga clic con el botón derecho en el proyecto **NotificationHub.Sample.API** y seleccione **Publicar**.
4. En el diálogo **Publicar**, seleccione **Azure** (primera opción) y, a continuación, seleccione **Siguiente**.
5. Seleccione **Azure App Service (Windows)** y, a continuación, seleccione **Siguiente**.
6. Se le redirigirá al siguiente diálogo, desde el que puede seleccionar el nombre de suscripción adecuado y las instancias de AppService. El back-end se implementará en Azure y se le redirigirá a una nueva dirección URL.

   :::image type="content" source="media/uwp-react/publish.png" alt-text="Publicar proyecto":::

7. Una vez publicado el back-end, agregue la dirección URL generada **aconfig.js**, que se encuentra en la carpeta **/app**. Asegúrese de anexar `/api/` después de la dirección URL.

## <a name="run-react-native-frontend-application-for-windows"></a>Ejecución de una aplicación de front-end React Native para Windows

La aplicación requiere que la aplicación móvil (ya sea Android o iOS) y la aplicación de administrador para UWP se ejecuten simultáneamente. Siga estos pasos para ejecutar ambas aplicaciones:

Abra la carpeta **app** en la ventana de terminal o shell que prefiera. A continuación, haga lo siguiente:

### <a name="windows"></a>Windows

1. Ejecute `npm install` para instalar todas las dependencias del paquete.
2. Ejecute `npm run start` para iniciar el servidor metro en una ventana de consola.
3. Abra otra ventana de terminal y ejecute `npx react-native run-windows` para ejecutar la aplicación para UWP.
4. Si se produce un error en la implementación o compilación, consulte la guía de solución de problemas.

### <a name="android"></a>Android

1. Configure Firebase en su proyecto React-Native para asegurarse de que puede usar las funcionalidades de notificación.
2. Después de configurar correctamente el proyecto Firebase, descargue el archivo **google-services.json** del portal de Firebase.
3. Reemplace **./app/android/app/google-services.json** por este nuevo archivo. Asegúrese de que el nombre del paquete de aplicación coincide con el configurado en Firebase. El nombre del paquete se configura en el archivo **AndroidManifest.xml**.
4. Para obtener más información sobre cómo configurar notificaciones en una aplicación React Native, consulte [Tutorial: envío de notificaciones push a React Native](/azure/developer/mobile-apps/notification-hubs-backend-service-react-native).
5. Una vez configuradas las notificaciones, ejecute `npm run start` para iniciar el servidor metro en una ventana de consola. Si la aplicación Windows ya se está ejecutando, puede omitir este paso.
6. En una nueva ventana de consola, ejecute `npx react-native run-android` para ejecutar la aplicación Android.

### <a name="ios"></a>iOS

1. Configure Firebase en su proyecto React-Native para asegurarse de que puede usar las funcionalidades de notificación.
2. En iOS, las notificaciones solo las pueden recibir las aplicaciones firmadas e instaladas a través de App Store o TestFlight. Debe crear una aplicación en su cuenta de desarrollador de Apple. El identificador de agrupación de la aplicación configurado en la cuenta de desarrollador de Apple debe configurarse en los archivos **Info.plist** y **Entitlements.plist** de su aplicación.
3. Para obtener más información sobre cómo configurar notificaciones en una aplicación React Native, consulte [Tutorial: envío de notificaciones push a React Native](/azure/developer/mobile-apps/notification-hubs-backend-service-react-native).
4. Una vez configuradas las notificaciones, ejecute `npm run start` para iniciar el servidor metro en una ventana de consola. Si la aplicación Windows ya se está ejecutando, puede omitir este paso.
5. En una nueva ventana de consola, ejecute `npx react-native run-ios` para ejecutar la aplicación iOS. Como se mencionó anteriormente, las notificaciones no funcionarán en iOS si se implementan localmente. No puede recibir notificaciones en el simulador de iOS.

## <a name="troubleshooting"></a>Solución de problemas

Es posible que reciba el siguiente error al ejecutar la aplicación React Native para Windows:

`error MSB4057: The target "Deploy" does not exist in the project`

Se trata de un problema conocido con React Native para Windows. Para solucionar este problema, descargue el archivo **.csproj** para los proyectos **CheckboxWindows** y **ReactNativeAsyncStorage** en Visual Studio después de abrir **app.sln** en la carpeta **app/windows**. A continuación, en el archivo **app.csproj** agregue la siguiente línea justo antes de `...</Project>`, y vuelva a cargar el proyecto:

```xml
<Target Name="Deploy"/>
```

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: envío de notificaciones push a aplicaciones React Native mediante Azure Notification Hubs](/azure/developer/mobile-apps/notification-hubs-backend-service-react-native)
- [Tutorial: envío de notificaciones push a dispositivos Android mediante el SDK de Firebase](android-sdk.md)
- [Azure Notification Hubs](notification-hubs-push-notification-overview.md)
- [React Native para Windows](https://microsoft.github.io/react-native-windows/)
