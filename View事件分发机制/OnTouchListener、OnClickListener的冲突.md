# OnTouchListener、OnClickListener的冲突

1. **触发顺序**：onTouch() > onTouchEvent() > onClick()

```
bt.setOnTouchListener(new View.OnTouchListener() {
       @Override
       public boolean onTouch(View v, MotionEvent event) {
            //do something
       return false;
       }
});
```



```
bt.setOnClickListener(new View.OnClickListener() {
       @Override
       public void onClick(View v) {
            //do something
       }
});
```



```
@Override
public boolean onTouchEvent(MotionEvent event) {
        //do something
        return super.onTouchEvent(event);
}
```

2. **Warning**：当对一个控件使用setOnTouchListener()或对自定义控件重写onTouchEvent()会出现警告

   > If a View that overrides onTouchEvent or uses an OnTouchListener does not also implement performClick and call it when clicks are detected, the View may not handle accessibility actions properly. Logic handling the click actions should ideally be placed in View#performClick as some accessibility services invoke performClick when a click action should occur.

   大意是：如果重写了view的onTouchEvent方法或者设置了OnTouchListener，但没有实现performClick并在检测到点击时调用它，view可能无法正确处理辅助操作。理想情况下，处理点击操作的逻辑应放在View#performClick中，因为某些辅助功能服务会在发生单击操作时调用performClick。

3. **原因**：onClick()会通过performClick()完成点击事件的，而在onToch()和onTouchEvent()的ACTION_UP过程中会启用一个新的线程来调用performClick()，因而可能会屏蔽掉onClick()中设置的事件

4. 解决办法：

   - 如果使用setOnTouchListner()，那么就在重写onTouch()的ACTION_UP的情况下调用performClick()

     ```
     bt.setOnTouchListener(new View.OnTouchListener() {
           @Override
           public boolean onTouch(View v, MotionEvent event) {
                switch (event.getAction()){
                         case MotionEvent.ACTION_DOWN:
                             break;
                         case MotionEvent.ACTION_MOVE:
                             break;
                         case MotionEvent.ACTION_UP:
                             //以下为添加内容
                             button.performClick();
                             break;
                 }
                 return false;//如果返回true，由于优先级较高，后面onTouchEvent、onClick将不会被触发
               }
          });
     ```

     

   - 如果重写onTouchEvent()，那么就在ACTION_UP情况下调用performClick()

     ```
     @Override
     public boolean onTouchEvent(MotionEvent ev) {
             switch (ev.getAction()) {
                     case MotionEvent.ACTION_DOWN:
                         break;
                     case MotionEvent.ACTION_CANCEL:
                         break;
                     //以下为添加内容
                     case MotionEvent.ACTION_UP:
                         performClick();
                         break;
                 }
             return true;
         }
     ```

     