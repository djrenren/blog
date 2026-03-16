---
layout: default
title: "John Renner"
custom_css:
    - about-me
---
{% capture page.head %}
<link rel="stylesheet" type="text/css" href="/assets/css/about-me.css" />
{% endcapture %}

<section>
<p>
    I'm a engineer turned researcher turned back to engineer, now team lead.
    My research has focused on using programming language techniques to solve
    security problems.
</p>
<p>
    <b>Email:</b> {{site.email}}
</p>
</section>

<section>
    <h2 class="section">Education</h2>
    <br>
    <div class="education">
    <div class="school">
        <div class="info">
            <img src="/assets/img/ucsdlogo.png" />
            <div class="description">
                PhD. Computer Science
            </div>
        </div>
        <div class="dates">
            2017-2022
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
            2012-2017
        </div>
    </div>
    </div>
</section>

<section class="employment">
    <h2 class="section">Full-time Employment</h2>
    <div class="job"> 
        <img src="/assets/img/cubist.svg" alt="Cubist, Inc">
        <div class="title">
            <h3>
                Founding Engineer
                <span class="company">&nbsp;@ Cubist</span>
            </h3>
            <span class="time">2022 - Current</span>
        </div>
        <div class="description">
            Developed white-label auth system including OIDC and Passkey stacks from spec in Rust. Organized the first team of junior engineers and led the development of a
            visual editor for policy-enforcing programs.
        </div>
    </div>
</section>

<section>
    <h2 class="section">Publications</h2>
    <div class="publications">
{% for pub in site.data.publications %}
{% include publication.html 
    src=pub.src
    title=pub.title
    conference=pub.conference
    authors=pub.authors
    id=forloop.index
    bibtex=pub.bibtex
%}
{% endfor %}
    </div>
</section>
<section>
<h2 class="section">Internships & Projects</h2>
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

