---
layout:         post
title:          Visual Studio
subtitle:       Visual Studio
date:           2022-11-25 17:44:00
author:         nomadli
header-img:     ../img/bg-coffee.jpeg
catalog:        true
tags:
        - other
---

* content
{:toc}

## 编译器命令
- %comspec% /k ""C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\Tools\VsDevCmd.bat""
- devenv X:\x\x.sln /clean Debug|Win32
    - Debug Release Debug|Win32 Release|Win32 ...
    - /Build        增量编译
    - /Clean        清空编译目录
    - /Project      指定生成、清理或部署的项目
    - /ProjectConfig重写解决方案配置中指定的项目配置。
    - /Rebuild      先清理，然后使用指定配置生成解决方案或项目
    - /Run          编译并运行指定的解决方案
    - /RunExit      编译并运行指定的解决方案然后关闭 IDE
    - /Command      启动 IDE 并执行该命令
    - /Deploy       生成并部署指定的生成配置
    - /Edit         在此应用程序的运行实例中打开指定文件
    - /LCID         设置 IDE 中用于用户界面的默认语言。
    - /Log          将 IDE 活动记录到指定的文件以用于疑难解答
    - /NoVSIP       禁用用于 VSIP 测试的 VSIP 开发人员许可证密钥
    - /Out          将生成日志追加到指定的文件中
    - /ResetAddin   移除与特定外接程序关联的命令和命令用户界面
    - /ResetSettings恢复 IDE 的默认设置或重置为指定的VSSettings文件
    - /ResetSkipPkgs清除所有添加到 VSPackages 的 SkipLoading 标记
    - /SafeMode     以安全模式启动 IDE，加载最少数量的窗口
    - /Upgrade      升级项目或解决方案以及其中的所有项目
    - /debugexe     打开指定要调试的可执行文件,其余部分将作为参数传递给此可执行文件
    - /useenv       使用 PATH、INCLUDE、LIBPATH和LIB环境变量而不是使用VC++生成的

## .Net 服务编译
- https://dist.nuget.org/win-x86-commandline/latest/nuget.exe
- nuget.exe install X:/xx/packages.config -O X:/xx/packages -configfile X:/x/nuget.config
```xml
<!--nuget.config 配置nuget的网络及服务器信息-->
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <config>
        <!--仅用于packages.config 安装下载的package的路径 否则默认 $(Solutiondir)/packages-->
        <add key="repositoryPath" value="%PACKAGEHOME%/External" />

        <!--dependencyVersion仅用于packages.config 包安装、还原和更新的默认Lowest、HighestPatch、HighestMinor、Highest-->
        <!--globalPackagesFolder仅用于PackageReference全局包文件夹默认%userprofile%\.nuget\packages NUGET_PACKAGES环境变量-->

        <!--如果命令没有指定源URL,则使用此配置的源-->
        <add key="defaultPushSource" value="https://MyRepo/ES/api/v2/package" />
        
        <!--设置访问源需要经过的http代理 no_proxy-->
        <add key="http_proxy" value="host" />
        <add key="http_proxy.user" value="username" />
        <add key="http_proxy.password" value="encrypted_password" />

        <!--maxHttpRequestsPerSource 每个包源的最大并行请求数-->
        <!--signatureValidationMode包安签名验证模式 accept require-->
    </config>

    <bindingRedirects>
        <!--是否执行自动绑定重定向-->
        <add key="skip" value="True" />
    </bindingRedirects>

    <packageRestore>
        <!--是否自动还原,EnableNuGetPackageRestore环境变量-->
        <add key="enabled" value="True" />

        <!--是否应在生成期间检查缺少的包-->
        <add key="automatic" value="True" />
    </packageRestore>

    <solution>
        <!--使用源代码管理时是否忽略包文件夹-->
        <add key="disableSourceControlIntegration" value="true" />
    </solution>

    <packageSources> <!--列出所有包源-->
        <clear /> <!--忽略之前配置的包源-->
        <add key="NuGet official package source" value="https://api.nuget.org/v3/index.json" />
        <add key="Contoso" value="https://MyRepo/ES/nuget" />
    </packageSources>

    <disabledPackageSources><!--禁止使用的源-->
        <add key="Contoso" value="true" />
    </disabledPackageSources>

    <packageSourceCredentials><!--包源的用户名密码-->
        <Contoso>
            <add key="Username" value="user@contoso.com" />
            <add key="Password" value="..." /><!--直接配置加密密码-->
        </Contoso>
        <Test_x0020_Source>
            <add key="Username" value="user" />
            <add key="ClearTextPassword" value="%TestSourcePassword%" /><!--环境变量密码或未加密密码-->
            <add key="ValidAuthenticationTypes" value="basic, negotiate" /><!--密码验证模式-->
        </Test_x0020_Source>
    </packageSourceCredentials>

    <apikeys><!--使用nuget setapikey 设置的密钥-->
        <add key="https://MyRepo/ES/api/v2/package" value="encrypted_api_key" />
    </apikeys>

    <trustedSigners><!--受信任的源签名-->
        <author name="microsoft">
            <certificate fingerprint="3F9001EA83C560D712C24CF213C3D312CB3BFF51EE89435D3430BD06B5D0EECE" hashAlgorithm="SHA256" allowUntrustedRoot="false" />
            <certificate fingerprint="AA12DA22A49BCE7D5C1AE64CC1F3D892F150DA76140F210ABD2CBFFCA2C18A27" hashAlgorithm="SHA256" allowUntrustedRoot="false" />
        </author>
        <repository name="nuget.org" serviceIndex="https://api.nuget.org/v3/index.json">
            <certificate fingerprint="0E5F38F57DC1BCC806D8494F4F90FBCEDD988B46760709CBEEC6F4219AA6157D" hashAlgorithm="SHA256" allowUntrustedRoot="false" />
            <certificate fingerprint="5A2901D6ADA3D18260B9C6DFE2133C95D74B9EEF6AE0E5DC334C8454D1477DF4" hashAlgorithm="SHA256" allowUntrustedRoot="false" />
            <owners>microsoft;aspnet;nuget</owners>
        </repository>
    </trustedSigners>
</configuration>
```
- X:/x/Common7/Tools/vsvars32.bat" && msbuild  "x:/x/x.csproj" /v:m /t:Rebuild /p:Configuration=Release;AllowUnsafeBlocks=true 
