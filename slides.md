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
background: /treasure.jpg
---

### Scala's Hidden Treasures
### Five ZIO-Compatible Libraries you didn't know you needed!

<b>December 2024</b>

<div class="absolute top-10 right-16">
  <img src="/functionalScalaLogoWhite.png" class="h-10" />
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

<div class="mt-4 flex h-4/5 w-full items-center gap-5 text-justify">
  <ul>
    <li v-click>Stream-based CSV processing with <b>Spata</b></li>
    <li v-click>Type-safe processing of Parquet files with <b>ZIO Apache Parquet</b></li>
    <li v-click>YAML processing with <b>YAML4s</b></li>
    <li v-click>Integration between Doobie and ZIO with <b>Tranzactio</b></li>
    <li v-click>High-performance embedded key-value stores with <b>ZIO LMDB</b></li> 
  </ul>
</div>

---
transition: slide-left
layout: cover
image: csv.jpg
---

## Stream-based CSV processing with **Spata**

---
transition: slide-left
layout: default
---

## Stream-based CSV processing with **Spata**

<div class="mt-4 flex h-4/5 w-full items-center gap-5 text-justify">
  <ul>
    <li v-click>Spata is a <b>functional CSV processor</b></li>
    <li v-click>Offers a <b>stream-based API</b>, backed by FS2</li>
    <li v-click>Supports <b>Scala 3</b></li>
    <li v-click>Allows <b>easy conversion</b> between CSV rows and case classes</li>
    <li v-click>Provides <b>precise information</b> about possible flaws and their location in source data</li>
  </ul>
</div>

---
transition: slide-left
layout: default
---

## **Example:** Chemical Elements CSV

<div class="flex h-4/5 w-full items-center">
  <table>
      <thead>
          <tr>
              <th><b>element</b></th>
              <th><b>symbol</b></th>
              <th><b>melting temperature [F]</b></th>
              <th><b>boiling temperature [F]</b></th>
          </tr>
      </thead>
      <tbody>
          <tr>
              <td>hydrogen</td>
              <td>H</td>
              <td>13.99</td>
              <td>20.271</td>
          </tr>
          <tr>
              <td>helium</td>
              <td>He</td>
              <td>0.95</td>
              <td>4.222</td>
          </tr>
          <tr>
              <td>lithium</td>
              <td>Li</td>
              <td>453.65</td>
              <td>1603</td>
          </tr>
      </tbody>
  </table>
</div>

---
transition: slide-left
layout: default
---

## **Example:** Chemical Elements CSV

<div class="flex h-4/5 w-full items-center">
```csv {all} {maxHeight:'400px'}
element,symbol,melting temperature [F],boiling temperature [F]
hydrogen,H,13.99,20.271
helium,He,0.95,4.222
lithium,Li,453.65,1603
```
</div>

---
transition: slide-left
layout: image-right
image: /todo.jpg
---

### **Example:** Chemical Elements CSV

<div class="mt-4 flex flex-col h-4/5 w-full justify-center gap-5 text-justify">
  <p class="text-xl">Steps we will follow:</p>
  <ul>
    <li v-click>Domain modelling</li>
    <li v-click>Define a <code>CSVParser</code></li>
    <li v-click>Define a <code>CSVRenderer</code></li>
    <li v-click>Read CSV using streams</li>
    <li v-click>Process CSV records</li>
    <li v-click>Write processed CSV</li>
  </ul>
</div>

---
transition: slide-left
layout: default
---

## Domain modelling

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3|4|5|6|8|10-14|16-17|17}{maxHeight:'400px'}
//> using jvm graalvm-java23:23.0.0
//> using scala 3.5.2
//> using dep dev.zio::zio-interop-cats:23.1.0.3
//> using dep info.fingo::spata:3.2.1
//> using dep dev.zio::zio-schema:1.5.0
//> using dep dev.zio::zio-schema-derivation:1.5.0

import zio.schema.*

final case class Element(element: String, symbol: String, meltingTemp: Double, boilingTemp: Double):
  self =>

  def updateTemps(f: Double => Double) =
    self.copy(meltingTemp = f(self.meltingTemp), boilingTemp = f(self.boilingTemp))

object Element:
  given Schema.Record[Element] = DeriveSchema.gen[Element]

```
</div>

---
transition: slide-left
layout: default
---

## Define a `CSVParser`

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3|5-10}{maxHeight:'400px'}
import info.fingo.spata.CSVParser
import zio.*
import zio.interop.catz.*

// Default CSVParser, according to RFC 4180
// Delimiter: Comma (,)
// Quote character: Double quote (")
// The first line of the file is considered the header row
// Headers will be mapped to case class fields
val csvParserWithDefaults: CSVParser[Task] = CSVParser[Task]
```
</div>

---
transition: slide-left
layout: default
---

## Define a `CSVParser`

<div class="flex h-4/5 w-full items-center">
```scala {5-10|6|7|8|9|12-15|13|14|15|17}{maxHeight:'400px'}
import info.fingo.spata.{CSVParser, Record}
import zio.*
import zio.interop.catz.*

final case class Element(
  element: String,      // Header: element
  symbol: String,       // Header: symbol
  meltingTemp: Double,  // Header: melting temperature [F]
  boilingTemp: Double   // Header: boiling temperature [F]
)

val csvParser: CSVParser[Task] =
  CSVParser.config
    .mapHeader(Map("melting temperature [F]" -> "meltingTemp", "boiling temperature [F]" -> "boilingTemp"))
    .parser[Task]

val csvParserPipe: fs2.Pipe[Task, Char, Record] = csvParser.parse
```
</div>

---
transition: slide-left
layout: default
---

## Define a `CSVRenderer`

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3|5-10}{maxHeight:'400px'}
import info.fingo.spata.CSVRenderer
import zio.*
import zio.interop.catz.*

// Default CSVRenderer, according to RFC 4180
// Delimiter: Comma (,)
// Quote character: Double quote (")
// The rendered CSV will contain a header row
// Case class fields will be mapped to headers
val csvRendererWithDefaults: CSVRenderer[Task] = CSVRenderer[Task]
```
</div>

---
transition: slide-left
layout: default
---

## Define a `CSVRenderer`

<div class="flex h-4/5 w-full items-center">
```scala {5-10|6|7|8|9|12-16|13|14|15|16|18}{maxHeight:'400px'}
import info.fingo.spata.{CSVParser, Record}
import zio.*
import zio.interop.catz.*

final case class Element(
  element: String,      // Header: element
  symbol: String,       // Header: symbol
  meltingTemp: Double,  // Header: melting temperature [F]
  boilingTemp: Double   // Header: boiling temperature [F]
)

val csvRenderer: CSVRenderer[Task] =
  CSVRenderer.config
    .fieldDelimiter(';')
    .mapHeader(Map("meltingTemp" -> "melting temperature [C]", "boilingTemp" -> "boiling temperature [C]"))
    .renderer[Task]

val csvRendererPipe: fs2.Pipe[Task, Record, Char] = csvRenderer.render
```
</div>

---
transition: slide-left
layout: default
---

## Process CSV

<div class="flex h-full w-full items-center">
```scala {1-7|9|11-16|18-19|21-33|24-25|26|27|28|22,29|30|31|32|33|35-37|37}{maxHeight:'400px'}
import java.nio.file.Paths
import zio.*
import zio.interop.catz.*
import zio.stream.interop.fs2z.*
import info.fingo.spata.{ CSVParser, CSVRenderer, Record }
import info.fingo.spata.io.{ Reader, Writer }
import zio.schema.*

object SpataExample extends ZIOAppDefault:

  ...

  val csvParserPipe: fs2.Pipe[Task, Char, Record] = ...

  ...
  val csvRendererPipe: fs2.Pipe[Task, Record, Char] = ...

  val fahrenheitCSV = "testdata/elements-fahrenheit.csv"
  val celsiusCSV    = "testdata/elements-celsius.csv"

  val processor: fs2.Stream[Task, Unit] =
    def fahrenheitToCelsius(f: Double): Double = (f - 32.0) * (5.0 / 9.0)

    Reader[Task]
      .read(Paths.get(fahrenheitCSV))
      .through(csvParserPipe)
      .toZStream()
      .mapZIO(record => ZIO.fromEither(record.to[Element]))
      .map(_.updateTemps(fahrenheitToCelsius))
      .map(_.toRecord) // toRecord is actually an extension method
      .toFs2Stream
      .through(csvRendererPipe)
      .through(Writer[Task].write(Paths.get(celsiusCSV)))

  val run =
    ZIO.log(s"Processing $fahrenheitCSV")
      *> processor.compile.drain

```
</div>

---
transition: slide-left
layout: default
---

### Converting a case class to a CSV `Record` using **ZIO Schema**

<div class="flex h-4/5 w-full items-center">
```scala {4-11|11|13-15|14|15}{maxHeight:'400px'}
import info.fingo.spata.Record
import zio.schema.*

final case class Element(element: String, symbol: String, meltingTemp: Double, boilingTemp: Double):
  self =>

  def updateTemps(f: Double => Double) =
    self.copy(meltingTemp = f(self.meltingTemp), boilingTemp = f(self.boilingTemp))

object Element:
  given Schema.Record[Element] = DeriveSchema.gen[Element]

extension [A](a: A)
  def toRecord(using schema: Schema.Record[A]) =
    Record.fromPairs(schema.fields.map(field => field.fieldName -> field.get(a).toString)*)

```
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