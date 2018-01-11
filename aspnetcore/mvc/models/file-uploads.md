---
title: Dateiuploads in ASP.NET Core
author: ardalis
description: Verwendung von modellbindung und streaming zum Hochladen von Dateien in ASP.NET-MVC der Core.
keywords: ASP.NET Core, Dateiupload, Modell binden, IFormFile, streaming
ms.author: riande
manager: wpickett
ms.date: 07/05/2017
ms.topic: article
ms.assetid: ebc98159-a028-4a94-b06c-43981c79c6be
ms.technology: aspnet
ms.prod: asp.net-core
uid: mvc/models/file-uploads
ms.openlocfilehash: e8608a46d6688df8da6c665a25b6f4db5f480461
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/10/2017
---
# <a name="file-uploads-in-aspnet-core"></a>Dateiuploads in ASP.NET Core

Durch [Steve Smith](https://ardalis.com/)

ASP.NET MVC-Aktionen unterstützen das Hochladen von einer oder mehreren Dateien, die mit einfachen Modell Bindung für kleinere Dateien oder streaming für größere Dateien.

[Anzeigen oder Herunterladen des Beispiels von GitHub](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/models/file-uploads/sample/FileUploadSample)

## <a name="uploading-small-files-with-model-binding"></a>Hochladen von kleinen Dateien mit Modellbindung

Um kleine Dateien hochzuladen, können Sie ein mehrteiliges HTML-Formular oder eine POST-Anforderung, die mit JavaScript zu erstellen. Ein Beispiel-Formulars mit Razor, die mehrere hochgeladene Dateien unterstützt, wird unten gezeigt:

```html
<form method="post" enctype="multipart/form-data" asp-controller="UploadFiles" asp-action="Index">
    <div class="form-group">
        <div class="col-md-10">
            <p>Upload one or more files using this form:</p>
            <input type="file" name="files" multiple />
        </div>
    </div>
    <div class="form-group">
        <div class="col-md-10">
            <input type="submit" value="Upload" />
        </div>
    </div>
</form>
```

Damit Dateiuploads unterstützt werden, muss in den HTML-Formularen das Attribut `enctype` mit dem Wert `multipart/form-data` angegeben werden. Das `files` input-Element im Beispiel unterstützt das Hochladen von mehreren Dateien.Entfernen Sie das Attribut `multiple`  für dieses Element input, kann nur eine einzelne Datei hochgeladen werden. Das oben gezeigte Markup rendert in einem Browser als:

![Dateiupload-Formular](file-uploads/_static/upload-form.png)

Durch die einzelnen Dateien auf den Server hochgeladen zugegriffen werden können [Modell Bindung](xref:mvc/models/model-binding) mithilfe der [IFormFile](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.iformfile) Schnittstelle. `IFormFile`weist die folgende Struktur:

```csharp
public interface IFormFile
{
    string ContentType { get; }
    string ContentDisposition { get; }
    IHeaderDictionary Headers { get; }
    long Length { get; }
    string Name { get; }
    string FileName { get; }
    Stream OpenReadStream();
    void CopyTo(Stream target);
    Task CopyToAsync(Stream target, CancellationToken cancellationToken = null);
}
```

> [!WARNING]
> Nicht abhängig ist, oder vertrauen der `FileName` Eigenschaft ohne Überprüfung. Die `FileName` Eigenschaft sollte nur zu Anzeigezwecken verwendet werden.

Beim Hochladen von Dateien mit modellbindung und `IFormFile` -Schnittstelle, die Aktionsmethode kann entweder ein einzelnes akzeptieren `IFormFile` oder ein `IEnumerable<IFormFile>` (oder `List<IFormFile>`), die mehrere Dateien darstellt. Das folgende Beispiel durchläuft einen oder mehrere hochgeladene Dateien, speichert sie in das lokale Dateisystem und gibt die Anzahl und Größe der Dateien hochgeladen.

[!INCLUDE [GetTempFileName](../../includes/GetTempFileName.md)]

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Controllers/UploadFilesController.cs?name=snippet1)]

Dateien, die hochgeladen wird, mithilfe der `IFormFile` Verfahren werden im Arbeitsspeicher oder auf einem Datenträger, auf dem Webserver vor der Verarbeitung gepuffert. In der Aktionsmethode die `IFormFile` Inhalt als Stream verfügbar sind. Zusätzlich zu den im lokalen Dateisystem Dateien in gestreamt werden [Azure Blob-Speicher](https://azure.microsoft.com/documentation/articles/vs-storage-aspnet5-getting-started-blobs/) oder [Entity Framework](https://docs.microsoft.com/ef/core/index).

Definieren Sie eine Eigenschaft vom Typ zum Speichern von binären Daten in einer Datenbank mithilfe von Entity Framework `byte[]` für die Entität:

```csharp
public class ApplicationUser : IdentityUser
{
    public byte[] AvatarImage { get; set; }
}
```

Geben Sie eine Viewmodel-Eigenschaft des Typs `IFormFile`:

```csharp
public class RegisterViewModel
{
    // other properties omitted

    public IFormFile AvatarImage { get; set; }
}
```

> [!NOTE]
> `IFormFile`kann direkt als einen Aktionsmethodenparameter oder als Viewmodel-Eigenschaft verwendet werden wie oben gezeigt.

Kopieren der `IFormFile` in einen Stream und speichern Sie sie auf das Bytearray:

```csharp
// POST: /Account/Register
[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Register(RegisterViewModel model)
{
    ViewData["ReturnUrl"] = returnUrl;
    if  (ModelState.IsValid)
    {
        var user = new ApplicationUser {
          UserName = model.Email,
          Email = model.Email
        };
        using (var memoryStream = new MemoryStream())
        {
            await model.AvatarImage.CopyToAsync(memoryStream);
            user.AvatarImage = memoryStream.ToArray();
        }
    // additional logic omitted
    
    // Don't rely on or trust the model.AvatarImage.FileName property 
    // without validation.
}
```

> [!NOTE]
> Seien Sie beim Speichern von Binärdaten in relationalen Datenbanken, wie sie die Leistung beeinträchtigen kann.

## <a name="uploading-large-files-with-streaming"></a>Hochladen großer Dateien beim streaming

Wenn die Größe oder die Häufigkeit des Dateiuploads Ressourcenproblemen für die app verursacht wird, erwägen Sie streaming Hochladen der Datei, anstatt als Ganzes gepuffert wird, wie in der oben gezeigte Modell Bindung-Ansatz. Bei der Verwendung `IFormFile` und modellbindung ist eine viel einfachere Lösung streaming erfordert eine Reihe von Schritten, um ordnungsgemäß zu implementieren.

> [!NOTE]
> Jede einzelne gepufferte Datei 64KB überschreiten wird aus dem RAM in eine temporäre Datei auf dem Datenträger auf dem Server verschoben werden. Die vom Dateiuploads verwendeten Ressourcen (Datenträger, RAM) richten sich nach der Anzahl und Größe der gleichzeitigen Dateiuploads. Streaming nicht so viel über bezogen ist, wird über Dezimalstellen. Wenn Sie versuchen, zu viele Uploads zu Puffern, stürzt der Site genügend Arbeitsspeicher oder Speicherplatz.

Im folgenden Beispiel wird veranschaulicht, wie mit JavaScript/Angular auf eine Controlleraktion gestreamt wird. Die Datei antiforgery-Token wird generiert, verwenden eines benutzerdefinierten Attributs für den Filter und HTTP-Header anstelle der im Anforderungstext übergeben. Da die Aktionsmethode die hochgeladenen Daten direkt verarbeitet wird, ist von einem anderen Filter wurden die modellbindung deaktiviert. Innerhalb der Aktion werden das Formular Inhalt lesen, verwenden eine `MultipartReader`, die jedes einzelne liest `MultipartSection`, Verarbeiten der Datei oder Speichern des Inhalts nach Bedarf. Sobald alle Abschnitte gelesen wurden, führt die Aktion einen eigenen wurden die modellbindung an.

Die erste Aktion wird das Formular geladen und ein antiforgery Token in einem Cookie gespeichert (über die `GenerateAntiforgeryTokenCookieForAjax` Attribut):

```csharp
[HttpGet]
[GenerateAntiforgeryTokenCookieForAjax]
public IActionResult Index()
{
    return View();
}
```

Das Attribut verwendet, die ASP.NET Core integrierte [Antiforgery](xref:security/anti-request-forgery) Support, um ein Cookie mit einem Anforderungstoken festlegen:

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Filters/GenerateAntiforgeryTokenCookieForAjaxAttribute.cs?name=snippet1)]

Angular übergibt eine antiforgery Token automatisch in einen Anforderungsheader und mit dem Namen `X-XSRF-TOKEN`. Die ASP.NET-MVC-Anwendung Core ist so konfiguriert, dass zu diesem Header in der Konfiguration in finden Sie unter *Startup.cs*:

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Startup.cs?name=snippet1)]

Die `DisableFormValueModelBinding` Attribut, das weiter unten gezeigte wird verwendet, um die deaktiviert wurden die modellbindung für die `Upload` Aktionsmethode.

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Filters/DisableFormValueModelBindingAttribute.cs?name=snippet1)]

Da wurden die modellbindung deaktiviert ist, die `Upload` Aktionsmethode keine Parameter annehmen. Es arbeitet direkt mit der `Request` Eigenschaft `ControllerBase`. Ein `MultipartReader` wird verwendet, um jedes Abschnitts lesen. Die Datei wird mit einem GUID-Dateinamen gespeichert und befindet sich die Schlüssel/Wert-Daten in eine `KeyValueAccumulator`. Sobald alle Abschnitte gelesen wurde, den Inhalt der `KeyValueAccumulator` werden verwendet, um die Daten des Formulars an einen Modelltyp binden.

Die vollständige `Upload` Methode wird unten gezeigt:

[!INCLUDE [GetTempFileName](../../includes/GetTempFileName.md)]

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Controllers/StreamingController.cs?name=snippet1)]

## <a name="troubleshooting"></a>Problembehandlung

Im folgenden finden einige allgemeine Probleme, die beim Arbeiten mit Hochladen von Dateien und deren möglichen Lösungen auftreten.

### <a name="unexpected-not-found-error-with-iis"></a>Unerwarteter Fehler bei IIS wurde Nichtgefunden

Der folgende Fehler gibt an, das Hochladen der Datei überschreitet den Server konfigurierten `maxAllowedContentLength`:

```
HTTP 404.13 - Not Found
The request filtering module is configured to deny a request that exceeds the request content length.
```

Die Standardeinstellung ist `30000000`, also ungefähr 28.6 MB. Der Wert kann angepasst werden, indem bearbeiten *"Web.config"*:

```xml
<system.webServer>
  <security>
    <requestFiltering>
      <!-- This will handle requests up to 50MB -->
      <requestLimits maxAllowedContentLength="52428800" />
    </requestFiltering>
  </security>
</system.webServer>
```

Diese Einstellung gilt nur für IIS. Das Verhalten auftreten nicht standardmäßig, wenn auf Kestrel hosten. Weitere Informationen finden Sie unter [Anforderungslimits \<RequestLimits\>](https://docs.microsoft.com/iis/configuration/system.webServer/security/requestFiltering/requestLimits/).

### <a name="null-reference-exception-with-iformfile"></a>NULL-Verweisausnahme auftreten mit IFormFile

Wenn Ihr Controller annehmende ist hochgeladenen Dateien mithilfe von `IFormFile` , aber Sie feststellen, dass der Wert immer null ist, vergewissern Sie sich, dass das HTML-Formular angegeben ist ein `enctype` Wert `multipart/form-data`. Wenn dieses Attribut nicht festgelegt ist die `<form>` -Element, das Hochladen der Datei wird nicht ausgeführt und die Grenze `IFormFile` Argumente werden null sein.
