# Data Encryption-In-Transit

Data in transit, or data in motion, is nothing but data actively moving from one location to another such as across the internet or through a private network.

## History
Several versions of SSL have been released after its advent in 1995 (SSL 2.0 by Netscape communications, SSL 1.0 was never released). Here is the list:

1. SSL 1.0, 2.0 and 3.0
2. TLS 1.0 (or SSL 3.1, released in 1999)
3. TLS 1.1 (or SSL 3.2, released in 2006)
4. TLS 1.2 (or SSL 3.3, released in 2008)

SSL was changed to TLS when it was handed over to IETF for standardizing the security protocol layer in 1999. After making few changes to SSL 3.0, IETF released TLS 1.0. TLS 1.0 is being used by several web servers and browsers till date. With the latest being TLS 1.2 released in 2008.

On Windows the support for SSL/TLS protocols is tied to the SCHANNEL component. So, if a specific OS version doesn’t support a SSL/TLS version, this means it remains unsupported. Taking this into consideration, kindly refer below table which depicts of what security protocols are supported on Windows OS:

Windows OS Version | SSL 2.0 | SSL 3.0 | TLS 1.0 | TLS 1.1 | TLS 1.2 
--- | --- | --- | --- |--- |--- 
Windows XP & Windows Server 2003 | Enabled | Enabled | Enabled | Not Supported | Not Supported 
Windows Vista | Enabled | Enabled | Default | Not Supported | Not Supported 
Windows Server 2008 | Enabled | Enabled | Default | [Disabled*](https://www.microsoft.com/security/blog/2017/07/20/tls-1-2-support-added-to-windows-server-2008/) | [Disabled*](https://www.microsoft.com/security/blog/2017/07/20/tls-1-2-support-added-to-windows-server-2008/)  
Windows 7 & Windows Server 2008 R2 | Enabled | Enabled | Default | [Disabled*](https://support.microsoft.com/en-us/help/3140245/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-wi) | [Disabled*](https://support.microsoft.com/en-us/help/3140245/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-wi)  
Windows 8 & Windows Server 2012 | Disabled | Enabled | Enabled | Enabled | Default
Windows 8.1 & Windows Server 2012 R2 | Disabled | Enabled | Enabled | Enabled | Default
Windows 10 | Disabled | Enabled | Enabled | Enabled | Default
Windows Server 2016 | Not Supported | Disabled | Enabled | Enabled | Default												

## Best Protocol for Data Encryption in Transit and Why?

It has been for several years, TLS 1.2 is the most current defined version of the security protocol for establishing encryption channels over computer networks. It established a host of new cryptographic options for communication. However, like some previous versions of the protocol, it also allowed older cryptographic techniques to be used, in order to support older computers. Unfortunately, that opened it up to vulnerabilities, as those older techniques have become more vulnerable as time has passed and computing power has become cheaper.

## Server-Side Changes 

###	Registry Changes for Enabling TLS 1.2 protocol at the OS Level:
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client]
"DisabledByDefault"=dword:00000000
"Enabled"=dword:00000001

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server]
"DisabledByDefault"=dword:00000000
"Enabled"=dword:00000001

#### Explanation of the subkey specified in the above mentioned registry hive.

Subkey | Description | Value |
--- | --- | --- |
Client | Controls the use of TLS 1.2 on the client | 1 (Enabled) |
Server | Controls the use of TLS 1.2 on the server | 1 (Enabled) |
DisabledByDefault | Flag to disable TLS 1.2 by default | 1 (Enabled) |

###	Registry Changes required at the SQL Server level:
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL12.MSSQLSERVER\MSSQLServer\SuperSocketNetLib]
"ForceEncryption"=dword:00000001

This registry change will make sure the connections are encrypted for both Windows & SQL logins.

#### Explanation of the subkey specified in the above mentioned registry hive.

Subkey | Description | Value |
--- | --- | --- |
ForceEncryption | All client connections to all services on the server are encrypted | 1 (Enabled) |

When enabling the Force Encryption to YES, individual SQL server instances will be encrypted. With this change, SQL Server will use self-signed certificate without CA.

#### The following configuration methods performed to verify the SQL connections to be encrypted are:

1.	Self-Signed Certificate Configuration:
When we set Force Encryption to YES, SQL Server will use self-signed certificate without CA.

2.	CA Certificate Configuration for SQL Server:
There are certain steps performed to configure SQL Server to use available CA Certificate and verified the communication between client and server are encrypted. 

## Transport Layer Security (TLS) Best practices with the .NET Framework:
To ensure .NET Framework applications remain secure, the TLS version should not be hardcoded. .NET Framework applications should use the TLS version the operating system (OS) supports.

This document targets developers who are:

1. Directly using the System.Net APIs (for example, System.Net.Http.HttpClient and System.Net.Security.SslStream).
2. Directly using WCF clients and services using the System.ServiceModel namespace.

#### Recommendations:
Target .NET Framework 4.7 or later versions on our apps. Target .NET Framework 4.7.1 or later versions on our WCF apps.
Do not specify the TLS version. Configure our code to let the OS decide on the TLS version.
Perform a thorough code audit to verify you're not specifying a TLS or SSL version.

### For TCP sockets networking
SslStream, using .NET Framework 4.7 and later versions, defaults to the OS choosing the best security protocol and version. To get the default OS best choice, if possible, don't use the method overloads of SslStream that take an explicit SslProtocols parameter. Otherwise, pass SslProtocols.None. We recommend that you don't use Default; setting SslProtocols.Default forces the use of SSL 3.0 /TLS 1.0 and prevents TLS 1.2.

### For WCF TCP transport using transport security with certificate credentials
WCF uses the same networking stack as the rest of the .NET Framework.
If you are targeting 4.7.1, WCF is configured to allow the OS to choose the best security protocol by default unless explicitly configured:

1. In our application configuration file, Or
2. In our application in the source code.

By default, .NET Framework 4.7 and later versions is configured to use TLS 1.2 and allows connections using TLS 1.1 or TLS 1.0. Configure WCF to allow the OS to choose the best security protocol by configuring our binding to use SslProtocols.None. This can be set on SslProtocols. SslProtocols.None can be accessed from Transport. NetTcpSecurity.Transport can be accessed from Security.
#### If we're using a custom binding:
Configure WCF to allow the OS to choose the best security protocol by setting SslProtocols to use SslProtocols.None.
Or configure the protocol used with the configuration path system.serviceModel/bindings/customBinding/binding/sslStreamSecurity:sslProtocols.
If we're not using a custom binding and we're setting our WCF binding using configuration, set the protocol used with the configuration path system.serviceModel/bindings/netTcpBinding/binding/security/transport:sslProtocols.

### For WCF Message Security with certificate credentials
.NET Framework 4.7 and later versions by default uses the protocol specified in the SecurityProtocol property. When the AppContextSwitch Switch.System.ServiceModel.DisableUsingServicePointManagerSecurityProtocols is set to true, WCF chooses the best protocol, up to TLS 1.0.

## If an app targets .NET Framework 4.7 or later versions
Followings are the sections by which we can audit our code to verify that we're not setting a specific TLS or SSL version:

### For .NET Framework 4.6 - 4.6.2 and not WCF
Set the DontEnableSystemDefaultTlsVersions AppContext switch to false. See Configuring security via AppContext switches.

### For WCF using .NET Framework 4.6 - 4.6.2 using TCP transport security with Certificate Credentials
We are required to install the latest OS patches and security updates (if applicable).
The WCF framework automatically chooses the highest protocol available up to TLS 1.2 unless you explicitly configure a protocol version. For more information, see the preceding section For WCF TCP transport using transport security with certificate credentials.

### For .NET Framework 3.5 - 4.5.2 and not WCF
It is recommended to upgrade the app to .NET Framework 4.7 or later versions. If we cannot upgrade, take the following steps.

Set the SchUseStrongCrypto and SystemDefaultTlsVersions registry keys to 1. See Configuring security via the Windows Registry. The .NET Framework version 3.5 supports the SchUseStrongCrypto flag only when an explicit TLS value is passed.

### For .NET Framework 3.5, we are required to install a hot patch so that TLS 1.2 can be specified by our program:

[KB3154518](https://support.microsoft.com/kb/3154518) | Reliability Rollup HR-1605 - Support for TLS System Default Versions included in the .NET Framework 3.5.1 on Windows 7 SP1 and Server 2008 R2 SP1 |
--- | --- |
[KB3154519](https://support.microsoft.com/kb/3154519) | Reliability Rollup HR-1605 - Support for TLS System Default Versions included in the .NET Framework 3.5 on Windows Server 2012 |
[KB3154520](https://support.microsoft.com/kb/3154520) | Reliability Rollup HR-1605 -Support for TLS System Default Versions included in the .NET Framework 3.5 on Windows 8.1 and Windows Server 2012 R2
[KB3156421](https://support.microsoft.com/kb/3156421) | 1605 Hotfix rollup 3154521 for the .NET Framework 4.5.2 and 4.5.1 on Windows

The .NET framework version 3.5.1 and earlier versions did not provide support for applications to use Transport Layer Security (TLS) System Default Versions as a cryptographic protocol. This update enables the use of TLS v1.2 in the .NET Framework 3.5.1. Once we enable the SystemDefaultTlsVersions .NET registry key, a different behavior occurs for each version of Windows, as shown in the following table:

Windows Version | SSL2 Client | SSL2 Server | SSL3 Client | SSL3 Server | TLS 1.0 Client | TLS 1.0 Server | TLS 1.1 Client | TLS 1.1 Server | TLS 1.2 Client | TLS 1.2 Server 
--- | --- | --- | --- |--- |--- | --- | --- | --- |--- |--- 
Windows Vista SP2 & Windows Server 2008 SP2 | Off | On | On | On | On | On | N/A | N/A | N/A | N/A |
Windows 7 SP1 & Windows Server 2008 R2 SP1 | Off | On | On | On | On | On | Off | Off | Off | Off |
Windows Server 2012 | Off | Off | On | On | On | On | On | On | On | On |
Windows 8.1 and Windows Server 2012 R2 | Off | Off | On | On | On | On | On | On | On | On |
Windows 10 | Off | Off | On | On | On | On | On | On | On | On |
Windows 10 (1511) | Off | Off | On | On | On | On | On | On | On | On |
Windows 10 (1607) and Windows Server 2016 | N/A | N/A | Off | Off | On | On | On | On | On | On |

### For WCF using .NET Framework 3.5 - 4.5.2 using TCP transport security with Certificate Credentials
These versions of the WCF framework are hardcoded to use values SSL 3.0 and TLS 1.0. These values cannot be changed. We are supposed to update and retarget to NET Framework 4.6 or later versions to use TLS 1.1 and 1.2.

### If an app targets .NET Framework 3.5
We must explicitly set a security protocol instead of letting .NET or the OS pick the security protocol, add SecurityProtocolTypeExtensions and SslProtocolsExtension enumerations to our code. SecurityProtocolTypeExtensions and SslProtocolsExtension include values for Tls12, Tls11, and the SystemDefault value. For more information, see Support for TLS System Default Versions included in .NET Framework 3.5 on Windows 8.1 and Windows Server 2012 R2.

### For .NET Framework 4.6 or later versions (Required to configure security via AppContext switches)
The AppContext switches described in this section are relevant if our app targets, or runs on, .NET Framework 4.6 or later versions. Whether by default, or by setting them explicitly, the switches should be false if possible. If you want to configure security via one or both switches, then don't specify a security protocol value in our code; doing so would override the switch(es).

The switches have the same effect whether you're doing HTTP networking (ServicePointManager) or TCP sockets networking (SslStream).

Switch.System.Net.DontEnableSchUseStrongCrypto
A value of false for Switch.System.Net.DontEnableSchUseStrongCrypto causes our app to use strong cryptography. A value of false for DontEnableSchUseStrongCrypto uses more secure network protocols (TLS 1.2, TLS 1.1, and TLS 1.0) and blocks protocols that are not secure. For more info, see The SCH_USE_STRONG_CRYPTO flag. A value of true disables strong cryptography for our app.

#### If an app targets .NET Framework 4.6 or later versions, this switch defaults to false. That's a secure default, which we recommend. If an app runs on .NET Framework 4.6, but targets an earlier version, the switch defaults to true. In that case, you should explicitly set it to false.

DontEnableSchUseStrongCrypto should only have a value of true if you need to connect to legacy services that don't support strong cryptography and can't be upgraded.
Switch.System.Net.DontEnableSystemDefaultTlsVersions
A value of false for Switch.System.Net.DontEnableSystemDefaultTlsVersions causes an app to allow the operating system to choose the protocol. A value of true causes an app to use protocols picked by the .NET Framework.

#### If an app targets .NET Framework 4.7 or later versions, this switch defaults to false. That's a secure default that we recommend. If an app runs on .NET Framework 4.7 or later versions, but targets an earlier version, the switch defaults to true. In that case, you should explicitly set it to false.

Switch.System.ServiceModel.DisableUsingServicePointManagerSecurityProtocols
A value of false for Switch.System.ServiceModel.DisableUsingServicePointManagerSecurityProtocols causes an application to use the value defined in ServicePointManager.SecurityProtocols for message security using certificate credentials. A value of true uses the highest protocol available, up to TLS1.0

#### For applications targeting .NET Framework 4.7 and later versions, this value defaults to false. For applications targeting .NET Framework 4.6.2 and earlier, this value defaults to true.

Switch.System.ServiceModel.DontEnableSystemDefaultTlsVersions
A value of false for Switch.System.ServiceModel.DontEnableSystemDefaultTlsVersions sets the default configuration to allow the operating system to choose the protocol. A value of true sets the default to the highest protocol available, up to TLS1.2.

#### For applications targeting .NET Framework 4.7.1 and later versions, this value defaults to false. For applications targeting .NET Framework 4.7 and earlier, this value defaults to true.

## Testing with TLS 1.2+

Following the fixes recommended in the section above, products should be regression-tested for protocol negotiation errors and compatibility with other operating systems.  

1. The most common issue in this regression testing will be a TLS negotiation failure due to a client connection attempt from an operating system or browser that does not support TLS 1.2.  For example, a Vista client will fail to negotiate TLS with a server configured for TLS 1.2+ as Vista’s maximum supported TLS version is 1.0.  That client should be either upgraded or decommissioned in a TLS 1.2+ environment.
2. Products using certificate-based Mutual TLS authentication may require additional regression testing as the certificate-selection code associated with TLS 1.0 was less expressive than that for TLS 1.2.  
If a product negotiates MTLS with a certificate from a non-standard location (outside of the standard named certificate stores in Windows), then that code may need updating to ensure the certificate is acquired correctly.
3. Service interdependencies should be reviewed for trouble spots.  
Any services which interoperate with 3rd-party services should conduct additional interop testing with those 3rd parties.

[Supported Reference Link](https://docs.microsoft.com/en-us/dotnet/framework/network-programming/tls)
