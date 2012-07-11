01 Connect policyhub masterfiles to central repo
================================================

The typical generic policy workflow goes from version control to
/var/cfengine/masterfiles on the policy hub, and from
/var/cfengine/masterfiles to /var/cfengine/inputs on the agent.

Task - Make sure that /var/cfengine/masterfiles on the policy hub is a
git clone of the central repository in the vagrant project. 

Central Repository: masterfiles.git (in the root of the Vagrant project
directory)
* the vagrant project directory is automatically mounted in each vagrant
* vm at /vagrant, so you should be able to access the repository at
* /vagrant/masterfiles.git from the policy hub.

If you remember from testing cf-sketch there was a sketch called
VCS::vcs_mirror. That sounds interesting, go ahead and check out that
sketches README.md to see what it does.

Well it can mirror git repositories, that sounds an awful lot like what
we need to do.  cf-sketch --install VCS::vcs_mirror

Solution
--------

bundle agent policyhub{
vars:
  redhat::
    "package_list" slist => { "git" };
    "central_git_repo" string => "/vagrant/masterfiles.git/";


classes:
  "masterfiles_is_clone" expression => fileexists("/var/cfengine/masterfiles/.git/config");

commands:
  !masterfiles_is_clone::
    "/usr/bin/git"
      handle => "policyhub_commands_git_remote_add_origin",
      args => "remote add --track master origin $(central_git_repo)",
      contain => in_dir("$(def.dir_masterfiles)"),
      comment => "$(def.dir_masterfiles) should be a clone of $(central_git_repo)";

packages:
  redhat::
    "$(package_list)"
      package_policy => "add",
      package_method => yum_rpm;

reports:
  cfengine::
    "This bundle is used to configure policyhub specific settings";
}