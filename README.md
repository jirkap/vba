# VBA 

[Collections, methods, objects and properties](https://www.wiseowl.co.uk/blog/s189/oop.htm)<br/>
[Peltier Tech Blog](https://peltiertech.com/)<br/>
[Good Patterns For VBA Error Handling – Stack Overflow](https://stackoverflow.com/questions/1038006/good-patterns-for-vba-error-handling)<br/>
[Declare a Workbook as a Global variable – Stack Overflow](https://stackoverflow.com/questions/31536519/declare-a-workbook-as-a-global-variable)

**Tools**<br/>
[Rubberduck](http://rubberduckvba.com/)<br/>
[VBA Auto-Indenter](https://www.automateexcel.com/vba/code-formatter/)<br/>
[https://github.com/Vitosh/VBA_personal](https://github.com/Vitosh/VBA_personal)

## AddIns

`%userprofile%\AppData\Roaming\Microsoft\AddIns`

## Module options

`Option Explicit` – use in every module to define variables using `Dim` or `ReDim` keywords

`Option Private Module` – hide module contents from the list of available macros

## Passing variables

`ByRef` – default,  alters passed value

`ByVal` – use a copy of the argument passed in and not modify the source

## Sub / Function accessibility

`Public` – Sub or Function available across modules

`Private` – Sub or Function available in containing module only

## Application

`Application.DisplayDocumentInformationPanel = True` – show metadata panel in document

## Error handling

### Resuming from labeled sequences without exiting Sub / Function

    ErrHandler:
        ' Some code
    Resume Label

    Label:
        ' Some code to be executed after ErrHandler  

## References

### Reference to Microsoft Forms 2.0 is not listed

Add via Browse... > System32 > FM20.dll 

***

# VBA snippets

## Set focus on UserForm load

    Private Sub UserForm_Activate()
        Control.SetFocus
    End Sub

## Press Enter in TextBox to run Sub

    Private Sub TextBox_KeyDown(ByVal KeyCode As MSForms.ReturnInteger, ByVal Shift As Integer)
        If KeyCode = 13 Then
            Control_Click
            KeyCode = 0
        End If
    End Sub

## Force Save As dialog

    ' In ThisWorkbook
    Private Sub Workbook_BeforeSave(ByVal SaveAsUI As Boolean, Cancel As Boolean)
        Dim strFile As String
        If SaveAsUI = True Then ' Check whether Save As was used
            Cancel = True
            strFile = JWorkbook.SaveAs("Filename", JConst.strFilterBrowseExcelMacroEnabled, "Dialog Title")
            If strFile = False Then
                Cancel = True
                Exit Sub
            End If
            Application.EnableEvents = False 
            ThisWorkbook.SaveAs File, xlOpenXMLWorkbookMacroEnabled
            Application.EnableEvents = True
        End If
    End Sub

## Simulating global variables

(Not reliable, throws the _out of range_ error unexpectedly)

    ' In Module1
    Public rngSomeRange As Range ' Application-wide, or
    Private rngSomeRange As Range ' module-wide, or
    Global rngSomeRange As Range ' ?

    Public Sub InitGlobals()
        Set rngSelectMain = Data.Range("rngSomeRange")
    End Sub

    ' In ThisWorkbook
    Private Sub Workbook_Open()
        Module1.InitGlobals
    End Sub

## Force users to enable Macros

(Keep stuff hidden until they do so)

    ' In ThisWorkbook
    Private Sub Workbook_Open()
        Dim ws As Worksheet
        For Each ws In ThisWorkbook.Worksheets
            ws.Visible = xlSheetVisible
        Next ws
        Sheets("WarningSheet").Visible = xlVeryHidden
    End Sub

    Private Sub Workbook_BeforeClose(Cancel As Boolean)
        Dim ws As Worksheet
        Sheets("WarningSheet").Visible = xlSheetVisible
        For Each ws In ThisWorkbook.Worksheets
            If ws.Name <> "WarningSheet" Then
                ws.Visible = xlVeryHidden
            End If
        Next ws
        ActiveWorkbook.Save
    End Sub

## Update all REF fields in a Word document

    Sub UpdateAllREFFields()
    ' Based on code at http://www.gmayor.com/installing_macro.htm
        Dim oStory As Range
        Dim oField As Field
        For Each oStory In ActiveDocument.StoryRanges
            For Each oField In oStory.Fields
                If oField.Type = wdFieldRef Then oField.Update
            Next oField
            If oStory.StoryType <> wdMainTextStory Then
                While Not (oStory.NextStoryRange Is Nothing)
                    Set oStory = oStory.NextStoryRange
                    For Each oField In oStory.Fields
                        If oField.Type = wdFieldRef Then oField.Update			
                    Next oField
                Wend
            End If
        Next oStory
        Set oStory = Nothing
        Set oField = Nothing
    End Sub
