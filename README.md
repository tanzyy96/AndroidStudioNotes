# Android Studio Notes

## 1.1 DataItem
DataItem refers to the main type of object that we are going to use in the app. This object contains attributes and methods relevant to the functionalities of the app, as well as get an set methods needed.

### 1.1.1 Create DataItem Class
```java
package com.example.android.data;

public class DataItem implements Parcelable {
  private ...;
  private ...;
  // constructors
  public DataItem(){}
  public DataItem(...){
    if (itemId == null) {
      itemId = UUID.randomUUID().toString();
    }
    ... }
  // get and set methods here
  // toString method here
  // Parcelable methods here
}
```
Parcelable allows for Android Intent to store higher level data structures inside its Extras.

### 1.1.2 Create DataProvider Class
```java
package com.example.android.data.model;

public class DataProvider {
  public static List<DataItem> dataItemList;
  public static Map<String, DataItem> dataItemMap
  static {
    dataItemList = new ArrayList<>();
    dataItemMap = new HashMap<>();
  }
  // eg for add item
  addItem(new DataItem(...))
  public static void addItem(DataItem dataItem) {
    dataItemList.add(dataItem);
    dataItemMap.put(dataItem.getItemId(), dataItem);
  }
}
```
Remember, List is an ordered array and Map is a dictionary.

## 1.2 DataItemAdapter
DataItemAdapter is the controller class that places the data from DataProvider into the Activity screen.

### 1.2.1 RecyclerView VS ListView

**1.2.1.1 LayoutManager**
Now using a RecyclerView, we can have
i. LinearLayoutManager - which supports both vertical and horizontal lists,
ii. StaggeredLayoutManager - which supports Pinterest like staggered lists,
iii. GridLayoutManager - which supports displaying grids as seen in Gallery apps.

* **Item Animator**
 Using the RecyclerView.ItemAnimator class, animating the views becomes so much easy and intuitive.

* **Item Decoration**
 The RecyclerView.ItemDecorator class gives huge control to the developers but makes things a bit more time consuming and complex.

* **OnItemTouchListener**
 Intercepting item clicks on a ListView was simple, thanks to its AdapterView.OnItemClickListener interface. But the RecyclerView gives much more power and control to its developers by the RecyclerView.OnItemTouchListener but it complicates things a bit for the developer.

### 1.2.2 Implementing RecyclerView
```java
package com.example.android.data;

public class DataItemAdapter extends RecyclerView.Adapter<DataItemAdapter.ViewHolder> {
  public static final String ITEM_KEY = ...;
  private List<DataItem> mItems;
  private Context mContext;
  // constructor
  public DataItemAdapter(Context c, List<DataITem> i) {
    this.mContext = c;
    this.mItems = i;
  }
  @Override
  public DataItemAdapter.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    LayoutInflater inflater = LayoutInflater.from(mContext);
    View itemView = inflater.inflate(R.layout.list_item, parent, false);
    ViewHolder vh = new ViewHolder(itemView);
    return vh;
  }
  @Override
  public void onBindViewHolder(DataItemAdapter.Viewholder holder, int position) {
    final DataItem item = mItems.get(position);
    try {
      holder.tvName.setText(item.getItemName());
      // String imageFile = item.getImage();
      // InputStream inputStream = mContext.getAssets().open(imageFile);
      // Drawable d = Drawable.createFromStream(inputStream, null);
      // holder.imageView.setImageDrawable(d);
    } catch (IOException e) { e.printStackTrace(); }
    holder.mView.setOnClickListener(new View.onClickListner() {
      @Override
      public void onClick(View v) {
        // for passing dataItem
        Intent intent = new Intent(mContext, otherActivity.class);
        intent.putExtra(ITEM_KEY, item);
        mContext.startActivity(intent);
      }
    });
    // can insert other actions
  }
  public int getItemCount() { return mItems.size();}
  public static class ViewHolder extends RecyclerView.ViewHolder {
    public TextView tvName;
    // other variables in the current Activity
    public View mView;
    public ViewHolder(View itemView) {
      super(itemView);
      tvName = (TextView) itemView.findViewById(R.id.itemView);
      // casting for other variables
      mView = itemView;
    }
  }
}
```
Other things to do:
i. Add "com.android.support:recyclerview v..." to build.gradle
ii. main_activity.xml -> add xmlns:app under RelativeLayout
ii. main_activity.xml -> add RecyclerView with a correct id -> add app:layoutManager="LinearLayoutManager"

## 1.2 Passing Intent & Extras
Intent is an abstract description of an operation, in this case startActivity.

### 1.2.1 Create New Activity
Create a new empty activity and under AndroidManifest->NewActivity, add android:parentActivityName="MainActivity" or whichever screen you want for backwards navigation.


### 1.2.2 Create and pass Intent via Extras()
```java
holder.mView.setOnClickListener(new View.onClickListner() {
  @Override
  public void onClick(View v) {
    // for passing dataItem
    Intent intent = new Intent(mContext, otherActivity.class);
    intent.putExtra(ITEM_KEY, item);
    mContext.startActivity(intent);
  }
});
```

### 1.2.3 Retrieve Intent
```java
public class OtherActivity extends AppCompatActivity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    ...
    DataItem item = getIntent().getExtras().getParcelable(DataItemAdapter.DataItemAdapter);

    // initiate variables to point to View objects
    tvName = (TextView) findViewById(R.id.tvItemName);
    ...
    // use Parcelable object to set View objects
    tvName.setText(item.getItemName());
    ...
    // For inserting images using InputStream
    // InputStream inputStream = null;
    // try {
    //   String imageFile = item.getImage();
    //   inputStream = getAssets().get(imageFile);
    //   Drawable d = Drawable.createFromStream(inputStream, null);
    //   itemImage.setImageDrawable(d);
    // } catch (IOException e) { e.printStackTrace();
    // } finally {
    //     if (inputStream != null) {
    //       try { inputStream.close();
    //        catch (IOException e) { e.printStackTrace(); }}
    //     }
    // }

  }
}
```

## 1.3 Using Parcelable
So far we have already integrated Parcelable inside the code suggested, but the idea is that Parcelable allows us to pass complex data structures inside getIntent().getExtras()
Just remember that we need to install the Parcelable package before Parcelable is available.
