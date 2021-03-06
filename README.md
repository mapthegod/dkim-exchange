Exchange DKIM Signer
=============

DKIM Signing Agent for Exchange Server.

This code is mainly based on the work of http://nicholas.piasecki.name/blog/2010/12/dkim-signing-outbound-messages-in-exchange-server-2007/

WARNING: Please read the 'Known Bugs' section before you continue!

If you have a version installed previous to 26.11.2013 read the Section [Update from Version 0.5](#update-from-version-05)

## Supported versions

The .dll is compiled for .NET 3.5 (Exchange 2007 and 2010) or .NET 4 (Exchange 2012)

### Exchange 2013

This Receive Agent is fully tested under Exchange 2013

### Exchange 2010

This Receive Agent is fully tested under Exchange 2010 SP3 with Windows Server 2008 R2.

There's are also compiled versions for no Service Pack, SP1 and SP2 installed.

### Exchange 2007

Exchange 2007 SP3 .dll is build and can be found in the release directory. Please check if those are working for you and send me a short notice.

## Installing the Transport Agent

1. Download the .zip and extract it e.g. on the Desktop: [Exchange DkimSigner Master.zip](https://github.com/Pro/dkim-exchange/archive/master.zip)
2. Open "Exchange Management Shell" from the Startmenu
3. Execute the following command to allow execution of local scripts (will be reset at last step): `Set-ExecutionPolicy Unrestricted`
4. Cd into the folder where the zip has been extracted.
5. Execute the install script `.\install.ps1`
6. Follow the instructions. For the configuration see next section.
7. Reset the execution policy: `Set-ExecutionPolicy Restricted`
8. Check EventLog for errors or warnings.
 Hint: you can create a user defined view in EventLog and then select "Per Source" and as the value "Exchange DkimSigner"

Make sure that the priority of the DkimSigner Agent is quite low so that no other agent messes around with the headers. Best set it to lowest priority.
To get a list of all the Export Agents use the Command `Get-TransportAgent`

To change the priority use `Set-TransportAgent -Identity "Exchange DkimSigner" -Priority 3`

### Configuring the agent
Edit the .config file to fit your needs.

```xml
  <domainSection>
    <Domains>
      <Domain Domain="example.com" Selector="sel2012" PrivateKeyFile="keys/example.com.private"/>
      <Domain Domain="example.org" Selector="sel2013" PrivateKeyFile="keys/example.org.private"/>
    </Domains>
  </domainSection>
  <customSection>
    <general LogLevel="3" HeadersToSign="From; Subject; To; Date; Message-ID;" Algorithm="RsaSha1" />
  </customSection>
```

You can add as many domain items as you need. For each domain item, the domain, the selector and the path to the private key file is needed.
This path may be relative (based on the location of the .dll) or absolute.

#### Logging
The dkim signing agent logs by default all errors and warnings into EventLog.
You can set the LogLevel in the .config file:

Possible values:
* 0 = no logging
* 1 = Error only
* 2 = Warn+Error
* 3 = Info+Warn+Error

### Creating the keys

You can use the following service for creating public and private keys:
http://www.port25.com/support/domainkeysdkim-wizard/

Or if you have a linux installation, use (from the opendkim package):
    opendkim-genkey -D target_directory/ -d example.com -s sel2012

The keys can be in DER or PEM format (the format will be automatically detected).
	
### Testing the setup

If you want to test, if everything is working, simply send a mail to check-auth@verifier.port25.com and you will get an immediate response with the results of the DKIM check.

## Updating the Transport Agent

If you want to update the Exchange DKIM Transport Agent simply re-download the .zip file and follow the steps in the installation section.

### Update from Version 0.5

If you have a version installed previous to 26.11.2013 (i.e. 0.5) read the following instructions to update to version 1.5 or above:

1. Backup your folder `C:\Program Files\Exchange DKIM`
2. Then execute the following commands in Exchange Management Shell:
<pre>
Net Stop MSExchangeTransport 
Disable-TransportAgent -Identity "Exchange DKIM" 
Uninstall-TransportAgent -Identity "Exchange DKIM" 
</pre>
3. Now delete the folder `C:\Program Files\Exchange DKIM` (keep the backup!!)
4. You will need some parts of your old config file in the new one. You also have to copy the keys back after installation (if in this folder).
5. Follow the instructions in the [Install Section](#installing-the-transport-agent).
6. When you copy the `<domain ...` parts from the old config, please change the tag to upper case: `<Domain ...`
7. That's it!

## Uninstalling the Transport Agent

Follow the install instructions but execute `.\uninstall.ps1` instead.

## Known bugs

* [unconfirmed] When using internal Receive Connectors as Relay, unicode characters may break and will be replaced with '?'. See: Issue #2

## Notes for developers

### Required DLLs for developing

It isn't allowed to distribute the .dll required for development of this transport agent.
http://blogs.msdn.com/b/webdav_101/archive/2009/04/02/don-t-redistribute-product-dlls-unless-you-know-its-safe-and-legal-to-do-so.aspx

Therefore you have to copy all files from 
<pre>
C:\Program Files\Microsoft\Exchange Server\V14\Public
Microsoft.Exchange.Data.Common.dll
Microsoft.Exchange.Data.Common.xml
Microsoft.Exchange.Data.Transport.dll
Microsoft.Exchange.Data.Transport.xml
</pre>
into the corresponding subdirectory from the Lib directory of this project.

#### Debugging
If you want to debug the .dll on your Exchange Server, you need to install [Visual Studio Remote Debugging](http://msdn.microsoft.com/en-us/library/vstudio/bt727f1t.aspx) on the Server.

1. After the Remote Debugging Tools are installed on the Server, open Visual Studio
2. Compile the .dll with Debug information
3. Copy the recompiled .dll to the server
4. In Visual Studio select Debug->Attach to Process
5. Under 'Qualifier' input the server IP oder Host Name
6. Select "Show processes from all users"
7. Select the process `EdgeTransport.exe` and then press 'Attach'
8. When reached, the process should stop at the breakpoint

## Changelog

* 27.11.2013 [1.5.1]:
	Added support for Exchange 2013
* 26.11.2013 [1.5]:
	Changed configuration file for better reading
	Added compiled files for Exchange 2010 SP1&SP2
	Added install and uninstall script
* 08.11.2013 [0.5]:
    Changed build structure to do a batch build for different Exchange versions.
	Build for Exchange 2010 and Exchange 2007.
* 24.02.2013:
	Added multi domain support

## TODO

* Allow use of relaxed or any combination between relaxed and simple
