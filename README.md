# bmt (brian's module tool)

This is my tool for managing my Perl modules and their repositories.
It's specifically designed for the way that I like to do things.

* [Makefile.PL as a modulino](https://www.masteringperl.org/2015/01/makefile-pl-as-a-modulino/)
* I store repos in GitHub, Bitbucket, and GitLab at the same time ([Use several Git Services at Once](https://briandfoy.github.io/use-several-git-services-at-once/))

Things are likely broken because I'm not supporting this for other
people to use, so I tend to take less care than I would otherwise. And,
this uses a Perl module that is on GitHub but that I haven't released
to CPAN, so good luck with that:

	use lib qw(/Users/brian/Dev/ghojo/lib);

But, you can use this as a starting point for your own customized tool.
Steal anything you like.

# About the tool

There are all sorts of tasks I do inside a Perl module repository. Many
of these tasks are informational, such as "tell me if the local version
of this module is higher than the version on CPAN":

	% bmt versions

This task then knows how to talk to MetaCPAN and how to discover the
local version.

Other tasks make changes. For example, I have a set of general GitHub
workflows that I use across all of my modules. To update those from
their repository ([briandfoy/github_workflows](https://github.com/briandfoy/github_workflows)) to the current repo:

	% bmt update_workflows

Then various magic happens, including modifying the general workflows
for local requirements, such as randomizing the [`cron` triggers](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule).

Most times I notice myself doing something often, I turn it into a command
in this tool. This script grows by accretion.

Not only that, but if I decide that I want some existing target to do
soemthing differently, I just change it. Indeed, one of the targets
brings up the script in my editor:

	% bmt edit

# Some interesting ideas to steal

## Do it the same way everywhere.

I standardized the GitHub issues labels for all of my repos. And, using
the GitHub API, I can easily handle that:

	% bmt update_github_labels

If I add or change labels, I just run this again. Easy peasy.

## Registering commands

I have all of these commands, and I want to run the `help` command to
see what it can do:

	% bmt help
	actions                   open the actions Github page
	adjust_appveyor           adjust AppVeyor config for min Perl versions
	adjust_linux_workflow     adjust linux workflow perl versions
	all_remote                configure the all remote
	appveyor                  open the Appveyor page
	appveyor_page             open the Appveyor page
	...

Instead of tracking this out of band, things get into this list because
the subroutine adds itself to the list at compile time through an
attribute:

	sub cpan_testers :Register("show Testers result for the latest version") ( $release = undef ) {

## Makefile.PL exposes its arguments

Read more about this in [Makefile.PL as a modulino](https://www.masteringperl.org/2015/01/makefile-pl-as-a-modulino/).
*Makefile.PL* is basically a Perl module's build file, and various steps
need some of the information from that file:

    $ bmt arguments
	{
	  "ABSTRACT_FROM" => "lib/App/bmt.pm",
	  "AUTHOR" => "brian d foy <briandfoy\@pobox.com>",
	  "BUILD_REQUIRES" => {},
	  "CONFIGURE_REQUIRES" => {
		"ExtUtils::MakeMaker" => "6.64",
		"File::Spec::Functions" => 0
	  },
	...
	}

Getting at these settings makes automation much easier.

## Compose commands

There are sets of lower-level commands that I want to run.  I just compose
them into a single command:

	sub update_all :Register("update many things at once") () {
		state @actions = qw(
			update_releaserc
			update_release_token
			update_appveyor
			update_workflows
			update_github_labels
			create_cb
			all_remote
			update_copyright
			);

		_combine_actions( @actions );

		'Updated everything';
		}




