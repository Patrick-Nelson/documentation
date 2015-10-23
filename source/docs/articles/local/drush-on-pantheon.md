---
title: Altering Drush Behavior with a Policy File
description: Learn how to use hook implementations within a policy file to modify the default behavior or version of Drush.
keywords: Drupal drush, command line, drupal, terminus drush, cli
---
One thing that makes Drush site aliases so powerful is the fact that they are defined in regular PHP executable files.  This gives you many options, as the alias definitions can easily be adjusted in code, factoring out common sections, or making other adjustments directly in the alias file. There are a number of reasons why one might want to separate this code from the alias data: 

* Pantheon automatically generates Drush site alias files for every site available to you. It is preferable to not edit the generated file, so that the local can be easily updated with a pristine copy any time the set of available sites changes.

* Future versions of Drush will be moving away from using PHP executable files for site aliases, and will instead encourage the use of YML files for alias definitions.  This eliminates the concern that a shared alias file might contain code that causes unpredictable side effects.

* Separating code from data is simply a best practice.

## Detect Your Drush Version

Start by running `drush version` to see which version you have installed. To get more information on your Drush configuration, run `drush status`.

## Create a Policy File
Drush provides a hook that allows you to separate code from data. The mechanism for altering Drush behavior is to use a policy file.  A policy is a regular Drush command file that contains hook implementations rather than Drush commands.

To get started, create a file called `policy.drush.inc` and place in in the .drush folder of your home directory.  There is an [example policy](http://www.drush.org/en/master/examples/) file in Drush’s examples folder you can use to get started.

This example writes a policy file that changes all remote aliases to use Drush 7 instead of Drush (the default version of Drush), but only if the target is the Pantheon platform.  Our `hook_drush_sitealias_alter` function looks like this:
```
function policy_drush_sitealias_alter(&$alias_record) {
  // Fix pantheon aliases!
  if ( isset($alias_record['remote-host']) &&
      (substr($alias_record['remote-host'],0,10) == 'appserver.') ) {
    $alias_record['path-aliases']['%drush-script'] = 'drush7';
  }
}
```

The `sitealias alter` hook takes a reference to one alias record, which it can adjust as desired.  We start out by testing to see if the record has a remote-host element that points to a server on the Pantheon platform.  If so, we set `%drush-script` to `drush7`.  Drush uses the `%drush-script` element to determine which program to use when running a remote Drush command for that alias. Pantheon has an environment set up so that there is a program called `drush7` available that points to Drush version 7.  With this policy file in place, you will be able to use the latest version of Drush on Pantheon:
```
    $ drush @pantheon.my-great-site.dev version
    Drush Version   :  7.0.0-rc1
```
Following the pattern of this example `alter` hook, you can make any adjustments needed to your alias files on the fly.  If you are currently embedding code inside the alias file directly, it's a good idea to convert it to a policy file, so you are ready for the shift to YML when it comes.

## See Also
[Five Steps to Feeling the Drush](https://pantheon.io/blog/five-steps-feeling-drupal-drush)  
[Drupal Drush Command-Line Utility](/docs/articles/local/drupal-drush-command-line-utility/)
