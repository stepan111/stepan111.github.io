
---
layout: post
title:  "Tips for writing infrastructure tests"
categories: Ansible
---

When you writing your tests it is better to remember that fastest test feedback will save your time. Based on this it is great to instruct your playbooks with smoke tests that will validate state of deployed component. And what could provide faster feedback that ansible's [failed_when](https://docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html#defining-failure) and [until](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#retrying-a-task-until-a-condition-is-met).

So here are example how to validate state of docker container  :

```
    - name: Validate that envoy container running
      shell: |
        sleep 10
        docker inspect --format="{{ '{{' }}.State.Status{{ '}}' }}" envoy
      changed_when: False
      retries: 5
      delay: 5
      register: inspect_status
      until: inspect_status.stdout.find("running") != -1


```
Running [this kind of tests](https://docs.ansible.com/ansible/latest/reference_appendices/test_strategies.html#modules-that-are-useful-for-testing) for each component is crucial for delivering consistent infrastructure.
In other words you need to automate smoke tests execution within ansible playbooks for each component.
