---
layout: post
title: "How to apply the Apache 2.0 License to your Open Source software project"
description: "Apache 2.0 License is one of the best licenses to use in your Open Source software project, but do you actually know how to properly apply it?"
date: 2021-10-02 15:15:00 +0300
image:
  path: assets/img/blog/apache-logo.svg
  width: 250
  height: 250
  caption: This image was used from Apache Software Foundation press kit (TM)
---

<p class="lead">
  I've always been a <span class="fst-italic">3-Clause BSD License</span> (AKA <i>New BSD License</i> or <span class="fst-italic">Modified BSD License</span>) guy.
  As I've matured, my understanding of Open Source licensing has grown as well, and so have my requirements. When I start a new Open Source
  project now, I just use <strong>Apache 2.0 license</strong>. I want to point out that I am not a lawyer, and I am certainly not your lawyer.
  My legal opinions are based on research and lengthy conversations with legal departments of companies that I worked for.  
</p>

### Why Apache 2.0 license?

There are various reasons why I prefer Apache 2.0 license over the others (specifically the 3-Clause BSD License).
This [WhiteSource article](https://www.whitesourcesoftware.com/resources/blog/top-10-apache-license-questions-answered/#7_What_is_the_difference_between_Apache_License_20_and_BSD)
covers the differences nicely, but for completeness, let me mentioned them here explicitly as well:

<dl class="row">
  <dt class="col-sm-3">Explicit grant of patent rights</dt>
  <dd class="col-sm-9">Apache License 2.0 explicitly lays down the grant of patent rights while using, modifying, or distributing Apache licensed software; it also lists the circumstances when such grant gets withdrawn.</dd>

  <dt class="col-sm-3">Precise definitions of the used concepts</dt>
  <dd class="col-sm-9">Apache License 2.0 explicitly defines all the terms and concepts that it uses. This leaves little scope for ambiguity.</dd> 

  <dt class="col-sm-3">Reusable without rewording</dt>
  <dd class="col-sm-9">
    Apache License 2.0 can be easily used by other projects without any rewording in the license document itself.
    It also means that it should be sufficient to provide the canonical URL <code>https://www.apache.org/licenses/LICENSE-2.0.txt</code>, where
    the copy of the license can be obtained, instead of the license text.
  </dd>
</dl>

These differences are most important to me, but other differences like _"prominent notices stating that You changed the files"_
might be more critical for you. Here is the full text of the Apache 2.0 license:

{% highlight text linenos %}
                                Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

1. Definitions.

   "License" shall mean the terms and conditions for use, reproduction,
   and distribution as defined by Sections 1 through 9 of this document.

   "Licensor" shall mean the copyright owner or entity authorized by
   the copyright owner that is granting the License.

   "Legal Entity" shall mean the union of the acting entity and all
   other entities that control, are controlled by, or are under common
   control with that entity. For the purposes of this definition,
   "control" means (i) the power, direct or indirect, to cause the
   direction or management of such entity, whether by contract or
   otherwise, or (ii) ownership of fifty percent (50%) or more of the
   outstanding shares, or (iii) beneficial ownership of such entity.

   "You" (or "Your") shall mean an individual or Legal Entity
   exercising permissions granted by this License.

   "Source" form shall mean the preferred form for making modifications,
   including but not limited to software source code, documentation
   source, and configuration files.

   "Object" form shall mean any form resulting from mechanical
   transformation or translation of a Source form, including but
   not limited to compiled object code, generated documentation,
   and conversions to other media types.

   "Work" shall mean the work of authorship, whether in Source or
   Object form, made available under the License, as indicated by a
   copyright notice that is included in or attached to the work
   (an example is provided in the Appendix below).

   "Derivative Works" shall mean any work, whether in Source or Object
   form, that is based on (or derived from) the Work and for which the
   editorial revisions, annotations, elaborations, or other modifications
   represent, as a whole, an original work of authorship. For the purposes
   of this License, Derivative Works shall not include works that remain
   separable from, or merely link (or bind by name) to the interfaces of,
   the Work and Derivative Works thereof.

   "Contribution" shall mean any work of authorship, including
   the original version of the Work and any modifications or additions
   to that Work or Derivative Works thereof, that is intentionally
   submitted to Licensor for inclusion in the Work by the copyright owner
   or by an individual or Legal Entity authorized to submit on behalf of
   the copyright owner. For the purposes of this definition, "submitted"
   means any form of electronic, verbal, or written communication sent
   to the Licensor or its representatives, including but not limited to
   communication on electronic mailing lists, source code control systems,
   and issue tracking systems that are managed by, or on behalf of, the
   Licensor for the purpose of discussing and improving the Work, but
   excluding communication that is conspicuously marked or otherwise
   designated in writing by the copyright owner as "Not a Contribution."

   "Contributor" shall mean Licensor and any individual or Legal Entity
   on behalf of whom a Contribution has been received by Licensor and
   subsequently incorporated within the Work.

2. Grant of Copyright License. Subject to the terms and conditions of
   this License, each Contributor hereby grants to You a perpetual,
   worldwide, non-exclusive, no-charge, royalty-free, irrevocable
   copyright license to reproduce, prepare Derivative Works of,
   publicly display, publicly perform, sublicense, and distribute the
   Work and such Derivative Works in Source or Object form.

3. Grant of Patent License. Subject to the terms and conditions of
   this License, each Contributor hereby grants to You a perpetual,
   worldwide, non-exclusive, no-charge, royalty-free, irrevocable
   (except as stated in this section) patent license to make, have made,
   use, offer to sell, sell, import, and otherwise transfer the Work,
   where such license applies only to those patent claims licensable
   by such Contributor that are necessarily infringed by their
   Contribution(s) alone or by combination of their Contribution(s)
   with the Work to which such Contribution(s) was submitted. If You
   institute patent litigation against any entity (including a
   cross-claim or counterclaim in a lawsuit) alleging that the Work
   or a Contribution incorporated within the Work constitutes direct
   or contributory patent infringement, then any patent licenses
   granted to You under this License for that Work shall terminate
   as of the date such litigation is filed.

4. Redistribution. You may reproduce and distribute copies of the
   Work or Derivative Works thereof in any medium, with or without
   modifications, and in Source or Object form, provided that You
   meet the following conditions:

   (a) You must give any other recipients of the Work or
   Derivative Works a copy of this License; and

   (b) You must cause any modified files to carry prominent notices
   stating that You changed the files; and

   (c) You must retain, in the Source form of any Derivative Works
   that You distribute, all copyright, patent, trademark, and
   attribution notices from the Source form of the Work,
   excluding those notices that do not pertain to any part of
   the Derivative Works; and

   (d) If the Work includes a "NOTICE" text file as part of its
   distribution, then any Derivative Works that You distribute must
   include a readable copy of the attribution notices contained
   within such NOTICE file, excluding those notices that do not
   pertain to any part of the Derivative Works, in at least one
   of the following places: within a NOTICE text file distributed
   as part of the Derivative Works; within the Source form or
   documentation, if provided along with the Derivative Works; or,
   within a display generated by the Derivative Works, if and
   wherever such third-party notices normally appear. The contents
   of the NOTICE file are for informational purposes only and
   do not modify the License. You may add Your own attribution
   notices within Derivative Works that You distribute, alongside
   or as an addendum to the NOTICE text from the Work, provided
   that such additional attribution notices cannot be construed
   as modifying the License.

   You may add Your own copyright statement to Your modifications and
   may provide additional or different license terms and conditions
   for use, reproduction, or distribution of Your modifications, or
   for any such Derivative Works as a whole, provided Your use,
   reproduction, and distribution of the Work otherwise complies with
   the conditions stated in this License.

5. Submission of Contributions. Unless You explicitly state otherwise,
   any Contribution intentionally submitted for inclusion in the Work
   by You to the Licensor shall be under the terms and conditions of
   this License, without any additional terms or conditions.
   Notwithstanding the above, nothing herein shall supersede or modify
   the terms of any separate license agreement you may have executed
   with Licensor regarding such Contributions.

6. Trademarks. This License does not grant permission to use the trade
   names, trademarks, service marks, or product names of the Licensor,
   except as required for reasonable and customary use in describing the
   origin of the Work and reproducing the content of the NOTICE file.

7. Disclaimer of Warranty. Unless required by applicable law or
   agreed to in writing, Licensor provides the Work (and each
   Contributor provides its Contributions) on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
   implied, including, without limitation, any warranties or conditions
   of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
   PARTICULAR PURPOSE. You are solely responsible for determining the
   appropriateness of using or redistributing the Work and assume any
   risks associated with Your exercise of permissions under this License.

8. Limitation of Liability. In no event and under no legal theory,
   whether in tort (including negligence), contract, or otherwise,
   unless required by applicable law (such as deliberate and grossly
   negligent acts) or agreed to in writing, shall any Contributor be
   liable to You for damages, including any direct, indirect, special,
   incidental, or consequential damages of any character arising as a
   result of this License or out of the use or inability to use the
   Work (including but not limited to damages for loss of goodwill,
   work stoppage, computer failure or malfunction, or any and all
   other commercial damages or losses), even if such Contributor
   has been advised of the possibility of such damages.

9. Accepting Warranty or Additional Liability. While redistributing
   the Work or Derivative Works thereof, You may choose to offer,
   and charge a fee for, acceptance of support, warranty, indemnity,
   or other liability obligations and/or rights consistent with this
   License. However, in accepting such obligations, You may act only
   on Your own behalf and on Your sole responsibility, not on behalf
   of any other Contributor, and only if You agree to indemnify,
   defend, and hold each Contributor harmless for any liability
   incurred by, or claims asserted against, such Contributor by reason
   of your accepting any such warranty or additional liability.

END OF TERMS AND CONDITIONS

APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. We also recommend that a
      file or class name and description of purpose be included on the
      same "printed page" as the copyright notice for easier
      identification within third-party archives.

Copyright [yyyy] [name of copyright owner]

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
{% endhighlight %}

There are two variants of the Apache 2.0 license: justified version and version with text aligned to the left.
The justified version is the one you get by fetching `https://www.apache.org/licenses/LICENSE-2.0.txt` - I call it the **canonical form**.
The license has 201 lines of text and is composed of the header (lines 1-3),
the terms and conditions (lines 5-176) and the appendix (lines 178-201).


The license has 201 lines of text, and it's in what I call the **canonical form**.
There are two variants of the Apache 2.0 license: justified version and
version with text aligned to the left. As the justified version is the one you get by fetching
`https://www.apache.org/licenses/LICENSE-2.0.txt`, I consider it the canonical one.

### License

To apply the license to your software project, create a `LICENSE` file in the source tree's top level.
Now copy the full Apache 2.0 License text from [https://www.apache.org/licenses/LICENSE-2.0.txt](https://www.apache.org/licenses/LICENSE-2.0.txt) into a `LICENSE` file.
Notice that the `LICENSE` file doesn't have any extension, but you can optionally name the file `LICENSE.txt` as well.

**Do not modify** the license text. If you used a 3-Clause BSD License in the past,
you usually downloaded the license template, changed it, and saved it as a `LICENSE` file.

{% highlight text %}
Copyright <YEAR> <COPYRIGHT HOLDER>

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.
{% endhighlight %}

You may be tempted to do the same with Apache 2.0 License. The license text contains what looks like
a copyright notice on line 189: 

{% highlight text %}
Copyright [yyyy] [name of copyright owner]
{% endhighlight %}

Actually, this copyright notice template and an additional boilerplate are part of the Apache 2.0 License appendix, which starts
at line 178 and explains how to apply the Apache License to individual source files instead
of having one single `LICENSE` file in the top level of the source tree. The text in the appendix is
very explicit about what the copyright notice template and additional boilerplate are for (use in source files). Changing
this copyright notice template directly in license text is IMHO an improper application of the license.
The whole appendix (lines 178-201) can technically be omitted from the license, as it's outside the scope of "Section
1 through 9", which forms the terms and conditions of the license.

My opinion is backed by the "**Reusable without rewording**" attribute of the Apache 2.0 License
I mentioned before. It's even backed by my experience in a big corporation - two years ago, I was working for one of the biggest tech companies in the world.
The company had established an Open Source office with a Legal Department dealing with use of Open Source within the company.
This Open Source office stipulated that for an Open Source project to be licensed under Apache 2.0 License, it is not required
to have an explicit Apache 2.0 License text as part of the project. 
It is absolutely sufficient if the Open Source project README file
contains the name of the license, either "Apache 2.0 License" or its [SPDX identifier](https://spdx.org/licenses/Apache-2.0.html).
Ideally, (but not required) canonical URL where the license can be obtained should be mentioned as well: `https://www.apache.org/licenses/LICENSE-2.0.txt`.
This is just a legal opinion of one big tech company. Don't take this legal opinion as implied truth.

### Copyright notice

In the previous section, we've stipulated that the Apache 2.0 License text doesn't actually have a place where to
put an explicit copyright notice. Since 1989, [Berne Convention](https://www.wipo.int/treaties/en/ip/berne/) established that the copyright is automatic,
and the use of copyright notices to claim the copyright has become optional.

This is how an **explicit copyright notice** looks like:

{% highlight text %}
Copyright 2021 Vladimír Gorej <vladimir.gorej@gmail.com>
{% endhighlight %}

When an explicit copyright notice is missing in Open Source software that you want to use in your project,
you're theoretically still OK, as it is possible to compile one using the following assumptions:

1. The name of the copyright holder is the name of the first committer to the project
2. The copyright holder contact is established by using the email address of the first committer
3. The year of the copyright notice is established as the year of the first commit to the project

If you cannot somehow obtain the above information, you're kind of screwed, as there is no way to 
establish the copyright notice for compliance purposes in big tech companies.

It's always a good idea to provide an explicit copyright notice when applying the Apache 2.0 License. But, where do we put one?
Well, if we would read the Apache 2.0 Licence terms and conditions carefully, **section 4.d)** (line 106) has an answer for that: **a "NOTICE" text file**.
Just create a new file called `NOTICE` and put your copyright notice there. 

Today there are three competing formats of copyright notices: 

<dl class="row">
  <dt class="col-sm-3">Historical format</dt>
  <dd class="col-sm-9">
    Copyright 2017-2019, Vladimír Gorej<br />
    Copyright 1998, Linus Torvalds
  </dd>

  <dt class="col-sm-3">Newer format</dt>
  <dd class="col-sm-9">
    Copyright The Ramda Adjunct contributors<br />
    Copyright The Kubernetes Authors
  </dd>

  <dt class="col-sm-3">Hybrid format</dt>
  <dd class="col-sm-9">
    Copyright 2017-2019 Vladimír Gorej and the Ramda Adjunct contributors
  </dd>
</dl>

Given above, your `NOTICE` file could look like this:

{% highlight text %}
Ramda Adjunct
Copyright 2017, Vladimír Gorej
{% endhighlight %}

### Closing words

Choose the appropriate Open Source license for the needs of your project.
If you're deciding between BSD, MIT, and Apache 2.0, go for Apache 2.0 License. The license
automatically grants patent rights. It's clear, explicit, and reusable without rewording. 
To apply the license to your Open Source software project, create two files: `LICENSE` and `NOTICE`
in the top level of the source tree. The license text that you fetched from `https://www.apache.org/licenses/LICENSE-2.0.txt`
goes into the `LICENSE` file. Your copyright notice goes into the `NOTICE` file. Doing these
two things makes the application of the Apache 2.0 License as explicit and clear as it can be. Sounds pretty easy, right?