# NotePad
## 实现时间戳和搜索功能的NotePad<br>

<!-- ![Image text](https://github.com/Cappuccino-Luo/NotePad/blob/master/NotePad1/pictures/1.png)<br> -->
<!-- 导入图片的格式有两种，上一行的一种和下两行的一种，但是用了html格式后上面一行的导入好像不可以再用了。这种文件与html有很多相同的，除一部分自己格式的特色（猜测）。 -->

NoteList使用SimpleCursorAdapter来装配数据，首先查询数据库的内容<br>

        Cursor cursor = getContentResolver().query(
                       getIntent().getData(),            // Use the default content URI for the provider.
                       PROJECTION,                       // Return the note ID ,  title and time for each note.
                       null,                             // No where clause, return all records.
                       null,                             // No where clause, therefore no where column values.
                       NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                    );

然后通过SimpleCursorAdapter来进行装配

        SimpleCursorAdapter adapter
                    = new SimpleCursorAdapter(
                              this,                             // The Context for the ListView
                              R.layout.noteslist_item,          // Points to the XML for a list item
                              cursor,                           // The cursor to get items from
                              dataColumns,
                              viewIDs
                      );

页面跳转：<br>
不管是可选菜单、上下文菜单中的操作，还是单击列表中的笔记条目，其相应的页面跳转都是通过Intent的Action+URI进行的。<br>
### 功能设计介绍
#### 添加时间戳功能：
1.添加时间戳的位置在主页面的每个列表项中添加，即在notelist_item.xml布局文件中添加一个

        <TextView
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true" />
        
2.在NoteList类的PROJECTION中添加COLUMN_NAME_MODIFICATION_DATE字段(该字段在NotePad中有说明) 

        private static final String[] PROJECTION = new String[] {
                    NotePad.Notes._ID, // 0
                    NotePad.Notes.COLUMN_NAME_TITLE, // 1
                    NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // time
            };
            
3.修改适配器内容，增加dataColumns中装配到ListView的内容，所以要同时增加一个ID标识来存放该时间参数。

                // The names of the cursor columns to display in the view, initialized to the title column
                String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,
                     NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE //增加时间参数} ;
                // The view IDs that will display the cursor columns, initialized to the TextView in noteslist_item.xml
                int[] viewIDs = { android.R.id.text1 ,android.R.id.text2};

4.在NoteEditor文件的updateNote方法中获取当前系统的时间，并对时间进行格式化

                        ContentValues values = new ContentValues();
                        Long now = Long.valueOf(System.currentTimeMillis());
                        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                        String time = sdf.format(new Date(Long.parseLong(String.valueOf(now))));
                        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, time);

#### 添加搜索框功能：
1.搜索组件在主页面的菜单选项中，所以应在list_options_menu.xml布局文件中添加搜索功能

    <item
        android:id="@+id/search"
        android:title="@string/search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always"
        tools:ignore="AppCompatResource">
    </item>

2.新建一个查找笔记内容的布局文件activity_search_result.xml;

                <?xml version="1.0" encoding="utf-8"?>
                <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                    xmlns:app="http://schemas.android.com/apk/res-auto"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical">
                    <LinearLayout
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:orientation="horizontal">
                        <Button
                            android:id="@+id/btn_back"
                            android:layout_width="66dp"
                            android:text="👈"
                            android:textSize="22.6dp"
                            android:background="#FAFAFA"
                            android:layout_height="wrap_content"/>
                        <SearchView
                            android:id="@+id/search"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:layout_alignParentTop="true"
                            android:iconifiedByDefault="false"
                            android:queryHint="Search for:" />
                    </LinearLayout>
                    <ListView
                        android:id="@android:id/list"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content" />
                </LinearLayout>

3.在NoteList类中的onOptionsItemSelected方法中添加search查询的处理(跳转)

        case R.id.search:
            Intent i = new Intent(NotesList.this,search_result.class);
            startActivity(i);
            return true;

4.新建一个search_result类用于search功能的功能实现

        package com.x1nge.notepaddemo2;
        import android.app.ListActivity;
        import android.content.ContentUris;
        import android.content.Intent;
        import android.database.Cursor;
        import android.graphics.drawable.Drawable;
        import android.net.Uri;
        import android.os.Bundle;
        import android.view.View;
        import android.widget.Button;
        import android.widget.EditText;
        import android.widget.ListView;
        import android.widget.SearchView;
        import android.widget.SimpleCursorAdapter;

        public class search_result extends ListActivity implements SearchView.OnQueryTextListener {

    /**
     * The columns needed by the cursor adapter
     */
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // time
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_search_result);

        Button btn_back = findViewById(R.id.btn_back);
        Drawable drawable = getDrawable(R.drawable.ic_menu_revert);
        btn_back.setCompoundDrawables(drawable,null, null, null);

        btn_back.setOnClickListener(new View.OnClickListener(){
            public void onClick(View v){
                Intent i = new Intent(search_result.this,NotesList.class);
                startActivity(i);
            }
        });

        SearchView sv = findViewById(R.id.search);

        // The user does not need to hold down the key to use menu shortcuts.
        setDefaultKeyMode(DEFAULT_KEYS_SHORTCUT);

        /* If no data is given in the Intent that started this Activity, then this Activity
         * was started when the intent filter matched a MAIN action. We should use the default
         * provider URI.
         */
        // Gets the intent that started this Activity.
        Intent intent = getIntent();

        // If there is no data associated with the Intent, sets the data to the default URI, which
        // accesses a list of notes.
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }

        //        // 设置是否自动缩小成图标
        ////        sv.setIconifiedByDefault(true);

        // 设置是否显示搜索按钮
        sv.setSubmitButtonEnabled(true);

        // 监听
        sv.setOnQueryTextListener(this);
    }

    @Override
    public boolean onQueryTextSubmit(String query) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like?";
        String[] selectionArgs = { "%" + query + "%" };
        String sortOrder = NotePad.Notes.COLUMN_NAME_TITLE + " DESC";

        /* Performs a managed query. The Activity handles closing and requerying the cursor
         * when needed.
         *
         * Please see the introductory note about performing provider operations on the UI thread.
         */
        //        Cursor cursor = managedQuery(
        //                getIntent().getData(),            // Use the default content URI for the provider.
        //                PROJECTION,                       // Return the note ID and title for each note.
        //                selection,                             // No where clause, return all records.
        //                selectionArgs,                             // No where clause, therefore no where column values.
        //                sortOrder
        ////                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                //        );
        Cursor cursor = getContentResolver().query(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note.
                selection,                             // No where clause, return all records.
                selectionArgs,                             // No where clause, therefore no where column values.
                sortOrder
        //                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                );

        /*
         * The following two arrays create a "map" between columns in the cursor and view IDs
         * for items in the ListView. Each element in the dataColumns array represents
         * a column name; each element in the viewID array represents the ID of a View.
         * The SimpleCursorAdapter maps them in ascending order to determine where each column
         * value will appear in the ListView.
         */

        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE , NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;

        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = { android.R.id.text1,R.id.time };

        // Creates the backing adapter for the ListView.
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,          // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );

        // Sets the ListView's adapter to be the cursor adapter that was just created.
        setListAdapter(adapter);
        return true;
    }

    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like?";
        String[] selectionArgs = { "%" + newText + "%" };
        String sortOrder = NotePad.Notes.COLUMN_NAME_TITLE + " DESC";

        /* Performs a managed query. The Activity handles closing and requerying the cursor
         * when needed.
         *
         * Please see the introductory note about performing provider operations on the UI thread.
         */
        //        Cursor cursor = managedQuery(
        //                getIntent().getData(),            // Use the default content URI for the provider.
                //                PROJECTION,                       // Return the note ID and title for each note.
        //                selection,                             // No where clause, return all records.
        //                selectionArgs,                             // No where clause, therefore no where column values.
        //                sortOrder
        ////                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        //        );
        Cursor cursor = getContentResolver().query(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note.
                selection,                             // No where clause, return all records.
                selectionArgs,                             // No where clause, therefore no where column values.
                sortOrder
        );
        //                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.

        /*
         * The following two arrays create a "map" between columns in the cursor and view IDs
         * for items in the ListView. Each element in the dataColumns array represents
         * a column name; each element in the viewID array represents the ID of a View.
         * The SimpleCursorAdapter maps them in ascending order to determine where each column
         * value will appear in the ListView.
         */

        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE , NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;

        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = { android.R.id.text1,R.id.time };

        // Creates the backing adapter for the ListView.
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,          // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );

        // Sets the ListView's adapter to be the cursor adapter that was just created.
        setListAdapter(adapter);
        return true;
    }

    /**
     * This method is called when the user clicks a note in the displayed list.
     *
     * This method handles incoming actions of either PICK (get data from the provider) or
     * GET_CONTENT (get or create data). If the incoming action is EDIT, this method sends a
     * new Intent to start NoteEditor.
     * @param l The ListView that contains the clicked item
     * @param v The View of the individual item
     * @param position The position of v in the displayed list
     * @param id The row ID of the clicked item
     */
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {

        // Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);

        // Gets the action from the incoming Intent
        String action = getIntent().getAction();

        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {

            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {

            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
 }

5.最后要在清单文件AndroidManifest.xml里面注册NoteSearch,否则无法实现界面的跳转

        <activity android:name=".search_result"></activity>


<div align="center">效果截图</div>
<div align="center"><img src="https://github.com/Cappuccino-Luo/NotePad/blob/master/NotePad1/pictures/1.png"></div>
<p align="center">进入NotePad时的界面</p>
<div align="center"><img src="https://github.com/Cappuccino-Luo/NotePad/blob/master/NotePad1/pictures/2.png"></div>
<p align="center">添加记录的界面</p><br>
<div align="center"><img src="https://github.com/Cappuccino-Luo/NotePad/blob/master/NotePad1/pictures/3.png"></div>
<p align="center">添加完成后的界面</p><br>
<div align="center"><img src="https://github.com/Cappuccino-Luo/NotePad/blob/master/NotePad1/pictures/4.jpg"></div>
<p align="center">搜索功能</p><br>
