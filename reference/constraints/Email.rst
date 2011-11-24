Email
=====

Valida che un valore sia un indirizzo email valido. Il valore sottostante è
forzato a stringa, prima di essere validato.

+----------------+---------------------------------------------------------------------+
| Si applica a   | :ref:`proprietà o metodo<validation-property-target>`               |
+----------------+---------------------------------------------------------------------+
| Opzioni        | - `message`_                                                        |
|                | - `checkMX`_                                                        |
+----------------+---------------------------------------------------------------------+
| Classe         | :class:`Symfony\\Component\\Validator\\Constraints\\Email`          |
+----------------+---------------------------------------------------------------------+
| Validatore     | :class:`Symfony\\Component\\Validator\\Constraints\\EmailValidator` |
+----------------+---------------------------------------------------------------------+

Uso di base
-----------

.. configuration-block::

    .. code-block:: yaml

        # src/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                email:
                    - Email:
                        message: The email "{{ value }}" is not a valid email.
                        checkMX: true

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        namespace Acme\BlogBundle\Entity;
        
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /** 
             * @Assert\Email(
             *     message = "The email '{{ value }}' is not a valid email.",
             *     checkMX = true
             * )
             */
             protected $email;
        }

Opzioni
-------

message
~~~~~~~

**tipo**: ``stringa`` **predefinito**: ``This value is not a valid email address``

Messaggio mostrato se il dato sottostante non è un indirizzo email valido.

checkMX
~~~~~~~

**tipo**: ``booleano`` **predefinito**: ``false``

Se ``true``, sarà usata la funzione `checkdnsrr`_ di PHP per verificare la validità
del record MX dell'host dell'email fornita.

.. _`checkdnsrr`: http://www.php.net/manual/en/function.checkdnsrr.php