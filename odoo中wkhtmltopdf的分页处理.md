## odoo中wkhtmltopdf的分页处理


原文  http://best.gooderp.org/d/44

Question:pdf报表分页问题一直是困扰一些新手的难题,不知道从何入手,其实很简单,只是没找到关键点.

1.pdf报表生成的都通过 wkhtmltopdf第三方插件生成的.

百度下wkhtmltopdf 分页相信结果很多各种测试案例都有.

2.通过对比发现其实wkhtmltopdf 插件就是利用html的css样式进行分页的(其实也就是先生成html在利用html转换成pdf)
到现在也许就更简单了, 百度html分页 答案更多..

3. 现在我就罗列出所有的关于分页的设置.及参数.(默认值应该是auto)

page-break-after  : auto | always | avoid | left | right
page-break-before : auto | always | avoid | left | right
page-break-inside : auto | avoid
4.具体使用.(下面就用相同的数据对比下两个分页的设置的不同|最常用的)

这个div的内容不能在同一页中强制在这个div前分页符.
 
