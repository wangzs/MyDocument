# Cocos2d-x编译运行等相关错误
## cocos2d-x android set up error - java.lang.NullPointerException
> 
原因：在eclipse打开Android工程时，==NDK_ROOT==未定义，则会在.cproject文件中插入如下内容：
```xml
<cconfiguration id="0.1230402123.1377291156">
    <storageModule buildSystemId="org.eclipse.cdt.managedbuilder.core.configurationDataProvider" id="0.1230402123.1377291156" moduleId="org.eclipse.cdt.core.settings" name="Debug">
        <externalSettings/>
            <extensions>
                <extension id="org.eclipse.cdt.core.VCErrorParser" point="org.eclipse.cdt.core.ErrorParser"/>
                <extension id="org.eclipse.cdt.core.GmakeErrorParser" point="org.eclipse.cdt.core.ErrorParser"/>
                <extension id="org.eclipse.cdt.core.CWDLocator" point="org.eclipse.cdt.core.ErrorParser"/>
                <extension id="org.eclipse.cdt.core.GCCErrorParser" point="org.eclipse.cdt.core.ErrorParser"/>
                <extension id="org.eclipse.cdt.core.GASErrorParser" point="org.eclipse.cdt.core.ErrorParser"/>
                <extension id="org.eclipse.cdt.core.GLDErrorParser" point="org.eclipse.cdt.core.ErrorParser"/>
            </extensions>
        </storageModule>
    <storageModule moduleId="org.eclipse.cdt.core.externalSettings"/>
</cconfiguration>
```
如果之后定义了==NDK_ROOT==，eclipse打开工程时，仍然会出现error的提示，无法编译工程。
**解决：**
删除.cproject文件中对应的上段cconfiguration块（关闭eclipse的情况下进行），再次打开eclipse，不再提示error。

## 编译时出现error: format not a string literal and no format arguments [-Werror=format-security]
使用CCLOG函数时，无法使用字符串变量的参数传递
解决方法：
```
// proj.android\jni\Application.mk
APP_CPPFLAGS += -Wno-error=format-security
```