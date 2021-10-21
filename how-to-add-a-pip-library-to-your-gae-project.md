# How to add a PIP library to your GAE project

(Date: 21/11/2018)

Usually, when you'd want to have a **PIP library** accessible to your web app project, you'd **install** the library on your **server** machine and every project on that server could access it.

**GAE** apps work a bit **differently**. Since a GAE app runs in a **sandboxed** environment, it **cannot** access any resources from the server it's running on.

That's why you need to include PIP libraries **directly** into your web app project.

You could do this **manually** by **copy/pasting** the library folder into your project. But instead, it's **better** to **automate** this process. Let us show you how.

### Step 1: Create a `libs` folder

Within the root of your GAE web app create a new folder called `libs` (short for "libraries").

<img class="img-fluid" src="https://storage.googleapis.com/smartninja/libs-folder-1542839445.png">

### Step 2: Add `libs` into `.gitignore`

If you use **GIT** (and you should), make sure to add the `libs` folder into the `.gitignore` file, because it shouldn't eat up your GIT space.

### Step 3: Create a `requirements.txt` file

If you've ever **automated** your PIP workflow, you're already familiar with this step. The `requirements.txt` file will hold a **list** of your PIP libraries.

Let's say you'd like to install two PIP libraries, **markdown2** and **bleach**. The most simple way is to include their **names** into the `requirements.txt` file, one in each line:

	markdown2
	bleach

But if you'd like to have more control over which **version** exactly of the libraries you want to have, you can write version numbers next to the names:

	markdown2==2.3.5
	bleach==2.1.2

### Step 4: Run the pip install command

Now it's time to **install** your PIP libraries. As we said, the libraries must be **included** in the project folder. Let's see how the **command** looks like:

	pip install -t libs -r requirements.txt

After you run this command, take a look into your `libs` folder. You should see the **newly** installed libraries in it.

The process is **not over** yet. There's one last thing to do before you can start using the new libraries in your project.

### Step 5: `appengine_config.py`

You need to **create** a file called `appengine_config.py` in the **root** of your project. In that file include the following code:

	from google.appengine.ext import vendor

	vendor.add('libs')

This will allow your GAE project to **access** the third-party libraries from PIP (the ones that you have installed in your project).

### Step 6: Start using the libs

Now you can start **using** the newly installed libraries in your project. **Congrats!**

<img class="img-fluid" width="500px" src="https://storage.googleapis.com/smartninja/thumbs-up-1542839860.png">

### Bonus step 7: Continuous integration (CI)

If you're using a **continuous integration** for your web app (and again, **you should**), make sure to include the pip install command (from step 4) in it.

### Bonus step 8: Upgrading the libraries

You can **upgrade** your PIP libs with the following command:

	pip install -t libs -r requirements.txt --upgrade

---

That's it! Hope this article **helped** you in your web app development and **saved** you some **time** and **nerves**. Make sure to **subscribe** to our **newsletter** below so you don't miss our future useful GAE-related articles.
