#####1.ContentProvider的onCreate和所在进程的Application的onCreate的调用顺序？
如果ContentProvider的宿主进程没有启动，那么就是ContentProvider的onCreate先被调用，接着才是Application的onCreate，这点和其他的三个主键是不一样的。

#####2.继承ContentProvider实现的5个方法运行那哪些线程中？
除了onCreate由系统回调运行在主线程（mH-->ActivityThread）,其他的四个方法query、update、insert、delete都运行在Binder线程池中。

#####3.在使用中通常需要识别不同的URI，即它们的path不同，借助UriMatcher可以建立Uri与UriCode的关联。
```
public static final String AUTHORITY = "com.jdqm.PROVIDER";
public static final String BOOK_URI = AUTHORITY + "/book";
public static final String USER_URI = AUTHORITY + "/user";

private static final int URI_BOOK_CODE = 1;
private static final int URI_USER_CODE = 2;

private static final UriMatcher sUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);

static {
    sUriMatcher.addURI(AUTHORITY, "book", URI_BOOK_CODE);
    sUriMatcher.addURI(AUTHORITY, "user", URI_USER_CODE);
}


private String getTableName(Uri uri) {
    int match = sUriMatcher.match(uri);
    switch (match) {
        case URI_BOOK_CODE:
            return "BOOK";
        case URI_USER_CODE:
            return "USER";
        default:
           return null;
    }
}

```