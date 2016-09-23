# Updating school-website on the Raspberry Pi Computer

As [school-website](https://github.com/CreativeKids/school-website) uses submodules you will need to:

1. SSH into the server
2. `cd lektor/school-website`
3. `git pull && git submodule update --recursive`
4. `lektor build -O build && lektor deploy`

OR

Go to: http://raspberrypi.local/update.php
