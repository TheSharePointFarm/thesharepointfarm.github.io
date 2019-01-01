---
layout: post
title: SharePoint Creates Legacy Contacts in Active Directory
tags: [sp2010]
---

A few people have noticed that SharePoint is creating Legacy Contacts in Active Directory with Exchange 2007 or 2010.  The reason is fairly simple... SharePoint doesn’t set the attribute msExchVersion.

When SharePoint creates a contact, the only properties it sets can be found in SetContactProperties:

```csharp
private void SetContactProperties(DirectoryEntry contactEntry, string Alias, bool SetAlias, string FirstName, string LastName, string ForwardingEmail, ContactFlags Flags)
{
    contactEntry.Properties["givenName"].Value = FirstName;
    contactEntry.Properties["sn"].Value = LastName;
    contactEntry.Properties["displayName"].Value = FirstName + " " + LastName;
    contactEntry.Properties["TargetAddress"].Value = "SMTP:" + ForwardingEmail;
    contactEntry.Properties["MAPIRecipient"].Value = false;
    contactEntry.Properties["InternetEncoding"].Value = 0x140000;
    if (SetAlias)
    {
        contactEntry.Properties["mailNickname"].Value = Alias;
    }
    if (Flags == ContactFlags.OnlyAllowAuthenticatedEmail)
    {
        contactEntry.Properties["msExchRequireAuthToSendTo"].Value = true;
    }
}
```

By leaving msExchVersion blank, Exchange 2007 and 2010 interpret the contact to be Legacy, e.g. an Exchange 2003 contact.