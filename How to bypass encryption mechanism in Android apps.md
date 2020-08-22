## How to bypass encryption mechanism in Android apps

Hi Folks, hope you are well. As you know developers and pentesters are always into a cat and mouse game. No matter how much we want to deny the fact but we make each other's life a little tough. Mobile application developers these days implement an encryption mechanism in their applications to prevent tampering. 

However, there are ways to bypass such mechanism. I am going to demonstrate a scenario which can be useful when next time you come across such application behaviour.

We are going to use Injured Android, [Injured Android](https://github.com/B3nac/InjuredAndroid) is a CTF style vulnerable application built to practice android testing. Huge shoutout to the author [B3nac](https://twitter.com/b3nac) for building this application.

## Why is it worth reading?

I have came across many application which highly rely on client side encryption to add an extra layer of security for us to uncover. When you intercept the traffic using your favourite proxy tool all you'll see is encrypted text. It is because all of the requests are being encrypted on the client side and sent to server. The server will have same encryption algorithm coded, so upon receiving the data it will decrypt and fetch the values.

If you are into android testing or want to get started this article is for you.

## Installation

Download the latest build from [here](https://github.com/B3nac/InjuredAndroid/releases). I am using [genymotion](https://www.genymotion.com/) for this article. Spin up a virtual device using genymotion and install the APK by entering following command.

`adb install injuredandroid.apk`

After successful installation you'll see following screen.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-15%2018-03-56.png "Home screen")

## Identification

We are interested in **FLAG SIX - LOGIN 3**. This is how our FlagSix screen looks like, it has a single textbox and a **SUBMIT** button.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-15%2018-15-09.png "Flag six activity")

I have tried entering random text and clicked on submit but nothing happened. Our next step is to look at the java source code to understand what is the functionality of this activity. 

Let's decompile the APK to fetch the application source code. I use [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) for the sake of simplicity, it makes the task a lot easier. Below is the synopsis of our APK from MobSF.


![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-15%2018-35-40.png "MobSF output")

There are whole bunch of [activities](https://www.tutorialspoint.com/android/android_acitivities.htm) to look at. **FlagSixLoginActivity** seems to be our destination.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-15%2018-40-40.png "Activity List")

## Developer Mode On

Let's have a look at the code.

```java
package b3nac.injuredandroid;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.EditText;
import androidx.appcompat.app.c;
import androidx.appcompat.widget.Toolbar;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
import com.google.android.material.snackbar.Snackbar;
import d.m.b.d;

public final class FlagSixLoginActivity extends c {
    private int t;

    static final class a implements View.OnClickListener {

        /* renamed from: c  reason: collision with root package name */
        final /* synthetic */ FlagSixLoginActivity f1979c;

        a(FlagSixLoginActivity flagSixLoginActivity) {
            this.f1979c = flagSixLoginActivity;
        }

        public final void onClick(View view) {
            if (this.f1979c.F() == 0) {
                if (view != null) {
                    Snackbar X = Snackbar.X(view, "Keys.", 0);
                    X.Y("Action", (View.OnClickListener) null);
                    X.N();
                    FlagSixLoginActivity flagSixLoginActivity = this.f1979c;
                    flagSixLoginActivity.G(flagSixLoginActivity.F() + 1);
                    return;
                }
                d.f();
                throw null;
            } else if (this.f1979c.F() != 1) {
            } else {
                if (view != null) {
                    Snackbar X2 = Snackbar.X(view, "Classes.", 0);
                    X2.Y("Action", (View.OnClickListener) null);
                    X2.N();
                    this.f1979c.G(0);
                    return;
                }
                d.f();
                throw null;
            }
        }
    }

    public final int F() {
        return this.t;
    }

    public final void G(int i) {
        this.t = i;
    }

    /* access modifiers changed from: protected */
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView((int) R.layout.activity_flag_six_login);
        i.g.a(this);
        C((Toolbar) findViewById(R.id.toolbar));
        ((FloatingActionButton) findViewById(R.id.fab)).setOnClickListener(new a(this));
    }

    public final void submitFlag(View view) {
        EditText editText = (EditText) findViewById(R.id.editText3);
        d.b(editText, "editText3");
        if (d.a(editText.getText().toString(), j.a("k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ=="))) {
            Intent intent = new Intent(this, FlagOneSuccess.class);
            FlagsOverview.D = true;
            new i().b(this, "flagSixButtonColor", true);
            startActivity(intent);
        }
    }
}
```

Now we can clealy see **submitFlag** method which will be called on click of **SUBMIT** button. It takes value from the text box, converts it to string and compares with output of `j.a("k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ==")` method. 

This application has some level of obfuscation used, we need to identify what is `j.a()` method. We have `b3nac/injuredandroid/j.java` class in our package.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-16%2018-28-19.png "j. java")

`a()` method has following code, it is a decryption method. We can clearly figure out from the code that [DES](https://en.wikipedia.org/wiki/Data_Encryption_Standard) symmetric-key algorithm is used. **Congrats first milestone achieved!**

Below code uses value from `f2002a` to generate the secret key.

```java
    public static String a(String str) {
        if (b(str)) {
            try {
                SecretKey generateSecret = SecretKeyFactory.getInstance("DES").generateSecret(new DESKeySpec(f2002a));
                byte[] decode = Base64.decode(str, 0);
                Cipher instance = Cipher.getInstance("DES");
                instance.init(2, generateSecret);
                String str2 = new String(instance.doFinal(decode));
                Log.d("Oh snap!", "Decrypted: " + str + " -> " + str2);
                return str2;
            } catch (InvalidKeyException | NoSuchAlgorithmException | InvalidKeySpecException | BadPaddingException | IllegalBlockSizeException | NoSuchPaddingException e2) {
                e2.printStackTrace();
            }
        } else {
            System.out.println("Not a string!");
            return str;
        }
    }
```

Our next goal is to find the key, it must be coded somewhere. **Expedition continues..**

Class `j` has byte array defined which takes value from `h.a()` method.

```private static final byte[] f2002a = h.a();```

### h.java have following code

```java
package b3nac.injuredandroid;

import android.util.Base64;

public class h {

    /* renamed from: a  reason: collision with root package name */
    private static byte[] f1997a = Base64.decode("Q2FwdHVyM1RoMXM=", 0);

    /* renamed from: b  reason: collision with root package name */
    private static String f1998b = "9EEADi^^:?;FC652?5C@:5]7:C632D6:@]4@>^DB=:E6];D@?";

    static {
        Base64.decode("e0NhcHR1cjNUaDFzVG9vfQ==", 0);
    }

    static byte[] a() {
        return f1997a;
    }

    static String b() {
        return f1998b;
    }
}
```

`a()` method returns byte array of the key, before that it is decoding a base64 string `Q2FwdHVyM1RoMXM=`

Let's decode it to find out the actual key.

`echo Q2FwdHVyM1RoMXM= | base64 -d`

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-16%2018-57-04.png "Secret key")

### Second milestone achieved

We got the key: `Captur3Th1s`

Now this key can be useful if we want decrypt the cipher text using some custom java class or online tool. But we are going to go for much simpler opttion.

### Let's take a pause here

While testing and application there are some keywords which will quickly help us identify classes, alogorithm used for the encryption. Although it can be little tricky when code is obfuscated.

#### Keywords

encrypt, decrypt, crypt, AES, DES, SecretKeyFactory, secretKey, Cipher, InvalidKeyException
etc.

### Back to our example

We have the decryption method at our disposal. Now as per our understanding the application is taking user input and comparing it with decrypted value of `k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ==`.

What if we can see what is the decrypted value of above mentioned cipher text when application is comparing the value? Yes!! we can definitely do that!

To accomplish this task we are going to use [FRIDA](https://frida.re/), it is a Dynamic instrumentation toolkit. Using frida we can modify application logic on run-time, it allows us to inject our scripts into running processes. It is a must have toolkit in your android testing arsenal.

**To summarize:** We'll hook the decryption method `a()` of class `j` and when application attempts to compare the strings, we'll print the plain text value.

### Frida on the roll

Let's have a quick look at the following js code which will invoke the decryption method `a()` with our supplied value.

#### decrypt.js

```js
console.log("Script loaded successfully ");

Java.perform(function x() {
    console.log("Inside java perform function");
    var my_class = Java.use("b3nac.injuredandroid.j"); // class responsible for decryption
    
    var string_class = Java.use("java.lang.String");

    my_class.a.overload("java.lang.String").implementation = function (x) { //hooking the new function
        console.log("*************************************")
        var my_string = string_class.$new("k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ=="); // cipher text
        console.log("Original arg: " + x);
        var ret = this.a(my_string);
        console.log("Return value: " + ret);
        console.log("*************************************")
        return ret;
    };
});
```

Time to run frida script.

`frida -U -f b3nac.injuredadnroid --no-pause -l decrypt.js`

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-22%2020-19-09.jpg "Frida invoke")

As we can see frida script is successfully ran, this will start the application in our emulator.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-15%2018-03-56.png "Home screen")

Frida script has overridden the decryption method. We'll see the effect of this when we interact with the flag6 activity. So let's go to flag six activity, enter any value and click on **SUBMIT** button.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-22%2020-31-11.png "Hooked")

There is no activity happened on the screen but if we go to our frida console we'll have the following output along with the decrypted text `{This_Isn't_Where_I_Parked_My_Car}`

By overriding the decryption method we have successfully decrypred the cipher text to plain text.

![](https://raw.githubusercontent.com/jaimingohel/1daylabs/master/assets/Screenshot%20from%202020-08-22%2020-34-54.png "Done")

Now imagaine, if this was a real application then you could have used same technique to inject your payloads before it gets encrypted. This way we have bypassed the encryption logic written in the application to make the app tamper free.

I enjoyed writting this, hope you enjoyed reading.

**Happy hacking :-)**

Author: [Jaimin Gohel](https://twitter.com/jaimin_gohel)


## References

* https://github.com/B3nac/InjuredAndroid
* https://github.com/MobSF/Mobile-Security-Framework-MobSF
* https://frida.re/docs/android/
