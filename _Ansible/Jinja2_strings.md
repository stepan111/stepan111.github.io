---
layout: post
title:  "Jinja2 processing inside strings"
categories: Ansible
---


Sometimes you have complex structures to process within your playbook. And writing your own filter will be to time consuming.
In this we can use jinja2 templates to process almost anything within playbook. Here are example.

We can use jinja2 inside ansible strings:

```
- name: run some command                     
   command: echo item {{item}}
   register: my_result                       
   with_items:
     - 1           
     - 2                                                 
     - 3                                                                                                                                
  - name: my debug
    debug:
      msg: "{{ "{% set output = [] " }}%}\
        {{ "{% for x in (my_result.results | map(attribute='stdout')) " }}%}\
          {{ "{{ output.append( 'The result was: ' ~ x ) " }}}}\            
        {{ "{% endfor " }}%}\                                                                                                                   
        {{ "{{output " }}}}"  

```

Output will be:
```
ok: [host1]: => {           
    "msg": [    
        "the result was: item 1",                                                                                                       
        "the result was: item 2",                                  
        "the result was: item 3"
    ]                                                                                                                                   
}
ok: [host2]: => {                   
    "msg": [                             
        "the result was: item 1",
        "the result was: item 2",              
        "the result was: item 3"
    ]                     
}   

```

True/False check :
```
 - { role: revitas-flexbi,   cognos_version: "{{ "{{ cognosVersion  " }}}}",
      BIlicensedCustomer: "{{ "{{ True if 'analyzer' in subscription else False  " }}}}",
      tags: ['ecmbi'] }

```

create dict inside jinja2:
```
       recovery_keys: "{{ "{% set output = dict()  " }}%}
                {{ "{% for line in init_out.stdout_lines - " }}%}
                  {{ "{% if line.find('Recovery Key') != -1 - " }}%}
                    {{ "{% set index =  output | length  " }}%}
                    {{ "{% set _ = output.update( { 'recovery_key_' + index | string  : line.split(':')[1] | trim } )   " }}%}
                  {{ "{%- endif  " }}%}
                {{ "{%- endfor  " }}%}
                {{ "{{ output  " }}}}"

```

Set fact for all hosts in group:

```
      - name: Set hugememory value
          set_fact:
            rac_hugememory: "{{ "{{ hugememory.stdout " }}}}"

    - set_fact: rac_hugememories="{{ "{{ rac_hugememories | default([]) + [ hostvars[item]['rac_hugememory'] | int ] " }}}}"
      with_items: "{{ "{{ groups['ec2Oracle'] " }}}}"

    - debug: msg="mem={{ "{{ rac_hugememories " }}}}"

```
