---
layout: post
title:  "Save and Change Tool Layout in Visual Studio"
comments: true
tags: [Productivity,Visual Studio,Macros,Layout]
---


There have been many people who have asked for a way to save window positions within Visual Studio. For example they layout all their tool windows in a particular order and wish to change it based on the task at hand, but want to get back to the original setup.

Previously when asked, I recommended an old [add-in](http://vswindowmanager.codeplex.com/) and always suggested they update the code for 2008. I got tired of that recommendation, I needed the ability, and I needed it in 2010.So I wrote the following macro to allow a developer to Save, Switch, and Delete window/tool positions within Visual Studio.

The Macros names are pretty self-describing but here is a screen shot of it in action:

![image](/posts_images/view_switcher.png)

As always if you want to bind it to a keyboard shortcut go to Tools-->Options-->Environment—>Keyboard. Then in the 'show commands containing' box type one of the following: SwitchCurrentView, SaveCurrentView, DeleteSavedView and bind to a shortcut key. (Try - CTRL+ALT+0)

Tested on VS2008 and VS2010 should work in VS2005

Create a new macro module called 'View' and paste in the following: (You may need to add a reference to System.Drawing)

{% highlight vbnet %}
Imports System
Imports EnvDTE
Imports System.Collections.Generic
Imports System.Windows.FormsPublic
Module View
    Sub SaveCurrentView()
        Dim viewName As String
        viewName = InputBox("Name your view:", "Save view layout")
        If Not String.IsNullOrEmpty(viewName) Then
            DTE.WindowConfigurations.Add(viewName)
        End If
    End Sub

    Sub SwitchCurrentView()
        Using frm As New frmViewSwitcher
            Dim winptr As WinWrapper = New WinWrapper
            Dim ret As DialogResult
            Try
                ret = frm.ShowDialog(winptr)
                If ret = DialogResult.Cancel Then
                    Exit Sub
                End If
                DTE.WindowConfigurations.Item(frm.SelectedView).Apply()
            Catch ex As Exception
                MsgBox(ex.Message)
            Finally
                ret = Nothing
                winptr = Nothing
            End Try
        End Using
    End Sub

    Sub DeleteSavedView()
        Using frm As New frmViewSwitcher
            frm.Name = "Delete View"
            Dim winptr As WinWrapper = New WinWrapper
            Dim ret As DialogResult
            Try
                ret = frm.ShowDialog(winptr)
                If ret = DialogResult.Cancel Then
                    Exit Sub
                End If
                Dim currentView As String = DTE.WindowConfigurations.ActiveConfigurationName
                Dim selectedView As String = frm.SelectedView
                If Not String.Compare(currentView, selectedView, True) = 0 Then
                    Dim conf As WindowConfiguration = DTE.WindowConfigurations.Item(frm.SelectedView)
                    conf.Delete()
                Else
                    MsgBox("Cannot delete current view!", MsgBoxStyle.Critical + MsgBoxStyle.OkOnly, "Current View")
                End If
            Catch ex As Exception
                MsgBox(ex.Message)
            Finally
                ret = Nothing
                winptr = Nothing
            End Try
        End Using
    End Sub
End Module

Partial Class frmViewSwitcher
    Inherits System.Windows.Forms.Form
    Implements IDisposable
    Private _viewList As List(Of String)
    Public Sub New()
        InitializeComponent()
        _viewList = GetViews()
        _viewList.Sort()
        For Each v As String In _viewList
            lstResults.Items.Add(v)
        Next
        lstResults.Items(0).Selected = True
        Me.Focus()
    End Sub
    WriteOnly Property Name() As String
        Set(ByVal value As String)
            Me.Text = value
        End Set
    End Property
    Private Function GetViews() As List(Of String)
        Dim rtnList As New List(Of String)
        For Each wc As WindowConfiguration In DTE.WindowConfigurations
            Dim viewName As String = wc.Name
            rtnList.Add(viewName)
        Next
        Return rtnList
    End Function
    Public ReadOnly Property SelectedView() As String
        Get
            Return lstResults.SelectedItems(0).Text
        End Get
    End Property
    Protected Overrides Sub Dispose(ByVal disposing As Boolean)
        Try
            If disposing AndAlso components IsNot Nothing Then
                components.Dispose()
            End If
        Finally
            MyBase.Dispose(disposing)
        End Try
    End Sub
    Private Const ControlWidth As Int16 = 400
    Private Const ListHeight As Int16 = 200
    Private Const Padding As Int16 = 5
    Private Const WindowHeight As Int16 = Padding + ListHeight + Padding
    Private Const WindowWidth As Int16 = Padding + ControlWidth + Padding
    Private components As System.ComponentModel.IContainer
    Private Sub InitializeComponent()
        Me.lstResults = New System.Windows.Forms.ListView
        Me.colViewName = New System.Windows.Forms.ColumnHeader
        Me.colViewName.Text = "View Name"
        Me.button = New System.Windows.Forms.Button
        Me.SuspendLayout()
        Me.lstResults.Anchor = CType((((System.Windows.Forms.AnchorStyles.Top Or System.Windows.Forms.AnchorStyles.Bottom) _
                    Or System.Windows.Forms.AnchorStyles.Left) _
                    Or System.Windows.Forms.AnchorStyles.Right), System.Windows.Forms.AnchorStyles)
        Me.lstResults.Columns.AddRange(New System.Windows.Forms.ColumnHeader() {Me.colViewName})
        Me.lstResults.FullRowSelect = True
        Me.lstResults.HideSelection = False
        Me.lstResults.Location = New System.Drawing.Point(Padding, Padding)
        Me.lstResults.MultiSelect = False
        Me.lstResults.Name = "lstResults"
        Me.lstResults.Size = New System.Drawing.Size(ControlWidth, ListHeight)
        Me.lstResults.TabIndex = 1
        Me.lstResults.UseCompatibleStateImageBehavior = False
        Me.lstResults.View = System.Windows.Forms.View.Details
        Me.colViewName.Width = CInt(lstResults.Width - 10)
        Me.button.DialogResult = System.Windows.Forms.DialogResult.Cancel
        Me.button.Size = New System.Drawing.Size(1, 1)
        Me.AllowDrop = False
        Me.AutoScaleDimensions = New System.Drawing.SizeF(6.0!, 13.0!)
        Me.AutoScaleMode = System.Windows.Forms.AutoScaleMode.Font
        Me.ClientSize = New System.Drawing.Size(WindowWidth, WindowHeight)
        Me.Controls.Add(Me.lstResults)
        Me.Controls.Add(Me.button)
        Me.CancelButton = Me.button
        Me.MaximizeBox = False
        Me.MinimizeBox = False
        Me.Name = "View Switcher"
        Me.ShowIcon = False
        Me.ShowInTaskbar = False
        Me.StartPosition = System.Windows.Forms.FormStartPosition.CenterScreen
        Me.Text = "View Switcher"
        Me.ResumeLayout(False)
        Me.PerformLayout()
    End Sub

    Friend WithEvents lstResults As System.Windows.Forms.ListView
    Friend WithEvents colViewName As System.Windows.Forms.ColumnHeader
    Friend WithEvents button As System.Windows.Forms.Button
    Private Sub lstResults_DoubleClick(ByVal sender As Object, ByVal e As System.EventArgs) Handles lstResults.DoubleClick
        Me.DialogResult = System.Windows.Forms.DialogResult.OK
    End Sub
    Private Sub lstResults_KeyUp(ByVal sender As Object, ByVal e As System.Windows.Forms.KeyEventArgs) Handles lstResults.KeyUp
        If e.KeyCode = Keys.Enter Then Me.DialogResult = System.Windows.Forms.DialogResult.OK
    End Sub
End Class

Public Class WinWrapper
    Implements System.Windows.Forms.IWin32Window
    Overridable ReadOnly Property Handle() As System.IntPtr Implements System.Windows.Forms.IWin32Window.Handle
        Get
            Dim iptr As New System.IntPtr(DTE.MainWindow.HWnd)
            Return iptr
        End Get
    End Property
End Class
{% endhighlight %}
