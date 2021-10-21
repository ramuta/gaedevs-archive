# Implement Google Cloud Storage in your GAE Python web app

(Date: 27/11/2018)

There are **two ways** of uploading files to Google Cloud from your GAE app:

- Blobstore
- Cloud storage

**Blobstore** is probably the **easiest** to implement because it's part of the standard GAE libraries. But it might **not** be the best choice. It doesn't give you nice URLs for your files and it's harder to manage the uploaded files.

Using **Cloud Storage** is a much **better** choice. At first, it seems harder to implement, but when you’ll go through this tutorial, you’ll be able to implement it very **quickly**.

So let's dive into it!

> This is an example for a Python 2.7 Standard GAE Runtime

### GoogleAppEngineCloudStorageClient

We'll build a simple **image uploader** app. First we need to install a **library** with a hilariously **long name**, called `GoogleAppEngineCloudStorageClient`. Add this into your `requirements.txt` file:

	GoogleAppEngineCloudStorageClient==1.9.22.1

Then run the pip install command which will install the library into your `libs` folder.

> If you don't know how to install PIP libraries for your GAE projects (it's different than usual), **stop at this point** and take a look at this tutorial: [How to add a PIP library to your GAE project](https://gaedevs.com/blog/how-to-add-a-pip-library-to-your-gae-project).

### HTML form

Let's create a simple HTML form because there's one **gotcha** that you need to be aware of:

	<form action="/upload" method="post" enctype="multipart/form-data">
        <input type="file" name="uploaded-file">

        <button type="submit">Upload</button>
    </form>

As you can see, your form needs to have the `enctype="multipart/form-data"` included in order to process files.

### UploadedFile model

Let's create a simple model that will **save** our uploaded files (or more precisely: URLs to files) into the Datastore.

Create a file called `uploaded_file.py` (inside the `models` folder) and enter this code in:

	from google.appengine.ext import ndb


	class UploadedFile(ndb.Model):
	    url = ndb.StringProperty(indexed=False)
	    # you can also add other fields, for example: uploader name, date created, tags etc.


### GCS bucket

The uploaded files will be saved in a Google Cloud Storage (GCS) bucket. So go head to the Google Cloud Console and create a bucket for your files.

The easiest way to find anything on the Cloud Console is via search:

<img class="img-fluid" width="300px" src="https://storage.googleapis.com/smartninja/search-create-bucket-1543333669.png">

Now fill the "Create a bucket" form. Make sure you choose a unique name. Don't use spaces between words, use a dash or a hyphen (`-`).

<img class="img-fluid" width="500px" src="https://storage.googleapis.com/smartninja/create-bucket-form-1543333585.png">

Click on create and your bucket is created. The next step is very important. You have to make the files in your bucket publicly accessible, if you'd like anyone to see, for example, the uploaded images (if you don't want to do this, skip this step).

First click on the Permissions tab and the Add members button:

<img class="img-fluid" width="500px" src="https://storage.googleapis.com/smartninja/bucket-permissions-1543335155.png">

And then add allUsers as a member who can view files:

<img class="img-fluid" width="500px" src="https://storage.googleapis.com/smartninja/bucket-add-members-1543335132.png">

### Upload helper

Now that we have our bucket set up, let's create a Python file which will store the code that will help us with the file uploading. Usually you'd store this in the `utils` folder, but for our simple project we'll have it in the root of our project:

	import os
	import cloudstorage
	import time
	from models.uploaded_file import UploadedFile
	
	
	GCS_BUCKET = '/your-bucket-name'  # enter the name of your bucket
	
	
	def upload_file_helper(uploaded_file):
	    file_content = uploaded_file.file.read()  # read the file
	
	    # edit the file name
	    file_name = str(uploaded_file.filename).replace(" ", "-").replace(".", "-")  # remove spaces
	    file_name += str(int(time.time()))  # add a timestamp at the end of the file
	
	    # file type
	    file_type = uploaded_file.type
	    file_name += "." + file_type.split("/")[1]  # if type is image/png, add .png at the end
	
	    # upload the file to Google Cloud Storage
	    gcs_file = cloudstorage.open(
	        GCS_BUCKET + '/' + file_name,
	        'w',
	        content_type=file_type,
	        retry_params=cloudstorage.RetryParams(backoff_factor=1.1)
	    )
	
	    gcs_file.write(file_content)
	    gcs_file.close()
	
	    # get the URL
	    url = 'http://localhost:8080/_ah/gcs' if is_local() else 'https://storage.googleapis.com'
	    url += GCS_BUCKET + '/' + file_name
	    
	    # store the URL in the Datastore
    	saved_file = UploadedFile(url=url)
    	saved_file.put()
	
	    return url
	
	
	def is_local():
	    """ Check if you are currently running on localhost or on GAE. """
	    if os.environ.get('SERVER_NAME', '').startswith('localhost'):
	        return True
	    elif 'development' in os.environ.get('SERVER_SOFTWARE', '').lower():
	        return True
	    else:
	        return False

Go through this code. It's pretty self-explanatory and has a lot of comments.

### Upload handler

Now we can create a handler that will receive the file from your HTML form and upload it to GCS using the Upload Helper.

> A "handler" is a synonym for a "controller".

Go to main.py (or wherever you keep your handlers) and add this code:

	class UploadFileHandler(BaseHandler):
	    def post(self):
	        if self.request.get('uploaded-file'):
	            uploaded_file = self.request.POST.get('uploaded-file')
	
	            url = upload_file_helper(uploaded_file=uploaded_file)
	
	            return self.render_template("upload_success.html", params={"url": url})
	        else:
	            return self.error(400)

Also, make sure to add the correct URL route:

	webapp2.Route('/upload', UploadFileHandler),

And you can add this code in the `upload_success.html` file:

	<h3>Success!</h3>
	
	<p>Your file has been successfully uploaded.</p>
	
	<p>See it here: <a href="{{url}}">{{url}}</a></p>
	
	<img width="300px" src="{{url}}">

### Let's try it out!

> You can find the whole example here: [https://github.com/ramuta/gaedevs-examples/tree/master/file-uploader](https://github.com/ramuta/gaedevs-examples/tree/master/file-uploader)

With everything in place, it's time to try out our new file uploader. Play around with it and maybe add some additional features specific to your project.

Let me know in the comments how it goes!
