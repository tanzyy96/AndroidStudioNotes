# Data Storage

## 1.1 Internal File Storage
We can store data on the internal memory of the Android app. This data cannot be seen via Android File Manager. However, it is still unsafe to be used to store sensitive data such as passwords and credit card information.

### 1.1.1 Writing to Internal File Storage
Before writing any code, we need to have access to the string we want to write and the file name. In this case, the string will be stored under R.string.mymessage.
```java
public static final String FILE_NAME = "myfile.txt";
```
```java
/*-----inside create/write function-----*/
String mymessage = getString(R.string.mymessage);
FileOutputStream fileOutputStream = null;
File file = new File(FILE_NAME);
try {
  fileOutputStream = openFileOutput(FILE_NAME, MODE_PRIVATE);
  fileOutputStream.write(mymessage.getBytes());
  // Toast.makeText(this,  ...)
  } catch (IOException e) {
    e.printStackTrace();
    // other error catches
  } finally {
    try {
      fileOutputStream.close();
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```
There are also shorter methods using Apache Commonds or Guava from Google.

### 1.1.2 Deleting file from Internal File Storage
```java
/*-----inside delete function-----*/
File file = new File(getFilesDir(), FILE_NAME);
if (file.exists()) {
	deleteFile(FILE_NAME);
} else {
	// insert error message
}
```

## 1.2 External File Storage
External files can be accessed by the user. However, because we are accessing the main memory of the Android device, we need to check if the external memory is *readable* and *writable*. After that, we need to request for user permissions to access the external memory.

### 1.2.1 Setting up Relevant Methods
Add the following code the the AndroidManifest
```java
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
```java
   /* Checks if external storage is available for read and write */
    public boolean isExternalStorageWritable() {
        String state = Environment.getExternalStorageState();
        return Environment.MEDIA_MOUNTED.equals(state);
    }

    /* Checks if external storage is available to at least read */
    public boolean isExternalStorageReadable() {
        String state = Environment.getExternalStorageState();
        return (Environment.MEDIA_MOUNTED.equals(state) ||
                Environment.MEDIA_MOUNTED_READ_ONLY.equals(state));
    }

    // Initiate request for permissions.
    private boolean checkPermissions() {

        if (!isExternalStorageReadable() || !isExternalStorageWritable()) {
            Toast.makeText(this, "This app only works on devices with usable external storage",
                    Toast.LENGTH_SHORT).show();
            return false;
        }

        int permissionCheck = ContextCompat.checkSelfPermission(this,
                Manifest.permission.WRITE_EXTERNAL_STORAGE);
        if (permissionCheck != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this,
                    new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},
                    REQUEST_PERMISSION_WRITE);
            return false;
        } else {
            return true;
        }
    }

    // Handle permissions result
    @Override
    public void onRequestPermissionsResult(int requestCode,
                                           @NonNull String permissions[],
                                           @NonNull int[] grantResults) {
        switch (requestCode) {
            case REQUEST_PERMISSION_WRITE:
                if (grantResults.length > 0
                        && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    permissionGranted = true;
                    Toast.makeText(this, "External storage permission granted",
                            Toast.LENGTH_SHORT).show();
                } else {
                    Toast.makeText(this, "You must grant permission!", Toast.LENGTH_SHORT).show();
                }
                break;
        }
    }
```
We also want to introduce a getFile() method to retrieve the file location.
```java
private File getFile() {
        return new File(Environment.getExternalStorageDirectory(), FILE_NAME);
        // return new File(FILE_NAME);
        // return new File(getFileDir(), FILE_NAME);
    }
```

## 1.2.2 Writing to External File Memory
Next, we will make adjustments to the existing file input method.
```java
				if (!permissionGranted){
            checkPermissions();
            return;
        }
				
        String mymessage = getString(R.string.mymessage);
        FileOutputStream fileOutputStream = null;
        File file = getFile();

        try {
            //fileOutputStream = openFileOutput(FILE_NAME, MODE_PRIVATE);
            fileOutputStream = new FileOutputStream(file);
            fileOutputStream.write(string.getBytes());
            Toast.makeText(this, "Created " + FILE_NAME, Toast.LENGTH_SHORT).show();
        } catch (IOException e) {
            e.printStackTrace();
            Toast.makeText(this, "Exception "+ e.getMessage(), Toast.LENGTH_SHORT).show();
        } finally {
            try {
                fileOutputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
				}
```

### 1.2.3 Deleting from External File Memory
Slight adjustments from the internal memory method.
```java
if (!permissionsGranted) {
	checkPermissions();
	return;
}
File file = getFile();
if (file.exists()) {
	file.delete();
} else {
	...
}
```

### 1.2.4 Reading from External File Memory
To read from external storage, this is a method involving BufferedMemory I picked up from a separate source.
```java
        if (!permissionGranted){
            checkPermissions();
            return;
        }
        TextView tV = (TextView) findViewById(R.id.textView);
        File file = getFile();
        if (file.exists()) {
            BufferedReader myReader = null;
            try {
                String s;
                String content = "";
                FileInputStream fileInputStream = new FileInputStream(file);
                myReader = new BufferedReader(new InputStreamReader(fileInputStream));
                while ((s = myReader.readLine())!= null) {
                    content += s + "\n";
                }
                tV.setText(content);
								// or whichever way you want to indicate 
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                myReader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        } else {
					Toast.makeText(this, "No such file found.", Toast.LENGTH_SHORT).show();
```
