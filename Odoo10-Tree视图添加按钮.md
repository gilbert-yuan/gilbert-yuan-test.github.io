用JS添加在Tree视图添加按钮。

1.需要在继承Tree视图，当继承RenderButton方法，当创建、导入按钮对应的 html 按钮块加载后，就可以向

   按钮块中添加一个新的按钮

2.按钮的样式尽量参照照创建导入的样式。

3.写对应的按钮的调用方法，也可以直接调用新的action 。例子中用的是调用ACTION的例子。

```js

 function execute_import_assets_action() {

        var self = this;

        self.rpc("/web/action/load", {action_id: "assets.import_assets_action"}).then(function (result) {

            return self.do_action(result);

        });

        return false;

    }

ListView.include({

        render_buttons: function () {

            var self = this;

            var add_button = false;

            if (!self.$buttons) { // Ensures that this is only done once

                add_button = true;

            }

            this._super.apply(self, arguments); // Sets this.$buttons

            if (add_button && self.model == 'maintenance.equipment') {

                self.rpc("web/dataset/call_kw", {

                    'method':"has_group",

                    'model':"res.users",

                    'args':["base.group_no_one"],

                    'kwargs':{},

                }).then(function (response) {

                    if(response){

                    self.$buttons.append(

                        "<button type=\"button\" class=\"btn btn-sm btn-default oe_highlight o_button_import_asssets\">" +

                        " 导入资产</button>");

                    self.$buttons.on('click', '.o_button_import_asssets', execute_import_assets_action.bind(self));

                    }

                })

            }

        }

    });

```

