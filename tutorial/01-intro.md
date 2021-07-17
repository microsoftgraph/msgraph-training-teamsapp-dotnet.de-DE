<!-- markdownlint-disable MD002 MD041 -->

In diesem Lernprogramm erfahren Sie, wie Sie eine Microsoft Teams-App mit ASP.NET Core und der Microsoft Graph-API erstellen, um Kalenderinformationen für einen Benutzer abzurufen.

> [!TIP]
> Wenn Sie es vorziehen, nur das abgeschlossene Lernprogramm herunterzuladen, können Sie das GitHub Repository herunterladen oder [klonen.](https://github.com/microsoftgraph/msgraph-training-teamsapp-dotnet) Anweisungen zum Konfigurieren der App mit einer App-ID und einem geheimen Schlüssel finden Sie in der README-Datei im **Demoordner.**

## <a name="prerequisites"></a>Voraussetzungen

Bevor Sie mit diesem Lernprogramm beginnen, sollten Sie Folgendes auf Ihrem Entwicklungscomputer installiert haben.

- [.NET SDK](https://dotnet.microsoft.com/download).
- [ngrok](https://ngrok.com/)

Sie sollten auch ein Microsoft-Geschäfts-, Schul- oder Unikonto in einem Microsoft 365 Mandanten haben, der [benutzerdefiniertes Teams Querladen von Apps aktiviert](/microsoftteams/platform/concepts/build-and-test/prepare-your-o365-tenant#enable-custom-teams-apps-and-turn-on-custom-app-uploading)hat. Wenn Sie nicht über ein Microsoft-Geschäfts-, Schul- oder Unikonto verfügen oder Ihre Organisation das Querladen von benutzerdefinierten Teams Apps nicht aktiviert hat, können Sie [sich für das Microsoft 365 Entwicklerprogramm registrieren,](https://developer.microsoft.com/office/dev-program) um ein kostenloses Office 365 Entwicklerabonnement zu erhalten.

> [!NOTE]
> Dieses Lernprogramm wurde mit .NET SDK Version 5.0.302 geschrieben. Die Schritte in diesem Handbuch funktionieren möglicherweise mit anderen Versionen, die jedoch nicht getestet wurden.

## <a name="feedback"></a>Feedback

Bitte geben Sie Feedback zu diesem Lernprogramm im [GitHub Repository.](https://github.com/microsoftgraph/msgraph-training-teamsapp-dotnet)
