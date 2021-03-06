# Horker Template Engine

The Horker Template Engine is a text preprocessor empowered by PowerShell.

## Installation

The Horker Template Engine is available in the [PowerShell Gallery](https://www.powershellgallery.com/packages/HorkerTemplateEngine).

```PowerShell
Install-Module HorkerTemplateEngine
```

## Cmdlet Synopsis

The Horker Template Engine exports a single cmdlet, `Invoke-TemplateEngine`.  It processes a document template that embeds PowerShell code and generates a plain text as an outcome to the standard output stream.

```PowerShell
Invoke-TemplateEngine [-Template] <string> [[-ProcessorFile] <string>] [<CommonParameters>]
```

- `-Template` A document template, given as a string.  If you want to provide a template from a file, use `Get-Content` and give it through pipeline.
- `-ProcessorFile` generates a internal processing script for debugging.

## Template Syntax

- Text portions enclosed with `<%` and `%>` are executed as PowerShell code.  The objects returned by the code into the standard output stream are written into a resultant document.
- Text portions enclosed with `<%` and `-%>` are processed as the same above, but the following newline will not appear in the output.
- The other text is written into the resultant document without any changes.

On code execution, the variables and the functions in the current session are available.

Note that, unlike usual PowerShell output, objects are not formatted in output; that is, they are converted into strings by the ToString() method and no newlines or spaces are inserted between them. See Example section for details.

## Examples

### Basic example

Consider a document template is prepared in `template.txt`, as follows.

```PowerShell
PS>Get-Content template.txt
Dear <% $customer %>,

This is a mail for you.
```

You can process a template to generate a document by the following command-line:

```PowerShell
PS>$customer = 'Bill'
PS>Get-Content $template | Invoke-TemplateEngine
Dear Bill,

This is a mail for you.
```

### Processing multiple documents

In practice, you may want to generate multiple documents at once.  It can be done by an ordinary PowerShell script as follows.

```PowerShell
$customers = "Bill", "Jane", "Steve"

$template = Get-Content template.txt

$customers | foreach {
  $customer = $_
  Invoke-TemplateEngine $template |
    Set-Content "mail_to_$customer.txt"
}
```

This script generates three files, in which each customer name is embeded.

### Output formatting

Let us see the following examples to understand how objects will be written into the output and how you can control it.

As an example data, we will use the following objects returned by the `dir` (`Get-ChildItem`) cmdlet:

```PowerShell
PS>dir

    Directory: C:\work

Mode          LastWriteTime Length Name
----          ------------- ------ ----
-a---- 2017/12/29     17:36    252 a.txt
-a---- 2017/12/29     17:36    252 b.txt
-a---- 2017/12/29     17:36    252 c.txt
```

First, a simple `dir` call in a template yields the following result:

```PowerShell
PS>Invoke-TemplateEngine '<% dir %>'
a.txtb.txtc.txt
```

This is because each FileInfo object is converted into a string by the ToString() method (which produces `a.txt` and so on), and no formatting is done including insertion of newlines or spaces.

If you want to format an output, put code explicitly to do so:

```PowerShell
PS>Invoke-TemplateEngine '<% (dir).FullName -join "`r`n" %>'
C:\work\a.txt
C:\work\b.txt
C:\work\c.txt
```

Or, you can use loop as follows:

```PowerShell
PS>Invoke-TemplateEngine '
>>><% dir | foreach { -%>
>>><%   $_.Name %>(size: <% $_.Length %>)
>>><% } -%>'
>>>
a.txt (size: 252)
b.txt (size: 252)
c.txt (size: 252)
```

Note that, in the above example, the first and last line end with `-%>` tags.  This tag is useful to supress unnecessary newlines after code blocks.

You can apply `Format-Table` and `Out-String` to generate host-like output, if you wish so.

```PowerShell
PS>Invoke-TemplateEngine '<% dir | ft -auto | out-string %>'

    Directory: C:\work

Mode          LastWriteTime Length Name
----          ------------- ------ ----
-a---- 2017/12/29     17:36    252 a.txt
-a---- 2017/12/29     17:36    252 b.txt
-a---- 2017/12/29     17:36    252 c.txt
```

## Dependencies

On runtime, there is no dependency on external modules.

On build time, it depends the following modules:

```PowerShell
Install-Module pester -Scope CurrentUser
Install-Module InvokeBuild -Scope CurrentUser
Install-Module XmlDoc2CmdletDoc -Destination lib
```

## License

The MIT License.
