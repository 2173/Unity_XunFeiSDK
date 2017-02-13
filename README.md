# Unity_XunFeiSDK
使用Unity5.x和AndroidStudio，为安卓添加语音识别。

版权声明：本文为孙思谋原创文章，未经允许不得转载
如果能找回微博账号，我会把它分享到微博上，再此先做备份。

网上大多是unity5.0以下和eclipse的教学。走了不少弯路。现在特地将在AndroidStudio和Unity5.0+环境下讯飞SDK接入的放法分享一下。
首先获取讯飞SDK的方法就不再多说了，网上随意搜一下就有很多。

获取APPID并且下载完SDK后，我们开始以下步骤：
1、AS新建工程，EmptyActivity，这样省了在AndroidManifest添加权限的步骤，同时可以删除layout文件夹中的布局文件，因为导入到unity中用不到。

2、File-New-New Module，新建AndroidLibrary，命名随意，作者在这里命名为speechrecognizer2,。

3、将SDK文件夹中的MSC.jar考到libs文件夹下。在main文件夹下新建文件夹jinLibs，将SDK中的so文件考进来，如图：
Image
4、在Unity安装目录下找到class.jar文件。同样，考到libs文件夹下。路径：
\Unity\Editor\Data\PlaybackEngines\AndroidPlayer\Variations\mono\Release\Classes

5、此时需要在工程中关联jar。File-Project Structure。左侧选中步骤二中新建的Library。点击右方加号，选择Files dependency。将步骤3、4中加入的jar包关联到module。如图：
Image

6、环境搭建完成，我们可以开心的写代码了。
MainActivity中的代码和网上其他的方法大同小异，直接上源码：
package com.ssm.ssm.speechrecognizer;

        import android.os.Bundle;
        import android.util.Log;
        import android.widget.Toast;

        import com.iflytek.cloud.InitListener;
        import com.iflytek.cloud.RecognizerListener;
        import com.iflytek.cloud.RecognizerResult;
        import com.iflytek.cloud.SpeechConstant;
        import com.iflytek.cloud.SpeechError;
        import com.iflytek.cloud.SpeechUtility;
        import com.iflytek.cloud.SpeechRecognizer;

        import com.unity3d.player.UnityPlayer;
        import com.unity3d.player.UnityPlayerActivity;

        import org.json.JSONArray;
        import org.json.JSONObject;
        import org.json.JSONTokener;

public class MainActivity extends UnityPlayerActivity {

    public SpeechRecognizer speechRecognizer;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        //注意这里的appid为讯飞官网上注册获得的appid
        SpeechUtility.createUtility(getApplicationContext(),"appid=XXXXXXXXX");

        initRecognizer();
    }

    //初始化
    private void initRecognizer(){
        speechRecognizer = SpeechRecognizer.createRecognizer(getApplicationContext(),mInitListener);
    }

    public InitListener mInitListener = new InitListener() {
        @Override
        public void onInit(int i) {
            UnityPlayer.UnitySendMessage("Manager", "Result", "init success!");
        }
    };

    //开始听写
    public void startSpeechListener(){

        UnityPlayer.UnitySendMessage("Manager", "Result", "startListening");

        speechRecognizer.setParameter(SpeechConstant.DOMAIN, "iat");
        speechRecognizer.setParameter(SpeechConstant.LANGUAGE, "zh_cn");
        speechRecognizer.setParameter(SpeechConstant.ACCENT, "mandarin");
        speechRecognizer.startListening(mRecognizerListener);
    }

    public RecognizerListener mRecognizerListener = new RecognizerListener(){

        @Override
        public void onBeginOfSpeech() {
            // TODO Auto-generated method stub
            //UnityPlayer.UnitySendMessage("Manager", "Result", "onBeginOfSpeech");
        }

        @Override
        public void onEndOfSpeech() {
            // TODO Auto-generated method stub
            //UnityPlayer.UnitySendMessage("Manager", "Result", "onEndOfSpeech");
        }

        @Override
        public void onError(SpeechError arg0) {
            // TODO Auto-generated method stub
            //UnityPlayer.UnitySendMessage("Manager", "Result", "onError");
        }

        @Override
        public void onEvent(int arg0, int arg1, int arg2, Bundle arg3) {
            // TODO Auto-generated method stub
            //UnityPlayer.UnitySendMessage("Manager", "Result", "onEvent");
        }

        @Override
        public void onResult(RecognizerResult recognizerResult, boolean isLast) {
            //UnityPlayer.UnitySendMessage("Manager", "Result", "listener");
            printResult(recognizerResult);
        }

        @Override
        public void onVolumeChanged(int arg0, byte[] arg1) {
            //UnityPlayer.UnitySendMessage("Manager", "Result", "onVolumeChanged");
            // TODO Auto-generated method stub
        }
    };

    //解析
    private void printResult(RecognizerResult results) {
        String json = results.getResultString();

        StringBuffer ret = new StringBuffer();
        try {
            JSONTokener tokener = new JSONTokener(json);
            JSONObject joResult = new JSONObject(tokener);

            JSONArray words = joResult.getJSONArray("ws");
            for (int i = 0; i < words.length(); i++) {
                // 转写结果词，默认使用第一个结果
                JSONArray items = words.getJSONObject(i).getJSONArray("cw");
                JSONObject obj = items.getJSONObject(0);
                ret.append(obj.getString("w"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        //将解析结果“"result:" + ret.toString()”发送至“Manager”这个GameObject，中的“Result”函数
        UnityPlayer.UnitySendMessage("Manager", "Result", "result:" + ret.toString());
    }
}
AndroidManifest中添加权限，同样，和其他教程大同小异，源码：
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.ssm.ssm.speechrecognizer">

    <application
        android:allowBackup="true"
        android:label="@string/app_name"
        android:supportsRtl="true">
        <activity android:name=".MainActivity"
            android:label="@string/app_name">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

    <!--连接网络权限,用于执行云端语音能力 -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <!--获取手机录音机使用权限,听写、识别、语义理解需要用到此权限 -->
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <!--读取网络信息状态 -->
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <!--获取当前wifi状态 -->
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <!--允许程序改变网络连接状态 -->
    <uses-permission android:name="android.permission.CHANGE_NETWORK_STATE"/>
    <!--读取手机信息权限 -->
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <!--读取联系人权限,上传联系人需要用到此权限 -->
    <uses-permission android:name="android.permission.READ_CONTACTS"/>
    <!--外存储写权限,构建语法需要用到此权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <!--外存储读权限,构建语法需要用到此权限 -->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <!--配置权限,用来记录应用配置信息 -->
    <uses-permission android:name="android.permission.WRITE_SETTINGS"/>
    <!--摄相头权限,拍照需要用到 -->
    <uses-permission android:name="android.permission.CAMERA" />

</manifest>

此时，作为AS，为了导出jar，需要在build.gradle中添加以下两段代码：
task makeJar(type: Copy) {
    delete 'build/libs/speechrecognizer.jar'
    from('build/intermediates/bundles/release/')
    into('build/libs/')
    include('classes.jar')
    rename ('classes.jar', 'speechrecognizer.jar')
}

makeJar.dependsOn(build)

// 在终端执行生成JAR包
// gradlew makeJar

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile files('libs/classes.jar')
}


build.gradle源码为：
apply plugin: 'com.android.library'

android {
    compileSdkVersion 24
    buildToolsVersion "24.0.2"

    defaultConfig {
        minSdkVersion 15
        targetSdkVersion 24
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
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:24.2.1'
    testCompile 'junit:junit:4.12'
    compile files('libs/Msc.jar')
    compile files('libs/classes.jar')
}
task makeJar(type: Copy) {
    delete 'build/libs/speechrecognizer.jar'
    from('build/intermediates/bundles/release/')
    into('build/libs/')
    include('classes.jar')
    rename ('classes.jar', 'speechrecognizer.jar')
}

makeJar.dependsOn(build)

// 在终端执行生成JAR包
// gradlew makeJar

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile files('libs/classes.jar')
}

7、至此，架包已经编写完成。现在导出jar。在Terminal中输入 gradlew makejar。如图：
Image

8、等待完成后即可在libs下看到jar文件：
Image

8、新建Unity项目，在Asset文件夹下新建如下结构：Image

9、如步骤8，在bin文件夹下拷贝AS导出的jar。libs文件夹下拷贝SDK中的MSC.jar，和os文件，注意，安卓5.0以上系统需要armeabi-v7a，不然会出现21002的错误。最后在Android文件夹下拷贝AndroidManifest文件。

10、至此，已经成功的将讯飞导入至Unity中，在Unity中编写代码调用即可。
源码：
public class XunFeiTest : MonoBehaviour
{
    private string showResult = "";

    void Start ()
    {  

    }

    void OnGUI ()
    { 
        if (GUILayout.Button ("startRecognizer", GUILayout.Height (100))) {  
            AndroidJavaClass jc = new AndroidJavaClass ("com.unity3d.player.UnityPlayer");  
            AndroidJavaObject jo = jc.GetStatic<AndroidJavaObject> ("currentActivity"); 
            jo.Call ("startSpeechListener");  
        }  

        GUILayout.TextArea (show + showResult, GUILayout.Width (200));  

    }

    public void Result (string recognizerResult)
    {  
        showResult += recognizerResult;  
    }
}   

11、demo编写完成，导出apk试一下吧！注意尽量使用真机测试。作者在用AS测试讯飞听写功能时就遇到模拟器无法获得权限的坑。


