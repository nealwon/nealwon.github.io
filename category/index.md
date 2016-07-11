---
layout: default
title: site.category
---

<script>
get = new Object;
$(function(){
var spli = location.href.split('?');
if (spli.length>1) {
    console.log(spli[1])
    gets = spli[1].split('&');
    for(i=0;i<gets.length;i++) {
        g = gets[i].split('=');
        get[g[0]] = g[1];
    }
}
console.log(get)
})
</script>
