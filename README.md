Android Upload Service
======================

## Purpose
I needed an easy and efficient way to upload multipart form data (HTTP parameters and files) to a server, and 
I haven't found anything useful so far that suited my needs. I also needed that the upload got handled in the
background and in the most efficient way on Android. More than that, I also needed to show upload status in the
Android Notification Center.

So, after some research on the web I found that the best way to do this is to implement an IntentService and notify
status with broadcast intents. This way the logic is decoupled from the UI and it's much more reusable. By using an
IntentService, you can do multiple uploads being sure that they will be performed sequentially, and so you don't 
have to deal with the nightmare of concurrency. Also, an IntentService is much more efficient than an AsyncTask, it
gets executed in the background and it's completely detached from the UI thread. If you need to show status in your UI,
read further and you'll discover how to do it very easily.

## Installation

Check out the project and add android-upload-service to your project as an [Android Library Project](http://developer.android.com/guide/developing/projects/projects-eclipse.html#ReferencingLibraryProject).

Add the following to your project's AndroidManifest.xml file:

    <service
        android:name="com.alexbbb.uploadservice.UploadService"
        android:enabled="true"
        android:exported="true" >
        <intent-filter>
            <action android:name="com.alexbbb.uploadservice.action.upload"/>
        </intent-filter>
    </service>

## How to start android upload service to upload files
    public void updateSomething(final Context context) {
        final UploadRequest request = new UploadRequest(context, "http://www.yoursite.com/your/script");

        request.addFileToUpload("/absolute/path/to/your/file", 
                                "parameter-name", 
                                "custom-file-name.extension",
                                "content-type")); //You can find many common content types defined as static constants in the ContentType class

        //You can add your own custom headers
        request.addHeader("your-custom-header", "your-custom-value");

        request.addParameter("parameter-name", "parameter-value");
        
        //If you want to add an array of strings, you can simply to the following:
        request.addParameter("array-parameter-name", "value1");
        request.addParameter("array-parameter-name", "value2");
        request.addParameter("array-parameter-name", "valueN");

        request.setNotificationConfig(
                android.R.drawable.ic_menu_upload, //Notification icon. You can use your own app's R.drawable.your_resource
                "notification title", //You can use your string resource with: context.getString(R.string.your_string)
                "upload in progress text",
                "upload completed successfully text",
                "upload error text",
                false); //Set this to true if you want the notification to be automatically cleared when upload is successful
        
        try {
            //Utility method that creates the intent and starts the upload service in the background
            //As soon as the service starts, you'll see upload status in Android Notification Center :)
            UploadService.startUpload(request);
        } catch (Exception exc) {
            //You will end up here only if you pass an incomplete UploadRequest
            Log.e("AndroidUploadService", exc.getLocalizedMessage(), exc);
        }
    }

## How to monitor upload status
Once the service is started, it publishes the upload status with broadcast intents. 
For the sake of simplicity and to not bother you with the writing of a broadcast receiver, 
an abstract broadcast receiver has been implemented for you and you just need to extend it and add your custom code.
So to listen for the status of the upload service in an Activity for example, you just need to do the following:

    public class YourActivity extends Activity {
    
        ...
        
        private final BroadcastReceiver uploadReceiver = new AbstractUploadServiceReceiver() {

            @Override
            public void onProgress(int progress) {
                Log.i("AndroidUploadService", "The progress is: " + progress);
            }

            @Override
            public void onError(Exception exception) {
                Log.e("AndroidUploadService", exception.getLocalizedMessage(), exception);
            }

            @Override
            public void onCompleted(int serverResponseCode, String serverResponseMessage) {
                Log.i("AndroidUploadService", "Upload completed: " + serverResponseCode + ", " + serverResponseMessage);
            }
        };
        
        @Override
        protected void onResume() {
            super.onResume();
            final IntentFilter intentFilter = new IntentFilter();
            intentFilter.addAction(UploadService.BROADCAST_ACTION);
            registerReceiver(uploadReceiver, intentFilter);
        }
        
        @Override
        protected void onPause() {
            super.onPause();
            unregisterReceiver(uploadReceiver);
        }
    
    }
    
## Do you use Android Upload Service in your project?
Let me know, and I'll be glad to include a link in the following list :)

- [VoiSmart IP Communicator](https://play.google.com/store/apps/details?id=com.voismart.softphone)
    
## License

    Copyright (C) 2013 Aleksandar Gotev

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
