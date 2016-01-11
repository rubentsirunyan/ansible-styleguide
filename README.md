# Ansible Styleguide

## Table of Contents
  
  1. [Start of Files](#start-of-files)
  1. [Quotes](#quotes)
  1. [Environment](#environment)
  1. [Booleans](#booleans)
  1. [Key value pairs](#key-value-pairs)
  1. [Sudo](#sudo)

## Start of Files

By default start your file with `---` to show that this is a yaml file. 

```yaml
#bad
- name: 'start robot named S1m0ne'
  service:
    enabled: true
    name: 's1m0ne'
    state: 'started'
  become: true
  
#good
---
- name: 'start robot named S1m0ne'
  service:
    enabled: true
    name: 's1m0ne'
    state: 'started'
  become: true
```

### Why

Because it's easy and we often mix and match, just stick to one style.


## Quotes

**We always quote strings** and prefer single quotes over double quotes. The only time you should use double quotes is when they are nested within single quotes (e.g. Jinja map reference), or when your string requires escaping characters (e.g. using "\n" to represent a newline). If you must write a long string, we use the "folded scalar" style and omit all special quoting. The only things you should avoid quoting are booleans (e.g. true/false) and numbers (e.g. 42).

```yaml
# bad
- name: start robot named S1m0ne
  service:
    name: s1m0ne
    state: started
    enabled: true
  become: true

# good
- name: 'start robot named S1m0ne'
  service:
    name: 's1m0ne'
    state: 'started'
    enabled: true
  become: true

# double quotes w/ nested single quotes
- name: 'start all robots'
  service:
    name: '{{ item["robot_name"] }}''
    state: 'started'
    enabled: true
  with_items: '{{ robots }}'
  become: true

# double quotes to escape characters
- name 'print some text on two lines'
  debug:
    msg: "This text is on\ntwo lines"

# folded scalar style
- name: 'robot infos'
  debug:
    msg: >
      Robot {{ item['robot_name'] }} is {{ item['status'] }} and in {{ item['az'] }}
      availability zone with a {{ item['curiosity_quotient'] }} curiosity quotient.
  with_items: robots

# folded scalar when the string has nested quotes already
- name: 'print some text'
  debug:
    msg: >
      “I haven’t the slightest idea,” said the Hatter.

# don't quote booleans/numbers
- name: 'download google homepage'
  get_url:
    dest: '/tmp'
    timeout: 60
    url: 'https://google.com'
    validate_certs: true
```
### Why?

Even though strings are the default type for YAML, syntax highlighting looks better when explicitly set types. This also helps troubleshoot malformed strings when they should be properly escaped to have the desired effect.

## Environment 

When provisioning a server with environment variables add the environment variables to `/etc/environment` with lineinfile. Do this from the ansible role that is associated with the service or application that is being installed. For example for tomcat installation the `CATALINA_HOME` environment variable is often used to reference the folder that contains tomcat and its associated webapps. 

```yaml
- name: 'add line CATALINA_HOME to /etc/environment'
  lineinfile:
    dest: '/etc/environment'
    line: 'CATALINA_HOME={{ tomcat_home }}'
    state: 'present'
  sudo: true
```

### Why?
Environment definition files are typically shared so blowing them away by templating them can cause problems. Having the specific environment variable included by `lineinfile` makes it easier to track which applications are dependent upon the environment variable.

## Booleans

```yaml
# bad
- name: 'start sensu-client'
  service:
    name: 'sensu-client'
    state: 'restarted'
    enabled: 1
  become: 'yes'
 
# good
- name: 'start sensu-client'
  service:
    name: 'sensu-client'
    state: 'restarted'
    enabled: true
  become: true
```

### Why?
There are many different ways to specify a boolean value in ansible, `True/False`, `true/false`, `yes/no`, `1/0`. While it is cute to see all those options we prefer to stick to one : `true/false`. The main reasoning behind this is that Java and JavaScript have similar designations for boolean values. 

## Key value pairs

Use only one space after the colon when designating a key value pair

```yaml
# bad
- name : 'start sensu-client'
  service:
    name    : 'sensu-client'
    state   : 'restarted'
    enabled : true
  become : true


# good
- name: 'start sensu-client'
  service:
    name: 'sensu-client'
    state: 'restarted
    enabled: true
  become: true
```

**Always use the map syntax,** regardless of how many pairs exist in the map.

```yaml
# bad
- name: 'create checks directory to make it easier to look at checks vs handlers'
  file: 'path=/etc/sensu/conf.d/checks state=directory mode=0755 owner=sensu group=sensu'
  become: true
  
- name: 'copy check-memory.json to /etc/sensu/conf.d'
  copy: 'dest=/etc/sensu/conf.d/checks/ src=checks/check-memory.json'
  become: true
  
# good
- name: 'create checks directory to make it easier to look at checks vs handlers'
  file:
    group: 'sensu'
    mode: '0755'
    owner: 'sensu'
    path: '/etc/sensu/conf.d/checks'
    state: 'directory'
  become: true
  
- name: 'copy check-memory.json to /etc/sensu/conf.d'
  copy:
    dest: '/etc/sensu/conf.d/checks/'
    src: 'checks/check-memory.json'
  become: true
```

### Why?

It's easier to read and it's not hard to do. It reduces changset collisions for version control.

## Sudo
Use the new `become` syntax when designating that a task needs to be run with `sudo` privileges

```yaml
#bad
- name: 'template client.json to /etc/sensu/conf.d/'
  template:
    dest: '/etc/sensu/conf.d/client.json'
    src: 'client.json.j2'
  sudo: true
 
# good
- name: 'template client.json to /etc/sensu/conf.d/'
  template:
    dest: '/etc/sensu/conf.d/client.json'
    src: 'client.json.j2'
  become: true
```
### Why?
Using `sudo` was deprecated at [Ansible version 1.9.1](http://docs.ansible.com/ansible/become.html)
