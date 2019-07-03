在大数据量的情况下是不能够使用 xlwt 的因为这个库只能生成旧版xls 格式的excel 所以要无所顾忌的下载excel 就要用能生成新版excel 格式的库
也就是 xlsxwriter 
下面是我简单的处理 生成excel 的主要过程

```python

 def style_data_xlsx(self, xls_workbook):
        date_style = xls_workbook.add_format({'num_format': 'dd/mm/yy', 'bold': True, 'font_name': 'SimSun', 'pattern':1,
                                            'align': 'right', 'bg_color': 'yellow', 'border': 1})
        datetime_style = xls_workbook.add_format({'num_format': 'dd/mm/yy hh:mm:ss', 'bold': False, 'font_name': 'SimSun',
                                                   'pattern':1,'align': 'right', 'bg_color': 'yellow', 'border': 1})
        float_style = xls_workbook.add_format({'bold': False, 'font_name': 'SimSun', 'pattern': 1,
                                               'align': 'right', 'bg_color': 'yellow', 'border': 1})
        colour_style = xls_workbook.add_format({'bold': True, 'font_name': 'SimSun', 'align': 'centre','pattern': 1,
                                                'bg_color': 'orange', 'border': 1})
        base_style = xls_workbook.add_format({'bold': False, 'font_name': 'SimSun', 'align': 'left', 'pattern': 1,
                                              'bg_color': 'yellow', 'border': 1})
        colour_style.set_bg_color('#FF9900') # 设置xlsxwriter 库中没有预存的颜色  预存的颜色如下
        """ 
            'black': '#000000',
            'blue': '#0000FF',
            'brown': '#800000',
            'cyan': '#00FFFF',
            'gray': '#808080',
            'green': '#008000',
            'lime': '#00FF00',
            'magenta': '#FF00FF',
            'navy': '#000080',
            'orange': '#FF6600',
            'pink': '#FF00FF',
            'purple': '#800080',
            'red': '#FF0000',
            'silver': '#C0C0C0',
            'white': '#FFFFFF',
            'yellow': '#FFFF00',
        """
        
        return colour_style, base_style, float_style, date_style, datetime_style

    def from_data_excel_xlsx(self, fields, rows_file_address):
        rows, file_address = rows_file_address
        xls = StringIO.StringIO()

        xls_workbook = xlsxwriter.Workbook(xls)
        worksheet = xls_workbook.add_worksheet(u'Sheet 1')
        colour_style, base_style, float_style, date_style, datetime_style = self.style_data_xlsx(xls_workbook)
        worksheet.set_row(0, 15)
        columnwidth = {}
        for row_index, row in enumerate(rows):
            for cell_index, cell_value in enumerate(row):
                str_len = 6
                if cell_value:
                    str_len = len(str(cell_value)) # (len(str(cell_value).encode('utf-8')) - len(str(cell_value))) / 2 +
                if cell_index in columnwidth:
                    if str_len > columnwidth.get(cell_index):
                        columnwidth.update(
                            {cell_index: str_len})
                else:
                    columnwidth.update(
                        {cell_index: str_len})
                if row_index == 0:
                    cell_style = colour_style
                elif row_index != len(rows) - 1:
                    cell_style = base_style
                    if isinstance(cell_value, basestring):
                        cell_value = re.sub("\r", " ", cell_value)
                    elif isinstance(cell_value, datetime.datetime):
                        cell_style = datetime_style
                    elif isinstance(cell_value, datetime.date):
                        cell_style = date_style
                    elif isinstance(cell_value, float) or isinstance(cell_value, int):
                        cell_style = float_style
                else:
                    cell_style = xls_workbook.add_format({'bold': False, 'font_name': 'SimSun',
                                                          'align': 'left','bg_color': 'yellow', 'border': 1, 'pattern':1})
                worksheet.write(row_index, cell_index, cell_value, cell_style)
                
        for column, widthvalue in columnwidth.items():
            """参考 下面链接关于自动列宽（探讨）的代码
             http://stackoverflow.com/questions/6929115/python-xlwt-accessing-existing-cell-content-auto-adjust-column-width
             仅仅作为参考，xlsxwriter 比较复杂 链接上讨论的是xlwt这个库的方法 """
            if (widthvalue + 3) * 367 >= 65536:
                widthvalue = 50
            worksheet.set_column(column, column, widthvalue + 4)  # 设置合适的列宽-并不完美，因为这个方法是参考 xlwt下的写法仅仅稍微好一点。
        # frozen headings instead of split panes
        # in general, freeze after last heading row
        worksheet.freeze_panes(1, 0, pane_type=1)   # 固定表头 第一行
        #worksheet.split_panes(15, 0)  # First row.
        # if user does unfreeze, don't leave a split there
        xls_workbook.close()
        xls.seek(0)
        data = xls.getvalue()
        return data

```
