---
layout: post
title: 안드로이드를 위한 Gradle
date: 2017-04-13
img: post-cover/gradle-for-android.png
tags: [books, review, gradle, android, til, study]
---

# 궁금했지만 항상 그냥 지나쳤던 안드로이드에서의 Gradle, 얉게 한번 파보자. ~~사실 별로 궁금해하지 않고 그냥 사용 하는 경우가 대다수..~~

## Gradle? 그래들? 그레이들? 일단 발음부터 좀 해결하자

- 책에서는 ‘그레이들’로 발음하는 경우가 많음
- 어차피 한글로 구글링해도 gradle로 검색됨
- 결론: 그래-들
- 빌드 자동화 툴 (친구들: ant, maven, make)

## 빌드란 무엇인가?

> 내가 작성한 코드를 `apk(android package)`, `aar(android archive)`로 만드는 과정

## 안드로이드 기본 프로젝트 구조를 살펴보자

Project 로 보기

![Project](/gradle-for-android/structure_project_small.png)

Android 로 보기

![Android](/gradle-for-android/structure_android_small.png)

---

## Gradle 관련 파일

build.gradle (Project: MyAppliation)

```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.0'
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
```

build.gradle (Module: app)

```gradle
apply plugin: 'com.android.application'

android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"
    defaultConfig {
        applicationId "com.example.ragu.myapplication"
        minSdkVersion 19
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.3.0'
    testCompile 'junit:junit:4.12'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
}
```

gradle.settings

```gradle
include ':app'
```

gradle.properties

```gradle
systemProp.http.proxyHost=70.10.15.10
systemProp.http.proxyPort=8080
systemProp.http.nonProxyHosts=70.*|localhost

systemProp.https.proxyHost=70.10.15.10
systemProp.https.proxyPort=8080
systemProp.https.nonProxyHosts=70.*|localhost
```

---

## 안드로이드 Gradle 플러그인

```gradle
buildscript {
    ...
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.0'
    }
}
```

- 안드로이드 스튜디오에서 Gradle 빌드를 할 수 있게 해주는 기능 제공
- 안드로이드 스튜디오와 함께 버전이 올라가고 있음

## 저장소 비교

```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {...}
}

allprojects {
    repositories {
        jcenter()
    }
}
```

저장소 이름|운영사|속도|규모|안드로이드 스튜디오 기본 저장소|사용자 친화도
-|-|-|-|-|-
jcenter|bintray.com|빠름(CDN 사용)|더큼|현재|높음(쉬움)
manveCentral|sonatype.org|덜빠름|큼|과거|낮음(어려움)

> bintray.com은 `jcenter`에서 `maven central`으로 배포하는 기능을 제공한다.

### 그 외 여러가지 저장소 설정 예

로컬에 있는 메이븐 캐시 사용

```gradle
repositories {
    mavenLocal()
}
```

로컬 파일 시스템에 있는 라이브러리 파일 참조

```gradle
repositories {
    flatDir {
        dirs 'lib'
    }
}
```

기타 메이븐 저장소 url을 직접 지정

```gradle
repositories {
    maven {
        url 'http://repo.mycompony.com/maven'
    }
}
```

비밀번호가 걸려있는 저장소

```gradle
repositories {
    maven {
        credentials {
            username 'username'
            password 'password'
        }
        url 'http://repo.mycompony.com/maven'
    }
}
```

---

## application vs. library

gradle.build(Module: app)

```gradle
apply plugin: 'com.android.application'

...
```


- com.android.application : apk(android package)
- com.android.library : aar(android archive)


## SDK Version

gradle.build(Module: app)

```gradle
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"
    defaultConfig {
        applicationId "com.example.ragu.myapplication"
        minSdkVersion 19
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    ...
}
```

속성 이름|내용
-|-
compileSdkVersion|컴파일에 사용할 SDK 버전
minSdkVersion|지원하는 최소 SDK 버전으로, SDK 버전이 이 값보다 낮은 기기에서는 해당 어플리케이션을 검색할 수 없음.
targetSdkVersion|어플리케이션이 의도하는 목적 SDK 버전. 최신버전보다 낮으면 안드로이드 스튜디오에서 경고 표시됨.

## applicationId(패키지 이름으로 많이 불림)

- 구글 플레이 스토어에서 유일한 이름이어야 한다.
- 한번 마켓에 올라가면 변경되어서는 안된다. -> 만약 변경하게되면 전혀 다른 앱이 된다.

## 빌드 타입 이해하기

gradle.build(Module: app)

```gradle
android {
    ...
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

- debug
- release

> 명시하지 않으면 기본 설정이 적용됨.

---

## 외부 라이브러리 의존성 추가

gradle.build(Module: app)

```gradle
android {...}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.1.0'
    testCompile 'junit:junit:4.12'
}
```

### Configuration

- compile, testCompile, androidTestCompile
- debugCompile, releaseCompile
- annotationProcessor, testAnnotationProcessor

### 라이브러리 위치 지정

- 로컬 파일 시스템 : files('rain.jar', 'scott.jar', ...) or fileTree(dir: 'libDir', include: ['*.jar'])
  > 실무에서는 외부에 공개된 라이브러리가 아닌 특정 회사에만 공개된 private 라이브러리를 참조하는 경우에만 사용한다. by 안드로이드를 위한 Gradle(저자: 유동환)
- 외부 저장소 : group: 'group', name: 'name', veresion: 'ver.' or 'group:name:version'
  - ex) group: 'com.android.support', name: 'appcompat-v7', version: '25.1.0'
- android library : 'group:name:version@aar'
- version 명시 : 'junit:junit:4.+' 와 같은 방법은 권장되지 않음.

---

## 자, 이제 Gradle을 이용해서 프로젝트를 빌드 해 보자

> 그래 위에 설명들은 알았다 치고, 어떻게 빌드하는거지?

### 첫번째 방법

![run](/gradle-for-android/run.png)

- 딱히 신경 안쓰고 걍 Run

![build](/gradle-for-android/build.png)

> 뭔가 내가 작성한게 적용이 안된것만 같은 느낌...

- [build]-[Clean Project]
- [build]-[Rebuild Project]

### 두번째 방법

안드로이드 Gradle 플러그인을 이용

![android gradle plugin](/gradle-for-android/android_gradle_plugin.png)

원하는 Task 더블 클릭

### 세번째 방법

쉘에서 직접 Gradle Task 실행하기

```shell
> ./gradlew clean
> ./gradlew test
> ./gradlew clean assembleDebug
> ./gradlew assemble
> ./gradlew build
```

### Gradle 내장 Task에 대한 자세한 설명은 생략하나, 기본적인 Task는 한번만 보고가자

#### Java 플러그인의 태스크 의존 관계(DAG; directed acyclic graph)

![Java Plugin DAG](/gradle-for-android/java_plugin_dag.png)

![Diagram of Tasks](/gradle-for-android/tasks_diagram.jpg)

~~뭘 좋아하는지 몰라서 다 준비해봤어~~

> 특정 Task를 제외하고 빌드하는 방법

```shell
> ./gradlew build -x check
```

### Lint 란 무엇인가? feat. check = test + lint

사전적 의미 -> 보푸라기

의심스럽거나, 에러를 발생하기 쉬운 코드에 flag 를 달아 놓는 것

Android Lint에서 검사하는 항목 중 대표적인 부분

- 사용하지 않는 리소스
- 국제화(I18N; Internationalization) 지원시 이상 여부
- 성능상 문제가 될 수 있는 부분
- minSdkLevel, targetSdkLevel, compileSdkLevel에 따라 오류가 발생할 수 있는 부분

을 빌드 시점에 알려준다.

---

## Product Flavors

![flavor](/gradle-for-android/flavor.png)

마켓별로 다른 package, version code, version name, sdk version 등을 따로 관리하고 싶은 경우 Gradle 의 `flavor` 기능을 이용하면 쉽게 적용을 할 수 있다.

#### 사용 예)

build.gradle(module: app)
```gradle
android {
    ...
    defaultConfig {...}
    buildTypes {...}
    productFlavors {
        demo {
            applicationId "io.github.ragubyun.myapplication.demo"
            versionName "1.0-demo"
        }
        full {
            applicationId "io.github.ragubyun.myapplication.full"
            versionName "1.0-full"
        }
    }
    sourceSets {
        demo {
            java.srcDirs = ['src/main/java', 'src/demo/java']
            res.srcDirs = ['src/main/res', 'src/demo/res']
        }
        full {
            java.srcDirs = ['src/main/java', 'src/full/java']
            res.srcDirs = ['src/main/res', 'src/full/res']
        }
    }
}
```

프로젝트 폴더 구조

![source sets](/gradle-for-android/source_sets.png)

Build Variants

![Build Variants](/gradle-for-android/build_variants.png)

- 사용자별 구분
- 국가별 구분
- 사용 환경에 따른 구분
- ...

> `applicationId`를 다르게 가져가므로 사실상 **완전히 별개의 앱**이라고 볼 수 있다.

> **중요!** Android Library Project는 `productFlavor`를 지원하지 않는다.

---

### Product Flavor in Looky

build.gradle(Module: app)

```gradle
android {
    ...
    productFlavors {
        internal {
            buildConfigField 'String', 'proxyHost', '"' + PROXY_HOST + '"'
            buildConfigField 'int', 'proxyPort', PROXY_PORT
            buildConfigField 'String', 'certKeystoreDigest', '"' + CERT_KEYSTORE_DIGEST + '"'
        }
        production {
            buildConfigField 'String', 'proxyHost', 'null'
            buildConfigField 'int', 'proxyPort', '0'
            buildConfigField 'String', 'certKeystoreDigest', 'null'
        }
    }
}
```

```groovy
buildConfigField 'type', 'name', 'value'
buildConfigField(T type, String name, String value)
```

gradle.properties

```gradle
PROXY_HOST=70.10.15.10
PROXY_PORT=8080
CERT_KEYSTORE_DIGEST=AAAAAgAAABS.......nBLz3uzNP05V3c7
```

Build Variants

![Build Variants Looky](/gradle-for-android/build_variants_looky.png)

NetModules.java

```java
@Provides
@Singleton
public OkHttpClient httpClient() {
    OkHttpClient.Builder clientBuilder = new OkHttpClient.Builder()
            .connectTimeout(1, TimeUnit.MINUTES)
            .readTimeout(1, TimeUnit.MINUTES)
            .writeTimeout(1, TimeUnit.MINUTES);

    if (isNotUnitTest() && BuildConfig.proxyHost != null) {
        Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress(BuildConfig.proxyHost, BuildConfig.proxyPort));
        clientBuilder.proxy(proxy);

        try {
            byte[] keyStoreBytes = Base64.decode(BuildConfig.certKeystoreDigest, Base64.DEFAULT);
            ByteArrayInputStream keyStore = new ByteArrayInputStream(keyStoreBytes);
            KeyStore keystore = KeyStore.getInstance("BKS");
            keystore.load(keyStore, "changeit".toCharArray());

            KeyManagerFactory kmf = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
            kmf.init(keystore, "changeit".toCharArray());

            TrustManagerFactory tmf = TrustManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
            tmf.init(keystore);

            SSLContext sslContext = SSLContext.getInstance("TLS");
            sslContext.init(kmf.getKeyManagers(), tmf.getTrustManagers(), null);
            SSLSocketFactory socketFactory = sslContext.getSocketFactory();

            clientBuilder.sslSocketFactory(socketFactory);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    return clientBuilder.build();
}
```

#### Build Variants Matrix

Build Type x Product Flavor

- buildType : debug, release
- productFlavor : demo, full
- build variants : demoDebug, demoRelease, fullDebug, fullRelease

---

## 부록 A : Custom Task

gradle.build(Project: MyApplication)
```groovy
buildscript {...}
allprojects {...}

task clean(type: Delete) {
    delete rootProject.buildDir
}

task sayHello(group: 'custom',
    description: 'This task says Hello',
    dependsOn: clean) << {
    println 'Hello Gradle!'
}
```

![Custom Task](/gradle-for-android/custom_task.png)

![Run Custom Task](/gradle-for-android/run_custom_task.png)

---

## 부록 B : support-v4 vs. appcompat-v7

### Support Library

> 이전 안드로이드 버전을 사용하고 있는 기기에서 새로운 API를 사용 할수 있도록 도와주는 호환성 라이브러리이며 새로운 안드로이드 버전이 나올때 마다 업데이트 된다. 주요 호환성 라이브러리로는 V4및 V7-AppCompat이 있다.

- support-v4

  > 안드로이드 API4부터 사용이 가능한 라이브러리로 API 11에서 소개된 Fragment와 Loader등 주요 클래스의 구현을 지원하며 ViewPager, DrawerLayout등 포함 되어 있다.

- appcompat-v7

  > V4를 이용하여 확장한 라이브러리로 단순히 액션바(API 11), 툴바(API 21)등을 지원한다.

안드로이드 스튜디오의 새로운 프로젝트 템플릿을 보면 support-v4와 appcompat-v7을 기본적으로 포함시켜 개발자로 하여금 의존할 수 있도록 노력하고있다. 즉, 서포트 라이브러리가 필요 하지 않는 경우라도 일반적으로 사용하기를 구글에서는 권하고 있다. by [꿈꾸는 개발자의 로그 블로그](http://www.kmshack.kr/2015/06/android-support-library/)

---

## 부록 C : 책 소개

![책 3권](/gradle-for-android/books.jpeg)

~~책 소개를 가장한 노트북 자랑질...~~

![안드로이드를 위한 Gradle](/gradle-for-android/book1_small.jpg)

- 110 페이지
- 국내도서
- 2016년 7월
- 안드로이드 특화
- 이틀
- 일단 얇아서 좋고, 안드로이드에서 사용하는 Gradle을 전반적으로 빠르게 훑어볼 수 있어서 더 좋다.

![Gradle 철저 입문](/gradle-for-android/book2_small.jpg)

- 620 페이지
- 번역서(일본)
- 2015년 12월
- 안드로이드 특화 아님
- 하루

![그레이들 레시비](/gradle-for-android/book3_small.jpg)

- 171 페이지
- 번역서(미쿡)
- 2016년 12월
- 안드로이드 특화
- 일주일째 보는 중
- 발생할 수 있는 문제 혹은 하고싶은 상황을 만들어 놓고 해결책을 제시해 가면서 점점 자세한 설명으로 들어가는 형태로 되어있어서 지겹지 않음.

끝
