# Google, please offer a true serverless in-memory cache

(Date: 05/11/2019)

I have been a loyal and happy Google App Engine (GAE) user for a very long time.

### 1st Gen GAE

I really loved the **1st generation** GAE, it offered all the services you needed in a **true serverless** fashion. Background tasks, cron jobs, a database (Datastore), and memory caching (memcache).

The only issue was **vendor lock-in**, but considering all the benefits I could live with that drawback.

### 2nd Gen GAE

Then came the 2nd generation GAE with a mission to **free** us all from the vendor lock-in "problem".

Migrating from the 1st generation GAE to the 2nd generation requires you to use a bit **different** set of tools and libraries in your web app.

For example, you'd have to use the [new ndb library](https://github.com/googleapis/python-ndb) for communicating with Datastore. It requires some code changes, but nothing terrible.

Or, for example, you'd have to use the new Cloud Tasks library instead of the old GAE tasks. Here's how the code looked before/after:

	# 1st gen GAE Python 2 background tasks
	from google.appengine.api import taskqueue
	
	taskqueue.add(
        queue_name="email-queue",
        url="/tasks/send-email",
        params=task_params
    )
    
    # 2nd gen GAE Python 3 Cloud Tasks library
    from google.cloud import tasks_v2
    
    client = tasks_v2.CloudTasksClient()
    parent = client.queue_path(project_id, region_name, queue_name)
    
    task = {
        'app_engine_http_request': {
            'http_method': 'POST',
            'relative_uri': "/tasks/send-email",
            'body': json.dumps(task_params).encode(),
        }
    }

    client.create_task(parent, task)

A bit more code is required in the 2nd Gen GAE apps compared to 1st Gen, but nothing terrible.

All of these tools (datastore, tasks, cron jobs) are available for the 2nd gen GAE in the same **serverless** (and **pay-as-you-go**) manner as they were for the 1st gen GAE.

**Except** for one: in-memory cache.

### Memorystore is not serverless

The 1st gen GAE has a **very convenient** in-memory caching implementation called **memcache** (based on Memcached).

In 2nd gen GAE, you cannot use memcache anymore, but there's another option that Google now offers: **Memorystore**.

Memorystore is a managed Redis service, which would be great except for the fact that it is **NOT SERVERLESS**.

Memorystore requires you to **spin up** a Redis **server instance** (in a similar way that you would start a Compute Engine instance), run it 24/7 and pay for it on a [per-hour basis](https://cloud.google.com/memorystore/pricing).

This is not serverless and it's **not a pay-as-you-go model** in a way that, for example, Cloud Tasks, Firestore/Datastore, and Cloud Scheduler offer (where you pay per operation).

Using **memcache** on the 1st gen GAE was very easy:

	from google.appengine.api import memcache
	
	memcache.add(key="some-key", value="some value")
	value = memcache.get(key="some-key")
	memcache.delete(key="some-key")

On the other hand, if you want to set up **Memorystore**, you have to spin up a Redis **server instance** and then configure a [VPC Access Connector](https://cloud.google.com/vpc/docs/configure-serverless-vpc-access) to connect your GAE app with the Redis server instance. For anyone that wants to have a **serverless** web app setup, this is an **overkill**.

### Google, please create a true serverless in-memory cache service

Caching is **the only missing piece** to complete your serverless offer, Google.

Instead of having us to spin up a fully-fledged Redis instance that runs 24/7, you should offer a **simple serverless pay-as-you-go Memcached service**.

If you do this, I can forgive that you don't offer an **in-built Google authentication** on GAE anymore:

	from google.appengine.api import users
	
	user = users.get_current_user()

I can forget that you ever had a GAE SDK with a nice GUI tool (**GAE Launcher**).

![](https://storage.googleapis.com/smartninja/gae-launcher-1634822733.png)

And I can forgive that you require me to install Python 2 and Java JDK even if the only thing I want to do is to run my Python 3 GAE app locally (Cloud SDK requires Python 2 and Firestore/Datastore Emulator requires Java JDK).

Please, Google, bring the (serverless) caching back. ;)
