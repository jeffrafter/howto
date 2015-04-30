# Bash

[http://www.kfirlavi.com/blog/2012/11/14/defensive-bash-programming/?utm_content=bufferbb394&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer](http://www.kfirlavi.com/blog/2012/11/14/defensive-bash-programming/?utm_content=bufferbb394&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer)

* Try to keep globals to minimum
* UPPER_CASE naming
* `readonly` declaration
* Use globals to replace cryptic `$0`, `$1`, etc.
* Globals I always use in my programs:


Additionally:

* Always quote all vars (use `"$var"` instead of `$var`)
* Always use `set -e` on scripts to set error handling
* Always use `-- -u` to require vars to be declared before using them