---
title: 'Como adicionar classe CSS na div #app do VueJs 3'
date: 2023-04-16 22:00:00 +0300
categories: [Programação]
tags: [programacao, vuejs, web, javascript]
image:
  path: '/v1681697073/blog/adicionar-css-div-app-vue-3/vue_r4rew8.jpg'
  width: 1000
  height: 400
---

Você já precisou adicionar uma classe CSS naquela div em que a sua aplicação vue é injeta dentro? Sim? Bom, caso você não
esteja usando Vite, por exemplo, ainda pode editar o html da página base que fica dentro do folder "public", não é uma
ótima solução, porém, funciona para casos simples.

A coisa complica quando você precisa fazer isso de maneira dinâmica (conforme o _template_ renderizada) ou mesmo por que
esta usando o Vite e não há mais a pastinha "public" para fazer o ajuste por lá.

Para resolver isso, você pode usar um dos hooks do cíclo de vida do Vue e, através de um simples `querySelector`, aplicar
as classes desejadas, ficando assim:

```vue
<template>
  <main-menu />
  <div class="page-wrapper">
    <message-display />
    <router-view></router-view>
    <page-footer />
  </div>
</template>

<script setup>
import { onMounted } from 'vue'
import MainMenu from '@/components/menu/MainMenu.vue'
import PageFooter from '@/components/page/PageFooter.vue'
import MessageDisplay from '@/components/MessageDisplay.vue'
onMounted(() => {
  document.querySelector('#app').classList.add('page')
})
</script>
```

Simples né? [Aqui](https://github.com/web-budget/front-end/blob/main/src/components/templates/LoginTemplate.vue) você
pode encontrar outro exemplo, todos eles utilizados no [webBudget](https://github.com/web-budget).
