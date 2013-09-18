Title: Filter Generic ListView by User
Date: 2012-09-27
Tags: python, django

I’ve been working quite a bit with Django’s class based generic views the
 past few weeks. Normally, if you just want to take advantage of class based
 views you can edit your urls.py and add the generic views directly there. For
 example, if you just want to list all the objects, you can use the generic
 ListView and add the following snippet to your urls.py urlpatterns:

    :::python
    url(r'^myobjects/$',ListView.as_view(
            queryset = MyObject.objects.order_by('-date'),
            template_name ='myapp/list.html'), name='list'),

A problem arises, however, if you want to return only the objects associated
with the logged in user. Normally in a view you simply filter based on
request.user:

    :::python
    user_objects = MyObject.objects.filter(user=request.user).order_by('-date')

In your urls.py, there is no request object, so it is simply not possible to
filter based on the user in urls.py. Instead, you’ll have to do a little bit
more work.

The generic class based ListView inherits a method get_queryset(self) whose
function is to return a list of objects for the view. By default, this method
looks like:

    :::python
    def get_queryset(self):
            """
            Get the list of items for this view. This must be an iterable, and may
            be a queryset (in which qs-specific behavior will be enabled).
            """
            if self.queryset is not None:
                queryset = self.queryset
                if hasattr(queryset, '_clone'):
                    queryset = queryset._clone()
            elif self.model is not None:
                queryset = self.model._default_manager.all()
            else:
                raise ImproperlyConfigured("'%s' must define 'queryset' or 'model'"
                                       % self.__class__.__name__)

The method first checks to see if we’ve already stored the queryset, and if so
it returns it. Otherwise, the method returns all the objects the database
manager cares to provide (line 11).

Luckily for us, we can also get the user who made the request in
get_queryset(self) because the dispatch method of ListView (which is also
inherited) stores the request made by the user in self.request. As you would
expect, the user object is found at self.request.user.

Now, its simply a matter of creating a new class based view in our views.py that
inherits from ListView. This new class overrides the get_queryset(self) method
to return a filtered queryset:

    :::python
    class MyObjectListView(ListView):
        """Use django generic ListView to list all the the MyObjects for the current
        user."""

        def get_queryset(self):
            """Override get_querset so we can filter on request.user """
            return MyObject.objects.filter(user=self.request.user).order_by('-date')

Hooking this new class based view up in our urls.py is also straightforward. You
simply have to add the following line to your urlpatterns (of course you also
have to import the new class based view as well):

    :::python
    url(r'^myobjectlist/$',
           MyObjectListView.as_view(
                template_name='myapp/list.html'
            ), name='list'),
