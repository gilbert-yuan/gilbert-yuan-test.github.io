1.odoo 中tree视图的翻页问题，删除一条记录，odoo会自动跳到前一页，打开一条记录再点击面包屑返回，它会跳到第一页，等等。
```js
ListView.Groups.include({
        render_dataset: function () {
            this.view.current_min = this.view.pager && this.view.pager.state ? this.view.pager.state.current_min: 1;
            console.log(this.view.current_min)
            var return_val = this._super.apply(this, arguments);
            return return_val
        }
    });
    ListView.include({
        do_delete: function (ids) {
            if (!(ids.length && confirm(_t("Do you really want to remove these records?")))) {
                return;
            }
            var self = this;
            return $.when(this.dataset.unlink(ids)).done(function () {
                _(ids).each(function (id) {
                    self.records.remove(self.records.get(id));
                });
                // Hide the table if there is no more record in the dataset
                if (self.display_nocontent_helper()) {
                    self.no_result();
                } else {
                    if (self.records.length && self.current_min === 1) {
                        // Reload the list view if we delete all the records of the first page
                        self.reload();
                    } else if (self.records.length && self.dataset.size() < self.current_min) {
                        // Load previous page if the current one is empty
                        self.pager.previous();
                    }
                    // Reload the list view if we are not on the last page
                    if (self.current_min + self._limit - 1 < self.dataset.size()) {
                        self.reload();
                    }
                }
                self.update_pager(self.dataset);
                self.compute_aggregates();
            });
        },
    })
```
2.odoo的所有的many2one搜索更多默认只有160条，当用户在这个搜索更多这个页面添加了自定义（收藏的搜索）时如果满足这个搜索条件的记录不在160条里面会导致搜索结果
为空。
```js
 var form_common = require('web.form_common');
    form_common.SelectCreateDialog.include({
        setup: function () {
            this.initial_ids = undefined;
            return this._super.apply(this, arguments);
        }
    });
```




