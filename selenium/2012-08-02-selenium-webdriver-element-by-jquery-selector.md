selenium webdriver - get webelement by jQuery selector
============================================

This can be necessary for example for selector like `#id > li:visible`.

If you will try to do `webdriver.find_element_by_css_selector` you will get an error message "The given selector `#id > li:visible` is either invalid or does not result in a WebElement."

The workaround is to use jQuery to find element. It can be done with this code (python):

    script = "return $('"+selector+"').get(0);"
    element = webdriver.execute_script(script);
