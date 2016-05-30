---
layout:   post
title:    Manage quota disks with SaltStack
date:     2016-05-20
summary:  I created a SaltStack instructions for managing users and their respective quotas. 
tags:     devops, sysadmin
---

I have a few servers where the root partition (OS) has the home
directory. This is a problem when users move and/or create new data
and the OS partition loses free space. And I mean all free space.

I created a SaltStack instructions for managing users and their
respective quotas.

This instructions configure blocks (hard and soft) and not inodes.

I use SaltStack >= 2015.8.3 (Beryllium).

### SaltStack states

Keep in mind that state {% raw %}`quota_assign_{{ user }}`{% endraw %}
returns "Clean" for users that don't exist on the host.

{% raw %}
    quota:
      pkg.installed

    quota_remount:
      mount.mounted:
        - name: /
        - device: {{ salt['cmd.run']('sed -n "/[^#][[:space:]]\/[[:space:]]/s/[[:space:]].*//p" /etc/fstab') }}
        - fstype: ext4
        - opts:
          - errors=remount-ro
          - usrjquota=aquota.user
          - jqfmt=vfsv1
        - dump: 0
        - pass_num: 1
    
    quota_check:
      cmd.run:
        - name: quotacheck -vgum /
        - creates: /aquota.user
        - require:
          - mount: quota_remount

    quota_on:
      quota.mode:
        - name: /
        - mode: on
        - quotatype: user
        - require:
          - cmd: quota_check

    {% for user, args in pillar.get('qusers', {}).iteritems() %}
    quota_assign_{{ user }}:
      cmd.run:
        - unless: if id {{ user }} > /dev/null ; then quota -u {{ user }} --hide-device | awk '{ print $2, $3}' | grep -q '{{ args['soft'] }} {{ args['hard'] }}' ; fi
        - name: setquota -u {{ user }} {{ args['soft'] }} {{ args['hard'] }} 0 0 /
    {% endfor %}
{% endraw %}

### SaltStack pillar

You should define in `/srv/pillar` users and their limits in KB, ie for
converting 20 GB to KB

```
GB=20 ; echo $(( ${GB}*1024*1024 ))
```

A pillar example for users with different limits

{% raw %}
    qusers:
      dev00:
        soft: 15728640
	hard: 20971520
      dev01:
        soft: 26214400
	hard: 31457280
      labs00:
        soft: 5242880
        hard: 7340032
{% endraw %}

Or in the following example I configured three groups

{% raw %}
    {% set devs = ['dev00','dev01','dev02'] %}
    {% set labs = ['labs00','labs01'] %}
    
    qusers:
      # devs
      {% for user in devs %}
      {{ user }}:
        soft: 15728640
        hard: 20971520
      {% endfor %}
      # labs
      {% for user in labs %}
      {{ user }}:
        soft: 5242880
        hard: 7340032
      {% endfor %}
{% endraw %}
