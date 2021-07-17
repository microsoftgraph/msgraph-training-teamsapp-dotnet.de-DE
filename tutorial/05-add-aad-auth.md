<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Single Sign-On-Authentifizierung mit Azure AD zu unterstützen. Dies ist notwendig, um das erforderliche OAuth-Zugriffstoken zum Aufruf der Microsoft Graph-API abzurufen. In diesem Schritt konfigurieren Sie die [Microsoft.Identity.Web-Bibliothek.](https://www.nuget.org/packages/Microsoft.Identity.Web/)

> [!IMPORTANT]
> Um das Speichern der Anwendungs-ID und des geheimen Schlüssels in der Quelle zu vermeiden, verwenden Sie den [.NET Secret Manager,](/aspnet/core/security/app-secrets) um diese Werte zu speichern. Der Geheime Manager dient nur zu Entwicklungszwecken, Produktions-Apps sollten einen vertrauenswürdigen geheimen Manager zum Speichern geheimer Schlüssel verwenden.

1. Öffnen Sie **"./appsettings.json",** und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="json" source="../demo/GraphTutorial/appsettings.example.json" highlight="2-8":::

1. Öffnen Sie Ihre CLI in dem Verzeichnis, in dem sich **GraphTutorial.csproj** befindet, und führen Sie die folgenden Befehle aus, und ersetzen Sie dabei `YOUR_APP_ID` Ihre Anwendungs-ID aus dem Azure-Portal und `YOUR_APP_SECRET` ihren geheimen Anwendungsschlüssel.

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

Implementieren Sie zunächst single sign-on im JavaScript-Code der App. Sie verwenden das [Microsoft Teams JavaScript SDK,](/javascript/api/overview/msteams-client) um ein Zugriffstoken abzurufen, das es dem im Teams-Client ausgeführten JavaScript-Code ermöglicht, AJAX-Aufrufe an die Web-API durchzuführen, die Sie später implementieren werden.

1. Öffnen Sie **./Pages/Index.cshtml,** und fügen Sie den folgenden Code innerhalb des `<script>` Tags hinzu.

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

    Dadurch wird die automatische `microsoftTeams.authentication.getAuthToken` Authentifizierung als der Benutzer aufgerufen, der bei Teams angemeldet ist. In der Regel sind keine Benutzeroberflächenaufforderungen beteiligt, es sei denn, der Benutzer muss zustimmen. Anschließend zeigt der Code das Token auf der Registerkarte an.

1. Speichern Sie Ihre Änderungen, und starten Sie die Anwendung, indem Sie den folgenden Befehl in Der CLI ausführen.

    ```Shell
    dotnet run
    ```

    > [!IMPORTANT]
    > Wenn Sie ngrok neu gestartet haben und ihre ngrok-URL geändert wurde, müssen Sie den ngrok-Wert **vor dem** Testen an der folgenden Stelle aktualisieren.
    >
    > - Der Umleitungs-URI in Ihrer App-Registrierung
    > - Der Anwendungs-ID-URI in Ihrer App-Registrierung
    > - `contentUrl` in manifest.json
    > - `validDomains` in manifest.json
    > - `resource` in manifest.json

1. Erstellen Sie eine ZIP-Datei mit **manifest.json**, **color.png** und **outline.png**.

1. Wählen Sie in Microsoft Teams in der linken Leiste **Apps** aus, wählen Sie **Hochladen einer benutzerdefinierten App** aus, und wählen Sie dann Hochladen für mich oder meine **Teams** aus.

    ![Screenshot der Hochladen eines benutzerdefinierten App-Links in Microsoft Teams](images/upload-custom-app.png)

1. Navigieren Sie zu der ZIP-Datei, die Sie zuvor erstellt haben, und wählen Sie **"Öffnen"** aus.

1. Überprüfen Sie die Anwendungsinformationen, und wählen Sie **"Hinzufügen"** aus.

1. Die Anwendung wird in Teams geöffnet und zeigt ein Zugriffstoken an.

Wenn Sie das Token kopieren, können Sie es in [jwt.ms](https://jwt.ms)einfügen. Stellen Sie sicher, dass die Zielgruppe (der `aud` Anspruch) Ihre Anwendungs-ID ist und der einzige Bereich (der `scp` Anspruch) der `access_as_user` von Ihnen erstellte API-Bereich ist. Das bedeutet, dass dieses Token keinen direkten Zugriff auf Microsoft Graph gewährt! Stattdessen muss die Web-API, die Sie bald implementieren werden, dieses Token mithilfe [des Im-Auftrag-von-Flusses](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) austauschen, um ein Token abzurufen, das mit Microsoft Graph Aufrufen funktioniert.

## <a name="configure-authentication-in-the-aspnet-core-app"></a>Konfigurieren der Authentifizierung in der ASP.NET Core-App

Fügen Sie zunächst die Microsoft Identity Platform-Dienste zur Anwendung hinzu.

1. Öffnen Sie die Datei **./Startup.cs,** und fügen Sie die folgende `using` Anweisung am Anfang der Datei hinzu.

    ```csharp
    using Microsoft.Identity.Web;
    ```

1. Fügen Sie die folgende Zeile direkt vor der `app.UseAuthorization();` Zeile in der Funktion `Configure` hinzu.

    ```csharp
    app.UseAuthentication();
    ```

1. Fügen Sie die folgende Zeile direkt hinter der `endpoints.MapRazorPages();` Zeile in der Funktion `Configure` hinzu.

    ```csharp
    endpoints.MapControllers();
    ```

1. Ersetzen Sie die vorhandene `ConfigureServices`-Funktion durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="ConfigureServicesSnippet":::

    Dieser Code konfiguriert die Anwendung so, dass Aufrufe von Web-APIs basierend auf dem JWT-Bearertoken im Header authentifiziert werden `Authorization` können. Außerdem werden die Tokenerfassungsdienste hinzugefügt, die dieses Token über den Im-Auftrag-von-Fluss austauschen können.

## <a name="create-the-web-api-controller"></a>Erstellen des Web-API-Controllers

1. Erstellen Sie ein neues Verzeichnis im Stammverzeichnis des Projekts namens **"Controller".**

1. Erstellen Sie eine neue Datei im **Verzeichnis "./Controllers"** mit dem Namen **"CalendarController.cs",** und fügen Sie den folgenden Code hinzu.

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

    Dadurch wird eine Web-API ( `GET /calendar` ) implementiert, die über die Registerkarte Teams aufgerufen werden kann. For now it simply tries to exchange the bearer token for a Graph token. Wenn ein Benutzer die Registerkarte zum ersten Mal lädt, tritt ein Fehler auf, da er der App noch nicht zugestimmt hat, den Zugriff auf Microsoft Graph in ihrem Auftrag zuzulassen.

1. Öffnen Sie **./Pages/Index.cshtml,** und ersetzen Sie die `successCallback` Funktion durch Folgendes.

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

    Dadurch wird die Web-API aufgerufen und die Antwort angezeigt.

1. Speichern Sie die Änderungen, und starten Sie die App neu. Aktualisieren Sie die Registerkarte in Microsoft Teams. Die Seite sollte angezeigt `consent_required` werden.

1. Überprüfen Sie die Protokollausgabe in Ihrer CLI. Beachten Sie zwei Dinge.

    - Ein Eintrag wie `Authenticated user: MeganB@contoso.com` . Die Web-API hat den Benutzer basierend auf dem Token authentifiziert, das mit der API-Anforderung gesendet wurde.
    - Ein Eintrag wie `AADSTS65001: The user or administrator has not consented to use the application with ID...` . Dies wird erwartet, da der Benutzer noch nicht aufgefordert wurde, den angeforderten Microsoft Graph Berechtigungsbereichen zuzustimmen.

## <a name="implement-consent-prompt"></a>Implementieren der Zustimmungsaufforderung

Da die Web-API den Benutzer nicht auffordern kann, muss die Registerkarte Teams eine Eingabeaufforderung implementieren. Dies muss nur einmal für jeden Benutzer erfolgen. Sobald ein Benutzer seine Zustimmung erteilt hat, muss er den Zugriff auf Ihre Anwendung nicht erneut erklären, es sei denn, er widerruft den Zugriff auf Ihre Anwendung explizit.

1. Erstellen Sie eine neue Datei im **Verzeichnis ./Pages** mit dem Namen **Authenticate.cshtml.cs,** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Pages/Authenticate.cshtml.cs" id="AuthenticateModelSnippet":::

1. Erstellen Sie eine neue Datei im **Verzeichnis ./Pages** mit dem Namen **Authenticate.cshtml,** und fügen Sie den folgenden Code hinzu.

    :::code language="razor" source="../demo/GraphTutorial/Pages/Authenticate.cshtml":::

1. Erstellen Sie eine neue Datei im **Verzeichnis ./Pages** mit dem Namen **"AuthComplete.cshtml",** und fügen Sie den folgenden Code hinzu.

    :::code language="razor" source="../demo/GraphTutorial/Pages/AuthComplete.cshtml":::

1. Öffnen Sie **./Pages/Index.cshtml,** und fügen Sie die folgenden Funktionen innerhalb des `<script>` Tags hinzu.

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="LoadUserCalendarSnippet":::

1. Fügen Sie die folgende Funktion innerhalb des `<script>` Tags hinzu, um ein erfolgreiches Ergebnis aus der Web-API anzuzeigen.

    ```javascript
    function renderCalendar(events) {
      $('#tab-container').empty();

      $('<pre/>').append($('<code/>', {
        text: JSON.stringify(events, null, 2),
        style: 'word-break: break-all;'
      })).appendTo('#tab-container');
    }
    ```

1. Ersetzen Sie den vorhandenen `successCallback` durch den folgenden Code.

    ```javascript
    successCallback: (token) => {
      loadUserCalendar(token, (events) => {
        renderCalendar(events);
      });
    }
    ```

1. Speichern Sie die Änderungen, und starten Sie die App neu. Aktualisieren Sie die Registerkarte in Microsoft Teams. Sie sollten ein Popupfenster erhalten, in dem Sie um Zustimmung zu den Microsoft Graph Berechtigungsbereichen gebeten werden. Nach der Annahme sollte die Registerkarte angezeigt `{ "status": "OK" }` werden.

    > [!NOTE]
    > Wenn die Registerkarte angezeigt `"FailedToOpenWindow"` wird, deaktivieren Sie Popupblocker in Ihrem Browser, und laden Sie die Seite erneut.

1. Überprüfen Sie die Protokollausgabe. Der Eintrag sollte angezeigt `Access token for Graph` werden. Wenn Sie dieses Token analysieren, werden Sie feststellen, dass es die Microsoft Graph Bereiche enthält, die in **appsettings.json** konfiguriert sind.

## <a name="storing-and-refreshing-tokens"></a>Speichern und Aktualisieren von Token

An diesem Punkt verfügt Ihre Anwendung über ein Zugriffstoken, das in der `Authorization` Kopfzeile von API-Aufrufen gesendet wird. Dies ist das Token, durch das die App im Namen des Benutzers auf Microsoft Graph zugreifen kann.

Dieses Token ist jedoch nur kurzzeitig verfügbar. Das Token läuft eine Stunde nach der Ausstellung ab. An dieser Stelle kommt das Aktualisierungstoken ins Spiel. Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.

Da die App die Microsoft.Identity.Web-Bibliothek verwendet, müssen Sie keine Tokenspeicher- oder Aktualisierungslogik implementieren.

Die App verwendet den Speichertokencache, der für Apps ausreicht, die beim Neustart der App keine Token beibehalten müssen. Produktions-Apps verwenden stattdessen möglicherweise die Optionen für [verteilten Cache](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in der Microsoft.Identity.Web-Bibliothek.

Die `GetAccessTokenForUserAsync` Methode behandelt den Ablauf und die Aktualisierung des Tokens für Sie. Es überprüft zuerst das zwischengespeicherte Token und gibt es zurück, wenn es nicht abgelaufen ist. Wenn es abgelaufen ist, wird das zwischengespeicherte Aktualisierungstoken verwendet, um ein neues abzurufen.

Der **GraphServiceClient,** den Controller über die Abhängigkeitsinjektion erhalten, ist mit einem Authentifizierungsanbieter vorkonfiguriert, der für Sie verwendet `GetAccessTokenForUserAsync` wird.
