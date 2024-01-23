# Remove-DuplicateItems

## Fork Details

### Changes 

This script works perfectly, except for the case where your duplicate messages are different item classes, such as when utilizing Metalogix Archive Manager for Exchange. MAM is bad at always removing the stubs when restoring emails to M365, and is unable to see/utilize M365's In-Place Archive feature. This means that if your stubs are moved to the In-Place archive, it's likely that your restored documents will also be when the ManagedFolderAssistant next runs, leaving users with two copies of each message, one of which  is a stub and will break if you delete the copy on the Metalogix server.

This script adds an additional Mode option called 'Stubs.' This mode only uses the InternetMessageId for building hashes. Additionally, it sorts by ItemClass instead of Date, so that IPM.Notes are first. Finally, it will not add any message with a type of IPM.Note.PamMessages to the global unique list, meaning that pagination of search results cannot cause restored messages to be deleted. Report mode is now prepended for three seperate cases:

1. If ItemClass *is not* IPM.Note.PamMessage and *is the first occurance* of that MessageID, "UNIQ" (Not Deleted)
2. If ItemClass *is* IPM.Note.PamMessage and *is the first occurance* of that MessageID, "STUB" (Not Deleted)
3. If this is *not the first occurance* of that MessageID, "DUPL" (Deleted)

### How To Use
You really have to wade through the comments to get it working in M365, so here's a brief setup process:

1. Create an Entra App Registration with the following permission: Office 365 Exchange Online: Application: full_access_as_app
2. Create a client secret and document it, along with the Application ID and the Tenant ID.

Run the following in PowerShell to do a dry-run (no-deletions) on the the In-Place Archive only:

```
$SECRET = ConvertTo-SecureString "SECRET" -AsPlainText -Force
$USER="user@domain.com"
$TENANTID="00000000-0000-0000-0000-000000000000"
$CLIENTID="00000000-0000-0000-0000-000000000000"

.\Remove-DuplicateItems.ps1 -report -Mailbox "$USER" -whatif:$true -Server outlook.office365.com -Verbose -Type Mail -TenantId $TENANTID -ClientId $CLIENTID -Secret $SECRET -archiveonly -mode Stubs -nosize -cleanupmode folder -excludefolders "Deleted Items" *>&1 |Tee-Object -filepath log.txt
```

- If you want to run it on the whole mailbox, remove the "-archiveonly" flag.

I suggest picking a smaller folder you know has duplicates and going over the logs to understand what will happen. When you're ready to delete stuff, set "-whatif:$false"

## Getting Started

This script will scan each folder of a given primary mailbox and personal archive (when
configured, Exchange 2010 and later) and removes duplicate items per folder. You can specify
how the items should be deleted and what items to process, e.g. mail items or appointments.
Sample scenarios are misbehaving 3rd party synchronization tool creates duplicate items or
(accidental) import of PST file with duplicate items. 

### About

For more information on this script, as well as usage and examples, see
the related blog article, [Removing Duplicate Items from a Mailbox](http://eightwone.com/2013/06/21/removing-duplicate-items-from-a-mailbox/).

## License

This project is licensed under the MIT License - see the LICENSE.md for details.

 
