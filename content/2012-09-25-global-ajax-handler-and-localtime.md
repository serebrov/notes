global ajax response handler and jquery.localtime plugin
============================================

The [jquery.localtime plugin](http://code.google.com/p/jquery-localtime) allows to convert date/time strings to a local user time on a client site.
By default it works when the page is loaded initially, but if some elements are updated via ajax then they do not converted and left in an UTC format.

Possible solution is to add some special handling to $.ajax 'success' handlers, but it can require a lot of modifications.
Better way is to set some global handler for all ajax requests and apply conversion to local time there.
I evaluated several approaches before I could find a working solution.

First try - run local time conversion in $.ajaxComplete (doesn't work)
---------------------------------------------

    $('body').ajaxComplete(function() {
        //code from jquery.localtime.js, jQuery.ready handler
        var format;
        var localise = function () {
            if (jQuery(this).is(":input")) {
                jQuery(this).val(jQuery.localtime.toLocalTime(jQuery(this).val(), format));
            } else {
                jQuery(this).text(jQuery.localtime.toLocalTime(jQuery(this).text(), format));
            }
        };
        var formats = jQuery.localtime.getFormat();
        var cssClass;
        for (cssClass in formats) {
            if (formats.hasOwnProperty(cssClass)) {
                format = formats[cssClass];
                //this will try to convert alrady converted texts and cause an exception
                jQuery("." + cssClass).each(localise);
            }
        }
    });

It doesn't work because `jQuery("." + cssClass).each(localise)` will try to apply localization to all elements, including these which already present in the page and converted. And localtime plugin starts throwing exceptions because it can not convert already converted data.

Second try - run local time conversion in $.ajaxComplete only for received data (doesn't work)
---------------------------------------------

    $('body').ajaxComplete(function(event, xhr, ajaxOptions) {
        //code from jquery.localtime.js, jQuery.ready handler
        var format;
        var localise = function () {
            if (jQuery(this).is(":input")) {
                jQuery(this).val(jQuery.localtime.toLocalTime(jQuery(this).val(), format));
            } else {
                jQuery(this).text(jQuery.localtime.toLocalTime(jQuery(this).text(), format));
            }
        };
        var formats = jQuery.localtime.getFormat();
        var cssClass;
        for (cssClass in formats) {
            if (formats.hasOwnProperty(cssClass)) {
                format = formats[cssClass];
                var result = $(xhr.resultText);
                result.find("." + cssClass).each(localise);
                xhr.resultText = //convert result back to text
            }
        }
    });

It doesn't work because ajaxComplete is invoked too late - when original $.ajax 'success' handler already done its work.

Third try - use $.ajaxSetup converters (doesn't work)
---------------------------------------------

    $.ajaxSetup({
        converters: {
            "html html": function( textValue ) {
                return localizeText(textValue);
            }
        }
    });

It doesn't work because converter is invoked only when we have two different formats like "text html" and we got unexpected format in ajax result.

Final try - use $.ajaxSetup prefilters (works!)
---------------------------------------------

    jQuery(function($) {
        $.ajaxPrefilter(function(options, originalOptions, jqXHR) {
            var success = options.success;
            //if reqested data type was text or html or *
            if (options.dataTypes.indexOf("text") !=-1 ||
                options.dataTypes.indexOf("html") != -1 ||
                options.dataTypes.indexOf("*") != -1
            ) {
                options.success = function(data, textStatus, jqXHR) {
                    // override success handling
                    data = localizeText(data);
                    if(typeof(success) === "function") return success(data, textStatus, jqXHR);
                };
            }
        });

        function localizeText(textValue) {
            var format;
            var localise = function () {
                if (jQuery(this).is(":input")) {
                    jQuery(this).val(jQuery.localtime.toLocalTime(jQuery(this).val(), format));
                } else {
                    jQuery(this).text(jQuery.localtime.toLocalTime(jQuery(this).text(), format));
                }
            };
            var formats = jQuery.localtime.getFormat();
            var cssClass;
            //convert text to jQuery var
            var result = $(textValue);
            for (cssClass in formats) {
                if (formats.hasOwnProperty(cssClass)) {
                    format = formats[cssClass];
                    result.find("." + cssClass).each(localise);
                }
            }
            var text = "";
            //convert jQuery var back to text
            $(result).each(function(index){
                if (this.nodeName.toLowerCase() == 'script') {
                    text += '<script type="text/javascript">'+$(this).html()+"</"+"script>";
                } else if (this.nodeType !== 1) {
                    //skip non-elements (HTML comments, text, etc)
                    return;
                } else {
                    text += $(this).appendTo('<div>').parent().html();
                }
            });
            return text;
        }
    });

This approach finally works.

The tricky part here is also a text-to-jQuery-and-back conversion.
While we can create a jQuery object (or array of objects) like this: `$(textValue)` and then process it, it is not so easy to convert it back to text.

After the conversion we have `result` as an array of jQuery object and if we do `result.html()` than we get only inner HTML of the first item.

And even if we wrap the result into a parent div like this `$(result).appendTo('<div>').parent().html()` than we get HTML but without javascript elements. So we need to iterate over the result and process HTML and javascript elements separately:

    $(result).each(function(index){
        if (this.nodeName.toLowerCase() == 'script') {
            text += '<script type="text/javascript">'+$(this).html()+"</"+"script>";
        } else if (this.nodeType !== 1) {
            //skip non-elements (HTML comments, text, etc)
            return;
        } else {
            text += $(this).appendTo('<div>').parent().html();
        }
    });


Links
-------------------
[Extending Ajax: Prefilters, Converters, and Transports](http://api.jquery.com/extending-ajax/)

[jQuery docs: $.ajaxComplete()](http://api.jquery.com/ajaxComplete/)

[Stackowerflow: How can I intercept ajax responses in jQuery before the event handler?](http://stackoverflow.com/questions/7256207/how-can-i-intercept-ajax-responses-in-jquery-before-the-event-handler)
