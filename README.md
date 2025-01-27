## MariGold.OpenXHTML.RtL
Based on [MariaGold.OpenXHTML v3.0.0](https://www.nuget.org/packages/MariGold.OpenXHTML/3.0.0).

## MariGold.OpenXHTML
OpenXHTML is a wrapper library for Open XML SDK to convert HTML documents into Open XML word documents. It has simply encapsulated the complexity of Open XML yet exposes the properties of Open XML for manipulation.

### Installing via NuGet

In Package Manager Console, enter the following command:
```
Install-Package MariGold.OpenXHTML
```
### Usage
To create an empty Open XML word document using the OpenXHTML, use the following code.

```csharp
using MariGold.OpenXHTML;

WordDocument doc = new WordDocument("sample.docx");
doc.Save();
```
To create an Open XML document from an HTML document, use the following code.

```csharp
using MariGold.OpenXHTML;

WordDocument doc = new WordDocument("sample.docx");
doc.Process(new HtmlParser("<div>sample text</div>"));
doc.Save();
```
Once the HTML is processed, you can access the Open XML document using the following properties in WordDocument.
 

```csharp
public WordprocessingDocument WordprocessingDocument { get; }
public MainDocumentPart MainDocumentPart { get; }
public Document Document { get; }
```
Any modifications on Open XML document should be done before the Save method. This has to be done since the Save method will write all the changes and unload the document from the memory. So any further modifications may result in an exception. For example, if you want to append a paragraph at the document body, try the following code.
```csharp
using MariGold.OpenXHTML;
using DocumentFormat.OpenXml.Wordprocessing;

WordDocument doc = new WordDocument("sample.docx");
doc.Process(new HtmlParser("<div>sample text</div>"));
doc.Document.Body.AppendChild<Paragraph>(new Paragraph(new Run(new Text("added text"))));
doc.Save();
```
You can also create an Open XML document in memory. Following example illustrates how to save the document in a MemoryStream.

```csharp
using (MemoryStream mem = new MemoryStream())
{
	WordDocument doc = new WordDocument(mem);
	doc.Save();
}		
```

#### Relative Images
OpenXHTML cannot process the images with relative URL. This can be solved using the ImagePath property to set the base address for every relative image paths. The image path can be either a URL or a physical folder address.

```csharp
using MariGold.OpenXHTML;

WordDocument doc = new WordDocument("sample.docx");
doc.ImagePath = "http:\\abc.com";
doc.Process(new HtmlParser("<img src=\"sample.png\" />"));
doc.Save();
```

You can also assign any file URI address on image path.
```csharp
doc.ImagePath = @"file:///C:/Img";
```

#### Base URL
Like relative images, an HTML document may also contain links with relative path. This can be resolved using the BaseURL property.

```csharp
using MariGold.OpenXHTML;

WordDocument doc = new WordDocument("sample.docx");
doc.BaseURL = "http:\\abc.com";
doc.Process(new HtmlParser("<a href=\"index.htm\">sample</a>"));
doc.Save();
```
Also, if there are any relative images in the given html document and ImagePath is not assigned, OpenXHTML will attempt to use BaseURL to resolve relative image paths. So using BaseURL, you can resolve both relative image paths and links. The reason to create a seperate property for image path is that sometimes image location is different from base URL address.

#### Uri Schema

The protocol relative URLs can be resolved using the UriSchema property. 

```csharp
doc.UriSchema = Uri.UriSchemeHttp;
```

#### HTML Parsing
OpenXHTML has a built-in HTML and CSS parser (MariGold.HtmlParser) which can be complectly replaced with any external HTML and CSS parser. The Process method in WordDocument class expects an IParser interface type implementation to process the HTML and CSS. You can create an implementation of this IParser interface to parse the HTML and CSS.
```csharp
public void Process(IParser parser);
```

```csharp
interface IParser
{
	string BaseURL { get; set; }
	string UriSchema { get; set; }

	decimal CalculateRelativeChildFontSize(string parentFontSize, string childFontSize);
	IHtmlNode FindBodyOrFirstElement();
}
```
Here is the structure of IParser. The BaseURL and UriSchema are just two simple properties to store the base url address and uri schema for processing the HTML images and links. Both properties are used to resolve the protocol free and relative path of external style sheet URLs. The CalculateRelativeChildFontSize method is used to calculate the relative child font size. For example, in the below html, the font size of the h1 tag is 20 pixel. 

```html
<div style="font-size:16px"><h1>sample</h1></div>
```

If you don't want to re-implement this functionality, you can simply use the CSSUtility class in your implementation.

```csharp
using MariGold.HtmlParser;

return CSSUtility.CalculateRelativeChildFontSize(parentFontSize, childFontSize);
```

The FindBodyOrFirstElement method is expected to return an IHtmlNode representation of html body tag and the hierarchy of its child elements. If the document does not have body element, then it is expected to return the first root element. All the CSS styles and HTML attributes of IHtmlNode must be resolved and filled in the respective properties.

#### References
[Convert HTML to Word Document using CKEditor and MariGold.OpenXHTML](https://www.codeproject.com/Tips/1193272/Convert-HTML-to-Word-Document-using-CKEditor-and-M)

[Windows Forms Application - Convert HTML Files To DOCX Files With MariGold.OpenXHTML](http://www.c-sharpcorner.com/article/convert-html-files-to-docx-files-with-marigold-openxhtml/)

[Implement Custom HTML Parser using AngleSharp](https://www.codeproject.com/Tips/1221178/Convert-HTML-to-Open-XML-Word-document-using-MariG)