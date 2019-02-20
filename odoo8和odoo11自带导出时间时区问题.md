## odoo8 odoo11 自带导出，时间字段存在时区问题

1. ODOO8 对应的解决方案
把下面的代码放到odoo可加载的 py文件中就可以了
```python
from openerp.fields import Datetime, Field
ISODATEFORMAT = '%Y-%m-%d'

Newdatetime = Datetime.convert_to_export
def convert_to_export(self, value, env):
 timezone = pytz.timezone(env.context.get('tz'))
    return_val = Newdatetime(self, value, env)
   if isinstance(return_val, datetime.datetime) and return_val:
         return self.to_string(return_val.replace(tzinfo=pytz.timezone('UTC')).astimezone(timezone))
   elif return_val:
          return self.to_string(self.from_string(return_val).replace(tzinfo=pytz.timezone('UTC')).astimezone(timezone))
        return return_val
Datetime.convert_to_export = convert_to_export
```
2.把下面代码放到ODOO11 可加载的py文件中即可
```python
import datetime, pytz
from odoo.fields import Datetime

Newdatetime = Datetime.convert_to_export
def convert_to_export(self, value, record):
  timezone = pytz.timezone(record._context.get('tz'))
  return_val = Newdatetime(self, value, record)
  if isinstance(return_val, datetime.datetime) and return_val:
      return self.to_string(return_val.replace(tzinfo=pytz.timezone('UTC')).astimezone(timezone))
  elif return_val:
       return self.to_string(self.from_string(return_val).replace(tzinfo=pytz.timezone('UTC')).astimezone(timezone))
 return return_val
Datetime.convert_to_export = convert_to_export


```


