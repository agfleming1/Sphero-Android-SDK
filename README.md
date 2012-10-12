![android.jpg](https://github.com/orbotix/Sphero-Android-SDK/raw/master/assets/image001.jpg)

# Android Developer Quick Start Guide

## Overview
 
This Guide walks you through the basics of creating mobile apps for Android that leverage the Orbotix Sphero SDK. The examples in this guide were built using Java and although we strive to support **ALL** the various Android devices available there are a few that are known to cause problems. Visit our developer forum for more information. The goal of this developer guide along with sample code is to give the developer a taste of the wide variety of things Sphero can do, respond to, and keep track of.

*In general this guide will walk you through:*

 - Installing the SDK
 - Changing Sphero's Color
 - Using the Roll Command to move Sphero 

### Before we begin - Installing Eclipse

    Notice: The Sphero Android SDK works with Android 2.2+

Before you begin to develop applications that interface with Sphero on Android, you will need to install the Android developer tools. We often use Eclipse but there are many other well written Android IDEs out there and these same basic steps are most often still applicable. 

 - Install the [Android SDK](http://developer.android.com/sdk/index.html) and the [Eclipse Plugin](http://developer.android.com/sdk/eclipse-adt.html)

## Installing the Sphero Android SDK

 - Download the latest version of the [Sphero Android SDK](https://github.com/orbotix/Sphero-Android-SDK/zipball/master)
     * *You can always keep up to date by watching our [GitHub Repository](https://github.com/orbotix/Sphero-Android-SDK)*

### Create a new Android project in Eclipse.  

When creating a new project it is important to take special notice to the Android API Level and the Java compliance level. The Sphero SDK currently supports: 
    
 - Andriod API level 8 (Android 2.2) or greater. 
 - Java language compliance level 6.0(1.6) .
    
### Integrating the Sphero Libraries 

 You can start a new Sphero project using the libraries in the library folder or start a project using one of the sample projects from the samples folder. This quick start guide describes how to start a new project. 
 
 To start, create a new Android project in your Eclipse workspace. Then, place the libs folder from the SDK's library folder into your Android project's folder.  
   
   ![QSG-libs.png](https://github.com/orbotix/Sphero-Android-SDK/raw/master/assets/image002.png)

### Setting the dependency to RobotLibrary.jar.  
   
Eclipse should automatically add RobotLibrary.jar to the Android Dependencies folder.  But, if it doesn not, set the dependency in the project's properties in the Properties->Java Build Path-> Libraries dialog.  This will allow your project access to all the public method names in RobotLibrary.jar. 
 
 ![QSG-jar-depend.png](https://github.com/orbotix/Sphero-Android-SDK/raw/master/assets/image003.png) 

## Using the Sphero Android SDK
     
### Add code to connect to a sphero.  
    
 The RobotLibrary includes a view called `SpheroConnectionView` which will 
 handle connecting to a Sphero. When the view fires a `onRobotConnected` event and your are ready to 
 send commands.  
    
  - To use the `SpheroConnectionView`, add the following code to your Acitivity's xml layout file
 
        <FrameLayout
            xmlns:android="http://schemas.android.com/apk/res/android"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:background="#ff888888" >

            <orbotix.view.connection.SpheroConnectionView
                android:id="@+id/sphero_connection_view"
                android:layout_width="fill_parent"
                android:layout_height="fill_parent"
                android:background="#FFF" />
                
        </FrameLayout>
       
  - This will show the `SpheroConnectionView` when the Activity starts.  It is important to put the view last in a frame layout, so when you hide the rest of your layout will be visible.  Also, you must register for the `OnRobotConnectionEventListener`.  This will fire events to let you know the user's interaction with the `SpheroConnectionView` and you can do what you please.  The code is as follows
    
   
	    final SpheroConnectionView mSpheroConnectionView = (SpheroConnectionView)findViewById(R.id.sphero_connection_view);
	    
        // Set the connection event listener 
        mSpheroConnectionView.setOnRobotConnectionEventListener(new OnRobotConnectionEventListener() {
        	// If the user clicked a Sphero and it failed to connect, this event will be fired
			@Override
			public void onRobotConnectionFailed(Robot robot) {}
			// If there are no Spheros paired to this device, this event will be fired
			@Override
			public void onNonePaired() {}
			// The user clicked a Sphero and it successfully paired.
			@Override
			public void onRobotConnected(Robot robot) {
				mRobot = robot;
				// Skip this next step if you want the user to be able to connect multiple Spheros
				mSpheroConnectionView.setVisibility(View.GONE);
			}
		});
	
  - These events are useful feedback from the user.  For example, you could use the `onNonePaired()` event to show a popup message that no Sphero's are conencted, and direct the user to a www.gosphero.com to buy one. 
	
  - Finally, you must disconnect the Sphero, and you must make sure to shutdown the connection view in your Acitivity's `onStop()` method.  Disconnecting Sphero, puts it back into the default stable state.  And, shutting down the connection view is needed, because if the user has bluetooth disabled, and chooses not to enable it with the pop-up presented, the view needs to be told to stop searching for the user enabling it.
  
	  	@Override
		protected void onStop() {
			super.onStop();
			
			// Shutdown Sphero connection view
			mSpheroConnectionView.shutdown();
			
			// Disconnect from the robot.
			RobotProvider.getDefaultProvider().removeAllControls();
		}
	

### Add code to blink the RGB LED.

Now it is time to add code that sends a command to Sphero. In this case we will blink the 
RGB LED blue. Commands are encapsulated by subclasses of DeviceCommand and message to Sphero using 
DeviceMesssenger singleton. For convenience, DeviceCommand subclasses have class `sendCommand()`
methods that will post messages to the DeviceMessenger singleton. Here is the code for the 
`blink()` method sends the SetRGBLEDCommand to blink LED.

	   private void blink(final boolean lit){
	       
	       if(mRobot != null){
	           
	           //If not lit, send command to show blue light, or else, send command to show no light
	           if(lit){
	               RGBLEDOutputCommand.sendCommand(mRobot, 0, 0, 0);        // 1
	           }else{
	               RGBLEDOutputCommand.sendCommand(mRobot, 0, 0, 255);      // 2
	           }
	           
	           //Send delayed message on a handler to run blink again
	           final Handler handler = new Handler();                       // 3
	           handler.postDelayed(new Runnable() {
	               public void run() {
	                   blink(!lit);
	               }
	           }, 1000);
	       }
	   }
	
1. This line will send a command to turn off the LED. `mRobot` is the Robot object that will receive the
	command, and last three parameters turn of the red, green, and blue components of the LED. A 0 value 
	for the color component will set the LED components brightness off.  
2. This line will send a command to turn on the blue LED at full brightness. 255 is full brightness, and is
	only set for the blue component of the LED.  
3.  This line creates a Handler that is used to post a delayed call to the `blink()` method after 1 second
	with the lit parameter inverted, so the next call toggles the LED on or off.

### Modify the AndroidManifest.xml file.

Before running the application, you will need to add permissions to use bluetooth, 

		    <uses-permission android:name="android.permission.BLUETOOTH" />
		    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />

### Run on a device.

 - Run the Application on a supported Android Device.  **Turn Bluetooth ON**.

 - At this point in time you will want to Pair your Android Device to Sphero from within the settings.

### Sendings Roll commands

 - Using Roll Commands to **Move** Sphero.

 - Using Roll Commands to **Stop** Sphero.

	So, you got the LED to blink… that's Awesome! But let's also take advantage of the amazing technology inside Sphero and drive the ball around a little bit. 
	In order to move Sphero you will need to send a RollCommand. The RollCommand takes two parameters.

   1.  Heading in degrees from 0° to 360° 
   2.  Speed from 0.0 to 1.0. 

For example, a heading of 90° at a speed of 0.5 will tell Sphero to turn clockwise 90° at half speed (1.0 is full speed). Once this command is issued Sphero will continue at this heading and speed until it hits something or runs out of range, so you will need to stop the ball using the RollCommand and `sendStop()`.

Now, it's time to modify the code. Let's send Sphero forward at full speed for 2 seconds. So, add the following method 
to the main activity.

	    private void drive() {
	    	if(mRobot != null) {
	    		// Send a roll command to Sphero so it goes forward at full speed.
	    		RollCommand.sendCommand(mRobot, 0.0f, 1.0f);                         // 1
	    		
	    		// Send a delayed message on a handler
	    		final Handler handler = new Handler();                               // 2
	    		handler.postDelayed(new Runnable() {
					
					@Override
					public void run() {
						// Send a stop to Sphero
						RollCommand.sendStop(mRobot);                               // 3
					}
				}, 2000);
	    		
	    	}
	    }

1. This line sends the heading of 0° and the maximum speed of 1.0 to Sphero.
2. This line creates the handle that is used to send the delayed stop command.
3. This line uses the RollCommand to send the stop.

	Next add a call to `drive()` in the `onActivityResult()` below the call to `blink()`.

**Run the application on an Android Device, if all went well Sphero should have moved forward just a little.**



## Where is Sphero going?

If you have successfully completed the quick start guide then Sphero should have moved after running the modified code.  What is interesting to note here is that Sphero just went in a *random* direction.  The direction was not random at all, Sphero believe it or not has a *front* and a *back*.  It is necessary for the application to determine what direction forward is for the user from the point of view of the ball.  We call this step `Calibration` and it is **required** to properly drive Sphero in a predictable direction.  To learn more about calibration and using the `BackLED` to set Sphero's orientation please check out the `UISampler` Sample project.

## Questions

For questions, please visit our developer's forum at [http://forum.gosphero.com/](http://forum.gosphero.com/)

