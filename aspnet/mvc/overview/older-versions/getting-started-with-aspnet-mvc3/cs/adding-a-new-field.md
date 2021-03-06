---
uid: mvc/overview/older-versions/getting-started-with-aspnet-mvc3/cs/adding-a-new-field
title: "Hinzufügen eines neuen Felds, der Movie-Modell und der Tabelle (c#) | Microsoft Docs"
author: Rick-Anderson
description: In diesem Lernprogramm erfahren Sie die Grundlagen der Erstellung einer ASP.NET MVC-Webanwendung mithilfe von Microsoft Visual Web Developer 2010 Express Service Pack 1, also...
ms.author: aspnetcontent
manager: wpickett
ms.date: 01/12/2011
ms.topic: article
ms.assetid: b4e76c1a-f66e-43a0-aa72-f39df79c07c1
ms.technology: dotnet-mvc
ms.prod: .net-framework
msc.legacyurl: /mvc/overview/older-versions/getting-started-with-aspnet-mvc3/cs/adding-a-new-field
msc.type: authoredcontent
ms.openlocfilehash: c10f3be30a92a605c34fa1c56fa3691389374beb
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/10/2017
---
<a name="adding-a-new-field-to-the-movie-model-and-table-c"></a>Hinzufügen eines neuen Felds, der Movie-Modell und der Tabelle (c#)
====================
Durch [Rick Anderson](https://github.com/Rick-Anderson)

> > [!NOTE]
> > Eine aktualisierte Version dieses Lernprogramms steht [hier](../../../getting-started/introduction/getting-started.md) , ASP.NET MVC 5 und Visual Studio 2013 verwendet. Es ist sicherer, viel einfacher, führen und weitere Funktionen veranschaulicht.
> 
> 
> In diesem Lernprogramm erfahren Sie die Grundlagen der Erstellung einer ASP.NET MVC-Webanwendung mithilfe von Microsoft Visual Web Developer 2010 Express Service Pack 1, also eine kostenlose Version von Microsoft Visual Studio. Bevor Sie beginnen, stellen Sie sicher, dass Sie die unten aufgeführten erforderlichen Komponenten installiert haben. Sie können alle installieren, indem Sie auf den folgenden Link: [Webplattform-Installer](https://www.microsoft.com/web/gallery/install.aspx?appid=VWD2010SP1Pack). Alternativ können Sie die erforderlichen Komponenten, die über die folgenden Links einzeln installieren:
> 
> - [Visual Studio Web Developer Express SP1-Voraussetzungen](https://www.microsoft.com/web/gallery/install.aspx?appid=VWD2010SP1Pack)
> - [ASP.NET MVC 3 Tools Update](https://www.microsoft.com/web/gallery/install.aspx?appsxml=&amp;appid=MVC3)
> - [SQL Server Compact 4.0](https://www.microsoft.com/web/gallery/install.aspx?appid=SQLCE;SQLCEVSTools_4_0)(Common Language Runtime + Tools unterstützen)
> 
> Wenn Sie Visual Studio 2010 anstelle von Visual Web Developer 2010 verwenden, die Komponenten installieren, indem Sie auf den folgenden Link: [Visual Studio 2010-Voraussetzungen](https://www.microsoft.com/web/gallery/install.aspx?appsxml=&amp;appid=VS2010SP1Pack).
> 
> Ein Visual Web Developer-Projekt mit C#-Quellcode ist zu diesem Thema steht zur Verfügung. [Die C#-Version herunterladen](https://code.msdn.microsoft.com/Introduction-to-MVC-3-10d1b098). Wenn Sie Visual Basic bevorzugen, wechseln Sie zu der [Visual Basic-Version](../vb/intro-to-aspnet-mvc-3.md) dieses Lernprogramm.


In diesem Abschnitt Sie einige Änderungen an der Modellklassen vornehmen und erfahren Sie, wie Sie das Datenbankschema die modelländerungen entsprechend aktualisieren.

## <a name="adding-a-rating-property-to-the-movie-model"></a>Hinzufügen einer Rating-Eigenschaft zum Movie-Modell

Starten, indem er ein neues `Rating` Eigenschaft, um die vorhandene `Movie` Klasse. Öffnen der *Movie.cs* und fügen die `Rating` wie die folgende Eigenschaft:

[!code-csharp[Main](adding-a-new-field/samples/sample1.cs)]

Die vollständige `Movie` -Klasse sieht wie der folgende Code:

[!code-csharp[Main](adding-a-new-field/samples/sample2.cs)]

Kompilieren Sie die Anwendung mithilfe der **Debuggen** &gt; **Film erstellen** Menübefehl.

Nun, dass Sie aktualisiert haben die `Model` -Klasse, Sie müssen auch zum Aktualisieren der *\Views\Movies\Index.cshtml* und *\Views\Movies\Create.cshtml* Vorlagen anzeigen, um die neue unterstützen`Rating`Eigenschaft.

Öffnen der *\Views\Movies\Index.cshtml* und fügen eine `<th>Rating</th>` Spaltenüberschrift direkt nach der **Preis** Spalte. Fügen Sie dann eine `<td>` Spalte am Ende der Vorlage zum Rendern der `@item.Rating` Wert. Im folgenden finden Sie welche die aktualisierte *Index.cshtml* Vorlage anzeigen sieht wie folgt aus:

[!code-cshtml[Main](adding-a-new-field/samples/sample3.cshtml)]

Öffnen Sie als Nächstes die *\Views\Movies\Create.cshtml* Datei, und fügen Sie das folgende Markup gegen Ende des Formulars. Dies rendert ein Textfeld, sodass Sie eine Bewertung angeben können, wenn ein neue Film erstellt wird.

[!code-cshtml[Main](adding-a-new-field/samples/sample4.cshtml)]

## <a name="managing-model-and-database-schema-differences"></a>Verwalten von Modell und Datenbank-Schema-Unterschiede

Sie haben jetzt den Code der Anwendung, um die Unterstützung der neuen `Rating` Eigenschaft.

Die Anwendung jetzt ausführen, und navigieren Sie zu der */Movies* URL. Wenn Sie dies tun, aber sehen die folgenden Fehler Sie:

![](adding-a-new-field/_static/image1.png)

Dieser Fehler gemeldet wird, da die aktualisierte `Movie` Modellklasse in der Anwendung unterscheidet sich von dem Schema der der `Movie` Tabelle der vorhandenen Datenbank. (Die Datenbanktabelle enthält nicht die Spalte `Rating`.)

Standardmäßig Wenn Sie Entity Framework Code First verwenden, um automatisch eine Datenbank erstellen, wie Sie weiter oben in diesem Lernprogramm ausgeführt haben Code First eine Tabelle der Datenbank hinzugefügt, um nachzuverfolgen, ob das Schema der Datenbank mit der Modellklassen synchron ist, die sie aus generiert wurde. Wenn sie nicht synchron sind, löst das Entity Framework einen Fehler auf. Dies erleichtert das Identifizieren von Problemen zur Entwicklungszeit zu verfolgen, die andernfalls nur (durch kryptische Fehler) zur Laufzeit auftreten. Die Synchronisierung wird überprüft-Funktion ist Ursachen die Fehlermeldung angezeigt werden, den Sie soeben gesehen haben.

Es gibt zwei Ansätze zum Beheben des Fehlers:

1. Lassen Sie die Datenbank von Entity Framework automatisch löschen und basierend auf dem neuen Modellklassenschema neu erstellen. Dieser Ansatz ist sehr praktisch, wenn aktiven Entwicklung auf eine Testdatenbank auf diese Weise, da es Ihnen ermöglicht, die das Modell und Datenbank-Schema schnell gemeinsam entwickeln. Der Nachteil ist jedoch, dass Sie in der Datenbank vorhandene Daten verloren gehen – damit Sie *nicht* dieser Ansatz in einer Produktionsdatenbank verwendet werden soll.
2. Ändern Sie das Schema der vorhandenen Datenbank explizit so, dass es mit den Modellklassen übereinstimmt. Der Vorteil dieses Ansatzes ist, dass Sie Ihre Daten behalten. Sie können diese Änderung entweder manuell oder durch Erstellen eines Änderungsskripts für die Datenbank vornehmen.

Für dieses Lernprogramm verwenden wir die erste Methode – stehen Ihnen die Entity Framework Code First die Datenbank automatisch neu erstellt, jedes Mal, wenn das Modell ändert.

## <a name="automatically-re-creating-the-database-on-model-changes"></a>Erstellen die Datenbank auf Modelländerungen automatisch erneut

Sehen wir die Anwendung aktualisieren, sodass Code First automatisch gelöscht und neu die Datenbank erstellt jedes Mal, wenn Sie das Modell für die Anwendung ändern.

> [!NOTE] 
> 
> **Warnung** sollten Sie diesen Ansatz, der automatisch löschen und Neuerstellen der Datenbank nur bei Verwendung eine Datenbank Entwicklungs- oder Testserver aktivieren und *nie* in einer Produktionsdatenbank, die tatsächlichen Daten enthält. Verwenden es auf einem Produktionsserver kann zu Datenverlust führen.


In **Projektmappen-Explorer**, klicken Sie mit der rechten Maustaste auf die *Modelle* Ordner wählen **hinzufügen**, und wählen Sie dann **Klasse**.

![](adding-a-new-field/_static/image2.png)

Benennen Sie die Klasse "MovieInitializer". Update der `MovieInitializer` Klasse enthält den folgenden Code:

[!code-csharp[Main](adding-a-new-field/samples/sample5.cs)]

Die `MovieInitializer` Klasse gibt an, dass die Datenbank, die vom Modell verwendeten gelöscht und automatisch neu erstellt werden, wenn die Modellklassen jemals ändern. Der Code enthält eine `Seed` Methode, um anzugeben, einigen Standarddaten hinzuzufügende automatisch an die Datenbank eine Zeit erstellt wurde (oder neu erstellt). Dies bietet eine gute Möglichkeit, füllen Sie die Datenbank mit Beispieldaten, ohne dass Sie manuell er jedes Mal auffüllen, die Sie vornehmen, dass ein Modell zu ändern.

Nun, dass Sie definiert haben die `MovieInitializer` -Klasse, sollten Sie die oben zu verknüpfen, sodass jedes Mal die Anwendung ausgeführt wird, er überprüft, ob die Modellklassen aus dem Schema in der Datenbank unterschiedlich sind. Wenn dies der Fall, können Sie den Initialisierer zum Neuerstellen der Datenbank das Modell auswählen und anschließend füllen Sie die Datenbank mit den Beispieldaten ausführen.

Öffnen der *"Global.asax"* -Datei im Stammverzeichnis der `MvcMovies` Projekt:

[![](adding-a-new-field/_static/image4.png)](adding-a-new-field/_static/image3.png)

Die *"Global.asax"* -Datei enthält die Klasse, die die gesamte Anwendung für das Projekt definiert und enthält eine `Application_Start` -Ereignishandler, der beim ersten der Anwendung Start ausgeführt.

Fügen Sie zwei using-Anweisungen am Anfang der Datei. Die erste verweist auf das Entity Framework-Namespace, und die zweite verweist auf den Namespace, in dem unsere `MovieInitializer` Leben Klasse:

[!code-csharp[Main](adding-a-new-field/samples/sample6.cs)]

Suchen Sie dann die `Application_Start` Methode und fügen Sie einen Aufruf von `Database.SetInitializer` am Anfang der Methode, wie unten dargestellt:

[!code-csharp[Main](adding-a-new-field/samples/sample7.cs)]

Die `Database.SetInitializer` Anweisung, die Sie gerade hinzugefügt haben gibt an, dass die Datenbank von verwendet die `MovieDBContext` Instanz sollte automatisch gelöscht und neu erstellt werden, wenn das Schema und die Datenbank nicht übereinstimmen. Und wie Sie gesehen haben, wird es auch die Datenbank mit den Beispieldaten, die im angegebenen Füllen der `MovieInitializer` Klasse.

Schließen der *"Global.asax"* Datei.

Führen Sie die Anwendung erneut aus, und navigieren Sie zu der */Movies* URL. Beim Starten der Anwendung erkannt, dass die Modellstruktur nicht mehr das Datenbankschema entspricht. Wird automatisch neu erstellt die Datenbank entsprechend die neuen Modellstruktur und füllt die Datenbank mit dem Beispiel Filme:

![7_MyMovieList_SM](adding-a-new-field/_static/image5.png)

Klicken Sie auf die **neu erstellen** Link, um einen neuen Film hinzufügen. Beachten Sie, dass Sie eine Bewertung hinzufügen können.

[![7_CreateRioII](adding-a-new-field/_static/image7.png)](adding-a-new-field/_static/image6.png)

Klicken Sie auf **Erstellen**. Der neue Film, einschließlich der Bewertung wird jetzt im Filme auflisten:

[![7_ourNewMovie_SM](adding-a-new-field/_static/image9.png)](adding-a-new-field/_static/image8.png)

In diesem Abschnitt haben gesehen, wie Sie Modellobjekte ändern und die Datenbank mit den Änderungen synchron bleiben. Außerdem haben Sie gelernt, eine Möglichkeit, eine neu erstellte Datenbank mit Beispieldaten aufzufüllen, damit Sie die Szenarien ausprobieren können. Als Nächstes sehen wir uns wie können Sie der Modellklassen umfangreichere Validierungslogik hinzu, und aktivieren einige Geschäftsregeln erzwungen werden.

>[!div class="step-by-step"]
[Zurück](examining-the-edit-methods-and-edit-view.md)
[Weiter](adding-validation-to-the-model.md)
