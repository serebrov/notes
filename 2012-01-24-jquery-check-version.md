jQuery - check minimal required version
============================================
To check whether jQuery is loaded to the page and verify minimum version:

    if (typeof jQuery == 'undefined' ||
    !/[1-9]\.[3-9].[1-9]/.test($.fn.jquery)
    ) {
        throw('jQuery version 1.3.1 or above is required');
    }

Here a regular expression determines a required jQuery version -

    /[X-9]\.[Y-9].[Z-9]/

For example, for 1.3.1 use

    /[1-9]\.[3-9].[1-9]/

and for 1.2.3 use

    /[1-9]\.[2-9].[3-9]/
