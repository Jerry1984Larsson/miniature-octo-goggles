# How to Create a Self-Signed Certificate with PowerShell

![Powershell logo](https://www.howtogeek.com/wp-content/uploads/csit/2020/03/23e4a5a4.png?width=1198&trim=1,1&bg-color=000&pad=1,1)

[Self-signed certificates](https://en.wikipedia.org/wiki/Self-signed_certificate) are an easy way to perform testing and other less important tasks. Self-signed certificates do not have a trusted chain of certificates backing them up and are signed by the user who created it. If you trust the entity that signed the certificate then you can use it just as you would a properly validated one.

If you need to create a self-signed certificate, one way you can do so is with PowerShell. In this article, you’re going to learn how to create a self-signed certificate in PowerShell.

## Creating a Self-Signed Certificate

To create a self-signed certificate with PowerShell, you can use the `New-SelfSignedCertificate` cmdlet. This cmdlet is included in the `PKI` module.

There are many options when it comes to creating certificates. Common self-signed certificate types are `SSLServerAuthentication` (default for the cmdlet) and `CodeSigning`. Also, you can create a `DocumentEncryptionCert`, which is very useful for encrypting files, and finally a `Custom` certificate that lets you specify many custom options.

Let’s go ahead and create a regular `SSLServerAuthentication` certificate. This is one that usually is used to protect websites with SSL encryption. You can see an example of this below. In this example, the certificate is being stored in the `Cert:LocalMachineMy Certificate Store`.

```powershell
$Params = @{
    "DnsName"           = @("mywebsite.com","www.mywebsite.com")
    "CertStoreLocation" = "Cert:LocalMachineMy"
    "NotAfter"          = (Get-Date).AddMonths(6)
    "KeyAlgorithm"      = "RSA"
  "KeyLength"         = "2048"
}

PS C:> New-SelfSignedCertificate @Params

PSParentPath: Microsoft.PowerShell.SecurityCertificate::LocalMachineMy

Thumbprint                                Subject              EnhancedKeyUsageList
----------                                -------              --------------------
4EFF6B1A0F61B4BG692C77F09889BD151EE8BB58  CN=mywebsite.com     {Client Authentication, Server Authentication}
```

If all went well, you should now have a newly-created certificate! You will notice that the output returns the subject but the subject only displays the first item passed to it via the `DnsName` parameter. This is because the second URL becomes part of the subject alternate list.

*Note if you attempt to run this, not as an Administrator, you will get an error message such as below:

```powershell
New-SelfSignedCertificate: CertEnroll::CX509Enrollment::_CreateRequest: Access denied. 0x80090010 (-2146893808 NTE_PERM)
```

As you can tell with the `Access denied`, you do not yet have permission to run this.*

## Finding Information on our Certificate

Let’s make sure the certificate was created the way we expected. To find information on a particular certificate with PowerShell, you can use the `Get-ChildItem` cmdlet, just as you might list files in a directory.

```powershell
PS C:> Get-ChildItem -Path "Cert:LocalMachineMy" | Where-Object Thumbprint -EQ 4EFF6B1A0F61B4BF692C77F09889AD151EE8BB58 | Select-Object *

PSPath                   : Microsoft.PowerShell.SecurityCertificate::LocalMachineMy4EFF6B1A0F61B4BF692C77F09889AD151EE8BB58
                           58
PSParentPath             : Microsoft.PowerShell.SecurityCertificate::LocalMachineMy
PSChildName              : 4EFF6B1A0F61B4BF692C77F09889AD151EE8BB58
PSDrive                  : Cert
PSProvider               : Microsoft.PowerShell.SecurityCertificate
PSIsContainer            : False
EnhancedKeyUsageList     : {Client Authentication (1.3.6.1.5.5.7.3.2), Server Authentication (1.3.6.1.5.5.7.3.1)}
DnsNameList              : {mywebsite.com, www.mywebsite.com}
SendAsTrustedIssuer      : False
EnrollmentPolicyEndPoint : Microsoft.CertificateServices.Commands.EnrollmentEndPointProperty
EnrollmentServerEndPoint : Microsoft.CertificateServices.Commands.EnrollmentEndPointProperty
PolicyId                 :
Archived                 : False
Extensions               : {System.Security.Cryptography.Oid, System.Security.Cryptography.Oid,
                           System.Security.Cryptography.Oid, System.Security.Cryptography.Oid}
FriendlyName             :
HasPrivateKey            : True
PrivateKey               : System.Security.Cryptography.RSACng
IssuerName               : System.Security.Cryptography.X509Certificates.X500DistinguishedName
NotAfter                 : 6/22/2020 11:50:15 AM
NotBefore                : 12/22/2019 10:40:20 AM
PublicKey                : System.Security.Cryptography.X509Certificates.PublicKey
RawData                  : {48, 130, 3, 55...}
SerialNumber             : 608C4D5E6B8D41B44ADDC6BD725FE264
SignatureAlgorithm       : System.Security.Cryptography.Oid
SubjectName              : System.Security.Cryptography.X509Certificates.X500DistinguishedName
Thumbprint               : 4EFF6B1A0F61B4BF692C77F09889AD151EE8BB58
Version                  : 3
Handle                   : 2628421609632
Issuer                   : CN=mywebsite.com
Subject                  : CN=mywebsite.com
```

There is a lot of great information here, but you may notice in the `DnsNameList` that both of the sites are now shown. In addition, the `NotAfter` date is correctly populated to be 6 months from the date of creation.

## Code Signing Certificate

If you work in PowerShell, you will know about [execution policies](https://adamtheautomator.com/set-executionpolicy/). If you have an execution policy set to `AllSigned` then you would need to sign each script that runs on your system. To create a certificate to do this, it’s pretty simple!

```powershell
PS C:> New-SelfSignedCertificate -Type 'CodeSigningCert' -DnsName 'MyHost'

PSParentPath: Microsoft.PowerShell.SecurityCertificate::LocalMachineMY

Thumbprint                                Subject              EnhancedKeyUsageList
----------                                -------              --------------------
14D535EG834370293BA103159EB00876A79959D8  CN=MyHost            Code Signing
```

## Document Protection Certificate

You may not have encountered this much before, but PowerShell, with the Data Protection API, can encrypt files on your system using a Document Protection Certificate. Using the `New-SelfSignedCertificate` cmdlet, we can easily make a certificate to encrypt your documents.

```powershell
$Params = @{
    "DnsName"           = "MyHost"
    "CertStoreLocation" = "Cert:CurrentUserMy"
    "KeyUsage"          = "KeyEncipherment","DataEncipherment","KeyAgreement"
    "Type"              = "DocumentEncryptionCert"
}

PS C:> New-SelfSignedCertificate @Params

Thumbprint                                Subject              EnhancedKeyUsageList
----------                                -------              --------------------
14D535EG934370293BA203159EB00876A79959D8  CN=MyHost            Document Encryption
```

With this type of certificate, you can now use the certificate created to encrypt and decrypt content using PowerShell commands like `Protect-CMSMessage` and `UnProtect-CMSMessage`.

Encrypting/decrypting content like this becomes useful if you need to pass the encrypted data around since you can then use this certificate on another system to decrypt the data. If you rely on the standard Data Protection API (DPAPI) built into Windows, then you would not be able to decrypt the data on other systems or for other users.

## Summary

PowerShell makes creating self-signed certificates incredibly easy to do. These certificates have a myriad of uses, but an important note to remember is that they should only be used in testing. You won’t have a valid certificate trust chain to validate your self-signed certificates.

Seeing how quick and easy it is to create self-signed certificates are, you can start doing this today and properly encrypting any connections or data that you need to!