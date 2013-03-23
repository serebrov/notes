PHP - utf-8 strings handling
============================================

Enable mbstring [function overloading mode](http://www.php.net/manual/en/mbstring.overload.php) and set default
encoding for string functions to utf-8 in php.ini:

    mbstring.internal_encoding = UTF-8
    mbstring.func_overload = 7

These settings allow to use "usual" php string functions like substr() for utf-8 strings.
It is not recommended to set function overloading in per-directory context (via .htaccess).

Default encoding can also be set using [mb_internal_encoding function](http://php.net/manual/en/function.mb-internal-encoding.php):

    mb_internal_encoding('UTF-8');

Or encoding can set explicitly as argument in mbstring function:

    $sub = mb_substr($mbstr, 0, 1, 'utf-8');

Links
--------------------------------------------

[PHP Docs - Multibyte String](http://www.php.net/manual/en/book.mbstring.php)

[Yii Wiki - How to set up Unicode](http://www.yiiframework.com/wiki/16/how-to-set-up-unicode/)
