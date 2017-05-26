---
layout: post
title: Barre de progression simpliste en VueJS
modified: 2017-03-23
tags: [vuejs, javascript]
categories: [code]
author: gabrielrobert
---

Ce petit component est très utile pour faire afficher une barre de progression en respectant la [reactivity](https://vuejs.org/v2/guide/reactivity.html)"&nbsp;de VueJS. Le code parle de lui-même, ce n'est même pas nécessaire de le décrire.

P.S Merci à [Udy](https://codepen.io/udyux/) pour le progress natif :)

{% highlight javascript %}
<template>
  <progress
    class="progress"
    :max="max"
    :value="percent"
    
    v-if="percent > 0"
  />
</template>

<script>
// Usage: <ProgressBar :percent="25" />

export default {
  name: 'progress-bar',
  props: {
    percent: {
      type: Number,
      required: false,
      default: 0
    },
    max: {
      type: Number,
      required: false,
      default: 100
    }
  }
}
</script>

<style lang="scss">
@import '~sass/tools';
.progress {
    position: relative;
    width: 100%;
    height: 10px;
    margin: 15px 0;
    color: cc(highlight);
    fill: cc(highlight);
    border: 1px solid cc(bg);
    &::-webkit-progress-bar {
        background-color: cc(bg);
        border: none;
        box-shadow: 0 0 0 1px cc(border) inset;
    }
    &::-webkit-progress-value {
        background-color: cc(highlight);
        border: none;
    }
}
</style>
{% endhighlight %}
