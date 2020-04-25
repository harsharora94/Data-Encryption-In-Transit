# Encryption-In-Transit

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
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2]

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client]
"DisabledByDefault"=dword:00000000
"Enabled"=dword:00000001

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server]
"DisabledByDefault"=dword:00000000
"Enabled"=dword:00000001

#### Explanation of the subkey specified in the above mentioned registry hive.

Subkey | Description | Value 
--- | --- | --- | --- |--- |--- 
Client | Controls the use of TLS 1.2 on the client | 1 (Enabled)
Server | Controls the use of TLS 1.2 on the server | 1 (Enabled)
DisabledByDefault | Flag to disable TLS 1.2 by default | 1 (Enabled)

###	Registry Changes required at the SQL Server level:

The following configuration methods performed to verify the SQL connections to be encrypted are:
1.	Self-Signed Certificate Configuration
2.	CA Certificate Configuration for SQL Server 

1.	Self-Signed Certificate Configuration:

The following change is introduced to make sure the connections are encrypted for both Windows & SQL logins.

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL12.MSSQLSERVER\MSSQLServer\SuperSocketNetLib]
"ForceEncryption"=dword:00000001

#### Explanation of the subkey specified in the above mentioned registry hive.

Subkey | Description | Value 
--- | --- | --- | --- |--- |--- 
ForceEncryption | All client connections to all services on the server are encrypted | 1 (Enabled)

When enabling the Force Encryption to YES, individual SQL server instances will be encrypted. With this change, SQL Server will use self-signed certificate without CA.

2.	CA Certificate Configuration for SQL Server:

There are certain steps performed to configure SQL Server to use available Sensis certificate and verified the communication between client and server are encrypted. 

## Testing with TLS 1.2+

Following the fixes recommended in the section above, products should be regression-tested for protocol negotiation errors and compatibility with other operating systems in your enterprise.  

1. The most common issue in this regression testing will be a TLS negotiation failure due to a client connection attempt from an operating system or browser that does not support TLS 1.2.  For example, a Vista client will fail to negotiate TLS with a server configured for TLS 1.2+ as Vista’s maximum supported TLS version is 1.0.  That client should be either upgraded or decommissioned in a TLS 1.2+ environment.
2. Products using certificate-based Mutual TLS authentication may require additional regression testing as the certificate-selection code associated with TLS 1.0 was less expressive than that for TLS 1.2.  
If a product negotiates MTLS with a certificate from a non-standard location (outside of the standard named certificate stores in Windows), then that code may need updating to ensure the certificate is acquired correctly.
3. Service interdependencies should be reviewed for trouble spots.  
Any services which interoperate with 3rd-party services should conduct additional interop testing with those 3rd parties.