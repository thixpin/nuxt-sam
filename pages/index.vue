<template>
  <div class="container mx-auto">
    <h1 class="text-4xl font-bold text-center">Welcome to Nuxt3 <span class="text-blue-400">+ TailwindCSS</span></h1>
    <p class="text-center text-lg mt-4 width-100">
      This is a simple Nuxt3 project with TailwindCSS, that deploy on AWS Lambda using 
      <span class="text-blue-500">AWS SAM</span>.
    </p>
    <p class="text-center text-lg mt-4">
      Here is the article that I wrote about this project: 
      <a href="https://kalaung.org/deploy-nuxtjs-app-to-aws-with-sam" target="_blank" class="text-blue-500">Deploy NuxtJs app to AWS with SAM</a>
    </p>
    <p class="text-center text-lg mt-4">
      You can see the architecture of this project on below.
    </p>
    <div class="flex justify-center mt-4 max-w-3xl mx-auto">
      <img src="/images/infra-architecture.png" alt="AWS SAM Architecture" />
    </div>
  </div>
</template>
