# NLA domain detection issue.

## This fix should resolve an issue where the network profile of a domain-joined machine is Private/Public, instead of DomainAuthenticated.

After the machine reboots and before the NIC initializes, the ``NLASVC`` service attempts to detect the presence of a domain.  
If the detection fails, the information will be cached and even though the NIC gets initialized, the OS still applies the cached information and hence the incorrect network profile is applied.

The following registry changes may help. At this point, I haven't had the chance to test these thoroughly, because the issue is not persistent.

First, disable the ``Domain Discovery`` negative cache by adding the ``NegativeCachePeriod`` registry key to the following subkey:

```powershell
# The default value is 45 seconds. Set it to 0 to disable caching.
New-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\Netlogon\Parameters -PropertyType 'DWORD' -Name NegativeCachePeriod -Value 0
```

If the issue isn't resolved, disable the DNS negative cache by adding the ``MaxNegativeCacheTtl`` registry key to the following subkey:

```powershell
# The default value is 5 seconds. Set it to 0 to disable caching.
New-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters -PropertyType 'DWORD' -Name MaxNegativeCacheTtl -Value 0
```

*NOTE: This registry key disables the Domain detection negative cache. NLA normally tries to detect the domain multiple times at network setup (triggered by route change, IP address change, etc), but if the first detection fails (for example ERROR_NO_SUCH_DOMAIN), the negative result gets cached in netlogon, and will be used in the next time NLA performs domain discovery.*

There is also another registry key we need to add:

```powershell
New-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\NlaSvc\Parameters -PropertyType 'DWORD' -Name AlwaysExpectDomainController -Value 1
```

*NOTE: This registry key alters the behavior when NLA retries domain detection.*

---

The solution was provided by Sunny Qi in the following Microsoft Learn Answers thread:  
https://learn.microsoft.com/en-us/answers/questions/400385/network-location-awareness-not-detecting-domain-ne
