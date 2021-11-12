# Generate barcodes with DLL in SSRS

## The DLL file
Copy the GenCode128.DLL into **D:\Program Files\Microsoft SQL Server Reporting Services\SSRS\ReportServer\bin**

## DLL Permission
Go to **D:\Program Files\Microsoft SQL Server Reporting Services\SSRS\ReportServer**
open the file **rssrvpolicy.config** with your preferred editor
and then after:
```xml
          </NamedPermissionSets>
          <CodeGroup class="FirstMatchCodeGroup" version="1" PermissionSetName="Nothing">
            <IMembershipCondition class="AllMembershipCondition" version="1" />
            <CodeGroup class="UnionCodeGroup" version="1" PermissionSetName="Execution" Name="Report_Expressions_Default_Permissions" Description="This code group grants default permissions for code in report expressions and Code element. ">
              <IMembershipCondition class="StrongNameMembershipCondition" version="1" PublicKeyBlob="0024000004800000940000000602000000240000525341310004000001000100512C8E872E28569E733BCB123794DAB55111A0570B3B3D4DE3794153DEA5EFB7C3FEA9F2D8236CFF320C4FD0EAD5F677880BF6C181F296C751C5F6E65B04D3834C02F792FEE0FE452915D44AFE74A0C27E0D8E4B8D04EC52A8E281E01FF47E7D694E6C7275A09AFCBFD8CC82705A06B20FD6EF61EBBA6873E29C8C0F2CAEDDA2" />
            </CodeGroup>
```
paste the following code:
```xml
            <CodeGroup class="FirstMatchCodeGroup" version="1" PermissionSetName="FullTrust" Name="IDAutomationNETAssembly" Description="This code group grants IDAutomationNETAssembly.dll Full Trust permission and allow Preview in Visual Studio.">
              <IMembershipCondition class="UrlMembershipCondition" version="1" Url="D:\Program Files\Microsoft SQL Server Reporting Services\SSRS\ReportServer\bin\GenCode128.dll" />
            </CodeGroup>
```

## Import the necessary DLLs in the Report Builder
Right click outside the report, then click on "Report Properties"
Select "References" in the left menu
Now it's necessary to add two references, the first one is to **GenCode128.dll**,
click on browse, navigate to the folder where we pasted it (**D:\Program Files\Microsoft SQL Server Reporting Services\SSRS\ReportServer\bin**) and select the dll file.

The second one it's the drawing DLL, click on browse again, go to **C:\Windows\Microsoft.NET\Framework\v2.0.50727**
and select the file **System.Drawing.dll**

## Add the custom code
In the report properties click on "Code" and then add the following code:
```
Function PaintBox(ByVal level As String) As System.Drawing.Bitmap 
Dim objBitmap As System.Drawing.Bitmap 
objBitmap = GenCode128.Code128Rendering.MakeBarcodeImage(level, 1, True)
Return objBitmap 
End Function

Function PaintBoxBmp(ByVal level As String) As Byte() 
Dim bmpImage As System.Drawing.Bitmap 
bmpImage = PaintBox(level) 
Dim stream As System.IO.MemoryStream = New IO.MemoryStream 
Dim bitmapBytes As Byte() 
bmpImage.Save(stream, System.Drawing.Imaging.ImageFormat.Bmp) 
bitmapBytes = stream.ToArray 
stream.Close() 
bmpImage.Dispose() 
Return bitmapBytes
End Function
```

## Use the function to generate barcodes
Inside a table cell or wherever you need, insert an image in your report
In the image properties, change the source to "Database"
In the "use field" click on the fx button and then add your expression using the created function, for example:
```
=Code.PaintBoxBmp(Fields!detail_column4.Value)
```
And finally in the MIME type select "image/bmp"


[Reference 1](https://social.technet.microsoft.com/wiki/contents/articles/29371.code-128-barcode-in-ssrs-open-source.aspx)
[Reference 2](https://www.codeproject.com/Articles/14409/GenCode128-A-Code128-Barcode-Generator)
