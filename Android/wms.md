WindowManagerService

WMS也属于系统服务，由SystemServer来启动，具体来说是 startOtherServices()，通过调用其main()方法创建。

1. 管理所有Window的状态，Window的状态保存在WindowState中，WMS中有一个集合 HashMap<IBinder, WindowState> mWindowMap = new HashMap<>()，用来保存所有的窗口状态；
2. 