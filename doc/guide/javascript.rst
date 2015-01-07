#######################
XBlocks and JavaScript 
#######################

The following section describes how you

*********************
JavaScript Runtimes
*********************

The application that runs XBlocks uses JavaScript to load the XBlock.
Specifically, the JavaScript code loads the XBLock's:

* Handler
* Children 
* Map of children
  
Following is an example JavaScript runtime from the `XBlock SDK`_.

=================================
The XBlock SDK JavaScript Runtime 
=================================

The file `1.js`_ in the `XBlock SDK`_ provides the JavaScript runtime for the
XBlock workbench:

.. code-block:: JavaScript

    var RuntimeProvider = (function() {

    var getRuntime = function(version) {
      if (! this.versions.hasOwnProperty(version)) {
        throw 'Unsupported XBlock version: ' + version;
      }
      return this.versions[version];
    };

    var versions = {
      1: {
        handlerUrl: function(block, handlerName, suffix, query) {
        suffix = typeof suffix !== 'undefined' ? suffix : '';
        query = typeof query !== 'undefined' ? query : '';
        var usage = $(block).data('usage');
        var url_selector = $(block).data('url_selector');
        if (url_selector !== undefined) {
            baseUrl = window[url_selector];
        }
        else {baseUrl = handlerBaseUrl;}

          // studentId and handlerBaseUrl are both defined in block.html
          return (baseUrl + usage +
                           "/" + handlerName +
                           "/" + suffix +
                   "?student=" + studentId +
                           "&" + query);

        children: function(block) {
          return $(block).prop('xblock_children');
        },
        childMap: function(block, childName) {
          var children = this.children(block);
          for (var i = 0; i < children.length; i++) {
            var child = children[i];
            if (child.name == childName) {
              return child
            }
          }
        }
      }
    };
    return {
      getRuntime: getRuntime,
      versions: versions
    };
  }());


The Runtime Handler
*********************

The JavaScript runtime initializes the XBlock each time it is loaded by
a user and returns the handler so the XBlock can communicate with the server.

From the example above, the following part of the runtime generates and returns
the handler to the XBlock:

.. code-block:: JavaScript

    . . .

    var versions = {
      1: {
        handlerUrl: function(block, handlerName, suffix, query) {
        suffix = typeof suffix !== 'undefined' ? suffix : '';
        query = typeof query !== 'undefined' ? query : '';
        var usage = $(block).data('usage');
        var url_selector = $(block).data('url_selector');
        if (url_selector !== undefined) {
            baseUrl = window[url_selector];
        }
        else {baseUrl = handlerBaseUrl;}

          // studentId and handlerBaseUrl are both defined in block.html
          return (baseUrl + usage +
                           "/" + handlerName +
                           "/" + suffix +
                   "?student=" + studentId +
                           "&" + query);

    . . . 

The runtime handler code is called by the XBlock's JavaScript code to get the XBlock URL. 

For example, in the example `Thumbs XBlock`_ in the `XBlock SDK`_, the
`thumbs.js`_ file gets the handler from the XBlock runtime:

.. code-block:: JavaScript

    var handlerUrl = runtime.handlerUrl(element, 'vote');


XBlock Children
*********************

The JavaScript runtime also returns the list of child XBlocks to the XBlock.

From the example above, the following part of the runtime generates and returns
the list of children to the XBlock:

.. code-block:: JavaScript

    . . .

    children: function(block) {
          return $(block).prop('xblock_children');
        },
    . . . 

NEED EXAMPLE OF XBLOCK USING THIS

WHEN XBLOCK USES THIS VS. MAP

XBlock Child Map
*********************

The JavaScript runtime also returns the a map of child XBlocks to the running
XBlock.

From the example above, the following part of the runtime generates and returns
the list of children to the XBlock:

.. code-block:: JavaScript

    . . .

    childMap: function(block, childName) {
      var children = this.children(block);
      for (var i = 0; i < children.length; i++) {
        var child = children[i];
        if (child.name == childName) {
          return child
        }
      }
    }
    . . . 

NEED EXAMPLE OF XBLOCK USING THIS


*********************************
Using JavaScript in Your XBlock
*********************************

You provide user interaction in your XBlock through JavaScript.

For example, the `thumbs.js`_ file in the `XBlock SDK`_ provides users with the
ability to vote up or down on content:

.. code-block:: JavaScript
  
  function ThumbsBlock(runtime, element, init_args) {
    function updateVotes(votes) {
        $('.upvote .count', element).text(votes.up);
        $('.downvote .count', element).text(votes.down);
    }

    var handlerUrl = runtime.handlerUrl(element, 'vote');

    $('.upvote', element).click(function(eventObject) {
        $.ajax({
            type: "POST",
            url: handlerUrl,
            data: JSON.stringify({voteType: 'up'}),
            success: updateVotes
        });
    });

    $('.downvote', element).click(function(eventObject) {
        $.ajax({
            type: "POST",
            url: handlerUrl,
            data: JSON.stringify({voteType: 'down'}),
            success: updateVotes
        });
    });

    return {};

You must add the JavaScript file to the fragment in your XBlock.  For example,
`thumbs.py`_ loads the JavaScript above into the fragment for the student view
of the Thumbs XBlock:

.. code-block:: Python
  
  js_str = pkg_resources.resource_string(__name__, "static/js/src/thumbs.js")
  frag.add_javascript(unicode(js_str))
  frag.initialize_js('ThumbsBlock')

See :ref:`fragment` for more information.

.. _XBlock SDK: https://github.com/edx/xblock-sdk

.. _1.js: https://github.com/edx/xblock-sdk/blob/master/workbench/static/workbench/js/runtime/1.js

.. _Thumbs XBlock: https://github.com/edx/xblock-sdk/tree/master/sample_xblocks/thumbs

.. _thumbs.js: https://github.com/edx/xblock-sdk/blob/master/sample_xblocks/thumbs/static/js/src/thumbs.js

.. _thumbs.py: https://github.com/edx/xblock-sdk/blob/master/sample_xblocks/thumbs/thumbs.py