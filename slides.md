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

<div class="mt-4 flex flex-col h-4/5 w-full items-center gap-5 text-justify">
  <p><b>5 libraries</b> to handle diverse data-processing tasks</p>
  <ul>
    <li v-click>Stream-based CSV processing with <b>Spata</b></li>
    <li v-click>Type-safe processing of Parquet files with <b>ZIO Apache Parquet</b></li>
    <li v-click>YAML processing with <b>YAML4s</b></li>
    <li v-click>High-performance embedded key-value stores with <b>ZIO LMDB</b></li> 
    <li v-click>Integration between Doobie and ZIO with <b>Tranzactio</b></li>
  </ul>
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/cageLetsGo.gif" class="h-70 rounded-md"/></div>
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

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/inAction.jpg" class="h-70 rounded-md"/></div>
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
```csv {1|2-4} {maxHeight:'300px'}
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
```scala {1|2|3|4|5-6|8-12|8|11-12}{maxHeight:'300px'}
//> using jvm graalvm-java23:23.0.0
//> using scala 3.5.2
//> using dep info.fingo::spata:3.2.1
//> using dep dev.zio::zio-interop-cats:23.1.0.3
//> using dep dev.zio::zio-schema:1.5.0
//> using dep dev.zio::zio-schema-derivation:1.5.0

final case class Element(element: String, symbol: String, meltingTemp: Double, boilingTemp: Double):
  self =>

  def updateTemps(f: Double => Double) =
    self.copy(meltingTemp = f(self.meltingTemp), boilingTemp = f(self.boilingTemp))

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

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/customizeParser.jpg" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

## Customizing the `CSVParser`

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

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/customRenderer.jpg" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

## Customizing the `CSVRenderer`

<div class="flex h-4/5 w-full items-center">
```scala {5-10|12-16|13|14|15|16|18}{maxHeight:'300px'}
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

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/bean.gif" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/evenBetter.jpg" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/macros.jpg" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/zioSchema.jpg" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

### Converting a case class to a CSV `Record` using **ZIO Schema**

<div class="flex h-4/5 w-full items-center">
```scala {4-11|11}{maxHeight:'300px'}
import info.fingo.spata.Record
import zio.schema.*

final case class Element(element: String, symbol: String, meltingTemp: Double, boilingTemp: Double):
  self =>

  def updateTemps(f: Double => Double) =
    self.copy(meltingTemp = f(self.meltingTemp), boilingTemp = f(self.boilingTemp))

object Element:
  given Schema[Element] = DeriveSchema.gen[Element]

```
</div>

---
transition: slide-left
layout: default
---

### Converting a case class to a CSV `Record` using **ZIO Schema**

<div class="flex h-4/5 w-full items-center">
```scala {11-12|14-16|15|16}{maxHeight:'300px'}
import info.fingo.spata.Record
import zio.schema.*

final case class Element(element: String, symbol: String, meltingTemp: Double, boilingTemp: Double):
  self =>

  def updateTemps(f: Double => Double) =
    self.copy(meltingTemp = f(self.meltingTemp), boilingTemp = f(self.boilingTemp))

object Element:
  // I can use a more specific type, because DeriveSchema.gen is a transparent macro!
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
    <li v-click><b>Spark is not required</b> to read/write Parquet files</li>
    <li v-click>Supports <b>streaming</b></li>
  </ul>
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/picard.jpg" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

## Domain modelling

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3|4|5|6-7|9-10|11|12|14-18|21-31|33-34|36-37|39-40|42-43}{maxHeight:'300px'}
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

```
</div>

---
transition: slide-left
layout: default
---

## Reading from CSV and processing

<div class="flex h-4/5 w-full items-center">
```scala {1-8|10|12-16|18-26}{maxHeight:'300px'}
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

## Writing to CSV

<div class="flex h-4/5 w-full items-center">
```scala {1-8|10-15|17-24}{maxHeight:'300px'}
import info.fingo.spata.{ CSVRenderer, Record }
import info.fingo.spata.io.Writer
import me.mnedokushev.zio.apache.parquet.core.hadoop.{ ParquetReader, ParquetWriter, Path }
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

def writeStreamToCsv(stream: ZStream[Scope, Throwable, Element], path: java.nio.file.Path) =
  stream
    .map(_.toRecord)
    .toFs2Stream
    .through(csvRenderer)
    .through(Writer[Eff].write(path))
    .compile
    .drain
```
</div>

---
transition: slide-left
layout: default
---

## Working with Parquet

<div class="flex h-4/5 w-full items-center">
```scala {1-5|7-9|11|14-15|16|17|18-23|24-26|27-28|29-30|31-37|35|38-44|45-46}{maxHeight:'300px'}
import me.mnedokushev.zio.apache.parquet.core.hadoop.{ ParquetReader, ParquetWriter, Path }
import org.apache.parquet.hadoop.*
import me.mnedokushev.zio.apache.parquet.core.filter.syntax.*
import me.mnedokushev.zio.apache.parquet.core.filter.*
import zio.*

val fahrenheitCSVFile      = Paths.get("testdata/elements-fahrenheit.csv")
val celsiusParquetFile     = Path(Paths.get("testdata/elements-celsius.parquet"))
val celsiusFilteredCSVFile = Paths.get("testdata/elements-celsius-filtered.csv")

val processor: RIO[ParquetWriter[Element] & ParquetReader[Element], Unit] =
  ZIO.scoped {
    for
      _                      <- ZIO.log(s"Writing all elements to $celsiusParquetFile")
      _                      <- ZIO.log(s"The Parquet schema will be:")
      parquetSchema           = Element.schemaEncoder.encode(Element.schema, "element", optional = false)
      _                      <- ZIO.log(parquetSchema.toString)
      //                      required group element {
      //                        required binary element (STRING);
      //                        required binary symbol (STRING);
      //                        required double meltingTemp;
      //                        required double boilingTemp;
      //                      }
      _                      <- ZIO.serviceWithZIO[ParquetWriter[Element]] {
                                  _.writeStream(celsiusParquetFile, readFromCsvAndProcess)
                                }
      _                      <- ZIO.log(s"Reading all elements from $celsiusParquetFile, as a Chunk")
      allElementsChunk       <- ZIO.serviceWith[ParquetReader[Element]](_.readChunk(celsiusParquetFile))
      _                      <- ZIO.log(s"Reading all elements from $celsiusParquetFile, as a ZStream")
      allElementsStream      <- ZIO.serviceWith[ParquetReader[Element]](_.readStream(celsiusParquetFile))
      _                      <- ZIO.log(s"Reading/filtering elements from $celsiusParquetFile, as a Chunk")
      filteredElementsChunk  <- ZIO.serviceWith[ParquetReader[Element]] {
                                  _.readChunkFiltered(
                                    celsiusParquetFile,
                                    filter(Element.element =!= "hydrogen" `and` Element.meltingTemp > 0)
                                  )
                                }
      _                      <- ZIO.log(s"Reading/filtering elements from $celsiusParquetFile, as a ZStream")
      filteredElementsStream <- ZIO.serviceWith[ParquetReader[Element]] {
                                  _.readStreamFiltered(
                                    celsiusParquetFile,
                                    filter(Element.element =!= "hydrogen" `and` Element.meltingTemp > 0)
                                  )
                                }
      _                      <- ZIO.log(s"Writing filtered elements to $celsiusFilteredCSVFile")
      _                      <- writeStreamToCsv(filteredElementsStream, celsiusFilteredCSVFile)
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
    <li v-click><b>yaml4s</b> allows to parse a <b>YAML string</b> as a `YAML` data type or Json</li>
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

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/picardInAction.jpg" class="h-70 rounded-md"/></div>
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
```scala {1|2|3-4|5|6|8|9-14|16-19|21|22|24-36|26-27|28|29|30|31|32|33|34-35}{maxHeight:'300px'}
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

  def writeFile(contents: String, fileName: String): Task[Long] =
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
      _                    <- writeFile(filteredPeopleYamlStr, filteredPeopleYaml)
    yield ()

```
</div>

---
transition: slide-left
layout: cover
---

## High-performance embedded key-value stores with <b>ZIO LMDB</b>

---
transition: slide-left
layout: default
---

## High-performance embedded key-value stores with <b>ZIO LMDB</b>

<div class="mt-4 flex h-3/5 w-full items-center gap-5 text-justify">
  <ul>
    <li v-click>Lightning Memory-Mapped Database</li>
    <li v-click>Embedded key/value store</li>
    <li v-click>Based on the <b>lmdb-java</b> library, it brings a higher-level ZIO API</li>
    <li v-click>JSON based storage using <b>zio-json</b></li>
    <li v-click>Very precise error management thanks to <b>Union Types</b></li>
    <li v-click>Supports <b>streaming</b></li>
  </ul>
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/letsDoThis.gif" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

## Domain modelling

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3|4|5|6|8|10-19|15}{maxHeight:'300px'}
//> using jvm graalvm-java23:23.0.0
//> using javaOpt "--add-opens", "java.base/java.nio=ALL-UNNAMED", "--add-opens", "java.base/sun.nio.ch=ALL-UNNAMED"
//> using scala 3.5.2
//> using dep fr.janalyse::zio-lmdb:1.8.2
//> using dep info.fingo::spata:3.2.1
//> using dep dev.zio::zio-interop-cats:23.1.0.3

import zio.json.*

final case class Element(
  element: String,
  symbol: String,
  meltingTemp: Double,
  boilingTemp: Double
) derives JsonCodec:
  self =>

  def updateTemps(f: Double => Double) =
    self.copy(meltingTemp = f(self.meltingTemp), boilingTemp = f(self.boilingTemp))

```
</div>

---
transition: slide-left
layout: default
---

## Playing with **ZIO-LMDB**

<div class="flex h-4/5 w-full items-center">
```scala {1-2|3|5|7|9-11|12-15|16|17|18|19|20|21-24|25-26|27-28|29-30|31-32}{maxHeight:'300px'}
import zio.*
import zio.json.*
import zio.lmdb.*

val collectionName = "elements"

val program =
  for
    _                    <- LMDB.collectionExists(
                              collectionName
                            ) @@ ZIOAspect.logged(s"Checking whether collection $collectionName exists")
    elements             <- LMDB.collectionCreate[Element](
                              collectionName,
                              failIfExists = false
                            ) @@ ZIOAspect.logged(s"Created collection $collectionName")
    _                    <- LMDB.collectionsAvailable() @@ ZIOAspect.logged("Available collections")
    _                    <- loadElementsFromCSV(elements)
    collected            <- elements.collect() @@ ZIOAspect.logged(s"Collected all $collectionName")
    collectionSize       <- elements.size() @@ ZIOAspect.logged(s"Number of collected $collectionName")
    _                    <- ZIO.foreach(collected)(Console.printLine(_))
    filtered             <- elements.collect(
                              keyFilter = _.startsWith("H"),
                              valueFilter = _.meltingTemp < -15
                            ) @@ ZIOAspect.logged(s"Filtered $collectionName")
    _                    <- ZIO.logInfo(s"Creating stream of elements for $collectionName")
    elementsStream        = elements.stream()
    _                    <- ZIO.logInfo(s"Creating stream of elements for $collectionName, including keys")
    symbolToElementStream = elements.streamWithKeys()
    _                    <- ZIO.log(s"Clearing collection $collectionName")
    _                    <- elements.clear()
    _                    <- ZIO.log(s"Dropping collection $collectionName")
    _                    <- LMDB.collectionDrop(collectionName)
  yield ()

```
</div>

---
transition: slide-left
layout: default
---

## Loading elements from CSV

<div class="flex h-4/5 w-full items-center">
```scala {1|3-7|9-11|14-22|15-16|17|18|19|12,20|21|22}{maxHeight:'300px'}
val fahrenheitCSV = "testdata/elements-fahrenheit.csv"

val csvParser: fs2.Pipe[Task, Char, Record] =
  CSVParser.config
    .mapHeader(Map("melting temperature [F]" -> "meltingTemp", "boiling temperature [F]" -> "boilingTemp"))
    .parser[Task]
    .parse

def loadElementsFromCSV(
  elements: LMDBCollection[Element]
): IO[Throwable | OverSizedKey | (CollectionNotFound | JsonFailure | StorageSystemError), Unit] =
  def fahrenheitToCelsius(f: Double): Double = (f - 32.0) * (5.0 / 9.0)

  ZIO.log(s"Loading $fahrenheitCSV into LMDB")
    *> Reader[Task]
        .read(Paths.get(fahrenheitCSV))
        .through(csvParser)
        .toZStream()
        .mapZIO(record => ZIO.fromEither(record.to[Element]))
        .map(_.updateTemps(fahrenheitToCelsius))
        .mapZIO(element => elements.upsertOverwrite(element.symbol, element))
        .runDrain

```
</div>

---
transition: slide-left
layout: default
---

## Providing layers

<div class="flex h-4/5 w-full items-center">
```scala {1|4|6-10|8|9}{maxHeight:'300px'}
object ZioLmdbExample extends ZIOAppDefault:
  ...

  val program = ...

  override def run =
    program.provide(
      LMDB.liveWithDatabaseName("elements-database"),
      Scope.default
    )

```
</div>

---
transition: slide-left
layout: cover
---

## Integration between Doobie and ZIO with **Tranzactio**

---
transition: slide-left
layout: default
---

## Integration between Doobie and ZIO with **Tranzactio**

<div class="mt-4 flex h-4/5 w-full items-center gap-5 text-justify">
  <ul>
    <li v-click><b>TranzactIO</b> is a ZIO wrapper for some Scala database access libraries</li>
    <li v-click><b>Doobie</b> and <b>Anorm</b>, for now</li>
    <li v-click>Supports <b>streaming</b></li>
  </ul>
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/beanDoubt.jpg" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

## **Example without TranzactIO** - Data modelling

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3|4|5|6|7|9|10|11|13-18|20-27}{maxHeight:'300px'}
//> using jvm graalvm-java11:22.3.3
//> using scala 3.5.2
//> using dep org.tpolecat::doobie-core:1.0.0-RC5
//> using dep org.tpolecat::doobie-h2:1.0.0-RC5
//> using dep dev.zio::zio:2.1.11
//> using dep dev.zio::zio-streams:2.1.11
//> using dep dev.zio::zio-interop-cats:23.1.0.3

// Based on https://github.com/typelevel/doobie/blob/main/modules/example/src/main/scala/example/FirstExample.scala
final case class Supplier(id: Int, name: String, street: String, city: String, state: String, zip: String)
final case class Coffee(name: String, supplierId: Int, price: Double, sales: Int, total: Int)

val suppliers =
  List(
    Supplier(101, "Acme, Inc.", "99 Market Street", "Groundsville", "CA", "95199"),
    Supplier(49, "Superior Coffee", "1 Party Place", "Mendocino", "CA", "95460"),
    Supplier(150, "The High Ground", "100 Coffee Lane", "Meadows", "CA", "93966")
  )

val coffees =
  List(
    Coffee("Colombian", 101, 7.99, 0, 0),
    Coffee("French_Roast", 49, 8.99, 0, 0),
    Coffee("Espresso", 150, 9.99, 0, 0),
    Coffee("Colombian_Decaf", 101, 8.99, 0, 0),
    Coffee("French_Roast_Decaf", 49, 9.99, 0, 0)
  )

```
</div>

---
transition: slide-left
layout: default
---

## **Example without TranzactIO** - Definining queries

<div class="flex h-4/5 w-full items-center">
```scala {1-2|4|5-10|12-13|15-16|18-19|21-23|24-33|34-43}{maxHeight:'300px'}
import doobie.*
import doobie.implicits.*

object Queries:
  def coffeesLessThan(price: Double): ConnectionIO[List[(String, String)]] =
    sql"""
      SELECT cof_name, sup_name
      FROM coffees JOIN suppliers ON coffees.sup_id = suppliers.sup_id
      WHERE price < $price
    """.query[(String, String)].to[List]

  def insertSuppliers(suppliers: List[Supplier]): ConnectionIO[Int] =
    Update[Supplier]("INSERT INTO suppliers VALUES (?, ?, ?, ?, ?, ?)").updateMany(suppliers)

  def insertCoffees(coffees: List[Coffee]): ConnectionIO[Int] =
    Update[Coffee]("INSERT INTO coffees VALUES (?, ?, ?, ?, ?)").updateMany(coffees)

  val allCoffees: ConnectionIO[List[Coffee]] =
    sql"SELECT cof_name, sup_id, price, sales, total FROM coffees".query[Coffee].to[List]

  val allCoffeesStream: fs2.Stream[ConnectionIO, Coffee] =
    sql"SELECT cof_name, sup_id, price, sales, total FROM coffees".query[Coffee].stream  

  val create: ConnectionIO[Int] =
    sql"""
      CREATE TABLE suppliers (
        sup_id   INT     NOT NULL PRIMARY KEY,
        sup_name VARCHAR NOT NULL,
        street   VARCHAR NOT NULL,
        city     VARCHAR NOT NULL,
        state    VARCHAR NOT NULL,
        zip      VARCHAR NOT NULL
      );
      CREATE TABLE coffees (
        cof_name VARCHAR NOT NULL,
        sup_id   INT     NOT NULL,
        price    DOUBLE  NOT NULL,
        sales    INT     NOT NULL,
        total    INT     NOT NULL
      );
      ALTER TABLE coffees
      ADD CONSTRAINT coffees_suppliers_fk FOREIGN KEY (sup_id) REFERENCES suppliers(sup_id);
    """.update.run

```
</div>

---
transition: slide-left
layout: default
---

### **Example without TranzactIO** - Transforming `ConnectionIO` to `ZIO`/`ZStream`

<div class="flex h-4/5 w-full items-center">
```scala {1-6|8|9-10|12-13|15-16|18-19|21-24|26-27}{maxHeight:'300px'}
import doobie.*
import doobie.implicits.*
import zio.*
import zio.stream.*
import zio.interop.catz.*
import zio.stream.interop.fs2z.*

object Repository:
  def coffeesLessThan(price: Double): RIO[Transactor[Task], List[(String, String)]] =
    ZIO.serviceWithZIO[Transactor[Task]](Queries.coffeesLessThan(price).transact(_))

  def insertSuppliers(suppliers: List[Supplier]): RIO[Transactor[Task], Int] =
    ZIO.serviceWithZIO[Transactor[Task]](Queries.insertSuppliers(suppliers).transact(_))

  def insertCoffees(coffees: List[Coffee]): RIO[Transactor[Task], Int] =
    ZIO.serviceWithZIO[Transactor[Task]](Queries.insertCoffees(coffees).transact(_))

  val allCoffees: RIO[Transactor[Task], List[Coffee]] =
    ZIO.serviceWithZIO[Transactor[Task]](Queries.allCoffees.transact(_))

  val allCoffeesStream: ZStream[Transactor[Task], Throwable, Coffee] =
    ZStream.serviceWithStream[Transactor[Task]] { transactor =>
      Queries.allCoffeesStream.transact(transactor).toZStream()
    }

  val create: RIO[Transactor[Task], Unit] =
    ZIO.serviceWithZIO[Transactor[Task]](Queries.create.transact(_)).unit

```
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/dry.jpg" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

### **Example without TranzactIO** - Transforming `ConnectionIO` to `ZIO`/`ZStream`

<div class="flex h-4/5 w-full items-center">
```scala {1-6|8|9-10|12-16|18-19|21-22|24-25|27-28|30-31|33-34}{maxHeight:'300px'}
import doobie.*
import doobie.implicits.*
import zio.*
import zio.stream.*
import zio.interop.catz.*
import zio.stream.interop.fs2z.*

object Repository:
  extension [A](connectionIO: ConnectionIO[A])
    def transactZIO: RIO[Transactor[Task], A] = ZIO.serviceWithZIO[Transactor[Task]](connectionIO.transact(_))

  extension [A](connectionIOStream: fs2.Stream[ConnectionIO, A])
    def transactZStream: ZStream[Transactor[Task], Throwable, A] =
      ZStream.serviceWithStream[Transactor[Task]] { transactor =>
        connectionIOStream.transact(transactor).toZStream()
      }

  def coffeesLessThan(price: Double): RIO[Transactor[Task], List[(String, String)]] =
    Queries.coffeesLessThan(price).transactZIO

  def insertSuppliers(suppliers: List[Supplier]): RIO[Transactor[Task], Int] =
    Queries.insertSuppliers(suppliers).transactZIO

  def insertCoffees(coffees: List[Coffee]): RIO[Transactor[Task], Int] =
    Queries.insertCoffees(coffees).transactZIO

  val allCoffees: RIO[Transactor[Task], List[Coffee]] =
    Queries.allCoffees.transactZIO

  val allCoffeesStream: ZStream[Transactor[Task], Throwable, Coffee] =
    Queries.allCoffeesStream.transactZStream

  val create: RIO[Transactor[Task], Unit] =
    Queries.create.transactZIO.unit

```
</div>

---
transition: slide-left
layout: default
---

## **Example without TranzactIO** - Running queries

<div class="flex h-4/5 w-full items-center">
```scala {1-6|8|9-21|11|12|13|14|15-16|17-18|19-20}{maxHeight:'300px'}
import zio.*
import zio.stream.*
import zio.interop.catz.*
import doobie.*
import doobie.implicits.*
import doobie.h2.H2Transactor

object DoobieExample extends ZIOAppDefault:
  val program: RIO[Transactor[Task], Unit] =
    for
      _                 <- Repository.create
      numberOfSuppliers <- Repository.insertSuppliers(suppliers)
      numberOfCoffees   <- Repository.insertCoffees(coffees)
      _                 <- ZIO.log(s"Inserted $numberOfSuppliers suppliers and $numberOfCoffees coffees")
      _                 <- ZIO.log("Getting all coffees")
      _                 <- ZStream.fromIterableZIO(Repository.allCoffees).mapZIO(Console.printLine(_)).runDrain
      _                 <- ZIO.log("Getting all coffees as ZStream")
      _                 <- Repository.allCoffeesStream.mapZIO(Console.printLine(_)).runDrain
      _                 <- ZIO.log("Getting cheap coffees")
      _                 <- ZStream.fromIterableZIO(Repository.coffeesLessThan(9.0)).mapZIO(Console.printLine(_)).runDrain
    yield ()
```
</div>

---
transition: slide-left
layout: default
---

## **Example without TranzactIO** - Providing layers

<div class="flex h-4/5 w-full items-center">
```scala {1-2|4-16|9|10|11|12|18}{maxHeight:'300px'}
object DoobieExample extends ZIOAppDefault:
  val program: RIO[Transactor[Task], Unit] = ...

  val transactorLayer: TaskLayer[Transactor[Task]] =
    ZLayer.scoped {
      ZIO.executorWith { executor =>
        H2Transactor
          .newH2Transactor[Task](
            url = "jdbc:h2:mem:test;DB_CLOSE_DELAY=-1",
            user = "sa",
            pass = "",
            connectEC = executor.asExecutionContext
          )
          .toScopedZIO
      }
    }

  val run = program.provide(transactorLayer)
```
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/dontNeedThat.jpg" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

### **Example without TranzactIO** - Combining queries into transactions

<div class="flex h-4/5 w-full items-center">
```scala {1|4-5|7-8|10-14}{maxHeight:'300px'}
object Queries:
  ...

  def insertSuppliers(suppliers: List[Supplier]): ConnectionIO[Int] =
    Update[Supplier]("INSERT INTO suppliers VALUES (?, ?, ?, ?, ?, ?)").updateMany(suppliers)

  def insertCoffees(coffees: List[Coffee]): ConnectionIO[Int] =
    Update[Coffee]("INSERT INTO coffees VALUES (?, ?, ?, ?, ?)").updateMany(coffees)

  def insertSuppliersAndCoffees(suppliers: List[Supplier], coffees: List[Coffee]): ConnectionIO[(Int, Int)] =
    for
      suppliers <- insertSuppliers(suppliers)
      coffees   <- insertCoffees(coffees)
    yield (suppliers, coffees)
```
</div>

---
transition: slide-left
layout: default
---

### **Example without TranzactIO** - Combining queries into transactions

<div class="flex h-4/5 w-full items-center">
```scala {1|7-8}{maxHeight:'300px'}
object Repository:
  extension [A](connectionIO: ConnectionIO[A])
    def transactZIO: RIO[Transactor[Task], A] = ZIO.serviceWithZIO[Transactor[Task]](connectionIO.transact)

  ...

  def insertSuppliersAndCoffees(suppliers: List[Supplier], coffees: List[Coffee]): RIO[Transactor[Task], (Int, Int)] =
    Queries.insertSuppliersAndCoffees(suppliers, coffees).transactZIO
```
</div>

---
transition: slide-left
layout: default
---

### **Example without TranzactIO** - Combining queries into transactions

<div class="flex h-4/5 w-full items-center">
```scala {1|4-13|7}{maxHeight:'300px'}
object DoobieExampleWithTransactions extends ZIOAppDefault:
  ...

  val program =
    for
      _                                    <- Repository.create
      (numberOfSuppliers, numberOfCoffees) <- Repository.insertSuppliersAndCoffees(suppliers, coffees)
      _                                    <- ZIO.log(s"Inserted $numberOfSuppliers suppliers and $numberOfCoffees coffees")
      _                                    <- ZIO.log("Getting all coffees")
      _                                    <- ZStream.fromIterableZIO(Repository.allCoffees).mapZIO(Console.printLine(_)).runDrain
      _                                    <- ZIO.log("Getting cheap coffees")
      _                                    <- ZStream.fromIterableZIO(Repository.coffeesLessThan(9.0)).mapZIO(Console.printLine(_)).runDrain
    yield ()
```
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/picardBetter.jpg" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/tranzactio.gif" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

### **Example with TranzactIO** - Transforming `ConnectionIO` to `ZIO`/`ZStream`

<div class="flex h-4/5 w-full items-center">
```scala {1-2|5-6|8-9|11-12|14-15|17-18|20-21}{maxHeight:'300px'}
import io.github.gaelrenoux.tranzactio.*
import io.github.gaelrenoux.tranzactio.doobie.*

object Repository:
  def coffeesLessThan(price: Double): TranzactIO[List[(String, String)]] =
    tzio(Queries.coffeesLessThan(price))

  def insertSuppliers(suppliers: List[Supplier]): TranzactIO[Int] =
    tzio(Queries.insertSuppliers(suppliers))

  def insertCoffees(coffees: List[Coffee]): TranzactIO[Int] =
    tzio(Queries.insertCoffees(coffees))

  val allCoffees: TranzactIO[List[Coffee]] =
    tzio(Queries.allCoffees)

  val allCoffeesStream: TranzactIOStream[Coffee] =
    tzioStream(Queries.allCoffeesStream)

  val create: TranzactIO[Unit] =
    tzio(Queries.create).unit

```
</div>

---
transition: slide-left
layout: default
---

<div class="flex w-full h-full justify-center items-center">
  <div><img src="/tranzactio2.jpg" class="h-70 rounded-md"/></div>
</div>

---
transition: slide-left
layout: default
---

### **Example with TranzactIO** - Transforming `ConnectionIO` to `ZIO`/`ZStream`

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3}{maxHeight:'300px'}
type TranzactIO[A] = ZIO[Connection, DbException, A]
type Connection    = Transactor[Task]
type TranzactIO[A] = ZIO[Transactor[Task], DbException, A]
```
</div>

---
transition: slide-left
layout: default
---

### **Example with TranzactIO** - Transforming `ConnectionIO` to `ZIO`/`ZStream`

<div class="flex h-4/5 w-full items-center">
```scala {1|2|3}{maxHeight:'300px'}
type TranzactIOSream[A] = ZStream[Connection, DbException, A]
type Connection         = Transactor[Task]
type TranzactIOSream[A] = ZStream[Transactor[Task], DbException, A]
```
</div>

---
transition: slide-left
layout: default
---

## **Example with TranzactIO** - Running queries

<div class="flex h-4/5 w-full items-center">
```scala {1-3|4-5|6-7|8|10|14|15|16|17|18-22|23-24|25-29}{maxHeight:'300px'}
import zio.*
import zio.stream.*
import zio.interop.catz.*
import doobie.*
import doobie.implicits.*
import io.github.gaelrenoux.tranzactio.*
import io.github.gaelrenoux.tranzactio.doobie.*
import org.h2.jdbcx.JdbcDataSource

object TranzactioExample extends ZIOAppDefault:

  val program: ZIO[Database, Throwable, Unit] =
    for
      _                 <- Database.transactionOrDie(Repository.create)
      numberOfSuppliers <- Database.transactionOrDie(Repository.insertSuppliers(suppliers))
      numberOfCoffees   <- Database.transactionOrDie(Repository.insertCoffees(coffees))
      _                 <- ZIO.log(s"Inserted $numberOfSuppliers suppliers and $numberOfCoffees coffees")
      _                 <- ZIO.log("Getting all coffees")
      _                 <- ZStream
                             .fromIterableZIO(Database.transactionOrDie(Repository.allCoffees))
                             .mapZIO(coffee => Console.printLine(coffee).orDie)
                             .runDrain
      _                 <- ZIO.log("Getting all coffees as ZStream")
      _                 <- Database.transactionOrDieStream(Repository.allCoffeesStream).mapZIO(Console.printLine(_)).runDrain
      _                 <- ZIO.log("Getting cheap coffees")
      _                 <- ZStream
                             .fromIterableZIO(Database.transactionOrDie(Repository.coffeesLessThan(9.0)))
                             .mapZIO(coffee => Console.printLine(coffee).orDie)
                             .runDrain
    yield ()
```
</div>

---
transition: slide-left
layout: default
---

### **Example with TranzactIO** - Combining queries into transactions

<div class="flex h-4/5 w-full items-center">
```scala {1|7-10}{maxHeight:'300px'}
object TranzactioExampleWithTransactions extends ZIOAppDefault:
  ...

  val program: ZIO[Database, DbException, Unit] =
    for
      _                                    <- Database.transactionOrDie(Repository.create)
      (numberOfSuppliers, numberOfCoffees) <- Database.transactionOrDie {
                                                Repository.insertSuppliers(suppliers)
                                                  <*> Repository.insertCoffees(coffees)
                                              }
      _                                    <- ZIO.log(s"Inserted $numberOfSuppliers suppliers and $numberOfCoffees coffees")
      _                                    <- ZIO.log("Getting all coffees")
      _                                    <- ZStream
                                                .fromIterableZIO(Database.transactionOrDie(Repository.allCoffees))
                                                .mapZIO(coffee => Console.printLine(coffee).orDie)
                                                .runDrain
      _                                    <- ZIO.log("Getting cheap coffees")
      _                                    <- ZStream
                                                .fromIterableZIO(Database.transactionOrDie(Repository.coffeesLessThan(9.0)))
                                                .mapZIO(coffee => Console.printLine(coffee).orDie)
                                                .runDrain
    yield ()
```
</div>

---
transition: slide-left
layout: default
---

## **Example with TranzactIO** - Providing layers

<div class="flex h-4/5 w-full items-center">
```scala {1-2|4|6-13|9|10|11}{maxHeight:'300px'}
object TranzactioExample extends ZIOAppDefault:
  val program: ZIO[Database, DbException, Unit] = ...

  val run = program.provide(Database.fromDatasource, dataSourceLayer)

  val dataSourceLayer =
    ZLayer.succeed {
      val dataSource = JdbcDataSource()
      dataSource.setURL("jdbc:h2:mem:test;DB_CLOSE_DELAY=-1")
      dataSource.setUser("sa")
      dataSource.setPassword("")
      dataSource
    }
```
</div>

---
transition: slide-left
layout: image-right
image: /summary.jpg
---

## **Summary**

<div class="mt-4 flex h-4/5 w-full items-center">
  <ul>
    <li v-click>We have seen <b>5 libraries</b> for data-processing in Scala 3, compatible with <code>ZIO</code>/<code>ZStream</code></li>
    <li v-click>You can seamlessly do <b>CSV</b>, <b>Parquet</b> and <b>YAML</b> processing, thanks to <b>Spata</b>, <b>zio-apache-parquet</b> and <b>yaml4s</b></li>
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
    <li v-click>You can integrate high-performance embedded key-value stores with <b>zio-lmdb</b> into your applications</li>
    <li v-click><b>TranzactIO</b> allows you to lift <code>ConnectionIO</code> from Doobie into <code>ZIO</code>/<code>ZStream</code> and do everything at the ZIO level, including transactions</li>
  </ul>
</div>

---
transition: slide-left
layout: image-right
image: /learn.jpg
---

## **Where to learn more**

<div class="flex h-4/5 w-full items-center">
  <ul>
    <li><a href="https://github.com/jorge-vasquez-2301/snippets-functional-scala-2024" target="_blank">scala-cli examples</a></li>
    <li><a href="https://github.com/fingo/spata" target="_blank">Spata repo</a></li>
    <li><a href="https://github.com/grouzen/zio-apache-parquet" target="_blank">zio-apache-parquet repo</a></li>
    <li><a href="https://github.com/hnaderi/yaml4s" target="_blank">yaml4s repo</a></li>
    <li><a href="https://github.com/dacr/zio-lmdb" target="_blank">zio-lmdb repo</a></li>
    <li><a href="https://github.com/gaelrenoux/tranzactio" target="_blank">Tranzactio repo</a></li>
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