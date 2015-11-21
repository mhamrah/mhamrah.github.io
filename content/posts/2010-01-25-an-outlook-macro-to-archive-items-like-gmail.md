---
title: An Outlook Macro to Archive Items like GMail
author: Michael
type: post
date: 2010-01-25
url: /2010/01/an-outlook-macro-to-archive-items-like-gmail/
dsq_thread_id:
  - 527224550
categories:
  - Programming
---
I&#8217;m a big fan of Gmail&#8217;s archive feature- I can move items out of the inbox quickly and find them later with search.  To mimic this behavior at work with outlook, I&#8217;ve created this quick and dirty macro which does the job.  Simply create a folder named &#8220;archive&#8221; in your mailbox.  When you run the macro, it will move the current item in your inbox to that folder.  I&#8217;ve created a keystroke which lets me run the macro easily.

<pre class="brush: vb; title: ; notranslate" title="">Sub ArchiveItem()

On Error Resume Next

 Dim objFolder As Outlook.MAPIFolder, objInbox As Outlook.MAPIFolder
 Dim objNS As Outlook.NameSpace, objItem As Outlook.MailItem

 Set objNS = Application.GetNamespace("MAPI")
 Set objInbox = objNS.GetDefaultFolder(olFolderInbox)
 Set objFolder = objInbox.Parent.Folders("Archive")

'Assume this is a mail folder
 If objFolder Is Nothing Then
 MsgBox "This folder doesn't exist!", vbOKOnly + vbExclamation, "INVALID FOLDER"
 End If

 If Application.ActiveExplorer.Selection.Count = 0 Then
 'Require that this procedure be called only when a message is selected
 Exit Sub
 End If

 For Each objItem In Application.ActiveExplorer.Selection
 If objFolder.DefaultItemType = olMailItem Then
 If objItem.Class = olMail Then
 objItem.Move objFolder
 End If
 End If
 Next

 Set objItem = Nothing
 Set objFolder = Nothing
 Set objInbox = Nothing
 Set objNS = Nothing
End Sub

</pre>
