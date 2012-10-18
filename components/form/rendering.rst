.. index::
    single: Forms
    single: ??

Form Rendering
==============

Template Engines
----------------

Twig
~~~~

show configuration

Symfony Templating Component
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

show configuration

forms and twig

forms and php templating



rendering a form in a template

what are template variables (form.vars.x), how to access them in blocks (x) vs.
in other parts of the template (form.vars.x)

add stuff from the cookbook entry form_customization


disabling fields

changing the label

checking a field for errors

.. _form-rendering-template:

.. index::
   single: Forms; Creating form classes

Rendering a Form in a Template
------------------------------

So far, you've seen how an entire form can be rendered with just one line
of code. Of course, you'll usually need much more flexibility when rendering:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}
        <form action="{{ path('task_new') }}" method="post" {{ form_enctype(form) }}>
            {{ form_errors(form) }}

            {{ form_row(form.task) }}
            {{ form_row(form.dueDate) }}

            {{ form_rest(form) }}

            <input type="submit" />
        </form>

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Default/newAction.html.php -->
        <form action="<?php echo $view['router']->generate('task_new') ?>" method="post" <?php echo $view['form']->enctype($form) ?>>
            <?php echo $view['form']->errors($form) ?>

            <?php echo $view['form']->row($form['task']) ?>
            <?php echo $view['form']->row($form['dueDate']) ?>

            <?php echo $view['form']->rest($form) ?>

            <input type="submit" />
        </form>

Let's take a look at each part:

* ``form_enctype(form)`` - If at least one field is a file upload field, this
  renders the obligatory ``enctype="multipart/form-data"``;

* ``form_errors(form)`` - Renders any errors global to the whole form
  (field-specific errors are displayed next to each field);

* ``form_row(form.dueDate)`` - Renders the label, any errors, and the HTML
  form widget for the given field (e.g. ``dueDate``) inside, by default, a
  ``div`` element;

* ``form_rest(form)`` - Renders any fields that have not yet been rendered.
  It's usually a good idea to place a call to this helper at the bottom of
  each form (in case you forgot to output a field or don't want to bother
  manually rendering hidden fields). This helper is also useful for taking
  advantage of the automatic :ref:`CSRF Protection<forms-csrf>`.

The majority of the work is done by the ``form_row`` helper, which renders
the label, errors and HTML form widget of each field inside a ``div`` tag
by default. In the :ref:`form-theming` section, you'll learn how the ``form_row``
output can be customized on many different levels.

.. tip::

    You can access the current data of your form via ``form.vars.value``:

    .. configuration-block::

        .. code-block:: jinja

            {{ form.vars.value.task }}

        .. code-block:: html+php

            <?php echo $view['form']->get('value')->getTask() ?>

.. index::
   single: Forms; Rendering each field by hand

Rendering each Field by Hand
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``form_row`` helper is great because you can very quickly render each
field of your form (and the markup used for the "row" can be customized as
well). But since life isn't always so simple, you can also render each field
entirely by hand. The end-product of the following is the same as when you
used the ``form_row`` helper:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_errors(form) }}

        <div>
            {{ form_label(form.task) }}
            {{ form_errors(form.task) }}
            {{ form_widget(form.task) }}
        </div>

        <div>
            {{ form_label(form.dueDate) }}
            {{ form_errors(form.dueDate) }}
            {{ form_widget(form.dueDate) }}
        </div>

        {{ form_rest(form) }}

    .. code-block:: html+php

        <?php echo $view['form']->errors($form) ?>

        <div>
            <?php echo $view['form']->label($form['task']) ?>
            <?php echo $view['form']->errors($form['task']) ?>
            <?php echo $view['form']->widget($form['task']) ?>
        </div>

        <div>
            <?php echo $view['form']->label($form['dueDate']) ?>
            <?php echo $view['form']->errors($form['dueDate']) ?>
            <?php echo $view['form']->widget($form['dueDate']) ?>
        </div>

        <?php echo $view['form']->rest($form) ?>

If the auto-generated label for a field isn't quite right, you can explicitly
specify it:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_label(form.task, 'Task Description') }}

    .. code-block:: html+php

        <?php echo $view['form']->label($form['task'], 'Task Description') ?>

Some field types have additional rendering options that can be passed
to the widget. These options are documented with each type, but one common
options is ``attr``, which allows you to modify attributes on the form element.
The following would add the ``task_field`` class to the rendered input text
field:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_widget(form.task, { 'attr': {'class': 'task_field'} }) }}

    .. code-block:: html+php

        <?php echo $view['form']->widget($form['task'], array(
            'attr' => array('class' => 'task_field'),
        )) ?>

If you need to render form fields "by hand" then you can access individual
values for fields such as the ``id``, ``name`` and ``label``. For example
to get the ``id``:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form.task.vars.id }}

    .. code-block:: html+php

        <?php echo $form['task']->get('id') ?>

To get the value used for the form field's name attribute you need to use
the ``full_name`` value:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form.task.vars.full_name }}

    .. code-block:: html+php

        <?php echo $form['task']->get('full_name') ?>

Twig Template Function Reference
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you're using Twig, a full reference of the form rendering functions is
available in the :doc:`reference manual</reference/forms/twig_reference>`.
Read this to know everything about the helpers available and the options
that can be used with each.

.. index::
   single: Forms; Theming
   single: Forms; Customizing fields

.. _form-theming:

Theming
-------

Every part of how a form is rendered can be customized. You're free to change
how each form "row" renders, change the markup used to render errors, or
even customize how a ``textarea`` tag should be rendered. Nothing is off-limits,
and different customizations can be used in different places.

Symfony uses templates to render each and every part of a form, such as
``label`` tags, ``input`` tags, error messages and everything else.

In Twig, each form "fragment" is represented by a Twig block. To customize
any part of how a form renders, you just need to override the appropriate block.

In PHP, each form "fragment" is rendered via an individual template file.
To customize any part of how a form renders, you just need to override the
existing template by creating a new one.

To understand how this works, let's customize the ``form_row`` fragment and
add a class attribute to the ``div`` element that surrounds each row. To
do this, create a new template file that will store the new markup:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Form/fields.html.twig #}
        {% block form_row %}
        {% spaceless %}
            <div class="form_row">
                {{ form_label(form) }}
                {{ form_errors(form) }}
                {{ form_widget(form) }}
            </div>
        {% endspaceless %}
        {% endblock form_row %}

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Form/form_row.html.php -->
        <div class="form_row">
            <?php echo $view['form']->label($form, $label) ?>
            <?php echo $view['form']->errors($form) ?>
            <?php echo $view['form']->widget($form, $parameters) ?>
        </div>

The ``form_row`` form fragment is used when rendering most fields via the
``form_row`` function. To tell the form component to use your new ``form_row``
fragment defined above, add the following to the top of the template that
renders the form:

.. configuration-block:: php

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}
        {% form_theme form 'AcmeTaskBundle:Form:fields.html.twig' %}

        {% form_theme form 'AcmeTaskBundle:Form:fields.html.twig' 'AcmeTaskBundle:Form:fields2.html.twig' %}

        <form ...>

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Default/new.html.php -->
        <?php $view['form']->setTheme($form, array('AcmeTaskBundle:Form')) ?>

        <?php $view['form']->setTheme($form, array('AcmeTaskBundle:Form', 'AcmeTaskBundle:Form')) ?>

        <form ...>

The ``form_theme`` tag (in Twig) "imports" the fragments defined in the given
template and uses them when rendering the form. In other words, when the
``form_row`` function is called later in this template, it will use the ``form_row``
block from your custom theme (instead of the default ``form_row`` block
that ships with Symfony).

Your custom theme does not have to override all the blocks. When rendering a block
which is not overridden in your custom theme, the theming engine will fall back
to the global theme (defined at the bundle level).

If several custom themes are provided they will be searched in the listed order
before falling back to the global theme.

To customize any portion of a form, you just need to override the appropriate
fragment. Knowing exactly which block or file to override is the subject of
the next section.

.. versionadded:: 2.1
   An alternate Twig syntax for ``form_theme`` has been introduced in 2.1. It accepts
   any valid Twig expression (the most noticeable difference is using an array when
   using multiple themes).

   .. code-block:: html+jinja

       {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}

       {% form_theme form with 'AcmeTaskBundle:Form:fields.html.twig' %}

       {% form_theme form with ['AcmeTaskBundle:Form:fields.html.twig', 'AcmeTaskBundle:Form:fields2.html.twig'] %}

For a more extensive discussion, see :doc:`/cookbook/form/form_customization`.

.. index::
   single: Forms; Template fragment naming

.. _form-template-blocks:

Form Fragment Naming
~~~~~~~~~~~~~~~~~~~~

In Symfony, every part of a form that is rendered - HTML form elements, errors,
labels, etc - is defined in a base theme, which is a collection of blocks
in Twig and a collection of template files in PHP.

In Twig, every block needed is defined in a single template file (`form_div_layout.html.twig`_)
that lives inside the `Twig Bridge`_. Inside this file, you can see every block
needed to render a form and every default field type.

In PHP, the fragments are individual template files. By default they are located in
the `Resources/views/Form` directory of the framework bundle (`view on GitHub`_).

Each fragment name follows the same basic pattern and is broken up into two pieces,
separated by a single underscore character (``_``). A few examples are:

* ``form_row`` - used by ``form_row`` to render most fields;
* ``textarea_widget`` - used by ``form_widget`` to render a ``textarea`` field
  type;
* ``form_errors`` - used by ``form_errors`` to render errors for a field;

Each fragment follows the same basic pattern: ``type_part``. The ``type`` portion
corresponds to the field *type* being rendered (e.g. ``textarea``, ``checkbox``,
``date``, etc) whereas the ``part`` portion corresponds to *what* is being
rendered (e.g. ``label``, ``widget``, ``errors``, etc). By default, there
are 4 possible *parts* of a form that can be rendered:

+-------------+--------------------------+---------------------------------------------------------+
| ``label``   | (e.g. ``form_label``)    | renders the field's label                               |
+-------------+--------------------------+---------------------------------------------------------+
| ``widget``  | (e.g. ``form_widget``)   | renders the field's HTML representation                 |
+-------------+--------------------------+---------------------------------------------------------+
| ``errors``  | (e.g. ``form_errors``)   | renders the field's errors                              |
+-------------+--------------------------+---------------------------------------------------------+
| ``row``     | (e.g. ``form_row``)      | renders the field's entire row (label, widget & errors) |
+-------------+--------------------------+---------------------------------------------------------+

.. note::

    There are actually 3 other *parts*  - ``rows``, ``rest``, and ``enctype`` -
    but you should rarely if ever need to worry about overriding them.

By knowing the field type (e.g. ``textarea``) and which part you want to
customize (e.g. ``widget``), you can construct the fragment name that needs
to be overridden (e.g. ``textarea_widget``).

.. index::
   single: Forms; Template fragment inheritance

Template Fragment Inheritance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In some cases, the fragment you want to customize will appear to be missing.
For example, there is no ``textarea_errors`` fragment in the default themes
provided with Symfony. So how are the errors for a textarea field rendered?

The answer is: via the ``form_errors`` fragment. When Symfony renders the errors
for a textarea type, it looks first for a ``textarea_errors`` fragment before
falling back to the ``form_errors`` fragment. Each field type has a *parent*
type (the parent type of ``textarea`` is ``text``, its parent is ``form``),
and Symfony uses the fragment for the parent type if the base fragment doesn't
exist.

So, to override the errors for *only* ``textarea`` fields, copy the
``form_errors`` fragment, rename it to ``textarea_errors`` and customize it. To
override the default error rendering for *all* fields, copy and customize the
``form_errors`` fragment directly.

.. tip::

    The "parent" type of each field type is available in the
    :doc:`form type reference</reference/forms/types>` for each field type.

.. index::
   single: Forms; Global Theming

Global Form Theming
~~~~~~~~~~~~~~~~~~~

In the above example, you used the ``form_theme`` helper (in Twig) to "import"
the custom form fragments into *just* that form. You can also tell Symfony
to import form customizations across your entire project.

Twig
....

To automatically include the customized blocks from the ``fields.html.twig``
template created earlier in *all* templates, modify your application configuration
file:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        twig:
            form:
                resources:
                    - 'AcmeTaskBundle:Form:fields.html.twig'
            # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <twig:config ...>
                <twig:form>
                    <resource>AcmeTaskBundle:Form:fields.html.twig</resource>
                </twig:form>
                <!-- ... -->
        </twig:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('twig', array(
            'form' => array('resources' => array(
                'AcmeTaskBundle:Form:fields.html.twig',
             ))
            // ...
        ));

Any blocks inside the ``fields.html.twig`` template are now used globally
to define form output.

.. sidebar::  Customizing Form Output all in a Single File with Twig

    In Twig, you can also customize a form block right inside the template
    where that customization is needed:

    .. code-block:: html+jinja

        {% extends '::base.html.twig' %}

        {# import "_self" as the form theme #}
        {% form_theme form _self %}

        {# make the form fragment customization #}
        {% block form_row %}
            {# custom field row output #}
        {% endblock form_row %}

        {% block content %}
            {# ... #}

            {{ form_row(form.task) }}
        {% endblock %}

    The ``{% form_theme form _self %}`` tag allows form blocks to be customized
    directly inside the template that will use those customizations. Use
    this method to quickly make form output customizations that will only
    ever be needed in a single template.

    .. caution::

        This ``{% form_theme form _self %}`` functionality will *only* work
        if your template extends another. If your template does not, you
        must point ``form_theme`` to a separate template.

PHP
...

To automatically include the customized templates from the ``Acme/TaskBundle/Resources/views/Form``
directory created earlier in *all* templates, modify your application configuration
file:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            templating:
                form:
                    resources:
                        - 'AcmeTaskBundle:Form'
        # ...


    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config ...>
            <framework:templating>
                <framework:form>
                    <resource>AcmeTaskBundle:Form</resource>
                </framework:form>
            </framework:templating>
            <!-- ... -->
        </framework:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array(
            'templating' => array('form' =>
                array('resources' => array(
                    'AcmeTaskBundle:Form',
             )))
            // ...
        ));

Any fragments inside the ``Acme/TaskBundle/Resources/views/Form`` directory
are now used globally to define form output.

.. _`Twig Bridge`: https://github.com/symfony/symfony/tree/master/src/Symfony/Bridge/Twig
.. _`form_div_layout.html.twig`: https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig
.. _`view on GitHub`: https://github.com/symfony/symfony/tree/master/src/Symfony/Bundle/FrameworkBundle/Resources/views/Form
