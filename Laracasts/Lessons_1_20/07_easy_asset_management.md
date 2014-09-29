Easy Asset Management
=====================

[environment/asset-manag](https://github.com/KLVTZ/Laracasts/tree/environment/asset-manag)

In this lesson, we take a look at asset management or file files that require us
to shrink, compile, and output towards a different output. One helpful took for
this is [Guard](https://github.com/guard/guard). Guard is a gem in Ruby that
provides management for our SASS, Coffee Script, as well as live reload!

We installed our gems and then called `guard init` inside our Laravel project.
That will allow us to create a Guard file. That guard file will allow us to
launch compass as well as indicate input and output of our coffee script files.
We can also indicate how we would like to compile and minify our other scripts
for performance gain via `guard-uglify`.

We also have a config file which is created from our compass file itself which will allow 
us to indicate where we want certain files to compile to. We can indicate
source of SASS, CSS, and our default HTTP Path.

After all our settings are set, we can call `guard`. Guard  will then listen for
any changes done in our Laravel project. We can then add coffee script or SASS
files. And after saving, it automatically will compile both into proper
JavaScript and CSS respectively.
