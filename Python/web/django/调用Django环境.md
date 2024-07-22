---
title: 调用Django环境
---



```python
import os
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'blog_api.settings.dev')

import django
django.setup()
from home import models
from user import models as user_models
query_set_list = models.RecentMovements.objects.all().order_by("-time").values("type",
                                                                               "model_name",
                                                                               "title",
                                                                          "time", "level","user")
for i in query_set_list:
    print(i)
```

