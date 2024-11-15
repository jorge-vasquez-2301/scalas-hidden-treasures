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
---

### Scala's Hidden Treasures
### Five ZIO-Compatible Libraries you didn't know you needed!

<style>
  .slidev-layout.cover {
      @apply !bg-[url('/background.png')] pl-20 pt-20
  }
</style>

<b>December 2024</b>

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
```csv {all} {maxHeight:'300px'}
element,symbol,melting temperature [F],boiling temperature [F]
hydrogen,H,13.99,20.271
helium,He,0.95,4.222
lithium,Li,453.65,1603
```
</div>

---
transition: slide-left
layout: default
---

## Domain modelling

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3|4|5|6|8|10-14|16-17|17}{maxHeight:'300px'}
//> using jvm graalvm-java23:23.0.0
//> using scala 3.5.2
//> using dep info.fingo::spata:3.2.1
//> using dep dev.zio::zio-interop-cats:23.1.0.3
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
```scala {1|2|3|5-10}{maxHeight:'300px'}
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
```scala {5-10|6|7|8|9|12-15|13|14|15|17}{maxHeight:'300px'}
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
```scala {1|2|3|5-10}{maxHeight:'300px'}
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
```scala {5-10|6|7|8|9|12-16|13|14|15|16|18}{maxHeight:'300px'}
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

<div class="flex h-4/5 w-full items-center">
```scala {1-7|9|11-16|18-19|21-33|24-25|26|27|28|22,29|30|31|32|33|35-37|37}{maxHeight:'300px'}
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
```scala {4-11|11|13-15|14|15}{maxHeight:'300px'}
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
layout: cover
---

## Type-safe processing of Parquet files with <br/> **ZIO Apache Parquet**

---
transition: slide-left
layout: default
---

### Type-safe processing of Parquet files with **ZIO Apache Parquet**

<div class="mt-4 flex h-4/5 w-full items-center gap-5 text-justify">
  <ul>
    <li v-click><b>ZIO-powered</b> wrapper for Apache Parquet's Java implementation</li>
    <li v-click>Leverages <b>ZIO Schema Derivation</b> to automatically derive codecs</li>
    <li v-click>Leverages <b>ZIO Schema Accessors</b> to provide type-safe filter predicates</li>
    <li v-click><b>No Spark required</b> to read/write Parquet files</li>
  </ul>
</div>

---
transition: slide-left
layout: default
---

## Domain modelling

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3|4|5|6|7|9|10|11|12|14-18|21-31|33-34|36-37|39-40|42-43|45-47}{maxHeight:'300px'}
//> using jvm graalvm-java11:22.3.3
//> using scala 3.5.2
//> using dep me.mnedokushev::zio-apache-parquet-core:0.1.4
//> using dep info.fingo::spata:3.2.1
//> using dep dev.zio::zio-interop-cats:23.1.0.3
//> using dep dev.zio::zio-schema:1.5.0
//> using dep dev.zio::zio-schema-derivation:1.5.0

import me.mnedokushev.zio.apache.parquet.core.codec.*
import me.mnedokushev.zio.apache.parquet.core.filter.*
import info.fingo.spata.Record
import zio.schema.*

final case class Element(element: String, symbol: String, meltingTemp: Double, boilingTemp: Double):
  self =>

  def updateTemps(f: Double => Double) =
    self.copy(meltingTemp = f(self.meltingTemp), boilingTemp = f(self.boilingTemp))

object Element:
  given schema: Schema.CaseClass4.WithFields[
    "element",
    "symbol",
    "meltingTemp",
    "boilingTemp",
    String,
    String,
    Double,
    Double,
    Element
  ] = DeriveSchema.gen[Element]

  // SchemaEncoder is used to generate the corresponding Parquet Schema when writing files
  given schemaEncoder: SchemaEncoder[Element] = Derive.derive[SchemaEncoder, Element](SchemaEncoderDeriver.default)

  // Element => Value
  given ValueEncoder[Element] = Derive.derive[ValueEncoder, Element](ValueEncoderDeriver.default)

  // Value => Element
  given ValueDecoder[Element] = Derive.derive[ValueDecoder, Element](ValueDecoderDeriver.default)

  given TypeTag[Element]                          = Derive.derive[TypeTag, Element](TypeTagDeriver.default)
  val (element, symbol, meltingTemp, boilingTemp) = Filter[Element].columns

extension [A](a: A)
  def toRecord(using schema: Schema.Record[A]) =
    Record.fromPairs(schema.fields.map(field => field.fieldName -> field.get(a).toString)*)

```
</div>

---
transition: slide-left
layout: default
---

## Reading from CSV and processing

<div class="flex h-4/5 w-full items-center">
```scala {1-8|10|12-16|18|20-28}{maxHeight:'300px'}
import java.nio.file.Paths
import info.fingo.spata.{ CSVParser, Record }
import info.fingo.spata.io.Reader
import zio.schema.*
import zio.*
import zio.stream.*
import zio.interop.catz.*
import zio.stream.interop.fs2z.*

type Eff[A] = RIO[Scope, A]

val csvParser: fs2.Pipe[Eff, Char, Record] =
  CSVParser.config
    .mapHeader(Map("melting temperature [F]" -> "meltingTemp", "boiling temperature [F]" -> "boilingTemp"))
    .parser[Eff]
    .parse

val elementsFahrenheitCSVFile = Paths.get("testdata/elements-fahrenheit.csv")

val readFromCsvAndProcess: ZStream[Scope, Throwable, Element] =
  def fahrenheitToCelsius(f: Double): Double = (f - 32.0) * (5.0 / 9.0)

  Reader[Eff]
    .read(elementsFahrenheitCSVFile)
    .through(csvParser)
    .toZStream()
    .mapZIO(record => ZIO.fromEither(record.to[Element]))
    .map(_.updateTemps(fahrenheitToCelsius))
```
</div>

---
transition: slide-left
layout: default
---

## Working with Parquet

<div class="flex h-4/5 w-full items-center">
```scala {1-2|3-6|7-12|14-19|21-28|30|33-34|35-40|41-43|44-45|46-52|50|53-54}{maxHeight:'300px'}
import info.fingo.spata.{ CSVRenderer, Record }
import info.fingo.spata.io.Writer
import me.mnedokushev.zio.apache.parquet.core.hadoop.{ ParquetReader, ParquetWriter, Path }
import org.apache.parquet.hadoop.*
import me.mnedokushev.zio.apache.parquet.core.filter.syntax.*
import me.mnedokushev.zio.apache.parquet.core.filter.*
import java.nio.file.Paths
import zio.schema.*
import zio.*
import zio.stream.*
import zio.interop.catz.*
import zio.stream.interop.fs2z.*

val csvRenderer: fs2.Pipe[Eff, Record, Char] =
  CSVRenderer.config
    .fieldDelimiter(';')
    .mapHeader(Map("meltingTemp" -> "melting temperature [C]", "boilingTemp" -> "boiling temperature [C]"))
    .renderer[Eff]
    .render

def writeToCsv(stream: ZStream[Scope, Throwable, Element], path: java.nio.file.Path) =
  stream
    .map(_.toRecord)
    .toFs2Stream
    .through(csvRenderer)
    .through(Writer[Eff].write(path))
    .compile
    .drain

val processor: RIO[ParquetWriter[Element] & ParquetReader[Element], Unit] =
  ZIO.scoped {
    for
      _                      <- ZIO.log(s"Writing all elements to $elementsCelsiusParquetFile, records' schema will be:")
      _                      <- ZIO.log(Element.schemaEncoder.encode(Element.schema, "element", optional = false).toString)
      //                      required group element {
      //                        required binary element (STRING);
      //                        required binary symbol (STRING);
      //                        required double meltingTemp;
      //                        required double boilingTemp;
      //                      }
      _                      <- ZIO.serviceWithZIO[ParquetWriter[Element]] {
                                  _.writeStream(elementsCelsiusParquetFile, readFromCsvAndProcess)
                                }
      _                      <- ZIO.log(s"Reading all elements from $elementsCelsiusParquetFile")
      allElementsStream      <- ZIO.serviceWith[ParquetReader[Element]](_.readStream(elementsCelsiusParquetFile))
      _                      <- ZIO.log(s"Reading and Filtering elements from $elementsCelsiusParquetFile")
      filteredElementsStream <- ZIO.serviceWith[ParquetReader[Element]] {
                                  _.readStreamFiltered(
                                    elementsCelsiusParquetFile,
                                    filter(Element.element =!= "hydrogen" `and` Element.meltingTemp > 0)
                                  )
                                }
      _                      <- ZIO.log(s"Writing filtered elements to $elementsCelsiusFilteredCSVFile")
      _                      <- writeToCsv(filteredElementsStream, elementsCelsiusFilteredCSVFile)
    yield ()
  }
```
</div>

---
transition: slide-left
layout: default
---

## Providing layers

<div class="flex h-4/5 w-full items-center">
```scala {1|5|7-12|9|10|11}{maxHeight:'300px'}
object ZIOApacheParquetExample extends ZIOAppDefault:
  
  ...

  val processor: RIO[ParquetWriter[Element] & ParquetReader[Element], Unit] = ...

  override def run =
    ZIO.log(s"Processing $elementsFahrenheitCSVFile")
      *> processor.provide(
        ParquetWriter.configured[Element](writeMode = ParquetFileWriter.Mode.OVERWRITE),
        ParquetReader.configured[Element]()
      )
```
</div>

---
transition: slide-left
layout: cover
---

## YAML parsing with **yaml4s**

---
transition: slide-left
layout: default
---

## YAML parsing with **yaml4s**

<div class="mt-4 flex h-4/5 w-full items-center gap-5 text-justify">
  <ul>
    <li v-click><b>yaml4s</b> allows to parse a YAML string as a `YAML` data type or Json</li>
    <li v-click>
      To parse <b>YAML as Json</b>, several libraries are supported
       <ul>
        <li>zio-json</li>
        <li>circe</li>
        <li>play-json</li>
        <li>spray-json</li>
        <li>json4s</li>
      </ul>
    </li>
  </ul>
</div>

---
transition: slide-left
layout: default
---

## **Example:** Integration with zio-json

<div class="flex h-4/5 w-full items-center">
```yaml {1-16|17-26|27-35}{maxHeight:'300px'}
- id: 671653478d28c542477d142f
  index: 0
  guid: 6e73efbe-7550-45e9-afae-d2a18d7e904c
  isActive: true
  balance: "$$1,884.35"
  picture: "http://placehold.it/32x32"
  age: 28
  eyeColor: green
  name: Olson Browning
  gender: male
  company: TSUNAMIA
  email: olsonbrowning@tsunamia.com
  phone: +1 (978) 403-3228
  address: "704 Malbone Street, Comptche, Arkansas, 4153"
  about: "Et ea reprehenderit nostrud eiusmod nisi adipisicing. Laborum ipsum proident mollit sint laborum. Elit aliqua dolore occaecat sint in do. Id ex consectetur dolor commodo cupidatat. Sunt esse in laborum id. Ipsum velit nisi quis magna do laborum pariatur nisi excepteur amet dolore incididunt do."
  registered: "2024-08-18T12:48:25 +04:00"
  latitude: 31.330656
  longitude: 58.974609
  tags:
    - cupidatat
    - ex
    - officia
    - veniam
    - reprehenderit
    - qui
    - nisi
  friends:
    - id: 0
      name: Lancaster Cabrera
    - id: 1
      name: Bernadette Drake
    - id: 2
      name: Pruitt Rocha
  greeting: "Hello, Olson Browning! You have 10 unread messages."
  favoriteFruit: strawberry
```
</div>

---
transition: slide-left
layout: default
---

## Domain modelling

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3-4|5-6|7-8|9-10|11|13|15|17-32|33-40}{maxHeight:'300px'}
//> using jvm graalvm-java23:23.0.0
//> using scala 3.5.2
// Cross platform backend
//> using dep dev.hnaderi::yaml4s-backend:0.3.0
// JVM-only backend
//> using dep dev.hnaderi::yaml4s-snake:0.3.0
// Native-only backend
//> using dep dev.hnaderi::yaml4s-libyaml:0.3.0
// JS-only backend
//> using dep dev.hnaderi::yaml4s-jsyaml:0.3.0
//> using dep dev.hnaderi::yaml4s-zio-json:0.3.0

import zio.json.*

final case class Friend(id: Int, name: String) derives JsonCodec

final case class Person(
  id: String,
  index: Int,
  guid: String,
  isActive: Boolean,
  balance: String,
  picture: String,
  age: Int,
  eyeColor: String,
  name: String,
  gender: String,
  company: String,
  email: String,
  phone: String,
  address: String,
  about: String,
  registered: String,
  latitude: Double,
  longitude: Double,
  tags: List[String],
  friends: List[Friend],
  greeting: String,
  favoriteFruit: String
) derives JsonCodec

```
</div>

---
transition: slide-left
layout: default
---

## Processing the YAML file

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3-4|5|6|8|9-14|10-11|12|13|14|16-19|17|18|19|21|22|24-36|26-27|28|29|30|31|32|33|34-35}{maxHeight:'300px'}
import zio.*
import zio.stream.*
import zio.json.*
import zio.json.ast.*
import dev.hnaderi.yaml4s.*
import dev.hnaderi.yaml4s.ziojson.*

object Yaml4sExample extends ZIOAppDefault:
  def readFile(fileName: String): Task[String] =
    ZStream
      .fromFileName(fileName)
      .via(ZPipeline.utf8Decode)
      .runCollect
      .map(_.mkString)

  def printFile(contents: String, fileName: String): Task[Long] =
    ZStream(contents)
      .via(ZPipeline.utf8Encode)
      .run(ZSink.fromFileName(fileName))

  val peopleYaml         = "testdata/people.yaml"
  val filteredPeopleYaml = "testdata/filteredPeople.yaml"

  val run =
    for
      _                    <- ZIO.log(s"Processing $peopleYaml")
      yamlStr              <- readFile(peopleYaml)
      yaml                 <- ZIO.fromEither(Backend.parse[YAML](yamlStr))
      json                 <- ZIO.fromEither(Backend.parse[Json](yamlStr))
      people               <- ZIO.fromEither(json.as[List[Person]])
      filteredPeople        = people.filter(_.tags.contains("est"))
      filteredPeopleJson   <- ZIO.fromEither(filteredPeople.toJsonAST)
      filteredPeopleYamlStr = Backend.print(filteredPeopleJson)
      _                    <- ZIO.log(s"Writing $filteredPeopleYaml")
      _                    <- printFile(filteredPeopleYamlStr, filteredPeopleYaml)
    yield ()

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

## **Contact me**

<div class="grid grid-cols-8 gap-4 items-center h-4/5 content-center text-2xl">
  <div class="col-span-1"><img src="/x.png" class="w-8" /></div> <div class="col-span-7">@jorvasquez2301</div>
  <div class="col-span-1"><img src="/linkedin.png" class="w-8" /></div> <div class="col-span-7">jorge-vasquez-2301</div>
  <div class="col-span-1"><img src="/email.png" class="w-8" /></div> <div class="col-span-7">jorge.vasquez@ziverge.com</div>
</div>

---
transition: slide-left
layout: cover
---

# Thank you!

<style>
  .slidev-layout.cover {
      @apply !bg-[url('/background.png')] pl-20 pt-20
  }
</style>