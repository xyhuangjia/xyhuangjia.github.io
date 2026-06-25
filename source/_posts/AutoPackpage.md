---
title: 一次脚本打包的实践
date: 2017-05-22 00:00:00
categories: 工具
tags: 自动化, 打包, iOS
---

<h3>业务场景</h3>

>&emsp;&emsp;&emsp;&emsp;小公司，经常要按需求方爸爸的要求来做业务调整，于是乎经常被测试妹子催着打个新测试包吧，打个新测试包吧···· &nbsp;急切的想找一个可以偷懒的方式来短暂解放自己的双手不用一直守在电脑前点啊点的。能够自动打包并发送到测试妹子的邮箱里实现这样的一条龙服务。自己能够偷偷的玩会，嘿嘿😜😜😜😜<!-- more -->

&emsp;&emsp;&emsp;&emsp;~~自己规划的需求很简单,流程共分为两步：**第一步，脚本自动打包，第二步，将ipa.zip发送到x相关人员的邮箱**~~

&emsp;&emsp;&emsp;&emsp;流程共分为三步：**第一步，脚本自动打包，第二步，将ipa.zip发送到x相关人员的邮箱。第三步：ipa包上传App Store**

&emsp;&emsp;&emsp;&emsp;之前在做自动化测试时候也曾经涉猎过脚本打包的业务，当时是在Jenkin环境下实现该业务的([Jenkins 打包 iOS](http://www.jianshu.com/p/047ac4c39297))。当时需求很简单也是预研性质的东西，只为能实现脚本打包。所以脚本实现方式比较粗犷，现在看来已经不能很完美的适应现在的要求，我需要一种更智能更优雅的实现方式。经过一段时间的资料查找,并没有找到一个很好的解决方案。后来朋友推荐[jkpang-庞](https://github.com/jkpang)写的一个打包脚本[PPAutoPackageScript](https://github.com/jkpang/PPAutoPackageScript)提供了很好的解决方案模型，于是就恬不知耻的按自己的需求悄悄改代码了。


<h3>脚本打包</h3>

&emsp;&emsp;&emsp;&emsp;脚本是使用别人的，前人栽树后人乘凉。我根据实际需要将原脚本中的一些固定参数改成手动选择输入的模式，在这里不做一一详细讲解。

<h3>将ipa.zip发送到相关人员的邮箱</h3>

接下来就是发送ipa到指定邮箱。在实现过程中要解决如下两个问题：

 - 每次上传发邮件都要输入密码
 ```
 #查阅资料后使用该方法 echo <123> | sudo -S <command>
 ```
 
 - 发邮件时修改上传大文件限制
 ```
 # 修改大文件限制 echo $macPassword | sudo -S postconf -e message_size_limit=0
postconf -d | grep size
```

<h3>打包ipa上传App Store</h3>

&emsp;&emsp;&emsp;&emsp;既然脚本打包并发送邮件都做成了，我是不是可以更懒点？上线时候是不是也可以通过脚本来完成？能够留出更多时间releax一下？答案绝对是可以的！ 其实对于命令行上传App Store我也觉得没什么好说的，其一是因为在apple官网上直接给出[altool命令](http://help.apple.com/itc/apploader/#/apdATD1E53-D1E1A1303-D1E53A1126)开发者只要遵循规则去填充就好。其次是因为在实际开发过程中我们大多数人使用其的频率实在算不上高。大家更多情况下还是会使用Xcode直接上传或者是通过Application Loader来上传，并不会选择的去使用命令行来实现上传任务。然而，我不喜欢静静守在机器前看进度条，于是我做了如下流程测试：

- 使用脚本验证
```
altool --validate-app -f file -u username [-p password] [--output-format xml]
```
- 脚本上传ipa
```
altool --upload-app -f file -u username [-p password] [--output-format xml]
```
- 捕获输出日志，根据日志内容判断上传结果。

```
if [[ "$result" != "" ]]
 then
     echo "上传到App Store成功"
     # 上传成功，通知开发人员
 else
     echo "上传到App Store失败"
     # 保留xml以备查询
 fi

 ```
 
实际测试过程中笔者得到如下格式的成功和失败两份xml文档，读者在操作时候自行选择是否参考


 ```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>dev-tools-info</key>
	<dict>
		<key>platforms</key>
		<dict>
			<key>iPhoneOS.platform</key>
			<dict>
				<key>paths</key>
				<array>
					<string>/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/usr/bin</string>
					<string>/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin</string>
					<string>/Applications/Xcode.app/Contents/Developer/usr/bin</string>
					<string>/usr/local/bin</string>
					<string>/usr/bin</string>
					<string>/Users/Master/.rbenv/shims</string>
					<string>/bin</string>
					<string>/usr/sbin</string>
					<string>/sbin</string>
					<string>/Applications/Xcode.app/Contents/Applications/Application Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Resources</string>
				</array>
			</dict>
		</dict>
		<key>search-method</key>
		<integer>1</integer>
		<key>search-method-string</key>
		<string>Enclosing Xcode</string>
		<key>xcode-info</key>
		<dict>
			<key>path</key>
			<string>/Applications/Xcode.app/Contents/Developer</string>
			<key>version.plist</key>
			<dict>
				<key>BuildVersion</key>
				<string>4</string>
				<key>CFBundleShortVersionString</key>
				<string>8.3.2</string>
				<key>CFBundleVersion</key>
				<string>12175</string>
				<key>ProductBuildVersion</key>
				<string>8E2002</string>
				<key>ProjectName</key>
				<string>IDEFrameworks</string>
				<key>SourceVersion</key>
				<string>12175000000000000</string>
			</dict>
		</dict>
	</dict>
	<key>os-version</key>
	<string>10.12.5</string>
	<key>product-errors</key>
	<array>
		<dict>
			<key>code</key>
			<integer>-18000</integer>
			<key>message</key>
			<string>ERROR ITMS-4238: "Redundant Binary Upload. There already exists a binary upload with build version '1.3' for train '1.3'" at SoftwareAssets/PreReleaseSoftwareAsset</string>
			<key>userInfo</key>
			<dict>
				<key>NSLocalizedDescription</key>
				<string>ERROR ITMS-4238: "Redundant Binary Upload. There already exists a binary upload with build version '1.3' for train '1.3'" at SoftwareAssets/PreReleaseSoftwareAsset</string>
				<key>NSLocalizedFailureReason</key>
				<string>ERROR ITMS-4238: "Redundant Binary Upload. There already exists a binary upload with build version '1.3' for train '1.3'" at SoftwareAssets/PreReleaseSoftwareAsset</string>
				<key>NSLocalizedRecoverySuggestion</key>
				<string>ERROR ITMS-4238: "Redundant Binary Upload. There already exists a binary upload with build version '1.3' for train '1.3'" at SoftwareAssets/PreReleaseSoftwareAsset</string>
			</dict>
		</dict>
	</array>
	<key>tool-path</key>
	<string>/Applications/Xcode.app/Contents/Applications/Application Loader.app/Contents/Frameworks/ITunesSoftwareService.framework</string>
	<key>tool-version</key>
	<string>1.9.784</string>
</dict>
</plist>
```
<center>以上为失败样式</center>

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>os-version</key>
    <string>10.11.2</string>
    <key>success-message</key>
    <string>No errors validating archive at /XXX/XXX.ipa</string>
    <key>tool-version</key>
    <string>1.1.902</string>
    <key>xcode-versions</key>
    <array>
        <dict>
            <key>path</key>
            <string>/Applications/Xcode.app</string>
            <key>version.plist</key>
            <dict>
                <key>BuildVersion</key>
                <string>7</string>
                <key>CFBundleShortVersionString</key>
                <string>7.2</string>
                <key>CFBundleVersion</key>
                <string>9548</string>
                <key>ProductBuildVersion</key>
                <string>7C68</string>
                <key>ProjectName</key>
                <string>IDEFrameworks</string>
                <key>SourceVersion</key>
                <string>9548000000000000</string>
            </dict>
        </dict>
    </array>
</dict>
</plist>

```
以上为成功样式(当时没拷贝文件现在找的一份[别人的](http://zackzheng.info/2015/12/27/2015-12-27-an-automated-script-for-building-archiving-submission-sending-emails/))

最后附上测试时使用的脚本

```
#App Store账号和密码
username=""
password=""

altoolPath="/Applications/Xcode.app/Contents/Applications/Application Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Versions/A/Support/altool"

# # 检验包内容$ altool

echo "验证ipa包开始"

$("${altoolPath}" --validate-app -f "UsedCars-v1.3.ipa" -u $username [-p $password] [--output-format xml]) > result.xml
# # 上传脚本
"${altoolPath}" --upload-app -f "UsedCars-v1.3.ipa" -u $username -p $password -t ios --output-format xml

FILE=$(cat result.xml)
#echo $FILE
#成功上传后包含字符
PATTERN="success-message"
result=$(echo $FILE | grep "${PATTERN}")
 if [[ "$result" != "" ]]
 then
     echo "上传到App Store成功"
     # 上传成功，通知开发人员
 else
     echo "上传到App Store失败"
     # 保留xml以备查询
 fi

```
经过不断的优化与调整后，最终完整代码如下：

```
# ===============================项目自定义部分(自定义好下列参数后再执行该脚本)============================= #
# 计时
SECONDS=0
# 是否编译工作空间 (例:若是用Cocopods管理的.xcworkspace项目,赋值true;用Xcode默认创建的.xcodeproj,赋值false)

echo "\033[36;1m请选择编译工作空间方式(输入序号,按回车即可) \033[0m"
echo "\033[33;1m1. Cocopods管理序号为1       \033[0m"
echo "\033[33;1m2. Xcode默认创建序号为2    \033[0m"

read archiveType
is_workspace="false"
if [ -n "$archiveType" ]
then
    if  [ "$archiveType" = "1" ] ; then
     is_workspace="true"
     elif  [ "$archiveType" = "2" ] ; then
     is_workspace="false"
     else
     echo "输入的参数无效!!!"
     exit  1
     fi    
fi
# echo $is_workspace

# 指定项目的scheme名称
# (注意: 因为shell定义变量时,=号两边不能留空格,若scheme_name与info_plist_name有空格,脚本运行会失败,暂时还没有解决方法,知道的还请指教!)
echo "\033[36;1m请输入工程名(左上角scheme)：\033[0m"
read scheme
# sleep 0.5
scheme_name="$scheme"
# echo $scheme_name
# 工程中Target对应的配置plist文件名称, Xcode默认的配置文件为Info.plist
info_plist_name="Info"
# 指定要打包编译的方式 : Release,Debug...
echo "\033[36;1m请选择打包编译方式(输入序号,按回车即可) \033[0m"
echo "\033[33;1m1. Debug序号为1       \033[0m"
echo "\033[33;1m2. Release序号为2    \033[0m"

read  configurationType
build_configuration="Debug"
if [ -n "$configurationType" ]
then
    if  [ "$configurationType" = "1" ] ; then
     build_configuration="Debug"
     elif  [ "$configurationType" = "2" ] ; then
     build_configuration="Release"
     else
     echo "输入的参数无效!!!"
     exit  1
     fi    
fi

# ===============================自动打包部分(无特殊情况不用修改)============================= #

# 导出ipa所需要的plist文件路径 (默认为AdHocExportOptionsPlist.plist)
ExportOptionsPlistPath="./PPAutoPackageScript/AdHocExportOptionsPlist.plist"
# 返回上一级目录,进入项目工程目录
cd ..
# 获取项目名称
project_name=`find . -name *.xcodeproj | awk -F "[/.]" '{print $(NF-1)}'`
# 获取版本号,内部版本号,bundleID
info_plist_path="$project_name/$info_plist_name.plist"
bundle_version=`/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" $info_plist_path`
bundle_build_version=`/usr/libexec/PlistBuddy -c "Print CFBundleIdentifier" $info_plist_path`
bundle_identifier=`/usr/libexec/PlistBuddy -c "Print CFBundleVersion" $info_plist_path`

# 删除旧.xcarchive文件
rm -rf ~/Desktop/$scheme_name-IPA/$scheme_name.xcarchive
# 指定输出ipa路径
export_path=~/Desktop/$scheme_name-IPA
# 指定输出归档文件地址
export_archive_path="$export_path/$scheme_name.xcarchive"
# 指定输出ipa地址
export_ipa_path="$export_path"
# 指定输出ipa名称 : scheme_name + bundle_version
ipa_name="$scheme_name-v$bundle_version"

# AdHoc,AppStore,Enterprise三种打包方式的区别: http://blog.csdn.net/lwjok2007/article/details/46379945
echo "\033[36;1m请选择打包方式(输入序号,按回车即可) \033[0m"
echo "\033[33;1m1. AdHoc       \033[0m"
echo "\033[33;1m2. AppStore    \033[0m"
echo "\033[33;1m3. Enterprise  \033[0m"
echo "\033[33;1m4. Development \033[0m"
# 读取用户输入并存到变量里
read parameter
sleep 0.5
method="$parameter"

# 判读用户是否有输入
if [ -n "$method" ]
then
    if [ "$method" = "1" ] ; then
    ExportOptionsPlistPath="./PPAutoPackageScript/AdHocExportOptionsPlist.plist"
    elif [ "$method" = "2" ] ; then
    ExportOptionsPlistPath="./PPAutoPackageScript/AppStoreExportOptionsPlist.plist"
    elif [ "$method" = "3" ] ; then
    ExportOptionsPlistPath="./PPAutoPackageScript/EnterpriseExportOptionsPlist.plist"
    elif [ "$method" = "4" ] ; then
    ExportOptionsPlistPath="./PPAutoPackageScript/DevelopmentExportOptionsPlist.plist"
    else
    echo "输入的参数无效!!!"
    exit 1
    fi
fi
echo "\033[36;1m请输入本机密码(输入密码,按回车即可) \033[0m"
read macPassowrd
echo "\033[32m*************************  开始构建项目  *************************  \033[0m"
# 指定输出文件目录不存在则创建
if [ -d "$export_path" ] ; then
echo $export_path
else
mkdir -pv $export_path
fi

# 判断编译的项目类型是workspace还是project
if $is_workspace ; then
# 编译前清理工程
xcodebuild clean -workspace ${project_name}.xcworkspace \
                 -scheme ${scheme_name} \
                 -configuration ${build_configuration}

xcodebuild archive -workspace ${project_name}.xcworkspace \
                   -scheme ${scheme_name} \
                   -configuration ${build_configuration} \
                   -archivePath ${export_archive_path}
else
# 编译前清理工程
xcodebuild clean -project ${project_name}.xcodeproj \
                 -scheme ${scheme_name} \
                 -configuration ${build_configuration}

xcodebuild archive -project ${project_name}.xcodeproj \
                   -scheme ${scheme_name} \
                   -configuration ${build_configuration} \
                   -archivePath ${export_archive_path}
fi

#  检查是否构建成功
#  xcarchive 实际是一个文件夹不是一个文件所以使用 -d 判断
if [ -d "$export_archive_path" ] ; then
echo "\033[32;1m项目构建成功 🚀 🚀 🚀  \033[0m"
else
echo "\033[31;1m项目构建失败 😢 😢 😢  \033[0m"
exit 1
fi

echo "\033[32m*************************  开始导出ipa文件  *************************  \033[0m"
xcodebuild  -exportArchive \
            -archivePath ${export_archive_path} \
            -exportPath ${export_ipa_path} \
            -exportOptionsPlist ${ExportOptionsPlistPath}
# 修改ipa文件名称
mv $export_ipa_path/$scheme_name.ipa $export_ipa_path/$ipa_name.ipa

# 检查文件是否存在
if [ -f "$export_ipa_path/$ipa_name.ipa" ] ; then
echo "\033[32;1m导出 ${ipa_name}.ipa 包成功 🎉  🎉  🎉   \033[0m"
# open $export_path
else
echo "\033[31;1m导出 ${ipa_name}.ipa 包失败 😢 😢 😢     \033[0m"
# 相关的解决方法
echo "\033[34mps:以下类型的错误可以参考对应的链接\033[0m"
echo "\033[34m  1.\"error: exportArchive: No applicable devices found.\" --> 可能是ruby版本过低导致,升级最新版ruby再试,升级方法自行百度/谷歌,GitHub issue: https://github.com/jkpang/PPAutoPackageScript/issues/1#issuecomment-297589697"
echo "\033[34m  2.\"No valid iOS Distribution signing identities belonging to team 6F4Q87T7VD were found.\" --> http://fight4j.github.io/2016/11/21/xcodebuild/ \033[0m"
exit 1
fi
# 输出打包总用时
echo "\033[36;1m使用PPAutoPackageScript打包总用时: ${SECONDS}s \033[0m"
# 发送邮件给同事
cd  $export_path
zipName="tempZipName.zip"
zip -r $zipName $export_ipa_path/$ipa_name.ipa
export LANG=C.UTF-8
#$echo <123> | sudo -S <command>
# 修改大文件限制
echo $macPassword | sudo -S postconf -e message_size_limit=0
postconf -d | grep size
# echo message_size_limit
# 压缩包路径
zipPath=$export_path/$zipName
# 邮件标题（使用UTF8编码）
emailTitle="新版测试包"
emailBody="新版测试包，有任何问题请及时与我联系"
( echo $emailBody; uuencode $zipPath ipa.zip) | mail -s $emailTitle xyhuangjia@yeah.net
echo "\033[36;1m发送邮件成功 \033[0m"

```
**注：**如有任何疑问请发邮件到xyhuangjia@yeah.net联系我
