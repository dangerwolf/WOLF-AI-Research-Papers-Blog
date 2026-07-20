+++
date = '{{ .Date }}'
draft = true
title = '{{ replace .File.ContentBaseName "-" " " | title }}'
tags=[]
categories = []

+++


<script> document.addEventListener('DOMContentLoaded', function() {     const content = document.querySelector('.post-content');     if (!content) return;     const headings = content.querySelectorAll('h2, h3');     if (headings.length === 0) return;      
                                                                   const toc = document.createElement('aside');     toc.id = 'dynamic-toc';     toc.innerHTML = '<div class="toc-title">目录</div><ul></ul>';     const ul = toc.querySelector('ul');      headings.forEach((heading, index) => {         if (!heading.id) heading.id = 'toc-head-' + index;         const li = document.createElement('li');         const a = document.createElement('a');         a.href = '#' + heading.id;         a.textContent = heading.textContent;         li.appendChild(a);         ul.appendChild(li);     });     document.body.appendChild(toc);      
                                                                   const style = document.createElement('style');     style.textContent = `         #dynamic-toc { position: fixed; top: 50px; right: 30px; width: 240px; max-height: 70vh; overflow-y: auto; padding: 15px; background: rgba(45, 55, 72, 0.95); border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.3); font-size: 0.9rem; z-index: 100; font-family: sans-serif; }         #dynamic-toc .toc-title { font-weight: bold; margin-bottom: 10px; color: #00bcd4; border-bottom: 1px solid #4a5568; padding-bottom: 5px; }         #dynamic-toc ul { list-style: none; padding-left: 0; margin: 0; }         #dynamic-toc li { margin: 6px 0; }         #dynamic-toc a { color: #a0aec0; text-decoration: none; display: block; transition: color 0.2s; line-height: 1.4; }         #dynamic-toc a:hover { color: #fff; }         @media screen and (max-width: 1200px) { #dynamic-toc { display: none; } }     `;     document.head.appendChild(style); }); </script>
