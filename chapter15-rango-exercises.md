#Making Rango Tango! Exercises {#chapter-ex}

So far we have been adding in different pieces of functionality to
Rango. We've been building up the application in this manner to get you
familiar with the Django Framework, and to learn about how to construct
the various parts of a website that you are likely to make in your own
projects. Rango however at the present moment is not very cohesive. In
this chapter, we challenge you to improve the application and its user
experience by bringing together functionality that we've already
implemented alongside some awesome new additions.

To make Rango more coherent and integrated it would be nice to add the
following functionality.

- Track the click throughs of Categories and Pages, i.e.:
	- count the number of times a category is viewed;
	- count the number of times a page is viewed via Rango; and
	- collect likes for categories (see [Django and Ajax Chapter]({#chapter-ajax})).
- Integrate the browsing and searching within categories, i.e.:
	- instead of having a disconnected search page, let users search for pages on each specific category page
	- let users filter the set of categories shown in the side bar (see [Django and Ajax Chapter]({#chapter-ajax})); and only updates the results (see [Django and Ajax Chapter]({#chapter-ajax}))
- Provide services for Registered Users, i.e.:
	- Assuming you have switched the `django-registration-redux`, we need to setup the registration form to collect the additional information (i.e. website, profile picture)
	- let users view their profile;
	- let users edit their profile; and
	- let users see the list of users and their profiles.

I> ###Note
I>
I> We won't be working through all of these tasks right now. Some will be
I> taken care of in the [Django and Ajax Chapter]({#chapter-ajax}), 
I> while others will be left to you to complete as additional exercises.

Before we start to add this additional functionality we will make a todo
list to plan our workflow for each task. Breaking tasks down into
sub-tasks will greatly simplify the implementation so that we are
attacking each one with a clear plan. In this chapter, we will provide
you with the workflow for a number of the above tasks. From what you
have learnt so far, you should be able to fill in the gaps and implement
most of it on your own (except those requiring AJAX). In the following
chapter, we have included hints, tips and code snippets elaborating on how to
implement these features. Of course, if you get really stuck, you can always check out our implementation on GitHub.

##Track Page Click Throughs

Currently, Rango provides a direct link to external pages. This is not
very good if you want to track the number of times each page is clicked
and viewed. To count the number of times a page is viewed via Rango you
will need to perform the following steps.

- Create a new view called `track_url()`, and map it to URL `/rango/goto/` and name it `'name=goto'`.
- The `track_url()` view will examine the HTTP `GET` request parameters and pull out the `page_id`. The HTTP `GET` requests will look something like `/rango/goto/?page_id=1`.
	- In the view, select/get the `page` with `page_id` and then increment the associated `views` field, and `save()` it.
	- Have the view redirect the user to the specified URL using Django's `redirect` method.
	- If no parameters are in the HTTP `GET` request for `page_id`, or the parameters do not return a `Page` object,  redirect the user to Rango's homepage.
- Update the `category.html` so that it uses `/rango/goto/?page_id=XXX`.
	- Remember to use  the `url` *templatetag* instead of using the direct URL i.e. 
	
	{lang="python",linenos=on}
		<a href="{% url 'goto' %}?pageid={{page.id}}"\>


I> ### `GET` Parameters Hint
I>
I> If you're unsure of how to retrieve the `page_id` *querystring* from the
I> HTTP `GET` request, the following code sample should help you.
I>
I> {lang="python",linenos=off}

I> 		if request.method == 'GET':
I> 					if 'page_id' in request.GET:
I> 						page_id = request.GET['page_id']
I>
I> Always check the request method is of type `GET` first, then you can
I> access the dictionary `request.GET` which contains values passed as part
I> of the request. If `page_id` exists within the dictionary, you can pull
I> the required value out with `request.GET['page_id']`.
I>
I> You could also do this without using a *querystring*, but through the
I> URL instead, i.e. `/rango/goto/<page_id>/`. In which case you would
I> need to create a `urlpattern` that pulls out the `page_id`.


##Searching Within a Category Page

Rango aims to provide users with a helpful directory of page links. At
the moment, the search functionality is essentially independent of the
categories. It would be nicer to have search integrated into category
browsing. Let's assume that a user will first browse their category of
interest first. If they can't find the page that they want, they can
then search for it. If they find a page that is suitable, then they can
add it to the category that they are in. Let's focus on the first
problem, of putting search on the category page. To do this, perform the
following steps:

- Remove the generic *Search* link from the menu bar, i.e. we are decommissioning the global search function.
- Take the search form and results template markup from `search.html` and place it into `category.html`.
- Update the search form so that that action refers back to the category page, i.e.:

	{lang="python",linenos=on}
		<form class="form-inline" id="user_form" 
			method="post" action="{% url 'category'  category.slug %}">

- Update the category view to handle a HTTP `POST` request. The view must then include any search results in the context dictionary for the template to render.
- Also, lets make it so that only authenticated users can search. So to restrict access within the `category.html` template use:

	{lang="python",linenos=off}

		{% if user.authenticated %} 
			<!-- Insert search code here -->
		{% endif %}
		

##Create and View Profiles

If you have swapped over to the `django-registration-redux` package,
then you'll have to collect the `UserProfile` data. To do this, instead
of re-directing the user to the Rango index page, you will need re-direct
them to a new form, to collect the user's profile picture and URL details. To add the
UserProfile registration functionality:

- Create a `profile_registration.html` which will display the `UserProfileForm`.
- Create a `register_profile()` view to capture the profile detials
- Map the view to a url, i.e. `rango/add_profile/`.
- In the `MyRegistrationView` update the `get_success_url()` to point to `rango/add_profile/`

Another useful feature to let users inspect and edit their own profile.
Undertake the following steps to add this functionality.

- First, create a template called `profile.html`. In this template, add in the fields associated with the user profile and the user (i.e. username, email, website and picture).
- Create a view called `profile()`. This view will obtain the data required to render the user profile template.
- Map the URL `/rango/profile/` to your new `profile()` view.
- In the base template add a link called *Profile* into the menu bar, preferably on the right-hand side with other user-related links. This should only be available to users who are logged in (i.e. `{% if user.is_authenticated %}`).

To let users browse through user profiles, you can create a users page,
that lists all the users. If you click on a user page, then you can see
their profile (but make sure that a user can only edit their own page).

In the next chapter, we provide a series of hints and tips to help you complete the aforementioned features.
