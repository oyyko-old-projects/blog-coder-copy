---
title: Texmacs/Mogan Note
date: 2023-07-05
tags: [note, code, mogan, texmacs]
---
# Note
## graphics-utils.scm
```scheme
;;These abbreviations are very convenient
;;to use. A nice naming scheme is :
;;
;;  -> b=bool ;
;;  -> i=integer ;
;;  -> f=float ;
;;  -> sy=symbol ;
;;  -> s=string ;
;;  -> o=Scheme object ;
;;  -> p=path.
;;  -> t=tree.
;;
;;  One can add the missing ones on demand.
(tm-define f2s float->string)
(tm-define s2f string->float)
(tm-define sy2s symbol->string)
(tm-define s2sy string->symbol)
(tm-define o2s object->string)
(tm-define s2o string->object)
(tm-define t2o tree->object)
(tm-define o2t object->tree)
```

## graphics-single.scm
editing routines for single graphical objects.

## graphics-env.scm
routines for managing the graphical context.

current-x在此定义。

## graphics-object.scm
routines for managing the graphical object.


## 研究图形移动功能
在edit_move函数中有下面一段
```scheme
(cond ((== (cadr mode) 'move)
                  (sketch-transform
                   (group-translate (- x group-old-x)
                                    (- y group-old-y))))
```

其中sketch-transform的定义为
```scheme
(tm-define (sketch-transform opn)
  (set! the-sketch (map opn the-sketch))
  (set! current-obj
	(if (graphics-group-mode? (graphics-mode)) '(nothing) #f))
  (set! current-path #f)
  (graphics-decorations-update))
```

而group-translate定义为：
```scheme
(define (group-translate x y)
  (lambda (o)
    (traverse-transform o (translate-point x y))))
```
其中用到了
```scheme
(define (traverse-transform o opn)
  (define (traverse o)
    (opn (if (pair? o) (map traverse o) o)))
  (traverse o))

(define (translate-point x y)
  (lambda (o)
    (if (match? o '(point :%2))
	`(point ,(f2s (+ x (s2f (cadr o)))) ,(f2s (+ y (s2f (caddr o)))))
        o)))
```

## 研究测试相关代码
## tm-ref
```scheme
(define-public (tm-ref t . l)
  (and (tm? t)
       (with r (select t l)
	 (and (nnull? r) (car r)))))
```
## tm?
```cpp
  tmscm_install_procedure ("tm?", contentP, 1, 0, 0);
```

```cpp
tmscm
contentP (tmscm t) {
  bool b= tmscm_is_content (t);
  return bool_to_tmscm (b);
}
```

```cpp
bool
tmscm_is_content (tmscm p) {
  if (tmscm_is_string (p) || tmscm_is_tree (p)) return true;
  else if (!tmscm_is_pair (p) || !tmscm_is_symbol (tmscm_car (p))) return false;
  else {
    for (p= tmscm_cdr (p); !tmscm_is_null (p); p= tmscm_cdr (p))
      if (!tmscm_is_pair (p) || !tmscm_is_content (tmscm_car (p))) return false;
    return true;
  }
}
```

因此tm-ref等代码实际上应用于tree数据结构。`tm?`就是用来检测这个的。例如：
```scheme
Scheme]  (tm? '(point 1 2))

#f
Scheme]  (tm? (stree->tree '(point 1 2)))

#t
```
之后tm-ref可以取出列表中除了car的第i个元素。
```scheme
Scheme]  (tm-ref (stree->tree '(1 2 3 4)) 0)

2
```

## 研究宏包
为什么有的包已经引入，有的需要作为宏包引入呢？
init中会初始化一些包。
## SRFI
列表长度length

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

## 23_15
```cpp
if (N(p) == 0)
      typeset_dynamic (tree (ERROR, "bad text-at"), ip);
    else {
      SI ox= (SI) p[0], oy= (SI) p[1], axis= (b->h() >> 1), x= ox, y= oy;
      if (halign == "left") x -= b->x1;
      else if (halign == "center") x -= ((b->x1 + b->x2) >> 1);
      else if (halign == "right") x -= b->x2;
      if (valign == "bottom") y -= b->y1;
      else if (valign == "axis") {
        axis= env->fn->yfrac - b->y1;
        y -= env->fn->yfrac;
      }
      else if (valign == "center") y -= ((b->y1 + b->y2) >> 1);
      else if (valign == "top") y -= b->y2;
      SI snap= env->get_length (TEXT_AT_SNAPPING);
      print (text_at_box (ip, b, x, y, ox - x, oy - y, axis, snap));
      SI pad = env->text_at_repulse;
      if (pad >= 0)
        env->white_zones << rectangle (x + b->x1 - pad, y + b->y1 - pad,
                                       x + b->x2 + pad, y + b->y2 + pad);
    }
```
在这里设置了偏移量。

```scheme
(tm-define (edit_left-button mode x y)
  (:require (== mode 'edit))
  (:state graphics-state)
  (display "graphics-single 377\n")
  (display* sticky-point "\n")
  (display* (current-in? (graphical-text-tag-list)) "\n")
  (set-texmacs-pointer 'graphics-cross)
  (cond (sticky-point
         (if (current-in? (graphical-text-tag-list))
             (object_commit)
             (next-point)))
        ((and (current-in? (graphical-text-tag-list))
              (== (car (graphics-mode)) 'edit)
              (graphical-contains-text-tag? (cadr (graphics-mode)))
              (not (graphical-contains-curve-tag? (cadr (graphics-mode))))
              (pointer-inside-graphical-text?))
         (set-texmacs-pointer 'text-arrow)
         (go-to (car (select-first (s2f current-x) (s2f current-y)))))
        (else
         (edit-insert x y)
         (display* "HERE\n")))
  (set! previous-leftclick `(point ,current-x ,current-y)))
```
进入`edit-insert`

```scheme
(define (edit-insert x y)
  (edit-clean-up)
  (object_create (cadr (graphics-mode)) x y))
```

进入`object_create`

```scheme
(tm-define (object_create tag x y)
  (texmacs-error "object-create" "invalid tag"))

(tm-define (object_create tag x y)
  (:require (== tag 'point))
  (object-set! `(point ,x ,y) 'new))

(tm-define (object_create tag x y)
  (:require (or (in? tag gr-tags-curves) (in? tag gr-tags-user)))
  (with o (graphics-enrich `(,tag (point ,x ,y) (point ,x ,y)))
    (graphics-store-state 'start-create)
    (set! current-point-no 1)
    (object-set! o 'checkout)
    (graphics-store-state #f)))

(tm-define (object_create tag x y)
  (:require (graphical-text-tag? tag))
  (with long? (graphical-long-text-tag? tag)
    (object-set! `(,tag ,(if long? `(document "") "") (point ,x ,y)) 'new)
    (and-with d (path->tree (cDr (cursor-path)))
      (when (tree-func? d 'document)
        (tree-go-to d 0 :start)))))
```

得到对应的重载为
```scheme
(tm-define (object_create tag x y)
  (:require (graphical-text-tag? tag))
  (display* "HERE\n")
  (with long? (graphical-long-text-tag? tag)
    (object-set! `(,tag ,(if long? `(document "") "") (point ,x ,y)) 'new)
    (and-with d (path->tree (cDr (cursor-path)))
      (when (tree-func? d 'document)
        (tree-go-to d 0 :start)))))
```

进入了这一行`(graphics-group-enrich-insert o)`

```scheme
(tm-define (graphics-group-enrich-insert t)
  (graphics-group-insert (graphics-enrich t)))
```

```scheme
(tm-define (graphics-enrich t)
  (let* ((l1 (graphics-all-attributes))
         (l2 (map gr-prefix l1))
         (l3 (map graphics-get-property l2))
         (tab (list->ahash-table (map cons l1 l3))))
    (graphics-enrich-bis t "default" tab)))
```

```scheme
(tm-define (graphics-enrich-bis t id tab)
  (set! tab (list->ahash-table (ahash-table->list tab)))
  (ahash-remove! tab "gid")
  (let* ((attrs (graphical-relevant-attributes t))
         (sel (ahash-table-select tab attrs))
         (l1 (cons (cons "gid" id) (ahash-table->list sel)))
         (l2 (map (lambda (x) (list (car x) (cdr x))) l1)))
    ;;(display* "l= " l2 "\n")
    (graphics-enrich-sub t l2)))
```


```cpp
class concater_rep {
  edit_env              env;        // the environment
  array<line_item>      a;          // the line items
  bool                  rigid;      // when surely not wrappable
```

## 设置属性
```scheme
(tm-define (object-set-text-at-halign val)
  (:argument val "Horizontal alignment")
  (:check-mark "*" (object-test-property? "text-at-halign"))
  (object-set-property "text-at-halign" val))
```

```scheme
(tm-define (object-set-property var val)
  (and-with t (tree-innermost graphical-context?)
    (object-set-property-bis t var val)))
```

```scheme
(define (object-set-property-bis t var val)
  (cond ((tree-is? t :up 'with)
         (with-set (tree-up t) var val 0))
        ((!= val "default")
         (tree-set! t `(with ,var ,val ,t)))))
```

## 菜单
```scheme
(tm-menu (text-at-halign-menu)
  ("Center" (object-set-text-at-halign "default"))
  ("Right" (object-set-text-at-halign "right"))
  ("Left1111" (object-set-text-at-halign "left")))
```


```scheme
(tm-define-macro (tm-menu head . l)
  (receive (opts body) (list-break l not-define-option?)
    `(tm-define ,head ,@opts (menu-dynamic ,@body))))
```

```scheme
(tm-define-macro (menu-dynamic . l)
  `($list ,@(map gui-make l)))
```

```scheme
(tm-define (gui-make x)
  ;;(display* "x= " x "\n")
  (cond ((symbol? x)
         (cond ((== x '---) '$---)
               ((== x '===) (gui-make '(glue #f #f 0 5)))
               ((== x '======) (gui-make '(glue #f #f 0 15)))
               ((== x '/) '$/)
               ((== x '//) (gui-make '(glue #f #f 5 0)))
               ((== x '///) (gui-make '(glue #f #f 15 0)))
               ((== x '>>) (gui-make '(glue #t #f 5 0)))
               ((== x '>>>) (gui-make '(glue #t #f 15 0)))
               ((== x (string->symbol "|")) '$/)
               (else
                 (texmacs-error "gui-make" "invalid menu item ~S" x))))
        ((string? x) x)
        ((and (pair? x) (ahash-ref gui-make-table (car x)))
         (apply (car (ahash-ref gui-make-table (car x))) (list x)))
        ((and (pair? x) (or (string? (car x)) (pair? (car x))))
         `($> ,(gui-make (car x)) ,@(cdr x)))
        (else
          (texmacs-error "gui-make" "invalid menu item ~S" x))))
```
菜单中的勾选
可能是check-mark和object-test-property?

```scheme
(tm-define (object-set-text-at-halign val)
  (:argument val "Horizontal alignment")
  (:check-mark "*" (object-test-property? "text-at-halign"))
  (object-set-property "text-at-halign" val))
```

```scheme
(define (object-test-property? var)
  (lambda (val)
    (if (== val "default") (set! val (tree->stree (get-init-tree var))))
    (== (object-get-property var) val)))
```

下一步观察`get-init-tree` 使用了GLUE

```lua
{
                scm_name = "get-init-tree",
                cpp_name = "get_init_value",
                ret_type = "tree",
                arg_list = {
                    "string"
                }
            },
```
```cpp
tree
edit_typeset_rep::get_init_value (string var) {
  if (init->contains (var)) {
    tree t= init [var];
    if (var == BG_COLOR && is_func (t, PATTERN)) t= env->exec (t);
    return is_func (t, BACKUP, 2)? t[0]: t;
  }
  if (N(pre)==0) typeset_preamble ();
  tree t= pre [var];
  if (var == BG_COLOR && is_func (t, PATTERN)) t= env->exec (t);
  return is_func (t, BACKUP, 2)? t[0]: t;
}
```

查到一个叫init的hashmap
```cpp
class edit_typeset_rep: virtual public editor_rep {
protected:
  tree the_style;                         // document style
  hashmap<path,hashmap<string,tree> > cur; // environment at different paths
  hashmap<string,tree> stydef;            // environment after styles
  hashmap<string,tree> pre;               // environment after styles and init
  hashmap<string,tree> init;              // environment changes w.r.t. style
```

```cpp
void
edit_typeset_rep::set_init (hashmap<string,tree> H) {
  init= hashmap<string,tree> (UNINIT);
  add_init (H);
}

void
edit_typeset_rep::add_init (hashmap<string,tree> H) {
  init->join (H);
  ::notify_assign (ttt, path(), subtree (et, rp));
  notify_change (THE_ENVIRONMENT);
}
```

在这里调用了set_init
```cpp
void
edit_typeset_rep::set_data (new_data data) {
  set_style (data->style);
  set_init  (data->init);
  set_fin   (data->fin);
  set_ref   (data->ref);
  set_aux   (data->aux);
  set_att   (data->att);
  notify_page_change ();
  add_init (data->init);
  notify_change (THE_DECORATIONS);
  typeset_invalidate_env ();
  iterator<string> it = iterate (data->att);
  while (it->busy()) {
    string key= it->next ();
    (void) call (string ("notify-set-attachment"),
                 buf->buf->name, key, data->att [key]);
  }
}
```

`set_data`的调用位置是
```cpp
void
set_buffer_data (url name, new_data data) {
  array<url> vs= buffer_to_views (name);
  for (int i=0; i<N(vs); i++) {
    view_to_editor (vs[i]) -> set_data (data);
    view_to_editor (vs[i]) -> init_update ();
  }
}
```

`set_buffer_data`的调用位置是
```cpp
void
set_buffer_tree (url name, tree doc) {
  tm_buffer buf= concrete_buffer (name);
  if (is_nil (buf)) {
    insert_buffer (name);
    buf= concrete_buffer (name);
    tree body= detach_data (doc, buf->data);
    set_document (buf->rp, body);
    buf->buf->title= propose_title (buf->buf->title, name, body);
    if (buf->data->project != "") {
      url prj_name= head (name) * as_string (buf->data->project);
      buf->prj= concrete_buffer_insist (prj_name);
    }
  }
  else {
    string old_title= buf->buf->title;
    string old_project= buf->data->project->label;
    tree body= detach_data (doc, buf->data);
    assign (buf->rp, body);
    set_buffer_data (name, buf->data);
    buf->buf->title= propose_title (old_title, name, body);
    if (buf->data->project != "" && buf->data->project != old_project) {
      url prj_name= head (name) * as_string (buf->data->project);
      buf->prj= concrete_buffer_insist (prj_name);
    }
  }
  pretend_buffer_saved (name);
}
```

其中`set_buffer_data (name, buf->data);` 而 函数第一行有`tm_buffer buf= concrete_buffer (name);`

```cpp
tm_buffer
concrete_buffer (url name) {
  int i, n= N(bufs);
  for (i=0; i<n; i++)
    if (bufs[i]->buf->name == name)
      return bufs[i];
  return nil_buffer ();
}
```

下一步问题：buf中的data怎么创建
```cpp
class tm_buffer_rep {
public:
  new_buffer buf;         // file related information
  new_data data;          // data associated to document
  array<tm_view> vws;     // views attached to buffer
  tm_buffer prj;          // buffer which corresponds to the project
  path rp;                // path to the document's root in the_et
  link_repository lns;    // global links
  bool notify;            // notify modifications to scheme

  inline tm_buffer_rep (url name):
    buf (name), data (),
    vws (0), prj (NULL), rp (new_document ()), notify (false) {}

  inline ~tm_buffer_rep () {
    delete_document (rp); }

  void attach_notifier ();
  bool needs_to_be_saved ();
  bool needs_to_be_autosaved ();
};
```

疑似相关代码:
```cpp
url temp= get_current_view_safe ();
  set_current_view (abstract_view (vw));
  if (is_none (tm_init_buffer_file))
    tm_init_buffer_file= "$TEXMACS_PATH/progs/init-buffer.scm";
  if (is_none (my_init_buffer_file))
    my_init_buffer_file= "$TEXMACS_HOME_PATH/progs/my-init-buffer.scm";
  if (exists (tm_init_buffer_file)) exec_file (tm_init_buffer_file);
  if (exists (my_init_buffer_file)) exec_file (my_init_buffer_file);
  set_current_view (temp);
```









## object-get-property
```scheme
(tm-define (object-get-property var)
  (tree->stree (get-env-tree var)))
```

```cpp
{
                scm_name = "get-env-tree",
                cpp_name = "get_env_value",
                ret_type = "tree",
                arg_list = {
                    "string"
                }
            },
```

```cpp
tree
edit_typeset_rep::get_env_value (string var, path p) {
  typeset_exec_until (p);
  tree t= cur[p][var];
  return is_func (t, BACKUP, 2)? t[0]: t;
}

tree
edit_typeset_rep::get_env_value (string var) {
 /* FIXME: tp is wrong (and consequently, crashes TeXmacs)
  *   when we call this routine from inside the code which
  *   is triggered by a button, for example.
  *
  * Test: fire TeXmacs, then open a new Graphics, then click
  *   on the icon for going in spline mode. Then it crashes,
  *   because we call (get-env-tree) from inside the Scheme.
  *   If we call (get-env-tree-at ... (cDr (cursor-path))),
  *   then it works.
  */
  return get_env_value (var, tp);
}
```

