selenium - problem with big file upload
============================================
Problem
--------------------------------------------

Selenium 2.25.0, python 2.7, ubuntu 12.04, Firebox 4.0 (yes, old version but we need it) hangs when uploading a file larger then (about) 600KB.

Solution
--------------------------------------------

For now I fixed this by commenting out local file upload (three first lines):

    def send_keys(self, *value):
            """Simulates typing into the element."""
            #local_file = LocalFileDetector.is_local_file(*value)
            #if local_file is not None:
                #value = self._upload(local_file)

            typing = []
            for val in value:
                if isinstance(val, Keys):
                    typing.append(val)
                elif isinstance(val, int):
                    val = str(val)
                    for i in range(len(val)):
                        typing.append(val[i])
                else:
                    for i in range(len(val)):
                        typing.append(val[i])
            self._execute(Command.SEND_KEYS_TO_ELEMENT, {'value': typing})

Usually I do not change external libraries code, but in this case it is easiest way and it should not be necessary (I hope) after next selenium library update.

Links
--------------------------------------------
[Bug report](http://code.google.com/p/selenium/issues/detail?id=3812)
