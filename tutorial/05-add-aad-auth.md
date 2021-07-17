<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="e8bcf-101">In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Single Sign-On-Authentifizierung mit Azure AD zu unterstützen.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-101">In this exercise you will extend the application from the previous exercise to support single sign-on authentication with Azure AD.</span></span> <span data-ttu-id="e8bcf-102">Dies ist notwendig, um das erforderliche OAuth-Zugriffstoken zum Aufruf der Microsoft Graph-API abzurufen.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="e8bcf-103">In diesem Schritt konfigurieren Sie die [Microsoft.Identity.Web-Bibliothek.](https://www.nuget.org/packages/Microsoft.Identity.Web/)</span><span class="sxs-lookup"><span data-stu-id="e8bcf-103">In this step you will configure the [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) library.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="e8bcf-104">Um das Speichern der Anwendungs-ID und des geheimen Schlüssels in der Quelle zu vermeiden, verwenden Sie den [.NET Secret Manager,](/aspnet/core/security/app-secrets) um diese Werte zu speichern.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-104">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="e8bcf-105">Der Geheime Manager dient nur zu Entwicklungszwecken, Produktions-Apps sollten einen vertrauenswürdigen geheimen Manager zum Speichern geheimer Schlüssel verwenden.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-105">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

1. <span data-ttu-id="e8bcf-106">Öffnen Sie **"./appsettings.json",** und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-106">Open **./appsettings.json** and replace its contents with the following.</span></span>

    :::code language="json" source="../demo/GraphTutorial/appsettings.example.json" highlight="2-8":::

1. <span data-ttu-id="e8bcf-107">Öffnen Sie Ihre CLI in dem Verzeichnis, in dem sich **GraphTutorial.csproj** befindet, und führen Sie die folgenden Befehle aus, und ersetzen Sie dabei `YOUR_APP_ID` Ihre Anwendungs-ID aus dem Azure-Portal und `YOUR_APP_SECRET` ihren geheimen Anwendungsschlüssel.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-107">Open your CLI in the directory where **GraphTutorial.csproj** is located, and run the following commands, substituting `YOUR_APP_ID` with your application ID from the Azure portal, and `YOUR_APP_SECRET` with your application secret.</span></span>

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="e8bcf-108">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="e8bcf-108">Implement sign-in</span></span>

<span data-ttu-id="e8bcf-109">Implementieren Sie zunächst single sign-on im JavaScript-Code der App.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-109">First, implement single sign-on in the app's JavaScript code.</span></span> <span data-ttu-id="e8bcf-110">Sie verwenden das [Microsoft Teams JavaScript SDK,](/javascript/api/overview/msteams-client) um ein Zugriffstoken abzurufen, das es dem im Teams-Client ausgeführten JavaScript-Code ermöglicht, AJAX-Aufrufe an die Web-API durchzuführen, die Sie später implementieren werden.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-110">You will use the [Microsoft Teams JavaScript SDK](/javascript/api/overview/msteams-client) to get an access token which allows the JavaScript code running in the Teams client to make AJAX calls to Web API you will implement later.</span></span>

1. <span data-ttu-id="e8bcf-111">Öffnen Sie **./Pages/Index.cshtml,** und fügen Sie den folgenden Code innerhalb des `<script>` Tags hinzu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-111">Open **./Pages/Index.cshtml** and add the following code inside the `<script>` tag.</span></span>

    ```javascript
    (function () {
      if (microsoftTeams) {
        microsoftTeams.initialize();

        microsoftTeams.authentication.getAuthToken({
          successCallback: (token) => {
            // TEMPORARY: Display the access token for debugging
            $('#tab-container').empty();

            $('<code/>', {
              text: token,
              style: 'word-break: break-all;'
            }).appendTo('#tab-container');
          },
          failureCallback: (error) => {
            renderError(error);
          }
        });
      }
    })();

    function renderError(error) {
      $('#tab-container').empty();

      $('<h1/>', {
        text: 'Error'
      }).appendTo('#tab-container');

      $('<code/>', {
        text: JSON.stringify(error, Object.getOwnPropertyNames(error)),
        style: 'word-break: break-all;'
      }).appendTo('#tab-container');
    }
    ```

    <span data-ttu-id="e8bcf-112">Dadurch wird die automatische `microsoftTeams.authentication.getAuthToken` Authentifizierung als der Benutzer aufgerufen, der bei Teams angemeldet ist.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-112">This calls the `microsoftTeams.authentication.getAuthToken` to silently authenticate as the user that is signed in to Teams.</span></span> <span data-ttu-id="e8bcf-113">In der Regel sind keine Benutzeroberflächenaufforderungen beteiligt, es sei denn, der Benutzer muss zustimmen.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-113">There is typically not any UI prompts involved, unless the user has to consent.</span></span> <span data-ttu-id="e8bcf-114">Anschließend zeigt der Code das Token auf der Registerkarte an.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-114">The code then displays the token in the tab.</span></span>

1. <span data-ttu-id="e8bcf-115">Speichern Sie Ihre Änderungen, und starten Sie die Anwendung, indem Sie den folgenden Befehl in Der CLI ausführen.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-115">Save your changes and start your application by running the following command in your CLI.</span></span>

    ```Shell
    dotnet run
    ```

    > [!IMPORTANT]
    > <span data-ttu-id="e8bcf-116">Wenn Sie ngrok neu gestartet haben und ihre ngrok-URL geändert wurde, müssen Sie den ngrok-Wert **vor dem** Testen an der folgenden Stelle aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-116">If you have restarted ngrok and your ngrok URL has changed, be sure to update the ngrok value in the following place **before** you test.</span></span>
    >
    > - <span data-ttu-id="e8bcf-117">Der Umleitungs-URI in Ihrer App-Registrierung</span><span class="sxs-lookup"><span data-stu-id="e8bcf-117">The redirect URI in your app registration</span></span>
    > - <span data-ttu-id="e8bcf-118">Der Anwendungs-ID-URI in Ihrer App-Registrierung</span><span class="sxs-lookup"><span data-stu-id="e8bcf-118">The application ID URI in your app registration</span></span>
    > - <span data-ttu-id="e8bcf-119">`contentUrl` in manifest.json</span><span class="sxs-lookup"><span data-stu-id="e8bcf-119">`contentUrl` in manifest.json</span></span>
    > - <span data-ttu-id="e8bcf-120">`validDomains` in manifest.json</span><span class="sxs-lookup"><span data-stu-id="e8bcf-120">`validDomains` in manifest.json</span></span>
    > - <span data-ttu-id="e8bcf-121">`resource` in manifest.json</span><span class="sxs-lookup"><span data-stu-id="e8bcf-121">`resource` in manifest.json</span></span>

1. <span data-ttu-id="e8bcf-122">Erstellen Sie eine ZIP-Datei mit **manifest.json**, **color.png** und **outline.png**.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-122">Create a ZIP file with **manifest.json**, **color.png**, and **outline.png**.</span></span>

1. <span data-ttu-id="e8bcf-123">Wählen Sie in Microsoft Teams in der linken Leiste **Apps** aus, wählen Sie **Hochladen einer benutzerdefinierten App** aus, und wählen Sie dann Hochladen für mich oder meine **Teams** aus.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-123">In Microsoft Teams, select **Apps** in the left-hand bar, select **Upload a custom app**, then select **Upload for me or my teams**.</span></span>

    ![Screenshot der Hochladen eines benutzerdefinierten App-Links in Microsoft Teams](images/upload-custom-app.png)

1. <span data-ttu-id="e8bcf-125">Navigieren Sie zu der ZIP-Datei, die Sie zuvor erstellt haben, und wählen Sie **"Öffnen"** aus.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-125">Browse to the ZIP file you created previously and select **Open**.</span></span>

1. <span data-ttu-id="e8bcf-126">Überprüfen Sie die Anwendungsinformationen, und wählen Sie **"Hinzufügen"** aus.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-126">Review the application information and select **Add**.</span></span>

1. <span data-ttu-id="e8bcf-127">Die Anwendung wird in Teams geöffnet und zeigt ein Zugriffstoken an.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-127">The application opens in Teams and displays an access token.</span></span>

<span data-ttu-id="e8bcf-128">Wenn Sie das Token kopieren, können Sie es in [jwt.ms](https://jwt.ms)einfügen.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-128">If you copy the token, you can paste it into [jwt.ms](https://jwt.ms).</span></span> <span data-ttu-id="e8bcf-129">Stellen Sie sicher, dass die Zielgruppe (der `aud` Anspruch) Ihre Anwendungs-ID ist und der einzige Bereich (der `scp` Anspruch) der `access_as_user` von Ihnen erstellte API-Bereich ist.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-129">Verify that the audience (the `aud` claim) is your application ID, and the only scope (the `scp` claim) is the `access_as_user` API scope you created.</span></span> <span data-ttu-id="e8bcf-130">Das bedeutet, dass dieses Token keinen direkten Zugriff auf Microsoft Graph gewährt!</span><span class="sxs-lookup"><span data-stu-id="e8bcf-130">That means that this token does not grant direct access to Microsoft Graph!</span></span> <span data-ttu-id="e8bcf-131">Stattdessen muss die Web-API, die Sie bald implementieren werden, dieses Token mithilfe [des Im-Auftrag-von-Flusses](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) austauschen, um ein Token abzurufen, das mit Microsoft Graph Aufrufen funktioniert.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-131">Instead, the Web API you will implement soon will need to exchange this token using the [on-behalf-of flow](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) to get a token that will work with Microsoft Graph calls.</span></span>

## <a name="configure-authentication-in-the-aspnet-core-app"></a><span data-ttu-id="e8bcf-132">Konfigurieren der Authentifizierung in der ASP.NET Core-App</span><span class="sxs-lookup"><span data-stu-id="e8bcf-132">Configure authentication in the ASP.NET Core app</span></span>

<span data-ttu-id="e8bcf-133">Fügen Sie zunächst die Microsoft Identity Platform-Dienste zur Anwendung hinzu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-133">Start by adding the Microsoft Identity platform services to the application.</span></span>

1. <span data-ttu-id="e8bcf-134">Öffnen Sie die Datei **./Startup.cs,** und fügen Sie die folgende `using` Anweisung am Anfang der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-134">Open the **./Startup.cs** file and add the following `using` statement to the top of the file.</span></span>

    ```csharp
    using Microsoft.Identity.Web;
    ```

1. <span data-ttu-id="e8bcf-135">Fügen Sie die folgende Zeile direkt vor der `app.UseAuthorization();` Zeile in der Funktion `Configure` hinzu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-135">Add the following line just before the `app.UseAuthorization();` line in the `Configure` function.</span></span>

    ```csharp
    app.UseAuthentication();
    ```

1. <span data-ttu-id="e8bcf-136">Fügen Sie die folgende Zeile direkt hinter der `endpoints.MapRazorPages();` Zeile in der Funktion `Configure` hinzu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-136">Add the following line just after the `endpoints.MapRazorPages();` line in the `Configure` function.</span></span>

    ```csharp
    endpoints.MapControllers();
    ```

1. <span data-ttu-id="e8bcf-137">Ersetzen Sie die vorhandene `ConfigureServices`-Funktion durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-137">Replace the existing `ConfigureServices` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="ConfigureServicesSnippet":::

    <span data-ttu-id="e8bcf-138">Dieser Code konfiguriert die Anwendung so, dass Aufrufe von Web-APIs basierend auf dem JWT-Bearertoken im Header authentifiziert werden `Authorization` können.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-138">This code configures the application to allow calls to Web APIs to be authenticated based on the JWT bearer token in the `Authorization` header.</span></span> <span data-ttu-id="e8bcf-139">Außerdem werden die Tokenerfassungsdienste hinzugefügt, die dieses Token über den Im-Auftrag-von-Fluss austauschen können.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-139">It also adds the token acquisition services that can exchange that token via the on-behalf-of flow.</span></span>

## <a name="create-the-web-api-controller"></a><span data-ttu-id="e8bcf-140">Erstellen des Web-API-Controllers</span><span class="sxs-lookup"><span data-stu-id="e8bcf-140">Create the Web API controller</span></span>

1. <span data-ttu-id="e8bcf-141">Erstellen Sie ein neues Verzeichnis im Stammverzeichnis des Projekts namens **"Controller".**</span><span class="sxs-lookup"><span data-stu-id="e8bcf-141">Create a new directory in the root of the project named **Controllers**.</span></span>

1. <span data-ttu-id="e8bcf-142">Erstellen Sie eine neue Datei im **Verzeichnis "./Controllers"** mit dem Namen **"CalendarController.cs",** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-142">Create a new file in the **./Controllers** directory named **CalendarController.cs** and add the following code.</span></span>

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Net;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Http;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using Microsoft.Identity.Web.Resource;
    using Microsoft.Graph;
    using TimeZoneConverter;

    namespace GraphTutorial.Controllers
    {
        [ApiController]
        [Route("[controller]")]
        [Authorize]
        public class CalendarController : ControllerBase
        {
            private static readonly string[] apiScopes = new[] { "access_as_user" };

            private readonly GraphServiceClient _graphClient;
            private readonly ITokenAcquisition _tokenAcquisition;
            private readonly ILogger<CalendarController> _logger;

            public CalendarController(ITokenAcquisition tokenAcquisition, GraphServiceClient graphClient, ILogger<CalendarController> logger)
            {
                _tokenAcquisition = tokenAcquisition;
                _graphClient = graphClient;
                _logger = logger;
            }

            [HttpGet]
            public async Task<ActionResult<string>> Get()
            {
                // This verifies that the access_as_user scope is
                // present in the bearer token, throws if not
                HttpContext.VerifyUserHasAnyAcceptedScope(apiScopes);

                // To verify that the identity libraries have authenticated
                // based on the token, log the user's name
                _logger.LogInformation($"Authenticated user: {User.GetDisplayName()}");

                try
                {
                    // TEMPORARY
                    // Get a Graph token via OBO flow
                    var token = await _tokenAcquisition
                        .GetAccessTokenForUserAsync(new[]{
                            "User.Read",
                            "MailboxSettings.Read",
                            "Calendars.ReadWrite" });

                    // Log the token
                    _logger.LogInformation($"Access token for Graph: {token}");
                    return Ok("{ \"status\": \"OK\" }");
                }
                catch (MicrosoftIdentityWebChallengeUserException ex)
                {
                    _logger.LogError(ex, "Consent required");
                    // This exception indicates consent is required.
                    // Return a 403 with "consent_required" in the body
                    // to signal to the tab it needs to prompt for consent
                    return new ContentResult {
                        StatusCode = (int)HttpStatusCode.Forbidden,
                        ContentType = "text/plain",
                        Content = "consent_required"
                    };
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Error occurred");
                    throw;
                }
            }
        }
    }
    ```

    <span data-ttu-id="e8bcf-143">Dadurch wird eine Web-API ( `GET /calendar` ) implementiert, die über die Registerkarte Teams aufgerufen werden kann. For now it simply tries to exchange the bearer token for a Graph token.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-143">This implements a Web API (`GET /calendar`) that can be called from the Teams tab. For now it simply tries to exchange the bearer token for a Graph token.</span></span> <span data-ttu-id="e8bcf-144">Wenn ein Benutzer die Registerkarte zum ersten Mal lädt, tritt ein Fehler auf, da er der App noch nicht zugestimmt hat, den Zugriff auf Microsoft Graph in ihrem Auftrag zuzulassen.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-144">The first time a user loads the tab, this will fail because they have not yet consented to allow the app access to Microsoft Graph on their behalf.</span></span>

1. <span data-ttu-id="e8bcf-145">Öffnen Sie **./Pages/Index.cshtml,** und ersetzen Sie die `successCallback` Funktion durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-145">Open **./Pages/Index.cshtml** and replace the `successCallback` function with the following.</span></span>

    ```javascript
    successCallback: (token) => {
      // TEMPORARY: Call the Web API
      fetch('/calendar', {
        headers: {
          'Authorization': `Bearer ${token}`
        }
      }).then(response => {
        response.text()
          .then(body => {
            $('#tab-container').empty();
            $('<code/>', {
              text: body
            }).appendTo('#tab-container');
          });
      }).catch(error => {
        console.error(error);
        renderError(error);
      });
    }
    ```

    <span data-ttu-id="e8bcf-146">Dadurch wird die Web-API aufgerufen und die Antwort angezeigt.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-146">This will call the Web API and display the response.</span></span>

1. <span data-ttu-id="e8bcf-147">Speichern Sie die Änderungen, und starten Sie die App neu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-147">Save your changes and restart the app.</span></span> <span data-ttu-id="e8bcf-148">Aktualisieren Sie die Registerkarte in Microsoft Teams.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-148">Refresh the tab in Microsoft Teams.</span></span> <span data-ttu-id="e8bcf-149">Die Seite sollte angezeigt `consent_required` werden.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-149">The page should display `consent_required`.</span></span>

1. <span data-ttu-id="e8bcf-150">Überprüfen Sie die Protokollausgabe in Ihrer CLI.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-150">Review the log output in your CLI.</span></span> <span data-ttu-id="e8bcf-151">Beachten Sie zwei Dinge.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-151">Notice two things.</span></span>

    - <span data-ttu-id="e8bcf-152">Ein Eintrag wie `Authenticated user: MeganB@contoso.com` .</span><span class="sxs-lookup"><span data-stu-id="e8bcf-152">An entry like `Authenticated user: MeganB@contoso.com`.</span></span> <span data-ttu-id="e8bcf-153">Die Web-API hat den Benutzer basierend auf dem Token authentifiziert, das mit der API-Anforderung gesendet wurde.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-153">The Web API has authenticated the user based on the token sent with the API request.</span></span>
    - <span data-ttu-id="e8bcf-154">Ein Eintrag wie `AADSTS65001: The user or administrator has not consented to use the application with ID...` .</span><span class="sxs-lookup"><span data-stu-id="e8bcf-154">An entry like `AADSTS65001: The user or administrator has not consented to use the application with ID...`.</span></span> <span data-ttu-id="e8bcf-155">Dies wird erwartet, da der Benutzer noch nicht aufgefordert wurde, den angeforderten Microsoft Graph Berechtigungsbereichen zuzustimmen.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-155">This is expected, since the user has not yet been prompted to consent for the requested Microsoft Graph permission scopes.</span></span>

## <a name="implement-consent-prompt"></a><span data-ttu-id="e8bcf-156">Implementieren der Zustimmungsaufforderung</span><span class="sxs-lookup"><span data-stu-id="e8bcf-156">Implement consent prompt</span></span>

<span data-ttu-id="e8bcf-157">Da die Web-API den Benutzer nicht auffordern kann, muss die Registerkarte Teams eine Eingabeaufforderung implementieren.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-157">Because the Web API cannot prompt the user, the Teams tab will need to implement a prompt.</span></span> <span data-ttu-id="e8bcf-158">Dies muss nur einmal für jeden Benutzer erfolgen.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-158">This will only need to be done once for each user.</span></span> <span data-ttu-id="e8bcf-159">Sobald ein Benutzer seine Zustimmung erteilt hat, muss er den Zugriff auf Ihre Anwendung nicht erneut erklären, es sei denn, er widerruft den Zugriff auf Ihre Anwendung explizit.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-159">Once a user consents, they do not need to reconsent unless they explicitly revoke access to your application.</span></span>

1. <span data-ttu-id="e8bcf-160">Erstellen Sie eine neue Datei im **Verzeichnis ./Pages** mit dem Namen **Authenticate.cshtml.cs,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-160">Create a new file in the **./Pages** directory named **Authenticate.cshtml.cs** and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Pages/Authenticate.cshtml.cs" id="AuthenticateModelSnippet":::

1. <span data-ttu-id="e8bcf-161">Erstellen Sie eine neue Datei im **Verzeichnis ./Pages** mit dem Namen **Authenticate.cshtml,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-161">Create a new file in the **./Pages** directory named **Authenticate.cshtml** and add the following code.</span></span>

    :::code language="razor" source="../demo/GraphTutorial/Pages/Authenticate.cshtml":::

1. <span data-ttu-id="e8bcf-162">Erstellen Sie eine neue Datei im **Verzeichnis ./Pages** mit dem Namen **"AuthComplete.cshtml",** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-162">Create a new file in the **./Pages** directory named **AuthComplete.cshtml** and add the following code.</span></span>

    :::code language="razor" source="../demo/GraphTutorial/Pages/AuthComplete.cshtml":::

1. <span data-ttu-id="e8bcf-163">Öffnen Sie **./Pages/Index.cshtml,** und fügen Sie die folgenden Funktionen innerhalb des `<script>` Tags hinzu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-163">Open **./Pages/Index.cshtml** and add the following functions inside the `<script>` tag.</span></span>

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="LoadUserCalendarSnippet":::

1. <span data-ttu-id="e8bcf-164">Fügen Sie die folgende Funktion innerhalb des `<script>` Tags hinzu, um ein erfolgreiches Ergebnis aus der Web-API anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-164">Add the following function inside the `<script>` tag to display a successful result from the Web API.</span></span>

    ```javascript
    function renderCalendar(events) {
      $('#tab-container').empty();

      $('<pre/>').append($('<code/>', {
        text: JSON.stringify(events, null, 2),
        style: 'word-break: break-all;'
      })).appendTo('#tab-container');
    }
    ```

1. <span data-ttu-id="e8bcf-165">Ersetzen Sie den vorhandenen `successCallback` durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-165">Replace the existing `successCallback` with the following code.</span></span>

    ```javascript
    successCallback: (token) => {
      loadUserCalendar(token, (events) => {
        renderCalendar(events);
      });
    }
    ```

1. <span data-ttu-id="e8bcf-166">Speichern Sie die Änderungen, und starten Sie die App neu.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-166">Save your changes and restart the app.</span></span> <span data-ttu-id="e8bcf-167">Aktualisieren Sie die Registerkarte in Microsoft Teams.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-167">Refresh the tab in Microsoft Teams.</span></span> <span data-ttu-id="e8bcf-168">Sie sollten ein Popupfenster erhalten, in dem Sie um Zustimmung zu den Microsoft Graph Berechtigungsbereichen gebeten werden.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-168">You should get a pop-up window asking for consent to the Microsoft Graph permissions scopes.</span></span> <span data-ttu-id="e8bcf-169">Nach der Annahme sollte die Registerkarte angezeigt `{ "status": "OK" }` werden.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-169">After accepting, the tab should display `{ "status": "OK" }`.</span></span>

    > [!NOTE]
    > <span data-ttu-id="e8bcf-170">Wenn die Registerkarte angezeigt `"FailedToOpenWindow"` wird, deaktivieren Sie Popupblocker in Ihrem Browser, und laden Sie die Seite erneut.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-170">If the tab displays `"FailedToOpenWindow"`, please disable pop-up blockers in your browser and reload the page.</span></span>

1. <span data-ttu-id="e8bcf-171">Überprüfen Sie die Protokollausgabe.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-171">Review the log output.</span></span> <span data-ttu-id="e8bcf-172">Der Eintrag sollte angezeigt `Access token for Graph` werden.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-172">You should see the `Access token for Graph` entry.</span></span> <span data-ttu-id="e8bcf-173">Wenn Sie dieses Token analysieren, werden Sie feststellen, dass es die Microsoft Graph Bereiche enthält, die in **appsettings.json** konfiguriert sind.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-173">If you parse that token, you'll notice that it contains the Microsoft Graph scopes configured in **appsettings.json**.</span></span>

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="e8bcf-174">Speichern und Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="e8bcf-174">Storing and refreshing tokens</span></span>

<span data-ttu-id="e8bcf-175">An diesem Punkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-Aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-175">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="e8bcf-176">Dies ist das Token, durch das die App im Namen des Benutzers auf Microsoft Graph zugreifen kann.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-176">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="e8bcf-177">Dieses Token ist jedoch nur kurzzeitig verfügbar.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-177">However, this token is short-lived.</span></span> <span data-ttu-id="e8bcf-178">Das Token läuft eine Stunde nach der Ausstellung ab.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-178">The token expires an hour after it is issued.</span></span> <span data-ttu-id="e8bcf-179">An dieser Stelle kommt das Aktualisierungstoken ins Spiel.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-179">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="e8bcf-180">Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-180">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="e8bcf-181">Da die App die Microsoft.Identity.Web-Bibliothek verwendet, müssen Sie keine Tokenspeicher- oder Aktualisierungslogik implementieren.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-181">Because the app is using the Microsoft.Identity.Web library, you do not have to implement any token storage or refresh logic.</span></span>

<span data-ttu-id="e8bcf-182">Die App verwendet den Speichertokencache, der für Apps ausreicht, die beim Neustart der App keine Token beibehalten müssen.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-182">The app uses the in-memory token cache, which is sufficient for apps that do not need to persist tokens when the app restarts.</span></span> <span data-ttu-id="e8bcf-183">Produktions-Apps verwenden stattdessen möglicherweise die Optionen für [verteilten Cache](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in der Microsoft.Identity.Web-Bibliothek.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-183">Production apps may instead use the [distributed cache options](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in the Microsoft.Identity.Web library.</span></span>

<span data-ttu-id="e8bcf-184">Die `GetAccessTokenForUserAsync` Methode behandelt den Ablauf und die Aktualisierung des Tokens für Sie.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-184">The `GetAccessTokenForUserAsync` method handles token expiration and refresh for you.</span></span> <span data-ttu-id="e8bcf-185">Es überprüft zuerst das zwischengespeicherte Token und gibt es zurück, wenn es nicht abgelaufen ist.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-185">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="e8bcf-186">Wenn es abgelaufen ist, wird das zwischengespeicherte Aktualisierungstoken verwendet, um ein neues abzurufen.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-186">If it is expired, it uses the cached refresh token to obtain a new one.</span></span>

<span data-ttu-id="e8bcf-187">Der **GraphServiceClient,** den Controller über die Abhängigkeitsinjektion erhalten, ist mit einem Authentifizierungsanbieter vorkonfiguriert, der für Sie verwendet `GetAccessTokenForUserAsync` wird.</span><span class="sxs-lookup"><span data-stu-id="e8bcf-187">The **GraphServiceClient** that controllers get via dependency injection is pre-configured with an authentication provider that uses `GetAccessTokenForUserAsync` for you.</span></span>
