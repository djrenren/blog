---
layout: default
title: "John Renner"
custom_css:
    - about-me
---
{% capture page.head %}
<link rel="stylesheet" type="text/css" href="/assets/css/about-me.css" />
{% endcapture %}

{% capture header %}
    <img class="photo" src="/assets/img/me.jpg"/>
    <div class="info">
        <h1 class="name">
            John Renner
        </h1>
        <p class="contact">
            <a href="mailto:john@jrenner.net" class="subtle"><i class="fa fa-envelope"></i> &nbsp;john@jrenner.net</a> &middot;
            <a href="//github.com/djrenren" class="subtle"><i class="fab fa-github"></i> djrenren</a>
        </p>
    </div>
{% endcapture %}
{% include chunk.html class="about-header" content=header %}

<section>
<h2>About Me</h2>
<p>
    I'm a graduate currently pursuing my PhD in Computer Science at UCSD.
    Much of my work has been focused on programming languages.
    If for any reason you'd like to connect with me, please reach out to me via email.
</p>
</section>

<section>
    <h2>Education</h2>
    <br>
    <div class="education">
    <div class="school">
        <div class="info">
            <img src="/assets/img/ucsdlogo.png" />
            <div class="description">
                PhD Student
            </div>
        </div>
        <div class="dates">
            2017-Now
        </div>
    </div>
    <div class="school">
        <div class="info">
            <img src="/assets/img/ritlogo.png" />
            <div class="description">
                B.S. Software Engineering
            </div>
        </div>
        <div class="dates">
            2017-Now
        </div>
    </div>
    </div>
</section>


<section>
<h2>Projects & Work</h2>
<br>
<div class="work">
    <div class="work-item">
        <img src="/assets/img/vscode.png"/>
        <div class="description">
        <h3><a href="https://github.com/google/kythe/tree/master/kythe/go/languageserver">
            Kythe Language Server
        </a></h3>
        <p>
          Implemented a Language Server capable of providing local cross-references and type information supplied by Kythe’s static index.
          My work was incorporated into the default workstation config at Google.
        </p>
        </div>
    </div>
    <div class="work-item">
        <img src="/assets/img/rustlogo.svg"/>
        <div class="description">
        <h3><a href="https://github.com/google/kythe/tree/master/kythe/rust">
            Rust Indexer for Kythe
        </a></h3>
        <p>
            Designed and built a tool for indexing cross-references in Rust code using the Kythe knowledge graph protocols,
            enabling definition lookups and codesearch.
        </p>
        </div>
    </div>
    <div class="work-item">
        <img src="/assets/img/fb.png"/>
        <div class="description">
        <h3>
            Facebook Cache Monitoring
        </h3>
        <p>
            Created a service for determining and alerting on realtime cache consistency for Facebook’s whole memcache and TAO
            deployment.
        </p>
        </div>
    </div>
</div>
</section>

<section>
    <h2>Publications</h2>
    <div class="publications">
{%- capture bibtex -%}
{% raw %}
@inproceedings{watt:2019:ct-wasm,
  author    = {Conrad Watt and John Renner and Natalie Popescu and Sunjay Cauligi and Deian Stefan},
  title     = {{CT-Wasm}: Type-Driven Secure Cryptography for the Web Ecosystem},
  booktitle = {ACM SIGPLAN Symposium on Principles of Programming Languages (POPL)},
  month     = {January},
  year      = {2019},
  publisher = {ACM}
}
{% endraw %}
{%- endcapture -%}
        {%include publication.html
            src="https://github.com/djrenren/blog/releases/download/papers/ct-wasm-popl-2019.pdf"
            title="CT-Wasm: Type-Driven Secure Cryptography for the Web Ecosystem"
            conference="POPL '19"
            authors="Conrad Watt, John Renner, Natalie Popescu, Sunjay Cauligi, Deian Stefan"
            id=2
            bibtex=bibtex
           %}

{%- capture bibtex -%}
@inproceedings{renner:2018:ct-wasm,
    author    = {John Renner and Sunjay Cauligi and Deian Stefan},
    title     = {Constant-Time {WebAssembly}},
    booktitle = {Principles of Secure Compilation (PriSC)},
    month     = {January},
    year      = {2018},
}
{%- endcapture -%}
        {%include publication.html
            src="https://github.com/djrenren/blog/releases/download/papers/ct-wasm-prisc-2018.pdf"
            title="Constant-time WebAssembly"
            conference="PriSC '18"
            authors="John Renner, Sunjay Cauligi, Deian Stefan"
            bibtex=bibtex
            id=1
           %}

    </div>
</section>
