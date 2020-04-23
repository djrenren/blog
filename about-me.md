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
    I'm a graduate currently pursuing my PhD in Computer Science at UCSD.
    My work uses programming language techniques to solve security problems.
</p>
<p>
    <b>Email:</b> john@jrenner.net
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
<h2 class="section">Projects & Work</h2>
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

