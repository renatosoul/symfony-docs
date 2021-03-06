.. index::
   single: DependencyInjection; Factories

Using a Factory to Create Services
==================================

Symfony's Service Container provides a powerful way of controlling the
creation of objects, allowing you to specify arguments passed to the constructor
as well as calling methods and setting parameters. Sometimes, however, this
will not provide you with everything you need to construct your objects.
For this situation, you can use a factory to create the object and tell
the service container to call a method on the factory rather than directly
instantiating the class.

Suppose you have a factory that configures and returns a new ``NewsletterManager``
object by calling the static ``createNewsletterManager()`` method::

    class NewsletterManagerStaticFactory
    {
        public static function createNewsletterManager()
        {
            $newsletterManager = new NewsletterManager();

            // ...

            return $newsletterManager;
        }
    }

To make the ``NewsletterManager`` object available as a service, you can
configure the service container to use the
``NewsletterManagerStaticFactory::createNewsletterManager()`` factory method:

.. configuration-block::

    .. code-block:: yaml

        # app/config/services.yml
        services:
            # ...

            AppBundle\Email\NewsletterManager:
                # call the static method
                factory: ['AppBundle\Email\NewsletterManagerStaticFactory', createNewsletterManager]

    .. code-block:: xml

        <!-- app/config/services.xml -->

        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                http://symfony.com/schema/dic/services/services-1.0.xsd">

            <services>
                <service id="AppBundle\Email\NewsletterManager">
                    <!-- call the static method -->
                    <factory class="AppBundle\Email\NewsletterManagerStaticFactory" method="createNewsletterManager" />
                </service>
            </services>
        </container>

    .. code-block:: php

        // app/config/services.php

        use AppBundle\Email\NewsletterManager;
        use AppBundle\NumberGenerator;
        use AppBundle\Email\NewsletterManagerStaticFactory;
        // ...

        $container->register(NumberGenerator::class)
            // call the static method
            ->setFactory(array(NewsletterManagerStaticFactory::class, 'createNewsletterManager'));

.. note::

    When using a factory to create services, the value chosen for class
    has no effect on the resulting service. The actual class name
    only depends on the object that is returned by the factory. However,
    the configured class name may be used by compiler passes and therefore
    should be set to a sensible value.

If your factory is not using a static function to configure and create your
service, but a regular method, you can instantiate the factory itself as a
service too. Later, in the ":ref:`factories-passing-arguments-factory-method`"
section, you learn how you can inject arguments in this method.

Configuration of the service container then looks like this:

.. configuration-block::

    .. code-block:: yaml

        # app/config/services.yml

        services:
            # ...

            AppBundle\Email\NewsletterManagerFactory: ~

            AppBundle\Email\NewsletterManager:
                # call a method on the specified factory service
                factory: 'AppBundle\Email\NewsletterManagerFactory:createNewsletterManager'

    .. code-block:: xml

        <!-- app/config/services.xml -->

        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                http://symfony.com/schema/dic/services/services-1.0.xsd">

            <services>
                <service id="AppBundle\Email\NewsletterManagerFactory" />

                <service id="AppBundle\Email\NewsletterManager">
                    <!-- call a method on the specified factory service -->
                    <factory service="AppBundle\Email\NewsletterManagerFactory"
                        method="createNewsletterManager"
                    />
                </service>
            </services>
        </container>

    .. code-block:: php

        // app/config/services.php

        use AppBundle\Email\NewsletterManager;
        use AppBundle\Email\NewsletterManagerFactory;
        // ...

        $container->register(NewsletterManagerFactory::class);

        $container->register(NewsletterManager::class)
            // call a method on the specified factory service
            ->setFactory(array(
                new Reference(NewsletterManagerFactory::class),
                'createNewsletterManager',
            ));

.. note::

    The traditional configuration syntax in YAML files used an array to define
    the factory service and the method name:

    .. code-block:: yaml

        # app/config/services.yml

        app.newsletter_manager:
            class: AppBundle\Email\NewsletterManager 
            # new syntax
            factory: 'AppBundle\Email\NewsletterManagerFactory:createNewsletterManager'
            # old syntax
            factory: ['@AppBundle\Email\NewsletterManagerFactory', createNewsletterManager]

.. _factories-passing-arguments-factory-method:

Passing Arguments to the Factory Method
---------------------------------------

.. tip::

    Arguments to your factory method are :ref:`autowired <services-autowire>` if
    that's enabled for your service.

If you need to pass arguments to the factory method you can use the ``arguments``
options. For example, suppose the ``createNewsletterManager()`` method in the previous
example takes the ``templating`` service as an argument:

.. configuration-block::

    .. code-block:: yaml

        # app/config/services.yml

        services:
            # ...

            AppBundle\Email\NewsletterManager:
                factory:   'AppBundle\Email\NewsletterManagerFactory:createNewsletterManager'
                arguments: ['@templating']

    .. code-block:: xml

        <!-- app/config/services.xml -->

        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services
                http://symfony.com/schema/dic/services/services-1.0.xsd">

            <services>
                <!-- ... -->

                <service id="AppBundle\Email\NewsletterManager">
                    <factory service="AppBundle\Email\NewsletterManagerFactory" method="createNewsletterManager"/>
                    <argument type="service" id="templating"/>
                </service>
            </services>
        </container>

    .. code-block:: php

        // app/config/services.php

        use AppBundle\Email\NewsletterManager;
        use AppBundle\Email\NewsletterManagerFactory;
        use Symfony\Component\DependencyInjection\Reference;

        // ...
        $container->register(NewsletterManager::class)
            ->addArgument(new Reference('templating'))
            ->setFactory(array(
                new Reference(NewsletterManagerFactory::class),
                'createNewsletterManager',
            ));
