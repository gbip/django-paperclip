Paperclip
=========

Add attachments to Django models, used in `MapEntity <https://github.com/makinacorpus/django-mapentity>`_.

=======
INSTALL
=======

Installing from pypi (using pip)

::

    pip install paperclip


Installing from github

::

    pip install -e git://github.com/makinacorpus/django-paperclip.git#egg=django-paperclip

============
REQUIREMENTS
============

Django-paperclip is supported only in Python3 (tested with Python 3.5 and 3.6).
It requires at least Django in the 1.11 version and has been tested with versions between 1.11 and 2.1.

=======
UPGRADE
=======

After upgrade to 0.4.0, if you want to enable links to Youtube/Soundcould media,
you have to add an additional column to the database:

::

    ALTER TABLE paperclip_attachment ADD COLUMN attachment_video VARCHAR(200) NOT NULL DEFAULT '';


===========
BASIC USAGE
===========

* Add ``easy_thumbnails``, ``embed_video`` and ``paperclip`` to ``INSTALLED_APPS``

* Include urls

::

    urlpatterns = [
        ...
        url(r'^paperclip/', include('paperclip.urls')),
        ...
    ]

* Include scripts in template

::

    <script src="{% static "paperclip/bootstrap-confirm.js" %}" type="text/javascript"></script>
    <script src="{% static "paperclip/spin.min.js" %}" type="text/javascript"></script>
    <script src="{% static "paperclip/paperclip.js" %}" type="text/javascript"></script>

If you use bootstrap 3 or bootstrap 4, please include ``paperclip/bootstrap-3-confirm.js`` or ``paperclip/bootstrap-4-confirm.js`` instead of ``paperclip/bootstrap-confirm.js``.

* Include list and form in template

::

    {% include 'paperclip/attachment_list.html' with object=my_instance attachment_form_next=my_instance.get_detail_url %}

* Add paperclip models in one of your apps

::

    class FileType(paperclip.models.FileType):
        pass

    class Attachment(paperclip.models.Attachment):
        pass

* Configure

Define the following django setting:

::

    PAPERCLIP_ENABLE_VIDEO = False,
    PAPERCLIP_FILETYPE_MODEL = 'myapp.FileType'
    PAPERCLIP_ATTACHMENT_MODEL = 'myapp.Attachment'
    PAPERCLIP_ACTION_HISTORY_ENABLED = True

* Make migration and migrate


=========
TEMPLATES
=========

Three templates are embeded and can easily be overriden :

* ``paperclip/attachment_list.html`` : renders a table displaying attached files for an object and an upload form using the two following templates
* ``paperclip/_attachment_table.html`` : renders a table displaying attached files for an object
* ``paperclip/_attachment_form.html`` : renders an upload form


============
TEMPLATETAGS
============

Two templatetags are provided :

get_attachments_for
````````````````````
Resolves attachments that are attached to a given object. You can specify the variable name in the context the attachments are stored using the `as` argument. Default context variable name is `attachments`. You can filter on a specified FileType with the optional `only_type` argument.

Examples

::

    {% get_attachments_for my_instance as "my_attachments" %}
    {% get_attachments_for my_instance as "my_attachments" only_type my_filetype_instance %}

attachment_form
```````````````
Renders a "upload attachment" form. `obj` argument is required and represents the instance to which you want to associate the file. A bound form can be given optionnaly with the argument ``form``. Important : a ``attachment_form_next`` variable is expected in context. If you want to use a custom form class, you can add ``attachment_form_class`` variable in context too

Examples

::

    {% with object=my_instance attachment_form_next=my_instance.get_detail_url %}
        {% attachment_form object %}
    {% endwith %}

    OR

    # views.py
    ...
    context['object'] = my_instance
    context['attachment_form_next'] = my_instance.get_detail_url(=)
    context['attachment_form_class'] = MyAttachmentForm
    ...

    # template
    {% attachment_form object %}

==================
USE A CUSTOM FORM
==================

You can use a custom django form by following this steps. Parenthetically, It's the recommended solution if you want to use django-crispy-forms or django-floppyforms.

* Write your custom form :

::

    from paperclip.forms import AttachmentForm

    class MyAttachmentForm(AttachmentForm):
        ...

Note: To be sure to not break the form logic, we recommend to inherit from the native ``paperclip.forms.AttachmentForm``.

* Add your form class in a ``attachment_form_class`` variable of the main view context

::

    context['attachment_form_class'] = MyAttachmentForm

* Override ``'add_attachment'`` and ``'update_attachment'`` URLs to provide your custom form class in arguments

::

    from my_app.forms import MyAttachmentForm

    urlpatterns = [
        url(r'^paperclip/', include('paperclip.urls')),
        ...
        url(r'^add-for/(?P<app_label>[\w\-]+)/'
            r'(?P<model_name>[\w\-]+)/(?P<pk>\d+)/$',
            'paperclip.views.add_attachment',
            kwargs={'attachment_form': MyAttachmentForm},
            name="add_attachment"),

        url(r'^update/(?P<attachment_pk>\d+)/$',
            'paperclip.views.update_attachment',
            kwargs={'attachment_form': MyAttachmentForm},
            name="update_attachment"),
        ...
    ]


Note: Be sure to write these URLs after having included paperclip URLs.

=======
CLEANUP
=======

Deleting or changing an attachment does not remove the old attached file from disk.
From time to time you can clean obsolete files by running:

::

    ./manage.py clean_attachments


=======
AUTHORS
=======

|makinacom|_

.. |makinacom| image:: https://github.com/makinacorpus.png
.. _makinacom:  https://www.makina-corpus.com


=======
LICENSE
=======

    * LGPL
