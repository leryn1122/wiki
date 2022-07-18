
记录了一些零碎而奇怪的问题, 避免其他文档过于零碎, 全部集中在这里.

<a name="vLW3I"></a>
### 破解 Beyond Compare

破解 Beyond Compare, 定期执行这个命令即可:

```
reg delete "HKEY_CURRENT_USER\Software\Scooter Software\Beyond Compare 4" /v CacheID /f
```

<a name="aXxYO"></a>
### Windows 远程桌面无法连接

Windows 远程桌面如果无法连接参考如下解决方案.

- [https://answers.microsoft.com/zh-hans/windows/forum/windows_10-win_family/win10家庭中文版/7a17ef28-955a-4c8b-9166-9da6cbb0f87c](https://answers.microsoft.com/zh-hans/windows/forum/windows_10-win_family/win10%E5%AE%B6%E5%BA%AD%E4%B8%AD%E6%96%87%E7%89%88/7a17ef28-955a-4c8b-9166-9da6cbb0f87c)
- [https://blog.csdn.net/daqiang012/article/details/82385720](https://blog.csdn.net/daqiang012/article/details/82385720)
- [https://www.cnblogs.com/roystime/p/9035128.html](https://www.cnblogs.com/roystime/p/9035128.html)
- [https://www.cnblogs.com/bribe/p/11196258.html](https://www.cnblogs.com/bribe/p/11196258.html)
```
# 管理员权限下开启命令行, 执行如下两个命令并重启机器
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\ /v AllowInsecureGuestAuth /d 1 /t REG_DWORD /f
reg add HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\CredSSP\Parameters /v AllowEncryptionOracle /d 2 /t REG_DWORD /f
```


```vbnet
Set oShell = CreateObject("WScript.Shell")
    Set oFS = CreateObject("Scripting.FileSystemObject")
        sHomeFolder = oShell.ExpandEnvironmentStrings("%USERPROFILE%")
        sJBDataFolder = oShell.ExpandEnvironmentStrings("%APPDATA%") + "\JetBrains"
        
        Set re = New RegExp
            re.Global     = True
            re.IgnoreCase = True
            re.Pattern    = "\.?(IntelliJIdea|GoLand|CLion|PyCharm|DataGrip|RubyMine|AppCode|PhpStorm|WebStorm|Rider).*"
            
            Sub removeEval(ByVal file, ByVal sEvalPath)
                bMatch = re.Test(file.Name)
                If Not bMatch Then
                    Exit Sub
                    End If
                    
                    If oFS.FolderExists(sEvalPath) Then
                        oFS.DeleteFolder sEvalPath, True 
                    End If
                End Sub
                
                If oFS.FolderExists(sHomeFolder) Then
                    For Each oFile In oFS.GetFolder(sHomeFolder).SubFolders
                    removeEval oFile, sHomeFolder + "\" + oFile.Name + "\config\eval"
                Next
            End If
            
            If oFS.FolderExists(sJBDataFolder) Then
                For Each oFile In oFS.GetFolder(sJBDataFolder).SubFolders
                removeEval oFile, sJBDataFolder + "\" + oFile.Name + "\eval"
            Next
        End If
        
MsgBox "done"
        
```
