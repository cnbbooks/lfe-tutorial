# Robustness

There are several things which are wrong with the [messenger example](../concurrent/example.md) from the previous chapter. For example if a node where a user is logged on goes down without doing a log off, the user will remain in the server's ``user-list`` but the client will disappear thus making it impossible for the user to log on again as the server thinks the user already logged on.

Or what happens if the server goes down in the middle of sending a message leaving the sending client hanging for ever in the ``await-result`` function?
