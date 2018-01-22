###1.通过包名和布局文件名，加载另外一个应用的布局。
```
String pkg = "layout.source.package.name";
Resources resources = null;
try {
    resources = getPackageManager().getResourcesForApplication(pkg);
} catch (PackageManager.NameNotFoundException e) {
    e.printStackTrace();
}
if (resources != null) {
    int identifier = resources.getIdentifier("layout_test", "layout", pkg);
    RemoteViews remoteViews = new RemoteViews(pkg, identifier);
    View view = remoteViews.apply(this, null);
    container.addView(view);
}
```        