## Cordova插件开发的一般顺序
1. 安装npm（百度）
2. 创建插件：plugman create --name 插件名字 --plugin_id 插件id --plugin_version 插件版本
3. 给插件添加安卓版本的实现代码：plugman platform add --platform_name android
4. 在插件根目录新建package.json文件，也可以通过命令行添加，我觉得麻烦，需要配置的无用信息太多
    ```java
    {
    "name": "插件名字",
    "version": "1.0.0",
    "description": "demo",
    "cordova": {
        "id": "插件id",
        "platforms": [
        "android"
        ]
    },
    "keywords": [],
    "author": "francis",
    "license": "MIT"
    }
    ```
5. 编写平台代码
6. 根据需要配置plugin.xml
   ```xml
   <?xml version='1.0' encoding='utf-8'?>
        <plugin id="com.plugin.MyVpnPlugin" version="1.0.0" xmlns="http://apache.org/cordova/ns/plugins/1.0" xmlns:android="http://schemas.android.com/apk/res/android">
        <name>MyVpnPlugin</name>
        <js-module name="MyVpnPlugin" src="www/MyVpnPlugin.js">
            <clobbers target="cordova.plugins.MyVpnPlugin" />
        </js-module>
        <platform name="android">
            <config-file parent="/*" target="res/xml/config.xml">
                <feature name="MyVpnPlugin">
                    <param name="android-package" value="com.plugin.vpn.MyVpnPlugin" />
                </feature>
            </config-file>
            <config-file parent="/*" target="AndroidManifest.xml"/>
            <config-file parent="/*" target="AndroidManifest.xml">
                <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
            </config-file>

            <config-file target="AndroidManifest.xml" parent="/manifest/application">
                <service android:name="com.plugin.vpn.core.LocalVpnService" android:permission="android.permission.BIND_VPN_SERVICE">
                    <intent-filter>
                        <action android:name="android.net.VpnService" />
                    </intent-filter>
                </service>
            </config-file>
            <source-file src="src/android/vpn" target-dir="src/main/java/com/plugin/vpn" />
            <framework src="io.apisense:rhino-android:1.1.1" />
            <framework src="org.bouncycastle:bcprov-jdk15on:1.57" />
            <framework src="com.android.support:appcompat-v7:28.0.0" />
        </platform>
    </plugin>
    ```
7. 新建cordova项目，cordova create VpnProject com.example.myvpn VpnProject 
8. 添加和删除插件： cordova plugin remove com.example.myvpnlibrary
                cordova plugin add C:\Users\francis.fan\MyVpnPlugin

## 一些细节问题  
