---
title: "TeXmacs/Mogan Note 3"
date: 2023-08-02T15:57:15+08:00
tag: [note, code, mogan, texmacs]
---

## 修改为两点画圆
研究三点画圆的代码实现

菜单中选择画圆的时候使用
```scheme
("Circle" (graphics-set-mode '(edit carc)))
```
但是edit carc没有继续出现。因此推测是得到mode之后判断，如果第一个值是edit则统一处理，不需要分别处理。

以
```scheme
(tm-define (edit_right-button mode x y)
  (:require (== mode 'edit))
  (:state graphics-state)
  (set-texmacs-pointer 'graphics-cross)
  (when current-obj
    (graphics-delete)))
```
为例子。

它在这里被调用

```scheme
(tm-define (graphics-release-right x y)
  ;;(display* "Graphics] Release-right " x ", " y "\n")  
  (when (not (inside-graphical-text?))
    (edit_right-button (car (graphics-mode)) x y)))
```
可以看到确实是按照`(car (graphics-mode))`来进行调用。

---

研究如何通过菜单进行markup中的两点画圆

```scheme
(menu-bind graphics-focus-menu
  (-> (eval (upcase-first (gr-mode->string (graphics-mode))))
      (link graphics-mode-menu))
  (if (inside-graphical-over-under?)
      ("Exit graphics" (graphics-exit-right)))
  (assuming (nnot (tree-innermost overlays-context?))
    (link graphics-focus-overlays-menu))
  (assuming (nnull? (graphics-mode-attributes (graphics-mode)))
    ---
    (assuming (graphics-mode-attribute? (graphics-mode) "color")
      (-> "Color" (link graphics-color-menu)))
    (assuming (graphics-mode-attribute? (graphics-mode) "fill-color")
      (-> "Fill color" (link graphics-fill-color-menu)))
    (assuming (graphics-mode-attribute? (graphics-mode) "opacity")
      (assuming (== (get-preference "experimental alpha") "on")
        (-> "Opacity" (link graphics-opacity-menu))))
    (assuming (graphics-mode-attribute? (graphics-mode) "pen-enhance")
      (-> "Enhance" (link graphics-pen-enhance-menu)))
```

这是焦点工具栏，在菜单中的版本。

```scheme
(tm-menu (graphics-property-icons)
```

这个是图形界面中的第三栏。除了属性卡之外的选项们。

```scheme
(tm-menu (graphics-icons)
  (link graphics-global-icons)
  /
  (link graphics-insert-icons)
  /
  (link graphics-group-icons))

(tm-menu (graphics-focus-icons)
  (mini #t
    (=> (balloon (eval (upcase-first (gr-mode->string (graphics-mode))))
                 "Current graphical mode")
        (link graphics-mode-menu)))
  (assuming (nnot (tree-innermost overlays-context?))
    (link graphics-focus-overlays-icons))
  (assuming (nnull? (graphics-mode-attributes (graphics-mode)))
    (link graphics-property-icons))
  (assuming (graphics-get-anim-type)
    /
    (mini #t
      (group "Status:")
      (=> (eval (graphics-get-anim-type))
          (link graphics-anim-type-menu))))
  /
  (link graphics-snap-icons))
```
这是整体。

则事实上要找的含有markup选项的菜单为
```scheme
(menu-bind graphics-mode-menu
  ("Point" (graphics-set-mode '(edit point)))
  ("Line" (graphics-set-mode '(edit line)))
  ("Polygon" (graphics-set-mode '(edit cline)))
  (-> "Curve"
```

其中含有插件的为

```scheme
(assuming (style-has? "std-markup-dtd")
    (with u '(arrow-with-text arrow-with-text*)
      (with l (list-filter u (lambda (s) (style-has? (symbol->string s))))
        (for (tag (sort l symbol<=?))
          ((eval (upcase-first (symbol->string tag)))
           (import-from (graphics graphics-markup))
           (graphics-set-mode `(edit ,tag))))))
    (with u (list-difference gr-tags-user '(arrow-with-text arrow-with-text*))
      (with l (list-filter u (lambda (s) (style-has? (symbol->string s))))
        (assuming (nnull? l)
          ---
          (for (tag (sort l symbol<=?))
            ((eval (upcase-first (symbol->string tag)))
             (import-from (graphics graphics-markup))
             (graphics-set-mode `(edit ,tag))))))))
```
