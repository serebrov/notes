phpmyadmin and eaccelerator problem
============================================

Error when trying to access phpmyadmin (in Chrome):

    Error 324 (net::ERR_EMPTY_RESPONSE): The server closed the connection without sending any data.

The easiest way to fix I found is to disable eaccelerator in .htaccess (create it in the phpmyadmin root folder and add this line:

    php_flag eaccelerator.enable 0
