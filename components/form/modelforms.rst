Forms and the Domain Model
==========================

creating forms backed by objects

default data (read from objects)

when are setters called? (by_reference)

mapping to-one relationships

mapping to-many relationships (adders, removers, how to configure them)

mapping objects with a constructor with arguments

guessing based on metadata

decoupling fields from the object (mapping = false)

customizing the mapping (property_path)

accessing the object in the view (form.vars.data)


.. index::
   single: Forms; Field type guessing

.. _book-forms-field-guessing:

Field Type Guessing
-------------------

Now that you've added validation metadata to the ``Task`` class, Symfony
already knows a bit about your fields. If you allow it, Symfony can "guess"
the type of your field and set it up for you. In this example, Symfony can
guess from the validation rules that both the ``task`` field is a normal
``text`` field and the ``dueDate`` field is a ``date`` field::

    public function newAction()
    {
        $task = new Task();

        $form = $this->createFormBuilder($task)
            ->add('task')
            ->add('dueDate', null, array('widget' => 'single_text'))
            ->getForm();
    }

The "guessing" is activated when you omit the second argument to the ``add()``
method (or if you pass ``null`` to it). If you pass an options array as the
third argument (done for ``dueDate`` above), these options are applied to
the guessed field.

.. caution::

    If your form uses a specific validation group, the field type guesser
    will still consider *all* validation constraints when guessing your
    field types (including constraints that are not part of the validation
    group(s) being used).

.. index::
   single: Forms; Field type guessing

Field Type Options Guessing
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In addition to guessing the "type" for a field, Symfony can also try to guess
the correct values of a number of field options.

.. tip::

    When these options are set, the field will be rendered with special HTML
    attributes that provide for HTML5 client-side validation. However, it
    doesn't generate the equivalent server-side constraints (e.g. ``Assert\MaxLength``).
    And though you'll need to manually add your server-side validation, these
    field type options can then be guessed from that information.

* ``required``: The ``required`` option can be guessed based on the validation
  rules (i.e. is the field ``NotBlank`` or ``NotNull``) or the Doctrine metadata
  (i.e. is the field ``nullable``). This is very useful, as your client-side
  validation will automatically match your validation rules.

* ``max_length``: If the field is some sort of text field, then the ``max_length``
  option can be guessed from the validation constraints (if ``MaxLength`` or ``Max``
  is used) or from the Doctrine metadata (via the field's length).

.. note::

  These field options are *only* guessed if you're using Symfony to guess
  the field type (i.e. omit or pass ``null`` as the second argument to ``add()``).

If you'd like to change one of the guessed values, you can override it by
passing the option in the options field array::

    ->add('task', null, array('max_length' => 4))
