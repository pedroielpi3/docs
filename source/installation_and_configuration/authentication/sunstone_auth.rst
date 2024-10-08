.. _sunstone_auth:

=======================
Sunstone Authentication
=======================

By default, Sunstone works with the default ``core`` authentication method (user and password) although you can configure any authentication mechanism supported by OpenNebula. In this section, you will learn how to enable other authentication.

* Authentication is based on the credentials stored in the OpenNebula database for the user. Depending on the type of these credentials the authentication method can be: ``remote``or ``opennebula``

The following sections explain the client to Sunstone server authentication methods.

.. _suntone_basic_auth:

Basic Auth
===========

In the basic mode, username and password are matched to those in OpenNebula's database in order to authorize the user at the time of login. JWT-based sessions are then used to authenticate and authorize the requests.

.. code-block:: yaml

    :auth: opennebula

.. _sunstone_remote_auth:

Remote Auth
===========

This method performs OpenNebula login based on user extraction and compares it with the password value in the user database.

To update existing users to use remote authentication, change the controller to public and update the password as follows:

The user password has to be changed by running one of the following commands:

.. prompt:: bash $ auto

    $ oneuser chauth johndoe public "johndoe"

New users with this authentication method should be created as follows:

.. prompt:: bash $ auto

    $ oneuser create johndoe "johndoe" --driver public

To enable this login method, set the ``:auth:`` option in ``/etc/one/fireedge-server.conf`` to ``remote`` and restart FireEdge:

.. code-block:: yaml

    :auth: remote

The login screen will not display the username and password fields anymore, as all information is fetched from the user certificate:

|sunstone_remote_login|

Note that OpenNebula will not verify that the user holds a valid certificate at the time of login: this is expected to be done by the external container of the Sunstone server (normally Apache), whose job is to tell the user's browser that the site requires a user certificate and to check that the certificate is consistently signed by the chosen Certificate Authority (CA). The setup with Apache/SAML is the more common and tested. However, it can rely on Apache/Nginx for OIDC.

.. warning:: The Sunstone authentication only handles the authentication of the user at the time of login. Authentication of the user certificate is a complementary setup, which can rely on Apache.

.. _sunstone_ldap_auth:

LDAP/AD Auth
============

This method performs the OpenNebula login by delegating the authentication on a specific LDAP/AD server or several servers. 

No special configuration is needed in Sunstone, the authentication method should be kept as 'opennebula' like in the :ref:`Basic Auth case <suntone_basic_auth>`. However, this needs to be set up in the OpenNebula core side, to set up the ldap configuration this :ref:`guide <ldap>` needs to be followed.

X.509 Auth
==========

This method performs the login to OpenNebula based on a X.509 certificate’s DN (Distinguished Name). The DN is extracted from the certificate and matched to the password value in the user database.

The user password has to be changed by running one of the following commands:

.. prompt:: bash $ auto

    $ oneuser chauth johndoe x509 "/C=ES/O=ONE/OU=DEV/CN=clouduser"

or the same command using a certificate file:

.. prompt:: bash $ auto

    $ oneuser chauth johndoe --x509 --cert /tmp/my_cert.pem

New users with this authentication method should be created as follows:

.. prompt:: bash $ auto

    $ oneuser create johndoe "/C=ES/O=ONE/OU=DEV/CN=clouduser" --driver x509

or using a certificate file:

.. prompt:: bash $ auto

    $ oneuser create new_user --x509 --cert /tmp/my_cert.pem

To enable this login method, set the ``:auth:`` option in ``/etc/one/fireedge-server.conf`` to ``x509`` and restart FireEdge:

.. code-block:: yaml

    auth: x509

The login screen will not display the username and password fields anymore, as all information is fetched from the user certificate:

|sunstone_remote_login|

.. note::

   To configure this function in mandatory to have an :ref:`Apache/Nginx <large_scale_deployment>`  below are the rules for each one

Apache
------

.. code-block:: yaml

    <VirtualHost *:443>
        ...
        SSLVerifyClient require
        SSLVerifyDepth 1

        RequestHeader set X-Client-Dn "%{SSL_CLIENT_S_DN}s" 
        <IfModule mod_ssl.c>
            SSLProxyEngine On
        </IfModule>
    </VirtualHost>

Nginx
-----
.. code-block:: yaml

    ssl_verify_client optional;  
    location / {
        ...
        proxy_set_header X-Client-Dn $client_dn;
    }
 

.. _sunstone_2f_auth:

Two Factor Authentication
=========================

You can get an additional authentication level by using a two-factor authentication that not only requests the username and password but also the one-time (or pre-generated security) keys generated by an authenticator application.

|sunstone_2fa_auth|

Authenticator App
------------------

This method requires a token generated by any of these applications: `Google Authentication <https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2&hl=en>`__, `Authy <https://authy.com/download/>`__ or `Microsoft Authentication <https://www.microsoft.com/en-us/p/microsoft-authenticator/9nblgggzmcj6?activetab=pivot:overviewtab>`__.

To enable this, you must follow these steps:

-  Log in to Sunstone and select menu **Settings**. Inside, find the section **Two Factor Authentication**.
-  Inside, find and select the button **Register authenticator App**.

|sunstone_setting_auth|

-  Scan the Qr code with the aforementioned apps and enter the verification code.

|sunstone_setting_tfa_app|

Internally Sunstone adds the field ``TWO_FACTOR_AUTH_SECRET``.

|sunstone_template_user_auth|

-  To disable 2FA, go to the **Settings**, find the section **Two Factor Authentication** tab and click remove button.

|sunstone_settings_2fa_dissable|


.. |sunstone_remote_login| image:: /images/sunstone_login_remote.png
.. |sunstone_2fa_auth| image:: /images/sunstone_login_2fa.png
.. |sunstone_setting_auth| image:: /images/sunstone-settings-auth.png
.. |sunstone_setting_tfa_app| image:: /images/sunstone-settings-2fa-app.png
.. |sunstone_template_user_auth| image:: /images/sunstone-template-user-auth.png
.. |sunstone_settings_2fa_dissable| image:: /images/sunstone-settings-2fa-dissable.png
