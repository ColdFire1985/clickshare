# clickshare
Repair Clickshare tool (uninstall and repair reg)


Removes Clickshare and repares the reg acl

try{
		Start-Process -filepath "C:\ClickShareApp\ClickShare\Update.exe" -ArgumentList "-uninstall"
		}
		catch{
			write-host "not found"
			
		}
		#ClickShare Desktop App Machine-Wide Installer	Barco N.V.	4.19.2.10	2023-01-03 01:00:00Z
		Remove-MSIApplications -Name "ClickShare*" -WildCard -ContinueOnError $true -ErrorAction SilentlyContinue
		
		
		###########
		New-PSDrive HKU Registry HKEY_USERS 
        $items = Get-ChildItem HKU:

        foreach( $i in $items)
        {
        if(test-path ("registry::$($i)\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders\"))
			{
			$acl1 = get-acl "registry::$($i)\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders\"
			$sid = New-Object System.Security.Principal.SecurityIdentifier ("S-1-15-2-1")
			$person_temp = $sid.Translate( [System.Security.Principal.NTAccount])
			$person = $person_temp.Value.Split("\")[1]
			$access = [System.Security.AccessControl.RegistryRights]"ReadKey"
			$inheritance = [System.Security.AccessControl.InheritanceFlags]"ContainerInherit,ObjectInherit"
			$propagation = [System.Security.AccessControl.PropagationFlags]"None"
			$type = [System.Security.AccessControl.AccessControlType]"Allow"
			$rule = New-Object System.Security.AccessControl.RegistryAccessRule($person,$access,$inheritance,$propagation,$type)
			$acl1.AddAccessRule($rule)
			$acl1 | Set-Acl
			}
        }

		Remove-PSDrive HKU

		$proc = Get-CimInstance Win32_Process -Filter "name = 'explorer.exe'"
		$usernamewithoutdomain = Invoke-CimMethod -InputObject $proc[0] -MethodName GetOwner | select-object -ExpandProperty user
		if (-not (Get-AppxPackage Microsoft.AAD.BrokerPlugin -user "$usernamewithoutdomain")) { Add-AppxPackage -Register "$env:windir\SystemApps\Microsoft.AAD.BrokerPlugin_cw5n1h2txyewy\Appxmanifest.xml" -DisableDevelopmentMode -ForceApplicationShutdown } Get-AppxPackage Microsoft.AAD.BrokerPlugin
		
