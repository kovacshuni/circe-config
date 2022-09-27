# circe-config

[![Latest Version Badge]][Latest Version]

Small library for translating between [HOCON], [Java properties], and JSON
documents and circe's JSON AST.

At a high-level it can be used as a [circe] powered front-end for the [Typesafe
config] library to enable boilerplate free loading of settings into Scala types.
More generally it provides parsers and printers for interoperating with
[Typesafe config]'s JSON AST.

 [HOCON]: https://github.com/lightbend/config/blob/master/HOCON.md
 [Java properties]: https://docs.oracle.com/javase/8/docs/api/java/util/Properties.html

## Usage

To use this library configure your sbt project with the following line:

```sbt
libraryDependencies += "com.hunorkovacs" %% "circe-config" % "0.10.0"
```

Compatible with Scala 3, 2.13, 2.12.

More specific version table:

| circe-config | Scala             | cats-effect | circe  |
| ------------ | ----------------- | ----------- | ------ |
| 0.8.0        | 2.12, 2.13        | 2.5.4       | 0.14.1 |
| 0.9.0        | 2.12, 2.13, 3.1.0 | 2.5.4       | 0.14.1 |
| 0.10.0       | 2.12, 2.13, 3.1.0 | 3.2.8       | 0.14.1 |

## Documentation

 - [API docs](https://circe.github.io/circe-config/io/circe/config/index.html)

## Example

The following examples use `io.circe:circe-generic` as a dependency to
automatically derive decoders. They load the configuration found in
[application.conf].

```scala
scala> import com.typesafe.config.{ ConfigFactory, ConfigMemorySize }
scala> import io.circe.generic.auto._
scala> import io.circe.config.syntax._
scala> import scala.concurrent.duration.FiniteDuration

scala> case class ServerSettings(host: String, port: Int, timeout: FiniteDuration, maxUpload: ConfigMemorySize)
scala> case class HttpSettings(server: ServerSettings, version: Option[Double])
scala> case class AppSettings(http: HttpSettings)

// Load default configuration and decode instances
scala> import io.circe.config.parser

scala> parser.decode[AppSettings]()
res0: Either[io.circe.Error,AppSettings] = Right(AppSettings(HttpSettings(ServerSettings(localhost,8080,5 seconds,ConfigMemorySize(5242880)),Some(1.1))))

scala> parser.decodePath[ServerSettings]("http.server")
res1: Either[io.circe.Error,ServerSettings] = Right(ServerSettings(localhost,8080,5 seconds,ConfigMemorySize(5242880)))

scala> val config = ConfigFactory.load()

// Decode instances from an already loaded configuration

scala> config.as[ServerSettings]("http.server")
res2: Either[io.circe.Error,ServerSettings] = Right(ServerSettings(localhost,8080,5 seconds,ConfigMemorySize(5242880)))

scala> config.as[HttpSettings]("http")
res3: Either[io.circe.Error,HttpSettings] = Right(HttpSettings(ServerSettings(localhost,8080,5 seconds,ConfigMemorySize(5242880)),Some(1.1)))

scala> config.as[AppSettings]
res4: Either[io.circe.Error,AppSettings] = Right(AppSettings(HttpSettings(ServerSettings(localhost,8080,5 seconds,ConfigMemorySize(5242880)),Some(1.1))))
```

If you are using [`cats.effect.IO`], or some other type `F[_]` that provides a
[`cats.ApplicativeError`], you can use the following:

```scala
scala> import cats.effect.unsafe.implicits.global
scala> import cats.effect.IO
scala> import io.circe.generic.auto._
scala> import io.circe.config.parser

scala> case class ServerSettings(host: String, port: Int)
scala> case class HttpSettings(server: ServerSettings, version: Option[Double])
scala> case class AppSettings(http: HttpSettings)

scala> parser.decodeF[IO, AppSettings]()
res0: cats.effect.IO[AppSettings] = IO(AppSettings(HttpSettings(ServerSettings(localhost,8080),Some(1.1))))

scala> val settings: IO[AppSettings] = parser.decodeF[IO, AppSettings]()
scala> settings.unsafeRunSync()
res1: AppSettings = AppSettings(HttpSettings(ServerSettings(localhost,8080),Some(1.1)))

scala> parser.decodePathF[IO, ServerSettings]("http.server")
res2: cats.effect.IO[ServerSettings] = IO(ServerSettings(localhost,8080))

scala> parser.decodePathF[IO, ServerSettings]("path.not.found").attempt.unsafeRunSync()
res3: Either[Throwable, ServerSettings] = Left(io.circe.ParsingFailure: Path not found in config)
```

This makes the configuration directly available in your `F[_]`, such as `cats.effect.IO`, which handles any errors.

[application.conf]: https://github.com/circe/circe-config/tree/master/src/test/resources/application.conf
[`cats.effect.IO`]: https://typelevel.org/cats-effect/datatypes/io.html
[`cats.ApplicativeError`]: https://typelevel.org/cats/api/cats/ApplicativeError.html

## Contributing

Contributions are very welcome. Please see [instructions](CONTRIBUTING.md) on
how to create issues and submit patches.

## Releasing

Have your ~/.sbt/1.0/credentials.sbt contain `credentials += Credentials("Sonatype Nexus Repository Manager", "oss.sonatype.org", "yourusername", "yourpassword")`

Comment out the snapshot possiblity in `build.sbt`. Somehow for me it's always seeing it as snapshot and will give HTTP 400 on Sonatype Nexus.
```
publishTo := Some {
  // if (isSnapshot.value)
  //   Opts.resolver.sonatypeSnapshots
  // else
    Opts.resolver.sonatypeStaging
}
```

To release version `x.y.z` run:

    > sbt -Dproject.version=x.y.z release

## License

Note from @kovacshuni: The original repo is [circe/circe-config](https://github.com/circe/circe-config). This repository was cloned only to release the [Scala 3 compatiblity change](https://github.com/circe/circe-config/pull/252), a feature mostly developed by @vhiairrassary, to a public repository (Sonatype) so that it's easy to access for the general public.

circe-config is licensed under the **[Apache License, Version 2.0][apache]** (the
"License"); you may not use this software except in compliance with the License.

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

 [apache]: http://www.apache.org/licenses/LICENSE-2.0
 [circe]: https://github.com/circe/circe
 [Typesafe config]: https://github.com/lightbend/config
 [CI]: https://github.com/circe/circe-config/actions
 [CI Status]: https://img.shields.io/github/workflow/status/circe/circe-config/Continuous%20Integration.svg
 [Latest Version Badge]: https://img.shields.io/maven-central/v/com.hunorkovacs/circe-config_3.svg
 [Latest Version]: https://maven-badges.herokuapp.com/maven-central/com.hunorkovacs/circe-config_3
