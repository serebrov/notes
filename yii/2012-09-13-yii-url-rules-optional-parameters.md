Yii url rules - optional parameters
====================================

Assume we have an action "articles/get" which accepts optional parameters and we want to setup following URLs:

    articles/[article id or name]
    articles/[article id or name]/draft
    articles/[article id or name]/revisions/99
    articles/[article id or name]/revisions/98/draft
    articles/revisions/[revision id]
    articles/revisions/[revision id]/draft

We have a list of articles and each article has several revisions. Also each revision can have draft and published version.

In the code we have a single 'article/get' action which allows us to get specific article (last revision) by name ('GET articles/my-article') or id ('GET articles/33').

Using additional parameters we can get last revision draft ('GET articles/33/draft'), get specific revision ('GET articles/33/revisions/99') or get article using revision ID only ('GET articles/revisions/99').

Now we can setup URL rules for these routes like this:

    //- articles/revisions/98/draft
    array('article/get',
    'pattern' => 'articles/revisions/<revision:\d+>/<version:published|draft>',
    'verb' => 'GET'
    ),
    //- articles/revisions/99
    array('article/get',
    'pattern' => 'articles/revisions/<revision:\d+>',
    'verb' => 'GET'
    ),
    //- articles/77/revisions/99/draft
    array('article/get',
    'pattern' => 'articles/<article:[\w\d\.]+>/revisions/<revision:\d+>/<version:published|draft>',
    'verb' => 'GET'
    ),
    //- articles/77/revisions/99
    array('article/get',
    'pattern' => 'articles/<article:[\w\d\.]+>/revisions/<revision:\d+>',
    'verb' => 'GET'
    ),
    //- articles/77/draft
    array('article/get',
    'pattern' => 'articles/<revision:[\w\d\.]+>/<version:published|draft>',
    'verb' => 'GET'
    ),
    // articles/77 (or articles/[name])
    array('articles/get',
    'pattern' => 'article/<article:[\w\d\.]+>',
    'verb' => 'GET'
    ),

But there is much better option. We can describe this a single URL route with optional parameters (regexp groups with trailing question mark do the trick):

    array('app/get',
    'pattern' => 'apps(/<app:[\w\d\.]+>)?(/updates/<update:\d+>)?(/<revision:published|draft>)?',
    'verb' => 'GET'
    ),

Optional parameters at the end
------------------------------
The special case is when we need to allow any number of additional parameters at the end of the URL. This case is supported by URL manager:

    If a pattern ends with '/*', it means additional GET parameters may be appended to the path info part of the URL; otherwise, the GET parameters can only appear in the query string part.
