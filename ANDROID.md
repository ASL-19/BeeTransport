# ShapeshifterAndroidKotlin

ShapeshifterAndroidKotlin is a wrapper for Shadowsocks that makes it available as a Pluggable Transport for Android apps. Shadowsocks is a fast, free, and open-source encrypted proxy project, used to circumvent Internet censorship by utilizing a simple, but effective encryption.

The Shadow transport shapes network traffic to resemble that of the popular shadowsocks proxy tool, but what does that mean? Shadowsocks keeps traffic from being blocked by an adversary by encrypting it. In the terminology of Pluggable Transports, this is known as “scrambling”.

Refer to main branch of the latest code for ShapeshifterAndroidKotlin library - https://github.com/OperatorFoundation/ShapeshifterAndroidKotlin

The latest version that is integrated to the Gershad App is 3.2.2 - https://github.com/OperatorFoundation/ShapeshifterAndroidKotlin/releases/tag/3.2.2

Note: The Shapeshifter version 3.2.2 works only on Android versions 11 (API level 30) and above. It has been tested on Android 11 and 12. Currently it throws SocketException 'Socket Closed' on Android versions less than 11. 
## Setting up dependencies

1) add the following at the end of repositories in your PROJECT's build.gradle:
```
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
}
```

2) add the dependency in your MODULE's build.gradle:
```
dependencies {
        // Be sure to replace TAG with the most recent version
        implementation 'com.github.OperatorFoundation:ShapeshifterAndroidKotlin:TAG'

        // Later releases of bouncycastle may not work with ShapeshifterAndroidKotlin
        implementation 'org.bouncycastle:bcpkix-jdk15on:1.58'

        implementation 'com.google.code.gson:gson:2.8.2'
        implementation 'com.google.guava:guava:31.0.1-android'
}
```

3) Kotlin version in your PROJECT's build.gradle should be at least what defined in the ShapeshifterAndroidKotlin/build.gradle, otherwise you might get a compile error.

4) Make sure the min SDK in your build.gradle is 16 or higher and targetSdkVersion is same or higher than that defined in ShapeshifterAndroidKotlin/Shadow/build.gradle for each of your PROJECT's build.gradle.

5) The Android Build Tools declared in PROJECT's build.gradle should be at least 7.1.0 version:
```
buildscript {
        ....
        dependencies {
            ....
            classpath 'com.android.tools.build:gradle:7.1.0'
        }
}
```

In your PROJECT's gradle-wrapper.properties, use compatible gradle version as specified here - https://developer.android.com/studio/releases/gradle-plugin#updating-gradle
```
distributionUrl=https\://services.gradle.org/distributions/gradle-7.2-bin.zip
```

6) You will need at least Java 11 Jdk assigned to your JAVA_HOME

```
$export JAVA_HOME={Path to Java 11 Jdk}/Contents/Home
```

## Using the Library

1) Create a shadow config, putting the password and cipher name.  If you're using DarkStar, put the Server's Persistent Public Key in place of the password.
```
val config = ShadowConfig(password, cipherName)
```

2) Make a Shadow Socket with the config, the host, and the port.
```
val shadowSocket = ShadowSocket(config, host, port)
```

3) Get the output stream and write some bytes.
```
shadowSocket.outputStream.write(textBytes)
```

4) Flush the output stream.
```
shadowSocket.outputStream.flush()
```

5) Get the input stream and read some bytes into an empty buffer.
```
shadowSocket.inputStream.read(buffer)
```

## Using the Library with OkHttp - The following code exists in ShadowSocketTest.kt
```
    @Test
    fun okhttpTestServer() {
        val config = ShadowConfig("enter the password here", "DarkStar")
        val client: OkHttpClient.Builder = OkHttpClient.Builder()
            .connectTimeout(15000, java.util.concurrent.TimeUnit.MILLISECONDS)
            .readTimeout(15000, java.util.concurrent.TimeUnit.MILLISECONDS)
            .writeTimeout(15000, java.util.concurrent.TimeUnit.MILLISECONDS)
        val okHttpClient = client.socketFactory(ShadowSocketFactory(config, "enter the proxy host here", 1234)).build()

        val request = Request.Builder()
            .url("https://")
            .build()

        okHttpClient.newCall(request).execute().use { response ->
            if (!response.isSuccessful) throw IOException("Unexpected code $response")

            for ((name, value) in response.headers) {
                println("$name: $value")
            }
            val body = response.body!!.string().trim()
            println(body)
        }
    }
```

## Using the library with Retrofit & OkHttp

```
        // You can assign the okHttpClient created in the previous topic on 'Using the Library with OkHttp' to the Retrofit.Builder as following:
        
        val apiService = Retrofit.Builder()
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .addConverterFactory(GsonConverterFactory.create())
                .baseUrl(URL)
                .client(okHttpClient)
                .build()
                .create(ApiService::class.java)
        
        // Now you may use apiService variable and call funstions defined in your APIService.java
```
