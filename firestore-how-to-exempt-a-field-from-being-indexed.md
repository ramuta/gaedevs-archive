# Firestore: How to exempt a field from being indexed

(Date: 02/01/2019)

Both Cloud Datastore and Firestore automatically index most of the fields in your database models.

### Datastore

For example, if you use Cloud Datastore with Python 2 on the Standard GAE Environment, you'd create your database model like this:

	class User(ndb.Model):
		name = ndb.StringProperty()
		age = ndb.IntegerProperty()
		active = ndb.BooleanProperty()

By default, all three fields would be indexed. But if you don't want some (or any) of them to be indexed, you can set their `indexed` attribute as False:

	class User(ndb.Model):
		name = ndb.StringProperty(indexed=False)
		age = ndb.IntegerProperty(indexed=False)
		active = ndb.BooleanProperty(indexed=False)

You can read more about optimizing indexes in this article: [Optimizing Indices In Google Cloud Datastore - How And Why?](https://gaedevs.com/blog/optimizing-indices-in-google-cloud-datastore-how-and-why)

### Firestore

But what if you use the new GAE Python Standard Environment, which uses Python 3 and Cloud Firestore?

You'd have to disable field indexing via the [Firebase Console](https://console.firebase.google.com).

Open your Firestore database in the Firebase Console. Click on **Indexes** and choose the **Single field** tab.

This page will show up:

<img class="img-fluid" src="https://storage.googleapis.com/smartninja/firestore-field-indexes-1546454886.png">

Then scroll down to the **Exemptions** section:

<img class="img-fluid" src="https://storage.googleapis.com/smartninja/firestore-exemptions-1546455223.png">

Click on **Add exemption** and add the model name and the field you'd like to have exempted from indexing:

<img class="img-fluid" width="400px" src="https://storage.googleapis.com/smartninja/firestore-exemption-2-1546455329.png">

Click Next and disable the indexes you'd like to have exempt:

<img class="img-fluid" width="400px" src="https://storage.googleapis.com/smartninja/firestore-exemption-3-1546455408.png">

Voila! You've successfully exempted a field from having an index created which will save your database storage costs.
