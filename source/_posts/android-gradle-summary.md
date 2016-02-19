title: gradle在android中的用法概要
date: 2015-11-30 09:44:48
tags: [gradle]
categories: [android]
toc: true
---
#### project中的build.gradle
- buildscript: 配置驱动build的代码，声明在Maven中央仓库，取classpath dependency，即android plugin for gradle xxx
- apply plugin：指明用到的plugin是android，就像在前面java程序中，用的plugin是java一样。
- android{...} 中配置了所有android构建的参数，这里也是android dsl的入口点。
- repositories：配置依赖管理的服务器，默认jcenter(),可以添加其他服务器，如MavenCenter().
- dependencies 指具体依赖什么库，用classpath指明
- 依赖jcenter服务器中的gradle库，包名："com.android.tools.build",版本x.x.x

<!-- more -->

		buildscript {
			repositories {
				jcenter()
			}
			dependencies {
				classpath 'com.android.tools.build:gradle:1.3.0'
			}
		}

#### model中的build.gradle
##### 基本配置说明
- apply plugin:'com.android.application' ，表示添加插件，application标识这个Model为应用的主程序，如果Model为一个库，那么apply plugin:'com.android.library'
- dependencies: 项目依赖，可以远程依赖，亦可以本地依赖。

		dependencies {
		    compile fileTree(dir: 'libs', include: ['*.jar'])
		    testCompile 'junit:junit:4.12'
		    compile 'com.android.support:appcompat-v7:23.1.0'
		    compile 'com.android.support:design:23.1.0'
		    compile project(':library')
		}
	
	- comple fileTree(dir: 'libs', include: ['*.jar'])表明编译时依赖libs文件下所有jar文件
	- compile project(':library')表明依赖本地名为library的Model库
	- compile 'com.android.support:design:23.1.0' 表明依赖远程库

##### android{...}中的配置
	android {
		complieSdkVersion xx
		buildToolsVersion xx
		
		defaultConfig {	
		}
		
		buildTypes {
		}
		
		compileOptions {
		}
		
		sourceSets {
		}
		
		lintOptions {
		}
		
		productFlavors {
			flavors1 {
			}
			flavors1 {
			}
		}
		
		signingConfigs {
			release {
				storeFile file("x.keystore")
				storePassword "xxx"
				keyAlias "xxx"
				keyPassword "xxx"
			}
			debug {
			}
		}
	}
- defaultConfig: 默认配置，相当于全局配置，buildType自动继承

		  defaultConfig {
			  applicationId "com.xxx"  (非必须配置，库类型的model无此配置)
			  minSdkVersion 14
			  targetSdkVersion 23
			  versionCode 1
			  versionName "1.0"	
		  }
- buildType：编译配置，有release 亦有debug,配置相同

		 release {
	            buildConfigField "boolean", "LOG_DEBUG", "false"
	            minifyEnabled false
	            shrinkResources true
	            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	            signingConfig signingConfigs.release
	        }
	     debug {
	         	  buildConfigField "boolean", "LOG_DEBUG", "true"
	            debuggable true
	            shrinkResources true
	            signingConfig signingConfigs.release
	        }

- compileOptions：编译选项，进行java版本配置

		compileOptions {
			sourceCompatibility JavaVersion.VERSION_1_7
			targetCompatibility JavaVersion.VERSION_1_7
		}
- sourceSets：源码设置，多为Ecipse中迁移过来的代码

		sourceSets {
			main {
				manifest.srcFile 'AndroidManifest.xml'
				java.srcDirs = ['src']
				resource.srcDirs = ['src']
				aidl.srcDirs = ['src']
				renderscript.srcDirs = ['src']
				res.srcDirs = ['res']
				assets.srcDirs = ['assets']
				jniLibs.srcDirs = ['libs']  （引用“.so”文件式的用法）
			}
		}

- lintOptions：设置编译的lint开关。程序在build时执行lint检查，有错误则停止构建。设置abortOnError false 关掉
- productFlavors：发布到不同渠道，且不同渠道中的包名不同，如下配置。也可以设置不同的AndroidManifest.xml文件。
	
		productFlavors {
			flavor1 {
				packageName='com.example.application1'
				manifest.srcFile 'exampleapk/AndroidManifest1.xml'
			}
			flavor2 {
				packageName='com.example.application2'
				manifest.srcFile 'exampleapk/AndroidManifest2.xml'
			}
		}
- signingConfigs——包签名的配置，可以配置具体的签名文件，签名密码等，可不用自己创建，点击build/generate signed apk，创建或选择签名文件，设置并记住密码。一般用读取配置文件的方式。

		Properties props = new Properties()
		props.load(new FileInputStream(file("signing.properties")))
		......
		signingConfigs {
	        release {
	            keyAlias props['KEY_ALIAS']
	            keyPassword props['KEY_PASSWORD']
	            storeFile file(props['KEYSTORE_FILE'])
	            storePassword props['KEYSTORE_PASSWORD']
	        }
	    }