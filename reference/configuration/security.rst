.. index::
   single: Sicurezza; Riferimento configurazione

Riferimento configurazione sicurezza
====================================

Il sistema di sicurezza è una delle parti più potenti di Symfony2 e può
essere controllato in gran parte tramite la sua configurazione.

Configurazione predefinita completa
-----------------------------------

La seguente è la configurazione predefinita completa del sistema di sicurezza.
Ogni parte sarà spiegata nella prossima sezione.

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            access_denied_url:    ~ # Esempio: /foo/error403

            # la strategia può essere: none, migrate, invalidate
            session_fixation_strategy:  migrate
            hide_user_not_found:  true
            always_authenticate_before_granting: false
            erase_credentials: true
            access_decision_manager:
                strategy: affirmative
                allow_if_all_abstain: false
                allow_if_equal_granted_denied: true
            acl:

                # un nome configurato nella sezione doctrine.dbal
                connection:           ~
                cache:
                    id:                   ~
                    prefix:               sf2_acl_
                provider:             ~
                tables:
                    class: acl_classes
                    entry: acl_entries
                    object_identity: acl_object_identities
                    object_identity_ancestors: acl_object_identity_ancestors
                    security_identity: acl_security_identities
                voter:
                    allow_if_object_identity_unavailable: true

            encoders:
                # Esempi:
                Acme\DemoBundle\Entity\User1: sha512
                Acme\DemoBundle\Entity\User2:
                    algorithm: sha512
                    encode_as_base64: true
                    iterations: 5000

                # Opzioni/valori di esempio di come potrebbe essere un encoder personalizzato
                Acme\Your\Class\Name:
                    algorithm:            ~
                    ignore_case:          false
                    encode_as_base64:     true
                    iterations:           5000
                    id:                   ~

            providers:            # Obbligatorio
                # Esempi:
                    memory:
                    name:                memory
                        users:
                        foo:
                            password:            foo
                            roles:               ROLE_USER
                        bar:
                            password:            bar
                            roles:               [ROLE_USER, ROLE_ADMIN]
                entity:
                    entity:
                        class:               SecurityBundle:User
                        property:            username

                # Esempio di fornitore personalizzato
                some_custom_provider:
                    id:                   ~
                    chain:
                        providers:            []

            firewalls:            # Obblifatorio
                # Esempi:
                somename:
                    pattern: .*
                    request_matcher: id.di.un.servizio
                    access_denied_url: /pippo/error403
                    access_denied_handler: id.di.un.servizio
                    entry_point: id.di.un.servizio
                    provider: nome_di_un_provider_di_cui_sopra
                    context: nome
                    stateless: false
                    x509:
                        provider: nome_di_un_provider_di_cui_sopra
                    http_basic:
                        provider: nome_di_un_provider_di_cui_sopra
                    http_digest:
                        provider: nome_di_un_provider_di_cui_sopra
                    form_login:
                        check_path: /login_check
                        login_path: /login
                        use_forward: false
                        always_use_default_target_path: false
                        default_target_path: /
                        target_path_parameter: _target_path
                        use_referer: false
                        failure_path: /foo
                        failure_forward: false
                        failure_handler: id.di.un.servizio
                        success_handler: id.di.un.servizio
                        username_parameter: _username
                        password_parameter: _password
                        csrf_parameter: _csrf_token
                        intention: authenticate
                        csrf_provider: my.csrf_provider.id
                        post_only: true
                        remember_me: false
                    remember_me:
                        token_provider: nome
                        key: unaQualcheChiaveSegreta
                        name: NomeDelCookie
                        lifetime: 3600 # in secondi
                        path: /pippo
                        domain: undominio.pippo
                        secure: false
                        httponly: true
                        always_remember_me: false
                        remember_me_parameter: _remember_me
                    logout:
                        path:   /logout
                        target: /
                        invalidate_session: false
                        delete_cookies:
                            a: { path: null, domain: null }
                            b: { path: null, domain: null }
                        handlers: [id.di.un.servizio, id.di.un.altro.servizio]
                        success_handler: id.di.un.servizio
                    anonymous: ~

                # Valori e opzioni predefiniti per ogni firewall
                ascoltatore_di_un_firewall:
                    pattern:              ~
                    security:             true
                    request_matcher:      ~
                    access_denied_url:    ~
                    access_denied_handler:  ~
                    entry_point:          ~
                    provider:             ~
                    stateless:            false
                    context:              ~
                    logout:
                        csrf_parameter:       _csrf_token
                        csrf_provider:        ~
                        intention:            logout
                        path:                 /logout
                        target:               /
                        success_handler:      ~
                        invalidate_session:   true
                        delete_cookies:

                            # Prototype
                            name:
                                path:                 ~
                                domain:               ~
                        handlers:             []
                    anonymous:
                        key:                  4f954a0667e01
                    switch_user:
                        provider:             ~
                        parameter:            _switch_user
                        role:                 ROLE_ALLOWED_TO_SWITCH

            access_control:
                requires_channel:     ~

                # usare il formato urldecoded
                path:                 ~ # Esempio: ^/percorso della risorsa/
                host:                 ~
                ip:                   ~
                methods:              []
                roles:                []
            role_hierarchy:
                ROLE_ADMIN:      [ROLE_ORGANIZER, ROLE_USER]
                ROLE_SUPERADMIN: [ROLE_ADMIN]

.. _reference-security-firewall-form-login:

Configurazione del form di login
--------------------------------

Quando si usa l'ascoltatore di autenticazione ``form_login`` dietro un firewall,
ci sono diverse opzioni comuni per configurare l'esoerienza del form di login:

Il form e il processo di login
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*   ``login_path`` (tipo: ``stringa``, predefinito: ``/login``)
    È l'URL a cui l'utente sarà rinviato (a meno che ``use_forward`` non sia
    ``true``) quando prova ad accedere a una risorsa protetta,
    ma non è autenticato.

    Questo URL **deve** essere accessibile da un utente normale e non autenticato,
    altrimenti si creerebbe un loop infinito. Per dettagli, vedere
    ":ref:`Evitare problemi comuni<book-security-common-pitfalls>`".

*   ``check_path`` (tipo: ``stringa``, predefinito: ``/login_check``)
    È l'URL a cui il form di login viene inviato. Il firewall intercetterà
    ogni richiesta (solo quelle ``POST``, per impostazione predefinita) a questo URL
    e processerà le credenziali di login inviate.

    Assicurarsi che questo URL sia coperto dal proprio firewall principale (cioè non
    creare un firewall separato solo per l'URL ``check_path``).

*   ``use_forward`` (tipo: ``booleano``, predefinito: ``false``)
    Se si vuole che l'utente sia rimandato al form di login invece di essere 
    rinviato, impostare questa opzione a ``true``.

*   ``username_parameter`` (tipo: ``stringa``, predefinito: ``_username``)
    Questo il nome del campo che si dovrebbe dare al campo username del proprio
    form di login. Quando si invia il form a ``check_path``, il sistema di
    sicurezza cercherà un parametro POST con questo nome.

*   ``password_parameter`` (tipo: ``stringa``, predefinito: ``_password``)
    Questo il nome del campo che si dovrebbe dare al campo password del proprio
    form di login. Quando si invia il form a ``check_path``, il sistema di
    sicurezza cercherà un parametro POST con questo nome.

*   ``post_only`` (tipo: ``booleano``, predefinito: ``true``)
    Per impostazione predefinita, occorre inviare il proprio form di login
    all'URL ``check_path`` usando una richiesta POST. Impostando questa opzione
    a ``true``, si può inviare una richiesta GET all'URL ``check_path``.

Rinvio dopo il login
~~~~~~~~~~~~~~~~~~~~

* ``always_use_default_target_path`` (tipo: ``booleano``, predefinito: ``false``)
* ``default_target_path`` (tipo: ``stringa``, predefinito: ``/``)
* ``target_path_parameter`` (tipo: ``stringa``, predefinito: ``_target_path``)
* ``use_referer`` (tipo: ``booleano``, predefinito: ``false``)
