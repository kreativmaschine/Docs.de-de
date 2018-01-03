---
uid: identity/overview/features-api/best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure
title: "Bewährte Methoden für die Bereitstellung von Kennwörtern und andere sensible Daten für ASP.NET und Azure App Service | Microsoft Docs"
author: Rick-Anderson
description: "In diesem Lernprogramm wird gezeigt, wie der Code sicher zu speichern und Zugriff auf sichere Informationen. Der wichtigste Punkt ist, dass Sie niemals Kennwörter oder andere Sen speichern..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 05/21/2015
ms.topic: article
ms.assetid: 97902c66-cb61-4d11-be52-73f962f2db0a
ms.technology: 
ms.prod: .net-framework
msc.legacyurl: /identity/overview/features-api/best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure
msc.type: authoredcontent
ms.openlocfilehash: 465c9cf6f452c268e7e23509e7a29547df5d3e83
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/10/2017
---
<a name="best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure-app-service"></a><span data-ttu-id="44f14-104">Bewährte Methoden für die Bereitstellung von Kennwörtern und andere sensible Daten für ASP.NET und Azure App Service</span><span class="sxs-lookup"><span data-stu-id="44f14-104">Best practices for deploying passwords and other sensitive data to ASP.NET and Azure App Service</span></span>
====================
<span data-ttu-id="44f14-105">Durch [Rick Anderson](https://github.com/Rick-Anderson)</span><span class="sxs-lookup"><span data-stu-id="44f14-105">by [Rick Anderson](https://github.com/Rick-Anderson)</span></span>

> <span data-ttu-id="44f14-106">In diesem Lernprogramm wird gezeigt, wie der Code sicher zu speichern und Zugriff auf sichere Informationen.</span><span class="sxs-lookup"><span data-stu-id="44f14-106">This tutorial shows how your code can securely store and access secure information.</span></span> <span data-ttu-id="44f14-107">Der wichtigste Punkt ist, sollten Sie niemals Kennwörter oder andere vertraulichen Daten im Quellcode speichern und Produktion geheime Schlüssel verwenden, im Modus für Entwicklung und Tests keine.</span><span class="sxs-lookup"><span data-stu-id="44f14-107">The most important point is you should never store passwords or other sensitive data in source code, and you shouldn't use production secrets in development and test mode.</span></span>
> 
> <span data-ttu-id="44f14-108">Im Beispielcode wird eine einfache WebJob-Konsolen-app und eine ASP.NET MVC-app, die Zugriff auf eine Datenbank Verbindung Zeichenfolge Kennwort Twilio, Google und SendGrid sichere Schlüssel benötigt.</span><span class="sxs-lookup"><span data-stu-id="44f14-108">The sample code is a simple WebJob console app and a ASP.NET MVC app that needs access to a database connection string password, Twilio, Google and SendGrid secure keys.</span></span>
> 
> <span data-ttu-id="44f14-109">Lokale wird Einstellungen und PHP ebenfalls erwähnt.</span><span class="sxs-lookup"><span data-stu-id="44f14-109">On premise settings and PHP is also mentioned.</span></span>


- [<span data-ttu-id="44f14-110">Arbeiten mit Kennwörtern in der Entwicklungsumgebung</span><span class="sxs-lookup"><span data-stu-id="44f14-110">Working with passwords in the development environment</span></span>](#pwd)
- [<span data-ttu-id="44f14-111">Arbeiten mit Verbindungszeichenfolgen in der Entwicklungsumgebung</span><span class="sxs-lookup"><span data-stu-id="44f14-111">Working with connection strings in the development environment</span></span>](#con)
- [<span data-ttu-id="44f14-112">WebJobs-Konsolen-apps</span><span class="sxs-lookup"><span data-stu-id="44f14-112">WebJobs console apps</span></span>](#wj)
- [<span data-ttu-id="44f14-113">Geheime Schlüssel in Azure bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="44f14-113">Deploying secrets to Azure</span></span>](#da)
- [<span data-ttu-id="44f14-114">Hinweise für lokale und PHP</span><span class="sxs-lookup"><span data-stu-id="44f14-114">Notes for On-Premise and PHP</span></span>](#not)
- <span data-ttu-id="44f14-115">[Additional Resources](#addRes) (Zusätzliche MSBuild-Ressourcen)</span><span class="sxs-lookup"><span data-stu-id="44f14-115">[Additional Resources](#addRes)</span></span>

<a id="pwd"></a>
## <a name="working-with-passwords-in-the-development-environment"></a><span data-ttu-id="44f14-116">Arbeiten mit Kennwörtern in der Entwicklungsumgebung</span><span class="sxs-lookup"><span data-stu-id="44f14-116">Working with passwords in the development environment</span></span>

<span data-ttu-id="44f14-117">Tutorials erfahren häufig vertrauliche Daten im Quellcode hoffentlich mit einer Einschränkung, dass sensible Daten nie im Quellcode gespeichert werden sollten.</span><span class="sxs-lookup"><span data-stu-id="44f14-117">Tutorials frequently show sensitive data in source code, hopefully with a caveat that you should never store sensitive data in source code.</span></span> <span data-ttu-id="44f14-118">Z. B. Mein [ASP.NET MVC 5-Anwendung mit SMS und e-Mail-2FA](../../../mvc/overview/security/aspnet-mvc-5-app-with-sms-and-email-two-factor-authentication.md) Lernprogramm erfahren Sie Folgendes in die *"Web.config"* Datei:</span><span class="sxs-lookup"><span data-stu-id="44f14-118">For example, my [ASP.NET MVC 5 app with SMS and email 2FA](../../../mvc/overview/security/aspnet-mvc-5-app-with-sms-and-email-two-factor-authentication.md) tutorial shows the following in the *web.config* file:</span></span>

[!code-xml[Main](best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure/samples/sample1.xml)]

<span data-ttu-id="44f14-119">Die *"Web.config"* Datei Quellcode ist, damit dieser geheimen Schlüssel nie in dieser Datei gespeichert werden sollen.</span><span class="sxs-lookup"><span data-stu-id="44f14-119">The *web.config* file is source code, so these secrets should never be stored in that file.</span></span> <span data-ttu-id="44f14-120">Glücklicherweise der `<appSettings>` Element verfügt über eine `file` -Attribut, Ihnen die ermöglicht Angabe eine externe Datei, die vertrauliche app-Konfigurationseinstellungen enthält.</span><span class="sxs-lookup"><span data-stu-id="44f14-120">Fortunately, the `<appSettings>` element has a `file` attribute that allows you to specify an external file that contains sensitive app config settings.</span></span> <span data-ttu-id="44f14-121">Solange die externe Datei nicht in Ihrer ursprünglichen Struktur aktiviert ist, können Sie Ihre vertraulichen Daten in eine externe Datei verschieben.</span><span class="sxs-lookup"><span data-stu-id="44f14-121">You can move all your secrets to an external file as long as the external file is not checked into your source tree.</span></span> <span data-ttu-id="44f14-122">Im folgenden Markup, die Datei z. B. *AppSettingsSecrets.config* enthält alle geheime app-Schlüssel:</span><span class="sxs-lookup"><span data-stu-id="44f14-122">For example, in the following markup, the file *AppSettingsSecrets.config* contains all the app secrets:</span></span>

[!code-xml[Main](best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure/samples/sample2.xml)]

<span data-ttu-id="44f14-123">Das Markup in der externen Datei (*AppSettingsSecrets.config* in diesem Beispiel), wird das gleiche Markup wurde gefunden, der *"Web.config"* Datei:</span><span class="sxs-lookup"><span data-stu-id="44f14-123">The markup in the external file (*AppSettingsSecrets.config* in this sample), is the same markup found in the *web.config* file:</span></span>

[!code-xml[Main](best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure/samples/sample3.xml)]

<span data-ttu-id="44f14-124">Die ASP.NET-Laufzeit führt den Inhalt der externen Datei mit dem Markup im &lt;"appSettings"&gt; Element.</span><span class="sxs-lookup"><span data-stu-id="44f14-124">The ASP.NET runtime merges the contents of the external file with the markup in &lt;appSettings&gt; element.</span></span> <span data-ttu-id="44f14-125">Die Common Language Runtime ignoriert das Dateiattribut ", wenn die angegebene Datei nicht gefunden werden kann.</span><span class="sxs-lookup"><span data-stu-id="44f14-125">The runtime ignores the file attribute if the specified file cannot be found.</span></span>

> [!WARNING]
> <span data-ttu-id="44f14-126">Sicherheit – fügen Sie keine Ihrer *geheime Schlüssel config* -Datei in Ihrem Projekt, oder checken Sie es in die quellcodeverwaltung.</span><span class="sxs-lookup"><span data-stu-id="44f14-126">Security - Do not add your *secrets .config* file to your project or check it into source control.</span></span> <span data-ttu-id="44f14-127">Standardmäßig legt Visual Studio die `Build Action` zu `Content`, was bedeutet, dass die Datei bereitgestellt wird.</span><span class="sxs-lookup"><span data-stu-id="44f14-127">By default, Visual Studio sets the `Build Action` to `Content`, which means the file is deployed.</span></span> <span data-ttu-id="44f14-128">Weitere Informationen finden Sie unter [Warum nicht alle Dateien im Projekt-Ordner bereitgestellt?](https://msdn.microsoft.com/en-us/library/ee942158(v=vs.110).aspx#can_i_exclude_specific_files_or_folders_from_deployment)</span><span class="sxs-lookup"><span data-stu-id="44f14-128">For more information see [Why don't all of the files in my project folder get deployed?](https://msdn.microsoft.com/en-us/library/ee942158(v=vs.110).aspx#can_i_exclude_specific_files_or_folders_from_deployment)</span></span> <span data-ttu-id="44f14-129">Obwohl Sie können eine beliebige Erweiterung für die *geheime Schlüssel config* Datei, es wird empfohlen, ihn zu *config*, wie die Konfigurationsdateien von IIS nicht verarbeitet werden.</span><span class="sxs-lookup"><span data-stu-id="44f14-129">Although you can use any extension for the *secrets .config* file, it's best to keep it *.config*, as config files are not served by IIS.</span></span> <span data-ttu-id="44f14-130">Beachten Sie auch, dass die *AppSettingsSecrets.config* Datei ist zwei Directory Ebenen oberhalb von der *"Web.config"* Datei, damit es vollständig außerhalb des dem Projektmappenverzeichnis liegt.</span><span class="sxs-lookup"><span data-stu-id="44f14-130">Notice also that the *AppSettingsSecrets.config* file is two directory levels up from the *web.config* file, so it's completely out of the solution directory.</span></span> <span data-ttu-id="44f14-131">Durch Verschieben der Datei aus dem Projektmappenverzeichnis &quot;Git hinzufügen \* &quot; wird nicht an Ihr Repository hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="44f14-131">By moving the file out of the solution directory, &quot;git add \*&quot; won't add it to your repository.</span></span>


<a id="con"></a>
## <a name="working-with-connection-strings-in-the-development-environment"></a><span data-ttu-id="44f14-132">Arbeiten mit Verbindungszeichenfolgen in der Entwicklungsumgebung</span><span class="sxs-lookup"><span data-stu-id="44f14-132">Working with connection strings in the development environment</span></span>

<span data-ttu-id="44f14-133">Visual Studio erstellt neue ASP.NET-Projekte, mit denen [LocalDB](https://blogs.msdn.com/b/sqlexpress/archive/2011/07/12/introducing-localdb-a-better-sql-express.aspx).</span><span class="sxs-lookup"><span data-stu-id="44f14-133">Visual Studio creates new ASP.NET projects that use [LocalDB](https://blogs.msdn.com/b/sqlexpress/archive/2011/07/12/introducing-localdb-a-better-sql-express.aspx).</span></span> <span data-ttu-id="44f14-134">LocalDB, die speziell für die Entwicklungsumgebung erstellt wurde.</span><span class="sxs-lookup"><span data-stu-id="44f14-134">LocalDB was created specifically for the development environment.</span></span> <span data-ttu-id="44f14-135">Es sind ein Kennwort erforderlich, daher brauchen Sie gar nichts Unternehmen, um zu verhindern, dass Kennwörter in Ihren Quellcode überprüft wird.</span><span class="sxs-lookup"><span data-stu-id="44f14-135">It doesn't require a password, therefore you don't need to do anything to prevent secrets from being checked into your source code.</span></span> <span data-ttu-id="44f14-136">Einige Entwicklungsteams verwenden Sie die Vollversion von SQL Server (oder andere DBMS), die ein Kennwort erfordern.</span><span class="sxs-lookup"><span data-stu-id="44f14-136">Some development teams use the full versions of SQL Server (or other DBMS) that require a password.</span></span>

<span data-ttu-id="44f14-137">Sie können die `configSource` Attribut zum Ersetzen des gesamtes `<connectionStrings>` Markup.</span><span class="sxs-lookup"><span data-stu-id="44f14-137">You can use the `configSource` attribute to replace the entire `<connectionStrings>` markup.</span></span> <span data-ttu-id="44f14-138">Im Gegensatz zu den `<appSettings>` `file` -Attribut, das das Markup, führt der `configSource` -Attribut ersetzt das Markup.</span><span class="sxs-lookup"><span data-stu-id="44f14-138">Unlike the `<appSettings>` `file` attribute that merges the markup, the `configSource` attribute replaces the markup.</span></span> <span data-ttu-id="44f14-139">Das folgende Markup zeigt die `configSource` Attribut in der *"Web.config"* Datei:</span><span class="sxs-lookup"><span data-stu-id="44f14-139">The following markup shows the `configSource` attribute in the *web.config* file:</span></span>

[!code-xml[Main](best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure/samples/sample4.xml?highlight=1)]

> [!NOTE]
> <span data-ttu-id="44f14-140">Bei Verwendung der `configSource` -Attribut an, wie oben gezeigt, um die Verbindungszeichenfolgen in eine externe Datei zu verschieben und Visual Studio eine neue Website erstellen, er kann nicht erkennen Sie eine Datenbank, und rufen Sie nicht die Möglichkeit, konfigurieren die Datenbank bei der Sie Pu Veröffentlichen in Azure aus Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="44f14-140">If you use the `configSource` attribute as shown above to move your connection strings to an external file, and have Visual Studio create a new web site, it won't be able to detect you are using a database, and you won't get the option of configuring the database when you publish to Azure from Visual Studio.</span></span> <span data-ttu-id="44f14-141">Bei Verwendung der `configSource` -Attribut, mithilfe von PowerShell zum Erstellen und Bereitstellen Ihrer Website und die Datenbank, oder Sie können Erstellen der Website und die Datenbank in das Portal vor dem veröffentlichen.</span><span class="sxs-lookup"><span data-stu-id="44f14-141">If you are using the `configSource` attribute, you can use PowerShell to create and deploy your web site and database, or you can create the web site and the database in the portal before you publish.</span></span> <span data-ttu-id="44f14-142">Die [neu AzureWebsitewithDB.ps1](https://gallery.technet.microsoft.com/scriptcenter/Ultimate-Create-Web-SQL-DB-9e0fdfd3) Skript erstellt eine neue Website und der Datenbank.</span><span class="sxs-lookup"><span data-stu-id="44f14-142">The [New-AzureWebsitewithDB.ps1](https://gallery.technet.microsoft.com/scriptcenter/Ultimate-Create-Web-SQL-DB-9e0fdfd3) script will create a new web site and database.</span></span>


> [!WARNING]
> <span data-ttu-id="44f14-143">Sicherheit – im Gegensatz zu den *AppSettingsSecrets.config* -Datei muss die externe Verbindung Zeichenfolgen-Datei im gleichen Verzeichnis wie der Stamm *"Web.config"* Datei, daher Sie Vorsichtsmaßnahmen treffen, um sicherzustellen, dass Sie müssen Überprüfen Sie nicht in Ihrem Quellrepository.</span><span class="sxs-lookup"><span data-stu-id="44f14-143">Security - Unlike the *AppSettingsSecrets.config* file, the external connection strings file must be in the same directory as the root *web.config* file, so you'll have to take precautions to ensure you don't check it into your source repository.</span></span>


> [!NOTE]
> <span data-ttu-id="44f14-144">**Sicherheitswarnung für geheime Schlüssel Datei:** ist ein bewährtes Produktion geheime Schlüssel in Test- und nicht zu verwenden.</span><span class="sxs-lookup"><span data-stu-id="44f14-144">**Security Warning on secrets file:** A best practice is to not use production secrets in test and development.</span></span> <span data-ttu-id="44f14-145">Produktion Kennwörter in Test- oder Entwicklungsumgebung mit Speicherverlusten dieser geheimen Schlüssel.</span><span class="sxs-lookup"><span data-stu-id="44f14-145">Using production passwords in test or development leaks those secrets.</span></span>


<a id="wj"></a>
## <a name="webjobs-console-apps"></a><span data-ttu-id="44f14-146">WebJobs-Konsolen-apps</span><span class="sxs-lookup"><span data-stu-id="44f14-146">WebJobs console apps</span></span>

<span data-ttu-id="44f14-147">Die *"App.config"* vom eine Konsolen-app verwendete Datei relative Pfade unterstützt, aber absolute Pfade werden unterstützt.</span><span class="sxs-lookup"><span data-stu-id="44f14-147">The *app.config* file used by a console app doesn't support relative paths, but it does support absolute paths.</span></span> <span data-ttu-id="44f14-148">Einen absoluten Pfad können Sie um Ihre vertraulichen Daten aus dem Projektverzeichnis zu verschieben.</span><span class="sxs-lookup"><span data-stu-id="44f14-148">You can use an absolute path to move your secrets out of your project directory.</span></span> <span data-ttu-id="44f14-149">Das folgende Markup zeigt der geheimen Schlüssel in der *C:\secrets\AppSettingsSecrets.config* Datei und nicht vertrauliche Daten in der *"App.config"* Datei.</span><span class="sxs-lookup"><span data-stu-id="44f14-149">The following markup shows the secrets in the *C:\secrets\AppSettingsSecrets.config* file, and non sensitive data in the *app.config* file.</span></span>

[!code-xml[Main](best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure/samples/sample5.xml?highlight=2)]

<a id="da"></a>
## <a name="deploying-secrets-to-azure"></a><span data-ttu-id="44f14-150">Geheime Schlüssel in Azure bereitstellen.</span><span class="sxs-lookup"><span data-stu-id="44f14-150">Deploying secrets to Azure</span></span>

<span data-ttu-id="44f14-151">Wenn Sie Ihre Web-app in Azure, Bereitstellen der *AppSettingsSecrets.config* (das gewünschte) Datei wird nicht bereitgestellt werden.</span><span class="sxs-lookup"><span data-stu-id="44f14-151">When you deploy your web app to Azure, the *AppSettingsSecrets.config* file won't be deployed (that's what you want).</span></span> <span data-ttu-id="44f14-152">Konnte zur der [Azure-Verwaltungsportal](https://azure.microsoft.com/services/management-portal/) und richten Sie sie manuell, um dies vorzunehmen:</span><span class="sxs-lookup"><span data-stu-id="44f14-152">You could go to the [Azure Management Portal](https://azure.microsoft.com/services/management-portal/) and set them manually, to do that:</span></span>

1. <span data-ttu-id="44f14-153">Wechseln Sie zu [https://portal.azure.com](https://portal.azure.com), und melden Sie sich mit Ihrem Azure-Anmeldeinformationen.</span><span class="sxs-lookup"><span data-stu-id="44f14-153">Go to [https://portal.azure.com](https://portal.azure.com), and sign in with your Azure credentials.</span></span>
2. <span data-ttu-id="44f14-154">Klicken Sie auf **Durchsuchen &gt; Web-Apps**, klicken Sie dann auf den Namen des Web-app.</span><span class="sxs-lookup"><span data-stu-id="44f14-154">Click **Browse &gt; Web Apps**, then click the name of your web app.</span></span>
3. <span data-ttu-id="44f14-155">Klicken Sie auf **alle Einstellungen &gt; Anwendungseinstellungen**.</span><span class="sxs-lookup"><span data-stu-id="44f14-155">Click **All settings &gt; Application settings**.</span></span>

<span data-ttu-id="44f14-156">Der **Anwendungseinstellungen** und **Verbindungszeichenfolge** Werte überschreiben die gleichen Einstellungen in der *"Web.config"* Datei.</span><span class="sxs-lookup"><span data-stu-id="44f14-156">The **app settings** and **connection string** values override the same settings in the *web.config* file.</span></span> <span data-ttu-id="44f14-157">In unserem Beispiel wir nicht diese Einstellungen in Azure bereitstellen, aber das wäre, wenn diese Schlüssel der *"Web.config"* Datei, die die Einstellungen im Portal angezeigt würde haben Vorrang vor.</span><span class="sxs-lookup"><span data-stu-id="44f14-157">In our example, we did not deploy these settings to Azure, but if these keys were in the *web.config* file, the settings shown on the portal would take precedence.</span></span>

<span data-ttu-id="44f14-158">Eine bewährte Methode besteht darin, befolgen eine [DevOps-Workflow](../../../aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything.md) und [Azure PowerShell](https://azure.microsoft.com/en-us/documentation/articles/install-configure-powershell/) (oder einem anderen Framework, z. B. [Chef](http://www.opscode.com/chef/) oder [Puppet](http://puppetlabs.com/puppet/what-is-puppet)), diese Einstellungswerte in Azure zu automatisieren.</span><span class="sxs-lookup"><span data-stu-id="44f14-158">A best practice is to follow a [DevOps workflow](../../../aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/automate-everything.md) and use [Azure PowerShell](https://azure.microsoft.com/en-us/documentation/articles/install-configure-powershell/) (or another framework such as [Chef](http://www.opscode.com/chef/) or [Puppet](http://puppetlabs.com/puppet/what-is-puppet)) to automate setting these values in Azure.</span></span> <span data-ttu-id="44f14-159">Das folgende PowerShell-Skript verwendet [Export CliXml](http://www.powershellcookbook.com/recipe/PukO/securely-store-credentials-on-disk) So exportieren Sie die verschlüsselten geheimen Schlüssel auf den Datenträger:</span><span class="sxs-lookup"><span data-stu-id="44f14-159">The following PowerShell script uses [Export-CliXml](http://www.powershellcookbook.com/recipe/PukO/securely-store-credentials-on-disk) to export the encrypted secrets to disk:</span></span>

[!code-powershell[Main](best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure/samples/sample6.ps1)]

<span data-ttu-id="44f14-160">In das obige Skript ist "Name" der Name des geheimen Schlüssels, z. B. "&quot;FB\_AppSecret&quot; oder"TwitterSecret".</span><span class="sxs-lookup"><span data-stu-id="44f14-160">In the script above, ‘Name' is the name of the secret key, such as ‘&quot;FB\_AppSecret&quot; or "TwitterSecret".</span></span> <span data-ttu-id="44f14-161">Sie können die Datei ".credential" unter Verwendung des Skripts im Browser anzeigen.</span><span class="sxs-lookup"><span data-stu-id="44f14-161">You can view the ".credential" file created by the script in your browser.</span></span> <span data-ttu-id="44f14-162">Der Codeausschnitt unten testet die Anmeldeinformationen-Dateien und legt die geheimen Schlüssel für die benannte Web-app:</span><span class="sxs-lookup"><span data-stu-id="44f14-162">The snippet below tests each of the credential files and sets the secrets for the named web app:</span></span>

[!code-powershell[Main](best-practices-for-deploying-passwords-and-other-sensitive-data-to-aspnet-and-azure/samples/sample7.ps1)]

> [!WARNING]
> <span data-ttu-id="44f14-163">Sicherheit – keine Kennwörter oder andere geheime Schlüssel in das PowerShell-Skript, das auf diese Weise entspricht dies der Fall ist der Zweck der Verwendung von PowerShell-Skript zum Bereitstellen von sensiblen Daten enthalten.</span><span class="sxs-lookup"><span data-stu-id="44f14-163">Security - Don't include passwords or other secrets in the PowerShell script, doing so defeats the purpose of using a PowerShell script to deploy sensitive data.</span></span> <span data-ttu-id="44f14-164">Die [Get-Credential](https://technet.microsoft.com/en-us/library/hh849815.aspx) Cmdlet bietet einen sicheren Mechanismus zum Abrufen eines Kennworts.</span><span class="sxs-lookup"><span data-stu-id="44f14-164">The [Get-Credential](https://technet.microsoft.com/en-us/library/hh849815.aspx) cmdlet provides a secure mechanism to obtain a password.</span></span> <span data-ttu-id="44f14-165">Eine UI-Eingabe kann zu verhindern, dass ein Kennwort zu gelangen.</span><span class="sxs-lookup"><span data-stu-id="44f14-165">Using a UI prompt can prevent leaking a password.</span></span>


### <a name="deploying-db-connection-strings"></a><span data-ttu-id="44f14-166">Bereitstellen von DB-Verbindungszeichenfolgen</span><span class="sxs-lookup"><span data-stu-id="44f14-166">Deploying DB connection strings</span></span>

<span data-ttu-id="44f14-167">DB-Verbindungszeichenfolgen werden an den Appeinstellungen auf ähnliche Weise behandelt.</span><span class="sxs-lookup"><span data-stu-id="44f14-167">DB connection strings are handled similarly to app settings.</span></span> <span data-ttu-id="44f14-168">Wenn Sie Ihre Web-app aus Visual Studio bereitstellen, wird die Verbindungszeichenfolge für Sie konfiguriert werden.</span><span class="sxs-lookup"><span data-stu-id="44f14-168">If you deploy your web app from Visual Studio, the connection string will be configured for you.</span></span> <span data-ttu-id="44f14-169">Dies ist im Portal zu überprüfen.</span><span class="sxs-lookup"><span data-stu-id="44f14-169">You can verify this in the portal.</span></span> <span data-ttu-id="44f14-170">Die empfohlene Methode zum Festlegen der Verbindungszeichenfolge wird mit PowerShell.</span><span class="sxs-lookup"><span data-stu-id="44f14-170">The recommended way to set the connection string is with PowerShell.</span></span> <span data-ttu-id="44f14-171">Ein Beispiel für ein PowerShell-Skript die eine Website und die Datenbank erstellt und wird die Verbindungszeichenfolge auf der Website herunterladen [neu AzureWebsitewithDB.ps1](https://gallery.technet.microsoft.com/scriptcenter/Ultimate-Create-Web-SQL-DB-9e0fdfd3) aus der [-Skript für Azure-Bibliothek](https://gallery.technet.microsoft.com/scriptcenter/site/search?f%5B0%5D.Type=RootCategory&amp;f%5B0%5D.Value=WindowsAzure).</span><span class="sxs-lookup"><span data-stu-id="44f14-171">For an example of a PowerShell script the creates a website and database and sets the connection string in the website, download [New-AzureWebsitewithDB.ps1](https://gallery.technet.microsoft.com/scriptcenter/Ultimate-Create-Web-SQL-DB-9e0fdfd3) from the [Azure Script library](https://gallery.technet.microsoft.com/scriptcenter/site/search?f%5B0%5D.Type=RootCategory&amp;f%5B0%5D.Value=WindowsAzure).</span></span>

<a id="not"></a>
## <a name="notes-for-php"></a><span data-ttu-id="44f14-172">Hinweise für PHP</span><span class="sxs-lookup"><span data-stu-id="44f14-172">Notes for PHP</span></span>

<span data-ttu-id="44f14-173">Seit der Schlüssel-/ Wertpaare für beide **Anwendungseinstellungen** und **Verbindungszeichenfolgen** sind gespeichert in Umgebungsvariablen in Azure App Service, Entwickler, die alle Web-app-Frameworks (z. B. PHP) können problemlos verwenden Diese Werte abrufen.</span><span class="sxs-lookup"><span data-stu-id="44f14-173">Since the key-value pairs for both **app settings** and **connection strings** are stored in environment variables on Azure App Service, developers that use any web app frameworks (such as PHP) can easily retrieve these values.</span></span> <span data-ttu-id="44f14-174">Finden Sie unter der Stefan Schackow [Windows Azure-Websites: wie Anwendungszeichenfolgen und Verbindung Zeichenfolgen können](https://azure.microsoft.com/blog/2013/07/17/windows-azure-web-sites-how-application-strings-and-connection-strings-work/) Blogbeitrag, die in einen PHP-Ausschnitt zum Lesen von Anwendungseinstellungen und Verbindungszeichenfolgen dargestellt wird.</span><span class="sxs-lookup"><span data-stu-id="44f14-174">See Stefan Schackow's [Windows Azure Web Sites: How Application Strings and Connection Strings Work](https://azure.microsoft.com/blog/2013/07/17/windows-azure-web-sites-how-application-strings-and-connection-strings-work/) blog post which shows a PHP snippet to read app settings and connection strings.</span></span>

## <a name="notes-for-on-premises-servers"></a><span data-ttu-id="44f14-175">Anmerkungen zu dieser lokalen Server</span><span class="sxs-lookup"><span data-stu-id="44f14-175">Notes for on-premises servers</span></span>

<span data-ttu-id="44f14-176">Wenn Sie zu einer lokalen Webservern bereitstellen, helfen Sie sichere Kennwörter durch [Verschlüsseln der Konfigurationsabschnitte Konfigurationsdateien](https://msdn.microsoft.com/en-us/library/ff647398.aspx).</span><span class="sxs-lookup"><span data-stu-id="44f14-176">If you are deploying to on-premises web servers, you can help secure secrets by [encrypting the configuration sections of configuration files](https://msdn.microsoft.com/en-us/library/ff647398.aspx).</span></span> <span data-ttu-id="44f14-177">Als Alternative können Sie den gleichen Ansatz für Azure-Websites empfohlen: Beibehalten der entwicklungseinstellungen in Konfigurationsdateien und verwendet Sie Werte für Umgebungsvariablen für die Produktion.</span><span class="sxs-lookup"><span data-stu-id="44f14-177">As an alternative, you can use the same approach recommended for Azure Websites: keep development settings in configuration files, and use environment variable values for production settings.</span></span> <span data-ttu-id="44f14-178">In diesem Fall jedoch, Sie schreiben müssen, Anwendungscode für die Funktionalität, die in Azure-Websites automatisch ausgeführt wird: Abrufen von Einstellungen aus der Umgebungsvariablen und anhand dieser Werte anstelle Dienstkonfigurations-dateieinstellungen bzw. Dienstkonfigurations-dateieinstellungen bei Umgebungsvariablen werden nicht gefunden.</span><span class="sxs-lookup"><span data-stu-id="44f14-178">In this case, however, you have to write application code for functionality that is automatic in Azure Websites: retrieve settings from environment variables and use those values in place of configuration file settings, or use configuration file settings when environment variables are not found.</span></span>

<a id="addRes"></a>
## <a name="additional-resources"></a><span data-ttu-id="44f14-179">Zusätzliche Ressourcen</span><span class="sxs-lookup"><span data-stu-id="44f14-179">Additional Resources</span></span>

<span data-ttu-id="44f14-180">Für ein Beispiel für ein PowerShell-Skript, das erstellt wird, eine Web-app + Datenbank, legt fest, die Verbindungszeichenfolge + app-Einstellungen, Download [neu AzureWebsitewithDB.ps1](https://gallery.technet.microsoft.com/scriptcenter/Ultimate-Create-Web-SQL-DB-9e0fdfd3) aus der [-Skript für Azure-Bibliothek](https://gallery.technet.microsoft.com/scriptcenter/site/search?f%5B0%5D.Type=RootCategory&amp;f%5B0%5D.Value=WindowsAzure).</span><span class="sxs-lookup"><span data-stu-id="44f14-180">For an example of a PowerShell script that creates a web app + database, sets the connection string + app settings, download [New-AzureWebsitewithDB.ps1](https://gallery.technet.microsoft.com/scriptcenter/Ultimate-Create-Web-SQL-DB-9e0fdfd3) from the [Azure Script library](https://gallery.technet.microsoft.com/scriptcenter/site/search?f%5B0%5D.Type=RootCategory&amp;f%5B0%5D.Value=WindowsAzure).</span></span> 

<span data-ttu-id="44f14-181">Finden Sie unter der Stefan Schackow [Windows Azure-Websites: wie Anwendungszeichenfolgen und Verbindung Zeichenfolgen arbeiten](https://azure.microsoft.com/blog/2013/07/17/windows-azure-web-sites-how-application-strings-and-connection-strings-work/)</span><span class="sxs-lookup"><span data-stu-id="44f14-181">See Stefan Schackow's [Windows Azure Web Sites: How Application Strings and Connection Strings Work](https://azure.microsoft.com/blog/2013/07/17/windows-azure-web-sites-how-application-strings-and-connection-strings-work/)</span></span>


<span data-ttu-id="44f14-182">Besonderer Dank gilt Barry Dorrans ( [ @blowdart ](https://twitter.com/blowdart) ) und Carlos Farre für die Überprüfung.</span><span class="sxs-lookup"><span data-stu-id="44f14-182">Special thanks to Barry Dorrans ( [@blowdart](https://twitter.com/blowdart) ) and Carlos Farre for reviewing.</span></span>