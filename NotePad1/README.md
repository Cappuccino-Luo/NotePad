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

<div align="center"><img src="https://github.com/Cappuccino-Luo/NotePad/blob/master/NotePad1/pictures/1.png"></div>
<p align="center">进入NotePad时的界面</p>
<div align="center"><img src="https://github.com/Cappuccino-Luo/NotePad/blob/master/NotePad1/pictures/2.png"></div>
<p align="center">添加记录的界面</p><br>
<div align="center"><img src="https://github.com/Cappuccino-Luo/NotePad/blob/master/NotePad1/pictures/3.png"></div>
<p align="center">添加完成后的界面</p><br>
<div align="center"><img src="https://github.com/Cappuccino-Luo/NotePad/blob/master/NotePad1/pictures/4.jpg"></div>
<p align="center">搜索功能</p><br>
