# Creating a penetration testing environment using Android Studio: Android Virtual Devices (AVD)
## [Download and install Android Studio](https://developer.android.com/studio)
* **Description:** This is Android’s official development IDE for Android applications. This development IDE allows users to write code intelligently, emulate different emulator environments, debug components and more. The Android Software Development Kit (SDK) and the Android Virtual Device (AVD) came as options to be installed with the Android Studio package. The Android SDK comes with a set of tools used in development and testing that can be used on the Android mobile application. The Android AVD is used to emulate the device and test its functionality when developing and/or testing.
* Setup Android Studio Tools
	* Go to Preferences -> Appearance & Behavior -> System Settings -> Android SDK. Click on the "SDK Tools" tab and make sure you have at least one version of the "Android SDK Build-Tools" installed.
* Set up a virtual device
	* From the Android Studio main screen, go to Configure -> AVD Manager.
	* Press the "+ Create Virtual Device" button.
	* Choose the type of hardware you'd like to emulate. Select an OS version to load on the emulator.
	* Choose a version that has Google API's enabled, so ADB root may be used.
	* We recommend version 7 or below so the System certificate store may be altered.
* Add Android Studio Tools to your PATH environment variable (for ADB usage).
	* We recommend placing it in .bashrc or .profile
	* `sudo nano ~/.bashrc`
	* add the line: `export PATH=$PATH:/Users/<username>/Library/Android/sdk/tools/`
	* Finally, `source` the file so the path is updated in any existing terminal windows: `source ~/.bashrc`
## Download common Android APK reverse engineering and pentesting tools
* Follow source instructions for your environment. For this project we all use macOS, therefore, the instructions reflect our environment
* Install brew, a package handler for macOS. It is not required, but it does make it easier to install tools via CLI.
	* In Terminal run: `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
* [JADX - Java to DeX Decompiler](https://github.com/skylot/jadx)
	* **Description:** Java to DeX Decompiler is a tool used to decompile the Nebraska application’s Android Application Package (APK). It can show the contents of the APK and the binaries/libaries it used.
	* Simply `brew install jadx` from terminal
* [APKTool - Reverse Engineering tool for Android APK files](https://ibotpeaches.github.io/Apktool/)
	* **Description:** APKTool is a reverse engineering tool. It can be used to decode resources and make modifications to an application to be built again for further testing.
	* Download Mac wrapper script (Right click, Save Link As apktool)
	* Download apktool-2
	* Rename downloaded jar to apktool.jar     e.g., `mv apktool_v2.1.jar apktool.jar`
	* Copy or Move both files (apktool.jar & apktool) to /usr/local/bin
	* Make sure both files are executable (chmod +x)
	* Try running apktool via cli
* [MITM Proxy - Local man-in-the-middle tool for SSL traffic analysis](https://mitmproxy.org/)
	* **Description:** MITM Proxy is a tool that can act as a local man in the middle tool for SSL traffic analysis. It was used to monitor requests that the application is being sent and received. This is helpful to discover if a mobile application’s traffic is secure or not.
	* `brew install mitmproxy`
	* [Prepare to install trusted certificates](http://wiki.cacert.org/FAQ/ImportRootCert#Android_Phones_.26_Tablets) so the certificate of MITMProxy is accepted from the Android OS (We believe it must be Android 7 or lower to do this)
	* Go to [CACert](https://www.cacert.org/index.php?id=3)
	* Download the root certificate and intermediate certificate in PEM format
	* obtain the hash of the root certificate using the 'old' style: `openssl x509 -inform PEM -subject_hash_old -in <root.crt> | head -1`
		* It should return '5ed36f99' (at the time this was written, this may change)
		* create a new file titled as the returned hash, and append '.0': `touch 5ed36f99.0`
	* cat the root certificate contents into the new file `cat <root.crt> > 5ed36f99.0`
	* Finally, run the openssl command to finalize the formatting for android: `openssl x509 -inform PEM -text -in <root.crt> -out /dev/null >> 5ed36f99.0`
	* Repeat these steps for the intermediate certificate (Note: the hash will be different for the intermediate certificate)
	* Turn on mitmproxy: in terminal run `mitmproxy`
	* Set the proxy on the android emulator to point toward mitmproxy
		* When running, Open the Extended Controls window (...) > go to Settings (cog icon) > select the Proxy tab > set Host name to the <proxy IP address> (the IP of the computer running the proxy) and the Port number to 8080
	* Place the certificate files onto the android device
		* Manually
			* First use `adb root` to enable root adb usage
			* Then, from the directory of your .0 certificate files, use, `adb push *.0 /sdcard/`
			* Next, use `adb shell` to access your emulator and go to your sdcard directory
			* Then use `mount -o rw,remount /system` to get read/write permissions
			* Finally, move the files to the certificate store: `mv *.0 /system/etc/security/cacerts`
		* Automatically (preferred)
			* Once mitmproxy is installed, is running, and the proxy is configured on the emulator, open the browser on the device and go to: http://mitm.it
			* Select the proper OS to install the certificates, follow the prompts, allowing the certificate to be installed.
		* Requests will now be captured in mitmproxy

* Download the APK you want to analyze, we suggest [APKPure](https://apkpure.com/)

* [Drozer - Android Security Testing framework](https://github.com/FSecureLABS/drozer)
	* **Description:** Drozer is a security testing framework for Android. It can analyze the mobile application for any faults in interprocess communication, possible security vulnerabilities the application may have, and exploits that may be used against the mobile application.
	* List of prequisite dependencies needed by Drozer to function properly
		* [Python2.7](https://www.python.org/downloads/)
		* [Protobuf 2.6 or greater](https://pypi.org/project/protobuf/#files)
			* pip install protobuf
		* [Pyopenssl 16.2 or greater](https://pypi.org/project/pyOpenSSL/#files)
			* pip install pyOpenSSL
		* [Twisted 10.2 or greater](https://pypi.org/project/Twisted/#files)
			* pip install Twisted
		* [Java Development Kit 1.7 or greater](https://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html)
		* [Android Debug Bridge](https://developer.android.com/studio/releases/platform-tools.html)
	* Go to  [drozer's github releases page](https://github.com/FSecureLABS/drozer/releases)
		* Download the latest drozer-x.x.x-py2-none-any.whl
		* Download the latest drozer-agent-x.x.x.apk
	* Then use `pip2.7 install drozer-x.x.x-py2-none-any.whl` wherever you downloaded the files above and pip will install drozer and dependencies, if possible.
	* Open Android Studio. From the Android Studio main screen, go to Configure -> AVD Manager.
	* Start your emulated device
	* After the device successfully booted, drag and drop the drozer-agent-x.x.x.apk over the device to install the drozer agent onto the phone or open a terminal and type `adb install drozer-agent-x.x.x.apk`
	* Open drozer agent on your emulated device
	* Click on the "OFF" button to the right of Embedded Server so it turns to "ON"
	* Open a terminal on your computer and type `adb forward tcp:31415 tcp:31415
	* In the same or another terminal, type `drozer console connect`

* [Firebase Scanner](https://github.com/shivsahni/FireBaseScanner)
	* **Description:** Firebase Scanner is another exploit discovery tool that was used against the mobile application. It specializes in discovering misconfigurations within the mobile application.
	* Dependencies
		* Python 2.7
	* Installation
		* Navigate to the FirebaseScanner repository (linked above)
		* Click 'Clone or Download'
		* In Terminal, navigate to a directory where you would like to install FirebaseScanner (e.g., /opt/)
		* Type: `git clone <paste the git clone link retrieved above>` and then press enter
	* Useage
		* From the repository directory (Where you installed it), type `python FirebaseMisconfig.py -p /<path>/<to>/<apk>/<your.apk>`

* [Frida](https://frida.re/)
  * **Description:** Frida is a dynamic instrumentation toolkit used to inject code snippets or scripts into native applications to test the security of the applications being examined.

* [Jarsigner](https://docs.oracle.com/javase/7/docs/technotes/tools/windows/jarsigner.html)
  * **Description:** jarsigner is a tool that comes with Oracle’s Java Development Kit (Java SE). It allows the user to view contents and other information as well as verify the integrity of a Java Archive file. In this case, it was used to verify the integrity of the Nebraska APK.
  * To download and install, click on [Oracle Java Development Kit (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
  
* nm, readelf, objdump
    * **Description:** <here>
    * <Where they are located>
* [OSWASP Zed Attack Proxy (ZAP)](https://github.com/zaproxy/zaproxy/wiki/Downloads)
	* **Description:** It is a great automation tool used to find security vulnerabilities and flaws when penetration testing
