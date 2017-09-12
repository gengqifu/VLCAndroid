# VLCAndroid
VLCAndroid in Android Studio

著名的开源播放器VLC Android版本。

1. 创建一个新的Android Studio项目VLCAndroid

2. 删除新工程中src/main/Java中所有文件和src/main/res文件夹，复制vlc-android/vlc-android/src中文件夹到src/main/java，复制vlc-android/vlc-android/res 和AndroidManifest.xml 到src/mian

3. 复制vlc-android/vlc-android/libvlc/build/output/aar/libvlc-3.0.0-2.1.0.aar到libs,

复制vlc-android/api/build/output/aar/api-release.aar到libs,

复制vlc-android/assets和flavors到app

4. build.gradle(project:VLC-Android):


6.// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.5.0'
        classpath 'com.jakewharton.sdkmanager:gradle-plugin:0.12.+'
        classpath 'com.android.databinding:dataBinder:1.0-rc4'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

5. Build.gradle(module:app):

apply plugin: 'com.android.application'
apply plugin: 'com.android.databinding'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"

    flavorDimensions "target", "abi"

    lintOptions {
        abortOnError false
        disable 'MissingTranslation', 'ExtraTranslation'
    }

    defaultConfig {
        applicationId "org.videolan.vlc"

        resValue "string", "build_time", buildTime()
        resValue "string", "build_host", hostName()
        resValue "string", "build_revision", revision()

    }

    buildTypes {
        release {
            minifyEnabled true
            shrinkResources false
            proguardFile 'proguard.cfg'
        }
        debug {
            applicationIdSuffix ".debug"
            jniDebuggable true
        }
    }
    productFlavors {
        vanilla {
            dimension "target"
            versionCode = 1
        }
//        chrome {
//            minSdkVersion 19
//            dimension "target"
//            versionCode = 2
//        }
        ARMv7 {
            dimension "abi"
            versionCode = 4
        }

    }
    // make per-variant version code
    applicationVariants.all { variant ->
        def manifestParser = new com.android.builder.core.DefaultManifestParser()
        // get the version code of each flavor
        def vlcVersion = manifestParser.getVersionName(android.sourceSets.main.manifest.srcFile)
        def targetVersion = variant.productFlavors.get(0).versionCode
        def abiVersion = variant.productFlavors.get(1).versionCode

        // set the composite code
        variant.mergedFlavor.versionCode = targetVersion * 10000000 + manifestParser.getVersionCode(android.sourceSets.main.manifest.srcFile) + abiVersion
        variant.mergedFlavor.versionName = vlcVersion

        //Custom APK name
        variant.outputs.each { output ->
            def outputName = "VLC-Android-"
            if (!variant.productFlavors.get(0).name.equals("vanilla"))
                outputName += variant.productFlavors.get(0).name.toUpperCase() + "-"
            outputName += vlcVersion + "-" + variant.productFlavors.get(1).name + ".apk"
            output.outputFile = new File(output.outputFile.parentFile, outputName);

            //set intents with correct package name
            output.processManifest.doLast{
                def manifestOutFile = output.processManifest.manifestOutputFile
                def newFileContents = manifestOutFile.getText('UTF-8').replace("_PACKAGENAME_", variant.applicationId)
                manifestOutFile.write(newFileContents, 'UTF-8')
            }
        }
    }

    sourceSets {
        main{
            manifest.srcFile 'src/main/AndroidManifest.xml'
            java.srcDirs = ['src/main/java', 'src/main/java']
            resources.srcDirs = ['src/main/java', 'src/main/java']
            aidl.srcDirs = ['src/main/java']
            res.srcDirs = ['src/main/res']
            assets.srcDirs = ['assets']
        }
//        release {
//            manifest.srcFile 'flavors/release/AndroidManifest.xml'
//        }
//        chrome{
//            Manifest.srcFile 'flavors/chrome/AndroidManifest.xml'
//            res.srcDirs = ['flavors/chrome/res']
//        }
    }


}
repositories {
    flatDir {
        dirs 'libs'
    }
}
dependencies {
    compile(name: 'libvlc-3.0.0-2.1.0', ext: 'aar')
    compile(name: 'api-release', ext: 'aar')
    compile 'com.android.support:appcompat-v7:23.1.1'
    compile 'com.android.support:cardview-v7:23.1.1'
    compile 'com.android.support:recyclerview-v7:23.1.1'
    compile 'com.android.support:design:23.1.1'
    compile 'com.android.support:support-annotations:23.1.1'
    compile 'com.android.support:preference-v7:23.1.1'
    compile 'com.android.support:percent:23.1.1'
    compile 'com.android.support:leanback-v17:23.1.1'
    compile 'com.android.support:preference-leanback-v17:23.1.1'
    compile files("libs/axmlrpc.jar")
    testCompile 'junit:junit:4.12'
}

def buildTime() {
    return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC"))
}

def hostName() {
    return System.getProperty("user.name") + "@" + InetAddress.localHost.hostName
}

def revision() {
//    def code = new ByteArrayOutputStream()
//    exec {
//        commandLine 'git', 'rev-parse', '--short', 'HEAD'
//        standardOutput = code
//    }
//    return code.toString()
    return "2237092"
}


6. 编译错误的处理：
layout文件中有类似这样的代码：
android:onClick="@{holder::onMoreClick}"
会被视为错误，将其修改为
android:onClick="@{holder.onMoreClick}"
的格式就可以了。
