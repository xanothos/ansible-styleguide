# Ansible Styleguide

## Table of Contents
  
  1. [Practices](#practices)
  1. [Start of Files](#start-of-files)
  1. [End of Files](#end-of-files)
  2. [Fully Qualified Collection Names](#fully-qualified-collection-name)
  3. [Quotes](#quotes)
  5. [Booleans](#booleans)
  6. [Key value pairs](#key-value-pairs)
  7. [Deprecated Options](#deprecated_options)
  8. [Hosts Declaration](#hosts-declaration)
  9. [Task Declaration](#task-declaration)
  10. [Include Declaration](#include-declaration)
  11. [Spacing](#spacing)

## Practices

You should follow the [Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html) defined by the Ansible documentation when developing playbooks.

### Why?

The Ansible developers have a good understanding of how the playbooks work and where they look for certain files. Following these practices will avoid a lot of problems.

### Why Doesn't Your Style Follow Theirs?

The script examples are inconsistent in style throughout the Ansible documentation; the purpose of this document is to define a consistent style that can be used throughout Ansible scripts to create robust, readable code.

## Start of Files

You should start your scripts with some comments explaining what the script's purpose does (and an example usage, if necessary), followed by `---` with blank lines around it, then followed by the rest of the script.

```yaml
#bad
- name: 'Change s1m0n3's status'
  ansible.builtin.service:
    enabled: true
    name: 's1m0ne'
    state: '{{ state }}'
  become: true
  
#good
##################################################################
#
# Example usage: ansible-playbook -e state=started playbook.yml
# This playbook changes the state of s1m0n3 the robot
#
##################################################################

---

- name: 'Change s1m0n3's status'
  ansible.builtin.service:
    enabled: true
    name: 's1m0ne'
    state: '{{ state }}'
  become: true
```

### Why?

This makes it easier to quickly find out the purpose/usage of a script, either by opening the file or using the `head` command.

## End of Files

You should always end your files with a newline.

### Why?

This is common Unix best practice, and avoids any prompt misalignment when printing files in a terminal.

## Fully Qualified Collection Name

Always use the fully qualified collection name (FQCN) to reference modules.

### Why?

To quote the Ansible documentation directly:
> In most cases, you can use the short module name debug even without specifying the collections: keyword. Despite that, we recommend you use the FQCN for easy linking to the module documentation and to avoid conflicting with other collections that may have the same module name.

## Quotes

**We always quote strings, but not task names**, and prefer single quotes over double quotes. The only time you should use double quotes is when they are nested within single quotes (e.g. Jinja map reference), or when your string requires escaping characters (e.g. using "\n" to represent a newline). If you must write a long string, we use the "folded block scalar" style and omit all special quoting ([see YAML syntax guide](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html) for information about the differences between folded and literal block scalar styles). The only values you should avoid quoting are booleans (e.g. true/false), numbers (e.g. 42), and variables defined in the local Ansible environment (e.g. boolean logic or names of variables to which values are assigned).

```yaml
# bad
- name: 'start robot named S1m0ne'
  ansible.builtin.service:
    name: s1m0ne
    state: started
    enabled: true
  become: true

# good
- name: start robot named S1m0ne
  ansible.builtin.service:
    name: 's1m0ne'
    state: 'started'
    enabled: true
  become: true

# double quotes w/ nested single quotes
- name: start all robots
  ansible.builtin.service:
    name: '{{ item["robot_name"] }}'
    state: 'started'
    enabled: true
  with_items: '{{ robots }}'
  become: true

# double quotes to escape characters
- name: print some text on two lines
  ansible.builtin.debug:
    msg: "This text is on\ntwo lines"

# folded scalar style
- name: robot infos
  ansible.builtin.debug:
    msg: >
      Robot {{ item['robot_name'] }} is {{ item['status'] }} and in {{ item['az'] }}
      availability zone with a {{ item['curiosity_quotient'] }} curiosity quotient.
  with_items: robots

# folded scalar when the string has nested quotes already
- name: print some text
  ansible.builtin.debug:
    msg: >
      “I haven’t the slightest idea,” said the Hatter.

# don't quote booleans/numbers
- name: download google homepage
  ansible.builtin.get_url:
    dest: '/tmp'
    timeout: 60
    url: 'https://google.com'
    validate_certs: true

# variables example 1
- name: set a variable
  ansible.builtin.set_fact:
    my_var: 'test'

# variables example 2
- name: print my_var
  ansible.builtin.debug:
    var: my_var
  when: ansible_os_family == 'Darwin'

# variables example 3
- name: set another variable
  ansible.builtin.set_fact:
    my_second_var: '{{ my_var }}'
```
### Why?

Even though strings are the default type for YAML, syntax highlighting looks better when explicitly set types. This also helps troubleshoot malformed strings when they should be properly escaped to have the desired effect.

## Booleans

Always use the values `true` and `false` when referencing boolean values.

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
While there are many different ways to specify a boolean value in ansible [`True/False`, `true/false`, `yes/no`, `1/0`], we prefer to stick to one: `true/false`. The main reasoning behind this is that Java and JavaScript have similar designations for boolean values. 

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
    state: 'restarted'
    enabled: true
  become: true
```

**Always use map syntax,** regardless of how many pairs exist in the map.

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

Readability.

## Deprecated options
Use the new `become` syntax when designating that a task needs to be run with `sudo` privileges

```yaml
#bad
- name: template client.json to /etc/sensu/conf.d/
  template:
    dest: '/etc/sensu/conf.d/client.json'
    src: 'client.json.j2'
  sudo: true
 
# good
- name: template client.json to /etc/sensu/conf.d/
  template:
    dest: '/etc/sensu/conf.d/client.json'
    src: 'client.json.j2'
  become: true
  
#bad
- name: using with_items
  ansible.builtin.debug:
    msg: 'The quick brownfox jumps over the lazy {{ item }}
  with_items: '{{ animals }}'
    
- name: using loop
  ansible.builtin.debug:
    msg: 'The quick brown {{ item }} jumps over the lazy dog.'
  loop: '{{ animals }}'
```
### Why?
Using `sudo` was deprecated at [Ansible version 1.9.1](https://docs.ansible.com/ansible/latest/user_guide/become.html).
As of [Ansible 2.5](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html), using `with_items` is not recommended for new development in favor of using `loop` instead.

## Hosts Declaration

`host` sections should follow this general order:

```yaml
# host declaration
# host options in alphabetical order
# pre_tasks
# roles
# tasks

# example
- hosts: 'webservers'
  remote_user: 'centos'
  vars:
    tomcat_state: 'started'
  pre_tasks:
    - name: 'set the timezone to America/Boise'
      lineinfile:
        dest: '/etc/environment'
        line: 'TZ=America/Boise'
        state: 'present'
      become: true
  roles:
    - { role: 'tomcat', tags: 'tomcat' }
  tasks:
    - name: 'start the tomcat service'
      service:
        name: 'tomcat'
        state: '{{ tomcat_state }}'
```

### Why?

A proper definition for how to order these maps produces consistent and easily readable code.

## Task Declaration

A task should be defined in such a way that it follows this general order:

```yaml
# task name
# tags
# task map declaration (e.g. service:)
# task parameters in alphabetical order (remember to always use multi-line map syntax)
# loop operators (e.g. loop)
# task options in alphabetical order (e.g. become, ignore_errors, register)

# example
- name: 'create some ec2 instances'
  tags: 'ec2'
  ec2:
    assign_public_ip: true
    image: 'ami-c7d092f7'
    instance_tags:
      Name: '{{ item }}'
    key_name: 'my_key'
  loop: '{{ instance_names }}'
  ignore_errors: true
  register: ec2_output
  when: ansible_os_family == 'Darwin'
```

### Why?

Similar to the hosts definition, having a well-defined style here helps create consistent code.

## Include Declaration

For `include_*/import_*` statements, make sure to quote filenames and only use blank lines between `include_*/import_*` statements if they are multi-line (e.g. they have tags).

```yaml
# bad
- include_tasks: other_file.yml

- include_tasks: 'second_file.yml'

- include_tasks: third_file.yml tags=third

# good

- include_tasks: 'other_file.yml'
- include_tasks: 'second_file.yml'

- include_tasks: 'third_file.yml'
  tags: 'third'
```

### Why?

This tends to be the most readable way to have `include_*/import_*` statements in your code.

## Spacing

You should have blank lines between two host blocks, between two task blocks, and between host and include blocks. When indenting, you should use 2 spaces to represent sub-maps, and multi-line maps should start with a `-`). For a more in-depth example of how spacing (and other things) should look, consult [style.yml](style.yml).

### Why?

Having a well-defined style here helps create consistent code.

## Variable Names

Use `snake_case` for variable names in your scripts.

```yaml
# bad
- name: 'set some facts'
  set_fact:
    myBoolean: true
    myint: 20
    MY_STRING: 'test'

# good
- name: 'set some facts'
  set_fact:
    my_boolean: true
    my_int: 20
    my_string: 'test'
```

### Why?

Ansible uses `snake_case` for module names so it makes sense to extend this convention to variable names.
