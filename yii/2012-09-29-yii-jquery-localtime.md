Yii and jquery.localtime.js - display dates in user local timezone
===========================================

With this method we work on the server with UTC timezone dates and convert them 
to a user local timezone on client.

Use ‘TIMESTAMP’ type for date/datetime DB fields
-------------------------------------------

Setup MySQL and PHP timezone
-------------------------------------------

Set UTC timezone for both MySQL and PHP.

Yii db config (MySQL):

    ‘db’ => array(
        'connectionString' => '...’,
        'initSQLs'=>"set time_zone='+00:00’;",
    );
    
And PHP timezone:

    date_default_timezone_set(‘UTC’);

Generate HTML with UTC dates in ISO 8601
-------------------------------------------
See [date formats here](http://code.google.com/p/jquery-localtime/wiki/Usage)
Helper functions to convert dates:
- MySQL date to UTC
 
    $utcString = gmdate('Y-m-d\TH:i:s\Z', strtotime($dateTimeString));

- UTC date to HTML
 
    echo CHtml::tag('span', array('class'=>'localtime'), $utcString);

- UTC date to timestamp
 
     $ts = strtotime($utcString);

Add localtime plugin and localtimex extension
-------------------------------------------
Include scripts into the page, see [jquery.localtime.js plugin here](http://code.google.com/p/jquery-localtime/) and 
jquery.localtimex.js code at the end.
To disable jquery.localtime.js default initialization pass empty format to setFormat() 
method (or remove jQuery.ready block at the end of jquery.localtime.js).

    <script type="text/javascript" src="/js/jquery.localtime-0.5.js"></script>
    <script type="text/javascript" src="/js/jquery.localtimex.js"></script>
    <script type="text/javascript">$.localtime.setFormat({}); </script>

Use localtimex (do not do initialization for jquery.localtime.js):

    <script type="text/javascript">
        $('.localtime').localtimex('dd MMM yyyy HH:mm:ss');
    </script>

By default plugin will localize specified elements and re-localize them in the case of ajax updates.
Date format is ISO 8601, see [description](http://code.google.com/p/jquery-localtime/wiki/Usage).

Date inputs
-------------------------------------------
jquery.localtimex.js plugin also supports date inputs localization (tested with jQuery UI datepicker and depends on it). 

It will convert UTC value to local time, so user can work with it. Before form submit plugin will convert it back to UTC.

jquery.localtimex.js
-------------------------------------------
    /**
     * @name jquery.localtimex.js
     * @author Boris Serebrov
     *
     * Depends on: jQuery, jQuery UI datepicker (local date parsing),
     * jquery.localtime-0.5.js (http://code.google.com/p/jquery-localtime/)
     *
     * Usage:
     * - do not configure jquery.localtime plugin, instead use this plugin
     *   on elements you need localize:
     *   $('.localtime').localtimex('dd MMM yyyy HH:mm:ss');
     *   $('.localdate').localtimex('dd MMM yyyy');
     *
     * Ajax response handling:
     * - by default plugin will localize elements after ajax requests
     * - to disable this behavior pass 'ajaxLocalize':false to options
     *   $('.localdate').localtimex('dd MMM yyyy', {'ajaxLocalize':false});
     *
     * Date inputs handling:
     * - server put UTC time into input (or leave it empty)
     * - UTC value converted to local time, user can modify it in local format
     * - on form submit value converted back to UTC and posted to server
     *
     * jQuery UI datepicker configuration:
     * - add 'localtimex' plugin to date input 
     * - set dateFormat of a date picker accordingly to localtime plugin output format
     *   - they use different standards for date presentation, 
     *     so we have to describe the same presentation twice
     *   - for example: localtime plugin - 'MM/dd/yyyy', datepicker - 'mm/dd/yy'
     * - initial input value should be UTC in ISO format or empty
     */
    (function($) {
      $.fn.localtimex = function(format, opt) {
    
        format = format || 'yyyy-MM-dd HH:mm:ss';
        opt = $.extend({
            'ajaxLocalize': true
        }, opt);
    
        if (opt.ajaxLocalize) {
          var self = this;
          $('body').ajaxComplete(function() {
            $(self.selector).each(function() {
              $.localtimex.localize(this, format);
            });
          });
        }
    
        return this.each(function() {
          $.localtimex.localize(this, format);
          if ($(this).is(":input")) {
            var self = this;
            $(this.form).on('submit form-pre-serialize', function() {
                var dt = $.datepicker.parseDate(
                  $(self).datepicker('option', 'dateFormat'), $(self).val()
                );
                if (dt) {
                  $(self).val(dt.toISOString());
                }
            });
          }
        });
      };
    
      $.localtimex = {
        localize: function(element, format) {
          if (!$(element).attr('data-localized')) {
            if ($(element).is(":input")) {
              var utcVal = $(element).val();
              //input can be empty initially
              if (utcVal.length) {
                $(element).attr('value', $.localtime.toLocalTime($(element).val(), format));
              }
            } else {
              $(element).text($.localtime.toLocalTime($(element).text(), format));
            }
          }
          $(element).attr('data-localized', true);
        }
      };
    }) (jQuery);
    
Links
-------------------------------------------
[Yii wiki: Local time zones and locales](http://www.yiiframework.com/wiki/197/local-time-zones-and-locales/)

[Yii wiki: Using International Dates](http://www.yiiframework.com/wiki/183/using-international-dates/)

[Yii extension: i18n-datetime-behavior](http://www.yiiframework.com/extension/i18n-datetime-behavior/)

[Yii forum: DATETIME and internationalization](http://www.yiiframework.com/forum/index.php/topic/9950-datetime-and-internationalization/)

[Yii forum: Dealing with i18N date formats](http://www.yiiframework.com/forum/index.php/topic/3649-dealing-with-i18n-date-formats/)

[Yii playground: User input advanced example](http://www.yiiplayground.cubedwater.com/index.php?r=InternationalizationModule/datetime/userinput)

[Yii playground: LocaleManager application component](http://www.yiiplayground.cubedwater.com/index.php?r=InternationalizationModule/datetime/localeManager)
