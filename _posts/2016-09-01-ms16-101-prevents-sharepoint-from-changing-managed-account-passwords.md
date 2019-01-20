---
layout: post
title: MS16-101 Prevents SharePoint From Changing Managed Account Passwords
tags: [sp2010,sp2013,sp2016]
---

[MS16-101](https://technet.microsoft.com/library/security/MS16-101) is a security update that prevents protocol fallback (Kerberos to NTLM) from taking place to change account passwords.

> This security update disables the ability of the Negotiate process to fall back to NTLM when Kerberos authentication fails for password change operations.
>
> Currently, the ability to change the passwords of disabled or locked-out accounts is supported only by NTLM. It is not supported by the Kerberos protocol. This security update prevents the Negotiate process from falling back to NTLM for password change operations when Kerberos authentication fails. Therefore, you will no longer be able to change the password for disabled or locked-out accounts after you install this security update. It is not secure to change disabled or locked-out user account passwords by using NTLM. This is why the ability of Negotiate to fall back to NTLM is disabled by this security update.

It is no longer possible to use the option 'Set account password to a new value' or 'Generate new password' for a Managed Account in SharePoint 2013 or 2016; SharePoint 2010 is also likely impacted, although I've not seen any reports, but the same API is used to change passwords. If you have MS16-101 installed on your Domain Controller(s) and/or SharePoint server(s), when you attempt to set a Managed Account's password or have SharePoint generate a random password, you will see the following error in Central Administration:

![SetPasswordError](/assets/images/2016/09/SetPasswordError.png)

In the ULS log, you'll see a similar error:

```csharp
Getting Error Message for Exception System.Web.HttpUnhandledException (0x80004005): 
Exception of type 'System.Web.HttpUnhandledException' was thrown. ---> System.ComponentModel.Win32Exception (0x80004005): 
The system cannot contact a domain controller to service the authentication request. Please try again later at 
Microsoft.SharePoint.Win32.SPNetApi32.NetUserChangePassword(String loginName, SecureString oldPassword, SecureString password) 
at Microsoft.SharePoint.Administration.SPManagedAccount.AttemptPasswordChange(SecureString possibleCurrentPassword, SecureString newPassword)
at Microsoft.SharePoint.Administration.SPManagedAccount.ChangePassword(SecureString newPassword, EventProcessingOptions eventFlags)
at Microsoft.SharePoint.ApplicationPages.ManageAccountPage.BtnSubmit_Click(Object sender, EventArgs args) 
at System.Web.UI.WebControls.Button.OnClick(EventArgs e)
at System.Web.UI.WebControls.Button.RaisePostBackEvent(String eventArgument)
at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint) 
at System.Web.UI.Page.HandleError(Exception e) 
at System.Web.UI.Page.ProcessRequestMain(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint) 
at System.Web.UI.Page.ProcessRequest(Boolean includeStagesBeforeAsyncPoint, Boolean includeStagesAfterAsyncPoint) 
at System.Web.UI.Page.ProcessRequest()
at System.Web.UI.Page.ProcessRequest(HttpContext context)
at System.Web.HttpApplication.CallHandlerExecutionStep.System.Web.HttpApplication.IExecutionStep.Execute()
at System.Web.HttpApplication.ExecuteStep(IExecutionStep step, Boolean& completedSynchronously
```

The underlying real error is 1265, or [ERROR_DOWNGRADE_DETECTED](https://msdn.microsoft.com/en-us/library/cc231199.aspx). Nothing too useful appears in network traces on the SharePoint server or Domain Controller, but based on the KB articles note, it is clear that SharePoint is using NTLM to attempt to change the password in Active Directory.

There are one of two solutions:

If you rely on automatic password change, or specifying a password to have SharePoint change it in Active Directory, uninstall [KB3177108](https://support.microsoft.com/en-us/kb/3177108) (Windows Server 2012/Windows Server 2012 R2) and [KB3167679](https://support.microsoft.com/en-us/kb/3167679) (Windows Server 2008 and Windows Server 2012 R2) from your Domain Controllers and SharePoint servers.

If you reset passwords in Active Directory (ADUC, PowerShell, etc.), and use the 'Use existing password' option, no changes need to be made as SharePoint only contacts the Domain Controller to validate the password.