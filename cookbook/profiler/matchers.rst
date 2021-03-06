.. index::
    single: Profiazione; Matcher

Usare un Matcher per abilitare dinamicamente il profilatore
===========================================================

Per impostazione predefinita, il profilatore è attivo solo nell'ambiente di sviluppo. Tuttavia,
è ipotizzabile che uno sviluppatore voglia vedere il profilatore anche in
produzione. Un'altra siturazione potrebbe essere quella di voler mostrare il profilatore solo
a un utente amministratore. In questi casi, si può abilitare il profilatore,
usando dei matcher.

Uso dei matcher predefiniti
---------------------------

Symfony fornisce un
:class:`built-in matcher <Symfony\\Component\\HttpFoundation\\RequestMatcher>`,
che può far corrispondere percorsi e IP. Per esempio, se si vuole mostrare il
profilatore solo accedendo con l'IP ``168.0.0.1``, si può
usare questa configurazione:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            # ...
            profiler:
                matcher:
                    ip: 168.0.0.1

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config>
            <framework:profiler
                ip="168.0.0.1"
            />
        </framework:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array(
            'profiler' => array(
                'ip' => '168.0.0.1',
            ),
        ));

Si può anche impostare un'opzione ``path``, per definire il percorso in cui il profilatore
debba essere attivo. Per esempio, impostandola ad ``^/admin/``, abiliterà
il profilatore solo per gli URL che iniziano per ``/admin/``.

Creare un matcher personalizzato
--------------------------------

Si può anche creare un matcher personalizzato. Non è altro che un servizio, che verifica
se il profilatore vada abilitato o meno. Per creare tale servizio, creare una classe
che implementi
:class:`Symfony\\Component\\HttpFoundation\\RequestMatcherInterface`. Questa
inferfaccia richiede un solo metodo:
:method:`Symfony\\Component\\HttpFoundation\\RequestMatcherInterface::matches`.
Questo metodo restituisce ``false`` per disabilitare il profilatore e ``true`` per
abilitarlo.

Per abilitare il profilatore solo per un utente con ``ROLE_SUPER_ADMIN``, si può usare
qualcosa come::

    // src/AppBundle/Profiler/SuperAdminMatcher.php
    namespace AppBundle\Profiler;

    use Symfony\Component\Security\Core\Authorization\AuthorizationCheckerInterface;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\RequestMatcherInterface;

    class SuperAdminMatcher implements RequestMatcherInterface
    {
        protected $authorizationChecker;

        public function __construct(AuthorizationCheckerInterface $authorizationChecker)
        {
            $this->authorizationChecker = $authorizationChecker;
        }

        public function matches(Request $request)
        {
            return $this->authorizationChecker->isGranted('ROLE_SUPER_ADMIN');
        }
    }

.. versionadded:: 2.6
    :class:`Symfony\\Component\\Security\\Core\\Authorization\\AuthorizationCheckerInterface` è stata
    introdotta in Symfony 2.6. In precedenza, si doveva usare il metodo ``isGranted`` di
    :class:`Symfony\\Component\\Security\\Core\\SecurityContextInterface`.

Occorre quindi configurare il servizio:

.. configuration-block::

    .. code-block:: yaml

        # app/config/services.yml
        services:
            app.profiler.matcher.super_admin:
                class: AppBundle\Profiler\SuperAdminMatcher
                arguments: ["@security.authorization_checker"]

    .. code-block:: xml

        <!-- app/config/services.xml -->
        <services>
            <service id="app.profiler.matcher.super_admin"
                class="AppBundle\Profiler\SuperAdminMatcher">
                <argument type="service" id="security.authorization_checker" />
        </services>

    .. code-block:: php

        // app/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;

        $container->setDefinition('app.profiler.matcher.super_admin', new Definition(
            'AppBundle\Profiler\SuperAdminMatcher',
            array(new Reference('security.authorization_checker'))
        );

.. versionadded:: 2.6
    Il servizio ``security.authorization_checker`` è stato introdotto in Symfony 2.6. Prima
    di Symfony 2.6, si doveva usare il metodo ``isGranted()`` del servizio ``security.context``.

Una volta registrato il servizio, l'unica cosa che resta è configurare il
profilatore per usare questo servizio come matcher:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            # ...
            profiler:
                matcher:
                    service: app.profiler.matcher.super_admin

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config>
            <!-- ... -->
            <framework:profiler
                service="app.profiler.matcher.super_admin"
            />
        </framework:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array(
            // ...
            'profiler' => array(
                'service' => 'app.profiler.matcher.super_admin',
            ),
        ));
