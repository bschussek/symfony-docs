Form Validation
===============

info about html5 validation

marking fields as required

validation groups

group sequences

customizing error messages (invalid, others)

form validation without the Validator component (event listener + addError())

mapping errors to different fields (error_mapping, error_bubbling)

.. index::
   single: Forms; Validation

Form Validation
---------------

In the previous section, you learned how a form can be submitted with valid
or invalid data. In Symfony2, validation is applied to the underlying object
(e.g. ``Task``). In other words, the question isn't whether the "form" is
valid, but whether or not the ``$task`` object is valid after the form has
applied the submitted data to it. Calling ``$form->isValid()`` is a shortcut
that asks the ``$task`` object whether or not it has valid data.

Validation is done by adding a set of rules (called constraints) to a class. To
see this in action, add validation constraints so that the ``task`` field cannot
be empty and the ``dueDate`` field cannot be empty and must be a valid \DateTime
object.

.. configuration-block::

    .. code-block:: yaml

        # Acme/TaskBundle/Resources/config/validation.yml
        Acme\TaskBundle\Entity\Task:
            properties:
                task:
                    - NotBlank: ~
                dueDate:
                    - NotBlank: ~
                    - Type: \DateTime

    .. code-block:: php-annotations

        // Acme/TaskBundle/Entity/Task.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Task
        {
            /**
             * @Assert\NotBlank()
             */
            public $task;

            /**
             * @Assert\NotBlank()
             * @Assert\Type("\DateTime")
             */
            protected $dueDate;
        }

    .. code-block:: xml

        <!-- Acme/TaskBundle/Resources/config/validation.xml -->
        <class name="Acme\TaskBundle\Entity\Task">
            <property name="task">
                <constraint name="NotBlank" />
            </property>
            <property name="dueDate">
                <constraint name="NotBlank" />
                <constraint name="Type">\DateTime</constraint>
            </property>
        </class>

    .. code-block:: php

        // Acme/TaskBundle/Entity/Task.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\Type;

        class Task
        {
            // ...

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('task', new NotBlank());

                $metadata->addPropertyConstraint('dueDate', new NotBlank());
                $metadata->addPropertyConstraint('dueDate', new Type('\DateTime'));
            }
        }

That's it! If you re-submit the form with invalid data, you'll see the
corresponding errors printed out with the form.

.. _book-forms-html5-validation-disable:

.. sidebar:: HTML5 Validation

   As of HTML5, many browsers can natively enforce certain validation constraints
   on the client side. The most common validation is activated by rendering
   a ``required`` attribute on fields that are required. For browsers that
   support HTML5, this will result in a native browser message being displayed
   if the user tries to submit the form with that field blank.

   Generated forms take full advantage of this new feature by adding sensible
   HTML attributes that trigger the validation. The client-side validation,
   however, can be disabled by adding the ``novalidate`` attribute to the
   ``form`` tag or ``formnovalidate`` to the submit tag. This is especially
   useful when you want to test your server-side validation constraints,
   but are being prevented by your browser from, for example, submitting
   blank fields.

Validation is a very powerful feature of Symfony2 and has its own
:doc:`dedicated chapter</book/validation>`.

.. index::
   single: Forms; Validation groups

.. _book-forms-validation-groups:

Validation Groups
~~~~~~~~~~~~~~~~~

.. tip::

    If you're not using :ref:`validation groups <book-validation-validation-groups>`,
    then you can skip this section.

If your object takes advantage of :ref:`validation groups <book-validation-validation-groups>`,
you'll need to specify which validation group(s) your form should use::

    $form = $this->createFormBuilder($users, array(
        'validation_groups' => array('registration'),
    ))->add(...);

If you're creating :ref:`form classes<book-form-creating-form-classes>` (a
good practice), then you'll need to add the following to the ``setDefaultOptions()``
method::

    use Symfony\Component\OptionsResolver\OptionsResolverInterface;

    public function setDefaultOptions(OptionsResolverInterface $resolver)
    {
        $resolver->setDefaults(array(
            'validation_groups' => array('registration')
        ));
    }

In both of these cases, *only* the ``registration`` validation group will
be used to validate the underlying object.

Groups based on Submitted Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 2.1
   The ability to specify a callback or Closure in ``validation_groups``
   is new to version 2.1

If you need some advanced logic to determine the validation groups (e.g.
based on submitted data), you can set the ``validation_groups`` option
to an array callback, or a ``Closure``::

    use Symfony\Component\OptionsResolver\OptionsResolverInterface;

    public function setDefaultOptions(OptionsResolverInterface $resolver)
    {
        $resolver->setDefaults(array(
            'validation_groups' => array('Acme\\AcmeBundle\\Entity\\Client', 'determineValidationGroups'),
        ));
    }

This will call the static method ``determineValidationGroups()`` on the
``Client`` class after the form is bound, but before validation is executed.
The Form object is passed as an argument to that method (see next example).
You can also define whole logic inline by using a Closure::

    use Symfony\Component\Form\FormInterface;
    use Symfony\Component\OptionsResolver\OptionsResolverInterface;

    public function setDefaultOptions(OptionsResolverInterface $resolver)
    {
        $resolver->setDefaults(array(
            'validation_groups' => function(FormInterface $form) {
                $data = $form->getData();
                if (Entity\Client::TYPE_PERSON == $data->getType()) {
                    return array('person');
                } else {
                    return array('company');
                }
            },
        ));
    }


Custom Validation Logic
-----------------------

Alternative 1: Callback Constraint
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

works only with objects for now!

Alternative 2: True/False Constraint
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

works only with objects for now!

Alternative 3: Custom Constraint
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

works also with plain arrays


