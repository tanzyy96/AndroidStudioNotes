# JSONHelper

## 1.1 JSON files
There are different types of JSON content we are working with:  
JSONObject:   
JSONArray: [{..}, {..}]  
JSONElement: {"..." : "..." }  

## 1.2 Setting up JSONHelper class
In JSONHelper class, we want to use Gson() methods. Gson methods only can take in objects not primitive data types, hence we need to create a DataItems static class.
```java
public class JSONHelper {

    static class DataItems {
        List<DataItem> dataItems;

        public List<DataItem> getDataItems() {
            return dataItems;
        }

        public void setDataItems(List<DataItem> dataItems) {
            this.dataItems = dataItems;
        }
    }
```

## 1.3 Export to JSON
Creates a JSON file from List<DataItem> object.
```java 
    public static final String FILE_NAME = "menuitems.json";
    public static final String TAG = "JSONHelper";
    
    /*   Export List<DataItem> to JSON  */
    public static boolean exportToJSON(Context context, List<DataItem> dataItemList) {

        DataItems dataItems = new DataItems();
        dataItems.setDataItems(dataItemList);

        Gson gson = new Gson();
        String jsonString = gson.toJson(dataItems);
        Log.i(TAG,"exportToJSON: " + jsonString);

        FileOutputStream fileOutputStream = null;
        File file = new File(Environment.getExternalStorageDirectory(), FILE_NAME);

        try {
            fileOutputStream = new FileOutputStream(file);
            fileOutputStream.write(jsonString.getBytes());
            Log.i(TAG, "Export successful.");
            return true;
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileOutputStream != null) {
                try {
                    fileOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        return false;
    }
 ```
 ## 1.4 Import from JSON
 Extracts DataItem objects from a JSON file
 ```java
     /*  Import from external JSON file  */
    public static List<DataItem> importFromJSON(Context context) {

        FileReader reader = null;
        try {
            File file = new File(Environment.getExternalStorageDirectory(), FILE_NAME);
            reader = new FileReader(file);
            Gson gson = new Gson();
            DataItems dataItems = gson.fromJson(reader, DataItems.class);
            return dataItems.getDataItems();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (reader != null){
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }


        return null;
    }
```

## 2.1 Attempt to import from JSON file from Data API
We want to be able to extract data from our HDB API for CZ2006. In the end I found a way to extracct the resource_id from the data api. The main challenge was to parse through the JSON file to find the exact line I needed. Definitely not as straightforward as using Python. And because we don't have a class with the exact same layout as the HDB JSON file (eg. DataItem), we cannot just use gson.getJson() method.
```java
    /*  Import from external resource file  */
    /*  This method is to extract a certain line from the JSON file */
    public static void importFromResource(Context context) {

        InputStreamReader reader = null;
        InputStream inputStream = null;
        try {
            inputStream = context.getResources().openRawResource(R.raw.hdb);
            reader = new InputStreamReader(inputStream);

            JsonParser parser = new JsonParser();
            JsonObject jsonArray = (JsonObject) parser.parse(reader);
            // extract resource id
            JsonObject result = (JsonObject) jsonArray.get("result");
            String resource_id = String.valueOf(result.get("resource_id"));
            Log.i(TAG, resource_id);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```
Since the HDB API is extracted from the Internet, we still need to figure out how to connect to the Internet via the Android App. Shouldn't be too difficult. Probably will have to create a unique class to suite the HDB API.

The following contains the original method used to import from Resources. Ignoring the try and catch clauses.
```java
  File file = context.getResources().openRawResource(R.raw.hdb);
  reader = new InputStreamReader(inputStream);
  
  Gson gson = new Gson();
  DataItems dataItems = gson.fromJson(reader, DataItems.class);
  return dataItems.getDataItems();
```

API on the gson.fromJson():  
![alt text](https://github.com/tanzyy96/AndroidStudioNotes/blob/master/Gson.fromjson().png "gson.fromJson()")
