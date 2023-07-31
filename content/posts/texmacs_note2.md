---
title: Texmacs/Mogan Note 2
date: 2023-07-13
tags: [note, code, mogan, texmacs]
---
# 23_15
```scheme
(get-init-tree "text-at-halign")
```
这个与缓存有关。缓存位置在`~/.Texmacs/`。如果不更改缓存，更改源代码不会生效。

# 23_7
```scheme
(tm-define (graphics-release-left x y)
  ;;(display* "Graphics] Release-left " x ", " y "\n")
  (if (inside-graphical-text?)
      (with-innermost t graphical-text-context?
        (let* ((ps (select-first (s2f x) (s2f y)))
               (p (and ps (car ps))))
          (if (and p (list-starts? p (tree->path t)))
              (go-to p)
              (tree-go-to t :start))))
      (edit_left-button (car (graphics-mode)) x y)))
```

```scheme
(tm-define (get-keyboard-modifiers)
  the-keyboard-modifiers)

(tm-define (set-keyboard-modifiers mods)
  (set! the-keyboard-modifiers mods))
```

结论：https://github.com/XmacsLabs/mogan/pull/796


# 问题
1. .ts文件怎么使用
2. C++代码中有很多字符串类型的东西可以用枚举类型来替代，可以重构一下？
3. 绘图模式有没有给座标系标注1、2、3的模式

# 编译时候出现问题
```
generating scheme glueA glue_font ... ok
generating scheme glueA glue_analyze ... ok
generating scheme glueA glue_drd ... ok
installing libmogan ..
error: install failed, method 'trim' is not callable (a nil value)
```

看起来是lua语言中的trim方法无法调用。
结论：有一个环境变量字符串被设置为nil，之后在他的上面调用trim方法则报错。

# 23_14
`edit_interface_rep::handle_mouse` 处理鼠标事件。

```cpp
edit_interface_rep::handle_mouse (string kind, SI x, SI y, int m, time_t t,
                                  array<double> data)
```
kind的类型可以有move, press-left, release-left, press-right 等。

```cpp
else {
    string rew= kind;
    SI dist= (SI) (5 * PIXEL / magf);
    rew= detect_left_drag ((void*) this, rew, x, y, t, m, dist);
    if (rew == "start-drag-left") {
      call_mouse_event (rew, left_x, left_y, m, t, data);
      delayed_call_mouse_event ("dragging-left", x, y, m, t, data);
    }
    else {
      rew= detect_right_drag ((void*) this, rew, x, y, t, m, dist);
      if (rew == "start-drag-right") {
        call_mouse_event (rew, right_x, right_y, m, t, data);
        delayed_call_mouse_event ("dragging-right", x, y, m, t, data);
      }
      else call_mouse_event (rew, x, y, m, t, data);
    }
  }
```
这段检测了是否有拖动发生，如果没有，最终会执行`call_mouse_event (rew, x, y, m, t, data);`

```cpp
static void
call_mouse_event (string kind, SI x, SI y, SI m, time_t t, array<double> d) {
  array<object> args;
  args << object (kind) << object (x) << object (y)
       << object (m) << object ((double) t) << object (d);
  call ("mouse-event", args);
}
```
然后转发给scheme来处理。

```scheme
(tm-define (mouse-event key x y mods time data)
  ;;(display* "mouse-event " key ", " x ", " y ", " mods ", " time ", " data "\n")
  (mouse-any key x y mods (+ time 0.0) data))
```
转到`mouse-any`

`mouse-any`再转回C++

```lua
{
                scm_name = "mouse-any",
                cpp_name = "mouse_any",
                ret_type = "void",
                arg_list = {
                    "string",
                    "int",
                    "int",
                    "int",
                    "double",
                    "array_double"
                }
            },
```

在`mouse_any`中进行dispatch. 

```cpp
  if (type == "press-left" || type == "start-drag-left") {
    if (mods > 1) {
      mouse_adjusting = mods;
      mouse_adjust_selection(x, y, mods);
    } else
      mouse_click (x, y);
  }
```

在这里进入mouse_click函数

```cpp
void
edit_interface_rep::mouse_click (SI x, SI y) {
  if (mouse_message ("click", x, y)) return;
  start_x= x;
  start_y= y;
  send_mouse_grab (this, true);
}
```

思路改为倒着找，从绘图的移动功能出发。因为只需要在绘图上进行改变鼠标样式的操作，而改变鼠标样式(`setCursor`)是依赖于具体的widget来执行的。

```scheme
(tm-define (edit_move mode x y)
```

```scheme
(tm-define (graphics-move x y)
  ;;(display* "Graphics] Move " x ", " y "\n")
  (when (not (inside-graphical-text?))
    (edit_move (car (graphics-mode)) x y)))
```

根据`("Move objects" (graphics-set-mode '(group-edit move)))`, 应该重载到group-edit版本。

测试发现正确:

```scheme
(tm-define (edit_move mode x y)
  (:require (eq? mode 'group-edit))
  (:state graphics-state)
  (display* "HERE\n")
  (cond (sticky-point
         (set! x (s2f x))
         (set! y (s2f y))
         (with mode (graphics-mode)
           (cond ((== (cadr mode) 'move)
                  (sketch-transform
                   (group-translate (- x group-old-x)
                                    (- y group-old-y))))
                 ((== (cadr mode) 'zoom)
                  (sketch-set! group-first-go)
                  (sketch-transform (group-zoom x y)))
                 ((== (cadr mode) 'rotate)
                  (sketch-set! group-first-go)
                  (sketch-transform (group-rotate x y)))))
         (set! group-old-x x)
         (set! group-old-y y))
        (multiselecting
         (graphical-object!
          (append
           (create-graphical-props 'default #f)
           `((with color red
               (cline (point ,selecting-x0 ,selecting-y0)
                      (point ,x ,selecting-y0)
                      (point ,x ,y)
                      (point ,selecting-x0 ,y)))))))
        (else
          (cond (current-path
                 (set-message (string-append "Left click: operate; "
                                             "Shift+Left click or Right click: select/unselect")
                              "Group of objects"))
                ((nnull? (sketch-get))
                 (set-message "Left click: operate"
                              "Group of objects"))
                (else
                  (set-message "Move over object on which to operate"
                               "Edit groups of objects")))
          (graphics-decorations-update))))
```

之后倒到`edit_graphics.cpp`中的:
```cpp
bool
edit_graphics_rep::mouse_graphics (string type, SI x, SI y, int mods, time_t t,
                                   array<double> data) {
```

倒退到
```cpp
bool
edit_graphics_rep::mouse_graphics (string type, SI x, SI y, int mods, time_t t,
                                   array<double> data) {
```

然后倒退到scheme中。没找到可以利用的。

换思路，搜索`: public QWidget`,看都有谁继承了。因为setCursor是它的成员函数。QtMainWindow也是Qt库中的，它也继承了QWidget.

看不出。直接看主界面怎么生成的吧。

`qt_tm_widget.hpp`中写了：

```cpp
  QMainWindow* mainwindow () {
    return qobject_cast<QMainWindow*> (qwid); 
  }
```

这个函数可以获取主页面的QWidget. 但它仍然是一个成员函数所以可能不太对。

在`qt_dialogues.cpp`中,发现了

```cpp
 QWidget* mainwindow = QApplication::activeWindow ();
```
这个可能是Qt自带的找到当前活跃窗口的函数。

# 23_14 Bug
```scheme
(tm-define (graphics-get-property var)
  (with val (graphics-get-raw-property var)
    (tm->stree val)))

(define (graphics-get-raw-property var)
  (with val (get-upwards-tree-property (graphics-graphics-path) var)
    (if (eq? val nothing)
        (get-default-tree-val var)
        (if (eq? (tm-car val) 'quote)
            (tree-ref val 0)
            val))))
```

## 研究 get-env-tree
```cpp
tree
edit_typeset_rep::get_env_value (string var, path p) {
  typeset_exec_until (p);
  tree t= cur[p][var];
  return is_func (t, BACKUP, 2)? t[0]: t;
}
```

## 鼠标点击研究
```cpp
  if (type == "press-left" || type == "start-drag-left") {
    if (mods > 1) {
      mouse_adjusting = mods;
      mouse_adjust_selection(x, y, mods);
    } else
      mouse_click (x, y);
  }
```

之后进入
```cpp
void
edit_interface_rep::mouse_click (SI x, SI y) {
  cout << "DEBUG: ";
  if (mouse_message ("click", x, y)) return;
  start_x= x;
  start_y= y;
  send_mouse_grab (this, true);
}
```

之后 
```cpp
inline void
send_mouse_grab (widget w, bool get_grab) {
  // request a mouse grab for the widget
  send<bool> (w, SLOT_MOUSE_GRAB, get_grab);
}
```

在这里似乎捕获了
```cpp
void
qt_tm_widget_rep::send (slot s, blackbox val) {
  switch (s) {
    case SLOT_INVALIDATE:
    case SLOT_INVALIDATE_ALL:
    case SLOT_EXTENTS:
    case SLOT_SCROLL_POSITION:
    case SLOT_ZOOM_FACTOR:
    case SLOT_MOUSE_GRAB:
      main_widget->send(s, val);
      return;
```

似乎会跑到这里
```cpp
    case SLOT_MOUSE_GRAB:
    {
      check_type<bool> (val, s);
      bool grab = open_box<bool>(val);
      if (grab && canvas() && !canvas()->hasFocus())
        canvas()->setFocus (Qt::MouseFocusReason);
    }
      break;
```




