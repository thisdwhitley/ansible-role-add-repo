# Ansible Role: add-repo

## ![Tested on](https://img.shields.io/badge/Fedora-27%2C28%2C29-blue.svg) ![Tested on](https://img.shields.io/badge/EL-7-red.svg)

This is an Ansible role to install repositories.  I am going to make an attempt
to make it generic enough to accept repositories of different types (but note
that I don't have a strong background or use case for debian based repos at this
point so I'm largely guessing).

I prefer to pass the variables "into" the role from the playbook versus by
including variable files.  This is because I hope to make the role usable by
other projects/roles.  I don't know if this logic makes sense or not, but I am
essentially attempting to remove the variables from the role itself.

My current (2019-05-09) order of preference for installing software is:

1. flatpak (I'm still learning about the benefits here)
2. package via repo
3. package (w/o repo)
4. from tar/etc
5. from source

It is also my opinion that if an entity is providing an RPM, it should provide
it via a repository.  And if it is providing a repo of RPMs, it should provide
an RPM to install that repo...I live in a dream world.

## Overview

At a very high level, this role will:

* attempt to determine what was passed into the role and *intelligently* do
  something with it...

This role is currently passing my pretty rudimentary tests for the following
operating systems:

* centos7
* fedora27
* fedora28
* fedora29

The testing of this role is very specific to the role I've set up in molecule,
but I think I'm ok with that.

## Important Notes

* I do not check to see if the repository is already configured on a system
  (with the thought that, for instance, a system will not install a repo RPM if
  it is already installed) because I am naive and can't pronounce *idempotent*
* the original intent was to extend this to deb repos too, and I think it would
  be easy, I just don't know enough about the use there

## Requirements

Any package or additional repository requirements will be addressed in the role.

## Role Variables

All of these variables should be considered **optional** however, be aware that
sanity checking is minimal (if at all):

* `repos` *ideally you could configure repos differently per repo, and this
          nested list of repos allows for that*
  * `repo_url`
    * this is the URL of a repo package for an RPM based system **NOTE: this is
      not a baseurl used to create your own repo file**
  * `repo_key_url`
    * if there is a repo key at a URL, provide it and it will get imported
  * `repo_codename` **NOT USED YET**
    * this would be the codename if using deb (such as `stable` or `jessie`)
  * `repo_section` **NOT USED YET**
    * this would be the section if using deb (such as `main`)

## Example Playbook

Playbook with configuration options specified:

```yaml
- hosts: localhost
  connection: local
  roles:
    - role: add-repo
      repos:
        - repo_url: "https://download1.rpmfusion.org/free/{{ (ansible_distribution == 'Fedora') | ternary('fedora','el') }}/rpmfusion-free-release-{{ ansible_distribution_major_version }}.noarch.rpm"
        - repo_url: "https://dl.fedoraproject.org/pub/epel/epel-release-latest-{{ ansible_distribution_major_version }}.noarch.rpm"
          repo_key_url: "https://archive.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-{{ ansible_distribution_major_version }}"
        - repo_url: "http://deb.debian.org/debian"
          repo_codename: "stretch-backports"
          repo_section: "main"
        - repo_url: "ppa:nginx/stable"
```

## Inclusion

I envision this role being included in a larger project through the use of a
`requirements.yml` file.  So here is an example of what you would need in your
file:

```yaml
# get the add-repo role from gitlab
- src: https://github.com/thisdwhitley/ansible-role-add-repo.git
  scm: git
  name: add-repo
```

Have the above in a `requirements.yml` file for your project would then allow
you to "install" it (prior to use in some sort of setup script?) with:

```bash
ansible-galaxy install -p ./roles -r requirements.yml
```

## Testing

I am relying heavily on the work by Jeff Geerling in using molecule for testing
this role.  The testing of this role is busted.  I have not figured out an
appropriate test using `testinfra` for what I'm hoping to achieve.  I've just
been using molecule for linting and test idempotency...

Please review those files, but here is a list of OSes currently being tested 
(using geerlingguy's container images):

* centos7
* fedora27
* fedora28
* fedora29

## To-do

* finish the work to get this working for debian based systems (the logic is
  already in `install.yml` for the most part)
* figure out how to test appropriately

## References

* [How I test Ansible configuration on 7 different OSes with Docker](https://www.jeffgeerling.com/blog/2018/how-i-test-ansible-configuration-on-7-different-oses-docker)

## License

All parts of this project are made available under the terms of the [MIT
License](LICENSE).