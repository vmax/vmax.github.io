---
title: Producing long-running reports
date: 2021-01-03
description: without using Celery
---

Sometimes there's a need to produce reports that take more to produce than
any sensible HTTP load balancer would allow (say, Heroku's 30 seconds limitation).

Usually, we'd overcome it by spawning another machine (_worker_).
It would handle the long-running tasks we hand off to it and won't be subject to
HTTP request-response cycle time limitations.

While being a generally preferred approach, I don't necessarily enjoy it.
It comes with its share of complexities, such as having more components to keep track of:
worker, message broker, result broker and storage for the produced reports.

An alternative approach I suggest would be to **stream** the response bits as they are being produced.
Then limitations become much more manageable: you have to send the first byte before 30 seconds,
each next chunk has 55 seconds to be produced ([source](https://devcenter.heroku.com/articles/http-routing#timeouts))

Here is how you would implement it with Django:

```python
import csv
import time
import io
from django.http import StreamingHttpResponse


def streaming_csv_view(request):
    def data():
        for i in range(30):
            yield f"Row {i}\n"  # yield results as they are produced
            time.sleep(2)  # simulate non-instant generation
    response = StreamingHttpResponse(data(), content_type="text/csv")
    response['Content-Disposition'] = 'attachment; filename="report.csv"'
    return response
```

Note that if you're using GUnicorn as your server, you'd need to switch to asynchronous workers (
if you were using `sync` workers before):</br>
`gunicorn django_app.wsgi -k gevent`
