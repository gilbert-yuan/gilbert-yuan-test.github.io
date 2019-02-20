## odoo编辑状态下不可见

odoo 10 中less出识 有些情况下要 让某些字段或者 按钮在编辑状态下不可见. 然而据我所知 目前是没有的
 所以自己摸索着看了点 less 的相关的东西 
 ```css
.o_form_view {
    &.o_form_editable {
      .oe_form_edit_invisivle {
          display: none;
      }
    }
  &.o_form_readonly {
    .oe_form_edit_invisivle {
        display: block;
    }
  }
}
```
