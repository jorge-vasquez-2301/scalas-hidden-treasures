---
info: Slides for presentation at Functional Scala 2024
theme: "default"
presenter: false
download: false
exportFilename: "presentation"
export:
  format: pdf
  timeout: 30000
  dark: auto
  withClicks: true
  withToc: true
highlighter: "shiki"
lineNumbers: true
monaco: false
remoteAssets: true
selectable: true
record: false
colorSchema: "dark"
aspectRatio: "16/9"
canvasWidth: 980
favicon: "functionalScalaIcon.png"
drawings:
  enabled: false
  persist: false
  presenterOnly: false
  syncAll: true

transition: slide-left
background: /web.jpg
---

### Scala's Hidden Treasures
### Five ZIO-Compatible Libraries you didn't know you needed!

<b>December 2024</b>

<div class="absolute top-10 right-16">
  <img src="/functionalScalaLogo.png" class="h-10" />
</div>

---
transition: slide-left
layout: image-right
image: /yo.jpg
class: "flex items-center text-2xl"
---

<div>
  <p>Jorge Vásquez</p>
  <p>Senior Software Engineer</p>
  <p><b>@Ziverge</b></p>
</div>

---
transition: slide-left
layout: image-right
image: ./agenda.jpg
---

## **Agenda**

<div class="mt-4 flex h-3/5 w-full items-center gap-5 text-justify">
  <ul>
    <li v-click>Conceptos básicos de <b>Programación Funcional (PF)</b></li>
    <li v-click>Conceptos básicos de <b>ZIO</b></li>
    <li v-click>Conceptos básicos de <b>ZIO HTTP</b></li>
    <li v-click>Ejemplo práctico: <b>Shopping Cart</b></li> 
  </ul>
</div>

---
transition: slide-left
layout: image-right
image: /summary.jpg
---

## **Summary**

<div class="mt-4 flex h-3/5 w-full items-center">
  <ul>
    <li v-click>La <b>Programación Funcional</b> no es un tema sólo para entornos académicos</li>
    <li v-click>La <b>Programación Funcional</b> permite construir aplicaciones completas en el mundo real</li>
  </ul>
</div>

---
transition: slide-left
layout: image-right
image: /summary.jpg
---

## **Summary**

<div class="mt-4 flex h-4/5 w-full items-center">
  <ul>
    <li v-click><b>ZIO</b> es una librería para Scala que permite construir aplicaciones asíncronas, concurrentes, resilientes, eficientes, fáciles de entender y testear</li>
    <li v-click>Existe todo un <b>ecosistema de librerías</b> alrededor de ZIO para diversas situaciones</li>
    <li v-click><b>ZIO HTTP</b> permite implementar servidores REST, autogenerar clientes, documentación y CLIs</li>
  </ul>
</div>

---
transition: slide-left
layout: image-right
image: /learn.jpg
---

## **Where to learn more**

<div class="flex h-3/5 w-full items-center">
  <ul>
    <li v-click><a href="https://github.com/zio" target="_blank">ZIO en GitHub</a></li>
    <li v-click><a href="https://zio.dev/" target="_blank">Sitio oficial de ZIO</a></li>
    <li v-click><a href="https://www.zionomicon.com/" target="_blank">Zionomicon</a></li>
    <li v-click><a href="https://jorgevasquez.blog/" target="_blank">Mi nuevo blog personal (jorgevasquez.blog)</a></li>
  </ul>
</div>

---
transition: slide-left
layout: image-right
image: /computer.png
---

## **Thank you!**

<div class="grid grid-cols-8 gap-4 items-center h-4/5 content-center text-2xl">
  <div class="col-span-1"><img src="/x.png" class="w-8" /></div> <div class="col-span-7">@jorvasquez2301</div>
  <div class="col-span-1"><img src="/linkedin.png" class="w-8" /></div> <div class="col-span-7">jorge-vasquez-2301</div>
  <div class="col-span-1"><img src="/email.png" class="w-8" /></div> <div class="col-span-7">jorge.vasquez@ziverge.com</div>
</div>