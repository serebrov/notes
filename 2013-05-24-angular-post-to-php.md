Angular.js POST data to PHP
============================================

By default angular.js sends all data in json.
So if you do a POST request to a PHP code then the $_POST superglobal will not be populated.

This can be solved in two ways - on the client side or on the server side.

Server-side solution
--------------------------------------------

On the server you can parse input and then decode data from json:

    $data = file_get_contents("php://input");
    $postData = json_decode($data);

Client-side solution
--------------------------------------------

On the client side the data can be sent in a way PHP expects it:

    $http({
        url:url
        data : $.param(data),
        method : 'POST',
        headers : {'Content-Type':'application/x-www-form-urlencoded; charset=UTF-8'}
    }).success(callback);

Here we set the 'Content-Type' header and encode data using jQuery's [$.param](http://api.jquery.com/jQuery.param/).
This also can be done for all requests as described [on stackoverflow](http://stackoverflow.com/questions/12190166/angularjs-any-way-for-http-post-to-send-request-parameters-instead-of-json).
