---
layout: post
title: Alert! - Read the docs
comments: true
---

*Public service announcement: read the docs, don't be stupid*

## A Wild Celery Appeared

I recently started work on a Python project that makes excessive use of Celery and SQS queues. These celery workers have been performing well; without any major hitches. That is, they worked fine until I added a `retry()`... Suddenly everything went to hell.

<!--break-->

I first noticed there was a problem when I logged into SQS and one of my test workers was completely out of control; there were thousands of queued messages, and I knew for a fact I had only triggered a single task. Assuming my code was faulty, I went on a wild rabbit hunt trying to figure out what I was doing wrong. Long story short: Read the Docs!

If you are smart (unlike me) and take a couple minutes to browse through the SQS Celery docs you will notice a nice section titled [caveats](http://docs.celeryproject.org/en/latest/getting-started/brokers/sqs.html#caveats). Simply put, there is a problem with countdown/retry tasks where they spawn huge amounts of messages if you are not careful:

> If a task is not acknowledged within the visibility_timeout, the task will be redelivered to another worker and executed.This causes problems with ETA/countdown/retry tasks where the time to execute exceeds the visibility timeout; in fact if that happens it will be executed again, and again in a loop.

And the solution is simple:

> You have to increase the visibility timeout to match the time of the longest ETA you are planning to use.

Oh, and guess what, the docs also tell you what to do. Add the following to your Celery settings:

```python
BROKER_TRANSPORT_OPTIONS = {'visibility_timeout': 43200}
```

*Obviously adjust the visibility timeout as necessary*

So, instead of frantically googling and trying to find the problem in my code I should have gone straight to the Celery docs. Maybe this will help spare someone the frustration of the same stupidity as me. Although its entirely possible that posts like this don't help :-P