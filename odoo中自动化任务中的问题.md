
## 最近在写自动化任务的时候遇见了BUG

自动化执行的任务的中调用了 controller 中的一个方法， 众所周知 contrloler中没有实例化的self odoo 提供了 odoo.http.request 来充当self的作用

例如
```python
from odoo.http import request
request.env['hr.holiday'].search([])
```
其实这样也很方便，`主要是自动触发的情况下会报出这个问题`     但是如果在自动化任务中这么调用会报错， （手动点击按钮运行的时候不会报错）
` RuntimeError： object unbound  `
以后使用起来也要注意；
```python
其实当从整体考虑的时候也很好去分析，request是在web端进行通讯的时候才会去实例化的对象，但是你自动任务（是任务管理器后台发起的动作）中调用request 就肯定是空对象了
```

功能性方法要独立到起来，不能放在contrlloer里面这样会给调用带来不必要的麻烦
