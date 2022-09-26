# AndroidMethodHook
结合阿里Sophix热修复方案(https://yq.aliyun.com/articles/74598?t=t1#)<br>
使用了dexmaker库，主要用来动态生成dex文件。<br>
使用方法：
``` java
HookManager.findAndHookMethod(MainActivity.class, "onCreate", Bundle.class, new MethodCallback() {
    @Override
    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
        super.beforeHookedMethod(param);
        Log.d("panda", "onCreate:"+param.thisObject.getClass().getName());
    }

    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
        super.afterHookedMethod(param);
        Log.d("panda", "i'm in method " +param.method.getName()+" afterHookedMethod");
    }
});
HookManager.startHooks(base);
```
通过对Native JmethodId内容替换实现method hook，为了达到类似于xposed的效果，需要动态生成method，使用了dexmaker库用以动态生成mehtod。<br><br>
每个需要hook的方法都会dexmaker生成一个一摸一样的方法，将this和传入参数封装成Object[] args传给MethodUtil类的invoke函数，然后回调MethodCallback实现类似于xposed mehtod hook。<br>
生成代理method方式如下：<br>

* MainActivity
``` java
package com.panda.hook.andhook;

public class MainActivity extends AppCompatActivity  {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
     public  double test(Object thiz,int a,int b,char cr){
        return (a+0.0)/b;
    }
}
```
* com_panda_hook_andhook_MainActivity
``` java
public class com_panda_hook_andhook_MainActivity {
  public com_panda_hook_andhook_MainActivity() {
      super();
  }

  public double test(Object arg12, int arg13, int arg14, char arg15) {
      return MethodUtil.invoke("com_panda_hook_andhook_MainActivity_testLIICD", this, new Object[]{
              arg12, Integer.valueOf(arg13), Integer.valueOf(arg14), Character.valueOf(arg15)}).doubleValue();
  }
}
```
