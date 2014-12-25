Node.js - how to get core module source
============================================

It is possible to ask node to show its core module source.
For example, we want to check the source of the readFileSunc() method:

    $ node
    > fs = require('fs');
    > fs.writeFileSync('fs.js', fs.toString())
    > fs.writeFileSync('fs.readFileSync.js', fs.readFileSync.toString())
    [Ctrl-C][Ctrl-C]

Now check the `fs.readFileSync.js` file in the current folder.

Also on some systems source code of core node modules is in the `/usr/lib/nodejs/`.

And another (less interesting) way to get core module source - is to look for it in the node github repository:
* check installed node.js version (node --version)
* find appropriate release on github (https://github.com/joyent/node/releases), for example [0.8.17](https://github.com/joyent/node/releases/tag/v0.8.17)
* click commit id on the left [c50c33e](https://github.com/joyent/node/commit/c50c33e9397d7a0a8717e8ce7530572907c054ad)
* click "Browse code" and see the [source](https://github.com/joyent/node/tree/c50c33e9397d7a0a8717e8ce7530572907c054ad)
