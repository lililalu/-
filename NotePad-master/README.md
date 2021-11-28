# NotePad
This is an AndroidStudio rebuild of google SDK sample NotePad
实现了两个基础功能，时间戳和搜索
改变颜色没能运行但有代码
时间戳代码：
Date nowTime = new Date(System.currentTimeMillis());
SimpleDateFormat sdFormatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String retStrFormatNowDate = sdFormatter.format(nowTime);
//The columns needed by the cursor adapter
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
    };
private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
private int[] viewIDs = { Android：R.id.text1,Android：R.id.text2 };

搜索代码：
public class NoteSearch extends Activity implements SearchView.OnQueryTextListener{
    ListView listview;//
    SQLiteDatabase sqLiteDatabase;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 修改时间
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);//设置全屏显示
        super.setContentView(R.layout.note_search);
        Intent intent = getIntent();

        // If there is no data associated with the Intent, sets the data to the default URI, which
        // accesses a list of notes.
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listview= (ListView) findViewById(R.id.list_view);//获取listview
        sqLiteDatabase=new NotePadProvider.DatabaseHelper(this).getReadableDatabase();//对数据库进行操作
        SearchView search= (SearchView) findViewById(R.id.search_view);//获取搜索视图
        search.setOnQueryTextListener(NoteSearch.this);
        listview.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
                //为每个item添加点击事件，点击可以查看笔记具体内容
                Uri uri = ContentUris.withAppendedId(getIntent().getData(), l);
                String action = getIntent().getAction();
                if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
                    setResult(RESULT_OK, new Intent().setData(uri));
                } else {
                    startActivity(new Intent(Intent.ACTION_EDIT, uri));
                }
            }
        });
    }

    public boolean onQueryTextSubmit(String query) {
        return true;
    }

    public boolean onQueryTextChange(String newText) {//实现模糊查询，通过标题或者内容进行查询
        Cursor cursor=sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION,
                NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?",
                new String[]{"%"+newText+"%","%"+newText+"%"},
                null,
                null,
                NotePad.Notes.DEFAULT_SORT_ORDER);
        int[] viewIDs = { android.R.id.text1 , android.R.id.text2};
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                NoteSearch.this,
                R.layout.noteslist_item,
                cursor,
                new String[]{NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE},
                viewIDs
        );
        listview.setAdapter(adapter);
        return true;
    }
}
