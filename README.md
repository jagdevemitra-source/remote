name: Build APK
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Java 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17
      - name: Create Project
        run: |
          rm -rf app build.gradle settings.gradle
          mkdir -p app/src/main/java/com/bldcremote app/src/main/res/layout
          echo "include ':app'" > settings.gradle
          echo "buildscript{repositories{google();mavenCentral()}dependencies{classpath 'com.android.tools.build:gradle:8.1.4'}}allprojects{repositories{google();mavenCentral()}}" > build.gradle
          echo "plugins{id 'com.android.application'}android{namespace 'com.bldcremote';compileSdk 34;defaultConfig{applicationId 'com.bldcremote';minSdk 21;targetSdk 34}}dependencies{implementation 'androidx.appcompat:appcompat:1.6.1'}" > app/build.gradle
          echo '<?xml version="1.0" encoding="utf-8"?><manifest xmlns:android="http://schemas.android.com/apk/res/android"><uses-permission android:name="android.permission.TRANSMIT_IR"/><application android:label="BLDC Remote" android:theme="@android:style/Theme.Black.NoTitleBar"><activity android:name=".MainActivity" android:exported="true"><intent-filter><action android:name="android.intent.action.MAIN"/><category android:name="android.intent.category.LAUNCHER"/></intent-filter></activity></application></manifest>' > app/src/main/AndroidManifest.xml
          echo '<?xml version="1.0" encoding="utf-8"?><LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" android:orientation="vertical" android:layout_width="match_parent" android:layout_height="match_parent" android:background="#000" android:padding="20dp"><TextView android:text="BLDC Remote No Ads" android:textColor="#FFF" android:gravity="center" android:layout_width="match_parent" android:layout_height="wrap_content"/><Button android:id="@+id/btnPower" android:text="POWER" android:layout_width="match_parent" android:layout_height="80dp" android:layout_marginTop="24dp"/><Button android:id="@+id/btnS1" android:text="SPEED 1-5" android:layout_width="match_parent" android:layout_height="60dp" android:layout_marginTop="16dp"/></LinearLayout>' > app/src/main/res/layout/activity_main.xml
          cat > app/src/main/java/com/bldcremote/MainActivity.java <<'JAVA'
          package com.bldcremote;
          import android.app.Activity;
          import android.hardware.ConsumerIrManager;
          import android.os.Bundle;
          import android.widget.Button;
          public class MainActivity extends Activity {
            public void onCreate(Bundle b){
              super.onCreate(b);
              setContentView(R.layout.activity_main);
              ConsumerIrManager ir = (ConsumerIrManager)getSystemService(CONSUMER_IR_SERVICE);
              int[] pat = {9000,4500,560,560,560,1690,560,560,560,560,560,1690,560,560,560,1690,560,560,560,560,560,560,560,1690,560,1690,560,1690,560,1690,560,1690,560,1690,560,1690,560,560,560,560,560,1690,560,560,560,1690,560,1690,560,1690,560,1690,560,560,560,1690,560,1690,560,560,560,560,560,1690,560,560,560,560};
              Button p = findViewById(R.id.btnPower);
              Button s = findViewById(R.id.btnS1);
              p.setOnClickListener(v->{try{ir.transmit(38000,pat);}catch(Exception e){}});
              s.setOnClickListener(v->{try{ir.transmit(38000,pat);}catch(Exception e){}});
            }
          }
          JAVA
      - name: Build APK
        run: |
          gradle wrapper --gradle-version 8.5
          chmod +x gradlew
         ./gradlew assembleDebug
      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: BLDC-Remote-APK
          path: app/build/outputs/apk/debug/app-debug.apk# remote