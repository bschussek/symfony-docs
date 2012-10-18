.. index::
    single: Forms
    single: Components; Form

The Form Component
==================

    The Form component allows you to easily create, process and reuse HTML
    forms.

<introduction>

Installation
------------

You can install the component in many different ways:

* Use the official Git repository (https://github.com/symfony/Form);
* Install it via PEAR (`pear.symfony.com/Form`);
* Install it via Composer (`symfony/form` on Packagist).

Configuration
-------------

.. tip::

    If you are working with the full-stack Symfony framework, the Form component
    is already configured for you and you can skip this section.

<configuration>

    use Symfony\Component\Form\Forms;

    $formFactory = Forms::createFormFactory();

<describe form factory>

<full setup>

    use Symfony\Component\Form\Forms;
    use Symfony\Component\Form\Extension\Csrf\CsrfExtension;
    use Symfony\Component\Form\Extension\Csrf\CsrfProvider\DefaultCsrfProvider;
    use Symfony\Component\Form\Extension\Templating\TemplatingExtension;
    use Symfony\Component\Form\Extension\Validator\ValidatorExtension;

    // Overwrite this with your own secret
    $csrfSecret = 'c2ioeEU1n48QF2WsHGWd2HmiuUUT6dxr';

    // Set up the form factory with all desired extensions
    $formFactory = Forms::createFormFactoryBuilder()
        ->addExtension(new CsrfExtension(new DefaultCsrfProvider($csrfSecret)))
        ->addExtension(new TemplatingExtension($phpEngine, null, array(
            realpath(__DIR__ . '/../vendor/symfony/framework-bundle/Symfony/Bundle/FrameworkBundle/Resources/views/Form'),
        )))
        ->addExtension(new ValidatorExtension($validator))
        ->getFormFactory();

.. note::

    <see> https://github.com/bschussek/php-forms

<publish form factory, e.g. through DI container>

Creating a Simple Form
----------------------

<sample form backed by arrays>

.. configuration-block::

    .. code-block:: php

        $form = $formFactory->createBuilder()
            ->add('task', 'text')
            ->add('dueDate', 'date')
            ->getForm();

        return $twig->render('

    .. code-block:: symfony

        // src/Acme/TaskBundle/Controller/DefaultController.php
        namespace Acme\TaskBundle\Controller;

        use Symfony\Bundle\FrameworkBundle\Controller\Controller;
        use Symfony\Component\HttpFoundation\Request;

        class DefaultController extends Controller
        {
            public function newAction(Request $request)
            {
                $form = $this->createFormBuilder()
                    ->add('task', 'text')
                    ->add('dueDate', 'date')
                    ->getForm();

                return $this->render('AcmeTaskBundle:Default:new.html.twig', array(
                    'form' => $form->createView(),
                ));
            }
        }

.. configuration-block::

    .. code-block:: php

        $defaults = array(
            'dueDate' => new \DateTime('tomorrow'),
        );

        $form = $formFactory->createBuilder('form', $defaults)
            ->add('task', 'text')
            ->add('dueDate', 'date')
            ->getForm();

    .. code-block:: symfony

        $defaults = array(
            'dueDate' => new \DateTime('tomorrow'),
        );

        $form = $this->createFormBuilder($defaults)
            ->add('task', 'text')
            ->add('dueDate', 'date')
            ->getForm();


Rendering the Form
~~~~~~~~~~~~~~~~~~


Handling Form Submissions
~~~~~~~~~~~~~~~~~~~~~~~~~

.. configuration-block::

    .. code-block:: php

        use Symfony\HttpFoundation\Request;

        $form = $formFactory->createBuilder()
            ->add('task', 'text')
            ->add('dueDate', 'date')
            ->getForm();

        $request = Request::createFromGlobals();

        if ($request->isMethod('POST')) {
            $form->bind($request);

            if ($form->isValid()) {
                // perform some action, such as saving the data to the database

                return $this->redirect($this->generateUrl('task_success'));
            }
        }

        // ...

    .. code-block:: symfony

        // ...

        public function newAction(Request $request)
        {
            // just setup a fresh $task object (remove the dummy data)
            $task = new Task();

            $form = $this->createFormBuilder($task)
                ->add('task', 'text')
                ->add('dueDate', 'date')
                ->getForm();

            if ($request->isMethod('POST')) {
                $form->bind($request);

                if ($form->isValid()) {
                    // perform some action, such as saving the task to the database

                    return $this->redirect($this->generateUrl('task_success'));
                }
            }

            // ...
        }


.. note::

    When not using HttpFoundation::

        if (isset($_POST[$form->getName()])) {
            $form->bind($_POST[$form->getName())

            // ...
        }

    but! no support for file uploads then


Form Validation
~~~~~~~~~~~~~~~

.. configuration-block::

    .. code-block:: php

        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\Type;

        $form = $formFactory->createBuilder()
            ->add('task', 'text', array(
                'constraints' => new NotBlank(),
            ))
            ->add('dueDate', 'date', array(
                'constraints' => array(
                    new NotBlank(),
                    new Type('\DateTime'),
                )
            ))
            ->getForm();

    .. code-block:: symfony

        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\Type;

        $form = $this->createFormBuilder()
            ->add('task', 'text', array(
                'constraints' => new NotBlank(),
            ))
            ->add('dueDate', 'date', array(
                'constraints' => array(
                    new NotBlank(),
                    new Type('\DateTime'),
                )
            ))
            ->getForm();


That's all you need to know for creating a basic form!
