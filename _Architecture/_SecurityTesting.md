---
layout: post
title:  "Security Testing"
categories: Architecture
---


It is a common tendency to implement Shift-Left on Security practice and run tests before apps deployed to production. I think that security in depth is main concept here. And it add complexity. Quite often a lot.
In any case when you implement new security checks, person/persons who own a code will get few more hours/days/weeks/years of additional work. Shift left on security should hide those hours by providing faster feedbacks to persons who own code. So just to conclude, your IaC should be in mature state when you implement new security monitors because people should have enough time to check appeared issues.

Anyway, here I would like to list opensource(and not realy) projects to harden your environment and way to inject it SDLC. Also I would discuss how it should be but not how it is right now and some realizations may contain technical difficulties.
---

So let's discuss when we can run security monitor:
* Right after image build (CI)
* During automated testing(CD)
* On image push into registry (like harbor does)
* As admission webhook in kubernetes
* As scheduled activity
* Continuously run in production as it exists


| tool | type | when to run it  | 4C |
-----------------------------------------
| kubescape | static analysis | on cluster creation/scheduled | cluster|
| trivy | static analysis | during CI/as admission webhook/scheduled|container|
| falco| dynamic analysis | During automated tests(acceptance)/in prod during whole lifecycle| cluster |
|AppArmor/seccomp| dynamic analysis | During automated tests(acceptance)/in prod during whole lifecycle| cluster |
| gatekeeper|-|During automated tests(acceptance)/in prod during whole lifecycle | cluster |

It is crucial to organize all tools alert into same place(SIEM).
