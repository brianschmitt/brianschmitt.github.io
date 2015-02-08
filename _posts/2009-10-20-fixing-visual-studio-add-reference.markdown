---
layout: post
title:  "Fixing the Visual Studio Add Reference Dialog – Quickly add a Project Reference"
comments: true
tags: [Visual Studio,Macros]
---


Today's macro addresses one of the most common complaints about Visual Studio - The &quot;Add Reference&quot; dialog - I will help you speed it up when you only need to add a project reference.

I know the issue is fixed in the latest beta of 2010 but I provide this here for those who are still using 2008 or lower.

I know we have all been there at one time or another – you add a new class library and want to add a reference in your UI to the project – but we dread the dialog box to add that reference - Will today be a lucky day and it only take 30 seconds to load? With this macro you can quickly add a reference to another project in the same solution. It offers a simple dialog box for you to choose one or more projects. Hope you find it just as useful as I do. [Kevin Dente](http://weblogs.asp.net/kdente/) had a great suggestion that the add reference dialog should support filtering/searching – This is possible and I'll see what I can do about it.

![image](/posts_images/add_project_reference.png)

Here is the code: (Scroll down for instructions on how you can add it to your context menu)

{% highlight vbnet %}
Imports SystemImports EnvDTE
Imports EnvDTE80
Imports EnvDTE90
Imports System.Diagnostics
Imports System.Windows.Forms
Imports System.Collections
Public Module References
    Public Sub AddProjectReference()
        Dim frm As New SelectProjectForm
        Dim winptr As WinWrapper = New WinWrapper
        Try
            Dim ret As DialogResult = frm.ShowDialog(winptr)
            If ret = DialogResult.Cancel Then
                Return
            End If
            Dim actvProjs As Array = DTE.ActiveSolutionProjects()
            Dim sngProj As Project = CType(actvProjs.GetValue(0), EnvDTE.Project)
            Dim vsProj As VSLangProj.VSProject = DirectCast(sngProj.Object, VSLangProj.VSProject)
            For Each proj As Project In frm.SelectedProjects
                Try
                    vsProj.References.AddProject(proj)
                Catch ex As Exception
                    MsgBox(ex.Message, MsgBoxStyle.Critical And MsgBoxStyle.OkOnly, &quot;Error - &quot; &amp; proj.Name)
                End Try
            Next
        Finally
            frm = Nothing
            winptr = Nothing
        End Try
    End Sub

    Private Class SelectProjectForm
        Inherits System.Windows.Forms.Form
        Public Sub New()
            InitializeComponent()
            lstProjects.DisplayMember = &quot;Name&quot;
            lstProjects.DataSource = GetAllProjects()
            lstProjects.Select()
        End Sub
        Public ReadOnly Property SelectedProjects() As Generic.List(Of Project)
            Get
                Dim lst As New Generic.List(Of Project)
                For Each itm As Object In lstProjects.SelectedItems
                    lst.Add(CType(itm, Project))
                Next
                Return lst
            End Get
        End Property
        Private Function GetAllProjects() As Generic.List(Of Project)
            Dim lst As New Generic.List(Of Project)
            For Each proj As Project In DTE.Solution.Projects
                If proj.Kind = Constants.vsProjectKindSolutionItems Then
                    lst.AddRange(GetSubProjects(proj.ProjectItems))
                Else
                    lst.Add(proj)
                End If
            Next
            Return lst
        End Function
        Private Function GetSubProjects(ByVal pis As ProjectItems) As Generic.List(Of Project)
            Dim lst As New Generic.List(Of Project)
            For Each pi As ProjectItem In pis
                If pi.Kind = Constants.vsProjectItemKindSolutionItems Then
                    lst.Add(pi.SubProject)
                ElseIf pi.Kind = Constants.vsProjectKindSolutionItems Then
                    lst.AddRange(GetSubProjects(pi.ProjectItems))
                End If
            Next
            Return lst
        End Function
        Private Sub InitializeComponent()
            Me.btnOk = New System.Windows.Forms.Button
            Me.btnCancel = New System.Windows.Forms.Button
            Me.lstProjects = New System.Windows.Forms.ListBox
            Me.SuspendLayout()
            Me.btnOk.Anchor = CType((System.Windows.Forms.AnchorStyles.Bottom Or System.Windows.Forms.AnchorStyles.Right), System.Windows.Forms.AnchorStyles)
            Me.btnOk.Location = New System.Drawing.Point(197, 230)
            Me.btnOk.Name = &quot;btnOk&quot;
            Me.btnOk.Size = New System.Drawing.Size(75, 23)
            Me.btnOk.TabIndex = 1
            Me.btnOk.Text = &quot;Ok&quot;
            Me.btnOk.UseVisualStyleBackColor = True
            Me.btnCancel.Anchor = CType((System.Windows.Forms.AnchorStyles.Bottom Or System.Windows.Forms.AnchorStyles.Right), System.Windows.Forms.AnchorStyles)
            Me.btnCancel.DialogResult = System.Windows.Forms.DialogResult.Cancel
            Me.btnCancel.Location = New System.Drawing.Point(116, 231)
            Me.btnCancel.Name = &quot;btnCancel&quot;
            Me.btnCancel.Size = New System.Drawing.Size(75, 23)
            Me.btnCancel.TabIndex = 2
            Me.btnCancel.Text = &quot;Cancel&quot;
            Me.btnCancel.UseVisualStyleBackColor = True
            Me.lstProjects.Anchor = CType((((System.Windows.Forms.AnchorStyles.Top Or System.Windows.Forms.AnchorStyles.Bottom) _
                        Or System.Windows.Forms.AnchorStyles.Left) _
                        Or System.Windows.Forms.AnchorStyles.Right), System.Windows.Forms.AnchorStyles)
            Me.lstProjects.FormattingEnabled = True
            Me.lstProjects.Location = New System.Drawing.Point(13, 13)
            Me.lstProjects.Name = &quot;lstProjects&quot;
            Me.lstProjects.SelectionMode = System.Windows.Forms.SelectionMode.MultiExtended
            Me.lstProjects.Size = New System.Drawing.Size(259, 212)
            Me.lstProjects.TabIndex = 3
            Me.AcceptButton = Me.btnOk
            Me.AutoScaleDimensions = New System.Drawing.SizeF(6.0!, 13.0!)
            Me.AutoScaleMode = System.Windows.Forms.AutoScaleMode.Font
            Me.CancelButton = Me.btnCancel
            Me.ClientSize = New System.Drawing.Size(284, 262)
            Me.Controls.Add(Me.lstProjects)
            Me.Controls.Add(Me.btnCancel)
            Me.Controls.Add(Me.btnOk)
            Me.FormBorderStyle = System.Windows.Forms.FormBorderStyle.SizableToolWindow
            Me.Name = &quot;Form1&quot;
            Me.StartPosition = System.Windows.Forms.FormStartPosition.CenterParent
            Me.Text = &quot;Select Project(s)&quot;
            Me.ResumeLayout(False)
        End Sub
        Friend WithEvents lstProjects As System.Windows.Forms.ListBox
        Friend WithEvents btnOk As System.Windows.Forms.Button
        Friend WithEvents btnCancel As System.Windows.Forms.Button
        Private Sub lstProjects_DoubleClick(ByVal sender As Object, ByVal e As System.EventArgs) Handles lstProjects.DoubleClick
            Me.DialogResult = System.Windows.Forms.DialogResult.OK
        End Sub
        Private Sub lstProjects_KeyUp(ByVal sender As Object, ByVal e As System.Windows.Forms.KeyEventArgs) Handles lstProjects.KeyUp
            If e.KeyCode = Keys.Enter Then Me.DialogResult = System.Windows.Forms.DialogResult.OK
        End Sub
        Private Sub btnOk_Click(ByVal sender As Object, ByVal e As System.EventArgs) Handles btnOk.Click
            Me.DialogResult = System.Windows.Forms.DialogResult.OK
        End Sub
    End Class
End Module
{% endhighlight %}

For those who would like to add the macro (or any macro) to your right click context menu follow these steps:
- Right Click on Menu Bar
- Choose Customize at bottom
- In the list choose &quot;Context Menus&quot; - Choose the Commands tab ![image](/posts_images/customize_toolbar.png)
- In the categories choose Macros and then locate your new macro ![image](/posts_images/command_macro.png)
- Drag it up to the context menu bar that appeared when you selected it above, hover over Project and Solution Context Menus, Project and then Drop it under Add Reference. ![image](/posts_images/customize_context_menu.png)
- Give it a friendly name. This will make it available when you right click on a project file in solution explorer&#160; ![image](/posts_images/context_add_project_ref.png)
