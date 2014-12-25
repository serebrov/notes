express.js and ejs - reuse template on server and client
============================================
[ejs](https://github.com/visionmedia/ejs) has a [client-side support](https://github.com/visionmedia/ejs#client-side-support) but documentation and examples do not describe how to reuse the same template on the server and on the client side.

For now I found two ways to do it. First way is to send a request from the client, get a template from a file and render it. And the second - put a template into the page when render it on the server and then just use the template on the client.

Dynamically get a template from the server and render it
--------------------------------------------
To be able to load a template from a file we need to put shared templates (used both on the sever and the client) under the "public" folder. Assume we have "public/tpl/template.ejs".

Render on the server (this code will be inside of a parent view which uses 'template.ejs' as partial):

    <div class="dataItems">
    <% for (var i=0; i < dataItems.length; i++) { %>
        <%- partial('../public/tpl/template', {data: dataItems[i]}) %>
    <% } %>
    </div>

Render on the client (you need to include client version of ejs.js into the page):

    function insertTemplate(data) {
        getTemplate('/tpl/template.ejs', function renderTemplate(err, tpl) {
        if (err) {
            throw err;
        }
        var tpl = require('ejs').render(tpl, {data: data});
        $(tpl).prependTo('div.dataItems');
        });
    }
    function getTemplate(file, callback) {
        $.ajax(file, {
            type: 'GET',
            success: function(data, textStatus, xhr) {
            return callback(null, data);
            },
            error: function(xhr, textStatus, error) {
            return callback(error);
            }
        });
    }

Here the 'getTemplate' function will load a template from the file on the server and return its contents. To reduce server load it could be enhanced to cache already loaded templates.
PREPARE A TEMPLATE ON THE SERVER AND USE IT ON THE CLIENT
With this method we do not need to put the template into the public folder. Assume we have our template in `views/_template.ejs`. Before rendering the view we put this template into the variable:

    var templates = {
        template: require('fs').readFileSync('./views/_template.ejs', 'utf-8')
    };
    res.render('myview', {
        …,
        templates: templates
    });

The server-side rendering will be usual use of partial. Another thing we need to do inside the view is to convert 'templates' view variable into client-side javascript variable.

    <div class="dataItems">
    <% for (var i=0; i < dataItems.length; i++) { %>
        <%- partial('_template', {data: dataItems[i]}) %>
    <% } %>
    </div>

    <script type="text/javascript">
        jQuery(document).ready(function(){
        //save templates to variable
        window.templates = <%-JSON.stringify(templates)%>;
        });
    </script>

Now to render the template on the client:

    var tplData = {...};
    var html = require('ejs').render(templates.template, {data: tplData});
    $(html).prependTo('div.dataItems');

Links
--------------------------------------------
[Discussion in the ejs issue tracker](https://github.com/visionmedia/ejs/issues/52)
