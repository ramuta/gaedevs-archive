# Optimizing Indices In Google Cloud Datastore - How And Why?

(Date: 19/11/2018)

You might already know what database indices (or indexes) are from relational databases. Their role is to make **queries faster**. But it's not obligatory to have them - they are an **optional** feature.

But with Google Cloud Datastore it's quite the **opposite**. You can't query the Datastore without indices. They are an **integral part** of the database. 

This is because Datastore is not a relational database, but instead a **NoSQL database**. It's a document-oriented database and each (searchable) field **needs** its **own index** to be successfully queried.

> P.S.: If you use Firestore, you might want to see this article: [Firestore: How to exempt a field from being indexed](https://gaedevs.com/blog/firestore-how-to-exempt-a-field-from-being-indexed)

Enough theory, let's take a more **practical** look into it.

### Example

Let's say that we have the following User model:

    class User(ndb.Model):
        first_name = ndb.StringProperty()
        last_name = ndb.StringProperty()
        email = ndb.StringProperty()
        phone = ndb.StringProperty()
        description = ndb.TextProperty()
        image_url = ndb.StringProperty()
        active = ndb.BooleanProperty(default=False)

As soon as we start **entering** users in the Datastore, the database will **create** indices for almost every attribute in this model. An index for `first_name` will be created. Then another one for `last_name`. And the same for `email`, `phone`, `image_url` and `active` attributes.

### Wait, what about the `description` attribute?

The `description` attribute is of type `TextProperty` and Google Datastore does not create indices for this type of attributes. It kind of makes sense, because the TextProperty content usually takes quite a lot of space. The same goes for `JsonProperty` and some other Datastore types (more about them [here](https://cloud.google.com/appengine/docs/standard/python/ndb/entity-property-reference#types)).

This also means that we **cannot** query a user's description in the Datastore. The following query would **not** work:

	users = User.query(User.description == "smart").fetch()

But you can query, for example, a user's first name:

	users = User.query(User.first_name == "Matt").fetch()

So Google Datastore makes sure that your database does not grow too much too fast by setting some **default restrictions** on specific Datastore types.

> If you'd still want to have the `description` field searchable, you should use Google Search API instead. More about this in one of the future posts.

### Can I restrict specific indices myself?

Yes! You don't want to have every (unrestricted) Datastore field indexed. For example, you might not want to have the `image_url` field indexed, because you know you'll never do a query on it.

So this is how you can **disable/prevent** an index for a certain model attribute:

	image_url = ndb.StringProperty(indexed=False)

This will **save** your Datastore space and keep you longer within the GAE free quota (or at least make you spend less on GAE).

Try to do a query on the `image_url` field now. As you'll see, the query will **not** work, which is what you'd want.

### Best practice

It's a good practice to have the `indexed` property set up for **every** attribute in each of your models, no matter if you set it up as `True` or `False`. 

Let's do this for our User model:

    class User(ndb.Model):
        first_name = ndb.StringProperty(indexed=True)
        last_name = ndb.StringProperty(indexed=True)
        email = ndb.StringProperty(indexed=True)
        phone = ndb.StringProperty(indexed=True)
        description = ndb.TextProperty(indexed=False)
        image_url = ndb.StringProperty(indexed=False)
        active = ndb.BooleanProperty(default=False, indexed=True)

As you can see, I added the `indexed` property **also** to the description attribute. I did this because of **consistency**. It's better to have this property on all your attributes so you can make **sure** that you didn't **forget** to add it somewhere.

This approach also requires that you **think** in advance about which of your model attributes will **need** queries and which will not. 

But also make sure that you don't **overthink** this and spend too much time on it. Even if you do a **mistake** and make some field unindexed, but then find out you shouldn't - don't worry! You can **undo** your mistake pretty easily.

### How to index an unindexed attribute

Let's say you've made the `phone` attribute unindexed (`indexed=False`), but after some time (when you already have a couple hundred users in your database) you figure out you **should** have indexed it.

First of all you have to make the change in your model:

	phone = ndb.StringProperty(indexed=True)

But be aware that this will **not** make Datastore create an index for all of the past entries in the database. It will only index the entries that:

- are **newly added** to the Datastore (so every new User), or
- have been **updated** AFTER you've made this field indexed.

In this situation, you can **either** leave the things as they are and **wait** for the old User entries to be updated - this works for example if your users are regularly updating their profiles.

**Or** if you don't want to wait, you can create a simple **for loop** which updates every User object in the Datastore:

	users = User.query().fetch()
	
	for user in users:
    	user.put()

> **Important:** if you have a **lot** of entries (like more than 5000) in the database, this can take quite some time and it can result in the **request timeout**. In this case, you'd want to use **background workers** where each of the workers would update a limited set of user objects.

This is it about the `indexed` property - I hope it helped you solve some particular problem or gave you an idea on how to optimize your Cloud Datastore! :) 

Stay tuned for our next blog posts by **subscribing** to our **newsletter** below.
