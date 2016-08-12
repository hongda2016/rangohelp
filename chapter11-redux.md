#User Authentication with `Django-Registration-Redux` {#chapter-redux}

In a previous chapter, we added in login and registration functionality by manually coding up the urls, views and templates. However, such functionality is common to many web application so developers have crated numerous add-on applications that can be included in your Django project to reduce the amount of code required to provide login, registration, one-step and two-step authentication, password change, password recovery, etc. In this chapter, we are going to use the package `django-registration-redux` to
provide these facilities. 

This will mean we will need re-factor our
code to remove the login and registration functionality we previously created, and then setup and configure our project to include the `django-registration-redux` application.
This chapter also will provide  you with some experience of using external
applications and show you how easily they can be plugged into your Django project.

##Setting up Django Registration Redux

To start we need to first install `django-registration-redux` into your environment using `pip`.

{lang="text",linenos=off}

	pip install django-registration-redux

Now that it is installed, we need to tell Django that we will be using
this application. Open up the `settings.py` file, and update the
`INSTALLED_APPS` list:

{lang="python",linenos=off}

	INSTALLED_APPS = [
		'django.contrib.admin',
		'django.contrib.auth',
		'django.contrib.contenttypes',
		'django.contrib.sessions',
		'django.contrib.messages',
		'django.contrib.staticfiles',
		'rango',
		'registration'  # add in the registration package
		]
	

While you are in the `settings.py` file you can also add:

{lang="python",linenos=off}

	# If True, users can register
	REGISTRATION_OPEN = True
	# One-week activation window; you may, of course, use a different value.
	ACCOUNT_ACTIVATION_DAYS = 7
	# If True, the user will be automatically logged in.
	REGISTRATION_AUTO_LOGIN = True  
	# The page you want users to arrive at after they successful log in
	LOGIN_REDIRECT_URL = '/rango/' 
	# The page users are directed to if they are not logged in,
	# and are trying to access pages requiring authentication 
	LOGIN_URL = '/accounts/login/' 

These settings should be pretty self explanatory. Now, in
`tango_with_django_project/urls.py` you can update the `urlpatterns` so
it includes a reference to the registration package:

{lang="python",linenos=off}

	url(r'^accounts/', include('registration.backends.simple.urls')),

The `django-registration-redux` package provides a number of different
registration backends, depending on your needs. For example you may want
a two-step process, where user is sent a confirmation email, and a
verification link. Here we will be using the simple one-step
registration process, where a user sets up their account by entering in
a username, email, and password, and is automatically logged in.

##Functionality and URL mapping
The Django Registration Redux package provides the machinery for
numerous functions. In the `registration.backend.simple.urls`, it
provides:

-   registration -\> `/accounts/register/`
-   registration complete -\> `/accounts/register/complete/`
-   login -\> `/accounts/login/`
-   logout -\> `/accounts/logout/`
-   password change -\> `/password/change/`
-   password reset -\> `/password/reset/`

While in the `registration.backends.default.urls` it also provides the
functions for activating the account in a two stage process:

-   activation complete (used in the two-step registration) -\>
    `activate/complete/`
-   activate (used if the account action fails) -\>
    `activate/<activation_key>/`
-   activation email (notifies the user an activation email has been
    sent out)

    > -   activation email body (a text file, that contains the
    >     activation email text)
    > -   activation email subject (a text file, that contains the
    >     subject line of the activation email)

Now the catch. While Django Registration Redux provides all this
functionality, it does not provide the templates. So we need to create
the templates associated with each view.

##Setting up the Templates
In the  [Django Registration Redux Quick Start Guide](https://django-registration-redux.readthedocs.org/en/latest/quickstart.html),
it provides an overview of what templates are required, but it is not
immediately clear what goes within each template. Rather than try and work it out from the code, we can take a look at a set of [templates written by Anders
Hofstee](https://github.com/macdhuibh/django-registration-templates) to quickly get the gist of what we need to code up. 

First, create a new directory in the `templates` directory, called
`registration`. This is where we will house all the pages associated
with the Django Registration Redux application, as it will look in this
directory for the templates it requires.

### Login Template

In `templates/registration` create the file, `login.html` with the
following code:

{lang="html",linenos=off}



	{% extends "rango/base.html" %}
	{% block body_block %}
		<h1>Login</h1>
		<form method="post" action=".">
			{% csrf_token %}
			{{ form.as_p }}
			<input type="submit" value="Log in" />
			<input type="hidden" name="next" value="{{ next }}" />
		</form>
		<p>
			Not  a member? 
			<a href="{% url 'registration_register' %}">Register</a>
		</p>
	{% endblock %}

Notice that whenever a URL is referenced, the `url` template tag is used
to reference it. If you visit, `http://127.0.0.1:8000/accounts/` then
you will see the list of URL mappings, and the names associated with
each URL (assuming that `DEBUG=True` in `settings.py`).

### Registration Template

In `templates/registration` create the file, `registration_form.html`
with the following code:

{lang="html",linenos=off}



	{% extends "rango/base.html" %}
	{% block body_block %}
		<h1>Register Here</h1>
		<form method="post" action=".">
			{% csrf_token %}
			{{ form.as_p }}
			<input type="submit" value="Submit" />
		</form>
	{% endblock %}


### Registration Complete Template

In `templates/registration` create the file,
`registration_complete.html` with the following code:

{lang="html",linenos=off}



	{% extends "rango/base.html" %}
	{% block body_block %}
		<h1>Registration Complete</h1>
		<p>You are now registered</p>
	{% endblock %}


### Logout Template

In `templates/registration` create the file, `logout.html` with the
following code:

{lang="html",linenos=off}



	{% extends "rango/base.html" %}
	{% block body_block %}
		<h1>Logged Out</h1>
		<p>You are now logged out.</p>
	{% endblock %}
		

### Try out the Registration Process

Run the server and visit: <http://127.0.0.1:8000/accounts/register/>

Note how the registration form contains two fields for password - so
that it can be checked. Try registering, but enter different passwords.

While this works, not everything is hooked up.

###Refactoring your project
Now you will need to update the `base.html` so that the new registration URL/views are used:

- Update register to point to `<a href="{% url 'registration_register' %}">`
- Update login to point to `<a href="{% url 'auth_login' %}">`, and
- Update logout to point to `<a href="{% url 'auth_logout' %}?next=/rango/">`
- In `settings.py`, update `LOGIN_URL` to be `'/accounts/login/'`.

Notice that for the logout, we have included a `?next=/rango/`. This is
so when the user logs out, it will redirect them to the index page of
rango. If we exclude it, then they will be directed to the log out page
(but that would not be very nice).

Next de-commission the `register`, `login`, `logout` functionality from
the `rango` application, i.e. remove the urls, views, and templates (or
comment them out).

### Modifying the Registration Flow

At the moment, when users register, it takes them to the registration
complete page. This feels a bit clunky, so instead, we can take them to
the main index page. This can be done by overriding the
`RegistrationView` provided by `registration.backends.simple.views`. To
do this update the `tango_with_django_project/urls.py` by importing
`RegistrationView`, add in the following registration class.

{lang="python",linenos=off}

	from registration.backends.simple.views import RegistrationView

	# Create a new class that redirects the user to the index page, 
	#if successful at logging
	class MyRegistrationView(RegistrationView):
		def get_success_url(self,request, user):
			return '/rango/'

Then update the `urlpatterns` by adding the following line before the pattern for `accounts`.

{lang="python",linenos=off}

	url(r'^accounts/register/$', 
		MyRegistrationView.as_view(), 
			name='registration_register'),
	

By doing so, when `accounts/register/` is visited this pattern is matched first, and then re-directed to our customised registration view.
	

X> ###Exercise and Hints
X>
X> - Provide users with password change functionality. 
X> - Hint: see [Anders Hofstee's Templates](https://github.com/macdhuibh/django-registration-templates/tree/master/registration) to get started.
X> - Hint: the URL to change passwords is `accounts/password/change/` and the URL to denote the password has been changed is: `accounts/password/change/done/`

