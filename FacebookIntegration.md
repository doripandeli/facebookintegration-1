In this tutorial I would like to show how to connect your Android application to Facebook API. There are a lot of mobile applications that use Facebook login, signup or even posting data. We will see how we can connect to Facebook API v4 . x.

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZMUxMd2RQaFZqTUE&export=download)

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZaDFVMTdxZERsOTA&export=download)

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZY1hCZHFFYTZfcHc&export=download)


Creating Android Project
-
Firstly make sure you have the latest version of Android Studio. Currently I am using version 1.4.1

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZcC1iaFFSejJyOWs&export=download)

Open it and create a New Project, name it as you wish.Click next and then choose **Minimum API level 17** then click next again. Choose Blank Activity as the first activity:

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZc2hTLXN1azNUODg&export=download)

 Rename this activity to : “**LoginActivity**”. 
 
![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZZFJZTVVRZy1rUTQ&export=download)

Click **Finish**.

Next we have to add another blank activity in the project. We can do this by right-clicking on package and go to **New** -> **Activity** -> **Blank Activity** as in picture below. Leave its name as it is and click finish.

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZakZFeXFaX25nRTQ&export=download)

Now we can run our app for the first time. Make sure you have connected you Android device or Emulator.

Creating Facebook App ID
-
To use Facebook API we have to add our app to Facebook Apps. You can click [here][1] to add your application.

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZa2ZZS0d6WlY0ZUU&export=download)

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZcWs3a0NwY3ZQRFk&export=download)

Choose a random category and click **Create App ID**.

On the next page, scroll down to the bottom and complete both fields with the project packages names.

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZWFJ0VF9RcVh3c2c&export=download)

Click **Next**.

Now we need to add a Development Key Hash. There are two ways of getting it. One of them is running a command in **Windows** or **Mac**, respectively:

*Windows:*
```keytool -exportcert -alias androiddebugkey -keystore %HOMEPATH%\.android\debug.keystore | openssl sha1 -binary | openssl base64```

*Mac:*
```keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64```

 The simplest way to get it is by using the lines of code below: 

```
public static String printKeyHash(Activity context) {
    PackageInfo packageInfo;
    String key = null;
    try {
        String packageName = context.getApplicationContext().getPackageName();
        packageInfo = context.getPackageManager().getPackageInfo(packageName,
                PackageManager.GET_SIGNATURES);
        Log.e("Package Name=", context.getApplicationContext().getPackageName());
        for (Signature signature : packageInfo.signatures) {
            MessageDigest md = MessageDigest.getInstance("SHA");
            md.update(signature.toByteArray());
            key = new String(Base64.encode(md.digest(), 0)); 
            Log.e("Key Hash=", key);
        }
    } catch (PackageManager.NameNotFoundException e1) {
        Log.e("Name not found", e1.toString());
    }
    catch (NoSuchAlgorithmException e) {
        Log.e("No such an algorithm", e.toString());
    } catch (Exception e) {
        Log.e("Exception", e.toString());    }   return key;
}
```
This code prints the Key Hash on **Error Log**. To get it printed, just write this line of code inside *onCreate* method of **LoginActivity** class:
```printKeyHash(this);```

After you run your application, you will get your key hash printed on Error log like this:

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZY3JvV2w5dUViRTg&export=download)

Copy this key and paste into *Development Key Hash* box.
Click **Next**.

Now go to **My Apps** by clicking [here][2]. Copy the App ID

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZVWNUaHVBRTdIUmM&export=download)

Open *strings.xml* at your project and add this line of code:
```<string name="app_id">{Your App ID here}</string>```

Setting up Facebook SDK
-
Open **Project Build Gradle** and add ```mavenCentral()``` line on both repositories.
Then open **Module Build Gradle** and add the *SDK library* by writing this line inside *dependencies*:
 ```compile 'com.facebook.android:facebook-android-sdk:4.6.0'```

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZWElQYkpQU0d3ejg&export=download)

 Now **sync** gradle.

Activities and Layouts
-
Open *AndroidManifest.xml* and add these lines of code inside `<application></application>` tags:
```
<activity
      android:name=".MainActivity"
      android:label="@string/app_name"            android:theme="@style/AppTheme.NoActionBar" >
</activity>
        
<meta-data
    android:name="com.facebook.sdk.ApplicationId"
    android:value="@string/app_id" />

<activity
    android:name="com.facebook.FacebookActivity"
    android:label="@string/app_name"
    android:screenOrientation="portrait"/>

<provider 		     android:authorities="com.facebook.app.FacebookContentProvider"         	    android:name="com.facebook.FacebookContentProvider"
android:exported="true"/>
```

We are done with *AndroidManifest.xml* file, now we are going to work with Java classes and layouts.

Firstly we are going to work with **LoginActivity.class**. This class will perform the Facebook API connection and get data from it. So we have to login with our credentials and then we get our data from API.

Everything we want *LoginActivity.class* to do, is just connecting to *Facebook API* and getting our data. For that we need some simple lines of Java code.

Add this lines before onCreate method inside the class:
```
private CallbackManager callbackManager;
private AccessTokenTracker accessTokenTracker;
private ProfileTracker profileTracker;

//Facebook login button
private FacebookCallback<LoginResult> callback = new FacebookCallback<LoginResult>() {
    @Override
    public void onSuccess(LoginResult loginResult) {
        Profile profile = Profile.getCurrentProfile();
        nextActivity(profile);
    }
    @Override
    public void onCancel() {        }
    @Override
    public void onError(FacebookException e) {      }
};
```

We just created a *FacebookCallback* called **callback**. This executes the next action after we get a response from Facebook API, and the method for that is ```onSuccess()```. 

Inside onSuccess method we are going to create a new Facebook Profile and get data for that profile.
A little bit later we are going to create a simple function called ```nextActivity()``` that is going to switch our activity.

We do not need the code of *Floating Action Button* so we are going to delete it and write some other lines. First we are going to initialize *Facebook SDK*. That will allow us to use its functions and methods. 
So, inside ```onCreate()``` function write down these lines of code:
```
FacebookSdk.sdkInitialize(getApplicationContext());
callbackManager = CallbackManager.Factory.create();
accessTokenTracker = new AccessTokenTracker() {
    @Override
    protected void onCurrentAccessTokenChanged(AccessToken oldToken, AccessToken newToken) {
    }
};

profileTracker = new ProfileTracker() {
    @Override
    protected void onCurrentProfileChanged(Profile oldProfile, Profile newProfile) {
        nextActivity(newProfile);
    }
};
accessTokenTracker.startTracking();
profileTracker.startTracking(); 
```

Those are the first things we need to we need to create and initialize. To make it work, we need to show that famous *Facebook Log in* button. We won’t make it from scratch because it exists inside *SDK’s libraries*, we just need to call it at our layout. So we will edit our LoginActivity’s layout. It’s name should be ```content_login.xml```. In fact, the latest version of Android Studio creates two default *.xml* files for every activity we create. The other layout file is called ```activity_login.xml```. Inside the ```activity_login.xml``` there is a floating button. We do not need it so you can delete the code. If you see inside ```content_login.xml``` there is only a ```TextView``` element. We will remove it and create a new ```LinearLayout```, *horizontally* oriented. Inside that layout we will add that log in button. So, you can simply paste the code below instead of the ```TextView``` element:

```
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <com.facebook.login.widget.LoginButton
        android:id="@+id/login_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:gravity="center"
        android:layout_margin="4dp"
        android:paddingTop="12dp"
        android:paddingBottom="12dp"/>

</LinearLayout>

```
I added some padding at top and bottom and centered inside that horizontal linear layout. We will use its id to call it inside our *LoginActivity*. Now let us go back to our Login class and create that button.

Inside ```onCreate()``` method, before the closing bracket add the lines of code below:
```
LoginButton loginButton = (LoginButton)findViewById(R.id.login_button);
callback = new FacebookCallback<LoginResult>() {
    @Override
    public void onSuccess(LoginResult loginResult) {
        AccessToken accessToken = loginResult.getAccessToken();
        Profile profile = Profile.getCurrentProfile();
        nextActivity(profile);
        Toast.makeText(getApplicationContext(), "Logging in...", Toast.LENGTH_SHORT).show();    }

    @Override
    public void onCancel() {
    }

    @Override
    public void onError(FacebookException e) {
    }
};
loginButton.setReadPermissions("user_friends");
loginButton.registerCallback(callbackManager, callback);

```

We just created a connection between the button in ```content_login.xml``` and *Facebook SDK* libraries. Now we are done with ```onCreate()``` function.

There are some ```@Overrided``` methods that we need to use inside *LoginActivity.class*. You can copy the lines below: 
```
@Override
protected void onResume() {
    super.onResume();
    //Facebook login
    Profile profile = Profile.getCurrentProfile();
    nextActivity(profile);
}

@Override
protected void onPause() {

    super.onPause();
}

protected void onStop() {
    super.onStop();
    //Facebook login
    accessTokenTracker.stopTracking();
    profileTracker.stopTracking();
}

@Override
protected void onActivityResult(int requestCode, int responseCode, Intent intent) {
    super.onActivityResult(requestCode, responseCode, intent);
    //Facebook login
    callbackManager.onActivityResult(requestCode, responseCode, intent);

}
```

The last function in this class is ```nextActivity()```. The only thing this function will do is just switch the activities and pass some data to the next activity.
```
private void nextActivity(Profile profile){
    if(profile != null){
        Intent main = new Intent(LoginActivity.this, MainActivity.class);
        main.putExtra("name", profile.getFirstName());
        main.putExtra("surname", profile.getLastName());
        main.putExtra("imageUrl", profile.getProfilePictureUri(200,200).toString());
        startActivity(main);
    }
}
```
We need **first** and **last name** of the profile, also a *200x200* profile **picture**. At this stage we can only get its *Uri*. Those three strings will be put as *extras* on our next activity, which we called ```MainActivity```.

MainActivity class
-
In this class, inside ```onCreate()``` method, we have a *float action button* created. We will need that in the future. Now make sure you have an *.xml* file inside `menu` folder in your project like in picture below:

![](https://docs.google.com/uc?authuser=0&id=0BzzlPph96zYZMW5zdGxkalJMZ3c&export=download)

 We will use that to create a *log-out* button. The code below will create this button and place it on the right side of the app’s *toolbar*:
```
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate the menu; this adds items to the action bar if it is present.
    getMenuInflater().inflate(R.menu.menu_login, menu);
    return true;
}

@Override
public boolean onOptionsItemSelected(MenuItem item) {
    // Handle action bar item clicks here. The action bar will
    // automatically handle clicks on the Home/Up button, so long
    // as you specify a parent activity in AndroidManifest.xml.
    int id = item.getItemId();

    //noinspection SimplifiableIfStatement
    if (id == R.id.action_settings) {
        logout();
        return true;
    }

    return super.onOptionsItemSelected(item);
}
```
We will create the ```logout()``` function later on.
You should notice that the name of *.xml* file will be inflated in the menu. The code below presents that menu item:

```
<menu 
	xmlns:android="http://schemas.android.com/apk/res/android
	xmlns:app="http://schemas.android.com/apk/res-auto"    xmlns:tools="http://schemas.android.com/tools"   tools:context="com.example.theodhor.facebookintegration.MainActivity">
      
    <item android:id="@+id/action_settings"
          android:title="@string/action_settings"
          android:orderInCategory="100"
          app:showAsAction="never"/>
</menu>
```
Make sure you set the right *context* (should be set to *.MainActivity*). Then you can change action title through ```strings.xml``` or even directly. We need to call it “LogOut”. So we can change it from strings.xml:
```
<string name="action_settings">LogOut</string>
```
That was about our toolbar and its button. 
Our ```nextActivity()``` function used in *LoginActivity* class, put some data as strings to pass on our next activity. Now that we are in that new activity we need to get them. To do that we should create three other strings inside ```onCreate()``` method and store this data on them:
```
Bundle inBundle = getIntent().getExtras();
String name = inBundle.get("name").toString();
String surname = inBundle.get("surname").toString();
String imageUrl = inBundle.get("imageUrl").toString();
```
To display those data we need to set up the ```content_main.xml``` layout. The code below adds the elements we need to display the data. Add these lines of code inside ```RelativeLayout``` tags:
```
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <TextView
        android:text="Hello:"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:textSize="20dp"
        android:layout_gravity="center_horizontal"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/nameAndSurname"
        android:textSize="22dp"
        android:textStyle="bold"
        android:layout_marginTop="10dp"
        android:layout_gravity="center_horizontal"/>
    <ImageView
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:id="@+id/profileImage"
        android:layout_marginTop="10dp"
        android:layout_gravity="center_horizontal"/>
</LinearLayout>
```
To display the profile name use the code below, and place it inside ```onCreate()``` method:
```
TextView nameView = (TextView)findViewById(R.id.nameAndSurname);
nameView.setText("" + name + " " + surname);
```
Next thing we want to display is the profile **picture**. From the last activity we got the picture *Uri* as a string. So we should use that Uri to download the picture as a *Bitmap*. For that, just create a new class, and put the code below inside it: 
```
public class DownloadImage extends AsyncTask<String, Void, Bitmap> {
    ImageView bmImage;

    public DownloadImage(ImageView bmImage) {
        this.bmImage = bmImage;
    }

    protected Bitmap doInBackground(String... urls) {
        String urldisplay = urls[0];
        Bitmap mIcon11 = null;
        try {
            InputStream in = new java.net.URL(urldisplay).openStream();
            mIcon11 = BitmapFactory.decodeStream(in);
        } catch (Exception e) {
            Log.e("Error", e.getMessage());
            e.printStackTrace();
        }
        return mIcon11;
    }

    protected void onPostExecute(Bitmap result) {
        bmImage.setImageBitmap(result);
    }
}
```
To display profile picture in our app, just write the line below inside ```onCreate()``` method in *MainActivity* class, after the last line we wrote.
```
new DownloadImage((ImageView)findViewById(R.id.profileImage)).execute(imageUrl);
```
It uses the *imageUrl* string we have, downloads the image and displays it inside the ```content_main.xml``` layout.

Now that displaying data is done, we will make our app to post to Facebook. The floating action button will be used to display the **share** dialog. Firstly open ```activity_main.xml``` and change:
```
android:src="@android:drawable/ic_dialog_email"
```
to:
```
android:src="@android:drawable/ic_menu_edit"
```
You can also change the button *color* by editing the color values in ```colors.xml```. I have used this color for my fab:
```
<color name="colorAccent">#5694f7</color>
```
Now that we edited our button, we should make it work. There are some simple lines of code to do that. 
Declare a ```private ShareDialog``` in *MainActivity* class:
```
private ShareDialog shareDialog;
```
Inside ```onCreate()``` method create this dialog:
```
shareDialog = new ShareDialog(this);
```
We want to show this dialog when we press the *Fab* button. Replace the ```Snackbar code``` with the one below inside the ```OnClick()``` method:
```
ShareLinkContent content = new ShareLinkContent.Builder().build();
shareDialog.show(content);
```
Our app can post to Facebook up to now. But we are not finished yet. *Logout* function is missing. To make the app log out from facebook, we first should make it understand if it is logged in. To do that we should initialize the Facebook SDK as we did on *LoginActivity*. Write this line of code inside ```onCreate()``` method:
```
FacebookSdk.sdkInitialize(getApplicationContext());
```
The ```logout()``` function should look like this:
```
public void logout(){
    LoginManager.getInstance().logOut();
    Intent login = new Intent(MainActivity.this, LoginActivity.class);
    startActivity(login);
    finish();
}
```
To log out, just call this method inside `onOptionsItemSelected()` method:
```
if (id == R.id.action_settings) {
    logout();
    return true;
}
```

Now you can run your app and post to Facebook!

  [1]: https://developers.facebook.com/quickstarts/?platform=android#_=_
  [2]: https://developers.facebook.com/apps/ 
  
