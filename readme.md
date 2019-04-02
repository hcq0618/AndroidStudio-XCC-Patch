# AndroidStudio-XCC-Patch
#### Android Studio XML编辑器加速补丁

## 应用场景

在大型Android项目工程中编辑XML文件时，如果当前module依赖项过多，编辑器在处理自动补全提示时可能会卡顿较长时间，开发效率极低。

此补丁主要功能就是在Android Studio中修改自动补全相关的逻辑，绕过多余的代码分支，以达到提速效果。测试多个大型项目使用后可以将自动补全提示速度降到毫秒级。

## 简单原理

补丁加速的原理是将`com.android.ide.common.repository.ResourceVisibilityLookup.Provider.get(com.android.builder.model.AndroidProject, com.android.builder.model.Variant)`
方法中对`com.android.ide.common.repository.ResourceVisibilityLookup.Provider.get(com.android.builder.model.AndroidArtifact)`方法的调用的返回值替换成`li.joker.AndroidStudioXMLCodeCompletionPatch.fakeTransparentVisibility`方法
的返回值来实现替换耗时代码逻辑的功能。

## 可能的副作用

使用该补丁替换原始逻辑后并不会对自动补全功能造成明显影响，因为原始代码中耗时操作主要是在过滤当前module依赖项中声明了public.xml，而不在public.xml中包含的资源。
public.xml功能请参考[What is the use of the res/values/public.xml file on Android?](https://stackoverflow.com/questions/9348614/what-is-the-use-of-the-res-values-public-xml-file-on-android)
，如果你的项目使用了很多包含public.xml的依赖，不建议使用此补丁，使用时如果引用了依赖库的资源就需要人工主动判断引用的资源是否是public，编译后请务必在运行时仔细测试资源获取是否正常。

The purpose of isKnown() method and isPublic() method are to find all resource names in the dependency tree of the current module, determine if they can be referenced, and submit the final visible resource to the IDE for auto-completion Suggestions. However, only 6 of the commonly used Android libraries that declare public.

Therefore, the judgment of ignoring public condition has little influence.



If your project uses a lot of dependencies that contain public.xml, it is not recommended to use this patch. If you refer to the resources of the dependent libraries, you need to manually determine whether the referenced resource is public.

After compilation, be sure to test the resource acquisition carefully at run time.

## 使用方法

如果你使用的macOS系统，那么恭喜你可以直接使用脚本运行：
```shell
curl -L https://github.com/mimers/AndroidStudio-XCC-Patch/raw/master/android-studio-cc-auto-patch.sh -o android-studio-cc-auto-patch.sh
bash android-studio-cc-auto-patch.sh
```
然后重启Android Studio即可。
恢复原始状态再次执行此脚本添加`-u`参数即可，一般在使用Android Studio增量更新时需要操作，更新完成后再重新打补丁

如果你使用的是其他操作系统，则需要手动下载jar包执行：
1. 首先备份Android Studio安装目录下的plugins/android/lib/sdk-common.jar文件，在使用增量更新方式时需要先恢复此文件到原始状态
2. 下载jar包,(https://github.com/mimers/AndroidStudio-XCC-Patch/releases/download/1.0/android-studio-cc-patch.jar)[https://github.com/mimers/AndroidStudio-XCC-Patch/releases/download/1.0/android-studio-cc-patch.jar]
3. 执行命令`java -jar android-studio-cc-patch.jar <你的sdk-common.jar文件绝对路径>`
4. 重启Android Studio

好了，现在重新感受下如丝般顺滑的Android Studio吧 🚀


If you are using macOS, then you can use the script to run:

```shell
curl -L https://github.com/mimers/AndroidStudio-XCC-Patch/raw/master/android-studio-cc-auto-patch.sh -o android-studio-cc-auto-patch.sh bash android-studio-cc-auto-patch.sh
Then restart Android Studio.
```

To restore the original state, execute this script again to add the -u parameter, which is generally required when incremental update using Android Studio is used, and then patch again after the update is completed.



If you are using another operating system, you will need to manually download the jar to execute:

1. First, backup the plugins/ Android /lib/sdk-common.jar file in the Android Studio installation directory, and restore this file to the original state when an incremental update is used
2. Download the jar package
3. Execute the command java-jar android-studio-cc-patch.jar < your sdk-common.jar file absolute path >
4. Restart the Android Studio

Now, to feel the silky smooth Android Studio 🚀
