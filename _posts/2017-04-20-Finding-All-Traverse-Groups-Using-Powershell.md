---
title:  "Finding All Traverse Groups Using Powershell"
date:   2017-04-20 17:56:55 +0000
categories: traverse fileshares powershell
---
When it comes to managing file shares one of the larger issues I deal with on a regular basis is identifying all the traverse groups that lead to a specific folder.

The first thing to do is identify what makes a traverse ACL.  Where I work we simply use **Read and Execute** rights and then apply them to the folder only, not inheriting further down the tree.

I first build those into variables I can use later on:

```powershell
# Build tests for traverse group
$TravRights = [System.Security.AccessControl.FileSystemRights]"ReadAndExecute,Synchronize"
$TravInheritanceFlag = [System.Security.AccessControl.InheritanceFlags]::None
$TravPropagationFlag = [System.Security.AccessControl.PropagationFlags]::None
$TravType = [System.Security.AccessControl.AccessControlType]::Allow
```

In order to properly handle the full path we'll identify the root of the path.  When it's a network share that's grabbing the third entry of the array created from the "string".split("\") as the first two entries are blank from the front double backslashes.  Otherwise I'm assuming it's a normal filesystem provider which can simply have it's qualifier split from the path.

```powershell
# Get the root of the path
if($Path.StartsWith("\\")) {
    $RootPath = "\\" + [string]::Join["\",$Path.Split("\"](2))
} else {
    $RootPath = Split-Path -Path $Path -Qualifier
}
```

Next, you need to build a list of the directories leading to the directory in question.  The first thing I do is pull apart the string using [String.Split()](http://ss64.com/ps/split.html).  Next I use the Split-Path cmdlet to identify the root drive.  We also need to pull out the root of the drive so all we're getting is a list of the directories in order.

Also, we'll create a path's arraylist which we'll use to parse through the directories and create a list of all thee paths not just the directories.  So if we pass into the function C:\Foo\Bar we get "C:\Foo", "C:\Foo\Bar back as an array list.

```powershell
# Split up paths and pull the root
$spath = $Path.Replace("$RootPath\", "").Split("\")
$Paths = New-Object System.Collections.ArrayList

# Build a list of all the directories that lead to the target directory
for($i = 0; $i -le $spath.Length; $i++) {
    if($spath[$i] -ne $null) {
        $PathToAdd = ""
        for($j = 0; $j -le $i; $j++) {
            $PathToAdd += "$($spath[$j])\"
        }
        # Add the new path to our list of paths
        $Paths.Add("$RootPath\$PathToAdd") | Out-Null
    }
}
```

Once we have our list of paths to check we can simply loop through the paths then gett the ACL for each path.  Once we have the ACL we can loop through the Access rules in each ACL and then check the access rule against the flags we defined earlier which are what we're looking for.

```powershell
# Loop through the paths and determine which contain a traverse group
foreach($item in $Paths) {
    $itemacl = Get-Acl -Path $item
        foreach($acl in $itemacl.Access) {
        # Check the acl for the traverse permissions defined earlier
        if(($acl.InheritanceFlags -eq $travInheritanceFlag) -and ($acl.PropagationFlags -eq $travPropagationFlag) -and ($acl.FileSystemRights -eq $travRights) -and ($acl.AccessControlType -eq $travType) -and ($acl.IsInherited -eq $false)) {
            # We&#039;ve now found a traverse group
            $SamAccountName = $acl.IdentityReference.ToString().Split["\"](1)

            # Make sure the account name isn&#039;t null, then get the group make sure it exists in AD then add the path to the group output object.
            if($null -ne $SamAccountName) {
                $ADObject = Get-ADGroup -Identity $SamAccountName
                if($ADObject -ne $null) {
                    $TraverseGroups += $ADObject | Add-Member -MemberType NoteProperty -Name TraversePath -Value $item -Force -PassThru
                }
            }
        }
    }
}
```

Once we've identified that we've found exactly what we're looking for and that the account name isn't null we can add it to our $TraverseGroups array.

The full code can be obtained from my GitHub repository [Find-TraverseGroups](https://github.com/jbolduan/Find-TraverseGroups/blob/master/Find-TraverseGroups.ps1).
