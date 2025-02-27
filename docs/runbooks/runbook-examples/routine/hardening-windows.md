---
title: Hardening Windows
description: With Octopus Deploy you can harden your Windows machines with a runbook as part of a routine operations task.
position: 60
---

Highly regulated industries such as finance need to make sure the Operating Systems (OS) are secure and protected from data breaches.  This often requires strict controls be implemented to reduce their attack surface.  In Windows Active Directory environments, this type of hardening can be performed by implementing Group Policy Objects (GPO).  In cases where GPO isn't an option, such as non-domain joined cloud servers, you could use a runbook to harden your Windows installation.

!include <security-disclaimer>

## Create the runbook

To create a runbook to harden your Windows installation:

1. From your project's overview page, navigate to **{{Operations, Runbooks}}**, and click **ADD RUNBOOK**.
1. Give the runbook a Name and click **SAVE**.
1. Click **DEFINE YOUR RUNBOOK PROCESS**, and then click **ADD STEP**.
1. Click **Script**, and then select the **Run a Script** step.
1. Give the step a name.
1. Choose the **Execution Location** on which to run this step.
1. In the **Inline source code** section, select **PowerShell** and add the following code:
:::warning
The following script is an example of what can be done, please review the script carefully before using.
:::

```PowerShell
Function Test-RegistryValue {
    param(
        [Alias("PSPath")]
        [Parameter(Position = 0, Mandatory = $true, ValueFromPipeline = $true, ValueFromPipelineByPropertyName = $true)]
        [String]$Path
        ,
        [Parameter(Position = 1, Mandatory = $true)]
        [String]$Name
    ) 

    process {
        Write-Verbose "Path:$($Path) Name:$($Name)"

        if (Test-Path $Path) {
            Write-Verbose "Path is here"
            $Key = Get-Item -LiteralPath $Path
            Write-Verbose "Key: $($Key)"
            if ($null -ne $Key.GetValue($Name, $null)) {
                    $true
            } else {
                $false
            }
        } else {
            Write-Verbose "NOT HERE"
            $false
        }
    }
}

$directory = "DIRECTORY"

$newItems = @(
       [pscustomobject]@{name='FEATURE_ENABLE_PRINT_INFO_DISCLOSURE_FIX';path='hklm:\SOFTWARE\WOW6432Node\Microsoft\Internet Explorer\Main\FeatureControl\';value=$directory}
       [pscustomobject]@{name="iexplore.exe";path='hklm:\SOFTWARE\WOW6432Node\Microsoft\Internet Explorer\Main\FeatureControl\FEATURE_ENABLE_PRINT_INFO_DISCLOSURE_FIX\';value='00000001'}
       [pscustomobject]@{name="FEATURE_ENABLE_PRINT_INFO_DISCLOSURE_FIX";path='hklm:\SOFTWARE\Microsoft\Internet Explorer\Main\FeatureControl\';value=$directory}
       [pscustomobject]@{name="iexplore.exe" ;path='hklm:\SOFTWARE\Microsoft\Internet Explorer\Main\FeatureControl\FEATURE_ENABLE_PRINT_INFO_DISCLOSURE_FIX\';value='00000001'}
       [pscustomobject]@{name="FEATURE_ALLOW_USER32_EXCEPTION_HANDLER_HARDENING";path='hklm:\SOFTWARE\Microsoft\Internet Explorer\Main\FeatureControl\';value=$directory}
       [pscustomobject]@{name="iexplore.exe";path='hklm:\SOFTWARE\Microsoft\Internet Explorer\Main\FeatureControl\FEATURE_ALLOW_USER32_EXCEPTION_HANDLER_HARDENING\';value='00000001'}
       [pscustomobject]@{name="FEATURE_ALLOW_USER32_EXCEPTION_HANDLER_HARDENING";path='hklm:\SOFTWARE\Wow6432Node\Microsoft\Internet Explorer\Main\FeatureControl\';value=$directory}
       [pscustomobject]@{name="iexplore.exe";path='hklm:\SOFTWARE\Wow6432Node\Microsoft\Internet Explorer\Main\FeatureControl\FEATURE_ALLOW_USER32_EXCEPTION_HANDLER_HARDENING\';value='00000001'}
       [pscustomobject]@{name="Virtualization";path='hklm:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\';value=$directory}
   )


Write-Verbose "Microsoft Internet Explorer Cumulative Security Update (MS15-124)" #####"
###Impact: A remote, unauthenticated attacker could exploit these vulnerabilities to conduct cross-site scripting attacks, elevate their privileges, execute arbitrary code or cause a denial of service condition on the targeted system ###

foreach ($newItem in $newItems) {
    $keyExists = Test-RegistryValue $newItem.path $newItem.name
    If ($keyExists -eq $false) {
        Write-Verbose "New item: $($newItem.path)$($newItem.name) value:$($newItem.value)"
        try {
            if ($newItem.value -eq $directory){
                New-Item -Name $newItem.name -Path $newItem.path -type Directory
            } else {
                New-Item -Name $newItem.name -Path $newItem.path -Value $newItem.value
            }   
        }
        catch {
            Write-Verbose "Error writing item, most likely it already exists. "
        }
    }
}

Write-Verbose "Enabled Cached Logon Credential"
### Impact : Unauthorized users can gain access to this cached information, thereby obtaining sensitive logon information 
Set-ItemProperty -Path 'hklm:\Software\Microsoft\Windows Nt\CurrentVersion\Winlogon' -Name "CachedLogonsCount" -Value "0"

Write-Verbose "Windows Update For Credentials Protection and Management (Microsoft Security Advisory 2871997)"
### Impact : If this vulnerability is successfully exploited, attackers can steal credentials of the system

try {
    New-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\Session Manager\' -Name "CWDIllegalInDllSearch" -Value "00000001" -PropertyType "DWord"   
}
catch {
    Write-Verbose "Error writing CWDIllegalInDllSearch, probably already exists"
}

Write-Verbose "Microsoft Windows Security Update Registry Key Configuration Missing (ADV180012) (Spectre/Meltdown Variant 4) "
###Impact : An attacker who has successfully exploited this vulnerability may be able to read privileged data across trust boundaries. Vulnerable code patterns in the operating system (OS) or in applications could allow an attacker to exploit this vulnerability. In the case of Just-in-Time (JIT) compilers, such as JavaScript JIT employed by modern web browsers, it may be possible for an attacker to supply JavaScript that produces native code that could give rise to an instance of CVE-2018-3639#
Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management' -Name "FeatureSettingsOverride" -Value "00000008"
Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management' -Name "FeatureSettingsOverrideMask" -Value "00000003"

Write-Verbose "Allowed Null Session"
### Impact : Unauthorized users can establish a null session and obtain sensitive information, such as usernames and/or the share list, which could be used in further attacks against the host
Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\LSA' -Name "RestrictAnonymous" -Value "00000001"
Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\LSA' -Name "everyoneincludesanonymous" -Value "00000000"


Set-ItemProperty -Path 'hklm:\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\Explorer' -Name "ForceActiveDesktopOn" -Value "00000000"
Set-ItemProperty -Path 'hklm:\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\Explorer' -Name "NoActiveDesktopChanges" -Value "00000001"
Set-ItemProperty -Path 'hklm:\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\Explorer' -Name "NoActiveDesktop" -Value "00000001"
Set-ItemProperty -Path 'hklm:\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\Explorer' -Name "ShowSuperHidden" -Value "00000001"

Write-Verbose "Microsoft Windows Explorer AutoPlay Not Disabled"
###Impact: Exploiting this vulnerability can cause malicious applications to be executed unintentionally at escalated privilege ###
try {
    New-ItemProperty -Path 'hklm:\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\Explorer' -Name "NoDriveTypeAutoRun" -Value "00000255" -PropertyType "DWord"    
}
catch {
    Write-Verbose "Error writing NoDriveTypeAutoRun, probably already exists"    
}
Set-ItemProperty -Path 'hklm:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer' -Name "NoDriveTypeAutoRun" -Value "00000001"

Write-Verbose "Windows Registry Setting To Globally Prevent Socket Hijacking Missing"
###Impact: If this registry setting is missing, in the absence of a SO_EXCLUSIVEADDRUSE check on a listening privileged socket, local unprivileged users can easily hijack the socket and intercept all data meant for the privileged process #####
Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Services\AFD\Parameters' -Name "ForceActiveDesktopOn" -Value "00000001"

Write-Verbose "Disable TLS 1.0"
###Impact: An attacker can exploit cryptographic flaws to conduct man-in-the-middle type attacks or to decryption communications###
#Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Client' -Name "Enabled" -Value "00000000"
#Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.0\Client' -Name "DisabledByDefault" -Value "00000001"

Write-Verbose "Disable TLS 1.1"
try {
    Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.1\Server' -Name "DisabledByDefault" -Value "0" -Type DWord
}
catch {
    Write-Verbose "Error updating TLS 1.1 DisabledByDefault, key probably doesn't exist"  
}

Write-Verbose "Disable SSL v3"
###Impact: SSL 3.0 is an obsolete and insecure protocol.
###Encryption in SSL 3.0 uses either the RC4 stream cipher, or a block cipher in CBC mode.
###RC4 is known to have biases, and the block cipher in CBC mode is vulnerable to the POODLE attack.
try {
    Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Client' -Name "DisabledByDefault" -Value "00000001"
}
catch {
    Write-Verbose "Error updating TLS 3.0 Client, key probably doesn't exist"  
}

try {
    Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\SSL 3.0\Server' -Name "Enabled" -Value "00000000"
}
catch {
    Write-Verbose "Error updating TLS 3.0 Server, key probably doesn't exist"  
}

Write-Verbose "Disable RC4 Protocols"
try {
    Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC4 128/128' -Name "Enabled" -Value "00000000"
}
catch {
    Write-Verbose "Error updating Ciphers\RC4 128/128, key probably doesn't exist"  
}

try {
    Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC4 40/128' -Name "Enabled" -Value "00000000"
}
catch {
    Write-Verbose "Error updating Ciphers\RC4 40/128, key probably doesn't exist"  
}

try {
    Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\RC4 56/128' -Name "Enabled" -Value "00000000"
}
catch {
    Write-Verbose "Error updating Ciphers\RC4 56/128, key probably doesn't exist"  
}

try {
    Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\Triple DES 168' -Name "Enabled" -Value "00000000"
}
catch {
    Write-Verbose "Error updating Ciphers\Triple DES 168, key probably doesn't exist"  
}

try {
    Set-ItemProperty -Path 'hklm:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Ciphers\Triple DES 168/168' -Name "Enabled" -Value "00000000"
}
catch {
    Write-Verbose "Error updating Ciphers\Triple DES 168/168, key probably doesn't exist"  
}


Write-Verbose "Microsoft Windows FragmentSmack Denial of Service Vulnerability (ADV180022)"
###Impact: A system under attack would become unresponsive with 100% CPU utilization but would recover as soon as the attack terminated. ###
Set-NetIPv4Protocol -ReassemblyLimit 0
Set-NetIPv6Protocol -ReassemblyLimit 0

Write-Verbose "MS15-011 Hardening UNC Paths Breaks GPO Access -	Microsoft Group Policy Remote Code Execution Vulnerability (MS15-011)"
###Impact: The vulnerability could allow remote code execution if an attacker convinces a user with a domain-configured system to connect to an attacker-controlled network ###

try {
    Set-ItemProperty -Path 'hklm:\SOFTWARE\Policies\Microsoft\Windows\NetworkProvider\HardenedPaths' -Name "\\*\netlogon" -Value "RequireMutualAuthentication=1, RequireIntegrity=1, RequirePrivacy=1"
}
catch {
    Write-Verbose "Error updating NetworkProvider\HardenedPaths - netlogon key probably doesn't exist"  
}

try {
    Set-ItemProperty -Path 'hklm:\SOFTWARE\Policies\Microsoft\Windows\NetworkProvider\HardenedPaths' -Name "\\*\sysvol" -Value "RequireMutualAuthentication=1, RequireIntegrity=1, RequirePrivacy=1"
}
catch {
    Write-Verbose "Error updating NetworkProvider\HardenedPaths - sysvol key probably doesn't exist"  
}

Write-Verbose "Windows Update for Credentials Protection and Management (Microsoft Security Advisory 2871997)"
### IMPACT If this vulnerability is successfully exploited, attackers can steal credentials of the system. ###

try {
    Set-ItemProperty -Path 'hklm:\System\CurrentControlSet\Control\SecurityProviders\WDigest' -Name "UseLogonCredential" -Value "0"
}
catch {
    Write-Verbose "Error updating SecurityProviders\WDigest key probably doesn't exist"  
}
Write-Verbose "Enabling strong cryptography for .NET V4..."

#x64

try {
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Wow6432Node\Microsoft\.NetFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord
}
catch {
    Write-Verbose "Error updating Wow6432Node SchUseStrongCrypto, key probably doesn't exist"  
}

#x86
try {
    Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NetFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value '1' -Type DWord
}
catch {
    Write-Verbose "Error updating SchUseStrongCrypto, key probably doesn't exist"  
}
```

Now you can have the peace of mind that the OS has been hardened.

## Samples

We have an [Octopus Admin](https://g.octopushq.com/OctopusAdminSamplesSpace) Space on our Samples instance of Octopus. You can sign in as `Guest` to take a look at these examples and more Runbooks in the `Deployment Target Management` project.
