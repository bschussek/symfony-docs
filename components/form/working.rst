Working with Forms
==================

Form Structure
--------------

form is a tree of forms, get(), add(), remove()

every method that can be called on the root form can also be called on
other forms, for example getData()

Setting the Default Values
--------------------------

default vs. "data" option vs. setData()

Dealing with Empty Values
-------------------------

converted to NULL by default

empty_data option can be set to change that, for example to empty string ''

Changing the Field Names
------------------------

.. index::
   single: Forms; Built-in field types

.. _book-forms-type-reference:

Built-in Field Types
--------------------

Symfony comes standard with a large group of field types that cover all of
the common form fields and data types you'll encounter:

.. include:: /reference/forms/types/map.rst.inc

You can also create your own custom field types. This topic is covered in
the ":doc:`/cookbook/form/create_custom_field_type`" article of the cookbook.

.. index::
   single: Forms; Field type options

Field Type Options
~~~~~~~~~~~~~~~~~~

Each field type has a number of options that can be used to configure it.
For example, the ``dueDate`` field is currently being rendered as 3 select
boxes. However, the :doc:`date field</reference/forms/types/date>` can be
configured to be rendered as a single text box (where the user would enter
the date as a string in the box)::

    ->add('dueDate', 'date', array('widget' => 'single_text'))

.. image:: /images/book/form-simple2.png
    :align: center

Each field type has a number of different options that can be passed to it.
Many of these are specific to the field type and details can be found in
the documentation for each type.

.. sidebar:: The ``required`` option

    The most common option is the ``required`` option, which can be applied to
    any field. By default, the ``required`` option is set to ``true``, meaning
    that HTML5-ready browsers will apply client-side validation if the field
    is left blank. If you don't want this behavior, either set the ``required``
    option on your field to ``false`` or :ref:`disable HTML5 validation<book-forms-html5-validation-disable>`.

    Also note that setting the ``required`` option to ``true`` will **not**
    result in server-side validation to be applied. In other words, if a
    user submits a blank value for the field (either with an old browser
    or web service, for example), it will be accepted as a valid value unless
    you use Symfony's ``NotBlank`` or ``NotNull`` validation constraint.

    In other words, the ``required`` option is "nice", but true server-side
    validation should *always* be used.

.. sidebar:: The ``label`` option

    The label for the form field can be set using the ``label`` option,
    which can be applied to any field::

        ->add('dueDate', 'date', array(
            'widget' => 'single_text',
            'label'  => 'Due Date',
        ))

    The label for a field can also be set in the template rendering the
    form, see below.


.. index::
   single: Forms; Rendering in a template

.. _book-form-creating-form-classes:

Creating Form Classes
---------------------

As you've seen, a form can be created and used directly in a controller.
However, a better practice is to build the form in a separate, standalone PHP
class, which can then be reused anywhere in your application. Create a new class
that will house the logic for building the task form:

.. code-block:: php

    // src/Acme/TaskBundle/Form/Type/TaskType.php
    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;

    class TaskType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder->add('task');
            $builder->add('dueDate', null, array('widget' => 'single_text'));
        }

        public function getName()
        {
            return 'task';
        }
    }

This new class contains all the directions needed to create the task form
(note that the ``getName()`` method should return a unique identifier for this
form "type"). It can be used to quickly build a form object in the controller:

.. code-block:: php

    // src/Acme/TaskBundle/Controller/DefaultController.php

    // add this new use statement at the top of the class
    use Acme\TaskBundle\Form\Type\TaskType;

    public function newAction()
    {
        $task = ...;
        $form = $this->createForm(new TaskType(), $task);

        // ...
    }

Placing the form logic into its own class means that the form can be easily
reused elsewhere in your project. This is the best way to create forms, but
the choice is ultimately up to you.

.. _book-forms-data-class:

.. sidebar:: Setting the ``data_class``

    Every form needs to know the name of the class that holds the underlying
    data (e.g. ``Acme\TaskBundle\Entity\Task``). Usually, this is just guessed
    based off of the object passed to the second argument to ``createForm``
    (i.e. ``$task``). Later, when you begin embedding forms, this will no
    longer be sufficient. So, while not always necessary, it's generally a
    good idea to explicitly specify the ``data_class`` option by adding the
    following to your form type class::

        use Symfony\Component\OptionsResolver\OptionsResolverInterface;

        public function setDefaultOptions(OptionsResolverInterface $resolver)
        {
            $resolver->setDefaults(array(
                'data_class' => 'Acme\TaskBundle\Entity\Task',
            ));
        }

.. tip::

    When mapping forms to objects, all fields are mapped. Any fields on the
    form that do not exist on the mapped object will cause an exception to
    be thrown.

    In cases where you need extra fields in the form (for example: a "do you
    agree with these terms" checkbox) that will not be mapped to the underlying
    object, you need to set the property_path option to ``false``::

        use Symfony\Component\Form\FormBuilderInterface;

        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder->add('task');
            $builder->add('dueDate', null, array('property_path' => false));
        }

    Additionally, if there are any fields on the form that aren't included in
    the submitted data, those fields will be explicitly set to ``null``.

    The field data can be accessed in a controller with::

        $form->get('dueDate')->getData();

.. index::
   single: Forms; Embedded forms

Embedded Forms
--------------

Often, you'll want to build a form that will include fields from many different
objects. For example, a registration form may contain data belonging to
a ``User`` object as well as many ``Address`` objects. Fortunately, this
is easy and natural with the form component.

Embedding a Single Object
~~~~~~~~~~~~~~~~~~~~~~~~~

Suppose that each ``Task`` belongs to a simple ``Category`` object. Start,
of course, by creating the ``Category`` object::

    // src/Acme/TaskBundle/Entity/Category.php
    namespace Acme\TaskBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Category
    {
        /**
         * @Assert\NotBlank()
         */
        public $name;
    }

Next, add a new ``category`` property to the ``Task`` class::

    // ...

    class Task
    {
        // ...

        /**
         * @Assert\Type(type="Acme\TaskBundle\Entity\Category")
         */
        protected $category;

        // ...

        public function getCategory()
        {
            return $this->category;
        }

        public function setCategory(Category $category = null)
        {
            $this->category = $category;
        }
    }

Now that your application has been updated to reflect the new requirements,
create a form class so that a ``Category`` object can be modified by the user::

    // src/Acme/TaskBundle/Form/Type/CategoryType.php
    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilderInterface;
    use Symfony\Component\OptionsResolver\OptionsResolverInterface;

    class CategoryType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder->add('name');
        }

        public function setDefaultOptions(OptionsResolverInterface $resolver)
        {
            $resolver->setDefaults(array(
                'data_class' => 'Acme\TaskBundle\Entity\Category',
            ));
        }

        public function getName()
        {
            return 'category';
        }
    }

The end goal is to allow the ``Category`` of a ``Task`` to be modified right
inside the task form itself. To accomplish this, add a ``category`` field
to the ``TaskType`` object whose type is an instance of the new ``CategoryType``
class:

.. code-block:: php

    use Symfony\Component\Form\FormBuilderInterface;

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        // ...

        $builder->add('category', new CategoryType());
    }

The fields from ``CategoryType`` can now be rendered alongside those from
the ``TaskType`` class. To activate validation on CategoryType, add
the ``cascade_validation`` option::

    public function setDefaultOptions(OptionsResolverInterface $resolver)
    {
        $resolver->setDefaults(array(
            'data_class' => 'Acme\TaskBundle\Entity\Category',
            'cascade_validation' => true,
        ));
    }

Render the ``Category`` fields in the same way
as the original ``Task`` fields:

.. configuration-block::

    .. code-block:: html+jinja

        {# ... #}

        <h3>Category</h3>
        <div class="category">
            {{ form_row(form.category.name) }}
        </div>

        {{ form_rest(form) }}
        {# ... #}

    .. code-block:: html+php

        <!-- ... -->

        <h3>Category</h3>
        <div class="category">
            <?php echo $view['form']->row($form['category']['name']) ?>
        </div>

        <?php echo $view['form']->rest($form) ?>
        <!-- ... -->

When the user submits the form, the submitted data for the ``Category`` fields
are used to construct an instance of ``Category``, which is then set on the
``category`` field of the ``Task`` instance.

The ``Category`` instance is accessible naturally via ``$task->getCategory()``
and can be persisted to the database or used however you need.

Embedding a Collection of Forms
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can also embed a collection of forms into one form (imagine a ``Category``
form with many ``Product`` sub-forms). This is done by using the ``collection``
field type.

For more information see the ":doc:`/cookbook/form/form_collections`" cookbook
entry and  the :doc:`collection</reference/forms/types/collection>` field type reference.



embedded forms

(directly using the builder vs. separate types)

virtual forms

collection forms

file uploads
