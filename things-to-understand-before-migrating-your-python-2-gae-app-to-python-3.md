# 11 Things To Understand Before Migrating Your Python 2 GAE App To Python 3

(Date: 28/01/2019)

If you're reading this it might be because you have at least one web app running on the old GAE Standard Environment that supports only Python 2 (and of course Java, PHP and Go).

Meanwhile, the **new generation** of the GAE Standard Environment has arrived and it enables us to build web apps using **Python 3**. But switching from the old GAE environment to the new one is not just about changing a few lines of code in a Python 2 web app to make it compatible with Python 3.

Instead, the new GAE Standard Environment brought some **significant changes** that you should be aware of if you are planning to **migrate** your Python web app from the old to the new Standard Environment. Let's take a look at them.

### 1. requirements.txt

Let's start with **good news**. You won't have to store all your third-party libraries in a special `libs` folder anymore!

Instead, you'll define them in the `requirements.txt` file, just like in any regular Python project. The new GAE Standard environment will **automatically** detect this file and install the libraries.

It was about time, right?

### 2. Datastore vs. Firestore

![](https://storage.googleapis.com/smartninja/datastore-vs-firestore-1548362356.png)

Firestore is basically a **new generation** Datastore. So, should you migrate your data from Datastore to Firestore?

Actually, Google will **do this for you** in the background. The only difference is that if you used Datastore before, you would still need to use Firestore in a **"Datastore mode"** (or maybe better put: in the Datastore way).

You can read more about it here: [Automatic Upgrade to Cloud Firestore](https://cloud.google.com/datastore/docs/upgrade-to-firestore).

So the good news is that **you won't have to do anything** to migrate data from Datastore to Firestore.

But that **doesn't** mean you won't have to do anything when migrating your Python 2 GAE app **code** to Python 3. The main reason is that the `ndb` library is not (yet?) supported on Python 3.

> Update (11 Apr 2019): Google is actively working on a Python 3 ndb library: [https://github.com/googleapis/python-ndb](https://github.com/googleapis/python-ndb).

Instead, you have to use a **new library** called `google-cloud-datastore`. This library helps you **communicate** with your Datastore (or with Firestore in the Datastore Mode). But the new library will require you to write the query code a bit **differently** than you were used to. See some **examples** [here](https://cloud.google.com/datastore/docs/datastore-api-tutorial) and [here](https://googleapis.github.io/google-cloud-python/latest/datastore/index.html).

There are some **gotchas** to consider though. One is that you'll have to put data in the Datastore as **dictionaries** (JSON) and you'll also receive data back as dictionaries (not objects, as you did with the `ndb` library). So don't forget to add the `dict_to_obj()` method to your classes. ;) 

> `index.yaml` for creating composite indexes still works as before.

If you'd like to set up your local environment with the new Firestore or Datastore Emulator, see [this blog post](https://gaedevs.com/blog/how-to-use-the-firestore-emulator-with-a-python-3-flask-app) and the code in [this GitHub repository](https://github.com/smartninja/gae-2nd-gen-examples).

### 3. Running different Python instances

More **good news**. You can run your new Python 3 instance **next** to the Python 2 instance in the **same** Cloud project.

![](https://storage.googleapis.com/smartninja/gae-instances-1548367174.png)

This means you **don't** have to create a new Google Cloud project, but instead you can run Python 3 instance(s) in the **existing** project.

For example, you would be able to create a Python 3 instance with a special version name (like `new` or `beta`). This instance, of course, would have **access** to the **same database** as your Python 2 instance does. And you would be able to give your users a **preview** of the new/beta web app on the separate `appspot.com` subdomain (e.g.: `https://new-dot-projectname.appspot.com`).

Pro-tip: just make sure to use the `--no-promote` flag when you deploy your Python 3 instance:

	gcloud app deploy --version new --no-promote

This means the `new` version will not become the default version.

### 4. webapp2

I personally really liked the webapp2 Python microframework, mainly because of its **simplicity**.

But it's time to say goodbye.

Why? Because Google isn't really doing anything to actively maintain it. They don't even consider it as their official product, they "only host it" on their GitHub account and ["can review any pull request"](https://github.com/GoogleCloudPlatform/webapp2/issues/137) made by the community volunteers.

![](https://storage.googleapis.com/smartninja/google-webapp-stance-1548358135.png)

Even though [webapp2 3.0.0b1](https://github.com/GoogleCloudPlatform/webapp2/releases/tag/3.0.0b1) (beta) does support Python 3, I would advise **against** using it. Instead, you should rewrite your web app into **Flask**.

You'll have to rewrite large parts of your web app **anyway** because many of the old Google Python 2 libraries (such as the `ndb`) won't work on the Python 3 environment. So why not **change** the whole framework along.

### 5. Cron jobs

This is the area where you very likely won't have to change anything.

According to [this documentation](https://cloud.google.com/appengine/docs/standard/python3/scheduling-jobs-with-cron-yaml) you can use `cron.yaml` in a Python 3 GAE app **in the same way** as you have used it in the Python 2 environment.

Google also created a **new service** called [Cloud Scheduler](https://cloud.google.com/scheduler/), which is a more advanced cron/scheduling service. If you want, you can use the Scheduler instead of GAE Cron Jobs, but make sure to check [Scheduler pricing](https://cloud.google.com/scheduler/pricing) before making any decision.

### 6. Task Queue API (background tasks)

Here's another GAE Standard library that **cannot** be used in the Python 3 environment: App Engine task queues.

![](https://storage.googleapis.com/smartninja/google-cloud-tasks-1548360144.png)

Instead, you'll have to use [Cloud Tasks](https://cloud.google.com/tasks/) via the `google-cloud-tasks` Python 3 library.

### 7. Memcache

Memcache will not be available for the Python 3 GAE Standard Environment (2nd Gen). 

Instead, you'd have to use **Redis** (see [Cloud Memorystore](https://cloud.google.com/memorystore/)).

How to connect your GAE app with Cloud Memorystore? VPC Connectors.

Using Memorystore will be more complex than using memcache on the old GAE, unfortunately. It will also be more expensive, because you'll have to run a Redis instance. 

The cheapest Cloud Memorystore Redis instance costs $0.049/hour, which means you'll be charged at least $36/month for each GAE app that uses the Memorystore (because you can't share Memorystore across many projects).

> Read [this article](https://gaedevs.com/blog/google-please-offer-a-true-serverless-in-memory-cache) for more info (and share it).

### 8. Mail API

GAE allowed you to **send emails** via the [Mail API](https://cloud.google.com/appengine/docs/standard/python/mail/). This worked for **smaller** web apps, but for anything **serious**, you had to turn to a **professional** third-party solution such as Sendgrid or Mailgun.

So, in case you did use the Mail API - guess what? Not supported on Python 3. Time to say goodbye. :)

### 9. Users API

This was one of the **nicest** features of the old GAE Standard Environment. Enabling Google login with just a few lines of code via the [Users API](https://cloud.google.com/appengine/docs/standard/python/users/).

The good old days are over and the Users API is **not available** on Python 3. Instead, you'll have to use [Firebase Authentication](https://firebase.google.com/docs/auth/) or, even better, the [Requests OAuthlib](https://github.com/requests/requests-oauthlib).

### 10. dev_appserver.py

You can run your new Python 3 web app with the "old" `dev_appserver.py` on localhost. Which is kind of funny, because gcloud and `dev_appserver.py` still require Python 2 to run. So you still need both Python versions installed.

Also be aware that the Datastore emulator is moving out of `dev_appserver.py` and is already available as an **extra feature** in gcloud (although still in beta for now). See this [link](https://cloud.google.com/datastore/docs/tools/datastore-emulator) for more info.

### 11. No plans for deprecating the old Standard Environment

Finally, just to make you a little bit **less stressed** about everything: Google has **no official plans to deprecate** the old Standard Environment.

![](https://storage.googleapis.com/smartninja/google-deprecating-gae-answer-1548356137.png)

Even though Python 2 has the **"end of life"** in **2020**, you will still be able to use it on Google App Engine. According to some Google insider, too much of the Google's infrastructure still relies on Python 2 so Google will take care of patching Python 2 if needed.

To conclude, some of the **services** a normal project would **need** on GAE are **not available yet** (Memcache) and others are still in **beta** (Scheduler, Tasks, etc.). So there's no hurry with the migration. But you **should** start planning it.

----

Let me know in the **comments** what else one should be aware of when migrating, and I will update the article if needed. Thanks!

Useful links:

- Some weird gotchas that the Pinterest team stumbled upon when they were [migrating from Python 2 to 3](https://youtu.be/e1vqfBEAkNA?t=1072)
- Python 3 examples (Flask) for the 2nd Gen GAE Environment: [GitHub repo](https://github.com/smartninja/gae-2nd-gen-examples)

*(Last update: 4 Oct 2019)*
