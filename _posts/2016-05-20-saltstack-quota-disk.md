---
layout:   post
title:    Manage quota disks with SaltStack
date:     2016-05-20
summary:  I created a SaltStack instructions for managing groups and their respective quotas. 
tags:     devops
---

I have a few servers where the root partition (OS) has the home
directory. This is a problem when users move and/or create new data
and the OS partition loses free space. And I mean all free space.

I created a SaltStack instructions for managing groups and their
respective quotas. 

This instructions were made following two ideas:

* First, groups over single users because it's more easy for managing.
* Second, this configures blocks and not inodes.

### SaltStack states

Keep in mind that state {% raw %}`quota_assign_{{ group }}`{% endraw %} accepts "*none*" as
valid argument when nobody has as quota's primary group. 

{% raw %}
    quota:
      pkg.installed

    quota_remount:
      mount.mounted:
        - name: /
        - device: {{ salt['cmd.run']('sed -n \'/[^#] \/ /s/ .*//p\' /etc/fstab') }}
        - fstype: ext4
        - opts:
          - errors=remount-ro
          - grpquota
        - dump: 0
        - pass_num: 1

    quota_check:
      cmd.run:
        - name: quotacheck -vgum /
        - creates: /aquota.group
        - require:
          - mount: quota_remount

    quota_on:
      quota.mode:
        - name: /
        - mode: on
        - quotatype: group
        - require:
          - cmd: quota_check

    {% for group, args in pillar['quota'].iteritems() %}
    {% if 'softblock' in args %}{% set softblock = args['softblock'] %}{% else %}{% set softblock = '0' %}{% endif %}
    {% if 'hardblock' in args %}{% set hardblock = args['hardblock'] %}{% else %}{% set hardblock = '0' %}{% endif %}
    quota_assign_{{ group }}:
      cmd.run:
        - unless: quota -g {{ group }} | grep -q '{{ softblock }} {{ hardblock }}\|none'
        - name: setquota -g {{ group }} {{ softblock }} {{ hardblock }} 0 0 /
    {% endfor %}
{% endraw %}

### SaltStack pillar

You should define in `/srv/pillar` usergroups and their limits (in KB), in the
following example I configured two groups

{% raw %}
    quota:
      labs:
        hardblock: 2048
      devs:
        softblock: 2048
        hardblock: 2560
{% endraw %}

### Tips: using quotas for other paths

Replace `quota_remount` and `quota_check` states by

{% raw %}
    {% set mountpoint = salt['pillar.get']('mountpoint:device', '/') %}

    quota_fstab:
      cmd.run:
        - unless: grep ' {{ mountpoint }} ' /etc/fstab | grep -q grpquota
        - name: awk '/[^#] \{{ mountpoint }} /{ sub(/$/, ",grpquota", $4) }1' /etc/fstab > /tmp/fstab && mv /tmp/fstab /etc/fstab
        - require:
          - pkg: quota

    quota_remount:
      cmd.run:
        - unless:  grep ' {{ mountpoint }} ' /proc/mounts | grep -q quota 
        - name: mount -o remount {{ mountpoint }}
        - require:
          - cmd: quota_fstab

    quota_check:
      cmd.run:
        - name: quotacheck -vgum {{ mountpoint }}
        - creates: {{ mountpoint }}/aquota.group
        - require:
          - cmd: quota_remount
{% endraw %}