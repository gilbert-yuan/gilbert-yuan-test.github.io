## 微信读书app上有很多免费的书可以读偶尔看 python数据指南实践 特来拓展下

下面是python中 打印的一个小例子
```python
import sys
from colorama import init
init(strip=not sys.stdout.isatty())
from termcolor import cprint
from pyfiglet import figlet_format
cprint(filget_format('welcome', font='starwars'), 'yellow', 'on_blue', attrs=['bold'])
```

